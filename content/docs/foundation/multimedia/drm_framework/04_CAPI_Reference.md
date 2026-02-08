# C API 参考

> 本文档描述 libnative_drm.so 导出 Native C API 接口。

## 库信息

| 属性 | 值 |
|------|-----|
| **库名** | libnative_drm.so |
| **头文件目录** | interfaces/kits/c/drm_capi/ |
| **系统能力** | SystemCapability.Multimedia.Drm.Core |
| **API 版本** | since 11 / since 12 |

## 头文件清单

| 文件 | 说明 |
|------|------|
| `native_mediakeysystem.h` | MediaKeySystem API |
| `native_mediakeysession.h` | MediaKeySession API |
| `native_drm_common.h` | 公共数据结构、枚举 |
| `native_drm_err.h` | 错误码定义 |
| `native_drm_base.h` | 基类定义 |
| `native_drm_object.h` | 内部实现对象 |

## MediaKeySystem API

### 系统能力查询

```c
// 检查 DRM 方案支持 (since 11)
bool OH_MediaKeySystem_IsSupported(const char *name);

// 检查 DRM + MimeType 支持 (since 11)
bool OH_MediaKeySystem_IsSupported2(const char *name, const char *mimeType);

// 检查 DRM + MimeType + 安全级别支持 (since 11)
bool OH_MediaKeySystem_IsSupported3(const char *name, const char *mimeType,
    DRM_ContentProtectionLevel contentProtectionLevel);

// 获取所有 DRM 系统 (since 12)
Drm_ErrCode OH_MediaKeySystem_GetMediaKeySystems(
    DRM_MediaKeySystemDescription *descs, uint32_t *count);
```

### 实例管理

```c
// 创建 MediaKeySystem 实例 (since 11)
Drm_ErrCode OH_MediaKeySystem_Create(const char *name, MediaKeySystem **mediaKeySystem);

// 销毁实例 (since 11)
Drm_ErrCode OH_MediaKeySystem_Destroy(MediaKeySystem *mediaKeySystem);
```

### 配置管理

```c
// 设置字符串配置 (since 11)
Drm_ErrCode OH_MediaKeySystem_SetConfigurationString(MediaKeySystem *mediaKeySystem,
    const char *configName, const char *value);

// 获取字符串配置 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetConfigurationString(MediaKeySystem *mediaKeySystem,
    const char *configName, char *value, int32_t valueLen);

// 设置字节数组配置 (since 11)
Drm_ErrCode OH_MediaKeySystem_SetConfigurationByteArray(MediaKeySystem *mediaKeySystem,
    const char *configName, uint8_t *value, int32_t valueLen);

// 获取字节数组配置 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetConfigurationByteArray(MediaKeySystem *mediaKeySystem,
    const char *configName, uint8_t *value, int32_t *valueLen);
```

### 证书 Provision

```c
// 生成证书请求 (since 11)
Drm_ErrCode OH_MediaKeySystem_GenerateKeySystemRequest(MediaKeySystem *mediaKeySystem,
    uint8_t *request, int32_t *requestLen, char *defaultUrl, int32_t defaultUrlLen);

// 处理证书响应 (since 11)
Drm_ErrCode OH_MediaKeySystem_ProcessKeySystemResponse(MediaKeySystem *mediaKeySystem,
    uint8_t *response, int32_t responseLen);
```

### 密钥会话

```c
// 创建密钥会话 (since 11)
Drm_ErrCode OH_MediaKeySystem_CreateMediaKeySession(MediaKeySystem *mediaKeySystem,
    DRM_ContentProtectionLevel *level, MediaKeySession **mediaKeySession);
```

### 离线密钥管理

