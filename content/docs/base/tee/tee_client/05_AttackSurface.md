# TEE Client 攻击面分析

## 1. 分析概述

### 1.1 分析范围

| 模块 | 评审状态 | 说明 |
|------|----------|------|
| libteec.so | ✅ 已分析 | 系统组件 TEE API 库 |
| libteec_vendor.so | ✅ 已分析 | 芯片组件 TEE API 库 |
| cadaemon | ✅ 已分析 | CA 守护进程 (SA 8001) |
| teecd | ✅ 已分析 | TEE 代理服务 |
| tlogcat | ✅ 已分析 | TEE 日志服务 |

### 1.2 信任边界

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

---

## 2. 外部输入清单

### 2.1 IPC 接口输入

**入口位置**: `services/cadaemon/src/ca_daemon/cadaemon_stub.cpp:30`

```cpp
int CaDaemonStub::OnRemoteRequest(uint32_t code, 
    MessageParcel &data, MessageParcel &reply, MessageOption &option)
```

| 操作码 | 输入数据 | 数据来源 | 风险等级 |
|--------|----------|----------|----------|
| INIT_CONTEXT (0) | `name` (字符串) | CA 应用 | 低 |
| FINAL_CONTEXT (1) | `TEEC_Context` (结构体) | CA 应用 | 中 |
| OPEN_SESSION (2) | `taPath`, `UUID`, `TEEC_Operation` | CA 应用 | **高** |
| CLOSE_SESSION (3) | `TEEC_Session`, `TEEC_Context` | CA 应用 | 中 |
| INVOKE_COMMND (4) | `commandID`, `TEEC_Operation` | CA 应用 | **高** |
| REGISTER_MEM (5) | `TEEC_SharedMemory` | CA 应用 | **高** |
| ALLOC_MEM (6) | `TEEC_SharedMemory` (含 size) | CA 应用 | **高** |
| RELEASE_MEM (7) | `TEEC_SharedMemory` | CA 应用 | 中 |
| SET_CALL_BACK (8) | `IRemoteObject` | CA 应用 | 低 |
| SEND_SECFILE (9) | `path`, `fd`, `fp` | CA 应用 | **高** |
| GET_TEE_VERSION (10) | 无 | - | 无 |

### 2.2 Socket 接口输入

**入口位置**: `frameworks/libteec_vendor/tee_client_socket.c`

| 函数 | 输入 | 数据来源 | 风险等级 |
|------|------|----------|----------|
| `ConnectToCaDaemon()` | 无 | - | 低 |
| `SendCaMsg()` | `CaRevMsg` (含 `cmd`, `caAuthInfo`) | CA 应用 | **高** |

**CaRevMsg 结构**:
```c
struct CaRevMsg {
    int cmd;                    // 操作命令
    CaAuthInfo caAuthInfo;      // CA 认证信息 (pid, uid, type, certs)
};
```

### 2.3 文件系统输入

| 输入类型 | 位置 | 说明 | 风险等级 |
|----------|------|------|----------|
| **TA 文件路径** | `interfaces/inner_api/tee_client_api.h:53` | `context.ta_path` 用户可控 | **高** |
| **TA 文件内容** | `frameworks/libteec_vendor/tee_client_app_load.c` | 加载 .sec 文件 | **高** |
| **设备节点** | `/dev/tee*`, `/dev/tc_ns` | 与驱动通信 | **高** |
| **配置文件** | `/system/etc/init/*.cfg` | 服务启动配置 | 中 |
| **日志文件** | `/data/log/tee/` | TEE 日志输出 | 低 |

**TA 路径输入示例**:
```c
// 位置: interfaces/inner_api/tee_client_api.h:53-61
// 用户可设置 ta_path 指定 TA 文件位置
TEEC_Context context;
context.ta_path = (uint8_t *)"/data/58dbb3b9-4a0c-42d2-a84d-7c7ab17539fc.sec";
```

### 2.4 共享内存参数输入

**入口位置**: `services/cadaemon/src/ca_daemon/cadaemon_service.cpp:827`

| 参数 | 类型 | 说明 | 风险等级 |
|------|------|------|----------|
| `operation.paramTypes` | uint32_t | 参数类型组合 | **高** |
| `operation.params[]` | TEEC_Parameter[4] | 参数数组 | **高** |
| `shm.size` | uint32_t | 共享内存大小 | **高** |
| `shm.buffer` | void* | 缓冲区指针 | **高** |

