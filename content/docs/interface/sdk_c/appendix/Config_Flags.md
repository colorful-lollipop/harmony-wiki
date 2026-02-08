# 附录：关键配置项

> **目的**: 汇总关键宏、配置项和常量定义  
> **生成时间**: 2025-02-06

---

## 1. N-API 版本常量

| 常量 | 值 | 说明 | 位置 |
|------|-----|------|------|
| `NAPI_VERSION` | 8 | N-API 版本 | `arkui/napi/native_api.h:42` |
| `NAPI_MODULE_VERSION` | 1 | 模块版本 | `third_party/node/src/node_api.h:38` |

---

## 2. N-API 属性特性

| 常量 | 值 | 说明 | 位置 |
|------|-----|------|------|
| `napi_default` | 0 | 默认属性 | `third_party/node/src/js_native_api_types.h:26` |
| `napi_writable` | 1 | 可写 | - |
| `napi_enumerable` | 2 | 可枚举 | - |
| `napi_configurable` | 4 | 可配置 | - |
| `napi_static` | 1024 | 静态属性 | - |
| `napi_default_method` | 1｜4 | 默认方法特性 | - |
| `napi_default_jsproperty` | 1｜2｜4 | 默认 JS 属性 | - |

---

## 3. HUKS 安全常量

| 常量 | 值 | 说明 | 位置 |
|------|-----|------|------|
| `OH_HUKS_MAX_KEY_SIZE` | 2048 | 最大密钥长度 | `security/huks/include/native_huks_type.h:91` |
| `OH_HUKS_MAX_KEY_ALIAS_LEN` | 64 | 密钥别名最大长度 | - |
| `OH_HUKS_MAX_PROCESS_NAME_LEN` | 50 | 进程名最大长度 | - |
| `OH_HUKS_MAX_RANDOM_LEN` | 1024 | 随机数最大长度 | - |
| `OH_HUKS_AE_TAG_LEN` | 16 | AEAD 认证标签长度 | - |
| `OH_HUKS_AE_NONCE_LEN` | 12 | AEAD nonce 长度 | - |
| `OH_HUKS_MAX_OUT_BLOB_SIZE` | 5242880 | 导出数据最大 5MB | - |

---

## 4. 证书管理常量

| 常量 | 值 | 说明 | 位置 |
|------|-----|------|------|
| `OH_CM_MAX_LEN_CERTIFICATE_CHAIN` | 24588 | 证书链最大长度 | `security/device_certificate/certmanager/include/cm_native_type.h` |
| `OH_CM_MAX_LEN_URI` | 256 | URI 最大长度 | - |
| `OH_CM_MAX_LEN_CERT_ALIAS` | 129 | 证书别名最大长度 | - |

---

## 5. 通用错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` / `OK` | 0 | 成功 |
| `PERMISSION_DENIED` | 201 | 权限拒绝 |
| `INVALID_PARAM` | 401 | 非法参数 |
| `NOT_SUPPORTED` | 801 | 不支持的功能 |

---

## 6. IPC 错误码

| 错误码 | 值 | 说明 | 位置 |
|--------|-----|------|------|
| `OH_IPC_SUCCESS` | 0 | 成功 | `IPCKit/ipc_error_code.h` |
| `OH_IPC_CHECK_PARAM_ERROR` | 1901001 | 参数错误 | - |
| `OH_IPC_MEM_ALLOC_ERROR` | 1901002 | 内存分配错误 | - |
| `OH_IPC_WRITE_TO_PARCEL_ERROR` | 1901003 | 写入 Parcel 错误 | - |
| `OH_IPC_REMOTE_OBJECT_NULL_ERROR` | 1901004 | 远程对象为空 | - |
| `OH_IPC_DEAD_REMOTE_OBJECT_ERROR` | 1901005 | 远程对象死亡 | - |

---

## 7. 多媒体常量

| 常量 | 说明 | 位置 |
|------|------|------|
| `AV_MAX_STRING_LEN` | 字符串最大长度 | `multimedia/av_codec/*.h` |
| `MAX_SAMPLE_RATE` | 最大采样率 | `multimedia/audio_framework/*.h` |
| `MAX_CHANNEL_COUNT` | 最大通道数 | - |

---

**配置项附录 - 基于代码生成**
