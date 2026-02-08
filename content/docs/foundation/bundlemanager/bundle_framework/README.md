# Bundle Framework Wiki - 文档说明

## 文档概述

本文档为 OpenHarmony 包管理子系统 (bundlemanager_bundle_framework) 的工程 Wiki，旨在帮助开发者快速理解项目架构、接口规范、构建流程和安全模型。

**生成时间**: 2026-02-06
**最后更新**: 2026-02-07
**维护者**: Bundle Framework 团队

---

## 覆盖范围

本 Wiki 涵盖以下内容：

| 章节 | 主题 | 状态 |
|------|------|------|
| [README](README.md) | 文档说明 | ✅ |
| [SUMMARY](SUMMARY.md) | 全站导航 | ✅ |
| [01_Overview](01_Overview.md) | 项目概览 | ✅ |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构 | ✅ |
| [02_Architecture](02_Architecture.md) | 架构说明 | ✅ |
| [03_CodeMap](03_CodeMap.md) | 代码地图 | ✅ |
| [03_N-API_Reference](03_N-API_Reference.md) | JS/N-API 接口 | ✅ |
| [04_Inner_API](04_Inner_API.md) | 内部 API | ✅ |
| [04_Interface](04_Interface.md) | 接口汇总 | ✅ (待创建) |
| [05_AttackSurface](05_AttackSurface.md) | 攻击面分析 | ✅ |
| [06_SecurityReview](06_SecurityReview.md) | 安全风险评估 | ✅ |
| [07_Security_Review](07_Security_Review.md) | 安全概览 | ✅ |
| [07_Build](07_Build.md) | 构建与产物 | ✅ |
| [08_Internals](08_Internals.md) | 内部实现 | ✅ |
| [08_FAQ](08_FAQ.md) | 常见问题 | ✅ |

---

## 未覆盖范围

以下内容不在本 Wiki 范围内：

- **测试代码**: 单元测试、集成测试、系统测试等测试代码 (`test/`, `*_test.cpp`)
- **外部依赖文档**: OpenHarmony 其他子系统的详细说明（请参考对应仓库 Wiki）
- **API 变更历史**: 版本演进和 API 变更记录
- **性能基准测试数据**: 性能指标和优化建议

---

## 阅读建议

### 新人入门路径

1. [01_Overview](01_Overview.md) - 了解项目定位和核心能力
2. [01_Directory_Structure](01_Directory_Structure.md) - 熟悉目录结构和模块职责
3. [02_Architecture](02_Architecture.md) - 理解系统架构和数据流
4. [03_CodeMap](03_CodeMap.md) - 掌握代码导航
5. 选择感兴趣的模块阅读 N-API 或 Inner API 文档

### 安全研究路径

1. [05_AttackSurface](05_AttackSurface.md) - 了解攻击面
2. [06_SecurityReview](06_SecurityReview.md) - 详细风险分析
3. [02_Architecture](02_Architecture.md) - 理解架构安全机制

### 按需查阅

- 需要使用 JS API → [03_N-API_Reference](03_N-API_Reference.md)
- 需要使用 Native API → [04_Inner_API](04_Inner_API.md)
- 需要查找代码位置 → [03_CodeMap](03_CodeMap.md)
- 需要构建/编译 → [07_Build](07_Build.md)
- 需要理解内部实现 → [08_Internals](08_Internals.md)
- 需要安全评估 → [05_AttackSurface](05_AttackSurface.md) + [06_SecurityReview](06_SecurityReview.md)
- 遇到问题 → [08_FAQ](08_FAQ.md)

---

## 文档规范

### 证据引用

所有关键结论必须可追溯到代码证据。引用格式：

- **文件路径**: `path/to/file:line`
- **符号名**: `ClassName::methodName()` / `MACRO_NAME`
- **配置项**: `BUILD.gn:target_name` / `feature_flag`

示例：
> BundleMgrService 是 SA 401 的实现，运行在 foundation 进程 (`sa_profile/401.json:5-10`)

### 术语约定

| 术语 | 说明 |
|------|------|
| Bundle | 应用安装包（HAP 文件） |
| HAP | Harmony Ability Package - OpenHarmony 应用包格式 |
| Module | HAP 文件中包含的一个或多个 Ability |
| Ability | 应用组件，代表功能 |
| SA | System Ability - 系统能力 |
| N-API | Node.js API - JS 本地接口 |
| Inner API | 内部接口，仅供子系统内部使用 |
| InstalldService | SA 511，特权文件操作服务 |
| Binder | OpenHarmony IPC 机制 |

---

## 更新指南

### 何时更新文档

当发生以下变更时，需要更新对应 Wiki 章节：

| 变更类型 | 更新章节 | 优先级 |
|----------|----------|--------|
| 新增 N-API 模块 | 03_N-API_Reference | 高 |
| N-API 签名变更 | 03_N-API_Reference | 高 |
| 新增 Inner API | 04_Inner_API | 高 |
| 新增/删除模块 | 01_Directory_Structure, 03_CodeMap | 中 |
| 架构变更 | 02_Architecture, 08_Internals | 高 |
| 新增安全风险 | 05_AttackSurface, 06_SecurityReview | 高 |
| BUILD.gn 变更 | 07_Build | 中 |

### 更新流程

1. 修改代码后，同步更新对应 Wiki 章节
2. 确保关键结论有代码证据（路径 + 符号）
3. 运行一致性检查（链接、术语）
4. 提交 Wiki 变更与代码变更

---

## 贡献指南

欢迎提交 Wiki 改进建议或修复：

1. 在 Wiki 仓库创建 Issue 描述问题
2. 或直接提交 PR 修改 wiki/ 目录
3. 遵循文档规范（证据引用、术语约定）

---

## 相关链接

- **官方文档**: [OpenHarmony 包管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/包管理子系统.md)
- **代码仓库**: [bundlemanager_bundle_framework](https://gitee.com/openharmony/bundlemanager_bundle_framework)
- **相关仓库**:
  - [bundlemanager_bundle_tool](https://gitee.com/openharmony/bundlemanager_bundle_tool)
  - [bundlemanager_distributed_bundle_framework](https://gitee.com/openharmony/bundlemanager_distributed_bundle_framework)
  - [developtools_packing_tool](https://gitee.com/openharmony/developtools_packing_tool)
