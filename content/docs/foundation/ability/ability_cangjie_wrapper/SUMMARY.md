# 文档导航

本文档为 `ability_cangjie_wrapper` 子系统的完整 Wiki，提供新人快速理解项目的多篇文档。

## 新人阅读路线

### 第一阶段：概览了解（建议 15 分钟）

1. **[README.md](README.md)** - Wiki 使用说明与覆盖范围
2. **[01_Overview.md](01_Overview.md)** - 项目定位、核心能力、运行环境

### 第二阶段：架构深入（建议 30 分钟）

3. **[02_Architecture.md](02_Architecture.md)** - 组件图、数据流、线程模型
4. **[03_APIs.md](03_APIs.md)** - N-API 完整清单与调用链

### 第三阶段：工程实践（建议 20 分钟）

5. **[04_Build.md](04_Build.md)** - GN targets 与编译产物
6. **[05_Security.md](05_Security.md)** - 安全风险评审

---

## 全站文档索引

### 核心文档

| 文档 | 说明 | 优先级 |
|-----|------|-------|
| [README.md](README.md) | Wiki 使用指南与覆盖范围 | 必读 |
| [01_Overview.md](01_Overview.md) | 项目定位与核心能力 | 必读 |
| [02_Architecture.md](02_Architecture.md) | 架构图、组件关系、数据流 | 必读 |
| [03_APIs.md](03_APIs.md) | N-API 导出清单与调用链 | 必读 |
| [04_Build.md](04_Build.md) | GN targets 与编译产物 | 推荐 |
| [05_Security.md](05_Security.md) | 安全风险分析与修复建议 | 推荐 |

### 附录

| 文档 | 说明 |
|-----|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏与 feature flags |

---

## 模块快速跳转

### 按功能分类

| 功能模块 | 核心类 | API 文档章节 |
|---------|-------|-------------|
| UIAbility 生命周期 | `UIAbility`, `UIAbilityContext` | 3.1 |
| 组件管理器 | `AbilityStage`, `AbilityStageContext` | 3.2 |
| 错误观测 | `ErrorManager`, `ErrorObserver` | 3.3 |
| 测试框架 | `TestRunner`, `AbilityDelegatorRegistry` | 3.4 |
| 意图传递 | `Want`, `WantValueType` | 3.5 |
| 应用恢复 | `AppRecovery` | 3.6 |
| 连接回调 | `ConnectOptions` | 3.7 |

### 按构建目标分类

| GN target | 说明 | Build 文档章节 |
|----------|-------|---------------|
| `kit.AbilityKit` | Kit 聚合目标 | 4.1 |
| `ohos.app.ability.ui_ability` | UIAbility 核心 | 4.2 |
| `ohos.app.ability.ability_stage` | 组件管理器 | 4.3 |

---

## 版本历史

| 版本 | 更新日期 | 主要变更 |
|-----|---------|---------|
| 6.1 | 2026-02-06 | 初始 Wiki 生成 |

---

## 贡献指南

欢迎完善本 Wiki！提交前请确保：

1. 所有关键结论有代码证据支持
2. API 清单与源码保持一致
3. 链接路径准确无误
4. 中文术语统一
