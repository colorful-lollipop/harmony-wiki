# 安全风险评审

## 7.1 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         不可信区域                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    用户应用 (App)                      │   │
│  └────────────────────────┬────────────────────────────────┘   │
│                           │  ▲                                  │
│  ┌────────────────────────┘  │                                  │
│  │                   │   IPC/Socket                         │
│  │                   ▼  │                                  │
│  │  ┌──────────────────────────────────────────────────┐   │   │
│  │  │              IPC Framework                       │   │   │
│  │  │  (Binder Driver, MessageParcel, IPCSkeleton)    │   │   │
│  │  └────────────────────────┬────────────────────────┘   │   │
│  │                           │  ▲                            │
│  │  ┌────────────────────────┘  │                            │
│  │  │                   │  │ System Service                  │
│  │  │                   ▼  │ (SA Process)                    │
│  │  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │  │              Trusted Service                  │   │   │
│  │  │  │  - SAMgr                                     │   │   │
│  │  │  │  - System Ability                            │   │   │
│  │  │  └──────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                    ┌─────────▼─────────┐                         │
│                    │   Linux Kernel    │                         │
│                    │   (Binder Driver) │                         │
│                    └───────────────────┘                         │
└─────────────────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| **N-API 接口** | `@ohos/rpc` 暴露的 JS 接口 | 高 |
| **C API** | `ipc_capi` 暴露的 C 接口 | 高 |
| **Binder Driver** | 内核 Binder 通信 | 中 |
| **MessageParcel 解析** | 数据反序列化 | 高 |
| **远程对象传递** | IRemoteObject 跨进程传递 | 高 |
| **DBinder 通信** | 跨设备 Socket 通信 | 中 |
| **接口令牌校验** | interface token 验证 | 中 |

## 7.2 数据流分析

