# 08_Security_Review - 安全风险评审

> 本文档基于代码证据对 Cast+ Stream 模块进行安全风险评审，包括攻击面分析、信任边界和可被利用点。

---

## 1. 安全概述

### 1.1 安全目标

| 目标 | 说明 | 实现方式 |
|------|------|----------|
| **认证** | 确保只有授权应用可使用投屏 | 系统权限 + PID 白名单 |
| **授权** | 区分镜像和流媒体权限 | 独立权限检查 |
| **机密性** | 保护传输数据 | AES-128 加密 |
| **完整性** | 防止数据篡改 | GCM 模式认证加密 |
| **可用性** | 防止拒绝服务 | 输入校验、资源限制 |

### 1.2 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           信任边界模型                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        非信任域 (Untrusted)                          │   │
│   │                                                                     │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│   │  │   第三方    │  │   网络      │  │   远程      │                 │   │
│   │  │   应用      │  │   传输      │  │   设备      │                 │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘                 │   │
│   │                                                                     │   │
│   └───────────────────────────┬─────────────────────────────────────────┘   │
│                               │ 边界1: IPC 接口                              │
│                               ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      半信任域 (Semi-Trusted)                         │   │
│   │                                                                     │   │
│   │  ┌─────────────────────────────────────────────────────────────┐   │   │
│   │  │              Cast+ Stream 模块 (本模块)                      │   │   │
│   │  │                                                             │   │   │
│   │  │  - 权限检查 (Permission)                                    │   │   │
│   │  │  - 输入校验 (Input Validation)                              │   │   │
│   │  │  - 会话管理 (Session)                                       │   │   │
│   │  │  - 协议处理 (RTSP)                                          │   │   │
│   │  └─────────────────────────────────────────────────────────────┘   │   │
│   │                                                                     │   │
│   └───────────────────────────┬─────────────────────────────────────────┘   │
│                               │ 边界2: 系统调用                              │
│                               ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        信任域 (Trusted)                              │   │
│   │                                                                     │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│   │  │   系统      │  │   内核      │  │   硬件      │                 │   │
│   │  │   服务      │  │   驱动      │  │   加速      │                 │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘                 │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 攻击面 | 入口点 | 风险等级 | 说明 |
|--------|--------|----------|------|
| **IPC 接口** | `CastSessionImplStub::OnRemoteRequest()` | 高 | 所有外部调用入口 |
| **网络输入** | `SoftBusConnection::OnBytesReceived()` | 高 | 网络数据接收 |
| **网络输入** | `TcpConnection::HandleReceivedData()` | 高 | TCP 数据接收 |
| **文件输入** | `CastLocalFileChannelServer::ProcessRequestData()` | 中 | HTTP 文件请求 |
| **RTSP 协议** | `RtspController::OnRequest()` | 高 | RTSP 请求处理 |
| **RTSP 协议** | `RtspController::OnResponse()` | 高 | RTSP 响应处理 |

### 2.2 攻击面详细分析

#### IPC 接口攻击面

```cpp
// 入口点: src/cast_session_impl_stub.cpp:30-35
int CastSessionImplStub::OnRemoteRequest(uint32_t code, MessageParcel &data, 
                                         MessageParcel &reply,
                                         MessageOption &option) {
    RETRUEN_IF_WRONG_TASK(code, data, reply, option);
    return EXECUTE_SINGLE_STUB_TASK(code, data, reply);
}
```

**风险**: 所有 IPC 调用均通过此入口，需确保权限检查和参数校验。

**防护措施**:
- 权限检查：`Permission::CheckMirrorPermission()` / `Permission::CheckStreamPermission()`
- PID 白名单：`Permission::CheckPidPermission()`
- 参数校验：每个 Task 方法内的参数校验

#### 网络输入攻击面

```cpp
// 入口点: src/channel/src/tcp/tcp_connection.cpp:186-236
void TcpConnection::HandleReceivedData() {
    // 读取 4 字节长度头
    int32_t totalLen = 0;
    int ret = recv(clientFd_, &totalLen, sizeof(int32_t), MSG_WAITALL);
    // ... 长度校验
    if (totalLen <= 0 || totalLen > MAX_PACKET_SIZE) {
        // 错误处理
    }
}
```

**风险**: 恶意构造的数据包可能导致缓冲区溢出或拒绝服务。

**防护措施**:
- 长度校验：`totalLen > 0 && totalLen <= MAX_PACKET_SIZE`
- 缓冲区限制：接收缓冲区大小限制

#### RTSP 协议攻击面

```cpp
// 入口点: src/rtsp/src/rtsp_controller.cpp:150-195
bool RtspController::OnRequest(std::shared_ptr<RtspRequest> request) {
    // 方法查找
    auto it = requestFuncMap_.find(request->method);
    if (it == requestFuncMap_.end()) {
        SendErrorResponse("405", "Method Not Allowed");
        return false;
    }
    // 调用处理函数
    return (this->*it->second)(request);
}
```

