# libc - OpenHarmony 适配文档

> **库版本**: 0.2.153
> **OH 组件**: @ohos/rust_libc
> **子系统**: thirdparty
> **最后更新**: 2026-02-08

---

## 文档简介

本 Wiki 记录了 **libc** crate 在 OpenHarmony 中的集成、适配和使用情况。

libc 是 Rust 生态系统中最重要的基础库之一，为 Rust 代码提供与 C 标准库（libc）的 FFI（Foreign Function Interface）绑定。在 OpenHarmony 中，libc 几乎是所有需要与操作系统内核交互的 Rust 代码的必选项。

**重点说明**：
- ✅ libc 原生支持 OpenHarmony（通过 `target_env = "ohos"`）
- ✅ Patch 数量极少（仅 1 个 CI 兼容性 Patch）
- ✅ 与 musl libc 共享大部分代码路径
- ⚠️ 有一些 OHOS 特定的适配（utmpx 布局、locale 常量、缺失函数等）

---

## 快速导航

### 📖 核心文档

| 文档 | 描述 | 推荐阅读顺序 |
|------|------|--------------|
| **[01_Overview.md](./01_Overview.md)** | 原始库简介、在 OH 中的作用和定位 | 1️⃣ 必读 |
| **[02_Patches.md](./02_Patches.md)** | Patch 详细分析（1 个 CI Patch）| 2️⃣ 推荐 |
| **[03_Build_Integration.md](./03_Build_Integration.md)** | OH 构建适配（BUILD.gn、build.rs）| 3️⃣ 推荐 |
| **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** | 依赖关系与使用场景 | 4️⃣ 推荐 |
| **[05_API_Differences.md](./05_API_Differences.md)** | API/接口差异（如有）| 5️⃣ 参考 |
| **[06_Security.md](./06_Security.md)** | 安全风险分析 | 6️⃣ 参考 |

### 📋 工作文档

| 文档 | 描述 |
|------|------|
| **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** | 项目评估结果（Phase 0）|
| **[_work/NOTES.md](./_work/NOTES.md)** | 分析过程记录（待补充）|
| **[_work/PLAN.md](./_work/PLAN.md)** | 任务进度（待补充）|

---

## 阅读路线建议

### 🎯 路线 1：快速了解（约 15 分钟）

如果你只想了解 libc 在 OpenHarmony 中的基本情况：

1. 📖 **[01_Overview.md](./01_Overview.md)** - 5 分钟
   - 了解 libc 的基本功能和作用
   - 了解 OHOS 特性总结

2. 📖 **[02_Patches.md](./02_Patches.md)** - 5 分钟
   - 了解唯一 Patch 的内容和目的
   - 评估升级风险

3. 📖 **[03_Build_Integration.md](./03_Build_Integration.md)** - 5 分钟
   - 了解 BUILD.gn 配置
   - 了解特性（Features）说明

### 🔧 路线 2：深度分析（约 45 分钟）

如果你需要深入了解 OHOS 的适配细节：

1. 📖 **[01_Overview.md](./01_Overview.md)** - 10 分钟
   - 完整阅读所有章节
   - 理解在 OH 系统层次中的位置

2. 📖 **[02_Patches.md](./02_Patches.md)** - 10 分钟
   - 详细的 Patch 分析
   - 升级策略和维护建议

3. 📖 **[03_Build_Integration.md](./03_Build_Integration.md)** - 15 分钟
   - 完整的构建系统说明
   - 与上游的差异
   - 特殊处理（禁用功能、类型对齐等）

4. 📖 **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 10 分钟
   - 依赖关系分析
   - 典型使用场景
   - 依赖图

### 🛡️ 路线 3：安全审计（约 30 分钟）

如果你关注安全性：

1. 📖 **[02_Patches.md](./02_Patches.md)** - 5 分钟
   - Patch 的安全影响

2. 📖 **[06_Security.md](./06_Security.md)** - 20 分钟
   - 已知 CVE 和修复状态
   - OHOS Patch 的安全风险
   - 升级建议

3. 📖 **[03_Build_Integration.md](./03_Build_Integration.md)** - 5 分钟
   - 构建配置的安全考虑

### 🚀 路线 4：升级/维护（约 60 分钟）

如果你计划升级 libc 版本或维护 OHOS 适配：

1. 📖 **[01_Overview.md](./01_Overview.md)** - 10 分钟
   - 基础信息和定位

2. 📖 **[02_Patches.md](./02_Patches.md)** - 15 分钟
   - 现有 Patch 的分析
   - 升级策略

3. 📖 **[03_Build_Integration.md](./03_Build_Integration.md)** - 20 分钟
   - 构建系统差异
   - OHOS 特定适配
   - 类型对齐规则

4. 📖 **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 10 分钟
   - 依赖关系
   - 影响范围

