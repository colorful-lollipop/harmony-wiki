# OpenHarmony Linux 内核配置仓库 Wiki

## 项目简介

本仓库（`kernel/linux/config`）是 OpenHarmony 项目的 Linux 内核配置文件仓库，托管于 OpenHarmony 组织。本仓库包含针对不同 Linux 内核版本（4.19.y、5.10.y、6.6）的 defconfig 配置文件，用于标准系统和小系统的内核编译配置。

本仓库**不包含** Linux 内核源代码，仅存储配置文件。与内核源码仓库（`kernel/linux/linux-*.*`）和构建脚本仓库（`kernel/linux/build`）配合使用，共同完成 OpenHarmony 系统的内核适配工作。

## 适用范围

本 Wiki 适用于以下读者群体：内核移植工程师，需要将 OpenHarmony 适配到新的芯片平台或开发板；系统架构师，需要了解 OpenHarmony 内核配置的层级关系和设计思路；安全评审人员，需要评估内核配置的安全影响和合规性；以及所有希望深入理解 OpenHarmony Linux 内核配置机制的开发者。

本 Wiki **不涵盖**以下内容：Linux 内核源码分析（请参考内核源码仓库）；HDF（Hardware Driver Foundation）驱动框架细节（请参考 `drivers/hdf_core` 仓库）；GN 构建系统通用用法（请参考 OpenHarmony 构建文档）；以及测试用例相关说明（测试文件不属于本仓库范围）。

## 文档结构

本 Wiki 采用分层结构组织，按照从宏观到微观的顺序引导读者理解整个项目。主要文档包括项目概览（README）、全站导航（SUMMARY）、项目概述（01_Overview）、目录结构说明（02_Directory_Structure）、配置分层架构详解（03_Config_Layers）、构建集成说明（04_Build_Integration）以及安全评审报告（05_Security_Review）。附录文档提供配置项清单和开发板移植指南等补充资料。

## 更新方式

本 Wiki 采用**代码驱动**的更新策略，即文档内容始终跟随仓库代码变化。当有新的 defconfig 文件添加、配置项修改或目录结构调整时，应同步更新对应的 Wiki 文档。Wiki 内容的可信度来源于**代码证据**，所有关键结论都必须能在仓库内找到直接证据，包括文件路径、必要时的行号、关键符号名称以及最小必要代码片段或调用链描述。

由于本仓库变更频率相对较低（主要在 OpenHarmony 版本发布时更新），建议在以下时机更新 Wiki：新增内核版本支持时、新增开发板或芯片平台配置时、配置分层架构发生调整时，以及进行安全评审时。

## 生成信息

- **仓库路径**：`kernel/linux/config`
- **仓库地址**：`https://gitee.com/openharmony/kernel_linux_config`
- **当前版本**：OpenHarmony 主线分支
- **最后更新**：2026年2月
