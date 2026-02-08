# Protocol Buffers (Protobuf) - OpenHarmony 适配文档

## 库概览

Protocol Buffers（ protobuf ）是 Google 开发的一种语言中立、平台中立的可扩展机制，用于序列化结构化数据。它是 OpenHarmony 系统中最重要的数据序列化库之一，被广泛应用于系统各模块的进程间通信（IPC）、数据持久化和网络传输等场景。

本 wiki 专注于记录 **OpenHarmony 对 protobuf 的适配和定制化内容**，包括 OH 特有的构建配置、依赖关系和使用场景。

## OpenHarmony 适配概述

| 维度 | 状态 |
|------|------|
| **上游版本** | 5.29.4 |
| **OH 版本** | 3.1 |
| **Patch 数量** | 1 个（ Bazel 兼容修复） |
| **源码修改** | 无 OH 特有源码修改 |
| **构建系统** | 完全适配 BUILD.gn |
| **依赖模块数** | 188+ BUILD.gn 文件 |

### 核心适配点

1. **构建系统适配**: 使用 `ohos_shared_library` 和 `ohos_static_library` 构建多种变体
2. **HILOG 集成**: 目标构建时启用日志功能
3. **安全加固**: PAC-RET 分支保护应用于 lite 版本
4. **多目标支持**: 同时提供 shared 和 static 链接选项

## 文档导航

| 文档 | 内容说明 |
|------|----------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议和文档索引 |
| [01_Overview.md](./01_Overview.md) | 原始库简介和 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（核心文档） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异分析 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速开始

### 在 OH 模块中引入 Protobuf

```gn
# 静态链接（推荐）
external_deps = [ "protobuf:protobuf_lite_static" ]

# 动态链接
external_deps = [ "protobuf:protobuf_lite" ]
```

### Protobuf Lite vs Full

| 版本 | 适用场景 | 功能差异 |
|------|----------|----------|
| **protobuf_lite** | 资源受限设备、移动端 | 去除反射、描述符等特性 |
| **protobuf_full** | 需要完整功能的场景 | 支持反射、自描述等全部特性 |

## 版本信息

- **上游项目**: [Protocol Buffers](https://github.com/protocolbuffers/protobuf)
- **上游版本**: 5.29.4
- **许可证**: BSD 3-Clause License
- **维护者**: OpenHarmony thirdparty 团队

## 相关资源

- [ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估原始记录
- [bundle.json](../bundle.json) - OH 组件配置
- [BUILD.gn](../BUILD.gn) - 构建配置
