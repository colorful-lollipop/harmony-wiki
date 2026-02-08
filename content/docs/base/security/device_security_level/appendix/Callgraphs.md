# 关键调用链

## 目的

本文档提供 DSLM 模块的关键调用链，帮助理解请求从入口到核心逻辑的完整流程。

## 适用范围

- ✅ 同步查询调用链
- ✅ 异步查询调用链
- ✅ 凭据验证调用链

## 同步查询调用链

### 调用流程图

```
应用进程                           DSLM 服务进程
    │                                    │
    │ RequestDeviceSecurityInfo()          │
    ▼                                    │
┌──────────────────────────┐                 │
│ DeviceSecurityInfo.cpp   │                 │
├──────────────────────────┤                 │
│ RequestDeviceSecurityInfo()              │
│  - 创建 DeviceSecurityLevelProxy          │
│  - 调用 LoadDslmService()            │
│  - proxy->RequestDeviceSecurityLevel()   │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DeviceSecurityLevelLoader │                 │
├──────────────────────────┤                 │
│ LoadDslmService()                     │
│  - 通过 SAMGR 加载 SA 3511           │
│  - 获取 IRemoteObject                  │
│  - 创建 DeviceSecurityLevelProxy        │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ IPC (Binder)                        │
├──────────────────────────┤                 │
│ MessageParcel:                        │
│  - WriteUint32(length)                  │
│  - WriteBuffer(identity, 64)            │
│  - WriteUint64(challenge)              │
│  - WriteUint32(timeout)               │
│  - WriteUint32(extra)                  │
│  - WriteRemoteObject(callback)          │
│  - WriteUint32(cookie)                 │
│  - SendRequest(CMD_GET_DEVICE_SECURITY_LEVEL)
│  - ReadUint32(status)                 │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DslmService                        │
├──────────────────────────┤                 │
│ OnRemoteRequest()                     │
│  - ReadInterfaceToken()                 │
│  - 调用 ProcessGetDeviceSecurityLevel()│
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DslmIpcProcess                   │
├──────────────────────────┤                 │
│ DslmProcessGetDeviceSecurityLevel()   │
│  - 调用 AddDslmDeviceInfo()           │
│  - 调用 RequestDeviceInfo()           │
│  - 创建 FSM 状态机                    │
│  - 保存 callback 到 RemoteHolder       │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DslmCoreProcess                   │
├──────────────────────────┤                 │
│ RequestDeviceInfo()                  │
│  - 调用 GetDeviceCred() [OEM]      │
│  - 调用 VerifyDslmCred() [OEM]      │
│  - 通过 Messenger 发送请求             │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ OEM Adapter (oem_property)         │
├──────────────────────────┤                 │
│ VerifyOhosDslmCred()                │
│  - ValidateCertChainAdapter()         │
│  - VerifyNonceOfCertChain()           │
│  - VerifyDslmCredential()             │
│  - CheckCredInfo()                    │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ 外部依赖                          │
├──────────────────────────┤                 │
│ DeviceManager: GetDeviceGroupInfo()  │
│ Huks: Sign/Verify()                 │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ 目标设备                          │
├──────────────────────────┤                 │
│ 返回凭据（证书链 + 签名）        │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DslmCoreProcess（响应处理）      │
├──────────────────────────┤                 │
│ ProcessDeviceInfoResponse()          │
│  - 更新 DslmDeviceInfo.result        │
│  - 触发 callback                    │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DslmCallbackProxy              │
├──────────────────────────┤                 │
│ ResponseDeviceSecurityLevel()           │
│  - 通过 IPC 调用应用 callback        │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ DeviceSecurityInfoCallbackStub     │
├──────────────────────────┤                 │
│ 调用用户提供的 callback              │
└──────────────────────────┘                 │
    │                                    │
    ▼                                    │
┌──────────────────────────┐                 │
│ 应用进程                          │
├──────────────────────────┤                 │
│ 用户 callback 函数                     │
└──────────────────────────┘                 │
```

### 关键函数调用路径

