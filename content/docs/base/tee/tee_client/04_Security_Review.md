# TEE Client 安全风险评审

## 1. 评审概述

### 1.1 评审范围

本评审覆盖 TEE Client 组件的以下模块：

| 模块 | 评审状态 | 说明 |
|------|----------|------|
| libteec.so | ✅ 已评审 | 系统组件 TEE API 库 |
| libteec_vendor.so | ✅ 已评审 | 芯片组件 TEE API 库 |
| cadaemon | ✅ 已评审 | CA 守护进程 (SA 8001) |
| teecd | ✅ 已评审 | TEE 代理服务 |
| tlogcat | ✅ 已评审 | TEE 日志服务 |

### 1.2 评审方法

- 静态代码分析
- 架构威胁建模
- 输入验证检查
- 依赖分析

### 1.3 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         信任边界示意图                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                    TEE Secure World (高信任区)                    │     │
│  │                     TA 代码、内核驱动                              │     │
│  │                                                                  │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │   TA 1   │  │   TA 2   │  │   ...   │  │  TZDriver│         │     │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                    │                                   │
│                          硬件强制隔离 (TrustZone)                        │
│                                    │                                   │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │                    REE (低信任区)                               │     │
│  │                    Linux 内核、普通应用                          │     │
│  │                                                                  │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │     │
│  │  │ cadaemon │  │  teecd   │  │ CA App   │  │  ...    │         │     │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │     │
│  │                                                                  │     │
│  │  ┌─────────────────────────────────────────────────────────────┐  │     │
│  │  │          libteec.so / libteec_vendor.so                      │  │     │
│  │  └─────────────────────────────────────────────────────────────┘  │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## 2. 攻击面分析

### 2.1 输入入口点

| 入口 | 类型 | 数据来源 | 处理模块 |
|------|------|----------|----------|
| IPC 接口 | OHOS IPC | CA 应用 | cadaemon |
| Unix Socket | 网络套接字 | libteec_vendor | teecd |
| TA 文件路径 | 文件路径 | CA 应用 | libteec/libteec_vendor |
| 共享内存参数 | 内存映射 | CA 应用 | cadaemon |
| 配置文件 | 文件 | 系统配置 | 各服务 |

### 2.2 敏感操作

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| 打开 `/dev/tee*` 设备 | 高 | 直接与 TEE 驱动交互 |
| TA 文件加载 | 高 | 加载安全应用代码 |
| CA 认证 | 高 | 验证客户端身份 |
| 共享内存映射 | 中 | 跨进程内存共享 |
| 系统调用参数 | 中 | ioctl 命令传递 |

---

## 3. 威胁模型

### 3.1 外部攻击者

**能力假设**：
- 能够发送恶意 CA 请求
- 无法直接访问 TEE 内部
- 无法篡改内核代码

**威胁**：
- 恶意参数注入
- 权限提升尝试
- 拒绝服务攻击

### 3.2 恶意 CA

**能力假设**：
- 合法注册的应用
- 可能传递恶意参数
- 可能尝试越权访问

**威胁**：
- 路径遍历攻击
- 参数溢出尝试
- 资源耗尽攻击

---

## 4. 安全风险清单

### 4.1 高风险项

#### 风险 1：TA 文件路径验证不严格

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **风险等级** | 高 |
| **影响范围** | libteec, libteec_vendor |
| **证据位置** | `interfaces/inner_api/tee_client_api.h:53-61` |

**问题描述**：
`context.ta_path` 支持指定 TA 文件路径，但验证可能不完整。

**触发条件**：
```c
// 恶意 CA 可能尝试
TEEC_Context context = {0};
context.ta_path = (uint8_t *)"/data/../../../etc/passwd";
```

**潜在影响**：
- 读取敏感文件
- 路径遍历攻击

