# OpenHarmony Linux Kernel 5.10 Wiki

## 关于本文档

本文档是 **OpenHarmony Linux Kernel 5.10** 的工程 Wiki，旨在帮助开发者快速理解和参与该项目。

## 覆盖范围

- **版本**: Linux 5.10.210 LTS
- **项目类型**: 操作系统内核
- **License**: GPL-2.0+
- **上游**: [Linux Stable 5.10.y](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-5.10.y)

## 文档结构

| 文档 | 内容 |
|------|------|
| [首页](./index.md) | 项目概览、核心能力、运行环境 |
| [项目定位](./01_Overview.md) | 项目边界、关键概念、架构概览 |
| [目录结构](./02_Architecture.md) | 模块职责、代码组织 |
| [架构图](./03_ArchDiagrams.md) | 组件图、数据流、线程模型 |
| [对外接口](./04_PublicAPI.md) | 系统调用、proc/sysfs、ioctl |
| [内部API](./05_InnerAPI.md) | 模块接口、依赖方向 |
| [构建系统](./06_BuildSystem.md) | Kconfig、Makefile、配置选项 |
| [编译产物](./07_BuildOutputs.md) | 产物清单、安装路径、加载关系 |
| [安全评审](./08_SecurityReview.md) | 攻击面、信任边界、风险点 |
| [问题定位](./09_Troubleshooting.md) | 常见问题与调试方法 |
| [附录-调用链](./appendix/Callgraphs.md) | 关键调用链分析 |
| [附录-配置选项](./appendix/ConfigFlags.md) | 关键宏与 Feature Flags |

## 新人阅读路线

### 快速入门（30分钟）
1. [首页](./index.md) - 了解项目定位
2. [项目定位](./01_Overview.md) - 理解核心概念
3. [目录结构](./02_Architecture.md) - 熟悉代码组织

### 深度理解（2小时）
1. [架构图](./03_ArchDiagrams.md) - 理解整体架构
2. [对外接口](./04_PublicAPI.md) - 了解用户空间接口
3. [构建系统](./06_BuildSystem.md) - 掌握编译配置

### 安全审计（专项）
1. [安全评审](./08_SecurityReview.md) - 了解安全风险
2. [问题定位](./09_Troubleshooting.md) - 掌握调试方法

## 更新方式

本文档由工程 Agent 自动生成，更新时请遵循以下流程：

1. 修改代码后，同步更新对应 Wiki 文档
2. 在 `wiki/_work/NOTES.md` 中记录关键变更
3. 更新 `wiki/_work/PLAN.md` 中的任务状态
4. 确保所有链接和证据代码路径有效

## 生成时间

**生成日期**: 2026-02-06

## 贡献指南

如需向 OpenHarmony 内核贡献代码：

1. 阅读根目录 `README` 文件中的贡献指南
2. 签署 DCO: https://dco.openharmony.io/sign-dco
3. 发送补丁至: kernel@openharmony.io
4. 订阅邮件列表: https://lists.openatom.io/postorius/lists/kernel.openharmony.io/

## 未覆盖范围

- 具体硬件驱动实现细节（数千个驱动）
- 各体系架构特有代码（arch/ 下的详细实现）
- 测试代码（test/、tests/ 目录）
- 上游内核文档已详细覆盖的内容
