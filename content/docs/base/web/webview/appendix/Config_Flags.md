# 配置标志与宏

> 本文档记录 ArkWeb WebView 组件的关键配置宏、功能开关和编译选项。

## 编译配置宏

### 架构相关宏

| 宏定义 | 值 | 用途 |
|--------|-----|------|
| `webview_arm64` | 条件定义 | ARM64 架构编译 |
| `webview_arm` | 条件定义 | ARM 架构编译 |
| `webview_x86_64` | 条件定义 | x86_64 架构编译 |
| `WEBVIEW_PACKAGE_NAME` | `"com.ohos.arkwebcore"` | 包名 |
| `WEBVIEW_SANDBOX_LIB_PATH` | 路径字符串 | 沙箱库路径 |
| `LEGACY_WEBVIEW_SANDBOX_LIB_PATH` | 路径字符串 | 旧版沙箱路径 |
| `WEBVIEW_CRASHPAD_HANDLER_SO` | `"libarkweb_crashpad_handler.so"` | 崩溃处理库 |
| `WEBVIEW_ENGINE_SO` | `"libarkweb_engine.so"` | 引擎库 |
| `WEBVIEW_SANDBOX_RELATIVE_LIB_PATH` | 路径字符串 | 相对库路径 |

**证据**: `ohos_nweb/BUILD.gn:31-38`

```gn
defines += [
    "WEBVIEW_PACKAGE_NAME=\"${webview_package_name}\"",
    "WEBVIEW_SANDBOX_LIB_PATH=\"${webview_sandbox_lib_path}\"",
    "LEGACY_WEBVIEW_SANDBOX_LIB_PATH=\"${legacy_webview_sandbox_lib_path}\"",
    "WEBVIEW_CRASHPAD_HANDLER_SO=\"${webview_crashpad_handler_so}\"",
    "WEBVIEW_SANDBOX_RELATIVE_LIB_PATH=\"${webview_sandbox_relative_lib_path}\"",
    "WEBVIEW_ENGINE_SO=\"${webview_engine_so}\"",
]
```

### 渲染配置宏

| 宏定义 | 默认值 | 用途 |
|--------|--------|------|
| `MULTI_PROCESS_RENDER` | 启用 | 多进程渲染 |
| `SITE_ISOLATION` | 启用 | 站点隔离 |
| `ENABLE_HARDWARE_ACCELERATION` | 启用 | 硬件加速 |

### 调试配置宏

| 宏定义 | 用途 |
|--------|------|
| `IS_ASAN` | Address Sanitizer 模式 |
| `ENABLE_DEBUG_LOG` | 调试日志 |
| `ENABLE_VERBOSE_LOG` | 详细日志 |

## 功能开关 (config.gni)

### 媒体功能开关

| 开关变量 | 默认值 | 用途 |
|----------|--------|------|
| `webview_audio_enable` | `true` | 音频播放 |
| `webview_media_player_enable` | `true` | 媒体播放 |
| `webview_camera_enable` | `true` | 相机访问 |
| `webview_media_avsession_enable` | `true` | 媒体会话 |
| `webview_drm_enable` | `true` | DRM 支持 |

### 传感器功能开关

| 开关变量 | 默认值 | 用途 |
|----------|--------|------|
| `webview_sensors_sensor_enable` | `true` | 传感器访问 |
| `webview_location_enable` | `true` | 位置服务 |

### 系统功能开关

| 开关变量 | 默认值 | 用途 |
|----------|--------|------|
| `webview_soc_perf_enable` | `true` | SoC 性能管理 |
| `webview_battery_manager_enable` | `true` | 电池管理 |
| `webview_power_manager_enable` | `true` | 电源管理 |
| `webview_telephony_enable` | `true` | 电话功能 |
| `webview_print_enable` | `true` | 打印功能 |
| `webview_enterprise_device_manager_enable` | `true` | 企业设备管理 |

### 性能优化开关

| 开关变量 | 默认值 | 用途 |
|----------|--------|------|
| `webview_preload_render_lib` | `true` | 预加载渲染库 |
| `webview_enable_heif_decoder` | `false` | HEIF 解码器 |

**证据**: `config.gni:14-30`

```gn
declare_args() {
    webview_soc_perf_enable = true
    webview_audio_enable = true
    webview_location_enable = true
    webview_media_player_enable = true
    webview_camera_enable = true
    webview_telephony_enable = true
    webview_battery_manager_enable = true
    webview_power_manager_enable = true
    webview_avcodec_enable = true
    webview_print_enable = true
    webview_enterprise_device_manager_enable = true
    webview_media_avsession_enable = true
    webview_sensors_sensor_enable = true
    webview_enable_heif_decoder = false
    webview_drm_enable = true
    webview_preload_render_lib = true
}
```

