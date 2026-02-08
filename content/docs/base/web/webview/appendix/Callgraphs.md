# 关键调用链

> 本文档记录 ArkWeb WebView 组件的关键调用链，从入口到核心逻辑的完整追踪。

## 入口点索引

### 1. WebView 创建调用链

```
JS/ArkTS: new webview.WebviewController()
    │
    ▼
N-API: napi_webview_controller.cpp
    │
    ▼ (napi_define_class → napi_wrap)
    │
    ▼
NapiWebviewController::JsConstructor()
    │
    ▼
WebviewController::WebviewController()
    │
    ▼
NWebHelper::Instance().CreateNWeb(createInfo)
    │
    ▼ (IPC)
    │
    ▼
libnweb.so: NWebHelper::CreateNWeb()
    │
    ▼
ArkWebCore.hap: Chromium WebContents::Create()
```

### 2. 页面加载调用链

```
JS/ArkTS: controller.loadUrl(url)
    │
    ▼
N-API: napi_webview_controller.cpp:LoadUrl()
    │
    ▼
NapiWebviewController::LoadUrl(env, info)
    │
    ▼
WebviewController::LoadUrl(url, httpHeaders)
    │
    ▼ (IPC)
    │
    ▼
ArkWebCore.hap:
    ├── content::WebContents::LoadURL(url)
    ├── net::URLRequest::Create()
    ├── net::HttpTransactionFactory::Create()
    └── blink::DocumentLoader::StartLoading()
```

### 3. JavaScript 执行调用链

```
JS/ArkTS: controller.runJavaScript(script)
    │
    ▼
N-API: napi_webview_controller.cpp:RunJavaScript()
    │
    ▼
NapiWebviewController::RunJavaScript(env, info)
    │
    ▼
WebviewController::RunJavaScript(script, callback)
    │
    ▼ (IPC)
    │
    ▼
ArkWebCore.hap:
    ├── content::RenderFrame::ExecuteJavaScript()
    ├── blink::LocalFrame::ExecuteJavaScript()
    ├── v8::Context::Enter()
    ├── v8::Script::Run()
    └── v8::Context::Exit()
```

### 4. 消息通信调用链

```
JS (Web): port.postMessage(message)
    │
    ▼
JS (Native): controller.postMessage(port, message)
    │
    ▼
N-API: napi_webview_controller.cpp:PostMessage()
    │
    ▼
NapiWebviewController::PostMessage(env, info)
    │
    ▼
WebviewController::PostMessage(port, message)
    │
    ▼ (IPC)
    │
    ▼
ArkWebCore.hap:
    ├── blink::WebMessagePort::PostMessage()
    ├── blink::LocalFrame::PostMessage()
    └── content::RenderFrame::ProcessWebMessage()
```

## 核心调用链详情

### WebView 创建完整调用链

```cpp
// 1. ArkTS 入口
// 文件: interfaces/kits/ani/webview/src/napi_webview_controller_ani.cpp
let controller = new webview.WebviewController();

// 2. N-API 构造函数
// 文件: interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp:348
static napi_value JsConstructor(napi_env env, napi_callback_info info) {
    // 创建 C++ 对象
    WebviewController* controller = new WebviewController();

    // 包装到 JS 对象
    napi_wrap(env, thisVar, controller,
        [](napi_env env, void* data, void* hint) {
            delete static_cast<WebviewController*>(data);
        },
        nullptr, nullptr);

    return thisVar;
}

// 3. Controller 初始化
// 文件: interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp
class WebviewController {
public:
    WebviewController() : nweb_(nullptr) {}

    void Init(int32_t instanceId) {
        instanceId_ = instanceId;
    }
};

// 4. 创建 NWeb 实例
// 文件: ohos_nweb/src/nweb_helper.cpp
std::shared_ptr<NWeb> NWebHelper::CreateNWeb(
    std::shared_ptr<NWebCreateInfo> create_info) {
    if (!IsInited()) {
        Init();
    }

    // 创建设置
    auto preference = std::make_shared<NWebPreferenceImpl>();
    preference->SetJavaScriptEnabled(true);

    // 调用引擎创建
    auto nweb = LoadAndCreateNWeb(create_info);
    if (nweb) {
        nweb->SetNWebCallback(create_info->nweb_delegate);
    }

    return nweb;
}

// 5. 引擎创建
// 文件: ohos_nweb/src/nweb_helper.cpp
std::shared_ptr<NWeb> NWebHelper::LoadAndCreateNWeb(
    std::shared_ptr<NWebCreateInfo> create_info) {
    // 加载动态库
    void* handle = dlopen("libarkweb_engine.so", RTLD_NOW);
    if (!handle) {
        WVLOG_E("Failed to load arkweb engine");
        return nullptr;
    }

    // 获取创建函数
    CreateNWebFunc createFunc =
        reinterpret_cast<CreateNWebFunc>(dlsym(handle, "CreateNWeb"));
    if (!createFunc) {
        WVLOG_E("Failed to find CreateNWeb symbol");
        return nullptr;
    }

    // 调用引擎创建
    return createFunc(create_info.get());
}
```

### 页面加载完整调用链

