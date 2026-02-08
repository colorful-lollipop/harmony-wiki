# 对外 API (N-API)

> HUKS 对外暴露的 N-API 接口详细说明

**目的**: 了解 HUKS 提供的 JS/TS API 接口、参数校验和错误处理
**适用范围**: 应用开发者
**相关文档**: [项目概览](./00_Overview.md) | [架构说明](./02_Architecture.md)

---

## 1. N-API 模块概览

### 1.1 模块列表

**证据**: `interfaces/kits/napi/src/huks_napi.cpp`, `huks_napi_ukey_module.cpp`

| 模块名 | JS 命名空间 | 注册点 | 函数数量 | 常量数量 |
|--------|-------------|--------|---------|---------|
| **主模块** | `@ohos.security.huks` | `huks_napi.cpp:833` | 40 | 25 |
| **UKey 模块** | `@ohos.security.huksExternalCrypto` | `huks_napi_ukey_module.cpp:109` | 5 | 3 |

### 1.2 API 版本演进

| 版本 | 新增功能 | 证据 |
|-----|---------|------|
| V8 | 基础 API | `v8/` 目录 |
| V9 | KeyItem 操作 | `v9/` 目录 |
| V12 | AsUser 操作 | `v12/` 目录 |

---

## 2. 主模块 API (`@ohos.security.huks`)

### 2.1 V8 API (旧版，11个)

**证据**: `interfaces/kits/napi/src/v8/`

| JS 方法名 | C++ 实现函数 | 文件 | 说明 |
|----------|-------------|------|------|
| `generateKey` | `HuksNapiGenerateKey` | `huks_napi_generate_key.cpp` | 生成密钥 |
| `deleteKey` | `HuksNapiDeleteKey` | `huks_napi_delete_key.cpp` | 删除密钥 |
| `getSdkVersion` | `HuksNapiGetSdkVersion` | `huks_napi_get_sdk_version.cpp` | 获取 SDK 版本 |
| `importKey` | `HuksNapiImportKey` | `huks_napi_import_key.cpp` | 导入密钥 |
| `exportKey` | `HuksNapiExportKey` | `huks_napi_export_key.cpp` | 导出公钥 |
| `getKeyProperties` | `HuksNapiGetKeyProperties` | `huks_napi_get_key_properties.cpp` | 获取密钥属性 |
| `isKeyExist` | `HuksNapiIsKeyExist` | `huks_napi_is_key_exist.cpp` | 检查密钥是否存在 |
| `init` | `HuksNapiInit` | `huks_napi_init.cpp` | 三阶段初始化 |
| `update` | `HuksNapiUpdate` | `huks_napi_update_finish.cpp` | 三阶段更新 |
| `finish` | `HuksNapiFinish` | `huks_napi_update_finish.cpp` | 三阶段完成 |
| `abort` | `HuksNapiAbort` | `huks_napi_abort.cpp` | 三阶段中止 |

### 2.2 V9 API (KeyItem 操作，14个)

**证据**: `interfaces/kits/napi/src/v9/`

| JS 方法名 | C++ 实现函数 | 文件 | 说明 |
|----------|-------------|------|------|
| `generateKeyItem` | `HuksNapiItemGenerateKey` | `huks_napi_generate_key_item.cpp` | 生成密钥 |
| `deleteKeyItem` | `HuksNapiDeleteKeyItem` | `huks_napi_delete_key_item.cpp` | 删除密钥 |
| `importKeyItem` | `HuksNapiImportKeyItem` | `huks_napi_import_key_item.cpp` | 导入密钥 |
| `importWrappedKeyItem` | `HuksNapiImportWrappedKeyItem` | `huks_napi_import_wrapped_key_item.cpp` | 导入包装密钥 |
| `exportKeyItem` | `HuksNapiExportKeyItem` | `huks_napi_export_key_item.cpp` | 导出公钥 |
| `getKeyItemProperties` | `HuksNapiGetKeyItemProperties` | `huks_napi_get_key_item_properties.cpp` | 获取密钥属性 |
| `isKeyItemExist` | `HuksNapiIsKeyItemExist` | `huks_napi_is_key_item_exist.cpp` | 检查密钥是否存在 |
| `hasKeyItem` | `HuksNapihasKeyItem` | `huks_napi_has_key_item.cpp` | 检查是否拥有密钥 |
| `attestKeyItem` | `HuksNapiAttestKeyItem` | `huks_napi_attest_key_item.cpp` | 密钥证明 |
| `anonAttestKeyItem` | `HuksNapiAnonAttestKeyItem` | `huks_napi_attest_key_item.cpp` | 匿名密钥证明 |
| `initSession` | `HuksNapiInitSession` | `huks_napi_init_session.cpp` | 会话初始化 |
| `updateSession` | `HuksNapiUpdateSession` | `huks_napi_update_finish_session.cpp` | 会话更新 |
| `finishSession` | `HuksNapiFinishSession` | `huks_napi_update_finish_session.cpp` | 会话完成 |
| `abortSession` | `HuksNapiAbortSession` | `huks_napi_abort_session.cpp` | 会话中止 |

