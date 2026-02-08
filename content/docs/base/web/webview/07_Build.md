# ArkWeb WebView 构建系统

## 1. 构建系统概述

ArkWeb WebView 使用 **GN (Generate Ninja)** 构建系统，这是 Chromium 和 OpenHarmony 标准的构建工具。

### 1.1 核心构建文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 定义构建目标（每个模块）|
| `bundle.json` | 组件元数据（版本、依赖、构建目标）|
| `config.gni` | 全局编译选项和 Feature Flags |
| `web_aafwk.gni` | AAFWK 相关路径定义 |

---

## 2. 主要 BUILD.gn 文件

### 2.1 核心模块

| BUILD.gn | 输出目标 | 产物 |
|---------|---------|------|
| `ohos_nweb/BUILD.gn` | `libnweb` | `arkweb_core_loader.so` |
| `ohos_adapter/BUILD.gn` | `nweb_ohos_adapter` | `libnweb_ohos_adapter.so` |
| `ohos_glue/BUILD.gn` | `ohos_base_glue_source` | `libohos_base_glue_source.z.so` |
| `ohos_interface/BUILD.gn` | (header actions) | 接口头文件生成 |

### 2.2 接口层

| BUILD.gn | 输出目标 | 产物 |
|---------|---------|------|
| `interfaces/kits/napi/BUILD.gn` | `webview_napi` | `libwebview_napi.so` |
| `interfaces/kits/ani/BUILD.gn` | `webview_ani` | `libwebview_ani.so` |
| `interfaces/kits/cj/BUILD.gn` | `cj_webview_ffi` | `libcj_webview_ffi.so` |
| `interfaces/native/BUILD.gn` | `ohweb` | `libohweb.so` |

### 2.3 系统服务

| BUILD.gn | 输出目标 | 产物 |
|---------|---------|------|
| `sa/web_native_messaging/BUILD.gn` | `web_native_messaging_service` | `libweb_native_messaging_service.z.so` |
| `sa/app_fwk_update/BUILD.gn` | `app_fwk_update_service` | `libapp_fwk_update_service.z.so` |

---

## 3. Feature Flags (config.gni)

### 3.1 功能开关

```gni
# config.gni
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
  webview_enable_heif_decoder = false      # 默认禁用
  webview_drm_enable = true
  webview_preload_render_lib = true
}
```

### 3.2 自动禁用逻辑

```gni
# 如果系统未包含某些组件，自动禁用对应功能
if (defined(global_parts_info) &&
    !defined(global_parts_info.multimedia_audio_framework)) {
  webview_audio_enable = false
}
```

### 3.3 路径配置

```gni
webview_package_name = "com.ohos.arkwebcore"
webview_hap_path = "/module_update/ArkWebCore/app/${webview_package_name}/ArkWebCore.hap"
webview_sandbox_path = "/data/storage/el1/bundle/arkwebcore/"
webview_engine_so = "libarkweb_engine.so"
webview_crashpad_handler_so = "libarkweb_crashpad_handler.so"
```

---

## 4. 编译产物清单

### 4.1 共享库

| 产物名称 | 路径 | 说明 |
|---------|------|------|
| `arkweb_core_loader.so` | `/system/lib/` | WebView 核心加载器 |
| `libnweb_ohos_adapter.so` | `/system/lib/` | 平台适配器库 |
| `libwebview_napi.so` | `/system/lib/module/web/` | N-API 绑定 |
| `libohweb.so` | `/system/lib/` | NDK 接口库 |
| `libarkweb_utils.so` | `/system/lib/` | 工具库 |

### 4.2 系统服务库

| 产物名称 | 路径 | 说明 |
|---------|------|------|
| `libweb_native_messaging_service.z.so` | `/system/lib/` | SA 8610 服务 |
| `libapp_fwk_update_service.z.so` | `/system/lib/` | SA 8350 服务 |

### 4.3 预编译包

