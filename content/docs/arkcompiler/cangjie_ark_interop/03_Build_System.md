# GN 构建系统

本文档描述 cangjie_ark_interop 的 GN 构建配置，包括 targets、依赖、编译选项等。

## 构建入口

| 文件 | 用途 |
|------|------|
| `BUILD.gn` (根目录) | SDK 模块导入和复制目标 |
| `ohos/BUILD.gn` | 核心模块构建配置 |
| `kit/CangjieKit/BUILD.gn` | Kit 接口导出 |

## 根构建配置 (BUILD.gn)

**位置**: `BUILD.gn:14-31`

```gn
import("//build/templates/cangjie/cjc.gni")

# SDK 模块列表
cangjie_ark_interop_sdk_modules = [
  "//arkcompiler/cangjie_ark_interop/ohos/ffi:ohos.ffi",
  "//arkcompiler/cangjie_ark_interop/ohos/encoding:ohos.encoding.json",
  "//arkcompiler/cangjie_ark_interop/ohos:ohos.ark_interop",
  "//arkcompiler/cangjie_ark_interop/ohos:ohos.ark_interop_helper",
  "//arkcompiler/cangjie_ark_interop/ohos/labels:ohos.labels",
  "//arkcompiler/cangjie_ark_interop/ohos/business_exception:ohos.business_exception",
  "//arkcompiler/cangjie_ark_interop/ohos/callback_invoke:ohos.callback_invoke"
]

# 复制目标
copy_ohos_cangjie_sdk_libs("copy_cangjie_ark_interop_libs") {
  ohos_inputs = cangjie_ark_interop_sdk_modules
  kit_inputs = [
    "//arkcompiler/cangjie_ark_interop/kit/CangjieKit:kit.CangjieKit"
  ]
}
```

---

## 模块构建配置 (ohos/BUILD.gn)

### utf16string (C++ 原生库)

**位置**: `ohos/BUILD.gn:17-45`

```gn
if (!build_ohos_sdk) {
  ohos_shared_library("utf16string") {
    include_dirs = [ "utf16string" ]
    sources = [
      "utf16string/utf16string.cpp",
      "utf16string/utf16string_cffi.cpp",
      "utf16string/utf16string_dfx.cpp",
    ]
    cflags = [
      "-std=c++17",
      "-Wno-gnu-zero-variadic-macro-arguments",
      "-fvisibility-inlines-hidden",
      "-fvisibility=hidden",
      "-fno-exceptions",
      "-fno-rtti",
      "-fmerge-all-constants",
      "-ffunction-sections",
      "-Wno-unused-private-field",
    ]
    if (current_os == "ohos") {
      cflags += [ "-fPIC" ]
    }
    innerapi_tags = [ "platformsdk" ]
    part_name = "cangjie_ark_interop"
    subsystem_name = "arkcompiler"
  }
}
```

**Target 属性**:
- 类型: `ohos_shared_library` (C++)
- 源码: 3 个 C++ 文件
- 编译选项: C++17, 无异常, 无 RTTI

---

### ohos.ark_interop (仓颉共享库)

**位置**: `ohos/BUILD.gn:48-94`

```gn
ohos_cangjie_shared_library("ohos.ark_interop") {
  sources = [
    "ark_interop/js_array_buffer.cj",
    "ark_interop/js_bigint.cj",
    "ark_interop/js_class.cj",
    "ark_interop/js_constants.cj",
    "ark_interop/js_exception.cj",
    "ark_interop/js_external.cj",
    "ark_interop/js_func.cj",
    "ark_interop/js_heap.cj",
    "ark_interop/js_idl_type.cj",
    "ark_interop/js_interop_type.cj",
    "ark_interop/js_module.cj",
    "ark_interop/js_promise.cj",
    "ark_interop/js_raw.cj",
    "ark_interop/js_runtime.cj",
    "ark_interop/js_symbol.cj",
    "ark_interop/js_type.cj",
    "ark_interop/jsarray.cj",
    "ark_interop/jscontext.cj",
    "ark_interop/jsobject.cj",
    "ark_interop/jsstring.cj",
    "ark_interop/slab.cj",
    "ark_interop/utf16_string.cj",
    "ark_interop/js_map.cj",
    "ark_interop/js_module_helper.cj",
    "ark_interop/api_config.cj",
    "ark_interop/kit_config.cj",
  ]

  external_deps = [ "napi:ark_interop" ]
  cj_deps = [
    "./labels:ohos.labels",
    "./business_exception:ohos.business_exception",
  ]

  cj_external_deps = [
    "hiviewdfx_cangjie_wrapper:ohos.hilog"
  ]

  deps = [
    ":utf16string",
  ]

  part_name = "cangjie_ark_interop"
  subsystem_name = "arkcompiler"
}
```

**Target 属性**:
- 类型: `ohos_cangjie_shared_library`
- 源码: 26 个 .cj 文件
- 仓颉依赖: `ohos.labels`, `ohos.business_exception`
- 外部依赖: `napi:ark_interop`, `hiviewdfx_cangjie_wrapper:ohos.hilog`
- 内部依赖: `utf16string`

---

### ohos.ark_interop_helper (仓颉共享库)

**位置**: `ohos/BUILD.gn:96-126`

