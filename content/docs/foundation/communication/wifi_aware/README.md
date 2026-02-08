# Wi-Fi Aware 模块 Wiki

## 项目概述

本文档描述 OpenHarmony Wi-Fi Aware 模块的架构、API、编译和安全分析。

**注意**: 本模块是**纯 C 原生库**，不包含 N-API（JavaScript）绑定。JS API 封装位于 OpenHarmony 其他仓库。

## 覆盖范围

| 文档 | 内容 |
|-----|------|
| [README](README.md) | 本文档，覆盖范围说明 |
| [SUMMARY](SUMMARY.md) | 全站导航 |
| [index](index.md) | 项目概览 |
| [架构](architecture.md) | 模块架构、层次、数据流 |
| [API 文档](api.md) | C API 参考（无 N-API） |
| [HAL 接口](hal.md) | 硬件抽象层接口 |
| [构建文档](build.md) | GN Targets 与编译产物 |
| [安全评审](security.md) | 安全风险分析 |

## 代码证据

所有关键结论均可追溯到以下源文件：

| 文件 | 路径 | 说明 |
|-----|------|-----|
| 框架实现 | `frameworks/source/wifiaware.c` | 172 行 |
| 对外 API | `interfaces/kits/wifiaware.h` | 278 行 |
| HAL 接口 | `hals/hal_wifiaware.h` | 63 行 |
| 构建配置 | `BUILD.gn` | 根构建文件 |
| 模块配置 | `bundle.json` | 组件元数据 |

## 版本信息

- **模块版本**: 3.1.0
- **基础 API 版本**: 1.0
- **扩展 API 版本**: 2.2
- **支持芯片**: Hi3861（当前仅此）
- **适用系统**: small, standard

## 更新说明

本文档随代码变更更新。使用 OpenHarmony GN 构建系统编译后可验证。

**生成时间**: 2025-02-06