### 2.3 V12 API (AsUser 操作，15个)

**证据**: `interfaces/kits/napi/src/v12/`

| JS 方法名 | C++ 实现函数 | 文件 | 说明 |
|----------|-------------|------|------|
| `generateKeyItemAsUser` | `HuksNapiItemGenerateKeyAsUser` | `huks_napi_generate_key_item_as_user.cpp` | 生成密钥（指定用户）|
| `deleteKeyItemAsUser` | `HuksNapiDeleteKeyItemAsUser` | `huks_napi_delete_key_item_as_user.cpp` | 删除密钥（指定用户）|
| `importKeyItemAsUser` | `HuksNapiImportKeyItemAsUser` | `huks_napi_import_key_item_as_user.cpp` | 导入密钥（指定用户）|
| `importWrappedKeyItemAsUser` | `HuksNapiImportWrappedKeyItemAsUser` | `huks_napi_import_wrapped_key_item_as_user.cpp` | 导入包装密钥（指定用户）|
| `exportKeyItemAsUser` | `HuksNapiExportKeyItemAsUser` | `huks_napi_export_key_item_as_user.cpp` | 导出公钥（指定用户）|
| `getKeyItemPropertiesAsUser` | `HuksNapiGetKeyItemPropertiesAsUser` | `huks_napi_get_key_item_properties_as_user.cpp` | 获取密钥属性（指定用户）|
| `hasKeyItemAsUser` | `HuksNapiHasKeyItemAsUser` | `huks_napi_has_key_item_as_user.cpp` | 检查是否拥有密钥（指定用户）|
| `attestKeyItemAsUser` | `HuksNapiAttestKeyItemAsUser` | `huks_napi_attest_key_item_as_user.cpp` | 密钥证明（指定用户）|
| `anonAttestKeyItemAsUser` | `HuksNapiAnonAttestKeyItemAsUser` | `huks_napi_attest_key_item_as_user.cpp` | 匿名密钥证明（指定用户）|
| `initSessionAsUser` | `HuksNapiInitSessionAsUser` | `huks_napi_init_session_as_user.cpp` | 会话初始化（指定用户）|
| `listAliases` | `HuksNapiListAliases` | `huks_napi_list_aliases.cpp` | 列出密钥别名 |
| `wrapKeyItem` | `HuksNapiWrapKey` | `huks_napi_wrap_key.cpp` | 包装密钥 |
| `unwrapKeyItem` | `HuksNapiUnwrapKey` | `huks_napi_unwrap_key.cpp` | 解包密钥 |

---

## 3. 导出的常量

### 3.1 错误码常量

**证据**: `interfaces/kits/napi/src/huks_napi.cpp:524-532`

| 常量对象 | 说明 |
|---------|------|
| `HuksExceptionErrCode` | 异常错误码 |
| `HuksErrorCode` | 错误码 |

### 3.2 算法参数常量

**证据**: `interfaces/kits/napi/src/huks_napi.cpp`

| 常量对象 | 说明 |
|---------|------|
| `HuksKeyPurpose` | 密钥用途（加密/解密/签名等）|
| `HuksKeyDigest` | 摘要算法（SHA256/SHA384/SHA512）|
| `HuksKeyPadding` | 填充模式（PKCS1/PKCS7/OAEP）|
| `HuksCipherMode` | 加密模式（ECB/CBC/CTR/GCM）|
| `HuksKeySize` | 密钥大小 |
| `HuksKeyAlg` | 密钥算法（RSA/ECC/AES/HMAC）|
| `HuksKeyGenerateType` | 密钥生成类型 |
| `HuksKeyFlag` | 密钥标志 |
| `HuksKeyStorageType` | 密钥存储类型 |

### 3.3 用户认证常量

**证据**: `interfaces/kits/napi/src/huks_napi.cpp`

| 常量对象 | 说明 |
|---------|------|
| `HuksUserAuthType` | 用户认证类型（PIN/人脸/指纹）|
| `HuksAuthAccessType` | 认证访问类型 |
| `HuksChallengeType` | 挑战类型 |
| `HuksUserAuthMode` | 用户认证模式 |
| `HuksChallengePosition` | 挑战位置 |

### 3.4 其他常量

**证据**: `interfaces/kits/napi/src/huks_napi.cpp`