**修复建议**：
```c
// 在 tee_client_app_load.c 中添加路径验证
static bool ValidateTaPath(const char *path) {
    // 1. 检查路径是否以 /data/ 开头
    if (strncmp(path, "/data/", 6) != 0) {
        return false;
    }
    // 2. 检查是否存在 ../
    if (strstr(path, "../") != NULL) {
        return false;
    }
    // 3. 使用 realpath 解析并验证
    char resolved[PATH_MAX];
    if (realpath(path, resolved) == NULL) {
        return false;
    }
    // 4. 验证解析后的路径仍在允许目录内
    if (strncmp(resolved, "/data/", 6) != 0) {
        return false;
    }
    return true;
}
```

---

#### 风险 2：共享内存大小检查不严格

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **风险等级** | 高 |
| **影响范围** | cadaemon |
| **证据位置** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp:643-648` |

**问题描述**：
`CheckSizeStatus` 函数中的整数溢出检查可能存在绕过。

**触发条件**：
```c
// 超大共享内存请求
TEEC_SharedMemory shm = {0};
shm.size = 0xFFFFFFFF;  // 4GB
```

**代码证据**：
```c
// cadaemon_service.cpp:643
static bool CheckSizeStatus(uint32_t shmInfoOffset, uint32_t refSize,
                            uint32_t totalSize, uint32_t memSize)
{
    return ((shmInfoOffset + refSize < shmInfoOffset) ||
            (shmInfoOffset + refSize < refSize) ||
            (shmInfoOffset + refSize > totalSize) ||
            (refSize > memSize));
}
```

**潜在影响**：
- 整数溢出导致检查绕过
- 内存分配失败或越界访问

**修复建议**：
使用安全的整数运算库或手动实现 safe_add：
```c
static bool SafeU32Add(uint32_t a, uint32_t b, uint32_t *result) {
    if (b > UINT32_MAX - a) {
        return false;  // 溢出
    }
    *result = a + b;
    return true;
}

static bool CheckSizeStatus(uint32_t shmInfoOffset, uint32_t refSize,
                            uint32_t totalSize, uint32_t memSize)
{
    uint32_t sum;
    if (!SafeU32Add(shmInfoOffset, refSize, &sum)) {
        return true;  // 检测到溢出
    }
    return (sum > totalSize) || (refSize > memSize);
}
```

---

#### 风险 3：Socket 连接无超时控制

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-003 |
| **风险等级** | 中 |
| **影响范围** | libteec_vendor |
| **证据位置** | `frameworks/libteec_vendor/tee_client_socket.c:220-240` |

**问题描述**：
`ConnectTeecdSocket` 中的连接重试机制可能存在拒绝服务风险。

**代码证据**：
```c
// tee_client_socket.c
#define TEECD_RETRY_COUNT 50
#define TEECD_RETRY_WAIT 200  // ms

static int32_t ConnectTeecdSocket(const char *socketName, CaRevMsg *caInfo)
{
    int32_t retry = TEECD_RETRY_COUNT;
    while (retry--) {
        int32_t s = socket(AF_UNIX, SOCK_STREAM, 0);
        if (s < 0) {
            continue;
        }
        if (connect(s, (struct sockaddr *)&addr, len) == 0) {
            return s;
        }
        close(s);
        usleep(TEECD_RETRY_WAIT * 1000);  // 200ms
    }
    return -1;
}
```

**潜在影响**：
- 50 次重试 * 200ms = 10 秒阻塞
- 恶意应用可导致 CA 启动缓慢

**修复建议**：
```c
// 添加总超时时间限制
#define CONNECT_TIMEOUT_MS 5000  // 5秒总超时