5. 📖 **[06_Security.md](./06_Security.md)** - 5 分钟
   - 安全考虑

---

## 关键发现摘要

### ✅ 原生支持
- libc crate **原生支持** OpenHarmony（`target_env = "ohos"`）
- 与 musl libc 共享大部分代码路径

### 🔧 Patch 数量极少
- 仅 1 个 Patch：`ci/sysinfo_guard.patch`
- 不影响核心功能，仅影响 CI 环境

### 📊 OHOS 特定适配

| 适配点 | 说明 |
|--------|------|
| **utmpx 布局** | 使用 musl 1.2 布局，字段类型与标准 musl 不同 |
| **locale 常量** | 支持 GNU 扩展的 locale 类别（LC_PAPER, LC_NAME 等）|
| **缺失函数** | 排除 OHOS 不支持的 POSIX 函数（消息队列、robust mutex 等）|
| **socket 选项** | 不支持较新的 socket 选项常量（SO_*_NEW）|
| **类型对齐** | 与 musl 共享相同的对齐规则 |

### ⚠️ 风险评估

| 风险项 | 风险等级 | 说明 |
|--------|----------|------|
| **上游版本升级** | 🟡 中 | 需验证 OHOS 特定适配 |
| **ABI 兼容性** | 🟢 低 | 与 OHOS C 库紧密相关，需确保一致 |
| **缺失功能** | 🟢 低 | 排除的功能在 OHOS 中本就不支持 |
| **Patch 维护** | 🟢 低 | 只有一个 CI Patch，影响范围小 |

---

## 常见问题（FAQ）

### Q1: 为什么 OpenHarmony 需要这个 crate？
**A**: libc 是 Rust 生态的基础。任何需要调用 C 函数、系统调用或访问操作系统资源的 Rust 代码都必须依赖它。OpenHarmony 使用 Rust 开发的系统组件都需要通过 libc 与系统交互。

### Q2: OHOS 版本的 libc 与上游版本有什么不同？
**A**: OHOS 版本主要增加了：
- `target_env = "ohos"` 的条件编译支持
- OHOS 特定的 utmpx 结构体布局
- OHOS 特有的 locale 常量扩展
- 排除 OHOS 不支持的 POSIX 函数
- 一个 CI 环境的兼容性 Patch

### Q3: OHOS 的 libc 支持哪些功能？
**A**: 支持大部分 POSIX 标准功能，包括文件操作、进程管理、线程、信号、网络、时间等。但不支持部分 POSIX 扩展，如 POSIX 消息队列、robust mutex 等。

### Q4: 如何升级 libc 到新版本？
**A**: 升级时需要：
1. 验证 OHOS 特定的适配是否仍然有效
2. 检查新增的代码块是否需要排除 OHOS
3. 运行测试确保与 OHOS C 库的兼容性
4. 保留 CI Patch 直到上游修复相关问题

### Q5: OHOS 使用的是 glibc 还是 musl？
**A**: OHOS 基于 **musl libc**。在 libc crate 中，OHOS 大部分代码路径与 musl 共享，但有一些特定差异。例如，OHOS 使用 musl 1.2 布局，但某些字段的类型与标准 musl 不同。

---

## 相关资源

### 外部参考
- [libc 官方文档](https://docs.rs/libc/)
- [libc GitHub 仓库](https://github.com/rust-lang/libc)
- [libc RFC](https://github.com/rust-lang/rfcs/blob/HEAD/text/1291-promote-libc.md)
- [OpenHarmony 文档中心](https://docs.openharmony.cn/)
- [musl libc](https://musl.libc.org/)

### 内部资源
- [README.md](../README.md) - 项目根目录 README
- [README.OpenSource](../README.OpenSource) - 开源许可信息
- [BUILD.gn](../BUILD.gn) - OH 构建配置
- [Cargo.toml](../Cargo.toml) - Cargo 包配置
- [build.rs](../build.rs) - 构建脚本
- [CONTRIBUTING.md](../CONTRIBUTING.md) - 贡献指南

### 相关 Issue 和 PR
- (待补充：如果存在相关的 GitHub Issue 或 PR，在此列出)

---

## 贡献指南

如果您发现本 Wiki 有错误或需要补充，欢迎：

1. 📝 直接编辑或提交 Issue
2. 🔍 提供新的分析或发现
3. 🐛 报告文档中的错误或不准确之处
4. 💡 提出改进建议

---

## 版本历史

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|----------|------|
| 1.0 | 2026-02-08 | 初始版本，创建核心文档 | Sisyphus |

---

**文档维护者**: Sisyphus (OpenHarmony Third-Party Wiki Agent)
**许可证**: 与项目相同（Apache 2.0 OR MIT）
**最后更新**: 2026-02-08