```c
// 获取离线密钥 ID 列表 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetOfflineMediaKeyIds(MediaKeySystem *mediaKeySystem,
    DRM_OfflineMediakeyIdArray *offlineMediaKeyIds);

// 获取离线密钥状态 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetOfflineMediaKeyStatus(MediaKeySystem *mediaKeySystem,
    uint8_t *offlineMediaKeyId, int32_t offlineMediaKeyIdLen,
    DRM_OfflineMediaKeyStatus *status);

// 清除离线密钥 (since 11)
Drm_ErrCode OH_MediaKeySystem_ClearOfflineMediaKeys(MediaKeySystem *mediaKeySystem,
    uint8_t *offlineMediaKeyId, int32_t offlineMediaKeyIdLen);
```

### 查询操作

```c
// 获取证书状态 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetCertificateStatus(MediaKeySystem *mediaKeySystem,
    DRM_CertificateStatus *certStatus);

// 获取最大内容保护级别 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetMaxContentProtectionLevel(MediaKeySystem *mediaKeySystem,
    DRM_ContentProtectionLevel *contentProtectionLevel);

// 获取统计信息 (since 11)
Drm_ErrCode OH_MediaKeySystem_GetStatistics(MediaKeySystem *mediaKeySystem,
    DRM_Statistics *statistics);
```

### 事件回调

```c
// 设置回调 (since 11, 旧版)
Drm_ErrCode OH_MediaKeySystem_SetMediaKeySystemCallback(MediaKeySystem *mediaKeySystem,
    MediaKeySystem_Callback callback);

// 设置回调 (since 12, 新版 - 含实例指针)
Drm_ErrCode OH_MediaKeySystem_SetCallback(MediaKeySystem *mediaKeySystem,
    OH_MediaKeySystem_Callback callback);
```

## MediaKeySession API

### 密钥请求处理

```c
// 生成密钥请求 (since 11)
Drm_ErrCode OH_MediaKeySession_GenerateMediaKeyRequest(MediaKeySession *mediaKeySession,
    DRM_MediaKeyRequestInfo *info, DRM_MediaKeyRequest *mediaKeyRequest);

// 处理密钥响应 (since 11)
Drm_ErrCode OH_MediaKeySession_ProcessMediaKeyResponse(MediaKeySession *mediaKeySession,
    uint8_t *response, int32_t responseLen,
    uint8_t *offlineMediaKeyId, int32_t *offlineMediaKeyIdLen);
```

### 密钥状态

```c
// 检查密钥状态 (since 11)
Drm_ErrCode OH_MediaKeySession_CheckMediaKeyStatus(MediaKeySession *mediaKeySession,
    DRM_MediaKeyStatus *mediaKeyStatus);

// 清除密钥 (since 11)
Drm_ErrCode OH_MediaKeySession_ClearMediaKeys(MediaKeySession *mediaKeySession);

// 获取保护级别 (since 11)
Drm_ErrCode OH_MediaKeySession_GetContentProtectionLevel(MediaKeySession *mediaKeySession,
    DRM_ContentProtectionLevel *contentProtectionLevel);
```

### 安全解码器

```c
// 检查是否需要安全解码器 (since 11)
Drm_ErrCode OH_MediaKeySession_RequireSecureDecoderModule(MediaKeySession *mediaKeySession,
    const char *mimeType, bool *status);
```

### 离线密钥释放

```c
// 生成离线释放请求 (since 11)
Drm_ErrCode OH_MediaKeySession_GenerateOfflineReleaseRequest(MediaKeySession *mediaKeySession,
    uint8_t *offlineMediaKeyId, int32_t offlineMediaKeyIdLen,
    uint8_t *releaseRequest, int32_t *releaseRequestLen);

// 处理离线释放响应 (since 11)
Drm_ErrCode OH_MediaKeySession_ProcessOfflineReleaseResponse(MediaKeySession *mediaKeySession,
    uint8_t *offlineMediaKeyId, int32_t offlineMediaKeyIdLen,
    uint8_t *releaseReponse, int32_t releaseReponseLen);

// 恢复离线密钥 (since 11)
Drm_ErrCode OH_MediaKeySession_RestoreOfflineMediaKeys(MediaKeySession *mediaKeySession,
    uint8_t *offlineMediaKeyId, int32_t offlineMediaKeyIdLen);
```

