# 构建系统与编译产物

## 概述

OpenHarmony HDI 使用 **GN (Generate Ninja)** 作为构建系统，通过 `interface.gni` 模板封装，根据不同系统类型选择对应的 HDI 构建模板。

## 构建模板体系

### 模板层级结构

```
interface.gni (顶层封装)
    ↓
┌─────────────────────────────────────────────────────┐
│  根据 is_standard_system / is_small_system /         │
│  is_mini_system 选择不同模板                         │
└─────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────┐
│  is_standard_system: hdi.gni  (IPC + Passthrough)   │
│  is_small_system:   hdi_small.gni (Passthrough)     │
│  is_mini_system:    hdi_mini.gni (Low Mode)        │
└─────────────────────────────────────────────────────┘
```

**证据**: `interface.gni:14-45`

### 系统类型配置

| 系统类型 | GNI 文件 | 模板名称 | 默认语言 | 生成模式 |
|---------|---------|---------|---------|---------|
| **Standard** | `//drivers/hdf_core/adapter/uhdf2/hdi.gni` | `hdi()` | C++ | IPC |
| **Small** | `//drivers/hdf_core/adapter/uhdf/hdi_small.gni` | `hdi_small()` | C | Passthrough |
| **Mini** | `//drivers/hdf_core/adapter/khdf/liteos_m/hdi_mini.gni` | `hdi_mini()` | C | Low |

---

## interface.gni 详解

**文件路径**: `interface.gni`

```gni
template("interface") {
  if (is_standard_system) {
    import("//drivers/hdf_core/adapter/uhdf2/hdi.gni")
    hdi(target_name) {
      # 默认值: system="full", mode="ipc", language="cpp"
      forward_variables_from(invoker, "*")
    }
  } else if (is_small_system) {
    import("//drivers/hdf_core/adapter/uhdf/hdi_small.gni")
    hdi_small(target_name) {
      # 默认值: system="lite", mode="passthrough", language="c"
      forward_variables_from(invoker, "*")
    }
  } else if (is_mini_system) {
    import("//drivers/hdf_core/adapter/khdf/liteos_m/hdi_mini.gni")
    hdi_mini(target_name) {
      # 默认值: system="mini", mode="low", language="c"
      forward_variables_from(invoker, "*",
                             ["subsystem_name", "part_name"])
    }
  }
}
```

**关键特性**:
- 使用 `forward_variables_from(invoker, "*")` 转发所有参数
- Mini 系统会过滤 `subsystem_name` 和 `part_name`

---

## BUILD.gn 配置详解

### 标准配置模板

**示例**: `audio/v1_0/BUILD.gn`

```gni
import("//build/config/components/hdi/hdi.gni")

hdi("audio") {
  module_name = "audio_service"
  
  sources = [
    "AudioTypes.idl",
    "IAudioAdapter.idl",
    "IAudioCallback.idl",
    "IAudioCapture.idl",
    "IAudioManager.idl",
    "IAudioRender.idl",
  ]
  
  language = "c"
  subsystem_name = "hdf"
  part_name = "drivers_interface_audio"
}
```

### 条件编译示例

**示例**: `sensor/v3_0/BUILD.gn`

```gni
import("//build/config/components/hdi/hdi.gni")

if (defined(ohos_lite)) {
  group("libsensor_proxy_3.0") {
    deps = []
    public_configs = []
  }
} else {
  hdi("sensor") {
    module_name = "sensor_service"
    
    sources = [
      "ISensorCallback.idl",
      "ISensorInterface.idl",
      "SensorTypes.idl",
      "ISensorPlugCallback.idl",
    ]
    
    innerapi_tags = [
      "chipsetsdk",
      "platformsdk_indirect",
    ]
    
    language = "cpp"
    subsystem_name = "hdf"
    part_name = "drivers_interface_sensor"
  }
}
```

---

## hdi() 模板参数

### 必需参数

| 参数 | 类型 | 说明 |
|-----|------|------|
| `module_name` | string | 驱动模块名称，用于生成 `HdfDriverEntry.moduleName` |
| `sources` | list | IDL 源文件列表 |
| `language` | string | 生成代码语言：`"c"` 或 `"cpp"` |

**证据**: `audio/v1_0/BUILD.gn:22-32`

### 可选参数

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| `subsystem_name` | string | - | 子系统名称（如 `"hdf"`）|
| `part_name` | string | - | 部件名称（如 `"drivers_interface_sensor"`）|
| `mode` | string | `"ipc"` | 生成模式：`"ipc"` 或 `"passthrough"` |
| `innerapi_tags` | list | [] | API 标签（如 `["chipsetsdk"]`）|
| `install_images` | list | `["system"]` | 安装目标分区 |
| `branch_protector_ret` | string | - | 分支保护类型（如 `"pac_ret"`）|

### 特殊配置示例

#### Passthrough 模式 (无 IPC)

**示例**: `input/v1_0/BUILD.gn`

```gni
hdi("input") {
  module_name = "input_service"
  sources = [
    "IInputCallback.idl",
    "IInputInterfaces.idl",
    "InputTypes.idl",
  ]
  
  install_images = [
    "system",
    "updater"
  ]
  mode = "passthrough"  # 直通模式，无 IPC 开销
  language = "cpp"
  subsystem_name = "hdf"
  part_name = "drivers_interface_input"
}
```

#### 分支保护 (安全增强)

**示例**: `codec/v4_0/BUILD.gn`

