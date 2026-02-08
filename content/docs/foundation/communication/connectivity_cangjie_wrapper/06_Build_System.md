# 构建系统

> connectivity_cangjie_wrapper GN Targets、编译产物与依赖关系

## GN 构建概述

项目使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统，配置文件位于各目录下的 `BUILD.gn` 文件。

### 构建模板

项目主要使用以下 GN 模板：

| 模板 | 用途 |
|------|------|
| `ohos_cangjie_shared_library` | 构建 Cangjie 共享库 (.so) |
| `copy_ohos_cangjie_sdk_api_lib` | 复制 SDK API 库文件 |

### 构建入口

**根 BUILD.gn**: `BUILD.gn`

```gn
import("//build/templates/cangjie/cjc.gni")

connectivity_cangjie_wrapper_packages_ohos = [
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/connection:ohos.bluetooth.connection",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/base_profile:ohos.bluetooth.base_profile",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth:ohos.bluetooth",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/hfp:ohos.bluetooth.hfp",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/constant:ohos.bluetooth.constant",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/wifi_manager:ohos.wifi_manager",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/ble:ohos.bluetooth.ble",
    "//foundation/communication/connectivity_cangjie_wrapper/ohos/bluetooth/a2dp:ohos.bluetooth.a2dp"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_connectivity_cangjie_libs") {
  ohos_inputs = connectivity_cangjie_wrapper_packages_ohos
}
```

## Targets 清单

### 蓝牙模块 Targets

| Target | 类型 | 路径 | 输出 |
|--------|------|------|------|
| `ohos.bluetooth` | ohos_cangjie_shared_library | `ohos/bluetooth/BUILD.gn` | `libohos.bluetooth.so` |
| `ohos.bluetooth.ble` | ohos_cangjie_shared_library | `ohos/bluetooth/ble/BUILD.gn` | `libohos.bluetooth.ble.so` |
| `ohos.bluetooth.a2dp` | ohos_cangjie_shared_library | `ohos/bluetooth/a2dp/BUILD.gn` | `libohos.bluetooth.a2dp.so` |
| `ohos.bluetooth.hfp` | ohos_cangjie_shared_library | `ohos/bluetooth/hfp/BUILD.gn` | `libohos.bluetooth.hfp.so` |
| `ohos.bluetooth.base_profile` | ohos_cangjie_shared_library | `ohos/bluetooth/base_profile/BUILD.gn` | `libohos.bluetooth.base_profile.so` |
| `ohos.bluetooth.connection` | ohos_cangjie_shared_library | `ohos/bluetooth/connection/BUILD.gn` | `libohos.bluetooth.connection.so` |
| `ohos.bluetooth.constant` | ohos_cangjie_shared_library | `ohos/bluetooth/constant/BUILD.gn` | `libohos.bluetooth.constant.so` |

### WLAN 模块 Targets

| Target | 类型 | 路径 | 输出 |
|--------|------|------|------|
| `ohos.wifi_manager` | ohos_cangjie_shared_library | `ohos/wifi_manager/BUILD.gn` | `libohos.wifi_manager.so` |

### Kit 层 Target

| Target | 类型 | 路径 | 输出 |
|--------|------|------|------|
| `kit.ConnectivityKit` | ohos_cangjie_shared_library | `kit/ConnectivityKit/BUILD.gn` | `libkit.ConnectivityKit.so` |

### SDK 复制 Targets

| Target | 类型 | 路径 | 说明 |
|--------|------|------|------|
| `copy_sdk_connectivity_cangjie_libs` | copy_ohos_cangjie_sdk_api_lib | `BUILD.gn` | 复制 ohos API 库 |
| `copy_sdk_connectivity_cangjie_libs_kit` | - | `bundle.json` | 复制 kit API 库 |

## Target 详细配置

### ble (ohos.bluetooth.ble)

**BUILD.gn 位置**: `ohos/bluetooth/ble/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.bluetooth.ble") {
  if (is_mingw || is_mac){
    sources = [ "../../../mock/ohos.bluetooth.ble.cj" ]
  } else {
    sources = [
      "ble.cj",
      "gatt_client_device.cj",
      "gatt_server.cj",
      "gatt_util.cj",
      "native.cj",
      "util.cj",
    ]
  }

  external_deps = [ "bluetooth:cj_bluetooth_ble_ffi" ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  cj_deps = [
    "../:ohos.bluetooth",
    "../constant:ohos.bluetooth.constant",
  ]

  subsystem_name = "communication"
  part_name = "connectivity_cangjie_wrapper"
}
```

**配置说明**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | 条件编译 | Windows/macOS 用 mock，其他平台用实际代码 |
| `external_deps` | `bluetooth:cj_bluetooth_ble_ffi` | 蓝牙 BLE FFI 依赖 |
| `cj_external_deps` | 5 个依赖 | Cangjie 互操作基础库 |
| `cj_deps` | 2 个模块依赖 | 蓝牙基础模块和常量 |
| `subsystem_name` | `communication` | 子系统名 |
| `part_name` | `connectivity_cangjie_wrapper` | 组件名 |

### wifi_manager (ohos.wifi_manager)

