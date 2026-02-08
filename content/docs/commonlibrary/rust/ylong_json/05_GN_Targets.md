# ylong_json GN Targets

## 目的

本文档详细描述 `ylong_json` 的 GN 构建配置和 targets。

## 适用范围

- 构建系统开发者
- 需要集成 ylong_json 的开发者
- 系统架构师

## 构建文件位置

- **主构建文件**: `BUILD.gn`
- **组件配置**: `bundle.json`

## BUILD.gn 详解

### 文件内容

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ... (license header)

import("//build/ohos.gni")
import("//build/test.gni")

ohos_rust_shared_library("lib") {
  crate_name = "ylong_json"
  crate_type = "dylib"
  crate_root = "src/lib.rs"

  subsystem_name = "commonlibrary"
  part_name = "ylong_json"

  sources = [ "src/lib.rs" ]

  deps = [ "//third_party/rust/crates/serde/serde:lib" ]

  external_deps = [ "rust_libc:lib" ]

  features = [
    "default",
    "vec_array",
    "btree_object",
    "ascii_only",
  ]
}

ohos_rust_unittest("rust_ylong_json_unit_test") {
  module_out_path = "ylong_json/ylong_json"
  sources = [ "src/lib.rs" ]
  deps = [
    ":lib",
    "//third_party/rust/crates/serde/serde:lib",
  ]

  external_deps = [ "rust_libc:lib" ]

  if (defined(global_parts_info) &&
      !defined(global_parts_info.third_party_rust_serde)) {
    deps += [ "//third_party/rust/crates/serde/serde:lib" ]
  } else {
    external_deps += [ "rust_serde:lib" ]
  }

  rustflags = [
    "--cfg=feature=\"default\"",
    "--cfg=feature=\"vec_array\"",
    "--cfg=feature=\"btree_object\"",
    "--cfg=feature=\"ascii_only\"",
  ]
}

group("unittest") {
  testonly = true
  deps = []
  if (!use_clang_coverage) {
    deps += [ ":rust_ylong_json_unit_test" ]
  }
}
```

**证据**: `BUILD.gn:1-71`

## Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `lib` | ohos_rust_shared_library | `.so` 动态库 | 核心库 |
| `rust_ylong_json_unit_test` | ohos_rust_unittest | 测试可执行文件 | 单元测试 |
| `unittest` | group | - | 测试分组 |

### Target: lib

**类型**: `ohos_rust_shared_library`

**属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| `crate_name` | "ylong_json" | Rust crate 名称 |
| `crate_type` | "dylib" | 动态库 |
| `crate_root` | "src/lib.rs" | crate 根文件 |
| `subsystem_name` | "commonlibrary" | 子系统名称 |
| `part_name` | "ylong_json" | 部件名称 |

**依赖**:

| 依赖类型 | 目标 | 说明 |
|----------|------|------|
| `deps` | `//third_party/rust/crates/serde/serde:lib` | serde 库 |
| `external_deps` | `rust_libc:lib` | libc 绑定 |

**Features**:

```gn
features = [
  "default",       # 默认 feature
  "vec_array",     # Array 使用 Vec 实现
  "btree_object",  # Object 使用 BTreeMap 实现
  "ascii_only",    # 仅 ASCII 字符输出
]
```

**说明**:
- 默认启用 `btree_object` 和 `vec_array`
- 未启用 `c_adapter` feature（C FFI 接口默认不包含）
- 输出动态库 `.so` 文件

### Target: rust_ylong_json_unit_test

**类型**: `ohos_rust_unittest`

**属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| `module_out_path` | "ylong_json/ylong_json" | 输出路径 |
| `sources` | `["src/lib.rs"]` | 源文件 |

**依赖**:

| 依赖类型 | 目标 | 说明 |
|----------|------|------|
| `deps` | `:lib` | 主库 |
| `deps` | `//third_party/rust/crates/serde/serde:lib` | serde 库 |
| `external_deps` | `rust_libc:lib` | libc 绑定 |

**条件依赖**:

```gn
if (defined(global_parts_info) &&
    !defined(global_parts_info.third_party_rust_serde)) {
  deps += [ "//third_party/rust/crates/serde/serde:lib" ]
} else {
  external_deps += [ "rust_serde:lib" ]
}
```

**Rustflags**:

```gn
rustflags = [
  "--cfg=feature=\"default\"",
  "--cfg=feature=\"vec_array\"",
  "--cfg=feature=\"btree_object\"",
  "--cfg=feature=\"ascii_only\"",
]
```

### Target: unittest

**类型**: `group`

