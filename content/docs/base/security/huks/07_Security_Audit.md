# 安全风险评审

> HUKS 安全设计和潜在风险分析

**目的**: 识别 HUKS 的安全风险点，提供修复建议
**适用范围**: 安全审计、系统开发者
**相关文档**: [项目概览](./00_Overview.md) | [架构说明](./02_Architecture.md) | [对外 API](./03_External_API.md)

---

## 1. 威胁模型

### 1.1 攻击面

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp`

| 攻击面 | 说明 |
|--------|------|
| **N-API 输入** | JS/TS 应用通过 N-API 传递恶意参数 |
| **IPC 通信** | 恶意应用通过 IPC 调用 HUKS 服务 |
| **文件存储** | 密钥文件的读写操作 |
| **权限控制** | 权限检查绕过 |
| **用户认证** | 用户认证结果伪造或重放 |

### 1.2 信任边界

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp`

```
┌─────────────────────────────────────────────────────────┐
│  不可信环境 (应用进程)                                 │
│  - 恶意应用                                          │
│  - 被攻击的应用                                        │
└─────────────────────────────────────────────────────────┘
                        ↓ [权限检查]
┌─────────────────────────────────────────────────────────┐
│  HUKS Service (普通执行环境)                            │
│  - 密钥存储（密文）                                    │
│  - 访问控制                                            │
│  - 会话管理                                            │
└─────────────────────────────────────────────────────────┘
                        ↓ [安全环境]
┌─────────────────────────────────────────────────────────┐
│  HUKS Core (TEE/安全环境)                               │
│  - 密钥明文                                            │
│  - 加密运算                                            │
│  - 访问控制                                            │
└─────────────────────────────────────────────────────────┘
```

---

## 2. 输入校验

### 2.1 参数类型检查

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp:56-63`

HUKS 使用 `napi_typeof` 检查参数类型。

**风险点**:
- ❌ 未检查参数类型可能导致类型混淆攻击
- ❌ 数组长度未限制可能导致缓冲区溢出

**当前实现**:
- ✅ 使用 `napi_typeof` 检查类型
- ✅ 最大数据长度限制: 100MB (0x6400000)

**证据**: `interfaces/kits/napi/include/v9/huks_napi_common_item.h`

### 2.2 参数值范围检查

**证据**: `services/huks_standard/huks_service/main/core/src/hks_client_check.c`

HUKS 检查参数值的有效性。

**风险点**:
- ❌ 未检查参数范围可能导致整数溢出
- ❌ 未检查参数组合可能导致逻辑错误

**当前实现**:
- ✅ 检查密钥大小的有效范围
- ✅ 检查算法参数的有效性

---

## 3. 权限控制

### 3.1 Token 类型检查

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp:66-81`

**风险点**:
- ❌ Token 类型检查不完整可能导致权限提升

**当前实现**:
- ✅ 检查 Token 类型 (TOKEN_NATIVE / TOKEN_SHELL / TOKEN_HAP)
- ✅ HAP 应用需要检查是否为系统应用
- ✅ 系统应用检查: `TokenIdKit::IsSystemAppByFullTokenID()`

### 3.2 Access Token 验证

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp:52`

**风险点**:
- ❌ 敏感权限未检查可能导致权限绕过

**当前实现**:
- ✅ 使用 `AccessTokenKit::VerifyAccessToken()` 验证权限
- ✅ 敏感权限: `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS`

### 3.3 UID 白名单检查

**证据**: `services/huks_standard/huks_service/main/core/src/hks_client_check.c:342-354`

**风险点**:
- ❌ UID 白名单可能包含不必要的 UID
- ❌ 白名单可被绕过

**当前实现**:
- ✅ 白名单检查用于修改存储级别
- ✅ 特殊 UID: ASSET_UID (6226)

**可信 UID 白名单**:
```cpp
static const int g_trustedUid[] = {
    1024, 3333, 3515, 3553, 6226, 7008, 7023, 7998
};
```

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp:154-163`

---

## 4. 可被利用点

### 4.1 跨账户访问权限绕过

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp:138-171`

**风险**:
- 调用者可能绕过跨账户访问权限检查，访问其他用户的密钥

**检查机制**:
- ✅ 检查 `HKS_TAG_SPECIFIC_USER_ID` 参数
- ✅ 调用 `SystemApiPermissionCheck()`
- ✅ 调用 `SensitivePermissionCheck("ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS")`
- ✅ 检查调用者 UID 是否在可信白名单中
- ✅ 检查 `specificUserId` 是否等于前台用户 ID

**建议**:
- 定期审查可信 UID 白名单
- 确保 `SystemApiPermissionCheck` 和 `SensitivePermissionCheck` 都正确实现

### 4.2 权限检查不完整

**证据**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp`

**风险**:
- 某些敏感操作可能缺少权限检查

**检查的敏感操作**:
- ✅ 跨用户访问密钥
- ✅ 修改存储级别
- ✅ UKey 操作

**建议**:
- 审查所有敏感操作的权限检查
- 确保所有 IPC 接口都调用权限检查函数

### 4.3 UID 白名单滥用

**证据**: `services/huks_standard/huks_service/main/core/src/hks_client_check.c`

**风险**:
- 白名单 UID 可能包含不必要的 UID
- 白名单可能被绕过

**建议**:
- 定期审查白名单 UID
- 确保每个 UID 都有明确的用途

