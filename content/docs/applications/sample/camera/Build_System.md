# GN 构建系统

## 概述

本项目使用 **GN (Generate Ninja)** 构建系统，这是 OpenHarmony 的标准构建工具。

**构建特点**:
- 目标平台: OpenHarmony Lite (轻量级设备)
- 构建工具: `hb` (OpenHarmony Build)
- 输出格式: `.hap` (HarmonyOS Ability Package) + 独立可执行文件

---

## BUILD.gn 文件清单

| 文件路径 | 模块 | 说明 |
|----------|------|------|
| `cameraApp/BUILD.gn` | cameraApp | HAP 包构建配置 |
| `gallery/BUILD.gn` | gallery | HAP 包构建配置 |
| `launcher/BUILD.gn` | launcher | HAP 包构建配置 |
| `setting/BUILD.gn` | setting | HAP 包构建配置 |
| `media/BUILD.gn` | media | 独立可执行文件构建 |

---

## cameraApp 构建配置

**文件**: `cameraApp/BUILD.gn`

### shared_library("cameraApp")

构建相机应用共享库。

```gn
shared_library("cameraApp") {
  sources = [
    "cameraApp/src/main/cpp/camera_ability.cpp",
    "cameraApp/src/main/cpp/camera_ability_slice.cpp",
    "cameraApp/src/main/cpp/camera_manager.cpp",
  ]

  deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/distributeddatamgr/kv_store/interfaces/inner_api/kv_store:kv_store",
    "//foundation/graphic/graphic_utils_lite:utils_lite",
    "//foundation/graphic/surface_lite",
    "//foundation/multimedia/camera_lite/frameworks:camera_lite",
    "//foundation/multimedia/media_lite/frameworks/recorder_lite:recorder_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]

  include_dirs = [
    "cameraApp/src/main/cpp",
    "${aafwk_lite_path}/interfaces/kits/ability_lite",
    "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
    "${aafwk_lite_path}/interfaces/kits/want_lite",
    "//foundation/multimedia/camera_lite/interfaces/kits",
    "//foundation/multimedia/camera_lite/interfaces/kits",
  ]
  
  ldflags = [
    "-L$ohos_root_path/sysroot/usr/lib",
    "-Wl,-rpath-link=$ohos_root_path/sysroot/usr/lib",
    "-lstdc++",
    "-lcamera_lite",
    "-lsurface",
    "-lrecorder_lite",
  ]
  
  defines = [
    "ENABLE_WINDOW=1",
    "ABILITY_WINDOW_SUPPORT",
  ]
}
```

#### 配置项说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **sources** | 3 个 .cpp 文件 | 相机应用源文件 |
| **deps** | 9 个依赖 | 包括 Ability、UI、相机、录制等库 |
| **include_dirs** | 6 个路径 | 头文件搜索路径 |
| **ldflags** | 链接选项 | 指定库文件和运行时库路径 |
| **defines** | 2 个宏 | `ENABLE_WINDOW=1` 启用窗口支持 |

### hap_pack("cameraApp_hap")

打包 HAP 应用包。

```gn
hap_pack("cameraApp_hap") {
  deps = [ ":cameraApp" ]
  mode = "hap"
  json_path = "cameraApp/src/main/config.json"
  ability_so_path = "$root_out_dir/libcameraApp.so"
  force = "true"
  cert_profile = "cert/camera_AppProvision_Release.p7b"
  resources_path = "cameraApp/src/main/resources"
  hap_name = "cameraApp"
  privatekey = "HOS Application Provision Release"
}
```

#### HAP 打包参数

| 参数 | 值 | 说明 |
|------|-----|------|
| **mode** | "hap" | 打包模式 |
| **json_path** | config.json | 应用配置文件路径 |
| **ability_so_path** | libcameraApp.so | Native 库路径 |
| **cert_profile** | .p7b | 签名证书 |
| **resources_path** | resources/ | 资源文件目录 |
| **hap_name** | cameraApp | 输出 HAP 名称 |

---

## gallery 构建配置

**文件**: `gallery/BUILD.gn`