static int32_t ConnectTeecdSocket(const char *socketName, CaRevMsg *caInfo)
{
    uint64_t startTime = GetTickCount();
    int32_t retry = TEECD_RETRY_COUNT;
    
    while (retry--) {
        uint64_t elapsed = GetTickCount() - startTime;
        if (elapsed >= CONNECT_TIMEOUT_MS) {
            tloge("Connect to teecd timeout\n");
            return -1;
        }
        
        int32_t s = socket(AF_UNIX, SOCK_STREAM, 0);
        if (connect(s, ...) == 0) {
            return s;
        }
        close(s);
        
        uint64_t remaining = CONNECT_TIMEOUT_MS - elapsed;
        usleep(MIN(remaining, TEECD_RETRY_WAIT * 1000));
    }
    return -1;
}
```

---

### 4.2 中风险项

#### 风险 4：上下文数量限制绕过

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **风险等级** | 中 |
| **影响范围** | libteec_vendor |
| **证据位置** | `frameworks/include/tee_client_inner.h:68` |

**问题描述**：
每个 CA 最多 16 个上下文，但验证可能在竞争条件下绕过。

**代码证据**：
```c
// tee_client_inner.h
#define MAX_CXTCNT_ONECA 16

// tee_client_api.c
static TEEC_Result ContextCountLimitCheck(const TEEC_Context *context)
{
    struct ListNode *node = NULL;
    uint32_t cnt = 0;
    if (LIST_EMPTY(&context_list)) {
        return TEEC_SUCCESS;
    }
    LIST_FOR_EACH(node, &context_list) {
        if (cnt++ >= MAX_CXTCNT_ONECA) {
            return TEEC_ERROR_BUSY;
        }
    }
    return TEEC_SUCCESS;
}
```

**潜在影响**：
- 资源耗尽
- 拒绝服务

---

#### 风险 5：操作参数类型检查不完整

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **风险等级** | 中 |
| **影响范围** | libteec, libteec_vendor |
| **证据位置** | `frameworks/libteec_vendor/tee_client_api.c` |

**问题描述**：
`TEEC_CheckOperation` 函数可能无法检测所有无效参数组合。

**代码证据**：
```c
// 参数类型宏定义
#define TEEC_PARAM_TYPES(param0Type, param1Type, param2Type, param3Type) \
    ((param3Type) << 12 | (param2Type) << 8 | (param1Type) << 4 | (param0Type))
```

**潜在影响**：
- 解析错误的参数
- 内存访问越界

---

#### 风险 6：ION 文件描述符处理

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-006 |
| **风险等级** | 中 |
| **影响范围** | cadaemon |
| **证据位置** | `services/cadaemon/src/ca_daemon/cadaemon_stub.cpp:200-223` |

**问题描述**：
ION fd 在传输过程中需要正确关闭，避免资源泄漏。

**代码证据**：
```c
static void CloseDupIonFd(TEEC_Operation *operation)
{
    uint32_t paramType[TEEC_PARAM_NUM] = { 0 };
    for (uint32_t paramCnt = 0; paramCnt < TEEC_PARAM_NUM; paramCnt++) {
        paramType[paramCnt] = TEEC_PARAM_TYPE_GET(operation->paramTypes, paramCnt);
        if (paramType[paramCnt] != TEEC_ION_INPUT) {
            continue;
        }
        if (operation->params[paramCnt].ionref.ion_share_fd >= 0) {
            close(operation->params[paramCnt].ionref.ion_share_fd);
        }
    }
}
```

**潜在影响**：
- FD 泄漏
- 资源耗尽

---

### 4.3 低风险项

#### 风险 7：日志信息泄露

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-007 |
| **风险等级** | 低 |
| **影响范围** | 所有模块 |
| **证据位置** | `frameworks/include/tee_log.h` |

**问题描述**：
调试日志可能泄露敏感信息。

**代码证据**：
```c
// tee_log.h
#define tlogi(fmt, ...)  // Info 级别日志
#define tlogd(fmt, ...)  // Debug 级别日志
#define tloge(fmt, ...)  // Error 级别日志
```

**修复建议**：
生产环境确保 `CONFIG_LOG_REPORT` 宏启用，限制日志级别。

---

#### 风险 8：TUI 字体哈希验证

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-008 |
| **风险等级** | 低 |
| **影响范围** | cadaemon (TUI) |
| **证据位置** | `services/cadaemon/build/standard/BUILD.gn:96-103` |

**问题描述**：
TUI 字体文件使用 MD5 哈希验证，但 MD5 已被认为不安全。

**代码证据**：
```gn
hash_string = "8978e05044e7089ad6a9de38c505c8148305607983487435a916d2610700a7ca"
```

**修复建议**：
考虑升级到 SHA-256 或更强的哈希算法。

---

## 5. 安全机制评估

### 5.1 已有的安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| 接口 Token 校验 | cadaemon_stub.cpp:561-565 | ✅ 有效 |
| 指针清零 | cadaemon_service.cpp:587-628 | ✅ 有效 |
| FD SCM_RIGHTS 传递 | tee_ca_daemon.c:60-112 | ✅ 有效 |
| 进程隔离 | IPC 框架 | ✅ 有效 |
| CFI sanitize | cadaemon BUILD.gn | ✅ 有效 |
| FDSAN 检测 | ENABLE_FDSAN_CHECK | ✅ 有效 |

### 5.2 缺失的安全机制

| 机制 | 建议实现位置 | 优先级 |
|------|--------------|--------|
| TA 文件路径白名单 | tee_client_app_load.c | 高 |
| 整数运算安全库 | 所有模块 | 高 |
| 连接超时控制 | tee_client_socket.c | 中 |
| 参数类型严格检查 | tee_client_api.c | 中 |
| ION FD 双重释放保护 | cadaemon_stub.c | 低 |

---

## 6. 安全加固建议

### 6.1 输入验证加固

```c
// 建议的输入验证函数模板
static bool ValidateInput(const void *input, size_t size, size_t maxSize) {
    // 1. NULL 检查
    if (input == NULL) {
        return false;
    }
    // 2. 大小检查
    if (size > maxSize) {
        return false;
    }
    // 3. 对齐检查
    if (size % sizeof(uint32_t) != 0) {
        return false;
    }
    return true;
}
```

### 6.2 内存安全加固

```c
// 使用安全的内存操作
#define MEMCPY_S(dest, destSize, src, srcSize) \
    memcpy_s(dest, destSize, src, srcSize)