### 事件回调

```c
// 设置回调 (since 11, 旧版)
Drm_ErrCode OH_MediaKeySession_SetMediaKeySessionCallback(MediaKeySession *mediaKeySession,
    MediaKeySession_Callback *callback);

// 设置回调 (since 12, 新版)
Drm_ErrCode OH_MediaKeySession_SetCallback(MediaKeySession *mediaKeySession,
    OH_MediaKeySession_Callback *callback);

// 销毁会话 (since 11)
Drm_ErrCode OH_MediaKeySession_Destroy(MediaKeySession *mediaKeySession);
```

## 回调类型定义

### MediaKeySystem 回调

```c
// 旧版 (since 11)
typedef Drm_ErrCode (*MediaKeySystem_Callback)(
    DRM_EventType eventType, uint8_t *info,
    int32_t infoLen, char *extra);

// 新版 (since 12) - 包含实例指针
typedef Drm_ErrCode (*OH_MediaKeySystem_Callback)(
    MediaKeySystem *mediaKeySystem, DRM_EventType eventType,
    uint8_t *info, int32_t infoLen, char *extra);
```

### MediaKeySession 回调

```c
// 旧版 (since 11)
typedef struct MediaKeySession_Callback {
    MediaKeySession_EventCallback eventCallback;     // 事件回调
    MediaKeySession_KeyChangeCallback keyChangeCallback; // 密钥变更回调
} MediaKeySession_Callback;

// 新版 (since 12)
typedef struct OH_MediaKeySession_Callback {
    OH_MediaKeySession_EventCallback eventCallback;
    OH_MediaKeySession_KeyChangeCallback keyChangeCallback;
} OH_MediaKeySession_Callback;
```

## 枚举类型

### DRM_EventType

| 值 | 说明 |
|---|------|
| `EVENT_PROVISION_REQUIRED` | 需要 Provision |
| `EVENT_KEY_REQUIRED` | 需要密钥 |
| `EVENT_KEY_EXPIRED` | 密钥过期 |
| `EVENT_VENDOR_DEFINED` | 厂商自定义 |
| `EVENT_EXPIRATION_UPDATE` | 过期时间更新 |

### DRM_ContentProtectionLevel

| 值 | 说明 |
|---|------|
| `CONTENT_PROTECTION_LEVEL_UNKNOWN` | 未知 |
| `CONTENT_PROTECTION_LEVEL_SW_CRYPTO` | 软件加密 |
| `CONTENT_PROTECTION_LEVEL_HW_CRYPTO` | 硬件加密 |
| `CONTENT_PROTECTION_LEVEL_ENHANCED_HW_CRYPTO` | 增强硬件加密 |

## 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `DRM_ERR_OK` | 0 | 成功 |
| `DRM_ERR_NO_MEMORY` | 24700501 | 内存错误 |
| `DRM_ERR_INVALID_VAL` | 24700503 | 无效参数 |
| `DRM_ERR_IO` | 24700504 | IO 错误 |
| `DRM_ERR_TIMEOUT` | 24700505 | 超时 |
| `DRM_ERR_UNKNOWN` | 24700506 | 未知错误 |
| `DRM_ERR_SERVICE_DIED` | 24700507 | 服务死亡 |
| `DRM_ERR_INVALID_STATE` | 24700508 | 无效状态 |
| `DRM_ERR_UNSUPPORTED` | 24700509 | 不支持 |
| `DRM_ERR_MAX_SYSTEM_NUM_REACHED` | 24700510 | 系统数达上限 |
| `DRM_ERR_MAX_SESSION_NUM_REACHED` | 24700511 | 会话数达上限 |

## 相关文档

- [JS N-API 参考](03_NAPI_Reference.md) - JS API
- [架构设计](05_Architecture.md) - 调用链
- [安全评审](08_Security_Review.md) - API 安全考量
