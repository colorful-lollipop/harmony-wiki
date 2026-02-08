# 内部模块与接口

> 本文档介绍 ArkWeb WebView 组件的内部模块接口、依赖方向、生命周期和稳定性标注。

## 模块概览

### 模块列表

| 模块 | 职责 | 稳定性 | 路径 |
|------|------|--------|------|
| **ohos_nweb** | 核心引擎封装 | Stable | `ohos_nweb/src/` |
| **ohos_adapter** | 系统适配层 | Stable | `ohos_adapter/` |
| **ohos_interface** | 接口定义契约 | Stable | `ohos_interface/include/` |
| **nativecommon** | N-API 通用组件 | Stable | `interfaces/kits/nativecommon/` |
| **sa** | 系统能力服务 | Stable | `sa/` |
| **arkweb_utils** | 工具库 | Stable | `arkweb_utils/` |

## ohos_nweb 核心模块

### 核心类

#### NWebHelper

**定位**: WebView 核心管理器单例
**头文件**: `ohos_interface/include/ohos_nweb/nweb_helper.h`
**实现**: `ohos_nweb/src/nweb_helper.cpp`

```cpp
class OHOS_NWEB_EXPORT NWebHelper {
public:
    // 单例获取
    static NWebHelper& Instance();

    // 初始化
    bool Init(bool from_ark = true);
    bool InitAndRun(bool from_ark = true);

    // WebView 创建
    std::shared_ptr<NWeb> CreateNWeb(
        std::shared_ptr<NWebCreateInfo> create_info);

    // 辅助管理器获取
    std::shared_ptr<NWebCookieManager> GetCookieManager();
    std::shared_ptr<NWebDataBase> GetDataBase();
    std::shared_ptr<NWebWebStorage> GetWebStorage();

    // WebView 获取
    std::shared_ptr<NWeb> GetNWeb(int32_t nweb_id);

    // 配置
    void SetBundlePath(const std::string& path);
    void SetRenderProcessMode(RenderProcessMode mode);

    // User Agent
    void SetAppCustomUserAgent(const std::string& userAgent);
    std::string GetDefaultUserAgent();

    // 资源预取
    void PrefetchResource(
        const std::shared_ptr<NWebEnginePrefetchArgs>& pre_args,
        const std::map<std::string, std::string>& additional_http_headers,
        const std::string& cache_key,
        const uint32_t& cache_valid_time);
};
```

#### NWeb

**定位**: WebView 核心接口
**头文件**: `ohos_interface/include/ohos_nweb/nweb.h`
**稳定性**: Stable

```cpp
class OHOS_NWEB_EXPORT NWeb {
public:
    virtual ~NWeb() = default;

    // 页面加载
    virtual int32_t LoadURL(const std::string& url) = 0;
    virtual int32_t LoadURLWithHeaders(
        const std::string& url,
        const std::map<std::string, std::string>& additionalHeaders) = 0;
    virtual int32_t LoadDataWithBaseURL(
        const std::string& baseUrl,
        const std::string& data,
        const std::string& mimeType,
        const std::string& encoding,
        const std::string& historyUrl) = 0;

    // 导航
    virtual int32_t Backward() = 0;
    virtual int32_t Forward() = 0;
    virtual int32_t BackwardOrForward(int32_t step) = 0;
    virtual int32_t Refresh() = 0;
    virtual int32_t Stop() = 0;

    // JavaScript
    virtual int32_t RunJavaScript(
        const std::string& script,
        std::shared_ptr<NWebValueCallback> callback) = 0;
    virtual int32_t RegisterJavaScriptProxy(
        const std::string& objectName,
        const std::vector<std::string>& methodNames,
        const std::shared_ptr<NWebValueCallback>& callback) = 0;
    virtual void UnregisterJavaScriptProxy(const std::string& objectName) = 0;

    // 事件回调
    virtual void SetNWebCallback(std::shared_ptr<NWebCallback>) = 0;

    // 销毁
    virtual void Destroy() = 0;
};
```

#### NWebCreateInfo

**定位**: WebView 创建信息
**头文件**: `ohos_interface/include/ohos_nweb/nweb_create_info.h`

```cpp
struct OHOS_NWEB_EXPORT NWebCreateInfo {
    std::string com_name;                          // 组件名称
    std::shared_ptr<NWebSurfaceAdapter> surface_adapter;  // Surface 适配器
    std::shared_ptr<NWebHandler> nweb_delegate;   // 事件处理器
    std::shared_ptr<NWebPreference> preference;    // 偏好设置
    std::shared_ptr<NWebPdfConfig> pdf_config;     // PDF 配置
};
```

