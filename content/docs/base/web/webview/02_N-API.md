# N-API 接口参考

> 本文档详细介绍 ArkWeb WebView 组件的 N-API（Node-API）接口，包括所有导出的 JS API、参数校验和错误处理。

## 模块概览

### N-API 模块列表

| 模块名 | 导出类/函数 | 路径 |
|--------|------------|------|
| `web.webview` | 26 个类 | `interfaces/kits/napi/common/napi_webview_native_module.cpp` |
| `web.netErrorList` | WebNetErrorList | `interfaces/kits/napi/web_net_error_code/napi_web_net_errorcode_module.cpp` |
| `web.WebNativeMessagingExtensionAbility` | 扩展 Ability | `interfaces/kits/napi/web_native_messaging_extension/ability/` |
| `web.WebNativeMessagingExtensionContext` | 扩展 Context | `interfaces/kits/napi/web_native_messaging_extension/context/` |

### 主模块 (web.webview)

**注册入口**: `interfaces/kits/napi/common/napi_webview_native_module.cpp:90-93`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_register_func = WebViewExport,
    .nm_modname = "web.webview",
};

extern "C" __attribute__((constructor)) void Register() {
    napi_module_register(&_module);
}
```

## WebviewController

**类名**: `WebviewController`
**C++ 实现**: `NapiWebviewController`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp`
**代码行数**: 347,820+ 行

### 构造函数

```typescript
constructor()
```

### 核心方法

| JS API | 参数 | 返回值 | 同步/异步 | 说明 |
|--------|------|--------|----------|------|
| `loadUrl` | `url: string`, `headers?: Record<string, string>[]` | `void` | 同步 | 加载 URL |
| `loadData` | `data: string`, `mimeType: string`, `encoding: string`, `baseUrl?: string`, `historyUrl?: string` | `void` | 同步 | 加载数据 |
| `loadWithDataAndBaseUrl` | 多个参数 | `void` | 同步 | 带 BaseURL 加载 |
| `backward` | - | `void` | 同步 | 后退 |
| `forward` | - | `void` | 同步 | 前进 |
| `backwardOrForward` | `step: number` | `void` | 同步 | 后退/前进指定步数 |
| `back` | - | `void` | 同步 | 后退（别名） |
| `forward_` | - | `void` | 同步 | 前进（别名） |
| `refresh` | - | `void` | 同步 | 刷新 |
| `stop` | - | `void` | 同步 | 停止加载 |
| `runJavaScript` | `script: string` | `Promise<string>` | 异步 | 执行 JS |
| `runJavaScriptExt` | `script: string` | `Promise<string>` | 异步 | 执行 JS（扩展） |
| `executeTypeScript` | `script: string` | `Promise<string>` | 异步 | 执行 TS |

### 事件方法

| JS API | 参数 | 说明 |
|--------|------|------|
| `on` | `event: string`, `callback: Function` | 绑定事件 |
| `off` | `event: string`, `callback?: Function` | 解绑事件 |
| `once` | `event: string`, `callback: Function` | 一次性事件 |

### 事件类型

| 事件名 | 回调参数 | 触发时机 |
|--------|----------|----------|
| `onPageBegin` | `WebPageBeginEvent` | 页面开始加载 |
| `onPageEnd` | `WebPageEndEvent` | 页面加载完成 |
| `onErrorReceive` | `WebErrorReceiveEvent` | 加载错误 |
| `onHttpErrorReceive` | `WebHttpErrorEvent` | HTTP 错误 |
| `onControllerAttached` | - | 控制器附加 |
| `onRenderProcessGone` | `RenderProcessGoneDetail` | 渲染进程终止 |
| `onRefreshAccessedHistory` | `RefreshHistoryEvent` | 历史记录刷新 |

### 消息通信

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `createWebMessagePorts` | - | `WebMessagePort[]` | 创建消息端口 |
| `postMessage` | `port: WebMessagePort`, `message: WebMessage` | `void` | 发送消息 |

### 查询方法

