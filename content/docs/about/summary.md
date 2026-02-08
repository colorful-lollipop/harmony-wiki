---
title: "全站导航"
type: docs
---

# OpenHarmony 文档仓库 Wiki - 全站导航

> 本文档提供 Wiki 全站导航，包含新人阅读顺序建议。

## 快速开始

### 📖 新人学习路线

适合初次了解本仓库的开发者：

1. [README - Wiki 说明](readme/) - 了解 Wiki 覆盖范围
2. [项目概览](readme/) - 了解项目背景
3. [目录结构](structure/) - 熟悉仓库结构
4. [内容体系](content/) - 了解文档分类
5. [贡献指南](contribute/) - 学习如何贡献

### 🔒 安全研究路线

适合安全研究员快速定位安全相关内容：

1. [项目概览](readme/) - 了解仓库性质
2. [内容体系](content/) - 查阅安全规范章节
3. 官方安全规范文档：
   - [C/C++ 安全编码规范](../zh-cn/contribute/OpenHarmony-c-cpp-secure-coding-guide.md)
   - [安全设计指南](../zh-cn/contribute/OpenHarmony-security-design-guide.md)
   - [安全测试指南](../zh-cn/contribute/OpenHarmony-security-test-guide.md)

### 🚀 贡献者快速路线

适合准备贡献文档的开发者：

1. [贡献指南](contribute/) - 完整贡献流程
2. [appendix/Templates.md](../appendix/Templates/) - 文档模板参考
3. [内容体系](content/) - 了解文档分类

## 目录

### 核心文档

| 文档 | 描述 | 阅读建议 |
|------|------|----------|
| [README](readme/) | Wiki 说明、覆盖范围、更新方式 | ⭐ 必读 |
| [项目概览](readme/) | 项目定位、版本历史、关键概念 | ⭐ 必读 |
| [目录结构](structure/) | 目录结构、模块职责 | ⭐ 必读 |
| [内容体系](content/) | 文档内容分类、命名规范 | 📋 参考 |
| [构建与发布](build/) | 构建工具、CI/CD、发布流程 | 🔧 开发用 |
| [贡献指南](contribute/) | 贡献方式、审批流程、质量标准 | 🤝 贡献者必读 |

### 附录

| 文档 | 描述 | 标签 |
|------|------|------|
| [History.md](../appendix/History/) | 版本发布历史详情 | 📜 历史 |
| [Templates.md](../appendix/Templates/) | 文档模板、写作规范 | 📝 模板 |

## 与官方文档的关联

### 官方资源

| 官方资源 | 链接 | 说明 |
|----------|------|------|
| OpenHarmony 官网 | https://www.openharmony.cn/ | 官方网站 |
| 中文文档导航 | [zh-cn/readme.md](../zh-cn/readme.md) | 中文文档入口 |
| 英文文档导航 | [en/readme.md](../en/readme.md) | 英文文档入口 |
| 贡献指南 | [zh-cn/contribute/参与贡献.md](../zh-cn/contribute/参与贡献.md) | 官方贡献指南 |

### 技术专题文档

| 专题 | 英文入口 | 中文入口 | 说明 |
|------|----------|----------|------|
| N-API 开发 | [napi/Readme-EN.md](../en/application-dev/napi/Readme-EN.md) | TODO | Node-API 完整文档（100+ 文件） |
| IPC/RPC | [ipc-rpc-overview.md](../en/application-dev/ipc/ipc-rpc-overview.md) | TODO | 进程间通信开发指南 |
| 应用开发 | [application-dev/Readme-EN.md](../en/application-dev/Readme-EN.md) | [application-dev/Readme-CN.md](../zh-cn/application-dev/Readme-CN.md) | 应用开发完整指南 |
| 设备开发 | [device-dev/Readme-EN.md](../en/device-dev/Readme-EN.md) | [device-dev/Readme-CN.md](../zh-cn/device-dev/Readme-CN.md) | 设备开发完整指南 |
| 设计规范 | [design/Readme-EN.md](../en/design/OpenHarmony-part-design.md) | TODO | 部件设计、API 治理规范 |

### 安全相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| C/C++ 安全编码规范 | [zh-cn/contribute/OpenHarmony-c-cpp-secure-coding-guide.md](../zh-cn/contribute/OpenHarmony-c-cpp-secure-coding-guide.md) | 安全编码最佳实践 |
| 安全设计指南 | [zh-cn/contribute/OpenHarmony-security-design-guide.md](../zh-cn/contribute/OpenHarmony-security-design-guide.md) | 安全架构设计指导 |
| 安全测试指南 | [zh-cn/contribute/OpenHarmony-security-test-guide.md](../zh-cn/contribute/OpenHarmony-security-test-guide.md) | 安全测试方法论 |

## 贡献此 Wiki

1. 在 `_work/NOTES.md` 中记录发现的事实
2. 更新 `_work/PLAN.md` 勾选进度
3. 修改 `wiki/` 目录下的相应文件
4. 提交 PR 或直接编辑

## 术语表

| 术语 | 说明 |
|------|------|
| API Level | OpenHarmony API 等级 |
| LTS | Long Term Support，长期支持版本 |
| XTS | X Test Suite，兼容性测试套件 |
| N-API | Node-API，用于 C/C++ 扩展的 API |
| IPC | Inter-Process Communication，进程间通信 |
| RPC | Remote Procedure Call，远程过程调用 |
| HDI | Hardware Driver Interface，硬件驱动接口 |

---

## 工作区

| 文件 | 描述 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 事实记录（代码证据汇总） |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

*最后更新：2026-02-07*
