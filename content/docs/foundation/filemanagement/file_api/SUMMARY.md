# File API Wiki 导航

本文档提供 File API Wiki 的完整导航结构，包含全站目录和两条推荐阅读路线：新人学习路线和安全研究路线。

## 1. 全站导航

### 1.1 基础文件

| 文件 | 说明 |
|------|------|
| [README](README.md) | 项目概览、文档导航、更新说明 |
| [SUMMARY](SUMMARY.md) | 全站导航（本文档）、推荐阅读路线 |
| [ASSESSMENT](_work/ASSESSMENT.md) | 项目评估报告 |
| [NOTES](_work/NOTES.md) | 代码证据库 |
| [PLAN](_work/PLAN.md) | 任务进度追踪 |

### 1.2 核心文档

| 章节 | 文件 | 说明 |
|------|------|------|
| 01 | [01_Overview.md](01_Overview.md) | 项目概览、能力边界、快速开始 |
| 02 | [02_Architecture.md](02_Architecture.md) | 架构设计、数据流、线程模型 |
| 03 | [03_CodeMap.md](03_CodeMap.md) | 目录结构、代码定位 |
| 04 | [04_Interface.md](04_Interface.md) | 对外接口文档 |
| 05 | [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |
| 06 | [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 |
| 07 | [07_Build.md](07_Build.md) | 构建配置、编译产物 |
| 08 | [08_Internals.md](08_Internals.md) | 内部实现细节 |

### 1.3 附录

| 文件 | 说明 |
|------|------|
| [appendix/FAQ.md](appendix/FAQ.md) | 常见问题解答 |
| [appendix/Changelog.md](appendix/Changelog.md) | 变更历史 |

---

## 2. 新人学习路线

本路线专为首次接触 File API 的开发者设计，帮助快速理解项目定位、掌握基本用法。

### 2.1 学习路径

```
step1: 阅读项目概览
    ↓
step2: 理解架构设计
    ↓
step3: 查看目录结构
    ↓
step4: 学习接口文档
    ↓
step5: 实践快速开始
```

### 2.2 推荐阅读顺序

| 顺序 | 文档 | 章节 | 预计时间 | 目标 |
|------|------|------|----------|------|
| 1 | [README](README.md) | 全部 | 3 分钟 | 了解 Wiki 范围 |
| 2 | [01_Overview.md](01_Overview.md) | 1-3 节 | 5 分钟 | 理解项目定位 |
| 3 | [01_Overview.md](01_Overview.md) | 5 节 | 5 分钟 | 掌握快速开始 |
| 4 | [02_Architecture.md](02_Architecture.md) | 1-2 节 | 10 分钟 | 理解整体架构 |
| 5 | [03_CodeMap.md](03_CodeMap.md) | 1-2 节 | 5 分钟 | 熟悉代码结构 |
| 6 | [04_Interface.md](04_Interface.md) | 2-3 节 | 15 分钟 | 掌握 API 使用 |
| 7 | [02_Architecture.md](02_Architecture.md) | 3-4 节 | 10 分钟 | 理解数据流 |

### 2.3 关键知识点

| 知识点 | 关联文档 | 章节 |
|--------|----------|------|
| 项目定位 | [01_Overview.md](01_Overview.md) | 1.1 |
| 能力边界 | [01_Overview.md](01_Overview.md) | 1.3 |
| 快速开始示例 | [01_Overview.md](01_Overview.md) | 5 |
| 架构设计 | [02_Architecture.md](02_Architecture.md) | 2 |
| API 使用方法 | [04_Interface.md](04_Interface.md) | 2-3 |

### 2.4 常见问题

| 问题 | 解答位置 |
|------|----------|
| File API 是什么？ | [01_Overview.md](01_Overview.md#11-项目定位) |
| 如何使用文件 API？ | [01_Overview.md](01_Overview.md#5-快速开始) |
| @ohos.fileio 和 @ohos.file 区别？ | [04_Interface.md](04_Interface.md#2-js-api-模块) |
| 同步和异步如何选择？ | [04_Interface.md](04_Interface.md#4-编程模型) |

---

## 3. 安全研究路线

本路线专为安全研究员设计，帮助快速定位攻击面、识别安全风险。

### 3.1 研究路径

```
step1: 了解项目概览
    ↓
step2: 分析攻击面
    ↓
step3: 评估安全风险
    ↓
step4: 深入架构细节
    ↓
step5: 查看内部实现
```

### 3.2 推荐阅读顺序

| 顺序 | 文档 | 章节 | 预计时间 | 目标 |
|------|------|------|----------|------|
| 1 | [README](README.md) | 全部 | 2 分钟 | 了解文档范围 |
| 2 | [01_Overview.md](01_Overview.md) | 1-2 节 | 5 分钟 | 理解项目边界 |
| 3 | [05_AttackSurface.md](05_AttackSurface.md) | 全部 | 20 分钟 | 识别攻击入口 |
| 4 | [06_SecurityReview.md](06_SecurityReview.md) | 全部 | 30 分钟 | 评估安全风险 |
| 5 | [02_Architecture.md](02_Architecture.md) | 2-3 节 | 10 分钟 | 理解信任边界 |
| 6 | [08_Internals.md](08_Internals.md) | 2-3 节 | 15 分钟 | 分析实现细节 |

### 3.3 攻击面速查

| 攻击类型 | 入口位置 | 文档章节 |
|----------|----------|----------|
| 路径遍历 | JS 参数传入 | [05_AttackSurface.md](#31-外部输入清单) |
| 符号链接攻击 | open/read 操作 | [05_AttackSurface.md](#32-敏感操作清单) |
| 权限绕过 | URI 解析 | [05_AttackSurface.md](#33-信任边界图) |
| 资源耗尽 | 大文件操作 | [06_SecurityReview.md](#5-资源相关风险) |
| 信息泄露 | 错误信息返回 | [06_SecurityReview.md](#4-逻辑漏洞) |

### 3.4 安全检查清单

| 检查项 | 文档位置 |
|--------|----------|
| 输入验证 | [05_AttackSurface.md](#31-外部输入清单) |
| 权限校验 | [05_AttackSurface.md](#32-敏感操作清单) |
| 路径处理 | [06_SecurityReview.md](#1-输入验证缺陷) |
| 内存安全 | [06_SecurityReview.md](#2-内存安全问题) |
| 并发安全 | [06_SecurityReview.md](#4-并发安全) |

### 3.5 关键代码位置

| 风险类型 | 文件路径 | 行号 |
|----------|----------|------|
| 路径拼接 | TODO | TODO |
| 权限校验 | TODO | TODO |
| URI 解析 | TODO | TODO |
| 文件打开 | TODO | TODO |
| 读写操作 | TODO | TODO |

---

## 4. 文档索引

### 4.1 按功能分类

#### API 使用

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览、快速开始 |
| [04_Interface.md](04_Interface.md) | N-API 详细接口 |

#### 架构理解

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 架构设计、数据流 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构、代码定位 |
| [08_Internals.md](08_Internals.md) | 内部实现细节 |

#### 安全分析

| 文档 | 说明 |
|------|------|
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面清单 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 |

#### 工程实践

| 文档 | 说明 |
|------|------|
| [07_Build.md](07_Build.md) | 构建配置、编译产物 |

### 4.2 按受众分类

#### 新人适用

| 优先级 | 文档 | 说明 |
|--------|------|------|
| 必读 | [01_Overview.md](01_Overview.md) | 项目入门 |
| 必读 | [04_Interface.md](04_Interface.md) | API 参考 |
| 推荐 | [03_CodeMap.md](03_CodeMap.md) | 代码导航 |
| 推荐 | [appendix/FAQ.md](appendix/FAQ.md) | 常见问题 |

#### 安全研究员适用

| 优先级 | 文档 | 说明 |
|--------|------|------|
| 必读 | [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |
| 必读 | [06_SecurityReview.md](06_SecurityReview.md) | 风险评估 |
| 推荐 | [02_Architecture.md](02_Architecture.md) | 架构理解 |
| 推荐 | [08_Internals.md](08_Internals.md) | 实现细节 |

#### 开发者适用

| 优先级 | 文档 | 说明 |
|--------|------|------|
| 必读 | [07_Build.md](07_Build.md) | 构建配置 |
| 必读 | [04_Interface.md](04_Interface.md) | 接口文档 |
| 推荐 | [02_Architecture.md](02_Architecture.md) | 架构设计 |
| 推荐 | [08_Internals.md](08_Internals.md) | 实现参考 |

---

## 5. 相关链接

### 5.1 项目资源

| 资源 | 链接 |
|------|------|
| 项目仓库 | https://gitee.com/openharmony/filemanagement_file_api |
| 架构图 | figures/file-api-architecture.png |

### 5.2 相关仓库

| 仓库 | 说明 |
|------|------|
| [filemanagement_dfs_service](https://gitee.com/openharmony/filemanagement_dfs_service) | 分布式文件服务 |
| [filemanagement_user_file_service](https://gitee.com/openharmony/filemanagement_user_file_service) | 用户文件服务 |
| [filemanagement_storage_service](https://gitee.com/openharmony/filemanagement_storage_service) | 存储服务 |
| [filemanagement_app_file_service](https://gitee.com/openharmony/filemanagement_app_file_service) | 应用文件服务 |

### 5.3 OpenHarmony 资源

| 资源 | 链接 |
|------|------|
| OpenHarmony 官网 | https://www.openharmony.cn/ |
| 开发者文档 | https://developer.openharmony.cn/ |
| API 参考 | https://developer.openharmony.cn/development-docs/ |

---

**最后更新**：2026-02-07

**版本**：2.0