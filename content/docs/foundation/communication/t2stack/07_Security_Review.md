# T2Stack 安全风险评审

## 目录

- [1. 攻击面清单](#1-攻击面清单)
- [2. 可利用风险点](#2-可利用风险点)
- [3. 修复建议](#3-修复建议)
- [4. 安全特性](#4-安全特性)
- [5. 审计范围与局限性](#5-审计范围与局限性)

---

## 1. 攻击面清单

### 1.1 网络接口

| 接口 | 说明 | 风险等级 |
|------|------|----------|
| **UDP Socket (设备发现)** | NStackX 使用的 CoAP 广播/单播端口 | 🔴 高 |
| **UDP Socket (Fillp)** | Fillp 流传输的 UDP 传输层 | 🔴 高 |
| **TCP Socket (DFile)** | DFile 使用的 TCP 会话 | 🟡 中 |

### 1.2 数据入口点

| 入口 | 说明 | 风险等级 |
|------|------|----------|
| **NSTACKX_DFileSendFiles()** | 发送文件列表 | 🟡 中 |
| **NSTACKX_DFileSetStoragePath()** | 设置存储路径 | 🔴 高 |
| **NSTACKX_GetDeviceList()** | 获取远程设备列表 | 🟡 中 |
| **FtRecv()** | 接收流数据 | 🟡 中 |

### 1.3 配置接口

| 接口 | 说明 | 风险等级 |
|------|------|----------|
| **FtSetSockOpt()** | 设置 socket 选项 | 🟡 中 |
| **FtConfigSet()** | Fillp 配置接口 | 🟢 低 |
| **NSTACKX_DFileSetCapabilities()** | 设置 DFile 能力 | 🟢 低 |

---

## 2. 可利用风险点

### 2.1 路径遍历风险 ⚠️

**风险 ID**: SEC-001

**证据**：
- `nstackx_dfile.h:419` - `NSTACKX_DFileSetStoragePath()` 接收路径参数
- `nstackx_dfile.h:425` - `NSTACKX_DFileSetRenameHook()` 处理文件名

**触发条件**：
```c
// 攻击者控制远程设备发送文件
NSTACKX_DFileSendFiles(sessionId, files, 1, userData);
// 其中 files[0] = "../../../etc/passwd"
```

**影响**：
- 攻击者可以将文件写入任意位置
- 可能覆盖系统文件
- 可能写入恶意脚本

**修复建议**：
```c
// 1. 规范化路径
char normalized[PATH_MAX];
if (realpath(path, normalized) == NULL) {
    return NSTACKX_EPATH_INVALID;
}

// 2. 检查路径前缀
if (strncmp(normalized, STORAGE_ROOT, strlen(STORAGE_ROOT)) != 0) {
    return NSTACKX_EPATH_OUT_OF_SCOPE;
}

// 3. 检查路径遍历
if (strstr(path, "..") != NULL) {
    return NSTACKX_EPATH_TRAVERSAL;
}
```

**优先级**：P0 - 紧急

---

### 2.2 缓冲区溢出风险 ⚠️

**风险 ID**: SEC-002

**证据**：
- `nstackx_dfile.h:42-44` - 常量定义
  - `NSTACKX_MAX_FILE_NAME_LEN = 256`
  - `NSTACKX_MAX_PATH_LEN = 256`
  - `NSTACKX_MAX_REMOTE_PATH_LEN = 1024`

- `nstackx_dfile.h:381-382` - `NSTACKX_DFileSendFiles()` 文件名参数
  - 传入文件名长度超过 256 未验证

**触发条件**：
```c
// 发送超长文件名
char longFileName[300];
memset(longFileName, 'A', 299);
longFileName[299] = '\0';

const char *files[] = { longFileName };
NSTACKX_DFileSendFiles(sessionId, files, 1, NULL);
// 可能触发栈溢出或堆溢出
```

**影响**：
- 代码执行劫持
- 拒绝服务
- 信息泄露

**修复建议**：
```c
// 在入口处验证长度
int32_t NSTACKX_DFileSendFiles(int32_t sessionId, const char *files[], 
                                 uint32_t fileNum, const char *userData) {
    for (uint32_t i = 0; i < fileNum; i++) {
        size_t len = strlen(files[i]);
        if (len >= NSTACKX_MAX_FILE_NAME_LEN || len == 0) {
            return NSTACKX_EPARAM_LEN_INVALID;
        }
        // 检查路径遍历
        if (strstr(files[i], "..") != NULL) {
            return NSTACKX_EPATH_TRAVERSAL;
        }
    }
    // ... 后续处理
}
```

**优先级**：P0 - 紧急

---

### 2.3 整数溢出风险 ⚠️

**风险 ID**: SEC-003

**证据**：
- `nstackx_dfile.h:39-40` - LiteOS 限制
  - `NSTACKX_DFILE_MAX_FILE_NUM = 10` (LiteOS)
  - `NSTACKX_MAX_FILE_LIST_NUM = 10` (LiteOS)

- `nstackx_dfile.h:49` - 标准系统限制
  - `NSTACKX_MAX_FILE_LIST_NUM = 500`

**触发条件**：
```c
// 整数溢出计算总内存
uint32_t totalSize = NSTACKX_MAX_FILE_LIST_NUM * MAX_FILE_SIZE;
// 如果 NSTACKX_MAX_FILE_LIST_NUM 被操控或计算溢出
```

**影响**：
- 内存分配过小
- 缓冲区溢出
- 拒绝服务

**修复建议**：
```c
// 使用安全乘法或预检查
uint64_t totalSize = (uint64_t)NSTACKX_MAX_FILE_LIST_NUM * MAX_FILE_SIZE;
if (totalSize > MAX_ALLOWED_SIZE) {
    return NSTACKX_EOVERFLOW;
}
char *buffer = malloc(totalSize);
```

**优先级**：P1 - 高

---

### 2.4 资源耗尽风险 ⚠️

**风险 ID**: SEC-004

**证据**：
- `nstackx_dfile.h:56-58` - VTRANS 缓冲区配置
  - `NSTACKX_VTRANS_DEFAULT_SIZE = 10MB`
  - `NSTACKX_VTRANS_STEP_SIZE = 5MB`
  - `NSTACKX_VTRANS_MAX_SIZE = 1GB`

**触发条件**：
```c
// 恶意客户端请求分配最大缓冲区
// 多次调用触发资源耗尽
```

**影响**：
- 内存耗尽
- 拒绝服务

**修复建议**：
```c
// 1. 全局内存限制
#define GLOBAL_MAX_BUFFER (100 * 1024 * 1024)  // 100MB

// 2. 会话级限制
#define SESSION_MAX_BUFFER (10 * 1024 * 1024)   // 10MB

// 3. 检查总内存使用
uint64_t currentTotal = GetGlobalBufferUsage();
if (currentTotal + requested > GLOBAL_MAX_BUFFER) {
    return NSTACKX_ERESOURCE_LIMIT;
}
```

**优先级**：P1 - 高

---

### 2.5 加密密钥处理风险 ⚠️

**风险 ID**: SEC-005

**证据**：
- `nstackx_dfile.h:308-313` - 密钥参数
  ```c
  NSTACKX_EXPORT int32_t NSTACKX_DFileServer(struct sockaddr_in *localAddr, 
      socklen_t addrLen, const uint8_t *key, uint32_t keyLen, 
      DFileMsgReceiver msgReceiver);
  ```
- 密钥以明文指针传递

**触发条件**：
```c
// 1. 栈上传递密钥（可能被窃取）
uint8_t key[16] = {0};
NSTACKX_DFileClient(..., key, 16, ...);

// 2. 密钥硬编码
const uint8_t key[] = "0123456789abcdef";
```

**影响**：
- 密钥泄露
- 中间人攻击
- 数据窃取

**修复建议**：
```c
// 1. 使用后清零密钥
uint8_t key[16] = {0};
int32_t ret = NSTACKX_DFileClient(..., key, 16, ...);
memset_s(key, sizeof(key), 0, sizeof(key));  // 使用安全内存操作

// 2. 使用密钥派生（若支持）
NSTACKX_DeriveKey(password, passwordLen, derivedKey, sizeof(derivedKey));

// 3. 使用安全存储（如 Keymaster）
NSTACKX_KeyFromKeymaster(KEY_ID, key, sizeof(key));
```

**优先级**：P1 - 高

---

### 2.6 回调函数验证缺失 ⚠️

**风险 ID**: SEC-006

**证据**：
- `nstackx_dfile.h:211` - 回调类型定义
  ```c
  typedef void (*DFileMsgReceiver)(int32_t sessionId, DFileMsgType msgType, 
      const DFileMsg *msg);
  ```

**触发条件**：
```c
// 传入空回调指针
NSTACKX_DFileServer(..., NULL, ...);
// 或恶意实现回调
void MaliciousCallback(...) {
    // 执行任意代码
}
```

**影响**：
- 空指针解引用崩溃
- 回调劫持

**修复建议**：
```c
// 验证回调非空
int32_t NSTACKX_DFileServer(..., DFileMsgReceiver msgReceiver, ...) {
    if (msgReceiver == NULL) {
        return NSTACKX_EPARAM_NULL;
    }
    // ...
}

// 注册回调白名单（若支持）
static DFileMsgReceiver g_approvedCallbacks[] = {
    &ApprovedCallback1,
    &ApprovedCallback2,
    NULL
};
```

**优先级**：P2 - 中

---

## 3. 修复建议

### 3.1 优先级矩阵

| 优先级 | 风险 ID | 问题 | 建议修复时间 |
|--------|----------|------|--------------|
| **P0** | SEC-001 | 路径遍历 | 立即修复 |
| **P0** | SEC-002 | 缓冲区溢出 | 立即修复 |
| **P1** | SEC-003 | 整数溢出 | 1 个月内 |
| **P1** | SEC-004 | 资源耗尽 | 1 个月内 |
| **P1** | SEC-005 | 密钥处理 | 1 个月内 |
| **P2** | SEC-006 | 回调验证 | 3 个月内 |

### 3.2 通用安全编码规范

```c
// 1. 参数验证宏
#define VALIDATE_PTR(ptr) do { \
    if ((ptr) == NULL) { \
        return NSTACKX_EPARAM_NULL; \
    } \
} while (0)

#define VALIDATE_LEN(len, max) do { \
    if ((len) >= (max) || (len) == 0) { \
        return NSTACKX_EPARAM_LEN_INVALID; \
    } \
} while (0)

// 2. 安全的字符串操作
#include "securec.h"
// 使用 memcpy_s, strcpy_s, strlen_s 等

// 3. 内存清零
memset_s(key, sizeof(key), 0, sizeof(key));
```

---

## 4. 安全特性

### 4.1 已启用的安全机制

| 机制 | 配置 | 效果 |
|------|------|------|
| **CFI** | `cfi = true`, `cfi_cross_dso = true` | 控制流完整性保护 |
| **PAC/Ret** | `branch_protector_ret = "pac_ret"` | 指针认证与返回地址保护 |
| **RELRO** | `-Wl,-z,relro,-z,now` | 只读重定位 |
| **Bounds Sanitize** | `boundary_sanitize = true` | 边界检查 |
| **Integer Overflow** | `integer_overflow = true` | 整数溢出检测 |
| **UBSan** | `ubsan = true` | 未定义行为检测 |

### 4.2 建议增加的安全机制

| 机制 | 说明 | 实现难度 |
|------|------|----------|
| **ASan** | 地址 sanitizer（调试时） | 低 |
| **MSan** | 内存 sanitizer（调试时） | 低 |
| **TSan** | 线程 sanitizer（调试时） | 低 |
| **Fuzzing** | 持续模糊测试 | 中 |
| **Security Audit** | 定期安全审计 | 中 |

---

## 5. 审计范围与局限性

### 5.1 已审计范围

| 组件 | 审计状态 | 备注 |
|------|----------|------|
| **Fillp API** | ✅ 已审计 | 主要接口已检查 |
| **DFile API** | ✅ 已审计 | 主要接口已检查 |
| **NStackX API** | ✅ 已审计 | 主要接口已检查 |
| **BUILD.gn** | ✅ 已审计 | 构建配置已检查 |
| **头文件** | ✅ 已审计 | 公共接口已检查 |

### 5.2 未审计范围

| 组件 | 原因 |
|------|------|
| **内部实现 (core/*.c)** | 需要逐行审计，工作量大 |
| **平台适配层 (platform/*)** | 平台相关代码未全面审计 |
| **第三方依赖** | libcoap, mbedtls, openssl 由各自团队审计 |
| **测试代码** | 测试代码不进入生产环境 |

### 5.3 审计方法

| 方法 | 应用 |
|------|------|
| **静态代码分析** | Cppcheck, Clang Static Analyzer |
| **动态测试** | AFL++ Fuzzing（建议） |
| **代码审查** | 人工审查关键路径 |
| **渗透测试** | 建议定期进行 |

---

## 相关文档

- [构建系统](./05_Build_System.md) - 安全编译配置
- [API 参考](./03_CAPI_Reference.md) - 安全使用 API
- [故障排查](./08_Troubleshooting.md) - 安全相关问题定位

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
*审计方法：静态代码分析 + 代码审查*