**BUILD.gn 位置**: `ohos/wifi_manager/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.wifi_manager") {
  if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.wifi_manager.cj" ]
  } else {
    sources = [
      "common.cj",
      "error_code.cj",
      "ip_info.cj",
      "wifi_info_elem.cj",
      "wifi_p2p_config.cj",
      "wifi_scan_info.cj",
      "wifi.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "wifi:cj_wifi_ffi" ]

  subsystem_name = "communication"
  part_name = "connectivity_cangjie_wrapper"
}
```

### kit.ConnectivityKit

**BUILD.gn 位置**: `kit/ConnectivityKit/BUILD.gn`

```gn
ohos_cangjie_shared_library("kit.ConnectivityKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/bluetooth/base_profile:ohos.bluetooth.base_profile",
    "../../ohos/bluetooth/ble:ohos.bluetooth.ble",
    "../../ohos/bluetooth:ohos.bluetooth",
    "../../ohos/bluetooth/connection:ohos.bluetooth.connection",
    "../../ohos/bluetooth/constant:ohos.bluetooth.constant",
    "../../ohos/bluetooth/hfp:ohos.bluetooth.hfp",
    "../../ohos/bluetooth/a2dp:ohos.bluetooth.a2dp",
    "../../ohos/wifi_manager:ohos.wifi_manager",
  ]

  subsystem_name = "communication"
  part_name = "connectivity_cangjie_wrapper"
}
```

## 依赖关系详解

### 外部依赖 (external_deps)

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| `bluetooth:cj_bluetooth_ble_ffi` | communication_bluetooth | BLE FFI 接口 |
| `bluetooth:cj_bluetooth_a2dp_ffi` | communication_bluetooth | A2DP FFI 接口 |
| `bluetooth:cj_bluetooth_hfp_ffi` | communication_bluetooth | HFP FFI 接口 |
| `bluetooth:cj_bluetooth_connection_ffi` | communication_bluetooth | 连接 FFI 接口 |
| `wifi:cj_wifi_ffi` | communication_wifi | WiFi FFI 接口 |

### Cangjie 外部依赖 (cj_external_deps)

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| `cangjie_ark_interop:ohos.business_exception` | arkcompiler_cangjie_ark_interop | 业务异常类 |
| `cangjie_ark_interop:ohos.callback_invoke` | arkcompiler_cangjie_ark_interop | 回调调用 |
| `cangjie_ark_interop:ohos.ffi` | arkcompiler_cangjie_ark_interop | FFI 工具 |
| `cangjie_ark_interop:ohos.labels` | arkcompiler_cangjie_ark_interop | API 标签 |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | hiviewdfx_cangjie_wrapper | 日志接口 |

### Cangjie 内部依赖 (cj_deps)

| 依赖项 | 用途 |
|--------|------|
| `ohos.bluetooth` | 基础日志、错误处理 |
| `ohos.bluetooth.constant` | 枚举常量 |
| `ohos.bluetooth.base_profile` | Profile 框架 |

## 编译产物

### 预期产物路径

```
out/ohos-arm-release/
└── libs/
    ├── libohos.bluetooth.so           # 基础模块
    ├── libohos.bluetooth.ble.so       # BLE 模块
    ├── libohos.bluetooth.a2dp.so      # A2DP 模块
    ├── libohos.bluetooth.hfp.so       # HFP 模块
    ├── libohos.bluetooth.base_profile.so  # 基础 Profile
    ├── libohos.bluetooth.connection.so    # 连接管理
    ├── libohos.bluetooth.constant.so      # 常量定义
    └── libohos.wifi_manager.so       # WiFi P2P 模块
```

### SDK 产物

SDK 构建时，产物会被复制到 SDK 目录：

```
sdk/...
├── api/
│   └── ConnectivityKit/
│       ├── ohos/
│       │   ├── bluetooth/
│       │   │   ├── ble/
│       │   │   ├── a2dp/
│       │   │   └── ...
│       │   └── wifi_manager/
│       └── kit/
│           └── ConnectivityKit/
└── native/
    └── lib/
        └── libkit.ConnectivityKit.so
```

## 跨平台支持

项目支持在 **Windows (mingw)** 和 **macOS** 上编译，使用 mock 代码作为桩实现：

```gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.bluetooth.ble.cj" ]
} else {
    sources = [ "ble.cj", "gatt_client_device.cj", ... ]
}
```

## bundle.json 配置

**文件位置**: `bundle.json`

```json
{
    "component": {
        "name": "connectivity_cangjie_wrapper",
        "subsystem": "communication",
        "adapted_system_type": ["standard"],
        "rom": "1100KB",
        "ram": "1176KB",
        "deps": {
            "components": [
                "cangjie_ark_interop",
                "hiviewdfx_cangjie_wrapper",
                "bluetooth",
                "wifi"
            ]
        },
        "build": {
            "sub_component": [ ... ],
            "inner_kits": [ ... ]
        }
    }
}
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](./02_Directory_Structure.md) | 模块划分与文件布局 |
| [架构说明](./03_Architecture.md) | 组件关系与数据流 |
| [内部 API](./05_Inner_API.md) | 模块接口与依赖方向 |
| [安全评审](./07_Security_Review.md) | 安全使用指南 |
