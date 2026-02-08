# ArkWeb WebView 对外接口文档

## 1. 接口总览

ArkWeb WebView 提供三类对外接口：

| 接口类型 | 适用语言 | 使用场景 | 性能 |
|---------|---------|---------|------|
| **N-API** | ArkTS/JS | UI 应用开发 | 有跨语言开销 |
| **NDK** | C/C++ | 高性能场景、游戏引擎 | 零拷贝 |
| **IPC** | 跨进程 | 系统服务调用 | Binder IPC |

---

## 2. N-API 接口

### 2.1 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `web.webview` |
| **导入方式** | `import webview from '@ohos.web.webview'` |
| **注册文件** | `interfaces/kits/napi/common/napi_webview_native_module.cpp:90-93` |

### 2.2 核心类清单

| 类名 | 职责 | 文件位置 |
|------|------|---------|
| `WebviewController` | WebView 核心控制器 | `webviewcontroller/napi_webview_controller.cpp` |
| `WebMessagePort` | 消息通道端口 | `webviewcontroller/napi_webview_controller.cpp` |
| `WebHistoryList` | 历史记录列表 | `webviewcontroller/napi_webview_controller.cpp` |
| `WebCookieManager` | Cookie 管理 | `webcookiemanager/napi_web_cookie_manager.cpp` |
| `WebStorage` | WebStorage 管理 | `webstorage/napi_web_storage.cpp` |
| `WebDataBase` | 数据库管理 | `webdatabase/napi_web_data_base.cpp` |
| `GeolocationPermissions` | 地理位置权限 | `webdatabase/napi_geolocation_permission.cpp` |
| `ProxyController` | 代理控制 | `proxycontroller/napi_proxy_controller.cpp` |
| `ProxyConfig` | 代理配置 | `proxycontroller/napi_proxy_config.cpp` |
| `ProxyRule` | 代理规则 | `proxycontroller/napi_proxy_rule.cpp` |
| `WebAdsBlockManager` | 广告拦截 | `webadsblockmanager/napi_web_adsblock_manager.cpp` |
| `WebAsyncController` | 异步控制 | `webasynccontroller/napi_web_async_controller.cpp` |
| `WebDownloadManager` | 下载管理 | `webviewcontroller/napi_web_download_manager.cpp` |
| `WebDownloadItem` | 下载项 | `webviewcontroller/napi_web_download_item.cpp` |
| `WebDownloadDelegate` | 下载委托 | `webviewcontroller/napi_web_download_delegate.cpp` |
| `WebSchemeHandler` | Scheme 处理器 | `webviewcontroller/napi_web_scheme_handler_request.cpp` |
| `WebSchemeHandlerRequest` | Scheme 请求 | `webviewcontroller/napi_web_scheme_handler_request.cpp` |
| `WebSchemeHandlerResponse` | Scheme 响应 | `webviewcontroller/napi_web_scheme_handler_request.cpp` |
| `NativeMediaPlayerHandler` | 原生媒体播放器 | `webviewcontroller/napi_native_media_player.cpp` |
| `UserAgentMetadata` | UA 元数据 | `webviewcontroller/napi_user_agent_metadata.cpp` |

### 2.3 WebviewController API 清单

#### 页面导航

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `loadUrl(url: string, headers?: Array<WebHeader>)` | URL, HTTP 头 | `void` | 同步 | `napi_webview_controller.cpp:LoadUrl()` |
| `loadData(data: string, mimeType: string, encoding: string, baseUrl?: string, historyUrl?: string)` | 数据, MIME 类型, 编码 | `void` | 同步 | `napi_webview_controller.cpp:LoadData()` |
| `backward()` | - | `void` | 同步 | `napi_webview_controller.cpp:Backward()` |
| `forward()` | - | `void` | 同步 | `napi_webview_controller.cpp:Forward()` |
| `refresh()` | - | `void` | 同步 | `napi_webview_controller.cpp:Refresh()` |
| `stop()` | - | `void` | 同步 | `napi_webview_controller.cpp:Stop()` |
| `getUrl()` | - | `string` | 同步 | `napi_webview_controller.cpp:GetUrl()` |
| `getTitle()` | - | `string` | 同步 | `napi_webview_controller.cpp:GetTitle()` |

#### JavaScript 交互

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `runJavaScript(script: string, callback?: Function)` | JS 代码 | `Promise<string>` | 异步 | `napi_webview_controller.cpp:RunJavaScript()` |
| `registerJavaScriptProxy(obj: object, name: string, syncMethods: Array<string>, asyncMethods?: Array<string>)` | 对象, 名称, 方法列表 | `void` | 同步 | `napi_webview_controller.cpp:RegisterJavaScriptProxy()` |
| `deleteJavaScriptRegister(name: string)` | 注册名 | `void` | 同步 | `napi_webview_controller.cpp:DeleteJavaScriptRegister()` |

#### 消息通道

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `createWebMessagePorts()` | - | `[WebMessagePort, WebMessagePort]` | 同步 | `napi_webview_controller.cpp:CreateWebMessagePorts()` |
| `postMessage(message: WebMessage, uri: string, ports?: Array<WebMessagePort>)` | 消息, URI, 端口 | `void` | 同步 | `napi_webview_controller.cpp:PostMessage()` |

