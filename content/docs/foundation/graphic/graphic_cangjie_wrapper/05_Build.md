# 构建配置与产物 - graphic_cangjie_wrapper

## 构建系统

- **构建工具**: GN (Generate Ninja)
- **Cangjie 编译模板**: `//build/templates/cangjie/cjc.gni`
- **OHOS 构建模板**: `//build/ohos.gni`

> **证据**: `kit/ArkGraphics2D/BUILD.gn:16-17`, `ohos/graphics/BUILD.gn:16-17`

---

## Targets 清单

### kit.ArkGraphics2D

**路径**: `kit/ArkGraphics2D/BUILD.gn`
**类型**: `ohos_cangjie_shared_library`

```gn
ohos_cangjie_shared_library("kit.ArkGraphics2D") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/graphics/color_space_manager:ohos.graphics.color_space_manager"
  ]

  subsystem_name = "graphic"
  part_name = "graphic_cangjie_wrapper"
}
```

| 属性 | 值 | 证据来源 |
|-----|-----|---------|
| sources | `["index.cj"]` | `BUILD.gn:20` |
| cj_deps | `ohos.graphics.color_space_manager` | `BUILD.gn:22` |
| subsystem_name | `"graphic"` | `BUILD.gn:24` |
| part_name | `"graphic_cangjie_wrapper"` | `BUILD.gn:25` |

---

### ohos.graphics

**路径**: `ohos/graphics/BUILD.gn`
**类型**: `ohos_cangjie_shared_library`

```gn
ohos_cangjie_shared_library("ohos.graphics") {
  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.graphics.cj" ]
  } else {
    sources = [ "graphics.cj" ]
  }

  subsystem_name = "graphic"
  part_name = "graphic_cangjie_wrapper"
}
```

| 属性 | 值 | 条件 | 证据来源 |
|-----|-----|------|---------|
| sources | `graphics.cj` 或 mock | mingw/mac 使用 mock | `BUILD.gn:21-25` |
| subsystem_name | `"graphic"` | - | `BUILD.gn:27` |
| part_name | `"graphic_cangjie_wrapper"` | - | `BUILD.gn:28` |

---

### ohos.graphics.color_space_manager

**路径**: `ohos/graphics/color_space_manager/BUILD.gn`
**类型**: `ohos_cangjie_shared_library`

```gn
ohos_cangjie_shared_library("ohos.graphics.color_space_manager") {
  if (is_mingw || is_mac) {
    sources = [ "../../../mock/ohos.graphics.color_space_manager.cj" ]
  } else {
    sources = [
      "cj_color_manager_log.cj",
      "cj_color_manager_utils.cj",
      "color_space_manager.cj",
      "color_space_primaries.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "graphic_2d:cj_color_manager_ffi" ]

  subsystem_name = "graphic"
  part_name = "graphic_cangjie_wrapper"
}
```

| 属性 | 值 | evidence来源 |
|-----|-----|-------------|
| sources | 4个.cj文件（或 mock） | `BUILD.gn:23-29` |
| cj_external_deps | 4个 Cangjie 依赖 | `BUILD.gn:31-36` |
| external_deps | `graphic_2d:cj_color_manager_ffi` | `BUILD.gn:38` |
| subsystem_name | `"graphic"` | `BUILD.gn:40` |
| part_name | `"graphic_cangjie_wrapper"` | `BUILD.gn:41` |

---

### copy_sdk_graphic_cangjie_libs

**路径**: `BUILD.gn`
**类型**: `copy_ohos_cangjie_sdk_api_lib`（自定义模板）

```gn
copy_ohos_cangjie_sdk_api_lib("copy_sdk_graphic_cangjie_libs") {
  ohos_inputs = graphic_cangjie_wrapper_packages_ohos
  kit_inputs = graphic_cangjie_wrapper_packages_kit
}

graphic_cangjie_wrapper_packages_ohos = [
    "//foundation/graphic/graphic_cangjie_wrapper/ohos/graphics:ohos.graphics",
    "//foundation/graphic/graphic_cangjie_wrapper/ohos/graphics/color_space_manager:ohos.graphics.color_space_manager"
]
graphic_cangjie_wrapper_packages_kit = [
    "//foundation/graphic/graphic_cangjie_wrapper/kit/ArkGraphics2D:kit.ArkGraphics2D",
]
```

> **证据**: `BUILD.gn:16-27`

---

## 产物清单

| Target | 产物名 | 预计路径 | 说明 |
|-------|-------|---------|-----|
| kit.ArkGraphics2D | `libkit.ArkGraphics2D.so` | `out/.../system/lib/` | Kit 动态库 |
| ohos.graphics | `libohos.graphics.so` | `out/.../system/lib/` | 框架动态库 |
| ohos.graphics.color_space_manager | `libohos.graphics.color_space_manager.so` | `out/.../system/lib/` | 核心模块动态库 |
| copy_sdk_graphic_cangjie_libs | SDK 文件 | `out/.../sdk/` | SDK 复制任务 |

> **说明**: 产物路径为推导值，实际路径取决于构建配置

---

## 产物运行时加载关系

```
应用加载 libkit.ArkGraphics2D.so
    │
    ├──→ libohos.graphics.so (依赖)
    │         │
    │         └──→ libohos.graphics.color_space_manager.so (依赖)
    │                   │
    │                   └──→ libcj_color_manager_ffi.so (graphic_2d, native)
    │
    └──→ libohos.ffi.so (cangjie_ark_interop, 运行时)
    └──→ libohos.business_exception.so (cangjie_ark_interop, 运行时)
    └──→ libohos.hilog.so (hiviewdfx_cangjie_wrapper, 运行时)
```

---

## 组件配置 (bundle.json)

```json
{
  "component": {
    "name": "graphic_cangjie_wrapper",
    "subsystem": "graphic",
    "adapted_system_type": ["standard"],
    "features": [],
    "rom": "100KB",
    "ram": "80KB",
    "deps": {
      "components": [
        "cangjie_ark_interop",
        "graphic_2d",
        "hiviewdfx_cangjie_wrapper"
      ]
    },
    "build": {
      "sub_component": [
        "//foundation/graphic/graphic_cangjie_wrapper/kit/ArkGraphics2D:kit.ArkGraphics2D",
        "//foundation/graphic/graphic_cangjie_wrapper/ohos/graphics:ohos.graphics",
        "//foundation/graphic/graphic_cangjie_wrapper/ohos/graphics/color_space_manager:ohos.graphics.color_space_manager"
      ],
      "inner_kits": [
        { "name": "//foundation/graphic/graphic_cangjie_wrapper/ohos/graphics/color_space_manager:ohos.graphics.color_space_manager" },
        { "name": "//foundation/graphic/graphic_cangjie_wrapper:copy_sdk_graphic_cangjie_libs" },
        { "name": "//foundation/graphic/graphic_cangjie_wrapper:copy_sdk_graphic_cangjie_libs_kit" }
      ]
    }
  }
}
```

> **证据**: `bundle.json:12-48`

---

## 相关文档

- [Architecture](02_Architecture.md)
- [N-API](03_N-API.md)
- [Inner API](04_Inner_API.md)