| 常量对象 | 说明 |
|---------|------|
| `HuksImportKeyType` | 导入密钥类型 |
| `HuksUnwrapSuite` | 解包套件 |
| `HuksSendType` | 发送类型 |
| `HuksKeyClassType` | 密钥类别类型 |
| `HuksTag` | 参数标签 |
| `HuksTagType` | 标签类型 |
| `HuksSecureSignType` | 安全签名类型 |
| `HuksRsaPssSaltLenType` | RSA PSS 盐长度类型 |
| `HuksAuthStorageLevel` | 认证存储级别 |
| `HuksKeyWrapType` | 密钥包装类型 |

---

## 4. UKey 扩展模块 API (`@ohos.security.huksExternalCrypto`)

**证据**: `interfaces/kits/napi/src/ukey/huks_napi_ukey.cpp`

| JS 方法名 | C++ 实现函数 | 说明 |
|----------|-------------|------|
| `registerProvider` | `HuksNapiRegisterProvider` | 注册 UKey 提供者 |
| `unregisterProvider` | `HuksNapiUnregisterProvider` | 注销 UKey 提供者 |
| `authUkeyPin` | `HuksNapiAuthUkeyPin` | UKey PIN 认证 |
| `getUkeyPinAuthState` | `HuksNapiGetUkeyPinAuthState` | 获取 PIN 认证状态 |
| `getProperty` | `HuksNapiGetProperty` | 获取 UKey 属性 |

**UKey 常量**:

| 常量对象 | 说明 |
|---------|------|
| `HuksExternalCryptoTagType` | 扩展标签类型 |
| `HuksExternalCryptoTag` | 扩展标签 |
| `HuksExternalPinAuthState` | PIN 认证状态 |

---

## 5. 参数校验机制

### 5.1 类型检查

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp:56-63`

```cpp
napi_valuetype valueType = napi_valuetype::napi_undefined;
NAPI_CALL(env, napi_typeof(env, object, &valueType));
if (valueType != napi_valuetype::napi_string) {
    HksNapiThrow(env, HUKS_ERR_CODE_ILLEGAL_ARGUMENT, "the type of alias isn't string");
}
```

### 5.2 参数解析

**支持的类型**:

| 类型 | N-API 函数 | 证据 |
|-----|-----------|------|
| int32 | `napi_get_value_int32` | `huks_napi_common_item.cpp:220` |
| uint32 | `napi_get_value_uint32` | `huks_napi_common_item.cpp:224` |
| int64 | `napi_get_value_int64` | `huks_napi_common_item.cpp:228` |
| bool | `napi_get_value_bool` | `huks_napi_common_item.cpp:232` |
| string | `napi_get_value_string_utf8` | `huks_napi_common_item.cpp:66-88` |
| Uint8Array | `napi_get_typedarray_info` | `huks_napi_common_item.cpp:109-150` |

### 5.3 数据长度限制

**最大数据长度**: 100MB (0x6400000)

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.h`

---

## 6. 错误处理

### 6.1 错误抛出宏

**证据**: `interfaces/kits/napi/include/v9/huks_napi_common_item.h:38-50`

```cpp
#define NAPI_THROW_BASE_RETURN(env, condition, ret, code, message) \
    if ((condition)) { \
        HKS_LOG_E(message); \
        napi_throw((env), NapiCreateError((env), (code), (message))); \
        return (ret); \
    }
```

### 6.2 错误码定义

**JS 层错误码** (HUKS_ERR_CODE_*):