#### 缩放控制

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `zoom(factor: number)` | 缩放因子 | `void` | 同步 | `napi_webview_controller.cpp:Zoom()` |
| `zoomIn()` | - | `void` | 同步 | `napi_webview_controller.cpp:ZoomIn()` |
| `zoomOut()` | - | `void` | 同步 | `napi_webview_controller.cpp:ZoomOut()` |
| `getZoomAccess()` | - | `boolean` | 同步 | `napi_webview_controller.cpp:GetZoomAccess()` |

#### 滚动控制

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `scrollTo(x: number, y: number)` | X, Y 坐标 | `void` | 同步 | `napi_webview_controller.cpp:ScrollTo()` |
| `scrollBy(deltaX: number, deltaY: number)` | 增量 X, Y | `void` | 同步 | `napi_webview_controller.cpp:ScrollBy()` |
| `slideScroll(vx: number, vy: number)` | 速度 X, Y | `void` | 同步 | `napi_webview_controller.cpp:SlideScroll()` |

#### 其他控制

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `setCustomUserAgent(userAgent: string)` | UserAgent 字符串 | `void` | 同步 | `napi_webview_controller.cpp:SetCustomUserAgent()` |
| `getCustomUserAgent()` | - | `string` | 同步 | `napi_webview_controller.cpp:GetCustomUserAgent()` |
| `setDownloadDelegate(delegate: WebDownloadDelegate)` | 下载委托 | `void` | 同步 | `napi_webview_controller.cpp:SetDownloadDelegate()` |
| `setUrlTrustList(trustList: string)` | 信任列表 JSON | `void` | 同步 | `napi_webview_controller.cpp:SetUrlTrustList()` |
| `createWebPrintDocumentAdapter(documentName: string)` | 文档名 | `WebPrintDocument` | 同步 | `napi_webview_controller.cpp:CreateWebPrintDocumentAdapter()` |
| `prefetchPage(url: string, additionalHeaders?: Map<string, string>)` | URL, 额外头 | `void` | 同步 | `napi_webview_controller.cpp:PrefetchPage()` |
| `getBackForwardEntries()` | - | `WebHistoryList` | 同步 | `napi_webview_controller.cpp:GetBackForwardEntries()` |

### 2.4 WebCookieManager API 清单

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `fetchCookie(url: string)` | URL | `Promise<string>` | 异步 | `napi_web_cookie_manager.cpp:FetchCookie()` |
| `configCookie(url: string, value: string)` | URL, Cookie 值 | `Promise<void>` | 异步 | `napi_web_cookie_manager.cpp:ConfigCookie()` |
| `clearAllCookies()` | - | `Promise<void>` | 异步 | `napi_web_cookie_manager.cpp:ClearAllCookies()` |
| `existCookie()` | - | `boolean` | 同步 | `napi_web_cookie_manager.cpp:ExistCookie()` |

### 2.5 ProxyController API 清单

| 方法 | 参数 | 返回 | 同步/异步 | 代码位置 |
|------|------|------|----------|---------|
| `setProxyOverride(rules: Array<ProxyRule>, bypassRules?: Array<string>)` | 代理规则, 绕过规则 | `Promise<void>` | 异步 | `napi_proxy_controller.cpp:SetProxyOverride()` |
| `clearProxyOverride()` | - | `Promise<void>` | 异步 | `napi_proxy_controller.cpp:ClearProxyOverride()` |
| `getProxyRules()` | - | `Array<ProxyRule>` | 同步 | `napi_proxy_controller.cpp:GetProxyRules()` |

---

## 3. NDK 接口

### 3.1 入口函数

```c
// arkweb_interface.h
void* OH_ArkWeb_GetNativeAPI(ArkWeb_NativeAPIVariantKind kind);
```

### 3.2 API 类型枚举

```c
// arkweb_type.h
typedef enum {
    ARKWEB_NATIVE_COMPONENT = 0,
    ARKWEB_NATIVE_CONTROLLER,
    ARKWEB_NATIVE_WEB_MESSAGE_PORT,
    ARKWEB_NATIVE_WEB_MESSAGE,
    ARKWEB_NATIVE_COOKIE_MANAGER,
    ARKWEB_NATIVE_JAVASCRIPT_VALUE
} ArkWeb_NativeAPIVariantKind;
```

### 3.3 Controller API

