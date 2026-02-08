# 安全隐私中心 Wiki

## 文档概述

本文档是 OpenHarmony 安全隐私中心（Security Privacy Center）模块的工程 Wiki，旨在帮助开发者快速理解项目架构、功能模块、构建配置以及安全考量。

## 覆盖范围

本文档涵盖以下内容：

| 章节 | 主题 | 说明 |
|-----|------|------|
| 01 | 项目概览 | 项目定位、核心能力、运行环境 |
| 02 | 目录结构 | 模块划分、文件组织 |
| 03 | 架构设计 | MVI 架构、组件关系、数据流 |
| 04 | N-API 接口 | 本项目不涉及 Native API |
| 05 | 内部 API | 模块接口、依赖方向 |
| 06 | 构建配置 | hvigor 构建、编译参数 |
| 07 | 编译产物 | .hap 文件、安装路径 |
| 08 | 安全评审 | 威胁模型、风险分析 |
| 09 | 常见问题 | 构建、运行、调试问题 |

## 未覆盖范围

- **测试代码**：`ohosTest/` 目录下的测试用例不纳入文档范围
- **外部依赖源码**：仅描述接口契约，不追溯第三方库内部实现
- **运行时动态行为**：仅描述设计时架构，不涉及运行时性能分析

## 阅读建议

### 新人阅读顺序

建议按照以下顺序阅读：

1. **01_Overview.md** → 了解项目定位和核心能力
2. **02_Directory_Structure.md** → 熟悉代码组织方式
3. **03_Architecture.md** → 掌握整体架构设计
4. **05_Inner_API.md** → 理解模块间接口
5. **06_Build_Config.md** → 掌握构建配置
6. **07_Build_Outputs.md** → 了解产物形态

### 按角色阅读

| 角色 | 推荐章节 |
|-----|---------|
| 业务开发者 | 概览 → 架构 → 内部 API |
| 安全审计人员 | 安全评审 → 权限模型 |
| 构建工程师 | 构建配置 → 编译产物 |
| UI 开发者 | 目录结构 → 架构（视图层） |

## 文档更新规则

### 更新触发条件

当代码发生以下变更时，应同步更新 Wiki：

| 变更类型 | 更新内容 |
|---------|---------|
| 新增/删除页面 | 02_Directory_Structure.md、03_Architecture.md |
| 新增系统 API 调用 | 05_Inner_API.md、08_Security_Review.md |
| 修改构建配置 | 06_Build_Config.md、07_Build_Outputs.md |
| 新增权限声明 | 08_Security_Review.md |
| 架构重构 | 03_Architecture.md、05_Inner_API.md |

### 更新方式

1. 克隆仓库：`git clone https://gitee.com/openharmony/security_privacy_center.git`
2. 编辑 `wiki/` 目录下的对应 Markdown 文件
3. 确保关键结论可追溯到代码证据（路径 + 符号）
4. 提交 PR 前运行一致性检查（Phase 7）

## 版本信息

| 属性 | 值 |
|-----|---|
| 文档版本 | 1.0.0 |
| 生成日期 | 2026-02-06 |
| 适用的 SDK 版本 | OpenHarmony SDK 23 |
| 维护者 | OpenHarmony Security Team |

## 相关链接

- **代码仓库**：https://gitee.com/openharmony/security_privacy_center
- **应用接入指南**：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/SecurityPrivacyCenter/auto-menu-guidelines.md
- **OpenHarmony 官方文档**：https://developer.huawei.com/consumer/cn/docs/