```gn
shared_library("gallery") {
  sources = [
    "src/gallery_ability.cpp",
    "src/gallery_ability_slice.cpp",
    "src/picture_ability_slice.cpp",
    "src/player_ability_slice.cpp",
  ]

  deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/distributeddatamgr/kv_store/interfaces/inner_api/kv_store:kv_store",
    "//foundation/graphic/graphic_utils_lite:utils_lite",
    "//foundation/graphic/surface_lite",
    "//foundation/multimedia/media_lite/frameworks/player_lite:player_lite",
    "//foundation/multimedia/media_lite/frameworks/recorder_lite:recorder_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]

  defines = [
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
    "ENABLE_WINDOW=1",
    "ABILITY_WINDOW_SUPPORT",
  ]
}

hap_pack("gallery_hap") {
  deps = [ ":gallery" ]
  mode = "hap"
  json_path = "config.json"
  ability_so_path = "$root_out_dir/libgallery.so"
  force = "true"
  cert_profile = "cert/gallery_AppProvision_Release.p7b"
  resources_path = "resources"
  hap_name = "gallery"
  privatekey = "HOS Application Provision Release"
}
```

---

## launcher 构建配置

**文件**: `launcher/BUILD.gn`

```gn
shared_library("launcher") {
  sources = [
    "launcher/src/main/cpp/app_info.cpp",
    "launcher/src/main/cpp/app_manage.cpp",
    "launcher/src/main/cpp/long_press_view.cpp",
    "launcher/src/main/cpp/main_ability.cpp",
    "launcher/src/main/cpp/main_ability_slice.cpp",
    "launcher/src/main/cpp/swipe_view.cpp",
    "launcher/src/main/cpp/time_weather_view.cpp",
    "launcher/src/main/cpp/view_group_page.cpp",
  ]

  deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/distributeddatamgr/kv_store/interfaces/inner_api/kv_store:kv_store",
    "//foundation/graphic/graphic_utils_lite:utils_lite",
    "//foundation/graphic/surface_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]

  defines = [
    "ENABLE_WINDOW=1",
    "ABILITY_WINDOW_SUPPORT",
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
  ]
}

hap_pack("launcher_hap") {
  deps = [ ":launcher" ]
  mode = "hap"
  json_path = "launcher/src/main/config.json"
  ability_so_path = "$root_out_dir/liblauncher.so"
  force = "true"
  cert_profile = "cert/com.huawei.launcher_AppProvision_release.p7b"
  resources_path = "launcher/src/main/resources"
  hap_name = "launcher"
  privatekey = "HOS Application Provision Release"
}
```

---

## setting 构建配置

**文件**: `setting/BUILD.gn`

```gn
shared_library("setting") {
  sources = [
    "setting/src/main/cpp/app_ability_slice.cpp",
    "setting/src/main/cpp/app_info_ability_slice.cpp",
    "setting/src/main/cpp/main_ability_slice.cpp",
    "setting/src/main/cpp/setting_about_ability_slice.cpp",
    "setting/src/main/cpp/setting_display_ability_slice.cpp",
    "setting/src/main/cpp/setting_main_ability.cpp",
    "setting/src/main/cpp/setting_utils.cpp",
    "setting/src/main/cpp/setting_wifi_ability_slice.cpp",
    "setting/src/main/cpp/setting_wifi_input_password_ability_slice.cpp",
    "setting/src/main/cpp/wpa_work.c",
  ]

  deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "//base/powermgr/powermgr_lite/frameworks:powermgr",
    "//base/startup/init/interfaces/innerkits:libbegetutil",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/distributeddatamgr/kv_store/interfaces/inner_api/kv_store:kv_store",
    "//foundation/graphic/graphic_utils_lite:utils_lite",
    "//foundation/graphic/surface_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
    "//third_party/wpa_supplicant/wpa_supplicant-2.9:wpa_supplicant",
  ]

  include_dirs = [
    "setting/src/main/cpp",
    "${aafwk_lite_path}/interfaces/kits/ability_lite",
    "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
    "${aafwk_lite_path}/interfaces/kits/want_lite",
    "//base/startup/init/interfaces/innerkits/include/syspara",
    "//base/security/permission_lite/interfaces/kits",
    "//third_party/wpa_supplicant/wpa_supplicant-2.9/src/common",
  ]

  ldflags = [
    "-lwpa",
    "-lwpa_client",
    "-lbegetutil",
    "-lpms_client",
  ]

  defines = [
    "ENABLE_WINDOW=1",
    "ABILITY_WINDOW_SUPPORT",
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
  ]
}

hap_pack("setting_hap") {
  deps = [ ":setting" ]
  mode = "hap"
  json_path = "setting/src/main/config.json"
  ability_so_path = "$root_out_dir/libsetting.so"
  force = "true"
  cert_profile = "cert/com.huawei.setting_AppProvision_release.p7b"
  resources_path = "setting/src/main/resources"
  hap_name = "setting"
  privatekey = "HOS Application Provision Release"
}
```