| JS API | 返回值 | 说明 |
|--------|--------|------|
| `getTitle` | `string` | 获取标题 |
| `getUrl` | `string` | 获取当前 URL |
| `getPageInfo` | `WebPageInfo` | 获取页面信息 |
| `canGoBack` | `boolean` | 能否后退 |
| `canGoForward` | `boolean` | 能否前进 |
| `getBackForwardEntries` | `WebHistoryList` | 获取历史列表 |
| `getProgress` | `number` | 获取加载进度 |
| `hasImage` | `Promise<boolean>` | 是否有图片 |
| `getCookieManager` | `WebCookieManager` | 获取 Cookie 管理器 |
| `getWebStorage` | `WebStorage` | 获取 Web Storage |
| `getDataBase` | `WebDataBase` | 获取数据库 |
| `getNativeMediaPlayer` | `NativeMediaPlayerHandler` | 获取原生播放器 |
| `getDownloadManager` | `WebDownloadManager` | 获取下载管理器 |

### 配置方法

| JS API | 参数 | 说明 |
|--------|------|------|
| `setWebViewNSAccessory` | `isAccessory: boolean` | 设置 NSA 配件模式 |
| `setBackgroundColor` | `color: number` | 设置背景色 |
| `setForegroundColor` | `color: number` | 设置前景色 |
| `zoom` | `factor: number` | 缩放 |
| `zoomIn` | - | 放大 |
| `zoomOut` | - | 缩小 |
| `getZoom` | `x: number`, `y: number` | 获取缩放比例 |
| `scroll` | `x: number`, `y: number` | 滚动 |
| `scrollBy` | `x: number`, `y: number` | 相对滚动 |
| `getScrollPosition` | `Promise<{x: number, y: number}>` | 获取滚动位置 |
| `putNetworkAvailable` | `available: boolean` | 设置网络状态 |
| `isActive` | - | 是否激活 |
| `hasFocus` | - | 是否有焦点 |
| `clearHistory` | - | 清除历史 |
| `clearSslCache` | - | 清除 SSL 缓存 |
| `clearClientAuthenticationCache` | - | 清除客户端认证缓存 |
| `deleteJavaScriptRegister` | `object: string` | 删除 JS 注册 |
| `convertProcToLocalUrl` | `procUrl: string` | 转换 URL |
| `getDefaultUserAgent` | - | 获取默认 UA |
| `setCustomUserAgent` | `userAgent: string` | 设置自定义 UA |
| `getUserAgent` | - | 获取 UA |
| `loadUrlWithParams` | `url: string`, `headers?: Record<string, string>[]` | 带参数加载 |

### PDF 方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `saveAsPdf` | `filePath: string`, `settings?: PdfSaveSettings` | `Promise<void>` | 保存为 PDF |
| `getPdf` | `settings?: PdfSaveSettings` | `Promise<Uint8Array>` | 获取 PDF 数据 |
| `createPrintDocument` | `jobName: string`, `settings?: PrintSettings` | `Promise<WebPrintDocument>` | 创建打印文档 |

### 离线方法

| JS API | 参数 | 说明 |
|--------|------|------|
| `prefetchResource` | `url: string`, `headers?: Record<string, string>[]` | 预取资源 |
| `warmupServiceWorker` | `url: string` | 预热 Service Worker |

### 调用链追踪

```
ArkTS (new WebviewController())
  │
  ▼
N-API (napi_webview_controller.cpp:684-866)
  │  DECLARE_NAPI_FUNCTION("loadUrl", LoadUrl)
  │  DECLARE_NAPI_FUNCTION("runJavaScript", RunJavaScript)
  │
  ▼
NapiWebviewController::LoadUrl()
  │
  ▼
WebviewController::LoadUrl(url, httpHeaders)
  │
  ▼
NWeb::LoadURL(url)
  │
  ▼ (IPC)
  │
  ▼
ArkWebCore.hap (Renderer Process)
  │
  ▼
Chromium (content::WebContents::LoadURL)
```

