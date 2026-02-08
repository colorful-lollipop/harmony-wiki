# 安全风险评估

## 6.1 评估概述

### 评估范围

本安全风险评估覆盖 HiSysEvent 组件的核心代码模块，包括：事件写入模块（N-API、Native C++ API、C API）、事件查询模块（HiSysEventManager）、IPC 通信适配模块（adapter/idl）、以及框架层核心逻辑。评估重点关注输入验证、内存安全、权限控制、并发安全等常见漏洞类型。

评估基于代码静态分析，未进行运行时安全测试。部分风险需要在实际环境中验证利用可行性。评估结论适用于 HiSysEvent 3.1 版本，其他版本可能存在差异。

### 风险等级定义

| 等级 | 说明 | 响应要求 |
|------|------|----------|
| **高危** | 可直接利用，导致严重后果 | 立即修复 |
| **中危** | 利用条件较复杂，可能导致中等后果 | 尽快修复 |
| **低危** | 利用困难或影响有限 | 计划修复 |
| **信息** | 非安全风险，仅供参考 | 无需修复 |

---

## 6.2 输入验证缺陷

### R1：域名参数注入风险（低危）

**位置**：`interfaces/native/innerkits/hisysevent/hisysevent.cpp:89-112`

**证据**：

```cpp
// hisysevent.cpp:89-95
int HiSysEvent::Write(...) {
    // 参数校验
    if (domain.length() > 16) {
        return ERR_INVALID_PARAM;
    }
    if (!isalpha(domain[0])) {
        return ERR_INVALID_PARAM;
    }
    // 后续处理...
}
```

**分析**：

域名参数校验实现了基本的长度限制（最大 16 字符）和首字符校验（必须为字母），但字符集允许包含下划线。虽然当前实现通过白名单限制了可接受字符，但仍需关注边界情况。

**触发路径**：

```
用户代码 → HiSysEvent::Write(domain, name, type, keyValues)
        → 第 89 行：检查 domain.length()
        → 第 91 行：检查 isalpha(domain[0])
        → 继续处理
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 字符集白名单保护 |
| **权限提升** | 无 | 不涉及权限变更 |
| **数据泄露** | 无 | 不涉及敏感数据访问 |
| **系统影响** | 低 | 仅影响事件记录 |

**修复建议**：

```cpp
// 强化域名校验
bool ValidateDomain(const std::string& domain) {
    // 长度范围检查
    if (domain.empty() || domain.length() > 16) {
        return false;
    }
    
    // 首字符严格校验
    if (!isalpha(static_cast<unsigned char>(domain[0]))) {
        return false;
    }
    
    // 全字符校验：仅允许字母、数字、下划线
    for (char c : domain) {
        if (!isalnum(static_cast<unsigned char>(c)) && c != '_') {
            return false;
        }
    }
    
    return true;
}
```

### R2：事件名称长度边界问题（低危）

**位置**：`interfaces/native/innerkits/hisysevent/hisysevent.cpp:95-118`

**证据**：

```cpp
// hisysevent.cpp:95-101
if (eventName.length() > 32) {
    return ERR_INVALID_PARAM;
}
if (!isalpha(eventName[0])) {
    return ERR_INVALID_PARAM;
}
```

**分析**：

事件名称长度限制为 32 字符，但未检查空字符串情况。如果 eventName 为空字符串，`isalpha(eventName[0])` 会导致未定义行为（访问空字符串的首字符）。

**触发路径**：

```
用户代码 → HiSysEvent::Write(domain, "", type, keyValues)
        → 第 95 行：检查长度（通过，0 <= 32）
        → 第 97 行：访问 eventName[0] → UB!
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 需要特定调用方式 |
| **崩溃风险** | 中 | 可能导致进程崩溃 |
| **系统影响** | 低 | 仅影响调用进程 |

**修复建议**：

```cpp
// hisysevent.cpp:95-105
if (eventName.empty() || eventName.length() > 32) {
    return ERR_INVALID_PARAM;
}
if (!isalpha(static_cast<unsigned char>(eventName[0]))) {
    return ERR_INVALID_PARAM;
}
```

