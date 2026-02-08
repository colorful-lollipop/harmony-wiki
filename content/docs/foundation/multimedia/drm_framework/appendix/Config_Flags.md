# 配置开关与宏

> 本文档描述 DRM Framework 中的关键配置开关、编译宏和 feature flags。

## 编译宏

### DRM_CAPI_IMPL

**定义位置**: `frameworks/c/drm_capi/BUILD.gn`

```python
defines = [ "DRM_CAPI_IMPL" ]
```

**用途**: 标记 C API 实现文件，区分头文件声明与实现

### 内置宏

| 宏 | 定义位置 | 用途 |
|-----|----------|------|
| DRM_INNER_ERR_BASE | drm_error_code.h | 内部错误码基值 |
| DRM_CAPI_ERR_BASE | native_drm_err.h | CAPI 错误码基值 |
| MAX_MEDIA_KEY_REQUEST_DATA_LEN | native_drm_common.h | 密钥请求数据最大长度 |
| MAX_INIT_DATA_LEN | native_drm_common.h | 初始化数据最大长度 |
| DRM_UUID_LEN | native_drm_common.h | DRM UUID 长度 |

## Feature Flags

### 构建配置

**文件**: `bundle.json`

```json
"features": [
    "drm_framework_service_support_lazy_loading"
]
```

| 特性 | 说明 |
|------|------|
| drm_framework_service_support_lazy_loading | DRM 服务支持懒加载模式 |

## 错误码基值

### 内部错误码 (DRM Service)

| 范围 | 说明 |
|------|------|
| DRM_INNER_ERR_BASE ~ DRM_INNER_ERR_BASE + 100 | DRM Service 内部错误 |

### C API 错误码

| 范围 | 说明 |
|------|------|
| DRM_CAPI_ERR_BASE (24700500) | C API 错误码基值 |

## 服务配置

### 常驻模式

**文件**: `services/etc/resident/drm_service.cfg`

```json
{
    "name": "drm_service",
    "path": "drm_service",
    "systemability": [
        3012
    ],
    "run-on-create": true,
    "permission": [...]
}
```

### 懒加载模式

**文件**: `services/etc/lazy_loading/drm_service.cfg`

```json
{
    "name": "drm_service",
    "path": "drm_service", 
    "systemability": [
        3012
    ],
    "run-on-create": false,
    "start-mode": "condition",
    "condition": "connectivity",
    "permission": [...]
}
```

## SA 能力配置

**文件**: `sa_profile/resident/3012.json`

```json
{
    "process": "sandboxed_process3012",
    "allow-privilege-process": true
}
```

## 调试配置

### 日志级别

| 级别 | 宏 | 说明 |
|------|-----|------|
| DEBUG | DRM_DEBUG_LOG | 调试日志 |
| INFO | DRM_INFO_LOG | 信息日志 |
| WARN | DRM_WARN_LOG | 警告日志 |
| ERROR | DRM_ERR_LOG | 错误日志 |

### 追踪开关

| 追踪点 | 说明 |
|--------|------|
| MediaKeySystem 生命周期 | Create/Destroy |
| KeySession 生命周期 | Create/Close |
| 证书 Provision | Request/Response |
| 许可证处理 | Request/Response |
| 解密操作 | Start/Complete |