### NWeb 事件回调

#### NWebCallback

**定位**: WebView 事件回调接口
**头文件**: `ohos_interface/include/ohos_nweb/nweb_callback.h`

```cpp
class OHOS_NWEB_EXPORT NWebCallback {
public:
    virtual void OnPageLoadEnd(const std::string& url, bool succeeded) = 0;
    virtual void OnPageBegin(const std::string& url) = 0;
    virtual void OnPageEnd(const std::string& url, int errCode, const std::string& errMsg) = 0;
    virtual void OnLoadingProgressChange(int32_t newProgress) = 0;
    virtual void OnTitleReceive(const std::string& title) = 0;
    virtual void OnGeolocationShow(const std::string& origin) = 0;
    virtual void OnGeolocationHide(const std::string& origin) = 0;
    virtual void OnRequestFocus() = 0;
    virtual void OnDownloadStart(const std::string& url, const std::string& userAgent,
        const std::string& contentDisposition, const std::string& mimeType,
        long long contentLength) = 0;
    virtual void OnRenderProcessNotResponding(const std::string& reason) = 0;
    virtual void OnRenderProcessResponding() = 0;
    virtual void OnRenderProcessGone(bool gone, const NWebRenderProcessGoneDetail& detail) = 0;
    virtual void OnRefreshAccessedHistory(const std::string& url, bool isRefreshed) = 0;
    virtual void OnUrlAccessForbidden(const std::string& url) = 0;
};
```

#### NWebHandler

**定位**: WebView 事件处理器接口
**头文件**: `ohos_interface/include/ohos_nweb/nweb_handler.h`

### NWeb 辅助接口

| 接口 | 职责 |
|------|------|
| `NWebCookieManager` | Cookie 管理 |
| `NWebDataBase` | 数据库管理 |
| `NWebWebStorage` | Web Storage 管理 |
| `NWebAdsBlockManager` | 广告拦截管理 |
| `NWebPreference` | 偏好设置 |
| `NWebPdfConfig` | PDF 配置 |

## ohos_adapter 适配器模块

### 适配器列表

#### 图形适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `GraphicAdapter` | `graphic_adapter.h` | 图形渲染 |
| `NativeImageAdapter` | `native_image_adapter.h` | 原生图像 |
| `VSyncAdapter` | `vsync_adapter.h` | 垂直同步 |

#### 媒体适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `MediaAdapter` | `media_adapter.h` | 媒体播放 |
| `CodecAdapter` | `codec_adapter.h` | 编解码 |
| `PlayerFrameworkAdapter` | `player_framework_adapter.h` | 播放框架 |
| `ImageAdapter` | `image_adapter.h` | 图像解码 |
| `ImageDecoderAdapter` | `image_decoder_adapter.h` | 图像解码器 |

#### 音频适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `AudioAdapter` | `audio_adapter.h` | 音频播放 |
| `AudioCapturerAdapter` | `audio_capturer_adapter.h` | 音频录制 |
| `AudioRendererAdapter` | `audio_renderer_adapter.h` | 音频渲染 |

#### 网络适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `NetConnectAdapter` | `net_connect_adapter.h` | 网络连接 |
| `NetProxyAdapter` | `net_proxy_adapter.h` | 网络代理 |

#### 输入适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `MMIAdapter` | `mmi_adapter.h` | 多模输入 |
| `MultimodalInputAdapter` | `multimodalinput_adapter.h` | 多模输入 |
| `InputMethodFrameworkAdapter` | `inputmethodframework_adapter.h` | 输入法 |

#### 系统服务适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `PowerMgrAdapter` | `power_mgr_adapter.h` | 电源管理 |
| `BatteryMgrAdapter` | `battery_mgr_adapter.h` | 电池管理 |
| `SensorAdapter` | `sensor_adapter.h` | 传感器 |
| `LocationAdapter` | `location_adapter.h` | 位置服务 |
| `DisplayManagerAdapter` | `display_manager_adapter.h` | 显示管理 |

#### 安全适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `AccessTokenAdapter` | `access_token_adapter.h` | 访问令牌 |
| `CertMgrAdapter` | `cert_mgr_adapter.h` | 证书管理 |
| `KeystoreAdapter` | `keystore_adapter.h` | 密钥库 |

#### 数据管理适配器

