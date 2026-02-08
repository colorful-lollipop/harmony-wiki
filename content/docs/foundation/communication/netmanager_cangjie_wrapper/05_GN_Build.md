# GN 构建配置与产物

## 构建系统概述

本项目使用 **GN (Generate Ninja)** 构建系统，通过仓颉编译器 (`cjc`) 编译仓颉源代码。

**证据来源**：`BUILD.gn:14`

---

## 根构建配置

### 文件位置

`BUILD.gn`

### 配置内容

```gn
import("//build/templates/cangjie/cjc.gni")

netmanager_cangjie_wrapper_packages_ohos = [
  "//foundation/communication/netmanager_cangjie_wrapper/ohos/net/http:ohos.net.http",
  "//foundation/communication/netmanager_cangjie_wrapper/ohos/net/connection:ohos.net.connection",
  "//foundation/communication/netmanager_cangjie_wrapper/ohos/net:ohos.net",
]
netmanager_cangjie_wrapper_packages_kit = [ "//foundation/communication/netmanager_cangjie_wrapper/kit/NetworkKit:kit.NetworkKit" ]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_netmanager_cangjie_libs") {
  ohos_inputs = netmanager_cangjie_wrapper_packages_ohos
  kit_inputs = netmanager_cangjie_wrapper_packages_kit
}
```

**证据来源**：`BUILD.gn:14-25`

### Targets 列表

| Target 名称 | 类型 | 描述 |
|------------|------|------|
| `copy_sdk_netmanager_cangjie_libs` | `copy_ohos_cangjie_sdk_api_lib` | SDK 复制任务 |

---

## ohos.net 模块

### 文件位置

`ohos/net/BUILD.gn`

### 配置内容

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.net") {
  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.net.cj" ]
  } else {
    sources = [ "net.cj" ]
  }

  subsystem_name = "netmanager"
  part_name = "netmanager_cangjie_wrapper"
}
```

### Target 详情

| 属性 | 值 | 说明 |
|------|------|------|
| `type` | `ohos_cangjie_shared_library` | 仓颉共享库 |
| `sources` | `["net.cj"]` 或 mock 文件 | 源文件 |
| `subsystem_name` | `netmanager` | 子系统名 |
| `part_name` | `netmanager_cangjie_wrapper` | 组件名 |

**条件编译**：Windows/Mac 平台使用 mock 实现

**证据来源**：`ohos/net/BUILD.gn:14-29`

---

## ohos.net.connection 模块

### 文件位置

`ohos/net/connection/BUILD.gn`

### 配置内容

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.net.connection") {
  if (is_mingw || is_mac) {
    sources = [ "../../../mock/ohos.net.connection.cj" ]
  } else {
    sources = [
      "connection_common.cj",
      "connection_ffi.cj",
      "net_connection.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "netmanager_base:cj_net_connection_ffi" ]

  subsystem_name = "netmanager"
  part_name = "netmanager_cangjie_wrapper"
}
```

### Target 详情

| 属性 | 值 | 说明 |
|------|------|------|
| `type` | `ohos_cangjie_shared_library` | 仓颉共享库 |
| `sources` | 3 个 cj 文件 | 源文件列表 |
| `external_deps` | `netmanager_base:cj_net_connection_ffi` | C 接口依赖 |
| `cj_external_deps` | 5 个仓颉组件 | 仓颉互操作依赖 |

**证据来源**：`ohos/net/connection/BUILD.gn:14-43`

---

## ohos.net.http 模块

### 文件位置

`ohos/net/http/BUILD.gn`

### 配置内容

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.net.http") {
  sources = [
    "http.cj",
    "http_common.cj",
    "http_ffi.cj",
  ]

  external_deps = [ "netstack:cj_net_http_ffi" ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.encoding.json",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  cj_deps = [
    "../connection:ohos.net.connection",
  ]

  subsystem_name = "netmanager"
  part_name = "netmanager_cangjie_wrapper"
}
```

### Target 详情

| 属性 | 值 | 说明 |
|------|------|------|
| `type` | `ohos_cangjie_shared_library` | 仓颉共享库 |
| `sources` | 3 个 cj 文件 | 源文件列表 |
| `external_deps` | `netstack:cj_net_http_ffi` | HTTP 栈依赖 |
| `cj_deps` | `ohos.net.connection` | 内部模块依赖 |

**证据来源**：`ohos/net/http/BUILD.gn:14-43`

---

## Kit 层

### 文件位置

`kit/NetworkKit/BUILD.gn`

### 配置内容

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("kit.NetworkKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/net:ohos.net",
    "../../ohos/net/connection:ohos.net.connection",
    "../../ohos/net/http:ohos.net.http",
  ]

  subsystem_name = "netmanager"
  part_name = "netmanager_cangjie_wrapper"
}
```

### Target 详情

| 属性 | 值 | 说明 |
|------|------|------|
| `type` | `ohos_cangjie_shared_library` | 仓颉共享库 |
| `sources` | `["index.cj"]` | Kit 入口文件 |
| `cj_deps` | 3 个内部模块 | 内部依赖 |

**证据来源**：`kit/NetworkKit/BUILD.gn:14-31`

---

## bundle.json 配置

### 文件位置

`bundle.json:31-44`

### 子组件配置

```json
"build": {
  "sub_component": [
    "//foundation/communication/netmanager_cangjie_wrapper/ohos/net/connection:ohos.net.connection",
    "//foundation/communication/netmanager_cangjie_wrapper/ohos/net/http:ohos.net.http",
    "//foundation/communication/netmanager_cangjie_wrapper/ohos/net:ohos.net",
    "//foundation/communication/netmanager_cangjie_wrapper/kit/NetworkKit:kit.NetworkKit"
  ],
  "inner_kits": [
    {
      "name": "//foundation/communication/netmanager_cangjie_wrapper:copy_sdk_netmanager_cangjie_libs"
    },
    {
      "name": "//foundation/communication/netmanager_cangjie_wrapper:copy_sdk_netmanager_cangjie_libs_kit"
    }
  ],
  "test": []
}
```

---

## 编译产物

### 预期产物

| 产物类型 | 产物路径 | 描述 |
|---------|---------|------|
| `.so` 共享库 | `out/.../libs/libohos.net.connection.z.so` | 连接模块库 |
| `.so` 共享库 | `out/.../libs/libohos.net.http.z.so` | HTTP 模块库 |
| `.so` 共享库 | `out/.../libs/libkit.NetworkKit.z.so` | Kit 接口库 |
| SDK 包 | `.../sdk/packages/` | 对外 SDK |

### 产物尺寸

| 指标 | 大小 |
|------|------|
| ROM | ~550KB |
| RAM | ~524KB |

**证据来源**：`bundle.json:20-21`

---

## 模块依赖图

```
kit/NetworkKit
    │
    ├── cj_deps:
    │   ├── ohos.net
    │   ├── ohos.net.connection
    │   │       └── external_deps: netmanager_base:cj_net_connection_ffi
    │   └── ohos.net.http
    │           ├── external_deps: netstack:cj_net_http_ffi
    │           └── cj_deps: ohos.net.connection
```

---

## 相关文档

- [API 参考](03_API_Reference.md) - API 说明
- [FFI 接口](04_FFI_Interface.md) - FFI 定义
- [系统架构](02_Architecture.md) - 架构设计