| 错误码 | 值 | 说明 |
|-------|-----|------|
| `HUKS_ERR_CODE_PERMISSION_FAIL` | 201 | 权限检查失败 |
| `HUKS_ERR_CODE_NOT_SYSTEM_APP` | 202 | 非系统应用 |
| `HUKS_ERR_CODE_ILLEGAL_ARGUMENT` | 401 | 非法参数 |
| `HUKS_ERR_CODE_NOT_SUPPORTED_API` | 801 | API 不支持 |
| `HUKS_ERR_CODE_FEATURE_NOT_SUPPORTED` | 12000001 | 特性不支持 |
| `HUKS_ERR_CODE_MISSING_CRYPTO_ALG_ARGUMENT` | 12000002 | 缺少加密算法参数 |
| `HUKS_ERR_CODE_INVALID_CRYPTO_ALG_ARGUMENT` | 12000003 | 无效加密算法参数 |
| `HUKS_ERR_CODE_FILE_OPERATION_FAIL` | 12000004 | 文件操作失败 |
| `HUKS_ERR_CODE_COMMUNICATION_FAIL` | 12000005 | 通信失败 |
| `HUKS_ERR_CODE_CRYPTO_FAIL` | 12000006 | 加密操作失败 |
| `HUKS_ERR_CODE_KEY_AUTH_PERMANENTLY_INVALIDATED` | 12000007 | 密钥认证永久失效 |
| `HUKS_ERR_CODE_KEY_AUTH_VERIFY_FAILED` | 12000008 | 密钥认证验证失败 |
| `HUKS_ERR_CODE_KEY_AUTH_TIME_OUT` | 12000009 | 密钥认证超时 |
| `HUKS_ERR_CODE_SESSION_LIMIT` | 12000010 | 会话数达到限制 |
| `HUKS_ERR_CODE_ITEM_NOT_EXIST` | 12000011 | 项目不存在 |
| `HUKS_ERR_CODE_EXTERNAL_ERROR` | 12000012 | 外部错误 |
| `HUKS_ERR_CODE_CREDENTIAL_NOT_EXIST` | 12000013 | 凭证不存在 |
| `HUKS_ERR_CODE_INSUFFICIENT_MEMORY` | 12000014 | 内存不足 |
| `HUKS_ERR_CODE_CALL_SERVICE_FAILED` | 12000015 | 调用服务失败 |
| `HUKS_ERR_CODE_DEVICE_PASSWORD_UNSET` | 12000016 | 设备密码未设置 |
| `HUKS_ERR_CODE_KEY_ALREADY_EXIST` | 12000017 | 密钥已存在 |
| `HUKS_ERR_CODE_INVALID_ARGUMENT` | 12000018 | 无效参数 |
| `HUKS_ERR_CODE_ITEM_EXISTS` | 12000019 | 项目已存在 |
| `HUKS_ERR_CODE_DEPENDENT_MODULES_ERROR` | 12000020 | 依赖模块错误 |
| `HUKS_ERR_CODE_PIN_LOCKED` | 12000021 | PIN 已锁定 |
| `HUKS_ERR_CODE_PIN_CODE_ERROR` | 12000022 | PIN 码错误 |
| `HUKS_ERR_CODE_PIN_NO_AUTH` | 12000023 | PIN 未认证 |
| `HUKS_ERR_CODE_BUSY` | 12000024 | 忙碌 |
| `HUKS_ERR_CODE_EXCEED_LIMIT` | 12000025 | 超出限制 |

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.h`

### 6.3 内部错误码

**证据**: `interfaces/inner_api/huks_standard/main/include/hks_error_code.h`

| 错误码 | 值 | 说明 |
|-------|-----|------|
| `HKS_ERROR_INVALID_ARGUMENT` | -1 | 无效参数 |
| `HKS_ERROR_NO_PERMISSION` | -5 | 无权限 |
| `HKS_ERROR_INVALID_ACCESS_TYPE` | -135 | 无效的访问类型 |
| `HKS_ERROR_NOT_SYSTEM_APP` | -140 | 非系统应用 |
| `HKS_ERROR_ACCESS_OTHER_USER_KEY` | -152 | 访问其他用户密钥 |
| `HKS_ERROR_INVALID_ACCESS_GROUP` | -183 | 无效的访问组 |
| `HKS_ERROR_INVALID_DEVELOPER_ID` | -184 | 无效的开发者 ID |

---

## 7. 异步模式

### 7.1 Promise 和 Callback

**支持模式**:
- **Promise 模式**: 返回 Promise 对象
- **Callback 模式**: 传入回调函数

**判断方式**: 根据是否传入 Callback 函数
- 不传 Callback → Promise 模式
- 传入 Callback → Callback 模式

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp`

### 7.2 异步工作队列

使用 `napi_queue_async_work()` 将异步任务加入工作队列。

**证据**: `interfaces/kits/napi/src/v9/huks_napi_common_item.cpp`

---

## 8. 调用链示例

### 8.1 generateKeyItem 调用链

```
JS: huks.generateKeyItem()
  ↓ NAPI
C++: HuksNapiItemGenerateKey()
  ↓ Client SDK
C: HksGenerateKey()
  ↓ IPC (Binder)
Service: HksIpcServiceGenerateKey()
  ↓ 权限检查
Service: HksCheckAccessible() + SensitivePermissionCheck()
  ↓ Core
Service: HksLocalGenerateKey()
  ↓ HAL
TEE: HAL 密钥生成
  ↓
Service: 存储密钥
  ↓
返回: Promise/callback
```

---

## 9. 相关文档

- [项目概览](./00_Overview.md) - HUKS 定位和核心能力
- [目录结构](./01_Directory_Structure.md) - 代码组织和模块职责
- [架构说明](./02_Architecture.md) - 架构设计和数据流
- [官方接口文档](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-universal-keystore-kit/Readme-CN.md) - OpenHarmony 官方 API 文档