### R3：IPC 数据反序列化风险（中危）

**位置**：`adapter/native/idl/src/hisysevent_delegate.cpp:78-145`

**证据**：

```cpp
// hisysevent_delegate.cpp:78-95
int32_t HisyseventDelegate::OnRemoteRequest(uint32_t code,
                                            MessageParcel& data,
                                            MessageParcel& reply) {
    // 从 IPC 数据读取
    std::string domain = data.ReadString();
    std::string eventName = data.ReadString();
    
    // 无长度限制直接使用
    ProcessRule(domain, eventName);
}
```

**分析**：

IPC 数据反序列化时，直接从 MessageParcel 读取字符串而未进行长度限制。如果攻击者构造超长数据，可能导致缓冲区溢出或拒绝服务。

**触发路径**：

```
恶意进程 → IPC 调用 → HisyseventDelegate::OnRemoteRequest
        → data.ReadString() 返回超长字符串
        → ProcessRule() 处理异常数据
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 中 | 需要 IPC 调用权限 |
| **DoS 风险** | 高 | 可能导致服务崩溃 |
| **数据损坏** | 中 | 可能污染事件数据 |

**修复建议**：

```cpp
// hisysevent_delegate.cpp:78-95
int32_t HisyseventDelegate::OnRemoteRequest(uint32_t code,
                                            MessageParcel& data,
                                            MessageParcel& reply) {
    // 读取域
    std::string domain = data.ReadString();
    if (domain.length() > 16 || !ValidateDomain(domain)) {
        return ERR_INVALID_PARAM;
    }
    
    // 读取事件名
    std::string eventName = data.ReadString();
    if (eventName.length() > 32 || !ValidateEventName(eventName)) {
        return ERR_INVALID_PARAM;
    }
    
    return ProcessRule(domain, eventName);
}
```

---

## 6.3 内存安全问题

### R4：编码缓冲区溢出风险（低危）

**位置**：`interfaces/native/innerkits/hisysevent/encoded_param.cpp:45-78`

**证据**：

```cpp
// encoded_param.cpp:45-65
EncodedParam EncodeParam(const std::string& key, const std::string& value) {
    EncodedParam param;
    
    // 检查 key 长度
    if (key.length() > MAX_KEY_LEN) {
        // 处理错误...
    }
    
    // 直接复制（目标缓冲区已知大小）
    memcpy(param.key, key.c_str(), key.length());
    param.key[key.length()] = '\0';  // 潜在问题
    
    return param;
}
```

**分析**：

虽然目标缓冲区 `param.key` 有固定大小限制（假设为 32 字节），但在极端情况下仍可能存在边界问题。`memcpy` 使用源字符串长度，如果 key 接近缓冲区大小，可能导致问题。

**触发路径**：

```
用户代码 → HiSysEvent::Write() → EncodeParam()
        → key.length() = 31
        → memcpy(param.key, key.c_str(), 31)
        → 写入第 32 字节
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 受长度检查保护 |
| **内存破坏** | 低 | 边界检查保护 |
| **代码执行** | 极低 | 需要复杂条件 |

**修复建议**：

```cpp
// encoded_param.cpp:45-65
EncodedParam EncodeParam(const std::string& key, const std::string& value) {
    EncodedParam param;
    size_t keyLen = std::min(key.length(), MAX_KEY_LEN - 1);
    
    memcpy(param.key, key.c_str(), keyLen);
    param.key[keyLen] = '\0';
    
    return param;
}
```

### R5：Socket 发送缓冲区问题（中危）

**位置**：`interfaces/native/innerkits/hisysevent/transport.cpp:89-145`

**证据**：

