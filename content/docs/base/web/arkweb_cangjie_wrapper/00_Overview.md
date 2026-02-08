# 项目概览

## 项目定位

`arkweb_cangjie_wrapper` 是 OpenHarmony 系统中基于 ArkWeb 能力的 Cangjie API 封装层。它为 Cangjie 开发者提供了一套类型安全、符合 Cangjie 语言习惯的 Web 组件操作接口，屏蔽了底层 Native 实现的复杂性。

### 核心职责

- 提供 Web 组件控制能力（页面导航、生命周期、JavaScript 交互）
- 提供 Cookie 管理能力（增删改查、第三方 Cookie 控制）
- 提供历史记录管理能力（获取历史列表、访问历史项）
- 封装 C/C++ Native 层实现，通过 FFI 暴露统一接口

### 项目特性

| 特性 | 说明 |
|------|------|
| Beta 功能 | 当前处于 Beta 阶段，功能持续完善 |
| 标准设备专属 | 仅支持 OpenHarmony 标准设备 |
| 类型安全 | 基于 Cangjie 类型系统，编译期检查 |
| 异步支持 | 关键操作支持异步回调模式 |
| 异常处理 | 通过 BusinessException 统一错误处理 |

## 核心能力

本项目目前开放以下三类核心 API：

### 1. WebviewController — Web 组件控制器

提供对 Web 组件的完整控制能力，包括但不限于：

- **页面导航**：`loadUrl()`、`reload()`、`goBack()`、`goForward()`、`goBackOrForward()`
- **JavaScript 交互**：`runJavaScript()`、`registerJavaScriptProxy()`
- **滚动与缩放**：`scrollTo()`、`scrollBy()`、`pageUp()`、`pageDown()`、`zoom()`、`zoomIn()`、`zoomOut()`
- **状态获取**：`getUrl()`、`getTitle()`、`getUserAgent()`、`canGoBack()`
- **安全浏览**：`enableSafeBrowsing()`、`isSafeBrowsingEnabled()`、`getSecurityLevel()`
- **历史管理**：`getBackForwardEntries()`、`clearHistory()`
- **缓存管理**：`removeCache()`
- **归档存储**：`storeWebArchive()`

### 2. WebCookieManager — Cookie 管理者

提供对 Web Cookie 的完整管理能力：

- **Cookie 获取**：`fetchCookie()` — 支持普通模式和隐私模式
- **Cookie 设置**：`configCookie()` — 支持 Set-Cookie 格式
- **Cookie 控制**：`isCookieAllowed()`、`setAcceptCookiesEnabled()`
- **第三方 Cookie**：`isThirdPartyCookieAllowed()`、`setAcceptThirdPartyCookieEnabled()`
- **Cookie 查询**：`hasCookie()` — 检查是否存在 Cookie
- **Cookie 清除**：`clearAllCookies()`、`clearSessionCookie()`

### 3. BackForwardList — 历史记录列表

提供对浏览器前进后退历史的管理能力：

- **属性访问**：`currentIndex` — 当前索引位置
- **属性访问**：`size` — 历史记录数量（最大 50 条）
- **方法调用**：`getItemAtIndex(index)` — 获取指定索引的历史项

## 运行环境

### 系统要求

| 要求 | 详情 |
|------|------|
| 操作系统 | OpenHarmony 标准设备 |
| API Level | 22+ |
| 系统能力 | SystemCapability.Web.Webview.Core |

### 外部依赖

| 依赖组件 | 用途 |
|----------|------|
| `webview` | 提供 Native WebView 实现和 FFI 接口 |
| `cangjie_ark_interop` | 提供 FFI 基础类型、异常类、API Level 注解 |
| `arkui_cangjie_wrapper` | 提供 UI 组件基础类型、资源字符串转换 |
| `multimedia_cangjie_wrapper` | 提供 PixelMap 支持（历史项图标） |
| `hiviewdfx_cangjie_wrapper` | 提供 HiLog 日志能力 |

### 资源占用

| 资源 | 大小 |
|------|------|
| ROM | 约 300KB |
| RAM | 约 216KB |

## 版本信息

| 项目 | 值 |
|------|-----|
| 项目名称 | arkweb_cangjie_wrapper |
| 当前版本 | 6.1 |
| 许可证 | Apache License 2.0 |
| 发布类型 | code-segment |

## 暂未开放功能

以下功能正在开发中，尚未对开发者开放：

- AdsBlockManager — 广告拦截配置
- BackForwardCacheOptions — 前进后退缓存配置
- GeolocationPermissions — 地理位置权限
- JsMessageExt — JavaScript 消息扩展
- MediaSourceInfo — 媒体源信息
- NativeMediaPlayerSurfaceInfo — 原生媒体播放器
- PdfData — PDF 数据生成
- ProxyConfig / ProxyController — 网络代理
- WebDataBase — Web 数据库管理
- WebDownload* — 下载管理相关
- WebHttpBodyStream — HTTP 请求体
- WebMessageExt / WebMessagePort — Web 消息传递
- WebResourceHandler — 资源加载控制
- WebSchemeHandler* — 自定义协议处理器
- WebStorageOrigin — Web 存储
- NativeMediaPlayerBridge/Handler — 原生媒体桥接

## 快速入门示例

### 创建 WebviewController

```cangjie
// 导入模块
import ohos.web.webview.WebviewController

// 创建控制器（可选指定 webTag）
let controller = WebviewController()
```

### 加载网页

```cangjie
// 加载 URL
controller.loadUrl("https://www.example.com")

// 带请求头加载
let headers = Array<WebHeader>()
headers.append(WebHeader("Custom-Header", "Value"))
controller.loadUrl("https://www.example.com", headers: headers)
```

### Cookie 操作

```cangjie
import ohos.web.webview.WebCookieManager

// 获取 Cookie
let cookie = WebCookieManager.fetchCookie("https://www.example.com")

// 设置 Cookie
WebCookieManager.configCookie("https://www.example.com", "key=value")

// 清除所有 Cookie
WebCookieManager.clearAllCookies()
```

### 历史记录

```cangjie
// 获取历史列表
let history = controller.getBackForwardEntries()

// 遍历历史项
for (i in 0..history.size) {
    let item = history.getItemAtIndex(i)
    print("URL: ${item.historyUrl}, Title: ${item.title}")
}
```

## 相关资源

| 资源 | 链接 |
|------|------|
| API 参考 | https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/ArkWeb/cj-apis-webview.md |
| 开发指南 | https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/web/cj-web-component-overview.md |
| 项目仓库 | https://gitee.com/openharmony/web_webview |
