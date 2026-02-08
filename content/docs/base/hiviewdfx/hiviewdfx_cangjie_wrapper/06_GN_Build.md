# GN 构建系统

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 的 GN 构建配置，包括 targets 定义、依赖关系和构建选项。

**构建系统**: GN (Generate Ninja)  
**构建模板**: `//build/templates/cangjie/cjc.gni`

---

## 根构建文件

### BUILD.gn

**文件位置**: `BUILD.gn`

**内容**:

```gn
import("//build/templates/cangjie/cjc.gni")

hiviewdfx_cangjie_wrapper_packages_ohos = [
  "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hilog:ohos.hilog",
  "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hiviewdfx:ohos.hiviewdfx",
  "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hiviewdfx/hi_app_event:ohos.hiviewdfx.hi_app_event",
  "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hi_trace_meter:ohos.hi_trace_meter",
]

hiviewdfx_cangjie_wrapper_packages_kit = [
  "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/kit/PerformanceAnalysisKit:kit.PerformanceAnalysisKit",
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_hiviewdfx_cangjie_libs") {
  ohos_inputs = hiviewdfx_cangjie_wrapper_packages_ohos
  kit_inputs = hiviewdfx_cangjie_wrapper_packages_kit
}
```

**数据来源**: `BUILD.gn:14-29`

### 配置说明

| 配置项 | 说明 |
|--------|------|
| `hiviewdfx_cangjie_wrapper_packages_ohos` | Ohos 模块列表 |
| `hiviewdfx_cangjie_wrapper_packages_kit` | Kit 模块列表 |
| `copy_ohos_cangjie_sdk_api_lib` | SDK 复制任务 |

---

## kit/PerformanceAnalysisKit

### BUILD.gn

**文件位置**: `kit/PerformanceAnalysisKit/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("kit.PerformanceAnalysisKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/hi_trace_meter:ohos.hi_trace_meter",
    "../../ohos/hilog:ohos.hilog",
    "../../ohos/hiviewdfx/hi_app_event:ohos.hiviewdfx.hi_app_event",
  ]

  subsystem_name = "hiviewdfx"
  part_name = "hiviewdfx_cangjie_wrapper"
}
```

**数据来源**: `kit/PerformanceAnalysisKit/BUILD.gn:16-30`

### Target 信息

| 属性 | 值 |
|------|-----|
| Target 类型 | ohos_cangjie_shared_library |
| Target 名称 | kit.PerformanceAnalysisKit |
| Sources | index.cj |
| Subsystem | hiviewdfx |
| Part | hiviewdfx_cangjie_wrapper |

### 依赖配置

| 依赖类型 | 依赖项 | 说明 |
|---------|-------|------|
| cj_deps | ohos.hi_trace_meter | 性能追踪模块 |
| cj_deps | ohos.hilog | 日志模块 |
| cj_deps | ohos.hiviewdfx.hi_app_event | 事件模块 |

---

## ohos/hilog

### BUILD.gn

**文件位置**: `ohos/hilog/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.hilog") {

  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.hilog.cj" ]
  } else {
    sources = [
      "hilog.cj",
      "hilog_channel.cj",
    ]
  }

  external_deps = [ "hilog:libhilog" ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.labels",
  ]

  subsystem_name = "hiviewdfx"
  part_name = "hiviewdfx_cangjie_wrapper"
}
```

**数据来源**: `ohos/hilog/BUILD.gn:16-39`

### Target 信息

| 属性 | 值 |
|------|-----|
| Target 类型 | ohos_cangjie_shared_library |
| Target 名称 | ohos.hilog |
| Sources | hilog.cj, hilog_channel.cj (或 mock) |

### 依赖配置

| 依赖类型 | 依赖项 | 说明 |
|---------|-------|------|
| external_deps | hilog:libhilog | Native 日志库 |
| cj_external_deps | cangjie_ark_interop:ohos.business_exception | 业务异常类 |
| cj_external_deps | cangjie_ark_interop:ohos.labels | 标签注解类 |

### 平台差异

| 平台 | Sources |
|------|---------|
| Windows (mingw) | mock/ohos.hilog.cj |
| macOS | mock/ohos.hilog.cj |
| Linux | hilog.cj, hilog_channel.cj |

---

## ohos/hi_trace_meter

### BUILD.gn

**文件位置**: `ohos/hi_trace_meter/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.hi_trace_meter") {

  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.hi_trace_meter.cj" ]
  } else {
    sources = [ "hi_trace_meter.cj" ]
  }

  external_deps = [ "hitrace:cj_hitracemeter_ffi" ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
  ]

  cj_deps = [ "../hilog:ohos.hilog" ]

  subsystem_name = "hiviewdfx"
  part_name = "hiviewdfx_cangjie_wrapper"
}
```

**数据来源**: `ohos/hi_trace_meter/BUILD.gn:16-38`