```cpp
// 1. N-API 入口
// 文件: interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp:684
static napi_value LoadUrl(napi_env env, napi_callback_info info) {
    // 获取参数
    napi_value thisVar = nullptr;
    napi_get_cb_info(env, info, &argc, argv, &thisVar, nullptr);

    // 解包 C++ 对象
    WebviewController* controller = nullptr;
    napi_unwrap(env, thisVar, (void**)&controller);

    // 解析 URL
    std::string url;
    NapiParseUtils::ParseString(env, argv[0], url);

    // 调用 Controller
    ErrCode errCode = controller->LoadUrl(url, httpHeaders);

    return nullptr;
}

// 2. Controller 处理
// 文件: ohos_nweb/src/nweb.cpp (假设)
int32_t WebviewController::LoadUrl(const std::string& url,
                                     const std::map<std::string, std::string>& headers) {
    if (!nweb_) {
        return ERROR_WEBVIEW_NOT_ATTACHED;
    }

    // 转发到 NWeb
    return nweb_->LoadURLWithHeaders(url, headers);
}

// 3. NWeb 核心
// 文件: ohos_interface/include/ohos_nweb/nweb.h
virtual int32_t LoadURLWithHeaders(
    const std::string& url,
    const std::map<std::string, std::string>& additionalHeaders) = 0;

// 4. Chromium 实现
// 文件: ArkWebCore.hap (Chromium 源码)
namespace content {

WebContents::LoadURLWithHeaders(const GURL& url,
                                 const net::HttpRequestHeaders& headers) {
    // 1. 验证 URL
    if (!url.is_valid()) {
        return net::ERR_INVALID_URL;
    }

    // 2. 创建 NavigationController
    NavigationController& controller = GetController();

    // 3. 加载 URL
    controller.LoadURL(url, Referrer(), headers, ui::PAGE_TRANSITION_TYPED);

    // 4. 通知开始导航
    NotifyNavigationStateChanged(INVALIDATE_TYPE_URL);
}

}  // namespace content
```

### JavaScript 执行完整调用链

```cpp
// 1. N-API 入口
// 文件: interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp:726
static napi_value RunJavaScript(napi_env env, napi_callback_info info) {
    // 创建 Promise
    napi_value promise;
    napi_deferred deferred;
    napi_create_promise(env, &deferred, &promise);

    // 解析脚本
    std::string script;
    NapiParseUtils::ParseString(env, argv[0], script);

    // 创建回调
    auto callback = std::make_shared<NWebValueCallbackImpl>(
        env, deferred, true);

    // 调用执行
    controller->RunJavaScript(script, callback);

    return promise;
}

// 2. Controller 转发
// 文件: ohos_nweb/src/nweb.cpp
int32_t WebviewController::RunJavaScript(
    const std::string& script,
    std::shared_ptr<NWebValueCallback> callback) {
    if (!nweb_) {
        return ERROR_WEBVIEW_NOT_ATTACHED;
    }

    return nweb_->RunJavaScript(script, callback);
}

// 3. Chromium V8 执行
// 文件: ArkWebCore.hap (Chromium 源码)
namespace blink {

LocalFrame::ExecuteJavaScript(const String& source) {
    // 1. 获取 V8 上下文
    v8::Local<v8::Context> context = GetScriptContext();
    if (context.IsEmpty()) {
        return nullptr;
    }

    // 2. 创建脚本
    v8::Local<v8::Script> script;
    v8::ScriptCompiler::Source source2(source);
    if (!v8::ScriptCompiler::Compile(
            context, &source2).ToLocal(&script)) {
        return nullptr;
    }

    // 3. 执行脚本
    v8::Local<v8::Value> result;
    if (!script->Run(context).ToLocal(&result)) {
        return nullptr;
    }

    return result;
}

}  // namespace blink
```

### 事件回调调用链

```
渲染进程 (Chromium)
    │
    ▼ (IPC 回调)
    │
content::RenderFrame::DidFinishLoad()
    │
    ▼
NWeb::OnPageEnd(url)
    │
    ▼ (回调注册)
    │
NWebCallback::OnPageEnd(url)
    │
    ▼ (N-API 回调)
    │
napi_call_function(env, callback, url)
    │
    ▼
JS 回调函数执行
```

## 系统能力调用链

### AppFwkUpdateService 调用链

```
N-API: 接口调用
    │
    ▼
AppFwkUpdateClient::VerifyPackageInstall()
    │
    ▼ (IPC)
    │
Binder 调用
    │
    ▼
AppFwkUpdateService::OnRemoteRequest()
    │
    ▼
AppFwkUpdateService::VerifyPackageInstall()
    │
    ▼
BundleMgr API
```

### WebNativeMessagingService 调用链

```
JS: chrome.runtime.sendNativeMessage()
    │
    ▼
N-API: WebNativeMessagingExtension
    │
    ▼
WebNativeMessagingClient::Connect()
    │
    ▼ (IPC)
    │
WebNativeMessagingService::Connect()
    │
    ▼
AbilityManager::ConnectAbility()
    │
    ▼
Extension 启动
```

## 相关文档

| 主题 | 文档 |
|------|------|
| 架构设计 | [01_Architecture.md](../01_Architecture.md) |
| N-API 接口 | [02_N-API.md](../02_N-API.md) |
| 内部模块 | [03_InnerAPI.md](../03_InnerAPI.md) |

---

[返回 SUMMARY.md](../SUMMARY.md)