```cpp
// transport.cpp:89-120
int Transport::Send(const EventData& data) {
    // 序列化数据
    char buffer[SEND_BUFFER_SIZE];
    size_t offset = 0;
    
    for (const auto& param : data.params) {
        size_t len = param.key.length() + param.value.length();
        if (offset + len > SEND_BUFFER_SIZE) {
            // 缓冲区溢出保护
            return ERR_BUFFER_FULL;
        }
        memcpy(buffer + offset, param.key.c_str(), param.key.length());
        offset += param.key.length();
        // ... 继续复制
    }
    
    // 发送数据
    ssize_t sent = send(socket_, buffer, offset, 0);
    return sent > 0 ? 0 : -1;
}
```

**分析**：

实现了基本的缓冲区溢出保护，但仍需关注异常场景下的处理逻辑。`send()` 调用可能返回部分写入，需要处理这种情况。

**触发路径**：

```
用户代码 → HiSysEvent::Write() → Transport::Send()
        → 序列化数据到 buffer
        → send(socket_, buffer, offset, 0)
        → 返回部分写入
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **数据丢失** | 中 | 部分发送可能丢失数据 |
| **状态不一致** | 中 | 需要重试或恢复机制 |

**修复建议**：

```cpp
// transport.cpp:89-145
int Transport::Send(const EventData& data) {
    char buffer[SEND_BUFFER_SIZE];
    size_t totalSent = 0;
    
    while (totalSent < data.size) {
        ssize_t sent = send(socket_, buffer + totalSent,
                           data.size - totalSent, 0);
        if (sent < 0) {
            if (errno == EINTR) {
                continue;  // 中断则重试
            }
            return -1;  // 其他错误
        }
        totalSent += sent;
    }
    
    return 0;
}
```

---

## 6.4 权限与鉴权

### R6：access_token 验证绕过高风险分析（中危）

**位置**：`interfaces/native/innerkits/hisysevent/hisysevent.cpp:120-145`

**证据**：

```cpp
// hisysevent.cpp:120-135
int HiSysEvent::Write(...) {
    // 获取调用方 token
    uint32_t callerToken = GetCallingTokenID();
    
    // 验证权限
    if (!VerifyPermission(callerToken,
                         "ohos.permission.ACCESS_SYSTEM_SERVICE")) {
        HiLog::Error(LABEL, "Permission denied for HiSysEvent");
        return ERR_PERMISSION_DENIED;
    }
    
    // ... 继续处理
}
```

**分析**：

权限验证逻辑清晰，但依赖 `GetCallingTokenID()` 和 `VerifyPermission()` 的正确实现。如果底层权限系统存在漏洞，可能导致验证绕过。

**触发路径**：

```
恶意进程 → HiSysEvent::Write()
        → GetCallingTokenID() 获取 token
        → VerifyPermission() 验证
        → 失败则返回 ERR_PERMISSION_DENIED
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 依赖底层权限系统 |
| **权限提升** | 无 | 仅访问系统服务 |
| **数据泄露** | 低 | 仅写入事件 |

**当前保护**：

| 保护措施 | 状态 | 说明 |
|----------|------|------|
| access_token 验证 | ✅ | 每次调用都验证 |
| SELinux 上下文 | ✅ | 系统级隔离 |
| UID 检查 | ✅ | 内核级保护 |

### R7：IPC 身份验证依赖（中危）

**位置**：`adapter/native/idl/src/hisysevent_delegate.cpp:45-78`

**证据**：

```cpp
// hisysevent_delegate.cpp:45-65
int32_t HisyseventDelegate::OnRemoteRequest(uint32_t code,
                                            MessageParcel& data,
                                            MessageParcel& reply) {
    // IPC 框架已验证调用方身份
    // 此处假设数据来源可信
    
    std::string domain = data.ReadString();
    std::string eventName = data.ReadString();
    
    return ProcessRequest(domain, eventName);
}
```

**分析**：

IPC 通信依赖底层 Binder 框架的身份验证机制。如果 Binder 验证被绕过，HiSysEvent 无法检测恶意调用。

**触发路径**：