## web_config.xml 配置

### 渲染配置

| 配置项 | 默认值 | 用途 |
|--------|--------|------|
| `renderConfig.renderProcessCount` | 20 | 渲染进程数 |
| `mediaConfig.backgroundMediaShouldSuspend` | `true` | 后台媒体暂停 |
| `touchEventConfig.touchEventShouldRegister` | `true` | 触摸事件注册 |

**证据**: `ohos_nweb/etc/web_config.xml`

```xml
<renderConfig>
    <renderProcessCount>20</renderProcessCount>
</renderConfig>

<mediaConfig>
    <backgroundMediaShouldSuspend>true</backgroundMediaShouldSuspend>
</mediaConfig>

<touchEventConfig>
    <touchEventShouldRegister>true</touchEventShouldRegister>
</touchEventConfig>
```

### 性能配置

| 配置项 | 默认值 | 用途 |
|--------|--------|------|
| `performanceConfig.LowerFrameRateConfig.visibleAreaRatio` | 0.1 | 降帧阈值 |
| `performanceConfig.LowerFrameRateConfig.visibleAreaRatioV2` | 0.3 | 降帧阈值 V2 |

### HTTP 缓存配置

| 配置项 | 默认值 | 用途 |
|--------|--------|------|
| `settingConfig.enableHttpCacheSimple` | `true` | 简单 HTTP 缓存 |
| `settingConfig.enableSetHttpCacheMaxSize` | `true` | 设置缓存最大大小 |

## 系统参数 (web.para)

### 运行时可调参数

| 参数名 | 类型 | 用途 |
|--------|------|------|
| `web.enable_javascript` | bool | JS 启用 |
| `web.enable_plugins` | bool | 插件启用 |
| `web.cache_size` | int | 缓存大小 |
| `web.log_level` | string | 日志级别 |

**证据**: `ohos_nweb/etc/para/web.para`

## N-API 模块注册标志

### 模块注册常量

| 常量 | 值 | 用途 |
|------|-----|------|
| `NAPI_MODULE_VERSION` | 1 | 模块版本 |
| `nm_modname` | `"web.webview"` | 模块名 |

**证据**: `interfaces/kits/napi/common/napi_webview_native_module.cpp`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_register_func = WebViewExport,
    .nm_modname = "web.webview",
};
```

## 错误码定义

### 业务错误码

| 错误码 | 常量名 | 用途 |
|--------|--------|------|
| 0 | `NO_ERROR` | 成功 |
| 401 | `PARAM_CHECK_ERROR` | 参数检查错误 |
| 17100001 | `INIT_ERROR` | 初始化错误 |
| 17100002 | `INVALID_URL` | 无效 URL |
| 17100003 | `MEMORY_ERROR` | 内存错误 |
| 17100004 | `UNSUPPORTED_OPERATION` | 不支持的操作 |
| 17100005 | `RENDER_PROCESS_GONE` | 渲染进程终止 |
| 17100006 | `WEBVIEW_NOT_ATTACHED` | WebView 未附加 |

### IPC 错误码

| 错误码 | 常量名 | 用途 |
|--------|--------|------|
| -1 | `PERMISSION_CHECK_ERROR` | 权限检查失败 |
| -2 | `CONTEXT_ERROR` | 上下文错误 |
| -3 | `WANT_FORMAT_ERROR` | Want 格式错误 |
| -4 | `CONNECTION_NOT_EXIST` | 连接不存在 |
| -5 | `MEMORY_ERROR` | 内存错误 |
| -7 | `IPC_ERROR` | IPC 错误 |

## 日志标签

### HILOG 标签

| 标签 | 模块 | 用途 |
|------|------|------|
| `ArkWeb` | 核心模块 | WebView 核心日志 |
| `WebAdapter` | 适配器 | 适配器日志 |
| `WebNativeMessaging` | 消息服务 | 原生消息日志 |

**证据**: `ohos_nweb/src/nweb_log.h`

```cpp
#define WVLOG_I(...) HILOG_INFO(OHOS_WEBCORE, __VA_ARGS__)
#define WVLOG_D(...) HILOG_DEBUG(OHOS_WEBCORE, __VA_ARGS__)
#define WVLOG_E(...) HILOG_ERROR(OHOS_WEBCORE, __VA_ARGS__)
#define WVLOG_W(...) HILOG_WARN(OHOS_WEBCORE, __VA_ARGS__)
```

## 相关文档

| 主题 | 文档 |
|------|------|
| 架构设计 | [01_Architecture.md](../01_Architecture.md) |
| 构建系统 | [04_Build.md](../04_Build.md) |

---

[返回 SUMMARY.md](../SUMMARY.md)