### 2.5 环境变量/系统属性输入

| 输入 | 位置 | 说明 | 风险等级 |
|------|------|------|----------|
| `调用者 PID` | `IPCSkeleton::GetCallingPid()` | 用于身份验证 | 中 |
| `调用者 UID` | `IPCSkeleton::GetCallingUid()` | 用于身份验证 | 中 |
| `调用者 TokenID` | `IPCSkeleton::GetCallingTokenID()` | 用于身份验证 | 中 |
| `进程名称` | `/proc/<pid>/comm` | 用于 CA 识别 | 低 |

---

## 3. 敏感操作清单

### 3.1 系统调用操作

| 操作 | 位置 | 说明 | 风险等级 |
|------|------|------|----------|
| `ioctl(TC_NS_CLIENT_IOCTL...)` | `tee_client_api.c` | 与 TEE 驱动通信 | **高** |
| `open("/dev/tee*")` | `tee_client_socket.c:67` | 打开 TEE 设备 | **高** |
| `mmap()` | `tee_client_api.c:1587` | 共享内存映射 | **高** |
| `tgkill()` | `cadaemon_service.cpp:287` | 发送信号给线程 | 中 |

### 3.2 特权接口调用

| 接口 | 位置 | 说明 | 风险等级 |
|------|------|------|----------|
| `SystemAbility::Publish()` | `cadaemon_service.cpp:147` | 发布系统服务 | **高** |
| `AddDeathRecipient()` | `cadaemon_service.cpp:228` | 注册死亡通知 | 中 |
| `dlopen()` | `cadaemon_service.cpp:129` | 加载动态库 | 中 |
| `fopen()` | `tee_client_app_load.c` | 打开 TA 文件 | **高** |

### 3.3 文件写入操作

| 操作 | 位置 | 说明 | 风险等级 |
|------|------|------|----------|
| `fwrite()` (日志) | `tee_sys_log.h` | 写入日志文件 | 低 |
| `ioctl(SEND_SECFILE)` | `tee_client_api.c` | 发送安全文件到 TEE | **高** |

### 3.4 内存操作

| 操作 | 位置 | 说明 | 风险等级 |
|------|------|------|----------|
| `malloc(size)` | 多处 | 动态内存分配 | **高** |
| `memcpy()` | 多处 | 内存拷贝 | **高** |
| `memset_s()` | 多处 | 安全内存清零 | 中 |
| `mmap()` | `tee_client_api.c` | 共享内存映射 | **高** |
| `munmap()` | `tee_client_api.c` | 解除内存映射 | 中 |

---

## 4. 攻击向量分析

### 4.1 外部攻击者

**能力假设**:
- 能够运行普通 CA 应用
- 无法直接访问 TEE 内部
- 无法篡改内核代码

**可行攻击**:

| 攻击类型 | 入口点 | 可行性 | 说明 |
|----------|--------|--------|------|
| 恶意参数注入 | IPC/Socket | 中 | 构造畸形参数 |
| 路径遍历 | ta_path | 高 | 利用 `../` 绕过 |
| 整数溢出 | size 参数 | 中 | 超大内存请求 |
| 拒绝服务 | 资源耗尽 | 高 | 大量会话/内存 |

### 4.2 恶意 CA

**能力假设**:
- 合法注册的应用
- 可能传递恶意参数
- 可能尝试越权访问

**可行攻击**:

| 攻击类型 | 入口点 | 可行性 | 说明 |
|----------|--------|--------|------|
| 路径遍历 | ta_path | **高** | 未严格验证路径 |
| 权限提升 | tokenid 伪造 | 低 | 依赖系统框架校验 |
| 内存破坏 | 共享内存 | 中 | 参数处理问题 |
| UAF | session 操作 | 中 | 生命周期管理 |

