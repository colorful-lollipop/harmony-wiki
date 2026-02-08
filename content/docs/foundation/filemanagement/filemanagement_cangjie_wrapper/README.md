# 文件管理仓颉封装 Wiki

> 本 Wiki 为 OpenHarmony `filemanagement_cangjie_wrapper` 仓库提供完整的技术文档
> 生成时间: 2026-02-06
> 适用范围: 仓颉（Cangjie）语言开发者、系统架构师、安全审计人员

## 文档覆盖范围

### 已覆盖
- ✅ 项目概览与核心能力
- ✅ 目录结构与模块职责
- ✅ 系统架构（组件图、数据流、线程模型）
- ✅ 对外 FFI 绑定与外部依赖接口
- ✅ 对外 Cangjie API（FileIo、File、Stream、Stat 等）
- ✅ 内部模块接口与依赖关系
- ✅ GN 构建目标与编译产物
- ✅ 安全风险评审（攻击面、信任边界、可被利用点）

### 未覆盖
- ❌ 测试框架与用例设计（按约束忽略测试目录）
- ❌ 性能基准测试数据
- ❌ 跨平台移植指南（当前仅支持 Standard 设备）
- ❌ 第三方集成案例（仅说明外部依赖）

## 如何使用本 Wiki

### 新人阅读顺序（推荐路线）

1. **[00_Overview.md](00_Overview.md)** - 了解项目定位、边界、核心能力
2. **[01_Directories_and_Modules.md](01_Directories_and_Modules.md)** - 熟悉代码组织结构
3. **[02_Architecture.md](02_Architecture.md)** - 理解架构层次、数据流、线程模型
4. **[03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md)** - 深入 FFI 接口（如需集成）
5. **[04_Public_API.md](04_Public_API.md)** - 学习对外的 Cangjie API 使用方法
6. **[05_Internal_API.md](05_Internal_API.md)** - 理解内部模块接口设计（如需扩展）
7. **[06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md)** - 了解构建系统与产物
8. **[07_Security_Review.md](07_Security_Review.md)** - 审查安全风险与修复建议

### 角色导向阅读

| 角色 | 推荐章节 | 目的 |
|------|----------|------|
| **仓颉开发者** | 00 → 04 | 快速上手 API 调用 |
| **架构师** | 00 → 02 → 05 | 理解分层设计、扩展点 |
| **安全审计** | 00 → 02 → 07 | 评估攻击面、风险点 |
| **构建工程师** | 00 → 06 | 掌握 GN 目标、依赖关系 |
| **集成开发者** | 00 → 03 → 04 | 理解 FFI 接口、外部依赖 |

## 文档更新机制

### 随代码更新

本 Wiki 通过以下方式保持同步：

1. **证据驱动**：所有关键结论必须包含代码证据（路径:行号 + 符号名）
2. **TODO 标记**：未确认内容标注 `TODO(需确认)` 并说明缺少的证据
3. **版本追踪**：每篇文档包含"最后更新"时间戳
4. **变更检查**：修改代码时需同步更新对应 Wiki 章节

### 贡献指南

如发现文档错误或需要补充：

1. 在对应 Markdown 文件中直接修改（保持证据链接完整性）
2. 更新 `wiki/_work/PLAN.md` 的进度状态
3. 确保所有结论有代码证据支撑
4. 遵循"硬性约束"（见 README.md:56-66）

## 相关链接

- [主仓库 README](../README.md)
- [主仓库 README（中文）](../README_zh.md)
- [工作笔记](./_work/NOTES.md)
- [工作计划](./_work/PLAN.md)
- [架构图](../figures/filemanagement_cangjie_wrapper_architecture_zh.png)

## 版本历史

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|----------|------|
| 1.0 | 2026-02-06 | 初始版本，完成所有章节 | Auto-Generated |

---

> **注意**：本仓库的 Cangjie API 处于 Beta 阶段（README:1），能力与限制可能随版本变化。建议配合最新 README 和源码使用本 Wiki。
