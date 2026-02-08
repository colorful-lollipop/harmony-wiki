# 构建配置

> GN 构建系统配置、Targets 与编译产物说明

## 概述

`location_cangjie_wrapper` 使用 OpenHarmony 的 **GN（Generate Ninja）** 构建系统，Cangjie 代码采用 `cjc` 编译器模板进行编译。

### 构建环境

| 组件 | 版本/配置 |
|------|-----------|
| 构建系统 | GN + Ninja |
| Cangjie 编译器 | cjc |
| 目标系统 | ohos（OpenHarmony） |
| 设备类型 | standard |

## 根目录 BUILD.gn

**文件路径**：`BUILD.gn`

### 配置内容

```gn
import("//build/templates/cangjie/cjc.gni")

location_cangjie_wrapper_packages_ohos = [
  "//base/location/location_cangjie_wrapper/ohos/geo_location_manager:ohos.geo_location_manager"
]

location_cangjie_wrapper_packages_kit = [
  "//base/location/location_cangjie_wrapper/kit/LocationKit:kit.LocationKit",
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_location_cangjie_libs") {
  ohos_inputs = location_cangjie_wrapper_packages_ohos
  kit_inputs = location_cangjie_wrapper_packages_kit
}
```

### 配置说明

| 变量/Target | 类型 | 说明 |
|-------------|------|------|
| `location_cangjie_wrapper_packages_ohos` | 数组 | ohos 模块包列表 |
| `location_cangjie_wrapper_packages_kit` | 数组 | kit 模块包列表 |
| `copy_sdk_location_cangjie_libs` | copy_ohos_cangjie_sdk_api_lib | SDK 复制任务 |

---

## kit/LocationKit/BUILD.gn

**文件路径**：`kit/LocationKit/BUILD.gn`

### Target 定义

```gn
ohos_cangjie_shared_library("kit.LocationKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/geo_location_manager:ohos.geo_location_manager"
  ]

  subsystem_name = "location"
  part_name = "location_cangjie_wrapper"
}
```

### Target 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| target_type | ohos_cangjie_shared_library | Cangjie 共享库 |
| sources | ["index.cj"] | 源文件列表 |
| cj_deps | 内部 Cangjie 依赖 | ohos.geo_location_manager |
| subsystem_name | "location" | 子系统名 |
| part_name | "location_cangjie_wrapper" | 部件名 |

### 依赖关系

```
kit.LocationKit
    └── cj_deps: ohos.geo_location_manager
```

---

## ohos/geo_location_manager/BUILD.gn

**文件路径**：`ohos/geo_location_manager/BUILD.gn`

### Target 定义

```gn
ohos_cangjie_shared_library("ohos.geo_location_manager") {

  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.geo_location_manager.cj" ]
  } else {
    sources = [
      "geo_location_manager.cj",
      "geo_location_manager_common.cj",
      "geo_location_manager_ffi.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "location:cj_geolocationmanager_ffi" ]

  subsystem_name = "location"
  part_name = "location_cangjie_wrapper"
}
```

### Target 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| target_type | ohos_cangjie_shared_library | Cangjie 共享库 |
| sources | 条件编译 | Windows/Mac 用 Mock，真机用实际实现 |
| cj_external_deps | Cangjie 外部依赖 | cangjie_ark_interop, hiviewdfx_cangjie_wrapper |
| external_deps | Native 外部依赖 | location:cj_geolocationmanager_ffi |
| subsystem_name | "location" | 子系统名 |
| part_name | "location_cangjie_wrapper" | 部件名 |

### 条件编译

| 条件 | 源文件 |
|------|--------|
| `is_mingw \|\| is_mac` | `mock/ohos.geo_location_manager.cj` |
| 其他 | `geo_location_manager.cj`<br>`geo_location_manager_common.cj`<br>`geo_location_manager_ffi.cj` |

### 依赖关系

```
ohos.geo_location_manager
    ├── cj_external_deps:
    │   ├── cangjie_ark_interop:ohos.ffi
    │   ├── cangjie_ark_interop:ohos.labels
    │   ├── cangjie_ark_interop:ohos.business_exception
    │   └── hiviewdfx_cangjie_wrapper:ohos.hilog
    │
    └── external_deps:
        └── location:cj_geolocationmanager_ffi
```

---

## bundle.json 配置

**文件路径**：`bundle.json`

