# Cast+ Stream Wiki

> OpenHarmony CastEngine 投屏流媒体模块工程文档
> 
> **生成时间**: 2026-02-07  
> **模块路径**: `/foundation/CastEngine/castengine_cast_plus_stream`  
> **版本**: 基于代码仓库最新状态  
> **更新说明**: 安全评审文档已更新，补充了更精确的代码证据

---

## 文档说明

本 Wiki 是针对 OpenHarmony CastEngine 项目中 `castengine_cast_plus_stream` 模块的工程文档，旨在帮助开发者快速理解项目结构、架构设计、接口定义和安全机制。

### 覆盖范围

- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构设计（组件图、数据流、状态机）
- ✅ 对外 API（IPC 接口）
- ✅ 内部模块接口与依赖关系
- ✅ GN 构建系统与编译产物
- ✅ 安全风险评审
- ✅ 常见问题与调试指南

### 未覆盖范围

- ❌ N-API 层（本模块为纯原生模块，无 JS 绑定）
- ❌ 测试代码（`test/` 目录）
- ❌ 父级框架（`castengine_cast_framework`）的 N-API 包装层

### 如何更新本文档

本文档基于代码分析自动生成。当代码发生变更时：

1. 重新运行分析工具收集最新信息
2. 更新 `wiki/_work/NOTES.md` 中的事实记录
3. 同步更新各章节文档
4. 更新 `wiki/README.md` 中的生成时间

---

## 快速导航

| 文档 | 内容 |
|------|------|
| [SUMMARY.md](./SUMMARY.md) | 全站导航与阅读路线 |
| [00_Overview.md](./00_Overview.md) | 项目概览与核心概念 |
| [01_Project_Positioning.md](./01_Project_Positioning.md) | 项目定位与边界 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构与模块职责 |
| [03_Architecture.md](./03_Architecture.md) | 架构设计与数据流 |
| [04_External_API.md](./04_External_API.md) | 对外 IPC API |
| [05_Internal_API.md](./05_Internal_API.md) | 内部模块接口 |
| [06_GN_Targets.md](./06_GN_Targets.md) | GN 构建目标 |
| [07_Build_Artifacts.md](./07_Build_Artifacts.md) | 编译产物 |
| [08_Security_Review.md](./08_Security_Review.md) | 安全风险评审 |
| [09_Troubleshooting.md](./09_Troubleshooting.md) | 问题定位指南 |
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 配置与宏定义 |

---

## 项目简介

**Cast+ Stream** 是 OpenHarmony CastEngine 的核心模块，负责实现媒体资源的投射到对端设备，并支持双端的播控。

### 主要功能

- **镜像投屏 (Mirror Cast)**: 将设备屏幕实时投射到对端
- **流媒体投屏 (Stream Cast)**: 将媒体文件投射到对端播放
- **双端播控**: 支持源端和接收端的播放控制
- **RTSP 协议**: 使用 RTSP 进行会话控制和参数协商
- **多通道传输**: 支持 SoftBus 和 TCP 两种传输方式

### 许可证

Apache License 2.0  
Copyright (c) 2024 Huawei Device Co., Ltd.

---

## 相关仓库

- [castengine_cast_framework](https://gitee.com/openharmony-sig/castengine_cast_framework) - CastEngine 框架
- [castengine_wifi_display](https://gitee.com/openharmony-sig/castengine_wifi_display) - WiFi Display 实现
- [castengine_dlna](https://gitee.com/openharmony-sig/castengine_dlna) - DLNA 协议支持
