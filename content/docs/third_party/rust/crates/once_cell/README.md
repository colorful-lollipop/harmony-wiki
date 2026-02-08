# once_cell - OpenHarmony 集成文档

[![上游版本](https://img.shields.io/badge/upstream-1.17.0-blue.svg)](https://github.com/matklad/once_cell)
[![许可证](https://img.shields.io/badge/license-Apache%202.0%20OR%20MIT-green.svg)](LICENSE-APACHE)
[![OH 版本](https://img.shields.io/badge/OH-6.1-orange.svg)](bundle.json)

> once_cell 提供**单次赋值**的线程安全单元格 (cell) 和延迟初始化 (lazy initialization) 类型，用于 Rust 中优雅地管理全局状态和延迟计算。

---

## 快速导航

### 文档索引

| 文档 | 描述 | 适合人群 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 作用和定位 | 所有人 |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（4 个 OH 提交） | 想了解 OH 适配细节的开发者 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配（BUILD.gn、bundle.json） | 需要修改构建配置的开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用（6 个依赖者） | 需要使用 once_cell 的开发者 |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | 新手入门 |
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | Phase 0 项目评估结果 | 想了解分析过程的开发者 |

---

## 一句话总结

once_cell 在 OpenHarmony 中是**零侵入式集成**的典范：仅添加构建系统配置，不修改源代码，为 OH 的 Rust 基础设施提供全局状态管理能力。

---

## 核心信息

### 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | once_cell |
| **上游版本** | 1.17.0 |
| **OH 组件版本** | 6.1 |
| **许可证** | Apache License 2.0 OR MIT |
| **上游地址** | https://github.com/matklad/once_cell |
| **OH 组件名** | rust_once_cell |
| **子系统** | thirdparty |

### OH 适配特征

| 特征 | 状态 | 说明 |
|------|------|------|
| **源代码修改** | ✅ 无 | 与上游完全一致 |
| **Patch 文件** | ✅ 无传统 .patch | 通过 Git 提交跟踪 |
| **构建配置** | ✅ 有 | BUILD.gn, bundle.json, OAT.xml |
| **依赖者数量** | 6 个 | clap, rustix, which-rs 等 |
| **回归风险** | ✅ 极低 | 升级只需更新配置 |

---

## OH 适配概述

### 为什么选择 once_cell

once_cell 是 OpenHarmony Rust 基础设施的关键组件：

✅ **上游活跃**: API 正被提议纳入 Rust 标准库 (RFC 2788)，设计稳定
✅ **性能优异**: 零成本抽象，读操作无锁
✅ **功能完备**: 提供同步/异步、线程安全/非线程安全等多种变体
✅ **零配置**: 纯 Rust 实现，无需平台特定代码
✅ **易于维护**: 无需修改源代码，仅构建系统适配

### OH 适配策略

once_cell 在 OH 中的集成采用**最简单的适配模式**：

```
上游 once_cell (v1.17.0)
    ↓
添加 BUILD.gn (GN 构建系统)
添加 bundle.json (组件元数据)
添加 OAT.xml (合规审计)
添加 README.OpenSource (开源声明)
    ↓
OH 组件 @ohos/rust_once_cell
```

**特点**:
- ✅ **零源代码修改**: 所有修改为构建系统和合规性配置
- ✅ **标准化**: 使用统一的 `ohos_cargo_crate` GN 模板
- ✅ **低维护成本**: 升级上游只需更新版本号

---

## 主要文档摘要

### 01_Overview.md - 原始库简介

**内容要点**:
- once_cell 的核心功能：单次赋值单元格和延迟初始化
- 在 OH 中的作用：基础设施级别的 Rust 工具库
- 与其他全局状态方案的对比
- 版本历史和未来趋势（可能纳入 std）

**适合人群**: 想快速了解 once_cell 的所有读者

### 02_Patches.md - Patch 详细分析

**内容要点**:
- 4 个 OH 特定修改的详细分析
- 每个 Patch 的修改目的、内容、OH 价值和升级建议
- Patch 汇总表和维护策略
- 升级流程和回归测试指南

**适合人群**: 想深入了解 OH 适配细节的开发者、维护者

### 03_Build_Integration.md - OH 构建适配

**内容要点**:
- BUILD.gn 完整结构和字段详解
- 关键编译选项和特性配置
- 与上游构建系统的差异
- 构建示例和故障排查
- 升级指南

**适合人群**: 需要修改构建配置、解决编译问题的开发者

### 04_Usage_in_OH.md - 依赖关系与使用

**内容要点**:
- 6 个直接依赖者的详细分析
- 每个依赖者的使用场景和代码示例
- 依赖关系图（Mermaid）
- 典型使用场景和性能分析
- 迁移指南（如果纳入 std）

**适合人群**: 需要使用 once_cell 的开发者、架构师

---

## 快速开始

### 作为开发者使用 once_cell

#### 步骤 1: 添加依赖

**GN 依赖** (BUILD.gn):
```gn
ohos_cargo_crate("your_lib") {
    deps = [
        "//third_party/rust/crates/once_cell:lib",
    ]
}
```

**Cargo 依赖** (Cargo.toml):
```toml
[dependencies]
once_cell = "1.17.0"
```

#### 步骤 2: 使用 API

```rust
use once_cell::sync::Lazy;

// 延迟初始化全局变量
static CONFIG: Lazy<Config> = Lazy::new(|| {
    Config::load_from_env()
});

fn main() {
    let config = &*CONFIG;
    println!("Port: {}", config.port);
}
```

#### 步骤 3: 编译

```bash
ninja -C out/default //your/path:your_lib
```

### 作为维护者升级 once_cell

#### 步骤 1: 检查新版本
```bash
cd third_party/rust/crates/once_cell
git fetch upstream
git tag  # 查看最新版本
```

#### 步骤 2: 更新配置

**BUILD.gn**:
```gn
cargo_pkg_version = "1.18.0"  # 更新为新版本
```

**README.OpenSource**:
```json
{
  "Version Number": "1.18.0"  # 更新版本号
}
```

#### 步骤 3: 编译验证
```bash
ninja -C out/default //third_party/rust/crates/once_cell:lib
ninja -C out/default //third_party/rust/crates/clap:lib  # 验证依赖者
```

---

## 关键特性

### 启用的特性 (BUILD.gn)

| 特性 | 说明 | 用途 |
|------|------|------|
| **std** | 标准库支持 | 提供 `sync::OnceCell`, `sync::Lazy` |
| **alloc** | 内存分配 | 所有模块的基础 |
| **race** | First one wins 初始化 | `race::OnceBox`, `race::OnceRef` |

### 未启用的特性

| 特性 | 未启用原因 | 说明 |
|------|----------|------|
| **parking_lot** | 性能优化价值有限 | 使用 parking_lot 替代 std::sync，节省内存 |
| **critical-section** | 不适用 | 嵌入式场景，OH 标准系统不需要 |

---

## 常见问题

### Q1: once_cell 在 OH 中的版本与上游不同？

**A**: 是的。
- **上游版本**: 1.17.0（原始库版本）
- **OH 组件版本**: 6.1（OH 内部组件版本）

这是 OH 第三方库的常见做法：组件版本与上游版本独立管理。

---

### Q2: 为什么没有源代码的 Patch？

**A**: once_cell 是纯 Rust 库，不依赖平台特定 API。OH 的标准库环境完全满足其需求，因此无需修改源代码。

所有修改均为构建系统和合规性配置：
- BUILD.gn: GN 构建脚本
- bundle.json: OH 组件元数据
- OAT.xml: OSS 审计配置
- README.OpenSource: 开源声明

---

### Q3: 如何升级 once_cell 到新版本？

**A**: 简单 3 步：

1. **下载新版本**: `git checkout v1.18.0`
2. **更新配置**:
   - BUILD.gn: `cargo_pkg_version = "1.18.0"`
   - README.OpenSource: `"Version Number": "1.18.0"`
3. **编译验证**: `ninja ... //third_party/rust/crates/once_cell:lib`

详见 [03_Build_Integration.md](./03_Build_Integration.md#升级指南)。

---

### Q4: 哪些 OH 模块在使用 once_cell？

**A**: 6 个直接依赖者：

1. **clap**: 命令行参数解析器
2. **rustix**: POSIX 系统调用绑定
3. **which-rs**: 路径查找工具（Windows）
4. **request/rustest**: 请求模块测试框架
5. **rust-openssl**: OpenSSL 绑定
6. **request/services**: 请求服务（dev-dependency）

详见 [04_Usage_in_OH.md](./04_Usage_in_OH.md#直接依赖者)。

---

### Q5: once_cell 未来会被标准库替代吗？

**A**: 可能。

once_cell 的 API 正在通过 [RFC 2788](https://github.com/rust-lang/rfcs/pull/2788) 提议纳入 Rust 标准库。

**时间线**: 预计 1-2 年内标准库 API 可能稳定

**影响**:
- 一旦稳定，OH 可以迁移到 `std::sync::OnceCell` 和 `std::sync::Lazy`
- 迁移路径很简单（更换导入语句）
- 暂时继续使用 once_cell

---

## 相关资源

### 官方资源

- **上游仓库**: https://github.com/matklad/once_cell
- **上游文档**: https://docs.rs/once_cell
- **RFC 2788**: https://github.com/rust-lang/rfcs/pull/2788
- **许可证**: Apache 2.0 OR MIT

### OpenHarmony 资源

- **OH 组件**: `@ohos/rust_once_cell`
- **子系统**: thirdparty
- **Inner Kit**: `//third_party/rust/crates/once_cell:lib`

### OH Issue

- **构建 Issue**: https://gitee.com/openharmony/build/issues/I6UFTP
- **许可证 Issue**: https://gitee.com/openharmony/third_party_rust_bindgen/issues/I8ZMCP

---

## 贡献指南

### 报告问题

如果在使用 once_cell 时遇到问题：

1. **构建问题**: 提交 Issue 到 OH 构建仓库
2. **库本身问题**: 提交 Issue 到上游 once_cell 仓库
3. **文档问题**: 提交 Issue 到 OH 第三方库仓库

### 提交改进

如果你想改进 OH 的 once_cell 集成：

1. **构建优化**: 改进 BUILD.gn 配置
2. **文档完善**: 完善 wiki 文档
3. **合规性**: 更新 OAT.xml 或 README.OpenSource

**注意**: 不要修改 once_cell 的源代码（src/），请将改进提交到上游。

---

## 附录

### 文件清单

| 文件 | 用途 | 是否 OH 特有 |
|------|------|------------|
| **BUILD.gn** | GN 构建脚本 | ✅ 是 |
| **bundle.json** | OH 组件元数据 | ✅ 是 |
| **OAT.xml** | OSS 审计配置 | ✅ 是 |
| **README.OpenSource** | 开源声明 | ✅ 是 |
| **Cargo.toml** | 包元数据 | ❌ 否（与上游一致） |
| **src/** | 源代码 | ❌ 否（与上游一致） |

### 依赖者清单

| 模块 | BUILD.gn | Cargo.toml | 主要用途 |
|------|----------|-------------|----------|
| **clap** | ✅ | ✅ | 命令行解析 |
| **rustix** | ❌ | ✅ | 系统调用 |
| **which-rs** | ❌ | ✅ | 路径查找 |
| **request/rustest** | ✅ | ✅ | 测试框架 |
| **rust-openssl** | ✅ | ✅ | OpenSSL 绑定 |
| **request/services** | ❌ | ✅ (dev) | 请求服务 |

---

**最后更新**: 2026-02-08
**文档维护**: Sisyphus Agent
**反馈渠道**: OpenHarmony 第三方库仓库 Issue
