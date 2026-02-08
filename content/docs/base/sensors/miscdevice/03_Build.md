# 构建配置

## 概述

miscdevice 使用 GN (Generate Ninja) 作为构建系统，构建配置定义在 `miscdevice.gni` 和各目录的 `BUILD.gn` 文件中。

## 根配置 (miscdevice.gni)

**文件**: `miscdevice.gni`

### Feature Flags

| Flag | 默认值 | 说明 |
|------|--------|------|
| `miscdevice_feature_vibrator_custom` | true | 启用自定义振动功能 |
| `miscdevice_feature_hiviewdfx_hisysevent` | true | 启用 HiSysEvent 事件上报 |
| `miscdevice_feature_vibrator_input_method_enable` | true | 启用输入法振动支持 |
| `miscdevice_feature_crown_vibrator_enable` | false | 启用表冠振动 |
| `miscdevice_feature_do_not_disturb_enable` | false | 启用勿扰模式 |

### 条件编译宏

```python
if (miscdevice_feature_vibrator_custom) {
    defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_CUSTOM" ]
}

if (miscdevice_feature_vibrator_input_method_enable) {
    defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_INPUT_METHOD" ]
} else {
    defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_PRESET_INFO" ]
}

if (miscdevice_feature_crown_vibrator_enable) {
    defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_CROWN" ]
}

if (miscdevice_feature_do_not_disturb_enable) {
    defines += [ "OHOS_BUILD_ENABLE_DO_NOT_DISTURB" ]
}
```

### HDI 条件配置

```python
if (miscdevice_feature_hdf_drivers_interface_vibrator) {
    defines += [ "HDF_DRIVERS_INTERFACE_VIBRATOR" ]
}

if (hdf_drivers_interface_light) {
    defines += [ "HDF_DRIVERS_INTERFACE_LIGHT" ]
}
```

## 关键 Targets

### 服务层 Targets

#### libmiscdevice_service (SA)

**类型**: `ohos_shared_library`
**输出**: `libmiscdevice_service.z.so`
**路径**: `services/miscdevice_service/BUILD.gn`

**Sources**:
```gn
sources = [
    "haptic_matcher/src/custom_vibration_matcher.cpp",
    "hdi_connection/adapter/src/compatible_light_connection.cpp",
    "hdi_connection/interface/src/light_hdi_connection.cpp",
    "hdi_connection/interface/src/vibrator_hdi_connection.cpp",
    "hdi_connection/adapter/src/vibrator_plug_callback.cpp",
    "src/miscdevice_common_event_subscriber.cpp",
    "src/miscdevice_dump.cpp",
    "src/miscdevice_observer.cpp",
    "src/miscdevice_service.cpp",
    "src/vibration_priority_manager.cpp",
    "src/vibrator_thread.cpp",
]
```

**Deps**:
```gn
deps = [
    "$SUBSYSTEM_DIR/frameworks/native/vibrator:miscdevice_service_stub",
    "$SUBSYSTEM_DIR/utils/common:libmiscdevice_utils",
]

external_deps = [
    "ability_base:configuration",
    "ability_runtime:app_manager",
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "cJSON:cjson",
    "common_event_service:cesfwk_innerkits",
    "data_share:datashare_consumer",
    "hilog:libhilog",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
]
```

**配置**:
```gn
shlib_type = "sa"  # 标记为 System Ability
part_name = "miscdevice"
subsystem_name = "sensors"
```

### 框架层 Targets

#### vibrator (JS N-API)

**类型**: `ohos_shared_library`
**输出**: `libvibrator.z.so`
**路径**: `frameworks/js/napi/BUILD.gn`

**Sources**:
```gn
sources = [
    "$SUBSYSTEM_DIR/frameworks/js/napi/vibrator/src/vibrator_js.cpp",
    "$SUBSYSTEM_DIR/frameworks/js/napi/vibrator/src/vibrator_napi_error.cpp",
    "$SUBSYSTEM_DIR/frameworks/js/napi/vibrator/src/vibrator_napi_utils.cpp",
    "$SUBSYSTEM_DIR/frameworks/js/napi/vibrator/src/vibrator_pattern_js.cpp",
]
```

**Deps**:
```gn
deps = [
    "$SUBSYSTEM_DIR/frameworks/native/vibrator:vibrator_interface_native",
    "$SUBSYSTEM_DIR/utils/common:libmiscdevice_utils",
]

external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "napi:ace_napi",
]
```

**安装路径**: `module/` 目录

#### vibrator_interface_native (Native API)

**类型**: `ohos_shared_library`
**输出**: `vibrator_agent.z.so` (重命名)
**路径**: `frameworks/native/vibrator/BUILD.gn`