**风险**: RTSP 消息解析可能存在漏洞，参数可能被恶意构造。

**防护措施**:
- 方法白名单：仅支持预定义的 RTSP 方法
- 参数解析安全：`RtspParse::ParseIntSafe()` 等安全解析函数
- 长度限制：字符串长度检查

---

## 3. 可被利用点分析

### 3.1 高风险点

#### 风险点 1: IPC 权限检查绕过

**证据**:
```cpp
// src/utils/src/permission.cpp:36-53
namespace {
std::string GetPermissionDescription(const std::string &permission)
{
    if (permission == "ohos.permission.ACCESS_CAST_ENGINE_MIRROR") {
        return "Mirror permission";
    }
    if (permission == "ohos.permission.ACCESS_CAST_ENGINE_STREAM") {
        return "Stream permission";
    }
    return "Unkown permission";
}

bool CheckPermission(const std::string &permission)
{
    CLOGE("%{public}s succ", GetPermissionDescription(permission).c_str());
    return true;  // ⚠️ 当前实现直接返回 true，未进行真实权限检查
}
} // namespace
```

**位置**: `src/utils/src/permission.cpp:48-52`

**调用点**: 
- `src/cast_session_impl_stub.cpp:82-84` - DoAddDeviceTask
- `src/cast_session_impl_stub.cpp:101-103` - DoRemoveDeviceTask
- `src/mirror/src/mirror_player_impl_stub.cpp:50` - 所有Mirror接口
- `src/stream/src/player/src/stream_player_impl_stub.cpp:35` - 所有Stream接口

**触发条件**: 任何 IPC 调用

**影响**: 权限检查被绕过，任何应用均可调用投屏接口

**修复建议**:
```cpp
// 应使用 AccessTokenKit 进行真实权限检查
bool CheckPermission(const std::string &permission) {
    AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();
    int result = AccessTokenKit::VerifyAccessToken(tokenId, permission);
    return result == PERMISSION_GRANTED;
}
```

#### 风险点 2: PID 白名单检查不完整

**证据**:
```cpp
// src/utils/src/permission.cpp:96-112
bool Permission::CheckPidPermission()
{
    std::lock_guard<std::mutex> lock(pidLock_);
    pid_t pid = IPCSkeleton::GetCallingPid();
    pid_t myPid = getpid();
    CLOGD("Calling pid is %{public}d, my pid is %{public}d", pid, myPid);
    if (pid == myPid || pid == 0) { // 0 means role is proxy
        return true;  // ⚠️ pid == 0 通过检查
    }

    auto it = std::find_if(pids_.begin(), pids_.end(), [pid](pid_t element) { return element == pid; });
    if (it == pids_.end()) {
        CLOGE("pid(%{public}d) is illegal", pid);
        return false;
    }
    return true;
}
```

**位置**: `src/utils/src/permission.cpp:96-112`

**分析**: 
- `pid == 0` 被解释为 "role is proxy" 并直接通过检查
- 注释说明这是为了支持 proxy 角色，但可能带来安全风险
- 如果攻击者能以 proxy 身份调用，可绕过 PID 白名单

**触发条件**: 通过 Proxy 调用（pid == 0）

**影响**: Proxy 调用绕过 PID 检查

**修复建议**: 
1. 明确 Proxy 场景的安全策略
2. 添加额外的认证机制用于 Proxy 场景
3. 或移除 pid == 0 的例外，要求所有调用都必须在白名单中

#### 风险点 3: RTSP 参数解析潜在整数溢出

**证据**:
```cpp
// src/rtsp/src/rtsp_parse.cpp:95-111
int RtspParse::ParseIntSafe(const std::string &str)
{
    if (str.size() == 0) {
        return INVALID_VALUE;
    }

    char *nextPtr = nullptr;
    long result = strtol(str.c_str(), &nextPtr, DECIMALISM);
    if (errno == ERANGE) {
        CLOGE("Parse int out of range");
        return INVALID_VALUE;
    } else if (*nextPtr != '\0') {
        CLOGE("Parse int error, invalid parament");
        return INVALID_VALUE;
    }
    return static_cast<int>(result);  // ⚠️ 64位系统上long可能大于int
}
```

**位置**: `src/rtsp/src/rtsp_parse.cpp:95-111`

**分析**: 
- ✅ 已实现 `errno == ERANGE` 检查，可检测溢出
- ⚠️ 但在64位系统上，`long` 为64位，`int` 为32位，当值在 `INT_MAX+1` 到 `LONG_MAX` 之间时，`strtol` 不会设置 `ERANGE`，但 `static_cast<int>` 会导致截断

**触发条件**: 发送 `INT_MAX+1` 到 `LONG_MAX` 范围内的整数值

**影响**: 值截断可能导致逻辑错误

