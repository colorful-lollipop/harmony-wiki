# link-cplusplus

> OpenHarmony Rust C++ 链接库

## 库概览

**link-cplusplus** 是一个 Rust 库，用于在链接阶段自动链接 C++ 标准库（libstdc++ 或 libc++）。

该库解决了 Rust 项目中 C++ 链接的标准化问题，使得：
- 应用可以统一选择使用 libstdc++ 或 libc++
- 库作者无需在编译时强制指定 C++ 库
- 下游应用可以覆盖上游库的链接选择

## OH 适配概述

| 项目 | 状态 |
|-----|------|
| **Patch 数量** | 0（干净的上游导入） |
| **OH 特有修改** | 仅 BUILD.gn 和 bundle.json 配置适配 |
| **构建适配** | 使用 ohos_cargo_crate 模板 |
| **安全风险** | 无已知漏洞 |

### 核心适配点

1. **BUILD.gn 配置**：使用 ohos_cargo_crate 模板封装上游 Cargo 项目
2. **bundle.json 定义**：OH 组件化配置
3. **源码保持**：上游源码完全保留，无任何 Patch

## 文档导航

### 必读文档

- [01_概述](01_概述.md)：库功能简介和 OH 定位
- [03_构建适配](03_构建适配.md)：BUILD.gn 配置说明

### 进阶文档

- [04_使用场景](04_使用场景.md)：在 OH Rust 项目中的使用方式
- [02_Patch 分析](02_Patch_分析.md)：Patch 清单（本库无 Patch）

## 快速开始

### OH 组件依赖

```json
{
  "name": "@ohos/rust_link_cplusplus",
  "version": "6.1",
  "part": "rust_link_cplusplus"
}
```

### Cargo.toml 使用

```toml
[dependencies]
link-cplusplus = "1"
```

## 版本信息

| 版本 | 来源 | 说明 |
|-----|------|------|
| 1.0.12 | 上游 | 原始版本 |
| 6.1 | OH | 组件化版本号 |

## 许可证

- Apache License 2.0
- MIT License

## 相关资源

- [上游仓库](https://github.com/dtolnay/link-cplusplus)
- [Cargo crate](https://crates.io/crates/link-cplusplus)
- [API 文档](https://docs.rs/link-cplusplus)

---

*最后更新：2026-02-08*
