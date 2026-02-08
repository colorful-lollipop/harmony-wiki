# GN Targets 与编译产物

## 目的

本文档详细梳理 Window Manager 子系统的 GN 构建目标、产物类型、依赖关系和输出路径。

## 构建系统概述

Window Manager 使用 **GN (Generate Ninja)** 构建系统，主要配置文件：
- `windowmanager_aafwk.gni` - 构建变量和特性开关
- `scene_board_enable.gni` - Scene Board 架构开关
- `bundle.json` - 组件配置和构建分组
- 各模块 `BUILD.gn` - 具体构建规则

## 核心构建目标

### 1. Window Manager Client 库

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `libwm` | shared_library | libwm.so | `wm/BUILD.gn:198` |
| `libwm_static` | static_library | libwm_static.a | `wm/BUILD.gn:40` |
| `libwm_lite` | shared_library | libwm_lite.so | `wm/BUILD.gn:366` |
| `libwm_ndk` | shared_library | libnative_window_manager.so | `wm/BUILD.gn:448` |

**libwm 依赖关系**:
```gn
deps = [
  "//foundation/window/window_manager/utils:libwmutil",
  "//foundation/window/window_manager/utils:libwmutil_base",
  "//foundation/window/window_manager/window_scene/common:window_scene_common",
  "//foundation/window/window_manager/window_scene/interfaces/innerkits:libwsutils",
  "//foundation/window/window_manager/window_scene/screen_session_manager_client:screen_session_manager_client",
  "//foundation/window/window_manager/window_scene/session:scene_session",
  "//foundation/window/window_manager/window_scene/session_manager:session_manager",
  "../dm:libdm",
]
```

### 2. Display Manager Client 库

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `libdm` | shared_library | libdm.so | `dm/BUILD.gn:102` |
| `libdm_static` | static_library | libdm_static.a | `dm/BUILD.gn:41` |
| `libdm_ndk` | shared_library | libnative_display_manager.so | `dm/BUILD.gn:175` |

### 3. Window Manager Server

| Target | 类型 | 输出 | 路径 | 条件 |
|--------|------|------|------|------|
| `libwms` | shared_library | libwms.so | `wmserver/BUILD.gn:204` | 非 Scene Board |
| `sms` | shared_library | sms.z.so | `wmserver/BUILD.gn:131` | 始终构建 |

**条件编译** (`wmserver/BUILD.gn:199-346`):
```gn
if (window_manager_use_sceneboard) {
  group("libwms") {
    deps = [ "../etc:wms_etc" ]
  }
} else {
  ohos_shared_library("libwms") {
    # 传统 WMS 实现
    sources = [ ... ]
    deps = [ ... ]
  }
}
```

### 4. Display Manager Server

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `libdms` | shared_library | libdms.so | `dmserver/BUILD.gn:52` |

### 5. Window Scene 组件 (Scene Board 架构)

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `scene_session` | shared_library | libscene_session.z.so | `window_scene/session/BUILD.gn` |
| `screen_session` | shared_library | libscreen_session.z.so | `window_scene/session/BUILD.gn` |
| `scene_session_manager` | shared_library | libscene_session_manager.z.so | `window_scene/session_manager/BUILD.gn` |
| `session_manager` | shared_library | libsession_manager.z.so | `window_scene/session_manager/BUILD.gn` |
| `session_manager_lite` | shared_library | libsession_manager_lite.z.so | `window_scene/session_manager/BUILD.gn` |
| `screen_session_manager` | shared_library | libscreen_session_manager.z.so | `window_scene/screen_session_manager/BUILD.gn` |
| `screen_session_manager_client` | shared_library | libscreen_session_manager_client.z.so | `window_scene/screen_session_manager_client/BUILD.gn` |
| `session_manager_service` | shared_library | libsession_manager_service.z.so | `window_scene/session_manager_service/BUILD.gn` |
| `window_scene_common` | shared_library | libwindow_scene_common.so | `window_scene/common/BUILD.gn` |
| `libwsutils` | shared_library | libwsutils.so | `window_scene/interfaces/innerkits/BUILD.gn` |