**说明**:
- 测试分组 target
- `testonly = true` 表示仅用于测试
- 条件编译：非覆盖率测试时包含单元测试

```gn
group("unittest") {
  testonly = true
  deps = []
  if (!use_clang_coverage) {
    deps += [ ":rust_ylong_json_unit_test" ]
  }
}
```

## bundle.json 详解

### 组件配置

```json
{
  "name": "@ohos/ylong_json",
  "version": "4.0",
  "description": "Serialization and deserialization for JSON.",
  "publishAs": "code-segment",
  "homePage": "https://gitcode.com/openharmony",
  "repository": "https://gitcode.com/openharmony/commonlibrary_rust_ylong_json",
  "license": "Apache License 2.0",
  "language": "",
  "segment": {
    "destPath": "commonlibrary/rust/ylong_json"
  },
  "private": false,
  "scripts": {},
  "envs": [],
  "dirs": [],
  "author": {},
  "contributors": [],
  "component": {
    "name": "ylong_json",
    "subsystem": "commonlibrary",
    "features": [],
    "adapted_system_type": ["standard"],
    "rom": "200KB",
    "ram": "~200KB",
    "deps": {
      "components": ["rust_libc"],
      "third_party": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "header": {
            "header_base": [],
            "header_files": []
          },
          "name": "//commonlibrary/rust/ylong_json:lib"
        }
      ],
      "test": ["//commonlibrary/rust/ylong_json:unittest"]
    }
  }
}
```

**证据**: `bundle.json:1-50`

### 关键字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | "@ohos/ylong_json" | 包名 |
| `version` | "4.0" | 版本 |
| `component.name` | "ylong_json" | 组件名 |
| `component.subsystem` | "commonlibrary" | 子系统 |
| `component.adapted_system_type` | ["standard"] | 适配标准系统 |
| `component.rom` | "200KB" | ROM 占用 |
| `component.ram` | "~200KB" | RAM 占用 |
| `component.deps.components` | ["rust_libc"] | 依赖组件 |
| `component.build.inner_kits` | `//commonlibrary/rust/ylong_json:lib` | 内部接口 |
| `component.build.test` | `//commonlibrary/rust/ylong_json:unittest` | 测试目标 |

## Feature Flags 对比

### Cargo.toml vs BUILD.gn

| Feature | Cargo.toml 默认 | BUILD.gn 配置 | 说明 |
|---------|----------------|---------------|------|
| `c_adapter` | ❌ | ❌ | C FFI 接口 |
| `btree_object` | ✅ | ✅ | Object BTree 实现 |
| `vec_array` | ✅ | ✅ | Array Vec 实现 |
| `ascii_only` | ❌ | ✅ | 仅 ASCII 输出 |
| `list_object` | ❌ | ❌ | Object LinkedList 实现 |
| `list_array` | ❌ | ❌ | Array LinkedList 实现 |
| `vec_object` | ❌ | ❌ | Object Vec 实现 |

**注意**: BUILD.gn 额外启用了 `ascii_only` feature。

## 使用方式

### 在 GN 中依赖 ylong_json

```gn
# 在 BUILD.gn 中添加依赖
external_deps = [ "ylong_json:lib" ]
```

### 在 bundle.json 中声明依赖

```json
{
  "component": {
    "deps": {
      "components": ["ylong_json"]
    }
  }
}
```

## 输出产物

### lib target

| 产物 | 类型 | 路径（示例） |
|------|------|-------------|
| `libylong_json.so` | 动态库 | `out/.../commonlibrary/rust/ylong_json/libylong_json.so` |

### 测试 target

| 产物 | 类型 | 说明 |
|------|------|------|
| `rust_ylong_json_unit_test` | 可执行文件 | 单元测试二进制 |

## 关键配置说明

### 为什么 crate_type 是 dylib？

- Rust 动态库可被其他 Rust 代码动态链接
- 与 C 动态库（cdylib）不同，dylib 保留 Rust 元数据
- 适合 Rust 组件间的动态链接

### 为什么未启用 c_adapter？

- `c_adapter` feature 启用 C FFI 接口
- 当前 BUILD.gn 配置未启用，意味着：
  - 不生成 C 头文件
  - C 代码无法直接调用
- 如需 C 接口，需要修改 BUILD.gn 添加 `c_adapter` feature

## 相关跳转

- [编译产物](06_Build_Artifacts.md) - 输出文件详情
- [配置选项](appendix/Config_Flags.md) - Feature flags 详细说明
- [架构说明](01_Architecture.md) - 架构设计
