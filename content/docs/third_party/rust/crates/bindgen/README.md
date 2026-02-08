# bindgen - OpenHarmony 适配文档

## 库概览

| 属性 | 值 |
|-----|-----|
| **库名称** | bindgen |
| **上游版本** | 0.70.1 |
| **上游地址** | https://github.com/rust-lang/rust-bindgen |
| **许可证** | BSD-3-Clause-LBNL |
| **OH 组件名** | @ohos/rust_bindgen |
| **OH 子系统** | thirdparty |
| **适配类型** | 原生集成（无 Patch） |

## 功能简介

bindgen 是一个自动 FFI 绑定生成工具，能够将 C/C++ 头文件转换为 Rust 代码，使 Rust 代码可以无缝调用 C/C++ 库。

### 在 OpenHarmony 中的作用

1. **Rust/C++ 互操作基础设施**: 为 OH 中 Rust 组件调用 C/C++ 库提供自动化绑定生成
2. **ANI (Ark Native Interface) 支持**: 为 ArkTS 运行时与原生代码交互提供绑定生成能力
3. **构建时工具**: 作为 host 工具链组件，在构建阶段生成绑定代码

## OH 适配概述

### 适配策略: 构建系统层封装

OpenHarmony 未对 bindgen 源码进行修改，而是通过以下方式完成适配：

1. **BUILD.gn 配置**: 将 Cargo 构建系统映射到 GN 构建系统
2. **rust_bindgen.gni 模板**: 提供声明式绑定生成接口
3. **rust_bindgen.py 包装器**: 处理 OH 特有的 clang 参数和环境

### 与上游的主要差异

| 方面 | 上游 | OpenHarmony |
|-----|------|-------------|
| 构建系统 | Cargo | GN + ohos_cargo_crate |
| 调用方式 | 命令行 / 库 API | rust_bindgen() GN 模板 |
| clang 集成 | 自动检测 | 显式配置 LLVM_CONFIG_PATH/CLANG_PATH |
| 参数处理 | 直接传递 | 过滤 OH 特有 clang 插件参数 |

## 文档导航

- [01_Overview.md](01_Overview.md) - 原始库简介与 OH 定位
- [02_Patches.md](02_Patches.md) - Patch 分析（本库无 Patch）
- [03_Build_Integration.md](03_Build_Integration.md) - OH 构建系统适配详解
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的依赖关系与使用场景
- [05_API_Differences.md](05_API_Differences.md) - API/接口差异（本库无差异）
- [06_Security.md](06_Security.md) - 安全风险分析

## 快速参考

### 在 BUILD.gn 中使用 bindgen

```gn
import("//build/ohos.gni")  # 自动导入 rust_bindgen.gni

rust_bindgen("my_bindings") {
  header = "my_header.h"
}

ohos_rust_static_library("my_lib") {
  deps = [ ":my_bindings" ]
  sources = [ "src/lib.rs" ]
  bindgen_output = get_target_outputs(":my_bindings")
  inputs = bindgen_output
  rustenv = [ "BINDINGS_RS_FILE=" + rebase_path(bindgen_output[0]) ]
}
```

### 生成绑定文件的使用

```rust
// 在 lib.rs 中包含生成的绑定
include!(env!("BINDINGS_RS_FILE"));

pub use self::bindings::*;
```

## 维护状态

- **最后更新**: 2024年 (升级至 0.70.1)
- **维护者**: fangting12@huawei.com
- **已知问题**: BUILD.gn 中版本号 (0.64.0) 与实际代码版本 (0.70.1) 不一致
