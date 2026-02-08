# 安全风险评审

## 目的

本文档基于代码证据，对 DSLM 模块进行安全风险评审，识别攻击面、信任边界和可被利用点。

## 适用范围

- ✅ 攻击面清单
- ✅ 信任边界与数据流
- ✅ 至少 5 条可被利用点（基于代码证据）
- ✅ 检查范围说明

## 检查范围

本文档基于以下代码范围进行安全评审：
- ✅ 主要源代码（排除 test/ 目录）
- ✅ IPC 接口与参数校验
- ✅ 凭据验证逻辑
- ✅ 消息通信
- ✅ 内存管理

**未检查范围**：
- ❌ 测试代码（test/ 目录）
- ❌ 外部依赖模块（dsoftbus、huks、device_auth 等）

## 攻击面清单

### 1. IPC 攻击面

| 攻击面 | 暴露点 | 证据 | 风险等级 |
|---------|--------|------|---------|
| **IPC 接口参数** | RequestDeviceSecurityLevel 参数 | `services/sa/standard/dslm_ipc_process.cpp` | 中 |
| **序列化/反序列化** | MessageParcel 读/写 | `device_security_level_proxy.cpp:67-78` | 中 |
| **调用者身份** | GetCallingPid()/GetCallingPid() | `dslm_ipc_process.cpp:108, dslm_ipc_process.c:168` | 低 |

### 2. 凭据验证攻击面

| 攻击面 | 暴露点 | 证据 | 风险等级 |
|---------|--------|------|---------|
| **证书链验证** | VerifyCredentialCb | `oem_property/common/dslm_credential_utils.c:180-206` | 高 |
| **ECDSA 签名验证** | EcdsaVerify | `dslm_credential_utils.c:522-577` | 高 |
| **Challenge-Nonce 验证** | VerifyNonceOfCertChain | `dslm_ohos_verify.c:254` | 中 |
| **凭据解析** | VerifyDslmCredential | `dslm_ohos_verify.c:235-278` | 中 |

### 3. 设备间通信攻击面

| 攻击面 | 暴露点 | 证据 | 风险等级 |
|---------|--------|------|---------|
| **消息发送** | SendMsgTo | `baselib/msglib/include/messenger.h:86-87` | 低 |
| **设备列表管理** | DslmDeviceInfo | `services/dslm/dslm_device_list.c` | 中 |

### 4. 内存管理攻击面

| 攻击面 | 暴露点 | 证据 | 风险等级 |
|---------|--------|------|---------|
| **设备列表内存** | AddDslmDeviceInfo/DelDslmDeviceInfo | `services/dslm/dslm_device_list.c` | 中 |
| **凭据信息内存** | FreeDeviceSecurityInfo | `interfaces/inner_api/src/standard/device_security_info.cpp` | 低 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                     不信任区域（外部）                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  应用进程（未信任）                                    │   │
│  │  - 调用 C API                                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ IPC（Binder）
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│              信任边界：DSL Service（SA 3511）                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  DslmService（System Ability）                           │   │
│  │  - 验证调用者身份（GetCallingPid）                    │   │
│  │  - 验证接口描述符（ReadInterfaceToken）               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  信任区域：DSL Core Logic                             │   │
│  │  - 设备列表管理（DslmDeviceInfo List）                 │   │
│  │  - 凭据验证（OEM Adapter）                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │
                    ┌───────────┬───────────┐
                    ▼             ▼             ▼
         ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
         │Huks         │  │DeviceManager│  │  DSoftBus    │
         │可信密钥存储  │  │设备管理器    │  │通信总线      │
         └─────────────┘  └─────────────┘  └─────────────┘