### 6. 工具库

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `libwmutil` | shared_library | libwmutil.so | `utils/BUILD.gn` |
| `libwmutil_base` | shared_library | libwmutil_base.so | `utils/BUILD.gn` |
| `libwmutil_static` | static_library | libwmutil_static.a | `utils/BUILD.gn` |
| `libedid_parse` | shared_library | libedid_parse.so | `edidparse/BUILD.gn` |

### 7. N-API 模块

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `windowstage` | group | - | `interfaces/kits/napi/BUILD.gn` |
| `window_napi` | shared_library | window_napi.so | `interfaces/kits/napi/window_runtime/BUILD.gn` |
| `display_napi` | shared_library | display_napi.so | `interfaces/kits/napi/display_runtime/BUILD.gn` |
| `screen_napi` | shared_library | screen_napi.so | `interfaces/kits/napi/screen_runtime/BUILD.gn` |
| `screenshot` | shared_library | screenshot.so | `interfaces/kits/napi/screenshot/BUILD.gn` |
| `pipwindow_napi` | shared_library | pipwindow_napi.so | `interfaces/kits/napi/picture_in_picture_napi/BUILD.gn` |
| `floatingball_napi` | shared_library | floatingball_napi.so | `interfaces/kits/napi/floating_ball_napi/BUILD.gn` |
| `extensionwindow_napi` | shared_library | extensionwindow_napi.so | `interfaces/kits/napi/extension_window/BUILD.gn` |
| `scene_session_manager_napi` | shared_library | - | `window_scene/interfaces/kits/napi/scene_session_manager/BUILD.gn` |
| `screen_session_manager_napi` | shared_library | - | `window_scene/interfaces/kits/napi/screen_session_manager/BUILD.gn` |
| `transaction_manager_napi` | shared_library | - | `window_scene/interfaces/kits/napi/transaction_manager/BUILD.gn` |
| `session_manager_service_napi` | shared_library | - | `window_scene/interfaces/kits/napi/session_manager_service/BUILD.gn` |

### 8. 扩展模块

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `libwindow_extension` | shared_library | libwindow_extension.so | `extension/window_extension/BUILD.gn` |
| `libwindow_extension_client` | shared_library | libwindow_extension_client.so | `extension/extension_connection/BUILD.gn` |
| `libmodal_system_ui_extension_client` | shared_library | libmodal_system_ui_extension_client.so | `extension/modal_system_ui_extension/BUILD.gn` |

### 9. 可执行文件

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `snapshot_display` | executable | snapshot_display | `snapshot/BUILD.gn` |
| `setresolution_screen` | executable | setresolution_screen | `setresolution/BUILD.gn` |

## 构建分组

### bundle.json 中的分组

```json
{
  "build": {
    "group_type": {
      "base_group": [
        "//foundation/window/window_manager/snapshot:snapshot_display",
        "//foundation/window/window_manager/setresolution:setresolution_screen",
        "//foundation/window/window_manager/interfaces/kits/napi:napi_packages",
        "//foundation/window/window_manager/interfaces/kits/ani:ani_packages",
        "//foundation/window/window_manager/window_scene/interfaces/kits/napi:window_scene_napi_packages"
      ],
      "fwk_group": [
        "//foundation/window/window_manager/dm:libdm",
        "//foundation/window/window_manager/wm:libwm",
        "//foundation/window/window_manager/wm:libwm_lite",
        "//foundation/window/window_manager/utils:libwmutil",
        "//foundation/window/window_manager/extension/window_extension:libwindow_extension"
      ],
      "service_group": [
        "//foundation/window/window_manager/sa_profile:wms_sa_profile",
        "//foundation/window/window_manager/dmserver:libdms",
        "//foundation/window/window_manager/wmserver:libwms"
      ]
    }
  }
}
```