**特殊依赖**:
- `wpa_supplicant`: WiFi 协议栈
- `powermgr`: 电源管理
- `permission_lite`: 权限管理

---

## media 构建配置

**文件**: `media/BUILD.gn`

### executable("camera_sample")

```gn
executable("camera_sample") {
  sources = [ "camera_sample.cpp" ]
  cflags = [ "-Wall" ]
  cflags_cc = cflags

  ldflags = [ "-lstdc++" ]
  ldflags += [ "-lpthread" ]
  ldflags += [ "-Wl,-rpath-link=$ohos_root_path/$root_out_dir" ]

  deps = [
    "//foundation/multimedia/camera_lite/frameworks:camera_lite",
    "//foundation/multimedia/media_lite/frameworks/recorder_lite:recorder_lite",
  ]
  output_dir = "$root_out_dir/dev_tools"
}
```

### executable("audio_capture_sample")

```gn
executable("audio_capture_sample") {
  sources = [ "audio_capture_sample.cpp" ]
  
  include_dirs = [
    "//foundation/multimedia/audio_lite/interfaces/kits",
    "//foundation/multimedia/media_utils_lite/interfaces/kits",
  ]

  deps = [
    "//foundation/multimedia/audio_lite/frameworks:audio_capturer_lite",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
  output_dir = "$root_out_dir/dev_tools"
}
```

### executable("player_sample")

```gn
executable("player_sample") {
  if (enable_media_passthrough_mode == true) {
    defines = [ "ENABLE_PASSTHROUGH_SAMPLE" ]
  }
  sources = [ "player_sample.cpp" ]
  
  deps = [
    "//foundation/multimedia/media_lite/frameworks/player_lite:player_lite",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
  output_dir = "$root_out_dir/dev_tools"
}
```

### lite_component("media_sample")

```gn
lite_component("media_sample") {
  features = [
    ":camera_sample",
    ":player_sample",
    ":audio_capture_sample",
  ]
}
```

---

## 公共依赖分析

### 依赖矩阵

| 模块 | ability_lite | bundle_lite | ui_lite | surface_lite | camera_lite | recorder_lite | player_lite | permission_lite | samgr_lite |
|------|:------------:|:-----------:|:-------:|:------------:|:-----------:|:-------------:|:-----------:|:---------------:|:----------:|
| cameraApp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | - | - | ✓ |
| gallery | ✓ | ✓ | ✓ | ✓ | - | ✓ | ✓ | - | ✓ |
| launcher | ✓ | ✓ | ✓ | ✓ | - | - | - | - | ✓ |
| setting | ✓ | ✓ | ✓ | ✓ | - | - | - | ✓ | ✓ |
| media | - | - | - | - | ✓ | ✓ | ✓ | - | - |

### 关键依赖说明

| 依赖库 | 功能 | 被依赖模块 |
|--------|------|------------|
| `ability_lite` | Ability 框架 | cameraApp, gallery, launcher, setting |
| `bundle_lite` | 包管理 | cameraApp, gallery, launcher, setting |
| `ui_lite` | 图形界面 | cameraApp, gallery, launcher, setting |
| `surface_lite` | 图形缓冲区 | cameraApp, gallery, launcher, setting |
| `camera_lite` | 相机服务 | cameraApp, media |
| `recorder_lite` | 录像服务 | cameraApp, gallery, media |
| `player_lite` | 播放服务 | gallery, media |
| `permission_lite` | 权限管理 | setting |
| `samgr_lite` | 系统服务管理 | cameraApp, gallery, launcher, setting |

---

## 关键宏定义 (defines)

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `ENABLE_WINDOW=1` | 所有 BUILD.gn | 启用窗口系统支持 |
| `ABILITY_WINDOW_SUPPORT` | 所有 BUILD.gn | Ability 窗口支持 |
| `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` | gallery, launcher, setting | 包管理器支持 |
| `ENABLE_PASSTHROUGH_SAMPLE` | media/player_sample | 透传模式示例 |

---

## 构建命令

```bash
# 选择开发板
hb set

# 构建整个项目
hb build camera_lite

# 构建单个模块（示例）
hb build cameraApp_hap
hb build gallery_hap
hb build launcher_hap
hb build setting_hap
hb build media_sample
```

---

## 相关链接

- [编译产物](./Build_Outputs.md) - 构建输出说明
- [目录结构](./Structure.md) - 源码组织
- [安全风险分析](./Security.md) - 构建安全考虑