```json
{
  "name": "@ohos/location_cangjie_wrapper",
  "component": {
    "name": "location_cangjie_wrapper",
    "subsystem": "location",
    "adapted_system_type": ["standard"],
    "rom": "120KB",
    "ram": "108KB",
    "deps": {
      "components": [
        "cangjie_ark_interop",
        "hiviewdfx_cangjie_wrapper",
        "location"
      ]
    },
    "build": {
      "sub_component": [
        "//base/location/location_cangjie_wrapper/ohos/geo_location_manager:ohos.geo_location_manager",
        "//base/location/location_cangjie_wrapper/kit/LocationKit:kit.LocationKit"
      ],
      "inner_kits": [
        { "name": "//base/location/location_cangjie_wrapper:copy_sdk_location_cangjie_libs" },
        { "name": "//base/location/location_cangjie_wrapper:copy_sdk_location_cangjie_libs_kit" }
      ]
    }
  }
}
```

### 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | @ohos/location_cangjie_wrapper | npm 包名 |
| subsystem | location | 所属子系统 |
| adapted_system_type | ["standard"] | 适配系统类型 |
| rom | 120KB | ROM 占用 |
| ram | 108KB | RAM 占用 |
| deps.components | 依赖组件列表 | cangjie_ark_interop, hiviewdfx_cangjie_wrapper, location |
| build.sub_component | 子组件列表 | ohos.geo_location_manager, kit.LocationKit |
| build.inner_kits | 内部接口包 | SDK 复制任务 |

---

## Targets 汇总表

| Target 名称 | 类型 | 输出 | 依赖 | 职责 |
|-------------|------|------|------|------|
| `kit.LocationKit` | ohos_cangjie_shared_library | .so/.cjlib | ohos.geo_location_manager | Kit 层导出 |
| `ohos.geo_location_manager` | ohos_cangjie_shared_library | .so/.cjlib | location FFI | 核心实现 |
| `copy_sdk_location_cangjie_libs` | copy_ohos_cangjie_sdk_api_lib | SDK 文件 | sub_component | SDK 复制 |

---

## 编译产物

### 预期产物

| 产物类型 | 路径模式 | 说明 |
|----------|----------|------|
| Cangjie 库 | `out/.../libs/libcj_*.so` | 编译生成的共享库 |
| SDK 包 | `sdk/.../packages/` | 分发给开发者的 SDK |
| 符号表 | `out/.../.cj_symbols/` | 调试符号 |

### 产物加载关系

```
应用运行时
    │
    ├── load @ohos/location_cangjie_wrapper (Package)
    │
    ├── load kit.LocationKit (CJ Library)
    │       │
    │       └── deps: ohos.geo_location_manager
    │
    └── load ohos.geo_location_manager (CJ Library)
            │
            └── external_deps: location:cj_geolocationmanager_ffi
                    │
                    └── load liblocation.z.so (Native Library)
```

---

## 构建命令

### 全量构建

```bash
# 编译 location_cangjie_wrapper 及其依赖
hb set -p <product_name>
hb build -f --gn-args location_cangjie_wrapper=true
```

### 模块构建

```bash
# 仅构建 location_cangjie_wrapper
hb build //base/location/location_cangjie_wrapper/...
```

### SDK 构建

```bash
# 构建 SDK
hb build -f --build-sdk
```

---

## 构建问题排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 找不到 cjc 编译器 | cangjie_ark_interop 未构建 | 先构建 cangjie_ark_interop |
| FFI 绑定失败 | location 组件未构建 | 先构建 base_location |
| Mock 代码被使用 | 编译平台判断错误 | 检查 is_mingw/is_mac 配置 |

### 调试命令

```bash
# 查看构建配置
gn args out/<product> --list | grep location

# 查看依赖树
gn deps //base/location/location_cangjie_wrapper/ohos/geo_location_manager

# 强制重新构建
rm -rf out/<product>/location_cangjie_wrapper
hb build //base/location/location_cangjie_wrapper/...
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_Architecture](01_Architecture.md) | 系统架构 |
| [02_API_Reference](02_API_Reference.md) | API 参考 |
| [04_Security](04_Security.md) | 安全注意事项 |

---

## 构建配置证据

| 配置项 | 证据位置 |
|--------|----------|
| BUILD.gn 模板导入 | `BUILD.gn:14` |
| kit/LocationKit 构建 | `kit/LocationKit/BUILD.gn:19-28` |
| ohos/geo_location_manager 构建 | `ohos/geo_location_manager/BUILD.gn:18-41` |
| 条件编译逻辑 | `ohos/geo_location_manager/BUILD.gn:20-28` |
| cj_external_deps | `ohos/geo_location_manager/BUILD.gn:30-35` |
| external_deps | `ohos/geo_location_manager/BUILD.gn:37` |
| bundle.json 配置 | `bundle.json:1-48` |