### 4.3 攻击路径图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           攻击路径图                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  外部攻击者                                                                   │
│      │                                                                       │
│      ├──► IPC 接口 ──► MessageParcel 解析 ──► 参数注入                        │
│      │                    │                                                  │
│      │                    ├──► ta_path 验证绕过 ──► 路径遍历                    │
│      │                    ├──► size 溢出 ──► 内存分配失败                     │
│      │                    └──► paramTypes 伪造 ──► 内存越界                   │
│      │                                                                       │
│      ├──► Unix Socket ──► cmd/caAuthInfo 注入                               │
│      │                                                                       │
│      └──► 设备节点 ──► ioctl 命令注入                                        │
│                                                                              │
│  恶意 CA                                                                      │
│      │                                                                       │
│      ├──► 上下文耗尽 ──► 拒绝服务                                            │
│      ├──► 会话耗尽 ──► 拒绝服务                                              │
│      └──► 共享内存耗尽 ──► 拒绝服务                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. 安全控制点

### 5.1 已实现的防护机制

| 机制 | 位置 | 说明 | 有效性 |
|------|------|------|--------|
| **Token 校验** | `cadaemon_stub.cpp:44` | 校验 IPC 调用者 Token | ✅ 有效 |
| **PID/UID 验证** | `cadaemon_service.cpp:303` | 校验进程身份信息 | ✅ 有效 |
| **上下文验证** | `cadaemon_service.cpp:295` | `IsValidContext()` 检查 | ✅ 有效 |
| **死亡通知** | `cadaemon_service.cpp:228` | CA 崩溃时清理资源 | ✅ 有效 |
| **整数溢出检查** | `cadaemon_service.cpp:643` | `CheckSizeStatus()` | ⚠️ 需改进 |
| **指针清零** | `cadaemon_service.cpp:589` | 防止信息泄露 | ✅ 有效 |
| **SCM_RIGHTS** | `tee_client_socket.c` | 安全 FD 传递 | ✅ 有效 |

### 5.2 缺失/不足的防护

| 防护点 | 状态 | 建议 |
|--------|------|------|
| ta_path 路径规范化 | ❌ 缺失 | 添加 `realpath()` 验证 |
| 会话数量限制 | ⚠️ 不足 | 增强 MAX_CXTCNT_ONECA 检查 |
| 速率限制 | ❌ 缺失 | 添加 API 调用频率限制 |
| 命令白名单 | ❌ 缺失 | 限制允许的 commandID 范围 |

---

## 6. 输入验证检查表

| 输入类型 | 验证点 | 验证函数 | 状态 |
|----------|--------|----------|------|
| TEEC_Context | NULL 检查 | 多处 | ✅ |
| TEEC_Session | NULL 检查 | 多处 | ✅ |
| UUID | 格式检查 | 间接验证 | ⚠️ |
| ta_path | 路径规范化 | ❌ 缺失 | ❌ |
| shm.size | 整数溢出 | `CheckSizeStatus()` | ⚠️ |
| paramTypes | 类型有效性 | 部分检查 | ⚠️ |
| commandID | 范围检查 | ❌ 缺失 | ❌ |
| buffer 指针 | NULL 检查 | 多处 | ✅ |

---

## 7. 附录：关键代码位置

### 7.1 输入处理入口

```cpp
// cadaemon_stub.cpp:30 - IPC 请求入口
int CaDaemonStub::OnRemoteRequest(uint32_t code, 
    MessageParcel &data, MessageParcel &reply, MessageOption &option)

// cadaemon_service.cpp:827 - OpenSession 处理
TEEC_Result CaDaemonService::OpenSession(TEEC_Context *context, 
    const char *taPath, int32_t &fd, ...)

// cadaemon_service.cpp:920 - InvokeCommand 处理  
TEEC_Result CaDaemonService::InvokeCommand(TEEC_Context *context, ...)
```

### 7.2 验证函数位置

```cpp
// cadaemon_service.cpp:295 - 上下文验证
bool CaDaemonService::IsValidContext(const TEEC_Context *context, ...)

// cadaemon_service.cpp:643 - 大小检查
static bool CheckSizeStatus(uint32_t shmInfoOffset, uint32_t refSize, ...)

// cadaemon_stub.cpp:44 - Token 验证
static bool CheckPermission(uint32_t code)
```

### 7.3 共享内存处理

```cpp
// cadaemon_service.cpp:1010 - 注册共享内存
TEEC_Result CaDaemonService::RegisterSharedMemory(...)

// cadaemon_service.cpp:1091 - 分配共享内存
TEEC_Result CaDaemonService::AllocateSharedMemory(...)

// cadaemon_service.cpp:723 - 参数解码
TEEC_Result CaDaemonService::GetTeecOptMem(...)
```

---

**文档版本**: 1.0
**更新时间**: 2026-02-07
