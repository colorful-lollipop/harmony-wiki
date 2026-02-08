# ArkWeb WebView 项目概览

## 一句话定义

**ArkWeb WebView** 是 OpenHarmony 系统官方提供的 WebView 组件，基于 Chromium/CEF 引擎构建，通过 N-API、NDK 和 IPC 接口为上层应用提供 Web 内容渲染能力。

---

## 能力边界

### 能做什么 ✅

| 能力类别 | 具体功能 | 接口类型 |
|---------|---------|---------|
| **页面渲染** | 加载并渲染 HTML/CSS/JavaScript 页面 | N-API / NDK |
| **页面导航** | 前进、后退、刷新、停止加载 | N-API / NDK |
| **JavaScript 交互** | 执行 JS 代码、注册 JS Bridge、双向通信 | N-API / NDK |
| **Cookie 管理** | 获取、设置、清除 Cookie | N-API / NDK |
| **数据存储** | LocalStorage、IndexedDB、WebSQL 管理 | N-API |
| **地理位置** | 获取设备地理位置（需权限） | N-API |
| **媒体播放** | 原生媒体播放器控制 | N-API |
| **下载管理** | 文件下载控制与事件监听 | N-API |
| **代理配置** | HTTP/HTTPS 代理规则设置 | N-API |
| **自定义协议** | 拦截自定义 URL Scheme | N-API / NDK |
| **广告拦截** | 基于规则的广告过滤 | N-API |
| **消息通道** | Web ↔ Native 双向消息传递 (WebMessagePort) | N-API / NDK |
| **原生消息扩展** | Web 应用与 Native Extension 通信 (SA 8610) | IPC |

### 不能做什么 ❌

| 限制类别 | 说明 |
|---------|------|
| **跨域访问** | 受同源策略限制，需 CORS 支持 |
| **本地文件直接访问** | 需通过自定义协议或 File API |
| **系统级操作** | 无法直接调用系统服务（需通过 N-API 桥接）|
| **多进程控制** | Renderer/GPU 进程由引擎管理，应用无法直接控制 |
| **内核定制** | Chromium 引擎为预编译 HAP，应用无法修改 |

---

## 运行环境

### 系统依赖

```
OpenHarmony 3.1+ (API 9+)
├── SystemCapability.Web.Webview.Core
├── 依赖子系统 (bundle.json 中声明):
│   ├── ability_runtime      # Ability 框架
│   ├── access_token         # 权限管理
│   ├── ace_engine           # ArkUI 引擎
│   ├── graphic_2d           # 图形渲染
│   ├── window_manager       # 窗口管理
│   ├── multimedia_camera_framework  # 相机
│   ├── location             # 定位服务
│   └── ... (共 60+ 依赖组件)
```

### 权限要求

| 权限 | 用途 | 级别 |
|------|------|------|
| `ohos.permission.INTERNET` | 网络访问 | normal |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络状态 | normal |
| `ohos.permission.LOCATION` | 地理位置 | dangerous |
| `ohos.permission.CAMERA` | 相机访问 | dangerous |
| `ohos.permission.MICROPHONE` | 麦克风 | dangerous |
| `ohos.permission.READ_MEDIA` | 读取媒体文件 | dangerous |
| `ohos.permission.WRITE_MEDIA` | 写入媒体文件 | dangerous |

### 资源占用

| 指标 | 数值 | 说明 |
|------|------|------|
| **ROM** | ~85MB | ArkWebCore.hap + 库文件 |
| **RAM** | ~150MB | 运行时内存（含 Renderer 进程）|
| **渲染进程上限** | 20 个 | 可配置 (web_config.xml) |

---

## 软件架构

### 分层架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (ArkTS/JS)                         │
│              import webview from '@ohos.web.webview'        │
└─────────────────────────┬───────────────────────────────────┘
                          │ N-API 绑定
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                 N-API 层 (interfaces/kits/napi)              │
│  ┌──────────────┬──────────────┬──────────────┐             │
│  │ WebviewController │ WebCookieManager │ WebStorage    │             │
│  └──────────────┴──────────────┴──────────────┘             │
└─────────────────────────┬───────────────────────────────────┘
                          │ nativecommon 封装
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              nativecommon 层 (通用组件)                       │
│         WebHistoryList, WebMessagePort 等引用计数管理        │
└─────────────────────────┬───────────────────────────────────┘
                          │ ohos_interface 接口
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              ohos_nweb 层 (核心引擎封装)                      │
│              NWebHelper, SurfaceAdapter                      │
│                libnweb.so (arkweb_core_loader)               │
└─────────────────────────┬───────────────────────────────────┘
                          │ dlopen 加载
                          ▼