#define MEMSET_S(dest, destSize, value, count) \
    memset_s(dest, destSize, value, count)
```

### 6.3 线程安全加固

```c
// 建议的锁使用模式
static pthread_mutex_t g_globalLock = PTHREAD_MUTEX_INITIALIZER;

TEEC_Result SafeOperation(TEEC_Context *context) {
    pthread_mutex_lock(&g_globalLock);
    
    TEEC_Result ret = DoOperation(context);
    
    pthread_mutex_unlock(&g_globalLock);
    return ret;
}
```

---

## 7. 安全相关配置

### 7.1 编译时安全配置

| 宏定义 | 位置 | 作用 |
|--------|------|------|
| `ENABLE_FDSAN_CHECK` | cadaemon/teecd | 启用文件描述符泄漏检测 |
| `CONFIG_LOG_REPORT` | libteec_vendor | 启用 Hisysevent 日志上报 |
| `CONFIG_FSWORK_THREAD_ELEVATE_PRIO` | teecd | 提升 FS Agent 线程优先级 |

### 7.2 运行时安全配置

| 配置项 | 文件 | 推荐值 |
|--------|------|--------|
| 字体哈希验证 | BUILD.gn | SHA-256 |
| 连接超时 | socket.c | ≤ 5秒 |
| 重试次数 | socket.c | ≤ 10次 |

---

## 8. 相关安全标准

- **GlobalPlatform TEE Client API Specification v1.0**
- **OWASP TEE 安全指南**
- **OpenHarmony 安全编码规范**

---

## 9. 结论

TEE Client 组件整体安全性良好，实现了以下关键安全机制：

✅ 进程隔离和权限检查  
✅ 文件描述符安全传递  
✅ 指针清零防止信息泄露  
✅ CFI 和 FDSAN 防护  

建议优先级修复以下风险：

| 优先级 | 风险 ID | 风险项 |
|--------|---------|--------|
| P0 | SEC-001 | TA 文件路径验证 |
| P1 | SEC-002 | 共享内存整数溢出 |
| P2 | SEC-003 | Socket 连接超时 |
| P3 | SEC-004 | 上下文数量限制 |