```gn
ohos_cangjie_shared_library("ohos.ark_interop_helper") {
  sources = [
    "ark_interop_helper/ark_api_call.cj",
    "ark_interop_helper/ark_api_call_async.cj",
    "ark_interop_helper/ark_container_scope.cj",
    "ark_interop_helper/ark_interop_helper.cj",
    "ark_interop_helper/ark_system_obj.cj",
    "ark_interop_helper/console.cj",
    "ark_interop_helper/timer.cj",
  ]

  cj_deps = [
    ":ohos.ark_interop",
    "./business_exception:ohos.business_exception",
    "./ffi:ohos.ffi",
    "./labels:ohos.labels",
  ]

  cj_external_deps = [
    "arkui_cangjie_wrapper:ohos.base",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [
    "ability_runtime:abilitykit_native",
    "ability_runtime:ark_interop_helper_ffi",
  ]

  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}
```

**Target 属性**:
- 类型: `ohos_cangjie_shared_library`
- 源码: 7 个 .cj 文件
- 仓颉依赖: `ohos.ark_interop`, `ohos.ffi`, `ohos.business_exception`, `ohos.labels`
- 外部依赖: `ability_runtime:abilitykit_native`, `ability_runtime:ark_interop_helper_ffi`

---

### ohos.ark_interop_macro (仓颉宏库)

**位置**: `ohos/BUILD.gn:128-142`

```gn
ohos_cangjie_macro_library("ohos.ark_interop_macro") {
  sources = [
    "ark_interop_macro/ark_idl_check.cj",
    "ark_interop_macro/ark_idl_class.cj",
    "ark_interop_macro/ark_idl_comm.cj",
    "ark_interop_macro/ark_idl_declare.cj",
    "ark_interop_macro/ark_idl_define.cj",
    "ark_interop_macro/ark_idl_func.cj",
    "ark_interop_macro/ark_idl_interface.cj",
    "ark_interop_macro/ark_idl_enum.cj",
  ]
  compilation_config = "pipeline=NG"
  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}
```

**Target 属性**:
- 类型: `ohos_cangjie_macro_library` (编译时宏)
- 源码: 9 个 .cj 文件
- 编译配置: `pipeline=NG` (新一代编译管道)

---

## 子模块构建配置

### ohos.ffi

**位置**: `ohos/ffi/BUILD.gn:16-35`

```gn
ohos_cangjie_shared_library("ohos.ffi") {
  sources = [
    "array_convert.cj",
    "cj_common_lambda.cj",
    "ffi_callback.cj",
    "ffi_data.cj",
    "ffiat_cpackage.cj",
    "malloc_free.cj",
    "remote_data_lite.cj",
    "ret_data.cj",
    "utility.cj",
  ]

  cj_external_deps = [ "hiviewdfx_cangjie_wrapper:ohos.hilog" ]

  external_deps = [ "napi:cj_bind_ffi" ]

  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}
```

---

### ohos.encoding.json

**位置**: `ohos/encoding/BUILD.gn:16-42`

```gn
# 共享库
ohos_cangjie_shared_library("ohos.encoding.json") {
  sources = [
    "json/json_exception.cj",
    "json/json_object.cj",
    "json/json_value.cj",
    "json/native.cj",
    "json/parse_json.cj",
    "json/write_buffer.cj",
  ]
  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}

# 静态库
ohos_cangjie_static_library("ohos.json.static") {
  sources = [
    "json/json_exception.cj",
    "json/json_object.cj",
    "json/json_value.cj",
    "json/native.cj",
    "json/parse_json.cj",
    "json/write_buffer.cj",
  ]
  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}
```

---

## kit/CangjieKit 构建配置

**位置**: `kit/CangjieKit/BUILD.gn`

```gn
ohos_cangjie_shared_library("kit.CangjieKit") {
  sources = [
    "index.cj",
  ]

  cj_deps = [
    "../../ohos/business_exception:ohos.business_exception",
    "../../ohos/callback_invoke:ohos.callback_invoke",
    "../../ohos/ffi:ohos.ffi",
    "../../ohos:ohos.ark_interop",
    "../../ohos:ohos.ark_interop_helper"
  ]

  subsystem_name = "arkcompiler"
  part_name = "cangjie_ark_interop"
}
```

---

## 组件配置 (bundle.json)

**位置**: `bundle.json`

```json
{
  "name": "@ohos/cangjie_ark_interop",
  "version": "6.1",
  "component": {
    "name": "cangjie_ark_interop",
    "subsystem": "arkcompiler",
    "syscap": [ "SystemCapability.ArkCompiler.CangjieInterop" ],
    "adapted_system_type": [ "standard" ],
    "rom": "1024KB",
    "ram": "2046KB",
    "deps": {
      "components": [ "ability_runtime", "napi" ]
    }
  }
}
```

---

## Targets 汇总表

| Target 名称 | 类型 | 源码数 | 主要依赖 |
|-------------|------|--------|----------|
| `utf16string` | ohos_shared_library (C++) | 3 | 无 |
| `ohos.ark_interop` | ohos_cangjie_shared_library | 26 | napi:ark_interop, labels, business_exception |
| `ohos.ark_interop_helper` | ohos_cangjie_shared_library | 7 | ark_interop, ffi, ability_runtime |
| `ohos.ark_interop_macro` | ohos_cangjie_macro_library | 9 | 无 |
| `ohos.ffi` | ohos_cangjie_shared_library | 9 | napi:cj_bind_ffi |
| `ohos.encoding.json` | ohos_cangjie_shared_library | 6 | 无 |
| `ohos.json.static` | ohos_cangjie_static_library | 6 | 无 |
| `kit.CangjieKit` | ohos_cangjie_shared_library | 1 | ark_interop, ark_interop_helper |