┌─────────────────────────────────────────────────────────────┐
│           ArkWebCore.hap (Chromium/CEF 引擎)                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Renderer │ │  GPU     │ │ Utility  │ │ Browser  │       │
│  │  Process │ │ Process  │ │ Process  │ │ Process  │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              ohos_adapter 层 (平台适配器)                     │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │ graphic_   │ │ audio_     │ │ location_  │ ...          │
│  │ adapter    │ │ adapter    │ │ adapter    │              │
│  └────────────┘ └────────────┘ └────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### 模块职责

| 模块 | 职责 | 关键组件 |
|------|------|---------|
| **ohos_nweb** | 引擎加载、生命周期管理、Surface 适配 | NWebHelper, NWebSurfaceAdapter |
| **ohos_adapter** | 平台服务抽象（图形、音频、网络等）| 30+ 适配器 |
| **ohos_glue** | 层间粘合代码生成 | Glue 代码 |
| **interfaces/kits/napi** | ArkTS/JS 绑定 | N-API 实现 |
| **interfaces/native** | C/C++ NDK 接口 | libohweb.so |
| **sa/** | 系统服务 | SA 8350, SA 8610 |

---

## 快速开始

### 1. 创建 WebView 组件 (ArkTS)

```typescript
// Index.ets
import webview from '@ohos.web.webview';

@Entry
@Component
struct WebViewDemo {
  // 创建 WebView 控制器
  controller: webview.WebviewController = new webview.WebviewController();

  build() {
    Column() {
      // 基础 WebView
      Web({ 
        src: 'https://www.example.com', 
        controller: this.controller 
      })
        .width('100%')
        .height('100%')
        .onPageBegin((event) => {
          console.log('Page start: ' + event.url);
        })
        .onPageEnd((event) => {
          console.log('Page finish: ' + event.url);
        })
        .onErrorReceive((event) => {
          console.error('Error: ' + event.error.getErrorInfo());
        });
    }
  }
}
```

### 2. 执行 JavaScript

```typescript
// 同步执行 JS
this.controller.runJavaScript('document.title')
  .then((result) => {
    console.log('Page title: ' + result);
  })
  .catch((error) => {
    console.error('JS execution failed: ' + error);
  });

// 带回调的 JS 执行
this.controller.runJavaScript('1 + 1', (result) => {
  console.log('Result: ' + result);
});
```

### 3. 注册 JavaScript Bridge

```typescript
// 定义要暴露给 JS 的对象
class JsBridge {
  test(): string {
    return 'Hello from Native';
  }

  asyncTestBool(boolValue: boolean): Promise<boolean> {
    return new Promise((resolve) => {
      resolve(!boolValue);
    });
  }
}

// 注册 JS Bridge
let jsBridge = new JsBridge();
this.controller.registerJavaScriptProxy(
  jsBridge, 
  "nativeBridge", 
  ["test"],                    // 同步方法
  ["asyncTestBool"]            // 异步方法
);

// 网页中调用
// window.nativeBridge.test();
// window.nativeBridge.asyncTestBool(true).then(result => console.log(result));
```

### 4. 使用 NDK (C/C++)

```cpp
#include <arkweb_interface.h>
#include <native_interface_arkweb.h>

// 获取 Native API
ArkWeb_NativeAPIVariantKind kind = ARKWEB_NATIVE_CONTROLLER;
NativeControllerAPI* controllerAPI = 
    (NativeControllerAPI*)OH_ArkWeb_GetNativeAPI(kind);

// 运行 JavaScript
controllerAPI->runJavaScript(
    webTag, 
    "console.log('Hello from NDK')", 
    callback
);
```

---

## 关键文件位置

| 文件类型 | 路径 | 说明 |
|---------|------|------|
| **N-API 入口** | `interfaces/kits/napi/common/napi_webview_native_module.cpp` | 模块注册 |
| **核心头文件** | `ohos_nweb/include/nweb_helper.h` | NWebHelper 类 |
| **配置文件** | `ohos_nweb/etc/web_config.xml` | WebView 行为配置 |
| **NDK 接口** | `interfaces/native/arkweb_interface.h` | C API 入口 |
| **SA 配置** | `sa/web_native_messaging/8610.json` | SA 8610 配置 |
| **构建配置** | `bundle.json` | 组件元数据 |

---

## 相关资源

- **OpenHarmony 官方文档**: https://gitee.com/openharmony/docs
- **ArkWeb API 参考**: `docs/en/application-dev/reference/apis-arkweb/`
- **Chromium 源码**: `third_party_chromium` 仓库
- **CEF 源码**: `third_party_cef` 仓库

---

*文档版本: 1.0*
*更新日期: 2026-02-07*
