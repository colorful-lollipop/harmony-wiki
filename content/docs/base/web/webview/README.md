# ArkWeb WebView 组件 Wiki

> 本 Wiki 旨在帮助开发者快速理解 OpenHarmony WebView（ArkWeb）组件的架构设计、API 接口、构建系统和安全机制。

## 概述

**ArkWeb** 是 OpenHarmony WebView 组件的 Native 引擎，基于 Chromium 和 CEF（Chromium Embedded Framework）构建，提供完整的 Web 渲染和 JavaScript 执行能力。

- **代码仓库**: `//base/web/webview`
- **子系统**: `web`
- **组件名**: `webview`
- **系统能力**: `SystemCapability.Web.Webview.Core`

## 文档覆盖范围

| 文档 | 覆盖内容 |
|------|----------|
| [README.md](./README.md) | 本文档，Wiki 使用指南 |
| [SUMMARY.md](./SUMMARY.md) | 全站导航与阅读路线 |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境 |
| [01_Architecture.md](./01_Architecture.md) | 组件图、数据流、线程模型、时序图 |
| [02_N-API.md](./02_N-API.md) | N-API 接口清单、JS API、参数校验 |
| [03_InnerAPI.md](./03_InnerAPI.md) | 内部模块接口、依赖方向、生命周期 |
| [04_Build.md](./04_Build.md) | GN targets、编译产物、安装路径 |
| [05_Security.md](./05_Security.md) | 攻击面、信任边界、风险分析 |

## 快速开始

### 新人阅读路线

1. **先读**: [00_Overview.md](./00_Overview.md) - 理解项目定位
2. **次读**: [01_Architecture.md](./01_Architecture.md) - 掌握整体架构
3. **深入**: [02_N-API.md](./02_N-API.md) - 了解 API 使用
4. **构建**: [04_Build.md](./04_Build.md) - 理解编译系统

### 代码导航

```
webview/
├── interfaces/        # 对外接口层 (N-API/ANI/NDK)
├── ohos_interface/    # 内部接口定义
├── ohos_nweb/        # 核心实现
├── ohos_adapter/     # 系统适配层 (38+ 适配器)
├── sa/               # 系统能力服务
└── arkweb_utils/     # 工具库
```

## 版本信息

- **当前版本**: 3.1
- **ROM 占用**: ~85MB
- **RAM 占用**: ~150MB
- **许可证**: Apache License 2.0

## 相关仓库

| 仓库 | 用途 |
|------|------|
| [ace_ace_engine](https://gitee.com/openharmony/arkui_ace_engine) | ArkUI 框架 |
| [third_party_cef](https://gitee.com/openharmony/third_party_cef) | CEF 依赖 |
| [third_party_chromium](https://gitee.com/openharmony/third_party_chromium) | Chromium 源码 |

## 如何更新本文档

本文档基于代码自动生成。如需更新：

1. 修改代码后，运行文档生成脚本
2. 或手动更新对应章节，添加代码证据（路径+行号）
3. 确保 `SUMMARY.md` 链接正确

## 注意事项

- ❌ 禁止引用测试代码作为业务证据
- ✅ 所有关键结论必须有代码证据支撑
- ✅ N-API 文档必须包含完整 API 清单表
- ✅ 安全文档必须包含攻击面和风险分析

---

最后更新: 2026-02-06
