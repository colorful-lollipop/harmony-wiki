# unicode-width - OpenHarmony Wiki

> Unicode 字符宽度计算库在 OpenHarmony 中的集成与适配文档

## 库概览

| 属性 | 内容 |
|------|------|
| **库名称** | unicode-width |
| **上游版本** | v0.1.14 |
| **上游地址** | https://github.com/unicode-rs/unicode-width |
| **许可证** | Apache-2.0 OR MIT |
| **OH 组件名** | rust_unicode_width |
| **所属子系统** | thirdparty |

## 核心特点

### 功能定位
- **用途**：计算 Unicode 字符和字符串的**显示宽度**（terminal column width）
- **标准**：遵循 Unicode Standard Annex #11
- **特性**：支持 CJK（中日韩）上下文的特殊宽度计算
- **设计**：`#![no_std]` 无标准库依赖

### OH 适配特点
| 特点 | 说明 |
|------|------|
| **Patch 数量** | **0** - 无代码修改，完全使用上游源码 |
| **构建复杂度** | 低 - 标准 ohos_cargo_crate 配置 |
| **依赖范围** | 中 - 被诊断工具和 Rust 编译器组件依赖 |
| **维护难度** | 极低 - 无 OH 特定代码 |

## 为什么不需要 Patch？

unicode-width 在 OpenHarmony 中**不需要任何 Patch**，原因如下：

1. **纯算法库**：仅包含 Unicode 宽度计算算法，无平台相关代码
2. **no_std 设计**：本身设计为无标准库依赖，天然适合嵌入式和 OH 环境
3. **API 简洁**：提供稳定的 trait 接口，无需扩展
4. **单文件实现**：src/lib.rs 单一文件，复杂度和维护成本低

## 文档导航

### 快速阅读路线

**如果你是第一次了解此库：**
1. [01_Overview.md](./01_Overview.md) - 了解库的基本功能和 OH 定位
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解谁在使用此库以及使用场景

**如果你是维护人员：**
1. [02_Patches.md](./02_Patches.md) - Patch 分析（确认无 Patch）
2. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置说明
3. [06_Security.md](./06_Security.md) - 安全风险评估

**如果你是开发者想了解使用：**
1. [01_Overview.md](./01_Overview.md) - 功能介绍
2. 参考上游文档：https://docs.rs/unicode-width

## 文档清单

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、功能说明、OH 定位 |
| [02_Patches.md](./02_Patches.md) | **Patch 分析** - 本文档核心，说明无 Patch 的情况 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配、BUILD.gn 解析 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、依赖图 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异分析（与上游无差异） |
| [06_Security.md](./06_Security.md) | 安全风险分析、CVE、升级建议 |

## 关键信息速查

### 上游版本信息
```toml
# Cargo.toml
[package]
name = "unicode-width"
version = "0.1.14"
edition = "2021"
license = "MIT OR Apache-2.0"
```

### OH 构建配置
```gn
# BUILD.gn
ohos_cargo_crate("lib") {
    crate_name = "unicode_width"
    crate_type = "rlib"
    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "0.1.14"
}
```

### 主要依赖者
```
codespan-reporting  →  诊断报告美化打印
rustfmt              →  代码格式化
rustc_parse          →  编译器解析器
rustc_span           →  源码位置信息
rustc_errors         →  编译器错误处理
```

## 维护者须知

### 升级检查清单
- [ ] 上游版本是否更新
- [ ] API 是否保持兼容
- [ ] Cargo.toml 中的 features 是否有变更
- [ ] 更新 BUILD.gn 中的 `cargo_pkg_version`

### 变更记录
| 日期 | 版本 | 变更 |
|------|------|------|
| 2023 | v0.1.14 | 初始引入 OH，无 Patch |

---

**注意**：本文档专注于 OpenHarmony 相关的集成和适配信息。关于库的使用方法、API 文档等通用信息，请参考上游官方文档：https://docs.rs/unicode-width