**修复建议**:
```cpp
int RtspParse::ParseIntSafe(const std::string &str) {
    if (str.empty()) return INVALID_VALUE;
    char *endPtr = nullptr;
    long result = strtol(str.c_str(), &endPtr, 10);
    if (errno == ERANGE) {
        return INVALID_VALUE;
    }
    if (*endPtr != '\0') {
        return INVALID_VALUE;
    }
    // 添加范围检查
    if (result > INT_MAX || result < INT_MIN) {
        return INVALID_VALUE;
    }
    return static_cast<int>(result);
}
```

### 3.2 中风险点

#### 风险点 4: 文件路径遍历

**证据**:
```cpp
// src/stream/src/local/src/cast_local_file_channel_server.cpp:130
CLOGE("not support local file url");
return false;
```

**位置**: `src/stream/src/local/src/cast_local_file_channel_server.cpp:130`

**分析**: 当前实现已禁用本地文件 URL 访问，仅支持 fd 方式，风险较低。

**建议**: 保持当前限制，不要开放文件路径访问。

#### 风险点 5: 缓冲区大小检查

**证据**:
```cpp
// src/channel/src/tcp/tcp_connection.cpp:195-200
if (totalLen <= 0 || totalLen > ILLEGAL_LENGTH) {
    CLOGE("Illegal packet length: %{public}d", totalLen);
    CloseConnection();
    return;
}
```

**位置**: `src/channel/src/tcp/tcp_connection.cpp:195-200`

**分析**: 有长度检查，但 `ILLEGAL_LENGTH` 值需确认是否合理。

### 3.3 低风险点

#### 风险点 6: 日志信息泄露

**证据**:
```cpp
// 多处使用 %{public}s 输出敏感信息
CLOGD("sessionKey: %{public}s", sessionKey);
```

**建议**: 审查日志输出，避免打印密钥、Token 等敏感信息。

#### 风险点 7: 定时器精度

**证据**:
```cpp
// 超时处理依赖系统定时器
```

**分析**: 定时器精度可能影响超时判断，但风险较低。

---

## 4. 安全机制评估

### 4.1 已实现的防护

| 防护机制 | 实现 | 评估 |
|----------|------|------|
| **权限检查** | `Permission` 类 | ⚠️ 当前实现为桩函数，需完善 |
| **PID 白名单** | `Permission::CheckPidPermission()` | ✅ 基本实现，有例外需处理 |
| **传输加密** | `EncryptDecrypt` AES-128 | ✅ 完整实现 |
| **输入校验** | `RtspParse::ParseIntSafe()` 等 | ⚠️ 部分函数需加强 |
| **缓冲区限制** | 长度检查 | ✅ 基本实现 |

### 4.2 缺失的防护

| 防护机制 | 建议 | 优先级 |
|----------|------|--------|
| **真实权限检查** | 接入 AccessTokenKit | 高 |
| **Rate Limiting** | 限制 IPC 调用频率 | 中 |
| **输入消毒** | 对所有外部输入进行消毒 | 中 |
| **审计日志** | 记录安全相关操作 | 低 |

---

## 5. 修复建议汇总

### 5.1 高优先级修复

1. **修复权限检查实现**
   - 文件: `src/utils/src/permission.cpp`
   - 修改: 接入真实的 AccessTokenKit 权限检查

2. **修复整数溢出风险**
   - 文件: `src/rtsp/src/rtsp_parse.cpp`
   - 修改: 添加范围检查

3. **审查 PID 白名单策略**
   - 文件: `src/utils/src/permission.cpp`
   - 修改: 明确 pid == 0 的处理策略

### 5.2 中优先级修复

4. **加强输入校验**
   - 对所有 IPC 参数进行更严格的校验
   - 对 RTSP 参数进行范围检查

5. **审查日志输出**
   - 移除敏感信息打印
   - 使用 %{private}s 替代 %{public}s

### 5.3 低优先级改进

6. **添加审计日志**
   - 记录关键安全操作
   - 支持安全事件追溯

7. **完善错误处理**
   - 统一错误码
   - 避免信息泄露

---

## 6. 检查范围与局限性

### 6.1 已检查范围

- ✅ IPC 接口权限检查
- ✅ 网络输入处理
- ✅ RTSP 协议解析
- ✅ 文件访问控制
- ✅ 加密实现
- ✅ 缓冲区管理

### 6.2 未检查范围

- ❌ 父级框架的安全实现
- ❌ 系统服务的安全机制
- ❌ 第三方库的安全漏洞
- ❌ 硬件加速的安全问题

### 6.3 局限性说明

1. 本评审仅基于静态代码分析，未进行动态测试
2. 未发现所有潜在漏洞，建议进行专业安全审计
3. 部分风险点依赖实际运行环境

---

## 7. 相关文档

- [04_External_API.md](./04_External_API.md) - 对外 API
- [05_Internal_API.md](./05_Internal_API.md) - 内部 API
- [09_Troubleshooting.md](./09_Troubleshooting.md) - 问题定位
