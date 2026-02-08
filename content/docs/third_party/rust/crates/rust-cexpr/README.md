# rust-cexpr

> C 表达式解析器和求值器在 OpenHarmony 中的集成文档

## 快速概览

| 属性 | 内容 |
|------|------|
| **库名称** | rust-cexpr (crate 名: cexpr) |
| **版本** | 0.6.0 |
| **许可证** | Apache-2.0 / MIT |
| **上游地址** | https://github.com/jethrogb/rust-cexpr |
| **OH 组件名** | rust_rust_cexpr |
| **所属子系统** | thirdparty |

### 一句话描述

rust-cexpr 是 bindgen 的基础设施库，用于解析 C/C++ 头文件中的宏定义表达式，将 `#define` 常量转换为 Rust 可用的数值。

## 本文档集重点

本文档集专注于 **OpenHarmony 对该库的集成与使用**，而非重复上游文档。

### 关键特点

🔹 **无 Patch 的纯净库** - 该库在 OH 中保持上游原样，无任何本地修改  
🔹 **bindgen 专用** - 当前仅 bindgen 依赖此库  
🔹 **维护成本低** - 无本地修改意味着升级简单  

## 文档导航

### 必读文档

| 文档 | 内容 | 推荐阅读顺序 |
|------|------|-------------|
| [01_Overview.md](./01_Overview.md) | 库简介、功能说明、OH 定位 | 1️⃣ |
| [02_Patches.md](./02_Patches.md) | Patch 分析（无 Patch 的说明） | 2️⃣ |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配 | 3️⃣ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | bindgen 中的使用场景 | 4️⃣ |

### 补充文档

| 文档 | 内容 |
|------|------|
| [05_API_Differences.md](./05_API_Differences.md) | API 差异（无差异） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速参考

### 依赖关系

```mermaid
graph LR
    A[应用] --> B[bindgen]
    B --> C[rust-cexpr]
    C --> D[nom]
```

### 构建配置

```gn
# BUILD.gn
ohos_cargo_crate("lib") {
  crate_name = "cexpr"
  crate_type = "rlib"
  deps = [ "//third_party/rust/crates/nom:lib" ]
}
```

### 核心使用场景

```rust
// bindgen 中使用 cexpr 解析宏
use cexpr::expr::EvalResult;
use cexpr::literal::CChar;

// 解析 #define VALUE 42
let result: EvalResult = ...;
match result {
    EvalResult::Int(Wrapping(v)) => println!("Integer: {}", v),
    EvalResult::Str(s) => println!("String: {:?}", s),
    _ => {}
}
```

## 维护信息

- **维护者**: xuelei3@huawei.com
- **OH 版本**: 6.1
- **最后评估**: 2026-02-07

---

*本文档是 OpenHarmony 第三方库 Wiki 的一部分*
