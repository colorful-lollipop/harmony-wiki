# heck - 大小写转换库

> OpenHarmony 第三方库文档 | `third_party/rust/crates/heck`

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 库概述与功能简介 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 在 OH 中的依赖关系与使用场景 |

## 快速概览

**heck** 是一个 Rust 字符串大小写转换库，支持 8 种常见命名规范：

- ✅ `UpperCamelCase` / `PascalCase`
- ✅ `snake_case` / `snek_case`
- ✅ `kebab-case`
- ✅ `lowerCamelCase`
- ✅ `SHOUTY_SNAKE_CASE`
- ✅ `Title Case`
- ✅ `SHOUTY-KEBAB-CASE`
- ✅ `Train-Case`

## OpenHarmony 适配要点

| 项目 | 说明 |
|------|------|
| **OH 组件名** | @ohos/rust_heck |
| **所属子系统** | thirdparty |
| **版本** | 0.4.1 |
| **Patch 数量** | **0**（无需任何 Patch） |
| **依赖者** | clap_derive（命令行解析宏） |
| **特殊配置** | 无（标准 ohos_cargo_crate 构建） |

## 为何无需 Patch？

heck 库在 OpenHarmony 中**无需任何定制化修改**，原因如下：

1. **纯算法实现**：仅进行字符串转换，无平台相关代码
2. **零依赖**：不依赖其他 crate
3. **全 Safe Rust**：使用 `#![forbid(unsafe_code)]` 禁止 unsafe
4. **标准库 API**：仅使用 Rust 标准库的字符串处理功能

## 许可证

- Apache License 2.0
- MIT License

双许可，与上游保持一致。

---

**文档生成时间**: 2025-02-08  
**评估版本**: heck v0.4.1  
**OH 版本**: 6.1