## WebCookieManager

**类名**: `WebCookieManager`
**C++ 实现**: `NapiWebCookieManager`
**源文件**: `interfaces/kits/napi/webcookiemanager/napi_web_cookie_manager.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getCookieManager` | - | `WebCookieManager` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `configCookie` | `url: string`, `cookie: string` | `Promise<void>` | 配置 Cookie |
| `fetchCookie` | `url: string` | `Promise<string>` | 获取 Cookie |
| `getCookie` | `url: string` | `Promise<string>` | 获取 Cookie |
| `setCookie` | `url: string`, `cookie: string` | `Promise<void>` | 设置 Cookie |
| `deleteExpiredCookies` | - | `Promise<void>` | 删除过期 Cookie |
| `deleteAllCookies` | - | `Promise<void>` | 删除所有 Cookie |
| `deleteSessionCookies` | - | `Promise<void>` | 删除会话 Cookie |
| `isCookieAllowed` | `url: string` | `Promise<boolean>` | 是否允许 Cookie |
| `putAcceptCookieEnabled` | `enabled: boolean` | `Promise<void>` | 设置接受 Cookie |
| `isAcceptCookieEnabled` | - | `Promise<boolean>` | 是否接受 Cookie |
| `putAcceptThirdPartyCookieEnabled` | `enabled: boolean` | `Promise<void>` | 设置接受第三方 Cookie |
| `isAcceptThirdPartyCookieEnabled` | - | `Promise<boolean>` | 是否接受第三方 Cookie |
| `putCookiePath` | `path: string` | `Promise<void>` | 设置 Cookie 路径 |
| `getCookiePath` | - | `Promise<string>` | 获取 Cookie 路径 |
| `putCookieExpiration` | `expiration: number` | `Promise<void>` | 设置过期时间 |
| `isCookieSameSiteEnabled` | - | `Promise<boolean>` | 是否启用 SameSite |
| `putCookieSameSiteConfig` | `url: string`, `config: SameSiteCookieConfig` | `Promise<void>` | 设置 SameSite 配置 |

## WebDataBase

