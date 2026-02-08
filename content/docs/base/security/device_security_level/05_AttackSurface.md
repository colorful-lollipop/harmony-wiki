# 攻击面分析

> 最后更新：2026-02-07
> 版本：v3.0.0

## 5.1 外部输入清单

DSL M 模块接收外部输入的入口点分析，这些输入点是需要重点关注的安全边界。

### 5.1.1 SDK API 参数输入

**输入源**：应用进程通过 C API 调用传入

| 输入项 | 数据类型 | 处理位置 | 风险等级 |
|--------|----------|----------|----------|
| `DeviceIdentify` | `DeviceIdentify *` | `interfaces/inner_api/src/standard/device_security_level_proxy.cpp` | 中 |
| `RequestOption.challenge` | `uint64_t` | `services/dslm/dslm_core_process.c` | 中 |
| `RequestOption.timeout` | `uint32_t` | `services/dslm/dslm_core_process.c` | 低 |

**代码证据**：`interfaces/inner_api/include/device_security_info.h:42-54`

```c
int32_t RequestDeviceSecurityInfo(
    const DeviceIdentify *identify,      // [in] 外部输入
    const RequestOption *option,          // [in] 外部输入
    DeviceSecurityInfo **info);
```

### 5.1.2 IPC 接口数据

**输入源**：其他进程通过 Binder IPC 发送的数据

| 输入项 | 类型 | 处理位置 | 风险等级 |
|--------|------|----------|----------|
| `MessageParcel` 数据 | `MessageParcel &` | `services/sa/standard/dslm_ipc_process.cpp:62` | 高 |
| `DeviceIdentify` 序列化数据 | `uint8_t[]` | `DslmGetRequestFromParcel()` | 高 |
| `RequestOption` 序列化数据 | `RequestOption` | `DslmGetRequestFromParcel()` | 中 |
| 回调对象引用 | `IRemoteObject *` | `RemoteHolder::Push()` | 中 |

**代码证据**：`services/sa/standard/dslm_ipc_process.cpp:62-80`

```cpp
int32_t DslmIpcProcess::DslmGetRequestFromParcel(
    MessageParcel &data, 
    DeviceIdentify &identify,
    RequestOption &option,
    sptr<IRemoteObject> &callback,
    uint32_t &cookie)
{
    // 从 MessageParcel 反序列化输入数据
    // 风险点: 未验证数据长度和有效性
}
```

### 5.1.3 凭证解析输入

**输入源**：远端设备通过 DSoftBus 发送的凭证数据

| 输入项 | 类型 | 处理位置 | 风险等级 |
|--------|------|----------|----------|
| 凭证字符串 | `const char *` | `oem_property/common/dslm_credential_utils.c` | 高 |
| JWS 头部 | `JSON *` | `dslm_credential_utils.c` | 中 |
| JWS 载荷 | `JSON *` | `dslm_credential_utils.c` | 中 |
| 签名数据 | `uint8_t[]` | `dslm_credential_utils.c` | 高 |

**代码证据**：`oem_property/common/dslm_credential_utils.c:89-150`

```c
int32_t ParseDslmCredential(
    const DslmCredBuff *credBuff, 
    DslmCredInfo *credInfo)
{
    // 解析 JWS 格式的凭证
    // 风险点: JSON 解析、字符串处理、缓冲区操作
}
```

### 5.1.4 配置文件输入

**输入源**：系统配置文件

| 输入项 | 文件路径 | 处理位置 | 风险等级 |
|--------|----------|----------|----------|
| 设备凭证文件 | `/system/etc/dslm_finger*.cfg` | `oem_property/ohos/standard/dslm_ohos_credential.c` | 低 |
| 服务配置 | `profile/dslm_service.cfg` | 系统加载时 | 低 |
| SA 配置 | `profile/dslm_service.xml` | SAMgr 加载时 | 低 |

---

## 5.2 敏感操作清单

### 5.2.1 系统调用操作

| 操作 | 调用位置 | 权限要求 | 风险等级 |
|------|----------|----------|----------|
| `dlopen()` 动态库加载 | `services/sa/standard/dslm_service.cpp:184` | 无 | 高 |
| 进程间通信 (IPC) | `services/sa/standard/dslm_ipc_process.cpp` | `ohos.permission.ACCESS_SERVICE_DM` | 高 |
| 文件读取 | `oem_property/ohos/standard/dslm_ohos_credential.c` | 文件系统权限 | 中 |

**代码证据**：`services/sa/standard/dslm_service.cpp:181-189`

```cpp
void DslmService::ProcessLoadPlugin(void)
{
#ifdef PLUGIN_SO_PATH
    handle_ = dlopen(PLUGIN_SO_PATH, RTLD_NOW);  // 动态库加载
    if (!handle_) {
        SECURITY_LOG_ERROR("load %{public}s failed for %{public}s", 
                           PLUGIN_SO_PATH, dlerror());
    }
#endif
}
```