```

## 可被利用点（基于代码证据）

### 风险 1：证书链验证可能绕过

**证据**：`oem_property/common/dslm_credential_utils.c:180-206`

```c
static bool VerifyCredentialCb(const CredentialCb *credCb) {
    // root key, signed by self
    int32_t ret = EcdsaVerify(&root->publicKey, &root->signature, &root->publicKey, root->algorithm);

    // intermediate key, signed by root key
    ret = EcdsaVerify(&intermediate->publicKey, &intermediate->signature, &root->publicKey, intermediate->algorithm);

    // last key, signed by intermediate key
    ret = EcdsaVerify(&last->publicKey, &last->signature, &intermediate->publicKey, last->algorithm);

    // payload, signed by last key
    ret = EcdsaVerify(&payload->payload, &payload->signature, &last->publicKey, TYPE_ECDSA_SHA_384);
    if (ret != SUCCESS) {
        ret = EcdsaVerify(&payload->payload, &payload->signature, &last->publicKey, TYPE_ECDSA_SHA_256);
    }
}
```

**风险**：如果 ECDSA 签名验证实现存在漏洞（如侧信道攻击、签名伪造），攻击者可能伪造设备凭据。

**影响**：高 - 可能伪造任意设备的安全等级。

**触发**：攻击者向 DSLM 提供伪造的凭据。

**修复建议**：
1. 确保使用密码学安全的 ECDSA 验证实现
2. 添加恒定时间比较（constant-time comparison）
3. 使用经过安全审计的加密库（已使用 OpenSSL，确保版本是最新的）

---

### 风险 2：Challenge-Nonce 验证可能存在重放攻击

**证据**：`oem_property/ohos/common/dslm_ohos_verify.c:254`

```c
ret = VerifyNonceOfCertChain(resultInfo.nonceStr, device, challenge);
```

**风险**：如果 Nonce 验证逻辑不够强（如只比较前几位），攻击者可能重放旧凭据。

**影响**：中 - 可能重放已过期的凭据。

**触发**：攻击者截获有效的凭据，在 Nonce 过期前重放。

**修复建议**：
1. 确保完整的 Nonce 比较（64 位或更高）
2. 添加 Nonce 过期时间检查
3. 记录已使用 Nonce，防止重放

---

### 风险 3：设备列表管理可能存在内存泄漏

**证据**：`services/dslm/dslm_device_list.c`（增删查操作）

**风险**：如果设备列表的 Add/Del 操作存在内存管理问题，可能导致内存泄漏或释放后使用。

**影响**：中 - 长时间运行可能导致内存耗尽。

**触发**：频繁的设备上线/离线操作。

**修复建议**：
1. 使用智能指针（Standard 版本）
2. 添加内存泄漏检测工具（ASan）
3. 代码审查所有 Add/Del/Find 操作的内存管理

---

### 风险 4：IPC 参数校验不完整

**证据**：`services/sa/standard/dslm_ipc_process.cpp:108-120`

```cpp
auto owner = IPCSkeleton::GetCallingPid();
// ... 处理 identify 和 option 参数
if (length == 0 || length > DEVICE_ID_MAX_LEN) {
    return ERR_INVALID_LEN_PARA;
}
```

**风险**：只校验了设备标识符的长度，未校验其他字段（challenge、timeout）。

**影响**：低 - 可能传递异常参数导致未定义行为。

**触发**：应用调用 API 时传入异常参数。

**修复建议**：
1. 添加所有参数的完整性校验
2. 对 timeout 进行范围检查（如 0-UINT32_MAX）
3. 对 challenge 进行格式或范围检查

---

### 风险 5：MessageParcel 序列化可能存在缓冲区溢出

**证据**：`interfaces/inner_api/src/standard/device_security_level_proxy.cpp:67-78`

```cpp
data.WriteUint32(length);
data.WriteBuffer(identify.identity, DEVICE_ID_MAX_LEN);
data.WriteUint64(option.challenge);
data.WriteUint32(option.timeout);
data.WriteUint32(option.extra);
```

**风险**：如果 MessageParcel 的 WriteBuffer 实现未正确检查缓冲区边界，可能存在缓冲区溢出。

**影响**：高 - 可能导致内存损坏或任意代码执行。

**触发**：攻击者构造超长的设备标识符。

**修复建议**：
1. 确保 MessageParcel 实现有边界检查
2. 使用长度前检查缓冲区可用空间
3. 添加溢出保护编译选项（已启用）

---

### 额外发现：安全加固已启用

**证据**：`interfaces/inner_api/BUILD.gn:102-109`

```gn
sanitize = {
  integer_overflow = true
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
}
```

**说明**：Standard 版本已启用以下安全加固：
- ✅ 整数溢出检查
- ✅ 未定义行为检测
- ✅ 边界检查
- ✅ 控制流完整性（CFI）

## 检查范围与局限性

### 已检查范围
- ✅ IPC 接口与参数校验
- ✅ 凭据验证逻辑（证书链、ECDSA 签名）
- ✅ Challenge-Nonce 验证
- ✅ 设备列表内存管理
- ✅ MessageParcel 序列化

### 未检查范围
- ❌ 外部依赖模块（huks、device_auth、dsoftbus）的内部实现
- ❌ OpenSSL 的具体使用方式
- ❌ DSoftBus 的消息格式和校验
- ❌ 系统级别的权限检查（access_token 模块）

### 说明

本评审基于 DSLM 模块自身的代码进行。部分安全依赖外部模块（如 huks 的密钥存储、device_auth 的设备认证），这些模块的安全性不在本评审范围内。

## 安全最佳实践建议

1. **持续更新加密库**：确保 OpenSSL 等加密库是最新的安全版本
2. **定期安全审计**：对凭据验证、IPC 参数校验等关键代码进行审计
3. **启用安全加固**：Standard 版本已启用 CFI、UBSan、Integer Overflow Sanitize
4. **输入校验**：完善所有外部输入的校验逻辑
5. **内存安全**：使用智能指针、内存泄漏检测

## 关键结论

1. **攻击面**：IPC、凭据验证、设备通信、内存管理
2. **信任边界**：应用进程 → DSL Service → DSL Core Logic → OEM Adapter → 外部依赖
3. **可被利用点**：证书链验证、Nonce 验证、内存管理、IPC 参数校验、MessageParcel 序列化
4. **安全加固**：Standard 版本已启用 CFI、UBSan、Integer Overflow Sanitize
5. **局限性**：未涵盖外部依赖模块的内部实现

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 系统架构
- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 调用链详解