### Target 信息

| 属性 | 值 |
|------|-----|
| Target 类型 | ohos_cangjie_shared_library |
| Target 名称 | ohos.hi_trace_meter |
| Sources | hi_trace_meter.cj (或 mock) |

### 依赖配置

| 依赖类型 | 依赖项 | 说明 |
|---------|-------|------|
| external_deps | hitrace:cj_hitracemeter_ffi | FFI 库 |
| cj_external_deps | cangjie_ark_interop:ohos.ffi | FFI 辅助类 |
| cj_external_deps | cangjie_ark_interop:ohos.labels | 标签注解类 |
| cj_deps | ohos.hilog | 日志模块 |

---

## ohos/hiviewdfx/hi_app_event

### BUILD.gn

**文件位置**: `ohos/hiviewdfx/hi_app_event/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.hiviewdfx.hi_app_event") {

  if (is_mingw || is_mac) {
    sources = [ "../../../mock/ohos.hiviewdfx.hi_app_event.cj" ]
  } else {
    sources = [
      "app_event_package_holder.cj",
      "cj_event.cj",
      "cj_event_ffi.cj",
      "cj_hiappevent_log.cj",
      "hi_app_event.cj",
      "utils.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
  ]

  cj_deps = [ "../../hilog:ohos.hilog" ]

  external_deps = [ "hiappevent:cj_hiappevent_ffi" ]

  subsystem_name = "hiviewdfx"
  part_name = "hiviewdfx_cangjie_wrapper"
}
```

**数据来源**: `ohos/hiviewdfx/hi_app_event/BUILD.gn:16-46`

### Target 信息

| 属性 | 值 |
|------|-----|
| Target 类型 | ohos_cangjie_shared_library |
| Target 名称 | ohos.hiviewdfx.hi_app_event |
| Sources | 6 个 cj 文件 (或 mock) |

### 依赖配置

| 依赖类型 | 依赖项 | 说明 |
|---------|-------|------|
| cj_external_deps | cangjie_ark_interop:ohos.business_exception | 业务异常类 |
| cj_external_deps | cangjie_ark_interop:ohos.ffi | FFI 辅助类 |
| cj_external_deps | cangjie_ark_interop:ohos.labels | 标签注解类 |
| cj_deps | ohos.hilog | 日志模块 |
| external_deps | hiappevent:cj_hiappevent_ffi | FFI 库 |

---

## ohos/hiviewdfx (聚合)

### BUILD.gn

**文件位置**: `ohos/hiviewdfx/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.hiviewdfx") {

  if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.hiviewdfx.cj" ]
  } else {
    sources = [ "hiviewdfx.cj" ]
  }

  subsystem_name = "hiviewdfx"
  part_name = "hiviewdfx_cangjie_wrapper"
}
```

**数据来源**: `ohos/hiviewdfx/BUILD.gn:16-29`

### Target 信息

| 属性 | 值 |
|------|-----|
| Target 类型 | ohos_cangjie_shared_library |
| Target 名称 | ohos.hiviewdfx |
| Sources | hiviewdfx.cj (或 mock) |

---

## bundle.json 组件配置

**文件位置**: `bundle.json`

```json
{
    "component": {
        "name": "hiviewdfx_cangjie_wrapper",
        "subsystem": "hiviewdfx",
        "adapted_system_type": ["standard"],
        "rom": "400KB",
        "ram": "420KB",
        "deps": {
            "components": [
                "cangjie_ark_interop",
                "hiappevent",
                "hilog",
                "hitrace"
            ]
        },
        "build": {
            "sub_component": [
                "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hilog:ohos.hilog",
                "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hiviewdfx:ohos.hiviewdfx",
                "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hiviewdfx/hi_app_event:ohos.hiviewdfx.hi_app_event",
                "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/ohos/hi_trace_meter:ohos.hi_trace_meter",
                "//base/hiviewdfx/hiviewdfx_cangjie_wrapper/kit/PerformanceAnalysisKit:kit.PerformanceAnalysisKit"
            ]
        }
    }
}
```

**数据来源**: `bundle.json:12-37`

---

## Targets 汇总

| Target | 类型 | 输出名 | 主要依赖 |
|--------|------|--------|---------|
| kit.PerformanceAnalysisKit | shared_library | libmodule.z.so | hi_trace_meter, hilog, hi_app_event |
| ohos.hilog | shared_library | libmodule.z.so | libhilog |
| ohos.hi_trace_meter | shared_library | libmodule.z.so | cj_hitracemeter_ffi |
| ohos.hiviewdfx.hi_app_event | shared_library | libmodule.z.so | cj_hiappevent_ffi |
| ohos.hiviewdfx | shared_library | libmodule.z.so | 无 |
| copy_sdk_hiviewdfx_cangjie_libs | copy | - | - |

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题 |
