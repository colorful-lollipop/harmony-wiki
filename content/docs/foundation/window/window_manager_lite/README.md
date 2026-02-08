# OpenHarmony Window Manager Lite - Wiki 文档

## 文档说明

本文档为 OpenHarmony `window_manager_lite` 子系统的工程 Wiki，面向两类读者：

1. **新人学习者** - 快速理解项目是什么、能做什么、怎么用
2. **安全研究员** - 识别攻击面、信任边界、潜在漏洞点

## 双路线导航

### 新人学习路线

| 阶段 | 文档 | 时间 | 目标 |
|------|------|------|------|
| 入门 | [01_Overview](./01_Overview.md) | 5分钟 | 理解项目定位和核心能力 |
| 结构 | [02_Directory_Structure](./02_Directory_Structure.md) | 15分钟 | 掌握模块划分和代码位置 |
| 架构 | [03_Architecture](./03_Architecture.md) | 30分钟 | 理解 C/S 架构和数据流 |
| 接口 | [04_Inner_API](./04_Inner_API.md) | - | 学习 API 使用方式 |
| 构建 | [05_GN_Build](./05_GN_Build.md) | - | 了解编译配置和产物 |

### 安全研究路线

| 阶段 | 文档 | 重点 |
|------|------|------|
| 概览 | [01_Overview](./01_Overview.md) + [03_Architecture](./03_Architecture.md) | 对外暴露面、信任边界 |
| 攻击面 | [07_Security](./07_Security.md) 攻击面章节 | 外部输入入口、敏感操作 |
| 风险评估 | [07_Security](./07_Security.md) 风险章节 | 可利用点、修复建议 |

## 文档清单

| 文档 | 说明 | 新人 | 安全 |
|------|------|------|------|
| [01_Overview](./01_Overview.md) | 项目概述、核心能力、运行环境 | ⭐⭐⭐ | ⭐⭐ |
| [02_Directory_Structure](./02_Directory_Structure.md) | 目录结构、模块职责、文件清单 | ⭐⭐⭐ | ⭐⭐ |
| [03_Architecture](./03_Architecture.md) | 架构设计、组件图、数据流、线程模型 | ⭐⭐⭐ | ⭐⭐⭐ |
| [04_Inner_API](./04_Inner_API.md) | 模块间接口定义和使用 | ⭐⭐ | ⭐⭐ |
| [05_GN_Build](./05_GN_Build.md) | GN 构建配置、编译产物 | ⭐⭐ | ⭐ |
| [06_IMS](./06_IMS.md) | 输入管理服务详解 | ⭐⭐ | ⭐⭐ |
| [07_Security](./07_Security.md) | 攻击面分析、风险评估、修复建议 | ⭐ | ⭐⭐⭐ |

## 关键信息速查

### 项目基础
- **组件名**: @ohos/window_manager_lite
- **版本**: 3.1
- **适配系统**: small（轻量级设备）
- **代码规模**: ~2800 行
- **ROM**: 110KB
- **RAM**: ~50KB

### 核心能力
- **WMS**: 窗口生命周期管理、几何操作、截图
- **IMS**: 输入设备管理、事件分发

### IPC 接口 (14个)
`CreateWindow`, `RemoveWindow`, `Show`, `Hide`, `MoveTo`, `Resize`, `RaiseToTop`, `LowerToBottom`, `Update`, `GetSurface`, `GetEventData`, `Screenshot`, `ClientRegister`, `GetLayerInfo`

### 安全要点
- 仅 Screenshot 有权限检查 (`ohos.permission.WRITE_MEDIA_IMAGES`)
- CreateWindow 无权限校验（潜在风险）
- 最大 32 个窗口限制

## 相关链接

- [OpenHarmony 图形子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E5%9B%BE%E5%BD%A2%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [Graphic Surface Lite](https://gitee.com/openharmony/graphic_surface_lite)
- [ArkUI UI Lite](https://gitee.com/openharmony/arkui_ui_lite)

---

**文档生成时间**: 2024-02-06  
**最后更新**: 2024-02-07（添加双路线导航）