| 入口 | 调用链 | 最终目标 | 证据 |
|------|----------|----------|------|
| `RequestDeviceSecurityInfo()` | DeviceSecurityInfo.cpp → Proxy → SA → Core → OEM → External | 同步查询 |
| `RequestDeviceSecurityInfoAsync()` | DeviceSecurityInfo.cpp → Proxy → SA → Core → OEM → External | 异步查询 |
| `VerifyOhosDslmCred()` | 证书链验证 → 签名验证 → Nonce 验证 → 等级提取 | 凭据验证 |

## 异步查询调用链

### 调用流程图（与同步类似，但通过回调返回）

异步查询的调用链与同步查询基本相同，区别在于：

1. **回调机制**：应用提供 `DeviceSecurityInfoCallback` 函数指针
2. **异步返回**：DslmCallbackProxy 通过 IPC 回调应用，而非同步返回

**关键区别**：
- **同步**：等待 IPC 响应，立即返回结果
- **异步**：立即返回，通过回调异步通知结果

## 凭据验证调用链

### 证书链验证流程

```
VerifyOhosDslmCred()
    │
    ▼
ValidateCertChainAdapter()
    │ 解析凭据为证书链
    │
    ▼
VerifyCredentialCb() [逐层验证]
    │
    ├──► EcdsaVerify(root->publicKey, root->signature, root->publicKey)
    │   Root Key 自签名（信任根）
    │
    ├──► EcdsaVerify(intermediate->publicKey, intermediate->signature, root->publicKey)
    │   Intermediate 由 Root 签名
    │
    ├──► EcdsaVerify(last->publicKey, last->signature, intermediate->publicKey)
    │   Last 由 Intermediate 签名
    │
    └──► EcdsaVerify(payload->payload, payload->signature, last->publicKey)
        │   Payload 由 Last 签名（算法：SHA384）
        │
        └──► SHA384 失败则回退 SHA256
```

### ECDSA 签名验证流程

**证据**：`oem_property/common/dslm_credential_utils.c:522-577`

```
EcdsaVerify(srcData, sigData, pbkData, algorithm)
    │
    ▼
EVP_DigestVerifyInit(ctx, NULL, type, NULL, pkey)
    │ 初始化验证上下文
    │
    ▼
EVP_DigestUpdate(ctx, srcData->data, srcData->length)
    │ 更新摘要
    │
    ▼
EVP_DigestVerifyFinal(ctx, sigData->data, sigData->length)
    │ 验证签名
    │
    ▼
返回结果（SUCCESS 或失败）
```

## 关键文件位置

| 功能 | 文件路径 | 关键函数 |
|------|----------|----------|
| API 实现 | interfaces/inner_api/src/standard/device_security_info.cpp | RequestDeviceSecurityInfo() |
| Proxy | interfaces/inner_api/src/standard/device_security_level_proxy.cpp | RequestDeviceSecurityLevel() |
| SA 服务 | services/sa/standard/dslm_service.cpp | OnRemoteRequest() |
| IPC 处理 | services/sa/standard/dslm_ipc_process.cpp | DslmGetRequestFromParcel() |
| 核心逻辑 | services/dslm/dslm_inner_process.c | RequestDeviceInfo() |
| 凭据验证 | oem_property/ohos/common/dslm_ohos_verify.c | VerifyOhosDslmCred() |
| 证书验证 | oem_property/common/dslm_credential_utils.c | VerifyCredentialCb() |
| 签名验证 | oem_property/common/dslm_credential_utils.c | EcdsaVerify() |

## 关键结论

1. **同步/异步**：调用链基本相同，区别在于返回方式（立即返回 vs 回调）
2. **IPC 通信**：通过 Binder 的 MessageParcel 序列化/反序列化
3. **凭据验证**：逐层验证证书链（Root → Intermediate → Leaf），每层使用 ECDSA 验证签名
4. **依赖外部**：DeviceManager（获取设备信息）、Huks（签名/密钥）

## 相关跳转

- [03_Architecture.md](../03_Architecture.md) - 系统架构
- [05_Inner_APIs.md](../05_Inner_APIs.md) - 内部接口
- [09_FAQ.md](../09_FAQ.md) - 常见问题
