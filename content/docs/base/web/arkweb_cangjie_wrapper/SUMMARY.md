# 文档导航

本文档是 `arkweb_cangjie_wrapper` 项目的完整工程 Wiki，提供了项目的全面技术参考。

## 快速开始

- [README](/wiki/README.md) — 文档说明和使用指南
- [00_Overview](/wiki/00_Overview.md) — 项目概览

## 核心文档

### 架构与结构

| 章节 | 内容 |
|------|------|
| [01_Directory_Structure](/wiki/01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture](/wiki/02_Architecture.md) | 架构设计、组件关系、FFI 调用模式 |
| [03_N_API_Reference](/wiki/03_N_API_Reference.md) | 完整 API 接口文档 |

### 构建与配置

| 章节 | 内容 |
|------|------|
| [05_GN_Build](/wiki/05_GN_Build.md) | GN 编译目标、依赖关系 |
| [06_Build_Artifacts](/wiki/06_Build_Artifacts.md) | 编译产物、安装路径 |

### 安全与问题

| 章节 | 内容 |
|------|------|
| [07_Security_Review](/wiki/07_Security_Review.md) | 安全风险评审与修复建议 |
| [08_Troubleshooting](/wiki/08_Troubleshooting.md) | 常见问题与定位路径 |

## 附录

| 文档 | 内容 |
|------|------|
| [appendix/Callgraphs](/wiki/appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Error_Codes](/wiki/appendix/Error_Codes.md) | 错误码完整列表 |

## 新人阅读顺序

建议按照以下顺序阅读以逐步深入：

1. `README.md` — 了解文档结构
2. `00_Overview.md` — 项目定位
3. `01_Directory_Structure.md` — 目录结构
4. `02_Architecture.md` — 架构设计
5. `03_N_API_Reference.md` — API 接口
6. `05_GN_Build.md` — 构建配置
7. `07_Security_Review.md` — 安全注意

## API 速查

### WebviewController

| 方法 | 功能 |
|------|------|
| `loadUrl()` | 加载 URL |
| `goBack()` / `goForward()` | 前进后退 |
| `runJavaScript()` | 执行 JavaScript |
| `registerJavaScriptProxy()` | 注册 JS 代理 |
| `getBackForwardEntries()` | 获取历史列表 |
| [完整列表 →](/wiki/03_N_API_Reference.md#webviewcontroller)

### WebCookieManager

| 方法 | 功能 |
|------|------|
| `fetchCookie()` | 获取 Cookie |
| `configCookie()` | 设置 Cookie |
| `clearAllCookies()` | 清除所有 Cookie |
| [完整列表 →](/wiki/03_N_API_Reference.md#webcookiemanager)

### BackForwardList

| 方法 | 功能 |
|------|------|
| `currentIndex` | 当前索引 |
| `size` | 列表大小 |
| `getItemAtIndex()` | 获取历史项 |
| [完整列表 →](/wiki/03_N_API_Reference.md#backforwardlist)

## 版本兼容性

| 项目 | 要求 |
|------|------|
| OpenHarmony | 标准设备 |
| API Level | 22+ |
| 依赖组件 | webview、cangjie_ark_interop、arkui_cangjie_wrapper 等 |

## 文档变更日志

| 日期 | 变更 |
|------|------|
| 2025-02-06 | 初始版本，完整文档生成 |