**类名**: `WebDataBase`
**C++ 实现**: `NapiWebDataBase`
**源文件**: `interfaces/kits/napi/webdatabase/napi_web_data_base.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getDataBase` | - | `WebDataBase` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setHttpAuthUsernamePassword` | `host: string`, `realm: string`, `username: string`, `password: string` | `Promise<void>` | 设置 HTTP 认证 |
| `getHttpAuthUsernamePassword` | `host: string`, `realm: string` | `Promise<[string, string]>` | 获取 HTTP 认证 |
| `deleteHttpAuthUsernamePassword` | - | `Promise<void>` | 删除 HTTP 认证 |
| `existHttpAuthUsernamePassword` | - | `Promise<boolean>` | 是否存在认证 |
| `getAllHttpAuthCredentials` | - | `Promise<HttpAuthCredentials[]>` | 获取所有认证 |
| `clearHttpAuthUsernamePassword` | - | `Promise<void>` | 清除认证 |
| `existGeolocation` | `origin: string` | `Promise<boolean>` | 是否存在位置权限 |
| `setGeolocation` | `origin: string`, `allow: boolean`, `deny?: boolean` | `Promise<void>` | 设置位置权限 |
| `getGeolocation` | `origin: string` | `Promise<GeolocationPermission>` | 获取位置权限 |
| `deleteGeolocation` | `origin: string` | `Promise<void>` | 删除位置权限 |
| `clearAllGeolocation` | - | `Promise<void>` | 清除所有位置权限 |
| `existOriginByPermission` | `origin: string`, `permission: Permission` | `Promise<boolean>` | 是否存在权限 |
| `getOriginByPermission` | `origin: string`, `permission: Permission` | `Promise<number>` | 获取权限 |
| `setOriginByPermission` | `origin: string`, `permission: Permission`, `result: number` | `Promise<void>` | 设置权限 |
| `getAllOrigin` | `permission: Permission` | `Promise<string[]>` | 获取所有 Origin |
| `clearOriginByPermission` | `origin: string`, `permission: Permission` | `Promise<void>` | 清除权限 |
| `clearAllOriginByPermission` | `permission: Permission` | `Promise<void>` | 清除所有权限 |
| `getQuota` | `origin: string` | `Promise<number>` | 获取配额 |
| `setQuota` | `origin: string`, `quota: number` | `Promise<void>` | 设置配额 |
| `getStorageUsage` | `origin: string` | `Promise<number>` | 获取存储使用 |
| `deleteOrigin` | `origin: string` | `Promise<void>` | 删除 Origin |
| `deleteAllOrigin` | - | `Promise<void>` | 删除所有 Origin |
| `getOriginList` | - | `Promise<string[]>` | 获取 Origin 列表 |

## WebStorage

**类名**: `WebStorage`
**C++ 实现**: `NapiWebStorage`
**源文件**: `interfaces/kits/napi/webstorage/napi_web_storage.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getWebStorage` | - | `WebStorage` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setLocalStorage` | `origin: string`, `key: string`, `value: string` | `Promise<void>` | 设置 LocalStorage |
| `getLocalStorage` | `origin: string`, `key: string` | `Promise<string>` | 获取 LocalStorage |
| `deleteLocalStorage` | `origin: string`, `key: string` | `Promise<void>` | 删除 LocalStorage |
| `clearLocalStorage` | `origin: string` | `Promise<void>` | 清除 LocalStorage |
| `getLocalStorageOrigin` | `origin: string` | `Promise<string[]>` | 获取 Origin 键列表 |
| `length` | `origin: string` | `Promise<number>` | 获取长度 |
| `key` | `origin: string`, `index: number` | `Promise<string>` | 获取键 |
| `getSessionStorage` | `origin: string`, `key: string` | `Promise<string>` | 获取 SessionStorage |
| `setSessionStorage` | `origin: string`, `key: string`, `value: string` | `Promise<void>` | 设置 SessionStorage |
| `deleteSessionStorage` | `origin: string`, `key: string` | `Promise<void>` | 删除 SessionStorage |
| `clearSessionStorage` | `origin: string` | `Promise<void>` | 清除 SessionStorage |
| `clearAllSessionStorage` | - | `Promise<void>` | 清除所有 SessionStorage |
| `clearAllLocalStorage` | - | `Promise<void>` | 清除所有 LocalStorage |

## WebMessagePort

**类名**: `WebMessagePort`
**C++ 实现**: `NapiWebMessagePort`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp`

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `close` | - | `Promise<void>` | 关闭端口 |
| `postMessageEvent` | `event: WebMessageEvent` | `Promise<void>` | 发送消息 |
| `onMessageEvent` | `callback: (event: WebMessageEvent) => void` | `void` | 监听消息 |

## WebDownloadManager

**类名**: `WebDownloadManager`
**C++ 实现**: `NapiWebDownloadManager`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_web_download_manager.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getDownloadManager` | - | `WebDownloadManager` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `download` | `url: string`, `downloadParams: DownloadParams` | `WebDownloadItem` | 开始下载 |
| `setDownloadDelegate` | `delegate: WebDownloadDelegate` | `void` | 设置下载委托 |
| `removeDownloadListener` | `listenerId: number` | `void` | 移除下载监听 |
| `onDownloadStart` | `callback: (item: WebDownloadItem) => void` | `number` | 下载开始回调 |
| `onDownloadProgress` | `callback: (item: WebDownloadItem) => void` | `number` | 下载进度回调 |
| `onDownloadFail` | `callback: (item: WebDownloadItem) => void` | `number` | 下载失败回调 |
| `cancelDownload` | `downloadId: number` | `Promise<void>` | 取消下载 |

## WebAdsBlockManager