| 适配器 | 头文件 | 职责 |
|--------|--------|------|
| `PasteboardAdapter` | `pasteboard_adapter.h` | 剪贴板 |
| `DataShareAdapter` | `datashare_adapter.h` | 数据共享 |
| `DistributedDataMgrAdapter` | `distributeddatamgr_adapter.h` | 分布式数据 |

### 适配器使用模式

```cpp
// 1. 定义接口 (ohos_interface/include/ohos_adapter/xxx_adapter.h)
class GraphicAdapter {
public:
    virtual int32_t DrawRect(int32_t x, int32_t y, int32_t w, int32_t h) = 0;
    virtual int32_t Clear() = 0;
protected:
    virtual ~GraphicAdapter() = default;
};

// 2. 实现接口 (ohos_adapter/graphic_adapter/src/xxx_adapter_impl.cpp)
class GraphicAdapterImpl : public GraphicAdapter {
public:
    int32_t DrawRect(int32_t x, int32_t y, int32_t w, int32_t h) override {
        // 调用 OpenHarmony Graphic 接口
    }
    int32_t Clear() override {
        // 清除绘制
    }
};

// 3. 注册实现 (ohos_adapter/ohos_adapter_helper.cpp)
AdapterHelper::Register<GraphicAdapter, GraphicAdapterImpl>();

// 4. 使用 (ohos_nweb/)
auto adapter = AdapterHelper::Get<GraphicAdapter>();
adapter->DrawRect(x, y, w, h);
```

## sa 系统能力模块

### AppFwkUpdateService (SA 8350)

**定位**: ArkWebCore HAP 包更新管理服务
**路径**: `sa/app_fwk_update/`

```cpp
// 接口定义
class AppFwkUpdateService : public SystemAbility, 
                           public AppFwkUpdateServiceStub {
    // 验证 HAP 安装
    int32_t VerifyPackageInstall(const std::string& bundleName, 
                                 const std::string& hapPath);
    
    // 更新 HAP
    int32_t UpdateHap(const std::string& bundleName,
                       const std::string& hapPath);
};

// 使用
AppFwkUpdateClient& client = AppFwkUpdateClient::GetInstance();
client.VerifyPackageInstall(bundleName, hapPath);
```

### WebNativeMessagingService (SA 8610)

**定位**: Web 与 Native Extension 通信服务
**路径**: `sa/web_native_messaging/`

```cpp
// 接口定义
class WebNativeMessagingService : public SystemAbility,
                                  public WebNativeMessagingServiceStub {
    // 连接 Extension
    void ConnectWebNativeMessagingExtension(...) override;
    
    // 断开连接
    void DisconnectWebNativeMessagingExtension(...) override;
};

// 客户端使用
WebNativeMessagingClient& client = WebNativeMessagingClient::GetInstance();
client.ConnectWebNativeMessagingExtension(token, want, callback, connectionId);
```

## nativecommon 通用组件

### TransferableObject

**定位**: 支持跨语言传递的对象基类
**头文件**: `interfaces/kits/nativecommon/include/transferable_object.h`

```cpp
class TransferableObject {
public:
    virtual napi_value ToNapiValue(napi_env env) = 0;
    static std::shared_ptr<TransferableObject> FromNapiValue(
        napi_env env, napi_value value);
};
```

### WebHistoryList

**定位**: 历史记录列表封装
**头文件**: `interfaces/kits/nativecommon/include/web_history_list.h`

```cpp
class WebHistoryList : public TransferableObject {
public:
    int32_t GetCurrentIndex();
    std::shared_ptr<NWebHistoryItem> GetCurrentItem();
    std::shared_ptr<NWebHistoryItem> GetItemAtIndex(int32_t index);
    int32_t GetSize();
};
```

### WebMessagePort

**定位**: 消息端口封装
**头文件**: `interfaces/kits/nativecommon/include/web_message_port.h`

```cpp
class WebMessagePort : public TransferableObject {
public:
    void Close();
    void PostMessage(const std::string& message);
    void SetMessageCallback(
        std::function<void(const std::string&)> callback);
};
```

## arkweb_utils 工具库

### ArkwebUtils

**定位**: 通用工具函数
**头文件**: `arkweb_utils/arkweb_utils.h`

```cpp
class ArkwebUtils {
public:
    // BundleName 管理
    static void SetBundleName(const std::string& bundleName);
    static std::string GetBundleName();
    
    // 路径处理
    static std::string GetWebEngineLibPath();
    static std::string GetWebEngineConfigPath();
};
```