| 产物名称 | 路径 | 说明 |
|---------|------|------|
| `ArkWebCore.hap` | `/system/app/com.ohos.arkwebcore/` | Chromium 引擎包 |

### 4.4 配置文件

| 产物名称 | 路径 | 说明 |
|---------|------|------|
| `web_config.xml` | `/system/etc/web/` | WebView 配置 |
| `web.para` | `/system/etc/param/` | 系统参数 |
| `web.para.dac` | `/system/etc/param/` | DAC 权限配置 |

---

## 5. 构建命令示例

### 5.1 完整构建

```bash
# 构建整个 WebView 组件
./build.sh --product-name <product> \
    --build-target //base/web/webview/...
```

### 5.2 单模块构建

```bash
# 构建核心引擎
./build.sh --product-name <product> \
    --build-target //base/web/webview/ohos_nweb:libnweb

# 构建 N-API 接口
./build.sh --product-name <product> \
    --build-target //base/web/webview/interfaces/kits/napi:webview_napi

# 构建 NDK 接口
./build.sh --product-name <product> \
    --build-target //base/web/webview/interfaces/native:ohweb

# 构建 SA 8610
./build.sh --product-name <product> \
    --build-target //base/web/webview/sa/web_native_messaging:web_native_messaging_service
```

### 5.3 测试构建

```bash
# 构建单元测试
./build.sh --product-name <product> \
    --build-target //base/web/webview/test/unittest/...

# 构建 Fuzz 测试
./build.sh --product-name <product> \
    --build-target //base/web/webview/test/fuzztest/...
```

---

## 6. 依赖关系

### 6.1 模块依赖图

```
libnweb.so (ohos_nweb)
    ├── ohos_glue:ohos_adapter_glue_source
    ├── ohos_glue:ohos_base_glue_source
    ├── ohos_glue:ohos_nweb_glue_source
    ├── sa/app_fwk_update:app_fwk_update
    ├── arkweb_utils:libarkweb_utils
    └── external: [ability_runtime, bundle_framework, graphic_2d, ...]

libwebview_napi.so (interfaces/kits/napi)
    ├── ohos_nweb:libnweb
    ├── ohos_adapter:nweb_ohos_adapter
    ├── interfaces/kits/nativecommon:webview_common
    ├── interfaces/native:ohweb
    └── external: [napi, hilog, ...]
```

### 6.2 外部依赖

```gn
# ohos_nweb:libnweb 的外部依赖
external_deps = [
    "ability_runtime:ability_manager",
    "ability_runtime:app_context",
    "bundle_framework:appexecfwk_base",
    "graphic_2d:libcomposer",
    "graphic_surface:surface",
    "hilog:libhilog",
    "ipc:ipc_core",
    "window_manager:libwm",
    # ... 共 15+ 个
]
```

---

## 7. 编译选项

### 7.1 架构相关

```gn
if (target_cpu == "arm64") {
  branch_protector_ret = "pac_ret"  # ARM64 分支保护
  defines = [ "webview_arm64" ]
} else if (target_cpu == "arm") {
  defines = [ "webview_arm" ]
} else if (target_cpu == "x86_64") {
  defines = [ "webview_x86_64" ]
}
```

### 7.2 安全编译选项

```gn
cflags = [
  "-Wall",
  "-Werror",
  "-g3",
]

if (use_hwasan) {
  defines += [
    "IS_ASAN",
    "WEBVIEW_SANDBOX_LIB_PATH_ASAN=\"${asan_webview_sandbox_lib_path}\"",
  ]
}
```

---

## 8. 构建产物分析

### 8.1 产物大小

| 产物 | 大致大小 | 说明 |
|------|---------|------|
| `ArkWebCore.hap` | ~80MB | 预编译 Chromium 引擎 |
| `arkweb_core_loader.so` | ~500KB | 核心加载器 |
| `libnweb_ohos_adapter.so` | ~2MB | 适配器库 |
| `libwebview_napi.so` | ~1MB | N-API 绑定 |

---

*文档版本: 1.0*  
*更新日期: 2026-02-07*
