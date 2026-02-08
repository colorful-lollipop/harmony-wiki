# request_cangjie_wrapper Wiki

## 文档概述

本文档为 OpenHarmony `request_cangjie_wrapper` 子系统的工程 Wiki，旨在帮助开发者快速理解项目定位、架构设计、API 接口、构建方式及安全风险。

**项目版本**: 6.1
**Wiki 版本**: 1.0
**最后更新**: 2026-02-07

---

## 文档结构

### 基础文件

| 文件 | 说明 | 用途 |
|------|------|------|
| [README](README.md) | 本文档 | 项目概览和导航 |
| [SUMMARY](SUMMARY.md) | 导航地图 | 双路线推荐阅读 |
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估 | 类型判定和策略决策 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据 | 技术结论的代码支撑 |

### 核心内容

#### A. 新人学习必备

| 序号 | 文件 | 说明 | 受众 |
|------|------|------|------|
| 01 | [01_Overview.md](01_Overview.md) | 项目定位、能力边界、快速开始 | 新人 |
| 02 | [02_Architecture.md](02_Architecture.md) | 组件图、数据流、生命周期 | 新人+开发 |
| 03 | [03_CodeMap.md](03_CodeMap.md) | 目录结构、核心文件定位 | 开发 |
| 04 | [04_Interface.md](04_Interface.md) | N-API 清单、配置说明 | 开发 |

#### B. 安全研究必备

| 序号 | 文件 | 说明 | 受众 |
|------|------|------|------|
| 05 | [05_AttackSurface.md](05_AttackSurface.md) | 外部输入、敏感操作、信任边界 | 安全 |
| 06 | [06_SecurityReview.md](06_SecurityReview.md) | 安全风险、修复建议 | 安全+开发 |

#### C. 工程实现

| 序号 | 文件 | 说明 | 受众 |
|------|------|------|------|
| 07 | [07_Build.md](07_Build.md) | GN targets、编译产物 | 开发 |
| 08 | [08_Internals.md](08_Internals.md) | 核心类、生命周期、内部 API | 高级开发 |

---

## 覆盖范围

| 分类 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 定位、能力、约束 |
| 架构设计 | ✅ 完成 | 组件图、数据流、线程模型 |
| 代码地图 | ✅ 完成 | 目录结构、核心文件定位 |
| 接口文档 | ✅ 完成 | API 清单、参数、返回值 |
| 攻击面分析 | ✅ 完成 | 外部输入、敏感操作、信任边界 |
| 安全评审 | ✅ 完成 | 风险评估、修复建议 |
| 构建配置 | ✅ 完成 | GN targets、产物、依赖 |
| 内部实现 | ✅ 完成 | 核心类、资源生命周期 |

---

## 双路线导航

### 新人学习路线

> **目标**: 快速理解项目并上手使用

**推荐阅读顺序**:
1. [01_Overview.md](01_Overview.md) → 理解项目定位
2. [SUMMARY.md](SUMMARY.md) → 熟悉文档结构
3. [03_CodeMap.md](03_CodeMap.md) → 了解代码结构
4. [04_Interface.md](04_Interface.md) → 掌握 API 使用
5. [02_Architecture.md](02_Architecture.md) → 深入架构设计

### 安全研究路线

> **目标**: 识别攻击面、评估风险

**推荐阅读顺序**:
1. [05_AttackSurface.md](05_AttackSurface.md) → 识别攻击面
2. [06_SecurityReview.md](06_SecurityReview.md) → 评估风险
3. [04_Interface.md](04_Interface.md) → 理解接口细节
4. [NOTES.md](_work/NOTES.md) → 追溯代码证据

---

## 快速导航

### 核心类速查

| 类名 | 文件:行号 | 职责 |
|------|-----------|------|
| `Task` | agent.cj:1661 | 任务主入口 |
| `Config` | agent.cj:578 | 任务配置 |
| `FileSpec` | agent.cj:342 | 文件规格 |
| `Progress` | agent.cj:1076 | 进度信息 |

### 外部链接

- [API 参考文档 (外部)](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-request-agent.md)
- [开发指南 (外部)](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/basic-services/request/cj-app-file-upload-download.md)
- [上游 request_request 仓库](https://gitcode.com/openharmony/request_request/blob/master/README.md)

---

## 文档更新方式

### 随代码更新策略

当代码变更时，需同步更新以下内容：

| 变更类型 | 更新文档 |
|----------|----------|
| API 变更 | [04_Interface.md](04_Interface.md) |
| 架构调整 | [02_Architecture.md](02_Architecture.md) |
| 构建配置变更 | [07_Build.md](07_Build.md) |
| 新增安全风险 | [06_SecurityReview.md](06_SecurityReview.md) |
| 新增文件 | [03_CodeMap.md](03_CodeMap.md) |

### 版本对应关系

| Wiki 版本 | 代码版本 | 更新日期 |
|-----------|----------|----------|
| 1.0 | 6.1 | 2026-02-07 |

---

## 受众指南

| 角色 | 推荐阅读 | 优先级 |
|------|----------|--------|
| **Cangjie 应用开发者** | 概览 → API → 快速开始 | ⭐⭐⭐ |
| **系统集成开发者** | 概览 → 架构 → 构建 | ⭐⭐ |
| **安全审计人员** | 攻击面 → 安全评审 → 接口 | ⭐⭐⭐ |
| **贡献者** | 架构 → 代码地图 → 内部实现 | ⭐⭐ |
| **架构师** | 概览 → 架构 → 安全 | ⭐⭐⭐ |

---

## 贡献指南

发现文档错误或需要补充？请通过 OpenHarmony 常规贡献流程提交 Issue 或 Patch。

---

## 局限性说明

1. **测试代码不计入分析范围**，本文档仅关注业务代码
2. **外部依赖文档引用官方链接**，详细内容请参考外部文档
3. **安全评审基于代码静态分析**，未进行渗透测试
4. **API 参数细节引用外部文档**，本 Wiki 提供概览性说明
5. **依赖的 request 服务内部实现未纳入分析范围**