| 函数 | 参数 | 返回 | 代码位置 |
|------|------|------|---------|
| `OH_NativeArkWeb_RunJavaScript()` | webTag, script, callback | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_RegisterJavaScriptProxy()` | webTag, objName, methods, asyncMethods, object | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_UnregisterJavaScriptProxy()` | webTag | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_LoadData()` | webTag, data, mimeType, encoding, baseUrl, historyUrl | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_LoadUrl()` | webTag, url | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_Refresh()` | webTag | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_Stop()` | webTag | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_GoBack()` | webTag | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_GoForward()` | webTag | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_PostMessageEvent()` | webTag, message | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_SetJavaScriptProxyValidCallback()` | webTag, callback | `int` | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_SetDestroyCallback()` | webTag, callback | `int` | `native_interface_arkweb.cpp` |

### 3.4 CookieManager API

| 函数 | 参数 | 返回 | 代码位置 |
|------|------|------|---------|
| `OH_ArkWebCookieManager_SaveCookieSync()` | - | `bool` | `native_interface_arkweb.cpp` |
| `OH_ArkWebCookieManager_SaveCookieAsync()` | callback | `void` | `native_interface_arkweb.cpp` |
| `OH_ArkWebCookieManager_FetchCookieSync()` | url | `char*` | `native_interface_arkweb.cpp` |
| `OH_ArkWebCookieManager_FetchCookieAsync()` | url, callback | `void` | `native_interface_arkweb.cpp` |
| `OH_ArkWebCookieManager_ConfigCookieSync()` | url, value | `bool` | `native_interface_arkweb.cpp` |
| `OH_ArkWebCookieManager_ClearAllCookiesSync()` | - | `bool` | `native_interface_arkweb.cpp` |

---

## 4. IPC 接口

### 4.1 SA 8610 - WebNativeMessagingService

**服务信息**:
- **SA ID**: 8610
- **进程名**: `web_native_messaging_service`
- **库文件**: `libweb_native_messaging_service.z.so`

**IDL 接口定义** (`sa/web_native_messaging/IWebNativeMessagingService.idl`):

```idl
interface OHOS.NWeb.IWebNativeMessagingService {
    void ConnectWebNativeMessagingExtension(
        [in] IRemoteObject token,
        [in] Want want,
        [in] IRemoteObject connectionCallback,
        [in] int connectionId,
        [out] int errorNum
    );

    void DisconnectWebNativeMessagingExtension(
        [in] int connectionId,
        [out] int errorNum
    );

    void StartAbility(
        [in] IRemoteObject token,
        [in] Want want,
        [in] StartOptions options,
        [out] int errorNum
    );

    void StopNativeConnectionFromExtension(
        [in] int connectionId,
        [out] int errorNum
    );
};
```

**错误码**:

| 错误码 | 值 | 说明 |
|--------|-----|------|
| SUCCESS | 0 | 成功 |
| PERMISSION_CHECK_ERROR | -1 | 权限检查失败 |
| CONTEXT_ERROR | -2 | 上下文错误 |
| WANT_FORMAT_ERROR | -3 | Want 格式错误 |
| CONNECTION_NOT_EXIST | -4 | 连接不存在 |
| MEMORY_ERROR | -5 | 内存错误 |
| CALLBACK_ERROR | -6 | 回调错误 |
| IPC_ERROR | -7 | IPC 错误 |
| SERVICE_INIT_ERROR | -8 | 服务初始化错误 |
| CONNECTION_ID_EXIST | -9 | 连接 ID 已存在 |
| REQUEST_SIZE_TOO_LARGE | -10 | 请求过大 |
| CONNECT_STATUS_ERROR | -11 | 连接状态错误 |
| ABILITY_CONNECTION_ERROR | -12 | Ability 连接错误 |
| SERVICE_DIED_ERROR | -13 | 服务死亡 |

### 4.2 SA 8350 - AppFwkUpdateService

**服务信息**:
- **SA ID**: 8350
- **进程名**: `app_fwk_update_service`
- **库文件**: `libapp_fwk_update_service.z.so`

**IDL 接口定义** (`sa/app_fwk_update/IAppFwkUpdateService.idl`):

```idl
interface OHOS.NWeb.IAppFwkUpdateService {
    void VerifyPackageInstall(
        [in] String bundleName,
        [in] String hapPath,
        [out] int success
    );

    [oneway] void NotifyFWKAfterBmsStart();

    void NotifyArkWebInstallSuccess();
};
```

**访问控制**:
- `VerifyPackageInstall()` 仅限 UID 5523 (FOUNDATION_UID) 调用

---

## 5. 使用示例

### 5.1 N-API 使用示例

```typescript
// 创建 WebView 并加载页面
import webview from '@ohos.web.webview';

@Entry
@Component
struct WebPage {
  controller: webview.WebviewController = new webview.WebviewController();

  build() {
    Column() {
      Web({ src: 'https://www.example.com', controller: this.controller })
        .onPageBegin((event) => {
          console.log('Page start: ' + event.url);
        })
        .onPageEnd((event) => {
          console.log('Page end: ' + event.url);
        });

      Button('Run JS')
        .onClick(() => {
          this.controller.runJavaScript('document.title')
            .then((title) => {
              console.log('Title: ' + title);
            });
        });
    }
  }
}
```

### 5.2 NDK 使用示例

```cpp
#include <arkweb_interface.h>
#include <native_interface_arkweb.h>

// 获取 Controller API
NativeControllerAPI* api = (NativeControllerAPI*)OH_ArkWeb_GetNativeAPI(
    ARKWEB_NATIVE_CONTROLLER
);

// 运行 JavaScript
const char* script = "console.log('Hello from NDK')";
api->runJavaScript(webTag, script, callback);
```

---

*文档版本: 1.0*  
*更新日期: 2026-02-07*