### 5.2.2 加密操作

| 操作 | 调用位置 | 加密算法 | 风险等级 |
|------|----------|----------|----------|
| ECDSA 签名验证 | `dslm_credential_utils.c:522-577` | ECDSA + SHA256/SHA384 | 高 |
| 证书链验证 | `external_interface_adapter.c:107-161` | X.509 + HUKS | 高 |
| Nonce 验证 | `dslm_ohos_verify.c:68-167` | SHA256 | 中 |
| 密钥 attestation | `hks_adapter.c` | RSA-2048 + PSS | 高 |

**代码证据**：`oem_property/common/dslm_credential_utils.c:522-540`

```c
int32_t EcdsaVerify(
    const DataBuffer *srcData,      // 签名原文
    const DataBuffer *sigData,      // 签名数据
    const DataBuffer *pbkData,      // 公钥
    uint32_t algorithm)             // 算法标识
{
    // 使用 OpenSSL EVP 接口进行 ECDSA 签名验证
    EVP_PKEY *pkey = EVP_PKEY_new();
    EVP_PKEY_CTX *ctx = EVP_PKEY_CTX_new_id(EVP_PKEY_EC, NULL);
    // ... 加密操作
}
```

### 5.2.3 权限检查操作

| 操作 | 调用位置 | 检查项 | 风险等级 |
|------|----------|--------|----------|
| IPC 调用者身份验证 | `IPCSkeleton::GetCallingPid()` | PID 校验 | 中 |
| 权限声明检查 | `profile/dslm_service.cfg` | APL 级别 | 中 |
| SELinux 上下文检查 | 系统框架 | u:r:dslm_service:s0 | 低 |

---

## 5.3 信任边界图

### 信任边界定义

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          信任边界划分                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     信任边界 1: 应用进程边界                          │   │
│  │                                                                      │   │
│  │  ┌─────────┐    ┌─────────────┐                                      │   │
│  │  │ 应用组件 │───►│  dslm_sdk   │                                      │   │
│  │  └─────────┘    │  (可信)     │                                      │   │
│  │                  └──────┬──────┘                                      │   │
│  └──────────────────────────┼─────────────────────────────────────────────┘   │
│                             │                                               │
│                             ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     信任边界 2: IPC 通信边界                         │   │
│  │                                                                      │   │
│  │              ┌────────┐    ┌────────────┐                           │   │
│  │              │ Binder │───►│ dslm_sa    │                           │   │
│  │              │ IPC    │    │ (可信)     │                           │   │
│  │              └────────┘    └──────┬─────┘                           │   │
│  └─────────────────────────────────┼───────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                  信任边界 3: 核心服务边界                             │   │
│  │                                                                      │   │
│  │   ┌───────────┐    ┌───────────┐    ┌───────────┐                  │   │
│  │   │ dslm_core │───►│   OEM     │───►│  外部依赖  │                  │   │
│  │   │ (可信)    │    │ adapter   │    │ (HUKS等)  │                  │   │
│  │   └───────────┘    └───────────┘    └───────────┘                  │   │
│  │                                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 边界跨越点分析

| 跨越点 | 方向 | 跨越方式 | 验证机制 |
|--------|------|----------|----------|
| 应用 → SDK | 输入 | C 函数调用 | 参数校验 |
| SDK → SA | 跨进程 | Binder IPC | 接口令牌验证 |
| SA → 核心 | 进程内 | 函数调用 | 内部接口检查 |
| 核心 → OEM | 插件调用 | 函数指针 | 插件签名验证 |
| OEM → 外部 | 系统调用 | HUKS/DeviceAuth | 硬件安全边界 |

**代码证据**：`services/sa/standard/dslm_service.cpp:131-134`

```cpp
if (IDeviceSecurityLevel::GetDescriptor() != data.ReadInterfaceToken()) {
    SECURITY_LOG_ERROR("local descriptor is not equal remote");
    break;  // 验证失败，拒绝请求
}
```

---

## 5.4 攻击向量分析

### 5.4.1 输入验证类攻击

#### 攻击向量 1: 畸形数据注入

**攻击描述**：

攻击者构造畸形的 `DeviceIdentify` 或 `RequestOption` 数据，通过 IPC 发送给 DSLM 服务，可能导致：

- 缓冲区溢出
- 整数溢出
- 空指针解引用
- 解析错误

**触发路径**：

```
恶意应用
    │
    ▼
Binder IPC (MessageParcel)
    │
    ▼
DslmGetRequestFromParcel()  // interfaces/inner_api/src/standard/dslm_ipc_process.cpp:62
    │
    ▼
ReadInterfaceToken() → ReadBuffer() → 数据解析
    │
    ▼
边界检查不充分 → 异常
```

**影响范围**：

- 服务崩溃 (DoS)
- 可能的代码执行

**证据代码**：`services/sa/standard/dslm_ipc_process.cpp:62-80`

