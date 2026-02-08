# ArkXtest Wiki 文档

## 文档概述

本文档是 **ArkXtest**（OpenHarmony 自动化测试框架）的工程 Wiki，旨在帮助开发者快速理解项目架构、模块职责、API 用法、构建方式以及安全风险。

## 覆盖范围

本文档覆盖 ArkXtest 框架的以下核心组件：

| 组件 | 描述 | 位置 |
|------|------|------|
| **JsUnit (Hypium)** | 单元测试框架，提供测试用例编写、断言、Mock、数据驱动等能力 | `jsunit/` |
| **UiTest** | UI 自动化测试框架，支持控件查找、操作模拟、事件注入 | `uitest/` |
| **PerfTest** | 白盒性能测试框架，支持代码段性能数据采集 | `perftest/` |
| **TestServer** | 测试服务 SA，提供高权限系统能力调用接口 | `testserver/` |

## 文档结构

| 文档 | 内容 |
|------|------|
| [SUMMARY.md](SUMMARY.md) | 全站导航与新人阅读路线 |
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [03_NAPI.md](03_NAPI.md) | N-API/ANI 接口清单、参数校验、错误码 |
| [04_InnerAPI.md](04_InnerAPI.md) | 内部模块接口、依赖方向、生命周期 |
| [05_GN_Build.md](05_GN_Build.md) | BUILD.gn targets 梳理、依赖关系 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物、安装路径、运行时加载关系 |
| [07_Security.md](07_Security.md) | 攻击面分析、信任边界、风险点与修复建议 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见构建/运行/调试问题与定位路径 |

## 快速开始

### 环境要求

- OpenHarmony SDK 与构建环境
- HDC 工具（用于设备通信）
- 目标设备已连接

### 构建命令

```bash
# 从 OpenHarmony 根目录执行
cd /path/to/openharmony

# 构建 UiTest 完整组件
./build.sh --product-name rk3568 --build-target uitestkit

# 构建 PerfTest 完整组件
./build.sh --product-name rk3568 --build-target perftestkit

# 构建 TestServer
./build.sh --product-name rk3568 --build-target test_server_service
./build.sh --product-name rk3568 --build-target test_server_client
```

## 术语表

| 术语 | 说明 |
|------|------|
| **N-API** | Node.js API，ArkTS-Dynamic 应用的原生接口绑定 |
| **ANI** | ArkTS Native Interface，ArkTS-Static 应用的原生接口绑定 |
| **SA** | System Ability，系统能力服务 |
| **IPC** | Inter-Process Communication，进程间通信 |
| **ABC** | ArkTS ByteCode，ArkTS 静态字节码 |
| **HiSysEvent** | OpenHarmony 系统事件服务 |

## 版本信息

- **模块支持**: API version 8+
- **License**: Apache License 2.0
- **资源占用**: ROM ~500KB, RAM ~100KB

## 维护说明

本文档基于代码分析自动生成，文档中的 API 清单、文件路径等信息均来源于代码证据。如需更新文档，请确保：

1. 关键结论可追溯到代码证据（文件路径 + 符号）
2. N-API 文档包含完整的 API 清单表
3. GN 文档包含 targets 与产物的映射关系
4. 安全文档每条风险都有可利用路径和修复建议

---

*文档生成时间: 2024-02-06*
