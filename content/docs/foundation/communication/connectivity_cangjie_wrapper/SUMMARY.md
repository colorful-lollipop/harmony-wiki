# 全站导航

> connectivity_cangjie_wrapper 工程 Wiki 完整导航

## 快速入口

- [首页](./index.md)
- [项目 README](../README.md)
- [API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_en/apis/ConnectivityKit)

## 文档目录

### 基础信息

| 章节 | 标题 | 说明 |
|------|------|------|
| 01 | [项目概览](./01_Project_Overview.md) | 项目定位、核心能力、运行环境、依赖关系 |
| 02 | [目录结构](./02_Directory_Structure.md) | 模块划分、文件布局、职责边界 |

### 架构与设计

| 章节 | 标题 | 说明 |
|------|------|------|
| 03 | [架构说明](./03_Architecture.md) | 组件图、数据流、线程模型、关键时序 |

### API 参考

| 章节 | 标题 | 说明 |
|------|------|------|
| 04 | [N-API 参考](./04_N-API_Reference.md) | Cangjie API 清单、参数校验、错误码 |
| 05 | [内部 API](./05_Inner_API.md) | 模块接口、依赖方向、稳定性标注 |

### 工程相关

| 章节 | 标题 | 说明 |
|------|------|------|
| 06 | [构建系统](./06_Build_System.md) | GN Targets、编译产物、依赖关系 |
| 07 | [安全评审](./07_Security_Review.md) | 攻击面分析、风险点、修复建议 |
| 08 | [FAQ 与排错](./08_FAQ_Troubleshooting.md) | 常见问题、定位路径、调试方法 |

### 附录

| 章节 | 标题 | 说明 |
|------|------|------|
| A1 | [关键调用链](./appendix/Callgraphs.md) | 入口→核心逻辑的调用链路 |

## 新人阅读路线

```
┌─────────────────────────────────────────────────────────────────┐
│                        新人阅读路线                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ① 项目概览                                                     │
│     └→ 了解项目定位、核心能力、支持设备类型                      │
│                                                                 │
│  ② 目录结构                                                     │
│     └→ 理解模块划分、文件布局                                    │
│                                                                 │
│  ③ N-API 参考                                                   │
│     └→ 学习具体 API 的使用方式                                   │
│                                                                 │
│  ④ FAQ 与排错（遇到问题时）                                      │
│     └→ 快速定位和解决问题                                        │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  深入学习（可选）                                                │
│                                                                 │
│  → 架构说明（理解设计思想）                                      │
│  → 内部 API（了解模块间依赖）                                    │
│  → 构建系统（理解编译流程）                                      │
│  → 安全评审（确保安全使用）                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 模块快速跳转

### 蓝牙模块

| 模块 | 说明 | 相关文档 |
|------|------|----------|
| BLE | 低功耗蓝牙 | [N-API 参考 - BLE](./04_N-API_Reference.md#ble-低功耗蓝牙) |
| A2DP | 音频分发配置 | [N-API 参考 - A2DP](./04_N-API_Reference.md#a2dp-音频分发配置) |
| HFP | 免提配置 | [N-API 参考 - HFP](./04_N-API_Reference.md#hfp-免提配置) |
| Connection | 连接管理 | [N-API 参考 - Connection](./04_N-API_Reference.md#connection-连接管理) |
| Constant | 常量定义 | [内部 API - Constant](./05_Inner_API.md#constant-蓝牙常量) |

### WLAN 模块

| 模块 | 说明 | 相关文档 |
|------|------|----------|
| P2P | 点对点连接 | [N-API 参考 - WiFi](./04_N-API_Reference.md#wlan-p2p) |

### Kit 层

| 模块 | 说明 | 相关文档 |
|------|------|----------|
| ConnectivityKit | 统一入口 | [架构说明](./03_Architecture.md#kit-层) |

## 贡献指南

欢迎贡献代码和文档！贡献前请阅读：

1. [OpenHarmony 代码贡献指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/contribute/代码贡献指南.md)
2. 本 Wiki 的 [README](./README.md) 中的更新方式说明

## 反馈与建议

如发现文档错误或有改进建议，请通过以下方式反馈：

- 在 [Gitee Issues](https://gitee.com/openharmony/connectivity_cangjie_wrapper/issues) 中提交问题
- 完善本文档后提交 Pull Request
