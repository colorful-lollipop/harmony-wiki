# FlatBuffers OpenHarmony Wiki

## 库概述

FlatBuffers 是 Google 开发的高性能跨平台序列化库，在 OpenHarmony 系统中主要服务于 AI 相关模块，提供高效的模型数据序列化和反序列化能力。

| 项目 | 内容 |
|-----|------|
| **版本** | v25.2.10 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/google/flatbuffers |
| **OH 组件** | @ohos/flatbuffers |
| **子系统** | thirdparty |

## OpenHarmony 适配要点

### Patch 概览

| Patch 文件 | 主要修改 | 分类 |
|-----------|---------|------|
| `grpc/build_grpc_with_cxx14.patch` | 显式指定 C++14 编译标准 | 构建适配 |
| `grpc/boringssl.patch` | 修复 BoringSSL 链接和编译问题 | Bugfix |

### 主要依赖者

- **MindSpore Lite**: AI 推理引擎，MindIR 模型解析
- **Neural Network Runtime**: 神经网络运行时，模型数据序列化

### OH 特有特性

- **仓颉语言绑定**: `cangjie/` 目录提供仓颉语言支持
- **GN 构建适配**: 原生支持 OpenHarmony GN 构建系统

## 文档导航

### 快速入门

建议阅读顺序：

1. **[概述](01_Overview.md)** - 了解 FlatBuffers 及其在 OH 中的定位
2. **[Patch 详解](02_Patches.md)** - 深入分析 OH 对上游的修改
3. **[构建适配](03_Build_Integration.md)** - 了解 GN 构建配置
4. **[OH 使用场景](04_Usage_in_OH.md)** - 查看依赖关系和使用方式

### 详细文档

- [Patch 详细分析](02_Patches.md) - 所有 Patch 的逐行分析
- [构建系统详解](03_Build_Integration.md) - BUILD.gn 配置详解
- [依赖关系图](04_Usage_in_OH.md) - OH 模块依赖关系

## 相关链接

- [上游官方文档](https://google.github.io/flatbuffers/)
- [上游 GitHub](https://github.com/google/flatbuffers)
- [OpenHarmony 第三方库规范](../README.md)
