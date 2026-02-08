# memoffset Wiki

## 库概述

memoffset 是一个 Rust 库，提供类似 C 语言 `offsetof` 的功能，用于计算结构体成员的内存偏移量。该库在 OpenHarmony 中主要用于支持 nix 和 rustix 等 Rust crates 进行底层系统调用绑定。

| 属性 | 值 |
|------|-----|
| 库名称 | memoffset |
| 版本 | v0.9.1 |
| 许可证 | Apache License V2.0 |
| 上游地址 | https://github.com/Gilnaa/memoffset |
| OH 组件 | @ohos/rust_memoffset |
| OH 版本 | 6.1 |

## OpenHarmony 适配状态

### Patch 状态

**该库无需任何 OpenHarmony 特定 Patch**

memoffset 是一个纯宏实现的跨平台库，其核心功能与操作系统无关。在 OpenHarmony 中无需任何修改即可正常工作。

### 适配复杂度

**低** - 只需基本的 BUILD.gn 构建配置

## 文档导航

### 快速入门

| 场景 | 文档 |
|------|------|
| 了解库的基本功能 | [01_Overview](01_Overview.md) |
| 查看 Patch 分析 | [02_Patches](02_Patches.md) |
| 了解构建配置 | [03_Build_Integration](03_Build_Integration.md) |
| 查看 OH 使用情况 | [04_Usage_in_OH](04_Usage_in_OH.md) |
| 查看 API 差异 | [05_API_Differences](05_API_Differences.md) |
| 安全风险分析 | [06_Security](06_Security.md) |

### 阅读路线建议

| 读者类型 | 推荐阅读顺序 |
|---------|-------------|
| 维护者 | 01 → 02 → 03 → 04 → 05 → 06 |
| 开发者 | 01 → 04 → 03 |
| 安全审计 | 06 → 02 → 05 |

## 关键信息摘要

### 该库在 OH 中的作用

memoffset 作为底层支持库，主要通过以下方式被 OH 使用：

1. **间接依赖**：作为 nix 和 rustix crate 的依赖
2. **底层支持**：提供结构体偏移量计算，用于系统调用参数构造
3. **跨平台兼容**：确保不同架构下内存布局的一致性

### 主要依赖者

| 模块 | 用途 |
|------|------|
| nix | Unix 系统调用绑定（socket 功能） |
| rustix | 现代 Unix 系统调用绑定 |

### 构建方式

- **类型**：Rust 静态库（rlib）
- **构建模板**：ohos_cargo_crate
- **Rust Edition**：2015

## 版本信息

### 当前版本

- **上游版本**：v0.9.1
- **OH 版本**：6.1

### 版本兼容性

| Rust 版本 | 兼容性 | 说明 |
|-----------|--------|------|
| 1.19+ | ✓ 支持 | MSRV (Minimum Supported Rust Version) |
| 1.77+ | ✓ 推荐 | 可使用标准库 `offset_of!` |

## 相关资源

- [上游仓库](https://github.com/Gilnaa/memoffset)
- [Rust 标准库文档](https://doc.rust-lang.org/std/mem/fn.offset_of.html)
- [OH Rust crates 仓库](https://gitee.com/openharmony/third_party_rust)

## 维护指南

### 升级建议

1. **关注上游更新**：定期检查 memoffset 上游版本
2. **测试兼容性**：确保新版本在 OH 构建系统中正常工作
3. **考虑 Rust 1.77+**：Rust 1.77 稳定后，可评估是否需要继续维护

### 问题反馈

如发现适配问题，请联系：
- 维护者：fangting12@huawei.com
- 或提交 Issue 到 OpenHarmony 第三方库仓库