```cpp
int32_t DslmIpcProcess::DslmGetRequestFromParcel(
    MessageParcel &data, 
    DeviceIdentify &identify,
    RequestOption &option,
    sptr<IRemoteObject> &callback,
    uint32_t &cookie)
{
    // 从 data 读取 identify
    uint32_t len = data.ReadUint32();  // 攻击者可控
    // 后续使用 len 作为缓冲区大小
}
```

#### 攻击向量 2: 凭证注入

**攻击描述**：

攻击者伪造或篡改设备凭证数据，注入到验证流程中。

**触发路径**：

```
伪造凭证
    │
    ▼
DSoftBus 消息
    │
    ▼
ParseDslmCredential()  // oem_property/common/dslm_credential_utils.c:89
    │
    ▼
JSON 解析 → JWS 验证
    │
    ▼
验证绕过 → 虚假身份
```

**影响范围**：

- 身份伪造
- 安全等级欺骗
- 信任链破坏

---

### 5.4.2 资源耗尽攻击

#### 攻击向量 3: 请求洪泛

**攻击描述**：

攻击者短时间内发送大量安全等级查询请求，耗尽服务资源。

**触发路径**：

```
恶意应用
    │
    ▼
RequestDeviceSecurityInfoAsync() × N
    │
    ▼
RemoteHolder::Push()  // 注册回调
    │
    ▼
设备列表增长 → notifyListSize 累积
    │
    ▼
资源耗尽
```

**代码证据**：`services/dslm/dslm_core_process.c:157-165`

```c
int32_t OnRequestDeviceSecLevelInfo(..., DslmNotifyListNode *node)
{
    // ...
    if (device->notifyListSize >= MAX_NOTIFY_SIZE) {  // MAX_NOTIFY_SIZE = 64
        SECURITY_LOG_ERROR("notify list full");
        return ERR_MSG_NEIGHBOR_FULL;
    }
    // 添加回调到列表
}
```

**防护机制**：

| 检查项 | 检查位置 | 返回值 |
|--------|----------|--------|
| notifyListSize ≤ 64 | `dslm_core_process.c:157` | `ERR_MSG_NEIGHBOR_FULL` |

---

### 5.4.3 权限提升攻击

#### 攻击向量 4: IPC 回调劫持

**攻击描述**：

攻击者通过 IPC 发送恶意回调对象引用，劫持回调分发流程。

**触发路径**：

```
恶意应用
    │
    ▼
构造恶意 IRemoteObject
    │
    ▼
IPC 请求携带恶意回调
    │
    ▼
RemoteHolder::Push()  // 存储回调引用
    │
    ▼
DslmCallbackProxy::ResponseDeviceSecurityLevel()  // 调用回调
    │
    ▼
恶意代码执行
```

**影响范围**：

- 回调目标劫持
- 敏感信息泄露
- 代码执行

---

### 5.4.4 重放攻击

#### 攻击向量 5: Nonce 重放

**攻击描述**：

攻击者重放之前截获的有效凭证请求/响应对。

**触发路径**：

```
合法请求/响应对
    │
    ▼
网络嗅探截获
    │
    ▼
重放请求
    │
    ▼
DslmGetRequestFromParcel()  // 解析请求
    │
    ▼
Challenge 验证 (如果存在)
    │
    ▼
验证通过 → 接受旧凭证
```

**防护机制**：`dslm_ohos_verify.c:68-167`

```c
static int32_t VerifyNonceOfCertChain(...)
{
    // 验证 Nonce 是否匹配
    // 验证时间戳防止重放
}
```

**防护效果**：

| 防护项 | 实现位置 | 说明 |
|--------|----------|------|
| Nonce 匹配 | `dslm_ohos_verify.c` | 要求请求和响应中的 Nonce 一致 |
| 时间戳 | `dslm_ohos_verify.c` | 限制凭证有效期 |

---

## 5.5 安全边界总结

### 输入处理安全矩阵

| 输入类型 | 入口点 | 验证机制 | 风险等级 |
|----------|--------|----------|----------|
| SDK API 参数 | C 函数入口 | 参数空值检查 | 低 |
| IPC 数据 | MessageParcel | 接口令牌验证 | 中 |
| 凭证数据 | ParseDslmCredential | JWS 签名验证 | 高 |
| 配置文件 | 文件读取 | 系统完整性保护 | 低 |

### 敏感操作授权矩阵

| 操作 | 调用者 | 权限要求 | 审计日志 |
|------|--------|----------|----------|
| 查询设备等级 | 系统组件 | 无（内部 API） | HiEvent |
| 加载插件 | 系统服务 | SELinux | HiEvent |
| 凭证验证 | 系统服务 | SELinux | HiEvent |
| 服务卸载 | SAMgr | 系统权限 | HiEvent |

---

## 下一章

- [06_SecurityReview.md](./06_SecurityReview.md) - 安全风险评估
- [07_Build.md](./07_Build.md) - 构建与产物
- [08_Internals.md](./08_Internals.md) - 内部实现细节
