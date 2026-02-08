# NFC 组件 Wiki

## 简介

本文档是 OpenHarmony NFC（Near Field Communication，近场通信）组件的工程 Wiki，提供从架构到实现的完整技术文档，帮助开发者快速理解和使用 NFC 组件。

## 文档范围

### 已覆盖内容
- 项目定位与核心能力
- 架构设计与组件关系
- 目录结构与模块职责
- N-API 接口详细文档
- 内部架构与模块依赖
- GN 构建目标与产物
- 安全风险评估
- 常见问题与调试方法

### 未覆盖内容
- 测试代码详解（test/ 目录内容）
- 具体硬件厂商适配指南
- 性能优化专项
- 第三方应用开发教程（请参考官方应用开发文档）

## 更新方式

本文档基于代码生成，建议随代码版本同步更新：

1. **代码变更时更新**：当接口、架构或安全相关代码变更时，同步更新对应文档
2. **定期复核**：建议每个版本发布前复核文档与代码一致性
3. **反馈机制**：发现文档与代码不符，请提交 Issue 或 PR

## 生成信息

- **生成时间**：2025年2月
- **代码版本**：OpenHarmony 5.0+ 
- **组件版本**：@ohos/nfc v3.1
- **仓库路径**：foundation/communication/nfc

## 快速导航

| 文档 | 内容 | 适用读者 |
|------|------|----------|
| [概述](00_Overview.md) | 项目定位、边界、核心能力 | 所有开发者 |
| [架构设计](01_Architecture.md) | 组件图、数据流、线程模型 | 架构师、资深开发 |
| [目录结构](02_Directory_Structure.md) | 模块划分、文件组织 | 新成员 |
| [N-API 接口](03_NAPI_Interfaces.md) | JS API 清单、参数、调用链 | 应用开发者 |
| [内部接口](04_Inner_API.md) | 模块接口、依赖方向 | 系统开发者 |
| [构建系统](05_GN_Targets.md) | GN 目标、产物、安装路径 | 构建工程师 |
| [安全评估](06_Security_Assessment.md) | 攻击面、风险、修复建议 | 安全工程师 |
| [构建产物](07_Build_Artifacts.md) | 输出文件、加载关系 | 系统集成 |
| [问题排查](08_Troubleshooting.md) | 常见问题、调试方法 | 所有开发者 |

## 术语表

| 术语 | 说明 |
|------|------|
| NFC | Near Field Communication，近场通信 |
| NCI | NFC Controller Interface，NFC 控制器接口 |
| HCE | Host Card Emulation，主机卡模拟 |
| NDEF | NFC Data Exchange Format，NFC 数据交换格式 |
| SA | System Ability，系统能力 |
| IDL | Interface Definition Language，接口定义语言 |
| N-API | Node-API，JavaScript 原生扩展接口 |
| AID | Application ID，应用标识符 |
| APDU | Application Protocol Data Unit，应用协议数据单元 |