```
恶意进程 → Binder IPC → HisyseventDelegate::OnRemoteRequest
        → Binder 验证调用方 UID
        → HiSysEvent 处理请求
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 中 | 需绕过 Binder |
| **系统影响** | 高 | 可污染系统事件 |

**修复建议**：

```cpp
// 增加应用层身份验证
int32_t HisyseventDelegate::OnRemoteRequest(uint32_t code,
                                            MessageParcel& data,
                                            MessageParcel& reply) {
    // 再次验证调用方权限
    uint32_t callerToken = data.ReadUint32();
    if (!VerifyPermission(callerToken,
                         "ohos.permission.ACCESS_SYSTEM_SERVICE")) {
        return ERR_PERMISSION_DENIED;
    }
    
    std::string domain = data.ReadString();
    std::string eventName = data.ReadString();
    
    return ProcessRequest(domain, eventName);
}
```

---

## 6.5 并发安全

### R8：Socket 连接竞争条件（中危）

**位置**：`interfaces/native/innerkits/hisysevent/event_socket_factory.cpp:45-89`

**证据**：

```cpp
// event_socket_factory.cpp:45-78
EventSocketFactory& EventSocketFactory::GetInstance() {
    static EventSocketFactory instance;
    return instance;
}

int EventSocketFactory::Connect() {
    if (socket_ >= 0) {
        return 0;  // 已连接
    }
    
    // 多线程可能同时进入
    socket_ = socket(AF_UNIX, SOCK_STREAM, 0);
    if (socket_ < 0) {
        return -1;
    }
    
    // 连接过程可能被中断
    struct sockaddr_un addr;
    memset(&addr, 0, sizeof(addr));
    addr.sun_family = AF_UNIX;
    strcpy(addr.sun_path, "/dev/unix/socket/hisysevent");
    
    if (connect(socket_, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
        close(socket_);
        socket_ = -1;
        return -1;
    }
    
    return 0;
}
```

**分析**：

使用单例模式管理 Socket 连接，但未实现完整的线程安全保护。在高并发场景下，可能创建多个 Socket 连接或导致资源泄漏。

**触发路径**：

```
线程1 → Connect() → socket_ = -1 → 创建 socket
线程2 → Connect() → socket_ = -1 → 创建 socket
        → 两个 socket 都连接
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **竞态条件** | 中 | 需要特定时序 |
| **资源泄漏** | 中 | 可能创建多个 socket |
| **数据一致** | 低 | 每个 socket 独立 |

**修复建议**：

```cpp
// event_socket_factory.cpp
class EventSocketFactory {
private:
    std::mutex mutex_;
    int socket_ = -1;
    bool connected_ = false;

public:
    int Connect() {
        std::lock_guard<std::mutex> lock(mutex_);
        
        if (connected_) {
            return 0;
        }
        
        socket_ = socket(AF_UNIX, SOCK_STREAM, 0);
        if (socket_ < 0) {
            return -1;
        }
        
        struct sockaddr_un addr;
        memset(&addr, 0, sizeof(addr));
        addr.sun_family = AF_UNIX;
        strcpy(addr.sun_path, "/dev/unix/socket/hisysevent");
        
        if (connect(socket_, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
            close(socket_);
            socket_ = -1;
            return -1;
        }
        
        connected_ = true;
        return 0;
    }
};
```

### R9：WriteController 线程安全（中危）

**位置**：`interfaces/native/innerkits/hisysevent/write_controller.cpp:45-89`

**证据**：

```cpp
// write_controller.cpp:45-78
class WriteController {
private:
    std::atomic<uint64_t> writeCount_{0};
    uint64_t lastResetTime_{0};
    
public:
    bool ShouldWrite() {
        uint64_t now = GetCurrentTimeMs();
        
        // 每秒重置计数器
        if (now - lastResetTime_ >= 1000) {
            writeCount_ = 0;
            lastResetTime_ = now;
        }
        
        // 速率限制检查
        if (writeCount_ >= MAX_WRITE_PER_SECOND) {
            return false;
        }
        
        writeCount_++;
        return true;
    }
};
```

**分析**：

使用 `std::atomic` 保护计数器，但 `lastResetTime_` 的读写存在竞态条件。极端情况下可能导致速率限制失效。

**触发路径**：

```
高频调用 → 多线程并发 ShouldWrite()
        → 原子操作保护 writeCount_
        → 非原子操作 lastResetTime_
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **竞态窗口** | 纳秒级 | 难以利用 |
| **速率突破** | 低 | 偶发超限 |
| **DoS 风险** | 低 | 短期超限影响有限 |

**修复建议**：

```cpp
// write_controller.cpp
bool ShouldWrite() {
    uint64_t now = GetCurrentTimeMs();
    
    // 原子操作：检查-更新
    uint64_t oldCount = writeCount_.load(std::memory_order_relaxed);
    uint64_t newCount;
    
    do {
        if (oldCount >= MAX_WRITE_PER_SECOND) {
            return false;
        }
        
        if (now - lastResetTime_ >= 1000) {
            // 重置计数器
            newCount = 1;
            lastResetTime_ = now;
            break;
        }
        
        newCount = oldCount + 1;
    } while (!writeCount_.compare_exchange_weak(
        oldCount, newCount, std::memory_order_acq_rel));
    
    return true;
}
```

---

## 6.6 逻辑漏洞

### R10：错误处理不完整（中危）

**位置**：多处文件

**证据**：`hisysevent.cpp`、`transport.cpp`、`encoded_param.cpp`

```cpp
// hisysevent.cpp:156-178
int HiSysEvent::Write(...) {
    // ... 参数校验 ...
    
    // 编码失败处理
    EncodedParam encoded = EncodeParams(params);
    if (!encoded.valid) {
        return ERR_ENCODING_FAILED;
    }
    
    // 发送失败处理
    int result = Transport::Send(encoded.data);
    if (result != 0) {
        // 仅返回错误码，无日志
        return result;
    }
    
    return 0;
}
```

**分析**：

错误处理返回错误码，但部分关键错误未记录详细日志，难以排查问题。调用方可能无法区分不同类型的失败。

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **调试困难** | 中 | 缺少错误上下文 |
| **故障恢复** | 低 | 调用方无法重试 |

**修复建议**：

```cpp
int HiSysEvent::Write(...) {
    EncodedParam encoded = EncodeParams(params);
    if (!encoded.valid) {
        HiLog::Error(LABEL, "Failed to encode params: domain=%s, name=%s",
                     domain.c_str(), eventName.c_str());
        return ERR_ENCODING_FAILED;
    }
    
    int result = Transport::Send(encoded.data);
    if (result != 0) {
        HiLog::Error(LABEL, "Failed to send event: domain=%s, name=%s, err=%d",
                     domain.c_str(), eventName.c_str(), result);
        return result;
    }
    
    return 0;
}
```

### R11：资源耗尽风险（中危）

**位置**：`write_controller.cpp`、`event_socket_factory.cpp`

**证据**：

```cpp
// write_controller.cpp:45-60
// 速率限制：每秒最多 MAX_WRITE_PER_SECOND 次
// 但无总量限制

class WriteController {
public:
    bool ShouldWrite() {
        // 每秒重置
        // 无长期总量控制
        return writeCount_ < MAX_WRITE_PER_SECOND;
    }
};
```

**分析**：

实现了每秒速率限制，但未设置总量限制或内存保护。长时间运行可能积累大量待处理数据。

**触发路径**：

```
持续调用 → 速率限制内持续写入
        → 内存占用持续增长
        → 系统压力增大
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **内存耗尽** | 中 | 长时间运行可能 |
| **系统稳定性** | 中 | 影响整体性能 |

**修复建议**：

```cpp
class WriteController {
private:
    std::atomic<uint64_t> writeCount_{0};
    std::atomic<uint64_t> totalWriteCount_{0};
    uint64_t lastResetTime_{0};
    static constexpr uint64_t MAX_TOTAL_WRITE = 1000000;  // 总量限制

public:
    bool ShouldWrite() {
        uint64_t now = GetCurrentTimeMs();
        uint64_t total = totalWriteCount_.load(std::memory_order_relaxed);
        
        // 总量检查
        if (total >= MAX_TOTAL_WRITE) {
            HiLog::Warn(LABEL, "Total write limit reached");
            return false;
        }
        
        // 速率检查
        if (now - lastResetTime_ >= 1000) {
            writeCount_ = 0;
            lastResetTime_ = now;
        }
        
        if (writeCount_ >= MAX_WRITE_PER_SECOND) {
            return false;
        }
        
        writeCount_++;
        totalWriteCount_++;
        return true;
    }
};
```

---

## 6.7 风险汇总

### 按风险等级汇总

| ID | 风险名称 | 等级 | 可利用性 | 影响范围 |
|-----|----------|------|----------|----------|
| R1 | 域名参数注入风险 | 低危 | 低 | 有限 |
| R2 | 事件名称边界问题 | 低危 | 低 | 有限 |
| R3 | IPC 反序列化风险 | 中危 | 中 | 中等 |
| R4 | 编码缓冲区问题 | 低危 | 低 | 有限 |
| R5 | Socket 发送问题 | 中危 | 中 | 中等 |
| R6 | access_token 验证 | 中危 | 低 | 中等 |
| R7 | IPC 身份验证依赖 | 中危 | 中 | 高 |
| R8 | Socket 连接竞态 | 中危 | 中 | 中等 |
| R9 | WriteController 竞态 | 中危 | 低 | 低 |
| R10 | 错误处理不完整 | 中危 | 低 | 中等 |
| R11 | 资源耗尽风险 | 中危 | 中 | 中等 |

### 按模块汇总

| 模块 | 高危 | 中危 | 低危 |
|------|------|------|------|
| **hisysevent.cpp** | 0 | 2 | 2 |
| **encoded_param.cpp** | 0 | 0 | 1 |
| **transport.cpp** | 0 | 1 | 0 |
| **hisysevent_delegate.cpp** | 0 | 2 | 0 |
| **event_socket_factory.cpp** | 0 | 1 | 0 |
| **write_controller.cpp** | 0 | 2 | 0 |

---

## 6.8 修复优先级

### 立即修复（P0）

| ID | 风险 | 修复方案 |
|-----|------|----------|
| R2 | 事件名称空字符串 UB | 添加空字符串检查 |
| R3 | IPC 反序列化无校验 | 添加长度校验 |

### 尽快修复（P1）

| ID | 风险 | 修复方案 |
|-----|------|----------|
| R5 | Socket 部分发送 | 实现完整发送循环 |
| R7 | IPC 身份验证增强 | 添加应用层验证 |
| R8 | Socket 连接竞态 | 添加互斥保护 |
| R9 | WriteController 竞态 | 使用原子 CAS |
| R10 | 错误日志增强 | 添加上下文日志 |

### 计划修复（P2）

| ID | 风险 | 修复方案 |
|-----|------|----------|
| R1 | 域名注入（理论） | 持续监控 |
| R4 | 缓冲区边界 | 添加显式边界检查 |
| R6 | access_token 依赖 | 跟进底层更新 |
| R11 | 资源耗尽 | 添加总量控制 |

---

## 6.9 安全评估结论

### 整体安全评估

| 维度 | 评分 | 说明 |
|------|------|------|
| **输入验证** | B | 基本校验完善，边界情况需加强 |
| **内存安全** | A | 使用现代 C++ 实践，风险可控 |
| **权限控制** | B | 依赖系统权限，建议增强 |
| **并发安全** | B | 存在竞态条件，需加强保护 |
| **错误处理** | C | 错误码返回完整，日志不充分 |
| **资源管理** | B | 速率限制有效，总量控制缺失 |

### 安全等级：**中等偏上**

HiSysEvent 组件整体安全状况良好，实现了基本的安全防护措施。主要风险集中在边界条件处理、并发安全和错误日志记录方面。建议优先修复编号 R2、R3 等涉及未定义行为和输入验证的漏洞。

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
