# 全站导航

> 本文档提供 Wiki 全站导航，包含所有页面的链接和阅读建议。

## 文档索引

### 入门指南

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [README.md](./README.md) | Wiki 使用指南 | 2 分钟 |
| [00_Overview.md](./00_Overview.md) | 项目定位与核心能力 | 5 分钟 |

### 架构设计

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [01_Architecture.md](./01_Architecture.md) | 组件图、数据流、线程模型 | 15 分钟 |
| [03_InnerAPI.md](./03_InnerAPI.md) | 内部模块接口与依赖 | 10 分钟 |

### 接口文档

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [02_N-API.md](./02_N-API.md) | N-API 接口清单与使用 | 20 分钟 |

### 构建部署

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [04_Build.md](./04_Build.md) | GN targets 与编译产物 | 10 分钟 |

### 安全评估

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [05_Security.md](./05_Security.md) | 攻击面与风险分析 | 15 分钟 |

## 新人阅读路线

```
┌─────────────────────────────────────────────────────────────┐
│  第 1 天: 快速入门                                           │
├─────────────────────────────────────────────────────────────┤
│  1. README.md → 了解 Wiki 结构                               │
│  2. 00_Overview.md → 理解项目定位                           │
│  3. 01_Architecture.md → 掌握整体架构                       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  第 2-3 天: API 学习                                        │
├─────────────────────────────────────────────────────────────┤
│  4. 02_N-API.md → 熟悉 N-API 接口                           │
│  5. 03_InnerAPI.md → 了解内部模块                          │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  第 4 天: 深入理解                                          │
├─────────────────────────────────────────────────────────────┤
│  6. 04_Build.md → 理解构建系统                              │
│  7. 05_Security.md → 安全机制分析                          │
└─────────────────────────────────────────────────────────────┘
```

## 主题快速跳转

### 按功能分类

**WebView 核心功能**
- [创建 WebView 实例](./02_N-API.md#webviewcontroller)
- [加载网页](./02_N-API.md#loadurl)
- [JavaScript 交互](./02_N-API.md#runjavascript)
- [Cookie 管理](./02_N-API.md#webcookemanager)

**高级功能**
- [下载管理](./02_N-API.md#webdownloadmanager)
- [广告拦截](./02_N-API.md#webadsblockmanager)
- [代理配置](./02_N-API.md#proxycontroller)
- [原生消息传递](./02_N-API.md#webnativemessagingextension)

**系统集成**
- [适配器层架构](./03_InnerAPI.md#ohos_adapter)
- [系统能力服务](./03_InnerAPI.md#sa)
- [权限管理](./05_Security.md#权限相关风险)

### 按代码模块分类

```
interfaces/kits/napi/     → 02_N-API.md
interfaces/kits/ani/       → 02_N-API.md (ANI 部分)
interfaces/native/        → 02_N-API.md (NDK 部分)
ohos_interface/           → 03_InnerAPI.md
ohos_nweb/               → 03_InnerAPI.md
ohos_adapter/            → 03_InnerAPI.md
sa/                      → 03_InnerAPI.md
BUILD.gn                 → 04_Build.md
```

## 常见问题索引

| 问题 | 答案位置 |
|------|----------|
| WebView 如何初始化？ | [02_N-API.md#初始化流程](./02_N-API.md#初始化流程) |
| 如何加载本地 HTML？ | [02_N-API.md#loadurl](./02_N-API.md#loadurl) |
| JavaScript 如何与 native 通信？ | [02_N-API.md#postmessage](./02_N-API.md#postmessage) |
| 构建产物有哪些？ | [04_Build.md#编译产物清单](./04_Build.md#编译产物清单) |
| 有哪些安全风险？ | [05_Security.md#风险清单](./05_Security.md#风险清单) |

## 代码证据索引

所有文档中的关键结论均可追溯到以下代码位置：

| 证据类型 | 代码位置 |
|----------|----------|
| N-API 注册 | `interfaces/kits/napi/common/napi_webview_native_module.cpp:90-93` |
| SA 定义 | `sa/web_native_messaging/common/web_native_messaging_common.h:42` |
| 权限校验 | `ohos_adapter/access_token_adapter/src/access_token_adapter_impl.cpp:29-34` |
| 构建配置 | `config.gni` / `BUILD.gn` |

---

[返回 README.md](./README.md)
