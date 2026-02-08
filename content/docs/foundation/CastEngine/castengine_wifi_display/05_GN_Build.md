# GN 构建配置

## 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口文件 |
| `config.gni` | 构建配置参数 |

## 根构建文件

**文件**: `BUILD.gn`

```gn
import("//build/ohos.gni")
import("//foundation/CastEngine/castengine_wifi_display/config.gni")

group("sharing_packages") {
  deps = [
    "//foundation/CastEngine/castengine_wifi_display/interfaces/innerkits/native/wfd:sharingwfd_client",
    "//foundation/CastEngine/castengine_wifi_display/interfaces/kits/js/wfd:sharingwfd_napi",
    "//foundation/CastEngine/castengine_wifi_display/sa_profile:sharing_sa_profile",
    "//foundation/CastEngine/castengine_wifi_display/services:sharing_services_package",
    "//foundation/CastEngine/castengine_wifi_display/services/etc:sharing_service.rc",
  ]
}
```

## 配置参数

**文件**: `config.gni`

```gn
SHARING_ROOT_DIR = "//foundation/CastEngine/castengine_wifi_display"

declare_args() {
  sharing_framework_support_wfd = true    # 支持 WFD
  wifi_display_support_sink = true       # 支持 Sink 端
  wifi_display_support_source = true     # 支持 Source 端
}
```

## Targets 清单

### 1. sharingwfd_napi（N-API 模块）

**BUILD.gn 位置**: `interfaces/kits/js/wfd/BUILD.gn`

```gn
ohos_shared_library("sharingwfd_napi") {
  install_enable = true

  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }

  include_dirs = [
    "$SHARING_ROOT_DIR",
    "$SHARING_ROOT_DIR/services/utils",
    "$SHARING_ROOT_DIR/interfaces/kits/js/wfd/include",
    "$SHARING_ROOT_DIR/interfaces/innerkits/native/wfd/include",
    "$SHARING_ROOT_DIR/frameworks/innerkitsimpl/native/wfd",
  ]

  sources = [
    "$SHARING_ROOT_DIR/frameworks/kitsimpl/js/wfd/native_module_ohos_wfd.cpp",
    "$SHARING_ROOT_DIR/frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp",
  ]

  public_configs = [
    ":sharing_service_config",
    "$SHARING_ROOT_DIR/tests:coverage_flags",
  ]

  deps = [
    "$SHARING_ROOT_DIR/interfaces/innerkits/native/wfd:sharingwfd_client",
    "$SHARING_ROOT_DIR/services/interaction/device_kit:dmkit",
    "$SHARING_ROOT_DIR/services/utils:sharing_utils",
  ]

  external_deps = [
    "ability_base:base",
    "ability_base:want",
    "ability_runtime:app_manager",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "c_utils:utils",
    "device_manager:devicemanagersdk",
    "graphic_surface:surface",
    "hilog:libhilog",
    "ipc:ipc_core",
    "napi:ace_napi",
    "safwk:system_ability_fwk",
  ]

  subsystem_name = "castplus"
  part_name = "sharing_framework"
}
```

**Target 类型**: `ohos_shared_library` (共享库)

**输出**: `libsharingwfd_napi.z.so`

### 2. sharing_service（服务模块）

**BUILD.gn 位置**: `services/BUILD.gn`

```gn
ohos_shared_library("sharing_service") {
  install_enable = true

  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    ubsan = true
    integer_overflow = true
  }

  deps = [
    "agent:sharing_agent_srcs",
    "configuration:sharing_configure_srcs",
    "context:sharing_context_srcs",
    "event:sharing_event_srcs",
    "interaction:sharing_interaction_srcs",
    "mediachannel:sharing_media_channel_srcs",
    "mediaplayer:sharing_media_player_srcs",
    "scheduler:sharing_scheduler_srcs",
  ]

  deps += [
    "codec:sharing_codec",
    "common:sharing_common",
    "network:sharing_network",
    "protocol/rtp:sharing_rtp",
    "protocol/rtsp:sharing_rtsp",
    "utils:sharing_utils",
  ]

  if (sharing_framework_support_wfd) {
    deps += [
      "impl/scene/wfd:sharing_wfd_srcs",
      "impl/screen_capture:sharing_screen_capture_srcs",
      "impl/wfd:sharing_wfd_session_srcs",
    ]
  }

  configs = [ "$SHARING_ROOT_DIR/tests:coverage_flags" ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libnativetoken",
    "access_token:libtoken_setproc",
    "audio_framework:audio_capturer",
    "audio_framework:audio_client",
    "audio_framework:audio_renderer",
    "cJSON:cjson_static",
    "graphic_2d:librender_service_base",
    "graphic_surface:surface",
    "hilog:libhilog",
    "samgr:samgr_proxy",
  ]

  subsystem_name = "castplus"
  part_name = "sharing_framework"
}
```

**Target 类型**: `ohos_shared_library` (共享库)

**输出**: `libsharing_service.z.so`

### 3. sharing_services_package（服务包）

**BUILD.gn 位置**: `services/BUILD.gn:17-27`

```gn
group("sharing_services_package") {
  deps = [
    ":sharing_service",
    "codec:sharing_codec",
    "common:kv_operator",
    "common:sharing_common",
    "network:sharing_network",
    "protocol/rtp:sharing_rtp",
    "utils:sharing_utils",
  ]
}
```

**Target 类型**: `group` (依赖组)

### 4. sa_profile（SA 配置）

**BUILD.gn 位置**: `sa_profile/BUILD.gn`

```gn
ohos_sa_profile("sharing_sa_profile") {
  sources = [
    "5527.json",
    "5528.json",
  ]
}
```

**Target 类型**: `ohos_sa_profile` (SA 配置)

### 5. sharing_service.rc（进程配置）

**BUILD.gn 位置**: `services/etc/BUILD.gn`

```gn
ohos_resource("sharing_service.rc") {
  sources = [ "sharing_service.cfg" ]
}
```

**Target 类型**: `ohos_resource` (资源)

## 编译安全配置

### CFI (Control Flow Integrity)

**位置**: `BUILD.gn:33-35`

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  ...
}
```

### 编译器标志

**位置**: `BUILD.gn:27-44`

```gn
config("wifi_display_default_config") {
  if (wifi_display_support_sink) {
    cflags = [
      "-Wall",
      "-Wextra",
      "-Werror",
      "-Wno-shadow",
      "-Wno-unused-parameter",
      "-Wno-missing-field-initializers",
      "-FS",
      "-O2",
      "-D_FORTIFY_SOURCE=2",
      "-fvisibility=hidden",
      "-fvisibility-inlines-hidden",
    ]
    cflags_cc = cflags
    ldflags = [ "-Werror" ]
  }
}
```

## Target ↔ 产物映射

| Target | 类型 | 输出 | 安装路径 |
|--------|------|------|----------|
| `sharingwfd_napi` | 共享库 | `libsharingwfd_napi.z.so` | `/system/lib64/module/` |
| `sharing_service` | 共享库 | `libsharing_service.z.so` | `/system/lib64/` |
| `sharing_sa_profile` | SA 配置 | 5527.json, 5528.json | `/system/profile/` |
| `sharing_service.rc` | 进程配置 | `sharing_service.cfg` | `/system/etc/` |

## 相关文档

- [编译产物](06_Build_Artifacts.md)
- [SA 配置](07_SA_Configuration.md)
