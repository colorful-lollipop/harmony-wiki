# Node.js（N-API 头文件库）

## 项目概述

本项目是 Node.js 的 N-API（Native API）头文件在 OpenHarmony 系统中的集成版本。Node.js v18.20.1 对应的 N-API 版本为 8，提供了 C/C++ 模块与 JavaScript 运行时交互的标准接口规范。

**重要说明**：本库仅包含头文件定义，不包含任何可执行源代码。所有内容均直接来自 Node.js 官方发布，未进行任何代码修改。

## OpenHarmony 适配定位

在 OpenHarmony 生态系统中，该库承担着 **Native 与 JavaScript/ArkTS 互操作基础设施** 的核心角色。任何需要将 C/C++ 能力暴露给上层应用的系统模块，都必须通过 N-API 接口实现。

## 文档导航

### 快速入门

- [阅读路线建议](SUMMARY.md) — 根据您的需求选择合适的文档阅读路径
- [项目评估结果](_work/ASSESSMENT.md) — 完整的技术评估报告

### 核心内容

- [原始库简介](01_Overview.md) — Node.js N-API 的功能概述
- [Patch 详细分析](02_Patches.md) — OpenHarmony 定制化代码说明
- [OH 构建适配](03_Build_Integration.md) — BUILD.gn 构建配置详解
- [依赖关系与使用](04_Usage_in_OH.md) — 在 OH 系统中的使用场景

## 关键特性

| 特性 | 说明 |
|------|------|
| 接口类型 | N-API（Native API）v8 |
| ABI 稳定性 | 完全 ABI 稳定，跨引擎兼容 |
| OH Patch | 无（零 Patch 策略） |
| 依赖模块数 | 15+ OpenHarmony 核心模块 |
| 许可证 | 多种许可证（详见 LICENSE 文件） |

## 版本信息

| 属性 | 值 |
|------|-----|
| Node.js 版本 | v18.20.1 |
| OH 组件版本 | 3.1 |
| N-API 版本 | 8 |
| 上游地址 | https://nodejs.org/ |

## 常见问题

### 这个库包含 Node.js 源代码吗？

不包含。本库仅包含 N-API 头文件定义，用于 C/C++ 模块与 JavaScript 运行时交互。Node.js 本身的源代码不在此仓库中。

### 为什么没有 Patch？

N-API 被设计为 ABI 稳定的跨引擎接口标准，其头文件定义不依赖于特定的 JavaScript 引擎实现。OpenHarmony 自研的 Ark 引擎完全兼容 N-API 标准，因此无需任何修改即可直接使用。

### 如何在 Native 模块中使用这些接口？

在您的 BUILD.gn 中添加以下依赖配置：

```gn
include_dirs += [ "//third_party/node/src" ]
```

然后在 C/C++ 代码中包含相应的头文件：

```c
#include <node_api.h>
#include <js_native_api.h>
```

## 相关资源

- [Node.js N-API 官方文档](https://nodejs.org/api/n-api.html)
- [OpenHarmony N-API 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/README.md)
- [Ark 引擎文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/runtime-runtime/README.md)

## 许可证

本库包含多种许可证内容，详细信息请参阅 [LICENSE](../LICENSE) 文件。主要许可证包括：ISC、MIT、BSD、Apache 2.0、ICU、zlib 等。
