# cxx - Rust 与 C++ 安全 FFI 库

## 库概述

**cxx** 是 OpenHarmony `third_party` 中的核心 Rust 库，提供 Rust 与 C++ 之间的安全外部函数接口（FFI）。该库通过代码生成和静态分析机制，使开发者能够在两种语言之间进行类型安全的互操作，无需依赖不安全的 C 风格绑定。

**上游信息**：
- 版本：1.0.130
- 许可证：Apache-2.0 / MIT
- 上游地址：https://github.com/dtolnay/cxx

**OH 适配版本**：5.0
**所属子系统**：thirdparty
**组件名称**：@ohos/rust_cxx

## 核心功能

cxx 的核心价值在于提供一种**类型安全**的 Rust/C++ 互操作方案。与传统的 `bindgen`/`cbindgen` 方案相比，cxx 通过以下机制确保安全性：

1. **统一的 FFI 边界定义**：使用 `#[cxx::bridge]` 属性宏集中声明共享类型和函数
2. **双向类型映射**：自动处理 Rust 标准库类型与 C++ 标准库类型之间的转换
3. **内存安全管理**：自动处理 `Box`/`unique_ptr`、`Vec`/`vector` 等智能指针的跨语言传递
4. **ABI 兼容性保证**：通过静态断言验证类型布局和调用约定

## OpenHarmony 适配特点

该库在 OpenHarmony 中的适配具有以下特点：

### 1. 无 Patch 依赖

cxx 库**没有**应用任何 OH 特定的代码 Patch。这意味着：

- 上游版本可直接集成，维护成本低
- 升级上游版本时冲突风险小
- 依赖 OH 构建系统（GN）进行平台适配

### 2. 构建系统适配

通过 `BUILD.gn` 文件完成 OH 构建系统集成：

- **Rust 库** (`lib`)：提供 `rlib` 静态库格式的 Rust 接口
- **C++ 运行库** (`cxx_cppdeps`)：提供 C++ 侧的类型转换和内存管理支持
- **过程宏** (`macro_lib`)：提供代码生成宏，仅对 Rust crates 可见
- **代码生成工具** (`cxxbridge`)：命令行工具，支持非 Cargo 构建系统

### 3. 关键配置

```gn
# 禁用 C++ 异常处理（符合 OH 整体策略）
defines = ["RUST_CXX_NO_EXCEPTIONS"]

# 平台相关的符号导出控制
if (is_win) {
  defines += ["CXX_RS_EXPORT=__declspec(dllexport)"]
} else {
  defines += ["CXX_RS_EXPORT=__attribute__((visibility(\"default\")))"]
}
```

## 提供给 OH 的组件

cxx 向 OH 系统提供以下 Inner Kits：

| 组件 | 类型 | 用途 |
|-----|------|-----|
| `//third_party/rust/crates/cxx:cxx_cppdeps` | 静态库 | C++ 运行时支持 |
| `//third_party/rust/crates/cxx:lib` | rlib | Rust 核心库 |
| `//third_party/rust/crates/cxx/macro:macro_lib` | 过程宏 | 代码生成宏 |
| `//third_party/rust/crates/cxx/gen/cmd:cxxbridge` | 二进制 | 代码生成工具 |

## 依赖关系

### 上游依赖

- `cxxbridge-macro`：代码生成宏实现
- `link-cplusplus`：C++ 编译器链接配置
- `cxxbridge-flags`：C++ 标准版本标志

### OH 依赖

cxx 本身被以下 OH 模块间接依赖，用于支持 Rust/C++ 互操作：

- `serde`：Rust 序列化库的 C++ 互操作支持
- `unicode-ident`：Unicode 标识符处理
- 其他需要 FFI 能力的 Rust crates

## 文档导航

| 文档 | 内容 |
|-----|------|
| [README.md](README.md) | 本文档，库概览和导航 |
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 |
| [01_Overview.md](01_Overview.md) | 原始库详细介绍 |
| [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配详解 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 中的使用方式和依赖关系 |

## 版本信息

| 版本 | 日期 | 变更 |
|-----|------|-----|
| 5.0 | - | OH 适配版本 |
| 1.0.130 | - | 上游版本 |