```gni
hdi("codec") {
  module_name = "codec_service"
  sources = [
    "CodecExtTypes.idl",
    "CodecTypes.idl",
    "ICodecCallback.idl",
    "ICodecComponent.idl",
    "ICodecComponentManager.idl",
  ]
  
  branch_protector_ret = "pac_ret"  # 启用 ARM PAC 分支保护
  language = "cpp"
  subsystem_name = "hdf"
  part_name = "drivers_interface_codec"
}
```

---

## bundle.json 配置

### 结构说明

**示例**: `audio/bundle.json`

```json
{
  "name": "@ohos/drivers_interface_audio",
  "description": "audio driver interface",
  "version": "3.2",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "drivers/interface/audio"
  },
  "component": {
    "name": "drivers_interface_audio",
    "subsystem": "hdf",
    "adapted_system_type": [
      "small",
      "standard"
    ],
    "features": [
      "drivers_interface_audio_feature_alsa_lib",
      "drivers_interface_audio_community",
      "drivers_interface_audio_feature_offload"
    ],
    "rom": "675KB",
    "ram": "1024KB",
    "deps": {
      "components": [
        "c_utils",
        "hdf_core",
        "hilog"
      ],
      "third_party": []
    },
    "build": {
      "sub_component": [
        "//drivers/interface/audio/effect/v1_0:libeffect_proxy_1.0",
        "//drivers/interface/audio/v6_0:libaudio_proxy_6.0"
      ],
      "inner_kits": [
        {
          "name": "//drivers/interface/audio/effect/v1_0:libeffect_proxy_1.0",
          "header": {
            "header_files": [],
            "header_base": "//drivers/interface/audio/effect"
          }
        }
      ]
    }
  }
}
```

### 关键字段说明

| 字段 | 说明 |
|-----|------|
| `name` | 模块包名，格式为 `@ohos/drivers_interface_<module>` |
| `component.name` | 组件名称，用于构建系统识别 |
| `component.subsystem` | 所属子系统，固定为 `"hdf"` |
| `component.adapted_system_type` | 适配的系统类型 |
| `component.features` | 特性开关 |
| `component.deps.components` | 依赖的其他组件 |
| `build.sub_component` | 需要构建的子目标列表 |
| `build.inner_kits` | 对外暴露的内部 API 和头文件 |

---

## 编译产物

### 标准编译产物

IDL 文件通过 `hdi()` 模板编译后生成以下产物：

```
.idl 文件
    ↓ hdi() 编译
┌─────────────────────────────────────────────────────────┐
│  产物清单                                                │
├─────────────────────────────────────────────────────────┤
│  lib<module>_client_v1.0.z.so   # 客户端代理库           │
│  lib<module>_stub_v1.0.z.so    # 服务端存根库            │
│  <module>_interface_driver.cpp  # 驱动入口模板代码        │
│  <module>_interface_service.h   # 服务接口头文件          │
└─────────────────────────────────────────────────────────┘
    ↓
安装到设备的路径: /system/lib[64]/hdf/xx/  或  /vendor/lib[64]/hdf/xx/
```

### 产物与 Target 映射

| BUILD.gn Target | 生成的产物 | 安装路径 |
|----------------|-----------|---------|
| `hdi("audio")` | `libaudio_client_*.so`, `libaudio_stub_*.so` | `/system/lib/hdf/audio/` |
| `hdi("sensor")` | `libsensor_client_*.so`, `libsensor_stub_*.so` | `/system/lib/hdf/sensor/` |
| `hdi("display_buffer")` | `libdisplay_buffer_*.so` | `/system/lib/hdf/display/` |

---

## 模块配置对比

### Audio 模块 (v1_0)

```gni
hdi("audio") {
  module_name = "audio_service"
  sources = [6 个 IDL 文件]
  language = "c"
  subsystem_name = "hdf"
  part_name = "drivers_interface_audio"
}
```

### Sensor 模块 (v3_0)

```gni
hdi("sensor") {
  module_name = "sensor_service"
  sources = [4 个 IDL 文件]
  language = "cpp"
  innerapi_tags = ["chipsetsdk", "platformsdk_indirect"]
  subsystem_name = "hdf"
  part_name = "drivers_interface_sensor"
}
```

### Camera 模块 (v1_0) - 复杂依赖

```gni
hdi("camera") {
  module_name = "camera_service"
  sources = [8 个 IDL 文件]
  sequenceable_pub_deps = [
    "../sequenceable/buffer_producer:libbuffer_producer_sequenceable_1.0",
  ]
  sequenceable_ext_deps = [
    "graphic_surface:buffer_handle",
    "graphic_surface:surface",
  ]
  language = "cpp"
  subsystem_name = "hdf"
  part_name = "drivers_interface_camera"
}
```

---

## 常见构建配置模式

### 1. 轻量级模块 (C 语言)

适用于简单接口，减少 IPC 开销：

```gni
language = "c"
mode = "passthrough"  # 可选
```

### 2. 功能丰富模块 (C++)

适用于复杂接口，需要面向对象特性：

```gni
language = "cpp"
```

### 3. 安全敏感模块

需要分支保护：

```gni
branch_protector_ret = "pac_ret"
```

### 4. 跨模块依赖

需要声明序列化依赖：

```gni
sequenceable_pub_deps = [...]  # 内部依赖
sequenceable_ext_deps = [...]  # 外部依赖
```