**Sources**:
```gn
sources = [ "vibrator_agent.cpp" ]

deps = [
    ":libvibrator_native",
    ":libvibrator_ndk",
    "$SUBSYSTEM_DIR/frameworks/native/light:light_ndk_header",
    "$SUBSYSTEM_DIR/utils/common:libmiscdevice_utils",
]
```

**Inner API**: `platformsdk` 标记

#### light_interface_native (Light API)

**类型**: `ohos_shared_library`
**输出**: `light_agent.z.so`
**路径**: `frameworks/native/light/BUILD.gn`

#### ohvibrator (C NDK)

**类型**: `ohos_shared_library`
**输出**: `ndk/vibrator.z.so`
**路径**: `frameworks/capi/BUILD.gn`

**Sources**:
```gn
sources = [ "vibrator.cpp" ]

defines = [ "API_EXPORT=__attribute__((visibility (\"default\")))" ]
```

#### vibrator_taihe_native (Taihe/ArkTS)

**类型**: `taihe_shared_library`
**输出**: `libvibrator_taihe_native.z.so`
**路径**: `frameworks/ets/taihe/BUILD.gn`

**Sources**:
```gn
sources = [
    ":run_taihe",           # 生成文件
    "ani_constructor.cpp",
    "ohos.vibrator.impl.cpp",
]
```

#### cj_vibrator_ffi (Cangjie FFI)

**类型**: `ohos_shared_library`
**输出**: `libvibrator_ffi.z.so`
**路径**: `frameworks/cj/BUILD.gn`

**Sources**:
```gn
sources = [ "src/vibrator_ffi.cpp" ]

external_deps = [ "napi:cj_bind_ffi" ]
```

### 工具库 Targets

#### libmiscdevice_utils

**类型**: `ohos_shared_library`
**输出**: `libmiscdevice_utils.z.so`
**路径**: `utils/common/BUILD.gn`

**Sources**:
```gn
sources = [
    "src/file_utils.cpp",
    "src/json_parser.cpp",
    "src/light_animation_ipc.cpp",
    "src/light_info_ipc.cpp",
    "src/permission_util.cpp",
    "src/vibrator_infos.cpp",
]
```

#### libvibrator_decoder

**类型**: `ohos_shared_library`
**输出**: `libvibrator_decoder.z.so`
**路径**: `utils/haptic_decoder/oh_json/BUILD.gn`

## 编译产物清单

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| `libmiscdevice_service.z.so` | SA | `system/lib/` | MiscDevice 系统服务 |
| `libvibrator.z.so` | N-API | `module/lib/` | JS 振动器绑定 |
| `vibrator_agent.z.so` | Native | `system/lib/` | Native API |
| `libvibrator_ffi.z.so` | FFI | `system/lib/` | Cangjie FFI |
| `libvibrator_taihe_native.z.so` | Taihe | `system/lib/` | ArkTS/Taihe |
| `liblight_agent.z.so` | Native | `system/lib/` | Native Light API |
| `libmiscdevice_utils.z.so` | Utils | `system/lib/` | 通用工具库 |
| `libvibrator_decoder.z.so` | Utils | `system/lib/` | 触觉解码器 |
| `vibrator_abc.abc` | Bytecode | `system/framework/` | ArkTS 字节码 |
| `libvibrator.ndk.zip` | NDK | `ndk/` | C NDK 包 |

## 运行时加载关系

```
应用进程
    │
    ├── libvibrator.z.so (JS N-API)
    │       │
    │       └── 加载 ──> vibrator_agent.z.so (Native Client)
    │                   │
    │                   └── IPC ──> libmiscdevice_service.z.so (SA)
    │
    ├── libvibrator_ffi.z.so (CJ FFI)
    │       │
    │       └── 加载 ──> vibrator_agent.z.so
    │
    └── libvibrator_taihe_native.z.so (Taihe)
            │
            └── 加载 ──> vibrator_agent.z.so
```

## 安全编译选项

| 选项 | 值 | 说明 |
|------|-----|------|
| `branch_protector_ret` | `"pac_ret"` | PAC/BTI 返回地址保护 |
| `sanitize.cfi` | `true` | 控制流完整性 |
| `sanitize.cfi_cross_dso` | `true` | 跨 DSO CFI |
| `-fvisibility=hidden` | 启用 | 符号隐藏 |
| `-ffunction-sections` | 启用 | 函数分段 |
| `-fdata-sections` | 启用 | 数据分段 |
| `-Oz` | 优化 | 大小优化 |

## 构建命令

```bash
# 构建 miscdevice part
./build.sh --product <product> --build-target miscdevice

# 仅构建 SA 服务
./build.sh --product <product> --build-target libmiscdevice_service

# 仅构建 JS N-API
./build.sh --product <product> --build-target vibrator_js_target

# 构建 NDK
./build.sh --product <product> --build-target libvibrator_ndk
```