## 特性开关 (Feature Flags)

### 来自 windowmanager_aafwk.gni

| Flag | 默认值 | 说明 |
|------|--------|------|
| `window_manager_use_sceneboard` | true | 使用 Scene Board 架构 |
| `window_manager_feature_subscribe_motion` | false | 运动传感器订阅 |
| `window_manager_feature_tp_enable` | false | TP 功能 |
| `window_manager_fold_ability` | true | 折叠屏支持 |
| `window_manager_feature_screen_active_mode` | true | 屏幕主动模式 |
| `window_manager_feature_screen_color_gamut` | true | 色域支持 |
| `window_manager_feature_screen_hdr_format` | true | HDR 格式 |
| `window_manager_feature_screen_color_space` | true | 色彩空间 |
| `window_manager_feature_multi_screen` | true | 多屏幕支持 |
| `window_manager_feature_multi_screen_frame_ctl` | true | 多屏幕帧控制 |
| `window_manager_feature_cam_mode` | true | 相机模式 |
| `window_manager_feature_multi_usr` | true | 多用户支持 |
| `window_manager_feature_screenless` | false | 无屏幕模式 |
| `window_manager_feature_support_dsoftbus` | true | 分布式软总线 |
| `window_manager_feature_support_dmsfwk` | true | 分布式框架 |
| `device_status_enable` | true | 设备状态 |

### BUILD.gn 中的条件编译

```gn
# 示例：wmserver/BUILD.gn
if (window_manager_use_sceneboard) {
  # Scene Board 模式
} else {
  # 传统模式
}

if (defined(global_parts_info.powermgr_power_manager)) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "POWER_MANAGER_ENABLE" ]
}

if (build_variant == "user") {
  defines += [ "IS_RELEASE_VERSION" ]
}
```

## 产物输出路径

### 标准系统输出

```
out/standard/
├── system/
│   ├── lib/
│   │   ├── libwm.so
│   │   ├── libwm_lite.so
│   │   ├── libdm.so
│   │   ├── libdms.so
│   │   ├── libwms.so (非 Scene Board)
│   │   ├── libscene_session_manager.z.so
│   │   ├── libscreen_session_manager.z.so
│   │   └── ...
│   ├── module/
│   │   ├── window_napi.so
│   │   ├── display_napi.so
│   │   └── ...
│   └── bin/
│       ├── snapshot_display
│       └── setresolution_screen
└── vendor/
    └── ...
```

### SA 配置文件

```
system/
└── profile/
    └── wms_sa_profile.xml (从 sa_profile/ 生成)
```

## 关键编译选项

### 安全编译选项

```gn
sanitize = {
  cfi = true                    # 控制流完整性
  cfi_cross_dso = true          # 跨 DSO CFI
  cfi_vcall_icall_only = true   # 仅虚函数/间接调用 CFI
  integer_overflow = true       # 整数溢出检查
  ubsan = true                  # 未定义行为检查
  boundary_sanitize = true      # 边界检查
}

branch_protector_ret = "pac_ret"  # 返回地址保护 (ARM)
```

### 架构特定选项

```gn
if (is_ohos && is_clang && target_cpu == "arm64") {
  ldflags = [
    "-Wl,--emit-relocs",
    "-Wl,--no-relax",
    "-mno-fix-cortex-a53-843419"
  ]
}
```

## 运行时加载关系

```
系统启动
    │
    ▼
SA Manager 加载 libwms.z.so / libscreen_session_manager.z.so
    │
    ▼
SystemAbility 注册 (SA ID 4606/4607)
    │
    ▼
应用启动
    │
    ▼
加载 libwm.so / libdm.so
    │
    ▼
通过 IPC 连接到 WMS/DMS Service
```

## 相关文档

- [目录结构](03_Directory_Structure.md)
- [内部 API](05_Inner_API.md)
- [附录/Config_Flags](appendix/Config_Flags.md)
