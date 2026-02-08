# 构建系统

## 4.1 GN 构建概述

本项目使用 **GN (Generate Ninja)** 作为构建系统，遵循 OpenHarmony 的 Cangjie 模块构建规范。

### 构建配置入口

| 文件 | 职责 |
|------|------|
| `BUILD.gn` (根目录) | 定义子系统和部件的构建入口 |
| `kit/SensorServiceKit/BUILD.gn` | Kit 模块构建 |
| `ohos/sensor/BUILD.gn` | OHOS 层模块构建 |

### 构建模板

使用 OpenHarmony 提供的 Cangjie 构建模板：

- `ohos_cangjie_shared_library`: 构建 Cangjie 共享库
- `copy_ohos_cangjie_sdk_api_lib`: 复制 SDK API 库

---

## 4.2 根构建配置

**位置**: `BUILD.gn`

```gn
import("//build/templates/cangjie/cjc.gn")

sensors_cangjie_wrapper_packages_ohos = [
    "//base/sensors/sensors_cangjie_wrapper/ohos/sensor:ohos.sensor",
]

sensors_cangjie_wrapper_packages_kit = [
    "//base/sensors/sensors_cangjie_wrapper/kit/SensorServiceKit:kit.SensorServiceKit",
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_sensors_cangjie_libs") {
  ohos_inputs = sensors_cangjie_wrapper_packages_ohos
  kit_inputs = sensors_cangjie_wrapper_packages_kit
}
```

### 配置说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `sensors_cangjie_wrapper_packages_ohos` | 列表 | OHOS 层包路径 |
| `sensors_cangjie_wrapper_packages_kit` | 列表 | Kit 层包路径 |
| `copy_sdk_sensors_cangjie_libs` | target | SDK 复制任务 |

---

## 4.3 OHOS 层构建

**位置**: `ohos/sensor/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.sensor") {

  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.sensor.cj" ]  // Windows/Mac 使用 mock
  } else {
    sources = [
      "error.cj",
      "ffi.cj",
      "log.cj",
      "sensor.cj",
      "sensor_manager.cj",
    ]
  }

  cj_deps = [
    "../../ohos/sensor:ohos.sensor",  // 自引用
  ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "sensor:cj_sensor_ffi" ]

  subsystem_name = "sensors"
  part_name = "sensors_cangjie_wrapper"
}
```

### Target 属性

| 属性 | 值 | 说明 |
|------|------|------|
| target_type | shared_library | Cangjie 共享库 |
| subsystem_name | sensors | 子系统名 |
| part_name | sensors_cangjie_wrapper | 部件名 |

### 源文件

| 平台 | 源文件 |
|------|--------|
| Linux/Android | `error.cj`, `ffi.cj`, `log.cj`, `sensor.cj`, `sensor_manager.cj` |
| Windows/Mac | `../../mock/ohos.sensor.cj` |

### Cangjie 依赖 (cj_deps)

| 依赖 | 用途 |
|------|------|
| `../../ohos/sensor:ohos.sensor` | 自引用 (支持循环导入) |

### Cangjie 外部依赖 (cj_external_deps)

| 依赖 | 用途 |
|------|------|
| `cangjie_ark_interop:ohos.business_exception` | 业务异常类 |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调调用框架 |
| `cangjie_ark_interop:ohos.ffi` | FFI 基础 |
| `cangjie_ark_interop:ohos.labels` | 注解标签 |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志输出 |

### 外部依赖 (external_deps)

| 依赖 | 用途 |
|------|------|
| `sensor:cj_sensor_ffi` | C++ FFI 实现 |

---

## 4.4 Kit 层构建

**位置**: `kit/SensorServiceKit/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("kit.SensorServiceKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/sensor:ohos.sensor",
  ]

  subsystem_name = "sensors"
  part_name = "sensors_cangjie_wrapper"
}
```

### Target 属性

| 属性 | 值 | 说明 |
|------|------|------|
| target_type | shared_library | Cangjie 共享库 |
| subsystem_name | sensors | 子系统名 |
| part_name | sensors_cangjie_wrapper | 部件名 |

### 源文件

| 文件 | 职责 |
|------|------|
| `index.cj` | 包入口，导出 `kit.SensorServiceKit` |

### Cangjie 依赖 (cj_deps)

| 依赖 | 用途 |
|------|------|
| `../../ohos/sensor:ohos.sensor` | OHOS 层传感器模块 |

---

## 4.5 bundle.json 配置

**位置**: `bundle.json`

```json
{
  "name": "@ohos/sensors_cangjie_wrapper",
  "description": "Provides APIs for performing basic operations on sensors...",
  "version": "6.1",
  "component": {
    "name": "sensors_cangjie_wrapper",
    "subsystem": "sensors",
    "adapted_system_type": [ "standard" ],
    "rom": "160KB",
    "ram": "144KB",
    "deps": {
      "components": [
        "cangjie_ark_interop",
        "hiviewdfx_cangjie_wrapper",
        "sensor"
      ]
    },
    "build": {
      "sub_component": [
        "//base/sensors/sensors_cangjie_wrapper/ohos/sensor:ohos.sensor",
        "//base/sensors/sensors_cangjie_wrapper/kit/SensorServiceKit:kit.SensorServiceKit"
      ],
      "inner_kits": [
        {
          "name": "//base/sensors/sensors_cangjie_wrapper:copy_sdk_sensors_cangjie_libs"
        },
        {
          "name": "//base/sensors/sensors_cangjie_wrapper:copy_sdk_sensors_cangjie_libs_kit"
        }
      ]
    }
  }
}
```

### 组件属性

| 属性 | 值 | 说明 |
|------|------|------|
| name | `@ohos/sensors_cangjie_wrapper` | NPM 包名 |
| version | 6.1 | 版本号 |
| subsystem | sensors | 所属子系统 |
| adapted_system_type | standard | 适配标准系统 |
| rom | 160KB | ROM 占用 |
| ram | 144KB | RAM 占用 |

### 依赖组件

| 组件 | 说明 |
|------|------|
| `cangjie_ark_interop` | Cangjie Ark 互操作框架 |
| `hiviewdfx_cangjie_wrapper` | 日志框架 |
| `sensor` | 传感器基础库 (包含 C++ FFI) |

### 子组件 (sub_component)

| 路径 | 说明 |
|------|------|
| `ohos/sensor:ohos.sensor` | OHOS 层接口 |
| `kit/SensorServiceKit:kit.SensorServiceKit` | Kit 层接口 |

### 内部套件 (inner_kits)

| 名称 | 说明 |
|------|------|
| `copy_sdk_sensors_cangjie_libs` | 复制 OHOS SDK |
| `copy_sdk_sensors_cangjie_libs_kit` | 复制 Kit SDK |

---

## 4.6 条件编译

### 平台判断

```gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.sensor.cj" ]
} else {
    sources = [
      "error.cj",
      "ffi.cj",
      "log.cj",
      "sensor.cj",
      "sensor_manager.cj",
    ]
}
```

**平台判断逻辑**:

| 条件 | 平台 | 源文件 |
|------|------|--------|
| `is_mingw` | Windows (MinGW) | mock |
| `is_mac` | macOS | mock |
| 默认 | Linux/Android | 实际实现 |

### Mock 用途

Mock 实现用于 Windows 和 macOS 开发环境的**编译支持**，不提供实际传感器功能。
