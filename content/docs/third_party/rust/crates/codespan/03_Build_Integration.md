# OpenHarmony 构建适配

## 3.1 构建系统概述

OpenHarmony 使用 GN（Generate Ninja）作为其构建系统，对于 Rust crates，则通过 `ohos_cargo_crate` 模板进行集成。codespan-reporting 作为 third_party 目录下的 Rust 库，遵循标准的 OH Rust 库集成模式。

### 3.1.1 构建模板说明

codespan-reporting 使用 `ohos_cargo_crate` 模板进行构建，该模板是 OpenHarmony 为 Rust crates 提供的标准化构建入口。

```gn
import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "codespan_reporting"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.11.1"
    cargo_pkg_authors = "Brendan Zabarauskas <bjzaba@yahoo.com.au>"
    cargo_pkg_name = "codespan-reporting"
    cargo_pkg_description = "Beautiful diagnostic reporting for text-based programming languages"
    
    deps = [
        "//third_party/rust/crates/termcolor:lib",
        "//third_party/rust/crates/unicode-width:lib",
    ]
    
    module_output_extension = ".rlib"
    part_name = "rust_codespan"
    subsystem_name = "thirdparty"
}
```

### 3.1.2 关键配置参数

| 参数 | 值 | 说明 |
|------|-----|------|
| crate_name | codespan_reporting | Rust 库内部名称（下划线格式） |
| crate_type | rlib | Rust 静态库类型，用于静态链接 |
| crate_root | src/lib.rs | 库入口文件路径 |
| edition | 2018 | Rust Edition 版本 |
| cargo_pkg_version | 0.11.1 | 对应上游版本 |
| module_output_extension | .rlib | 输出文件扩展名 |
| part_name | rust_codespan | OH 组件分区名称 |
| subsystem_name | thirdparty | 所属子系统 |

## 3.2 依赖配置

### 3.2.1 直接依赖

codespan-reporting 在 OH 中的直接依赖如下：

| 依赖库 | OH 路径 | 用途 |
|--------|---------|------|
| termcolor | //third_party/rust/crates/termcolor:lib | 终端颜色输出支持 |
| unicode-width | //third_party/rust/crates/unicode-width:lib | Unicode 字符宽度计算 |

### 3.2.2 依赖关系图

```
codespan-reporting (rlib)
├── termcolor (rlib)
│   └── 系统依赖：libc
└── unicode-width (rlib)
    └── 无额外依赖
```

### 3.2.3 与上游依赖的对比

| 依赖类型 | 上游配置 | OH 配置 |
|----------|----------|----------|
| termcolor | crates.io 版本 | //third_party/rust/crates/termcolor:lib |
| unicode-width | crates.io 版本 | //third_party/rust/crates/unicode-width:lib |
| serde | 可选依赖，未启用 | 与上游一致 |

**说明**：OH 中的依赖配置与上游完全一致，只是将 crates.io 的版本依赖替换为 OH 内部的路径依赖。

## 3.3 构建产物

### 3.3.1 输出文件

构建完成后，生成的产物包括：

| 文件类型 | 路径模式 | 说明 |
|----------|----------|------|
| 静态库 | out/.../obj/third_party/rust/crates/codespan/codespan-reporting/src/lib.rlib | Rust 静态库文件 |
| 归档信息 | out/.../build.ninja | 构建依赖记录 |

### 3.3.2 组件注册

codespan-reporting 在 OH 组件系统中的注册信息：

```json
{
  "name": "@ohos/rust_codespan",
  "description": "A Rust library that provides support for source code display and formatting.",
  "version": "6.1",
  "license": "Apache License 2.0",
  "component": {
    "name": "rust_codespan",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/codespan/codespan-reporting:lib"
        }
      ]
    }
  }
}
```

## 3.4 与上游构建的差异

### 3.4.1 构建系统差异

| 方面 | 上游构建 | OH 构建 |
|------|----------|----------|
| 构建工具 | Cargo | GN + Ninja + Cargo |
| 配置文件 | Cargo.toml | BUILD.gn + Cargo.toml |
| 依赖解析 | crates.io | OH third_party 目录 |
| 输出位置 | target/ | out/.../obj/ |

### 3.4.2 配置映射关系

| Cargo 配置 | GN 配置 | 说明 |
|------------|---------|------|
| name | crate_name | 库名称转换 |
| version | cargo_pkg_version | 版本声明 |
| edition | edition | Rust Edition |
| dependencies | deps | 依赖列表 |
| features | 无 | OH 未启用可选特性 |

### 3.4.3 OH 特有的配置

**part_name 和 subsystem_name**：这两个配置是 OH 特有的，用于将 Rust 库纳入 OH 的组件化管理体系：

- **part_name**：定义库所属的分区名称（rust_codespan）
- **subsystem_name**：定义所属的子系统名称（thirdparty）

## 3.5 构建验证

### 3.5.1 构建命令

```bash
# 在 OH 根目录下执行
hb build -p rust_codespan

# 或直接使用 ninja
ninja out/.../third_party/rust/crates/codespan/codespan-reporting/src/lib.rlib
```

### 3.5.2 测试验证

该库本身没有直接的单元测试，但通过其使用者（cxx/gen/cmd）进行集成验证：

```bash
# 构建 cxx/gen/cmd 模块
hb build -p cxx

# 运行测试
# 验证诊断输出功能是否正常
```

### 3.5.3 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 依赖找不到 | termcolor 或 unicode-width 未构建 | 先构建相关依赖 |
| 版本不兼容 | Cargo.lock 冲突 | 清理构建缓存后重试 |
| 编译错误 | Rust 版本不兼容 | 使用 OH 推荐的 Rust 版本 |

## 3.6 版本管理

### 3.6.1 版本对应关系

| OH 版本 | codespan 版本 | 说明 |
|---------|---------------|------|
| 6.1 | 0.11.1 | 当前使用的版本 |

### 3.6.2 版本升级流程

1. 更新 Cargo.toml 中的版本号
2. 更新 BUILD.gn 中的 cargo_pkg_version
3. 更新 bundle.json 中的 version
4. 验证依赖兼容性
5. 运行集成测试

**建议**：升级前先检查上游的 Release Notes，确认是否有破坏性变更。
