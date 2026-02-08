# OpenHarmony 分布式音频 Wiki

## 简介

本文档是 OpenHarmony **distributed_audio**（分布式音频）组件的工程 Wiki，提供项目架构、接口说明、构建系统和安全风险分析的全面参考。

## 生成信息

- **生成时间**: 2025-02-07
- **代码版本**: 4.0
- **仓库路径**: foundation/distributedhardware/distributed_audio
- **子系统**: distributedhardware

## 重要发现摘要

### 🔴 高危安全风险（已确认）

1. **[DAudioNotifyInner 权限缺失](08_Security_Review.md#8-daudionotifyinner-接口缺少权限检查)**
   - 位置: `services/audiomanager/servicesource/src/daudio_source_stub.cpp:174`
   - 影响: 任何应用均可发送音频事件通知，干扰分布式音频状态

2. **[UpdateWorkMode 权限逻辑错误](08_Security_Review.md#9-updatedaudioworkmodeinner-权限检查逻辑错误)**
   - 位置: `services/audiomanager/servicesource/src/daudio_source_stub.cpp:189`
   - 影响: 权限检查逻辑完全反转，未授权访问可修改工作模式

### 📊 项目特征

- **类型**: SystemAbility 系统服务 (SA 4805/4806)
- **语言**: C++ (无 N-API/JS 接口)
- **架构**: Source/Sink 双端分布式架构
- **IPC 接口**: 18 个接口方法（Source 7 个 + Sink 11 个）

## 覆盖范围

### 已覆盖内容

- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构说明（组件图、数据流、线程模型）
- ✅ IPC 接口（SA 4805/4806）
- ✅ Inner API（Native C++ SDK）
- ✅ GN 构建系统与编译产物
- ✅ 安全风险评审（攻击面、信任边界、可利用点）

### 未覆盖内容

- ❌ N-API 接口（本项目无 JS 接口）
- ❌ 测试代码（已按约束排除）
- ❌ 详细实现代码逐行分析

## 阅读建议

### 新用户入门路线

1. [项目概览](00_Overview.md) - 了解分布式音频基本概念
2. [项目定位与边界](01_Project_Boundary.md) - 明确项目职责和运行环境
3. [目录结构](02_Directory_Structure.md) - 了解代码组织方式
4. [架构说明](03_Architecture.md) - 深入理解系统架构

### 开发者参考路线

1. [对外 API](04_Public_API.md) - 了解 IPC 接口定义
2. [内部 API](05_Inner_API.md) - 了解模块间接口
3. [GN Targets](06_GN_Targets.md) - 了解构建系统
4. [编译产物](07_Build_Artifacts.md) - 了解输出文件

### 安全审计路线

1. [安全风险评审](08_Security_Review.md) - 全面了解安全风险
2. [信任边界](08_Security_Review.md#信任边界) - 了解关键检查点

## 更新方式

本文档基于代码仓库自动生成，建议随代码版本更新而更新。更新步骤：

1. 重新运行代码扫描工具
2. 更新关键接口和架构变化
3. 同步安全分析结果
4. 更新本文档的生成时间和版本信息

## 术语表

| 术语 | 说明 |
|------|------|
| Source | 主控端，发送音频到远端设备 |
| Sink | 被控端，接收远端音频并播放 |
| SA | System Ability，系统能力服务 |
| HDF | Hardware Driver Framework，硬件驱动框架 |
| HDI | Hardware Driver Interface，硬件驱动接口 |
| IPC | Inter-Process Communication，进程间通信 |
| DFX | Diagnostics, Feedback, eXtension，诊断反馈扩展 |

## 相关链接

- [OpenHarmony 官网](https://www.openharmony.cn)
- [分布式硬件框架](https://gitee.com/openharmony/distributedhardware_distributed_hardware_fwk)
- [音频框架](https://gitee.com/openharmony/multimedia_audio_framework)

---

*本文档由工程 Agent 自动生成，如有疑问请参考源码或联系维护者。*