**类名**: `WebAdsBlockManager`
**C++ 实现**: `NapiWebAdsBlockManager`
**源文件**: `interfaces/kits/napi/webadsblockmanager/napi_web_adsblock_manager.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getAdsBlockManager` | - | `WebAdsBlockManager` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setAdsBlockRules` | `rules: string[]` | `Promise<void>` | 设置广告拦截规则 |
| `getAdsBlockRules` | - | `Promise<string[]>` | 获取广告拦截规则 |
| `setAdsBlockEnabled` | `enabled: boolean` | `Promise<void>` | 设置是否启用 |
| `isAdsBlockEnabled` | - | `Promise<boolean>` | 是否启用 |

## ProxyController

**类名**: `ProxyController`
**C++ 实现**: `NapiProxyController`
**源文件**: `interfaces/kits/napi/proxycontroller/napi_proxy_controller.cpp`

### 静态方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getProxyController` | - | `ProxyController` | 获取单例 |

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getProxy` | - | `Promise<ProxyInfo>` | 获取代理信息 |
| `setProxy` | `config: ProxyConfig` | `Promise<void>` | 设置代理 |

## WebAsyncController

**类名**: `WebAsyncController`
**C++ 实现**: `NapiWebAsyncController`
**源文件**: `interfaces/kits/napi/webasynccontroller/napi_web_async_controller.cpp`

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setWebCacheEnabled` | `enabled: boolean` | `void` | 设置 Web 缓存 |
| `setInitializeAhead` | `url: string`, `withHeaders: boolean` | `Promise<void>` | 预初始化 |

## WebSchemeHandler

**类名**: `WebSchemeHandler`
**C++ 实现**: `NapiWebSchemeHandler`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_web_scheme_handler_request.cpp`

### 实例方法

| JS API | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `registerScheme` | `scheme: string`, `handler: WebSchemeHandler` | `void` | 注册自定义协议 |
| `unRegisterScheme` | `scheme: string` | `void` | 注销自定义协议 |

## WebHistoryList

**类名**: `WebHistoryList`
**C++ 实现**: `NapiWebHistoryList`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp`

### 实例方法

| JS API | 返回值 | 说明 |
|--------|--------|------|
| `getSize` | `Promise<number>` | 获取历史条目数 |
| `getCurrentIndex` | `Promise<number>` | 获取当前索引 |
| `getItemAtIndex` | `Promise<WebHistoryItem>` | 获取指定条目 |
| `getCurrentItem` | `Promise<WebHistoryItem>` | 获取当前条目 |

## NativeMediaPlayerHandler

**类名**: `NativeMediaPlayerHandler`
**C++ 实现**: `NapiNativeMediaPlayerHandler`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_native_media_player.cpp`

### 实例方法

| JS API | 参数 | 说明 |
|--------|------|------|
| `onplay` | `callback: Function` | 播放回调 |
| `onpause` | `callback: Function` | 暂停回调 |
| `onseeking` | `callback: Function` | 跳转回调 |
| `onseeked` | `callback: Function` | 跳转完成回调 |
| `ontimeupdate` | `callback: Function` | 时间更新回调 |
| `onended` | `callback: Function` | 结束回调 |
| `onerror` | `callback: Function` | 错误回调 |
| `onloadedmetadata` | `callback: Function` | 元数据加载完成回调 |
| `onwaitingfordata` | `callback: Function` | 等待数据回调 |

## UserAgentMetadata

**类名**: `UserAgentMetadata`
**C++ 实现**: `NapiUserAgentMetadata`
**源文件**: `interfaces/kits/napi/webviewcontroller/napi_user_agent_metadata.cpp`

### 属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| `brand` | `string` | 品牌 |
| `version` | `string` | 版本 |
| `platform` | `string` | 平台 |
| `architecture` | `string` | 架构 |

## 参数校验

### 通用校验规则

| 参数类型 | 校验规则 | 错误码 |
|----------|----------|--------|
| `url` | 非空，最大长度 2048，合法 URL 格式 | 401 |
| `string` | 非空，最大长度限制 | 401 |
| `number` | 范围检查 | 401 |
| `array` | 长度限制 | 401 |
| `object` | 属性完整性 | 401 |

### 校验实现

**文件**: `interfaces/kits/napi/common/napi_parse_utils.cpp`

```cpp
// URL 校验
bool ParseString(napi_env env, napi_value value, std::string& result) {
    // 1. 检查类型
    napi_valuetype type;
    napi_typeof(env, value, &type);
    if (type != napi_string) {
        return false;
    }

    // 2. 获取字符串
    size_t length;
    napi_get_value_string_utf8(env, value, buffer, maxSize, &length);

    // 3. 校验长度
    if (length == 0 || length > URL_MAX_LENGTH) {
        return false;
    }

    // 4. 校验 URL 格式
    if (!IsValidUrl(result)) {
        return false;
    }

    return true;
}

