# OpenHarmony Misc Device Wiki

## 概述

本文档是 OpenHarmony `miscdevice` 子系统的工程 Wiki，提供完整的技术架构、API 接口、构建配置和安全分析文档。

## 项目定位

`miscdevice` 是 OpenHarmony sensors 子系统的杂项设备模块，负责管理：

- **振动器 (Vibrator)**: 提供触觉反馈功能，支持预设振动效果和自定义振动模式
- **LED 指示灯 (Light)**: 控制设备 LED 灯光显示

## 版本信息

- **组件版本**: 3.1
- **系统能力**: 
  - `SystemCapability.Sensors.MiscDevice`
  - `SystemCapability.Sensors.MiscDevice.Lite`
- **License**: Apache License 2.0
- **资源占用**: ROM ~1024KB, RAM ~4096KB

## 文档覆盖范围

### 已覆盖内容
- 项目架构与模块职责
- 多语言 API 绑定层 (JS N-API、C API、Taihe ArkTS、Cangjie FFI)
- System Ability 服务层实现
- IPC/Binder 通信机制
- GN 构建配置与编译产物
- 权限控制与安全机制

### 未覆盖内容
- 测试代码 (`test/` 目录)
- HDF 驱动层实现 (位于 `drivers/` 子系统)
- 设备树配置 (位于 `device/` 子系统)

## 读者对象

- **应用开发者**: 需要使用振动器/灯光 API
- **框架开发者**: 需要理解 Native 客户端实现
- **安全审计人员**: 需要了解攻击面与安全控制
- **系统集成工程师**: 需要理解 SA 注册与 IPC 通信

## 快速导航

| 主题 | 文档 | 说明 |
|------|------|------|
| 概览 | [00_Overview](00_Overview.md) | 项目定位、架构图、核心概念 |
| 架构 | [01_Architecture](01_Architecture.md) | 分层架构、数据流、线程模型 |
| API | [02_NAPI](02_NAPI.md) | JS/C/ArkTS API 详细文档 |
| 构建 | [03_Build](03_Build.md) | GN Targets、编译产物、Feature Flags |
| 安全 | [04_Security](04_Security.md) | 攻击面分析、风险评估、修复建议 |

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-06 | 1.0 | 初始版本，完成基础架构文档 |

## 文档生成

- **生成时间**: 2026-02-06
- **源码版本**: 基于 `gitee.com/openharmony/sensors_miscdevice`
- **生成方式**: 自动从代码分析生成

## 贡献指南

如需更新本文档，请：

1. 修改对应 `.md` 文件
2. 确保关键结论有代码证据（文件路径 + 行号）
3. 保持术语一致
4. 运行 `lsp_diagnostics` 验证链接有效性
