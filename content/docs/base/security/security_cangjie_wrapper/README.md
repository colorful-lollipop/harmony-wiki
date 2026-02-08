# OpenHarmony security_cangjie_wrapper Wiki

## 项目概述

本文档是 OpenHarmony 安全子系统 `security_cangjie_wrapper` 仓库的工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、编译构建以及安全评审信息。

**当前状态**：Beta 特性（详见 [README](../README.md)）

**Wiki 类型**：标准版（6-8 篇核心文档）

---

## 覆盖范围

| 章节 | 内容 | 状态 |
|------|------|------|
| [项目评估（ASSESSMENT.md）](_work/ASSESSMENT.md) | 项目类型、受众分析、文档策略 | ✅ |
| [概览](00_Overview.md) | 项目定位、能力、约束 | ✅ |
| [目录结构](01_Directory_Structure.md) | 模块职责与文件组织 | ✅ |
| [架构说明](02_Architecture.md) | 组件图、数据流、FFI 机制 | ✅ |
| [N-API 参考](03_N-API_Reference.md) | Crypto/Huks API 详细文档 | ✅ |
| [内部 API](04_Internal_API.md) | 模块接口与依赖 | ✅ |
| [GN Targets](05_GN_Targets.md) | 构建目标与产物 | ✅ |
| [编译产物](06_Build_Artifacts.md) | .so/.abc 文件与安装路径 | ✅ |
| [安全评审](07_Security_Review.md) | 攻击面、风险、修复建议 | ✅ |

---

## 阅读导航

本 Wiki 提供两条阅读路线，满足不同受众需求：

### 🎓 新人学习路线

**目标**：30 分钟内快速上手

1. [概览](00_Overview.md) → 理解项目定位
2. [目录结构](01_Directory_Structure.md) → 找到代码位置
3. [N-API 参考](03_N-API_Reference.md) → 学习 API 使用
4. [架构说明](02_Architecture.md) → 理解调用链路

**详细导航**：[查看完整学习路线](SUMMARY.md#🎓-新人学习路线快速上手)

### 🔒 安全研究路线

**目标**：深度分析安全风险

1. [安全评审](07_Security_Review.md) → 识别攻击面
2. [架构说明](02_Architecture.md) → 理解 FFI 机制
3. [N-API 参考](03_N-API_Reference.md) → 分析 API 入口
4. [内部 API](04_Internal_API.md) → 定位模块边界

**详细导航**：[查看完整研究路线](SUMMARY.md#🔒-安全研究路线深度分析)

---

## 文档特点

### ✅ 证据优先

每个技术结论都可追溯至代码证据：
- 文件路径（含行号格式：`path:line`）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链
- **禁止凭空猜测**；无法确认处标注 `TODO(证据不足)`

### ✅ 受众导向

**新人学习者**：
- ✅ 5 分钟理解项目定位
- ✅ 15 分钟找到核心代码位置
- ✅ 30 分钟理解基本架构

**安全研究员**：
- ✅ 快速识别所有外部输入入口
- ✅ 定位敏感操作和权限检查点
- ✅ 每个风险都有可利用性评估和修复建议

### ✅ 全局一致

- 术语统一，内部链接有效
- 所有代码片段有语法标注
- 证据链完整可追溯

---

## 未覆盖范围

- 详细 API 使用示例（请参考 [Cangjie API 官方文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)）
- 底层 native 实现细节（由 `crypto_framework` 和 `huks` 子系统提供）
- 测试代码分析（遵循约束，不引用测试代码）
- 跨仓库集成指南

---

## 文档更新方式

### 何时需要更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. **新增/删除模块**：更新目录结构与模块职责说明
2. **新增 API**：更新 N-API 参考章节
3. **修改构建配置**：更新 GN Targets 与编译产物章节
4. **安全相关变更**：更新安全评审章节

### 更新步骤

1. 修改对应的 `wiki/*.md` 文件
2. 同步更新 `SUMMARY.md` 中的导航链接
3. 执行 `lsp_diagnostics` 验证文档无格式错误
4. 提交变更（与代码变更同 PR 或独立 PR）

---

## 生成信息

- **代码仓库**：`base/security/security_cangjie_wrapper`
- **评估日期**：2025-02-07
- **Wiki 状态**：标准版（完整覆盖核心功能）
- **生成工具**：OpenCode Wiki Generator Agent
- **证据来源**：所有关键结论均可追溯至源码

---

## 相关链接

### 官方文档

- [项目 README](../README.md) - 项目官方说明
- [Cangjie ArkInterop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - Cangjie 基础能力
- [Crypto Framework](https://gitcode.com/openharmony/security_crypto_framework) - 加密框架实现
- [HUKS](https://gitcode.com/openharmony/security_huks) - 密钥管理实现

### 工作区

- [项目评估](wiki/_work/ASSESSMENT.md) - 项目画像与文档策略
- [事实记录](wiki/_work/NOTES.md) - 代码证据汇总
- [任务进度](wiki/_work/PLAN.md) - Wiki 生成进度追踪

### 快速跳转

- [完整导航](SUMMARY.md) - 双路线阅读指南
- [按功能分类](SUMMARY.md#快速跳转) - 功能导向的快速导航
- [术语表](SUMMARY.md#术语表) - 常见术语解释

---

## 贡献与反馈

本文档由 OpenCode Wiki Generator Agent 自动生成，基于代码静态分析和架构理解。

如发现文档错误或有改进建议，欢迎：
1. 提交 Issue 反馈问题
2. 提交 PR 改进文档内容
3. 参考 [OpenHarmony 代码贡献指南](https://gitcode.com/openharmony/docs/blob/master/en/contribute/code-contribution.md)