// 抛出参数错误
void ThrowErrorByErrcode(napi_env env, int32_t errCode) {
    napi_throw(env, CreateBusinessError(env, errCode));
}
```

### 错误码定义

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 17100001 | 初始化错误 |
| 17100002 | 无效 URL |
| 17100003 | 内存不足 |
| 17100004 | 不支持的操作 |
| 17100005 | 渲染进程终止 |
| 17100006 | WebView 尚未附加 |

## 错误处理

### 错误码枚举

```cpp
enum ErrorCode {
    NO_ERROR = 0,
    PARAM_CHECK_ERROR = 401,
    UNKNOWN_ERROR = -1,
    NAPI_ERRNO_MAX = 1000000
};
```

### 异常抛出

```cpp
// 参数检查错误
if (url.empty() || url.length() > URL_MAXIMUM) {
    BusinessError::ThrowErrorByErrcode(env, PARAM_CHECK_ERROR);
    return nullptr;
}

// 自定义错误消息
napi_throw_error(env, nullptr, "Invalid URL format");

// 类型错误
napi_throw_type_error(env, nullptr, "Expected string parameter");
```

### Promise 错误处理

```typescript
try {
    controller.loadUrl(invalidUrl);
} catch (error) {
    console.error('Error code: ' + error.code);
    console.error('Error message: ' + error.message);
}

controller.runJavaScript(script)
    .then(result => console.log(result))
    .catch(error => console.error(error.code, error.message));
```

## 异步编程模型

### Promise 模式

```cpp
static napi_value RunJavaScript(napi_env env, napi_callback_info info)
{
    // 1. 创建 Promise
    napi_value promise;
    napi_deferred deferred;
    napi_create_promise(env, &deferred, &promise);

    // 2. 创建回调实现
    auto callback = std::make_shared<NWebValueCallbackImpl>(
        env, deferred, true);

    // 3. 调用核心引擎
    webviewController->RunJavaScript(script, callback);

    // 4. 返回 Promise
    return promise;
}
```

### Callback 模式

```cpp
static napi_value OnPageBegin(napi_env env, napi_callback_info info)
{
    // 1. 获取回调函数
    napi_value callback = argv[0];
    napi_ref callbackRef;
    napi_create_reference(env, callback, 1, &callbackRef);

    // 2. 设置事件监听
    webviewController->SetPageBeginHandler(
        [env, callbackRef](const std::string& url) {
            napi_value callback, jsUrl;
            napi_get_reference_value(env, callbackRef, &callback);
            napi_create_string_utf8(env, url.c_str(), url.length(), &jsUrl);
            napi_call_function(env, nullptr, callback, 1, &jsUrl, nullptr);
        }
    );

    return nullptr;
}
```

## 相关文档

| 主题 | 文档 |
|------|------|
| 架构设计 | [01_Architecture.md](./01_Architecture.md) |
| 内部模块 | [03_InnerAPI.md](./03_InnerAPI.md) |
| 构建系统 | [04_Build.md](./04_Build.md) |
| 安全机制 | [05_Security.md](./05_Security.md) |

---

[返回 SUMMARY.md](./SUMMARY.md) | [上一章: 架构设计](./01_Architecture.md) | [下一章: 内部接口](./03_InnerAPI.md)