### IPC 数据流

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Client  │────▶│  Proxy   │────▶│  Binder   Stub   │
│  App │────▶│     │     │  IPC     │     │  Driver  │     │  IPC    │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                │                  │                  │
     │ WriteXXX()     │ SendRequest()    │ OnRemoteRequest()│ HandleRequest()
     ▼                ▼                  ▼                  ▼
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│MessageParcel│   │MessageParcel│   │MessageParcel│   │MessageParcel│
└──────────┘     └──────────┘     └──────────┘     └──────────┘
```

### 信任链

| 阶段 | 数据来源 | 信任假设 |
|------|----------|----------|
| Client 写入 | 不可信用户输入 | ❌ 需要校验 |
| Proxy 序列化 | 可信框架 | ✅ |
| Binder 传输 | 内核空间 | ✅ |
| Stub 反序列化 | 不可信数据 | ❌ 需要校验 |
| 业务处理 | 业务逻辑 | ⚠️ 视情况 |

## 7.3 安全风险清单

### 风险 1：MessageParcel 缓冲区溢出

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-001 |
| **严重程度** | 高 |
| **类型** | 内存安全 |
| **证据** | `message_parcel.h:97-105` - `ReadRawData()` |

**问题描述**:
```cpp
// message_parcel.h:97
const void *ReadRawData(size_t &rawDataSize) const;
```

`ReadRawData()` 返回的 `rawDataSize` 可能被恶意构造的数据控制。

**触发条件**:
1. 远程攻击者发送畸形 IPC 消息
2. `WriteRawData()` 写入超大数据（> 1MB）
3. `ReadRawData()` 返回大小与预期不符

**影响**:
- 堆溢出
- 远程代码执行（RCE）

**修复建议**:
```cpp
// 在 ReadRawData 前进行边界检查
size_t maxSize = data.GetRawDataCapacity();
if (rawDataSize > maxSize || rawDataSize > MAX_ALLOWED_SIZE) {
    return nullptr;
}
```

### 风险 2：接口令牌缺失校验

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-002 |
| **严重程度** | 高 |
| **类型** | 认证绕过 |
| **证据** | `message_parcel.h:81-88` - `WriteInterfaceToken()` |

**问题描述**:
`WriteInterfaceToken()` / `ReadInterfaceToken()` 用于接口校验，但 Stub 可能未正确校验。

**触发条件**:
1. Stub 未调用 `ReadInterfaceToken()`
2. 或未与预期 Descriptor 比对
3. 攻击者伪造接口令牌调用敏感方法

**影响**:
- 未授权访问敏感系统服务
- 权限提升

**修复建议**:
```cpp
int TestAbilityStub::OnRemoteRequest(uint32_t code, MessageParcel &data,
                                    MessageParcel &reply, MessageOption &option) {
    // 必须校验接口令牌
    if (data.ReadInterfaceToken() != GetDescriptor()) {
        return -1;  // ERR_INVALID_PARAMETERS
    }
    // ...
}
```

**证据**: `README_zh.md:351` - 示例代码

### 风险 3：跨设备 DBinder 会话劫持

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-003 |
| **严重程度** | 中 |
| **类型** | 中间人攻击 |
| **证据** | `dbinder_service.h:77` - `SessionInfo` |

**问题描述**:
DBinder 使用 DSoftBus Socket 进行跨设备通信，但会话建立过程可能存在劫持风险。

**触发条件**:
1. 攻击者控制网络中间节点
2. DSoftBus 未启用加密
3. SessionInfo 中的设备 ID 未校验

**影响**:
- 跨设备 IPC 窃听
- 会话注入

**修复建议**:
- 启用 DSoftBus TLS 加密
- 校验设备证书
- 实现设备认证

### 风险 4：死亡通知回调滥用

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-004 |
| **严重程度** | 中 |
| **类型** | 拒绝服务 |
| **证据** | `iremote_object.h:123-130` - `AddDeathRecipient()` |

**问题描述**:
`AddDeathRecipient()` 注册死亡通知，但未限制注册数量。

**触发条件**:
1. 恶意应用大量注册死亡通知
2. 内存耗尽
3. 服务崩溃

**影响**:
- 资源耗尽（DoS）
- 进程崩溃

**修复建议**:
```cpp
// 添加注册数量限制
static constexpr size_t MAX_DEATH_RECIPIENTS = 16;
if (deathRecipients_.size() >= MAX_DEATH_RECIPIENTS) {
    return ERR_NO_MEMORY;
}
```

### 风险 5：文件描述符泄漏

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-005 |
| **严重程度** | 中 |
| **类型** | 资源泄漏 |
| **证据** | `message_parcel.h:59-66` - `WriteFileDescriptor()` |

**问题描述**:
`WriteFileDescriptor()` 传递文件描述符，但接收方可能未正确关闭。

**触发条件**:
1. 发送方写入 FD
2. 接收方未调用 `reclaim()` 释放
3. FD 泄漏

**影响**:
- 文件句柄泄漏
- 资源耗尽

**修复建议**:
```cpp
// 接收方使用完毕后必须 reclaim
auto fd = data.ReadFileDescriptor();
if (fd >= 0) {
    // 使用 FD
    close(fd);  // 或 data.reclaim()
}
```

### 风险 6：SetCallingIdentity 滥用

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-IPC-006 |
| **严重程度** | 高 |
| **类型** | 权限提升 |
| **证据** | `ipc_skeleton.h:178` - `SetCallingIdentity()` |

**问题描述**:
`SetCallingIdentity()` 允许设置调用身份，可能被滥用于权限提升。

**触发条件**:
1. 普通应用调用 `SetCallingIdentity()`
2. 设置为高权限 Token
3. 调用敏感系统服务

**影响**:
- 权限提升
- 未授权访问

**修复建议**:
```cpp
// 仅系统服务可调用，移除普通应用调用权限
if (!IsSystemService()) {
    return ERR_PERMISSION_DENIED;
}
```

## 7.4 安全编码规范

### 输入校验

```cpp
// ✅ 正确: 校验输入参数
int32_t OnRemoteRequest(uint32_t code, MessageParcel &data,
                        MessageParcel &reply, MessageOption &option) override {
    // 1. 校验接口令牌
    if (data.ReadInterfaceToken() != GetDescriptor()) {
        return ERR_INVALID_PARAMETERS;
    }
    
    // 2. 校验 code 范围
    if (code < MIN_TRANSACTION_ID || code > MAX_TRANSACTION_ID) {
        return ERR_INVALID_PARAMETERS;
    }
    
    // 3. 校验数据长度
    if (data.GetDataSize() < MIN_REQUIRED_SIZE) {
        return ERR_INVALID_PARAMETERS;
    }
    
    return ERR_OK;
}
```

### 资源管理

```cpp
// ✅ 正确: 自动释放 MessageParcel
{
    MessageParcel data, reply;
    auto fd = data.ReadFileDescriptor();
    if (fd < 0) {
        return ERR_INVALID_PARAMETERS;
    }
    // 使用完毕后自动析构释放
}
// 或显式 reclaim
data.reclaim();
reply.reclaim();
```

### 错误处理

```cpp
// ✅ 正确: 完整的异常处理
int MyStub::OnRemoteRequest(uint32_t code, MessageParcel &data,
                            MessageParcel &reply, MessageOption &option) {
    // 校验失败返回错误码
    if (!data.WriteInterfaceToken(GetDescriptor())) {
        return ERR_NO_MEMORY;
    }
    
    // 写入结果
    if (!reply.WriteInt32(result)) {
        return ERR_NO_MEMORY;
    }
    
    return ERR_OK;
}
```

## 7.5 安全测试清单

### 静态检查

- [ ] 代码审计所有 `OnRemoteRequest()` 实现
- [ ] 检查 `ReadInterfaceToken()` 调用点
- [ ] 验证边界校验逻辑

### 模糊测试

- [ ] MessageParcel 序列化/反序列化
- [ ] 畸形 IPC 消息注入
- [ ] 超大数据包处理

### 渗透测试

- [ ] 跨设备 DBinder 会话劫持
- [ ] 权限提升尝试
- [ ] 拒绝服务攻击

---

*证据来源*:
- `message_parcel.h:26-164` - MessageParcel 接口
- `iremote_object.h:34-145` - IRemoteObject 接口
- `ipc_skeleton.h:22-178` - IPCSkeleton 接口
- `dbinder_service.h:59-165` - DBinderService 接口
- Phase 1 全局扫描结果
