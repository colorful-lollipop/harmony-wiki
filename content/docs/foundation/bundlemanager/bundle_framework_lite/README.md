# Bundle Framework Lite Wiki

## 简介

本文档是 OpenHarmony **bundle_framework_lite**（包管理子系统 Lite 版）的智能 Wiki，旨在同时满足**新人学习者**和**安全研究员**的需求。

**生成时间**: 2026-02-07
**版本**: 1.0

---

## 文档范围

本文档覆盖以下内容：

### 面向新人学习者
- [项目概览](01_Overview.md) - 快速理解项目定位和能力边界
- [目录结构与代码地图](02_CodeMap.md) - 快速定位核心代码
- [架构与数据流](03_Architecture.md) - 建立全局架构认知
- [对外接口文档](04_Interface.md) - API 使用参考

### 面向安全研究员
- [攻击面分析](05_AttackSurface.md) - 识别所有外部输入和敏感操作
- [安全风险评估](06_SecurityReview.md) - 深度安全分析与利用点

### 工程实现细节
- [构建与产物](07_Build.md) - GN 构建配置和编译产物
- [内部实现细节](08_Internals.md) - 核心类和资源生命周期

---

## 未覆盖内容

- 测试代码（`test/`、`unittest/`、`fuzz/` 等目录）
- 详细的性能优化指南
- 各芯片平台的移植指南

---

## 如何使用本文档

### 新人学习路线

```
1. 阅读项目概览（15 分钟）
   ↓
2. 理解架构与数据流（30 分钟）
   ↓
3. 参考目录结构定位代码（15 分钟）
   ↓
4. 使用对外接口文档（按需查阅）
```

**预计时间**: 1 小时即可建立基本认知

### 安全研究路线

```
1. 阅读攻击面分析（20 分钟）
   ↓
2. 深入安全风险评估（60 分钟）
   ↓
3. 结合架构图理解数据流（30 分钟）
   ↓
4. 参考内部实现细节（按需查阅）
```

**预计时间**: 2 小时即可完成安全审计

---

## 如何更新本文档

1. **修改代码后**：同步更新对应的 wiki 页面
2. **新增功能时**：在相关页面添加说明，必要时更新架构图
3. **发现安全问题时**：在 [安全风险评估](06_SecurityReview.md) 添加新条目
4. **定期 review**：建议每个版本发布前全面 review 文档准确性

---

## 快速导航

### 核心文档
- [项目概览](01_Overview.md) - 项目定位、能力边界、快速开始
- [目录结构与代码地图](02_CodeMap.md) - 代码导航图
- [架构与数据流](03_Architecture.md) - 组件图、数据流、线程模型
- [对外接口文档](04_Interface.md) - C API、IPC 接口、JS 绑定

### 安全文档
- [攻击面分析](05_AttackSurface.md) - 外部输入、敏感操作、信任边界
- [安全风险评估](06_SecurityReview.md) - 输入验证、内存安全、权限控制

### 工程文档
- [构建与产物](07_Build.md) - GN targets、编译产物、Feature 开关
- [内部实现细节](08_Internals.md) - 核心类/结构体、资源生命周期

### 工作文档
- [项目评估](wiki/_work/ASSESSMENT.md) - Phase 0 评估结果
- [代码证据](wiki/_work/NOTES.md) - 所有技术结论的代码证据
- [任务计划](wiki/_work/PLAN.md) - 文档构建进度追踪

---

## 质量保证

本文档遵循以下质量标准：

- ✅ **证据优先**：所有技术结论均可追溯到具体代码路径
- ✅ **受众导向**：新人路线和安全路线分离，满足不同需求
- ✅ **全局一致**：术语统一，内部链接有效
- ✅ **无测试代码**：所有文档内容忽略测试相关代码
- ✅ **中文为主**：专业术语保留英文

---

## 参考链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony)
- [bundle_framework_lite 仓库](https://gitee.com/openharmony/bundlemanager_bundle_framework_lite)
- [Bundle Framework 完整版](https://gitee.com/openharmony/bundlemanager_bundle_framework)（标准系统版本）

---

*本文档基于代码证据生成，所有关键结论均可追溯到具体代码路径。*