## 生命周期管理

### WebView 生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                        WebView 生命周期                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐      ┌─────────┐      ┌─────────┐              │
│   │ 创建    │ ───▶ │  初始化  │ ───▶ │ 加载 URL │              │
│   │ Create  │      │ Init()   │      │ LoadURL │              │
│   └─────────┘      └─────────┘      └────┬────┘              │
│                                          │                     │
│                                          ▼                     │
│   ┌─────────┐      ┌─────────┐      ┌─────────┐              │
│   │ 销毁    │ ◀─── │  暂停    │ ◀─── │  显示    │              │
│   │ Destroy │      │ Pause()  │      │  Resume │              │
│   └─────────┘      └─────────┘      └─────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 核心生命周期方法

| 阶段 | 方法 | 说明 |
|------|------|------|
| 创建 | `NWebHelper::CreateNWeb()` | 创建 WebView 实例 |
| 初始化 | `NWeb::Init()` | 初始化渲染 |
| 加载 | `NWeb::LoadURL()` | 加载页面 |
| 暂停 | `NWeb::Pause()` | 暂停渲染 |
| 恢复 | `NWeb::Resume()` | 恢复渲染 |
| 销毁 | `NWeb::Destroy()` | 销毁资源 |

### 渲染进程生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                      渲染进程生命周期                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│   │  创建     │───▶│  活跃     │───▶│  空闲     │              │
│   │ Create    │    │ Active    │    │ Idle      │              │
│   └──────────┘    └─────┬──────┘    └─────┬──────┘              │
│                          │                 │                     │
│                          │  超时退出       │                     │
│                          ▼                 ▼                     │
│                    ┌──────────┐    ┌──────────┐              │
│                    │  后台     │    │  退出     │              │
│                    │ Background│    │ Exit      │              │
│                    └──────────┘    └──────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 稳定性标注

### 稳定性等级

| 等级 | 标注 | 说明 |
|------|------|------|
| **Stable** | 稳定接口 | 可直接使用，推荐 |
| **Beta** | 测试接口 | 可能在未来版本变更 |
| **Internal** | 内部接口 | 仅限系统使用，不对外公开 |

### 稳定性分布

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| **interfaces/kits/napi/** | Stable | N-API 接口 |
| **interfaces/native/** | Stable | NDK 接口 |
| **ohos_interface/include/** | Stable | 接口契约 |
| **ohos_nweb/src/** | Stable | 核心实现 |
| **ohos_adapter/** | Stable | 系统适配 |
| **sa/** | Stable | 系统能力 |
| **arkweb_utils/** | Stable | 工具库 |

## 依赖方向

### 模块依赖图

```
interfaces/kits/napi/  ──────┐
                              │
interfaces/kits/ani/  ──────┤
                              │
interfaces/native/     ──────┤
                              │
nativecommon          ──────┤
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  ohos_interface/include/                     │
│         (接口契约层 - 纯虚接口，不依赖实现)                  │
└─────────────────────────────────────────────────────────────┘
                              │
           ┌──────────────────┴──────────────────┐
           ▼                                      ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│       ohos_nweb/        │      │      ohos_adapter/      │
│    (核心实现层)         │      │   (适配器实现层)        │
│  依赖 ohos_interface   │      │  依赖 ohos_interface   │
│  依赖 ohos_adapter     │      │  依赖系统服务          │
│  依赖 ArkWebCore.hap   │      │                       │
└─────────────────────────┘      └─────────────────────────┘
           │                                      │
           │                                      │
           ▼                                      ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│     ArkWebCore.hap     │      │   OpenHarmony 系统      │
│   (Chromium 引擎)       │      │      服务层              │
└─────────────────────────┘      └─────────────────────────┘
```

### 依赖原则

| 原则 | 描述 |
|------|------|
| **接口隔离** | 上层依赖接口，不依赖实现 |
| **单向依赖** | 依赖方向从外到内 |
| **无环依赖** | 禁止循环依赖 |
| **最小依赖** | 只依赖必要的模块 |

## 相关文档

| 主题 | 文档 |
|------|------|
| 架构设计 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_N-API.md](./02_N-API.md) |
| 构建系统 | [04_Build.md](./04_Build.md) |
| 安全机制 | [05_Security.md](./05_Security.md) |

---

[返回 SUMMARY.md](./SUMMARY.md) | [上一章: N-API 接口](./02_N-API.md) | [下一章: 构建系统](./04_Build.md)