### 4.4 参数类型混淆

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp`

**风险**:
- 参数类型检查不完整可能导致类型混淆攻击

**当前实现**:
- ✅ 使用 `napi_typeof` 检查类型
- ✅ 最大数据长度限制

**建议**:
- 确保所有参数都进行类型检查
- 使用强类型 API

### 4.5 整数溢出

**证据**: `services/huks_standard/huks_service/main/core/src/hks_client_check.c`

**风险**:
- 参数值未检查范围可能导致整数溢出

**当前实现**:
- ✅ 检查密钥大小的有效范围
- ✅ 检查算法参数的有效性

**建议**:
- 确保所有数值参数都检查范围
- 使用安全的算术运算

---

## 5. 内存安全

### 5.1 密钥存储安全

**证据**: `services/huks_standard/huks_service/main/hks_storage/src/hks_storage_manager.c`

**风险**:
- 密钥明文可能泄露到日志
- 密钥文件可能被未授权访问

**当前实现**:
- ✅ 密钥以密文形式存储
- ✅ 密钥明文仅在安全环境（TEE）中访问
- ✅ 存储级别: DE / CE / ECE

### 5.2 Buffer 管理

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp`

**风险**:
- Buffer 大小未限制可能导致缓冲区溢出

**当前实现**:
- ✅ 最大数据长度限制: 100MB
- ✅ 使用 `napi_get_typedarray_info` 获取 Buffer 信息

**建议**:
- 确保所有 Buffer 操作都检查大小
- 使用安全的内存操作函数

---

## 6. 信息泄露

### 6.1 错误信息泄露

**证据**: `interfaces/kits/napi/include/v9/huks_napi_common_item.h`

**风险**:
- 错误信息可能包含敏感信息

**当前实现**:
- ✅ 使用通用错误消息
- ✅ 不暴露内部实现细节

### 6.2 日志泄露

**证据**: `frameworks/huks_standard/main/common/include/hks_log.h`

**风险**:
- 日志可能包含敏感密钥信息

**当前实现**:
- ✅ 日志级别控制: `huks_enable_log`
- ✅ 不在日志中输出密钥明文

**建议**:
- 定期审查日志输出
- 确保不输出敏感信息

---

## 7. 竞态条件

### 7.1 密钥删除竞态

**证据**: `services/huks_standard/huks_service/main/core/src/hks_client_service.c`

**风险**:
- 密钥可能在使用过程中被删除

**当前实现**:
- ✅ 使用互斥锁保护密钥存储操作

**证据**: `utils/mutex/`

### 7.2 会话管理竞态

**证据**: `services/huks_standard/huks_service/main/core/src/hks_session_manager.c`

**风险**:
- 会话可能被并发访问导致状态不一致

**当前实现**:
- ✅ 使用会话 ID 标识会话
- ✅ 会话超时清理

---

## 8. 修复建议

### 8.1 输入校验

1. **加强参数类型检查**:
   - 确保所有参数都进行类型检查
   - 使用强类型 API

2. **加强参数值范围检查**:
   - 检查所有数值参数的范围
   - 防止整数溢出

3. **限制数据长度**:
   - 确保所有 Buffer 操作都检查大小
   - 使用安全的内存操作函数

### 8.2 权限控制

1. **完善权限检查**:
   - 审查所有敏感操作的权限检查
   - 确保所有 IPC 接口都调用权限检查函数

2. **审查白名单**:
   - 定期审查可信 UID 白名单
   - 确保每个 UID 都有明确的用途

3. **加强系统应用检查**:
   - 确保系统应用检查正确实现
   - 防止权限提升

### 8.3 内存安全

1. **加强密钥存储安全**:
   - 确保密钥以密文形式存储
   - 确保密钥明文仅在安全环境（TEE）中访问

2. **加强 Buffer 管理**:
   - 确保所有 Buffer 操作都检查大小
   - 使用安全的内存操作函数

### 8.4 信息泄露

1. **减少错误信息泄露**:
   - 使用通用错误消息
   - 不暴露内部实现细节

2. **减少日志泄露**:
   - 定期审查日志输出
   - 确保不输出敏感信息

### 8.5 竞态条件

1. **加强并发控制**:
   - 使用互斥锁保护共享资源
   - 确保原子操作

2. **加强会话管理**:
   - 使用会话 ID 标识会话
   - 会话超时清理

---

## 9. 检查范围

### 9.1 已检查范围

- ✅ N-API 参数校验 (`interfaces/kits/napi/src/v9/huks_napi_common_item.cpp`)
- ✅ IPC 权限检查 (`services/huks_standard/huks_service/main/os_dependency/idl/ipc/hks_permission_check.cpp`)
- ✅ 客户端参数检查 (`services/huks_standard/huks_service/main/core/src/hks_client_check.c`)
- ✅ 密钥存储管理 (`services/huks_standard/huks_service/main/hks_storage/src/hks_storage_manager.c`)
- ✅ 会话管理 (`services/huks_standard/huks_service/main/core/src/hks_session_manager.c`)

### 9.2 未检查范围

- ⚠️ 加密引擎实现 (`frameworks/huks_standard/main/crypto_engine/`)
- ⚠️ HAL 接口实现 (`services/huks_standard/huks_engine/main/core_dependency/`)
- ⚠️ UKey 扩展实现 (`services/huks_standard/huks_service/extension/ukey/`)

---

## 10. 相关文档

- [项目概览](./00_Overview.md) - HUKS 定位和核心能力
- [架构说明](./02_Architecture.md) - 架构设计和数据流
- [对外 API](./03_External_API.md) - API 接口详细说明
- [官方安全文档](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/UniversalKeystoreKit/Readme-CN.md) - OpenHarmony 安全文档
