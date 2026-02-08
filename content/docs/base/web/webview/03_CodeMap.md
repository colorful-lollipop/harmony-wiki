# ArkWeb WebView 目录结构与代码地图

## 1. 顶层目录结构

```
webview/
├── ohos_nweb/                    # 核心引擎层
├── ohos_adapter/                 # 平台适配层 (30+ 适配器)
├── ohos_interface/               # 接口定义层
├── ohos_glue/                    # 胶水代码层
├── ohos_wrapper/                 # C 风格包装层
├── interfaces/                   # 对外接口层
│   ├── kits/napi/               # ArkTS N-API 接口
│   ├── kits/ani/                # ArkNative Interface
│   ├── kits/cj/                 # Cangjie FFI 接口
│   ├── kits/nativecommon/       # 通用组件
│   └── native/                  # C/C++ NDK 接口
├── sa/                          # 系统服务 (SA)
│   ├── app_fwk_update/          # SA 8350
│   └── web_native_messaging/    # SA 8610
├── arkweb_utils/                # 工具库
├── test/                        # 测试代码 (忽略)
├── wiki/                        # 本文档目录
├── bundle.json                  # 组件元数据
├── config.gni                   # 构建配置
└── web_aafwk.gni               # AAFWK 路径定义
```

---

## 2. 核心模块代码地图

### 2.1 ohos_nweb/ - 核心引擎层

```
ohos_nweb/
├── include/                     # 公共头文件
│   ├── nweb_helper.h           # NWebHelper 核心类
│   ├── nweb_adapter_helper.h   # 适配器辅助类
│   ├── nweb_c_api.h            # C API 接口
│   ├── nweb_config_helper.h    # 配置管理
│   ├── nweb_handler.h          # 事件回调接口
│   ├── nweb_init_params.h      # 初始化参数
│   ├── nweb_surface_adapter.h  # Surface 适配器
│   └── ...
├── src/                         # 实现代码
│   ├── nweb_helper.cpp         # 核心实现 (46K 行)
│   ├── nweb_surface_adapter.cpp
│   ├── nweb_enhance_surface_adapter.cpp
│   ├── nweb_config_helper.cpp
│   ├── nweb_hisysevent.cpp
│   └── nweb_crashpad_handler_main.cpp
├── etc/                         # 配置文件
│   ├── web_config.xml          # 核心配置
│   ├── para/web.para           # 系统参数
│   └── para/web.para.dac       # DAC 权限
├── prebuilts/                   # 预编译包
│   ├── arm64/ArkWebCore.hap
│   └── arm/ArkWebCore.hap
└── BUILD.gn                     # 构建配置
```

**关键类定位**:

| 类名 | 文件路径 | 职责 |
|------|---------|------|
| `NWebHelper` | `include/nweb_helper.h`<br>`src/nweb_helper.cpp` | 核心管理类，单例模式，负责引擎初始化和 WebView 创建 |
| `NWebSurfaceAdapter` | `include/nweb_surface_adapter.h`<br>`src/nweb_surface_adapter.cpp` | Surface 管理，渲染输出 |
| `NWebConfigHelper` | `include/nweb_config_helper.h`<br>`src/nweb_config_helper.cpp` | 解析 web_config.xml |
| `NWebHisysevent` | `include/nweb_hisysevent.h`<br>`src/nweb_hisysevent.cpp` | 系统事件上报 |

### 2.2 ohos_adapter/ - 平台适配层

```
ohos_adapter/
├── graphic_adapter/            # 图形适配器
├── audio_adapter/              # 音频适配器
├── location_adapter/           # 定位适配器
├── camera_adapter/             # 相机适配器
├── battery_mgr_adapter/        # 电池管理适配器
├── power_mgr_adapter/          # 电源管理适配器
├── access_token_adapter/       # 权限令牌适配器
├── net_connect_adapter/        # 网络连接适配器
├── net_proxy_adapter/          # 网络代理适配器
├── mmi_adapter/                # 多模输入适配器
├── hisysevent_adapter/         # 系统事件适配器
├── display_manager_adapter/    # 显示管理适配器
├── sensor_adapter/             # 传感器适配器
└── ... (共 30+ 适配器)
```

**适配器命名规范**:

```
<service>_adapter/
├── include/
│   ├── i_<service>_adapter.h      # 接口定义
│   └── <service>_adapter_impl.h   # 实现声明
├── src/
│   └── <service>_adapter_impl.cpp # 实现
└── BUILD.gn (通常在父目录统一配置)
```

### 2.3 interfaces/kits/napi/ - N-API 接口层

```
interfaces/kits/napi/
├── common/                               # 公共模块
│   ├── napi_webview_native_module.cpp   # 模块注册入口
│   ├── napi_parse_utils.cpp             # 参数解析工具
│   └── business_error.cpp               # 错误处理
├── webviewcontroller/                    # WebView 控制器
│   ├── napi_webview_controller.cpp      # 主控制器 (300+ 方法)
│   ├── napi_webview_controller.h
│   ├── webview_controller.cpp
│   ├── webview_javascript_result_callback.cpp
│   ├── napi_web_download_manager.cpp
│   ├── napi_web_scheme_handler_request.cpp
│   └── ...
├── webcookiemanager/                     # Cookie 管理
│   └── napi_web_cookie_manager.cpp
├── webstorage/                           # WebStorage
│   └── napi_web_storage.cpp
├── webdatabase/                          # 数据库
│   ├── napi_web_data_base.cpp
│   └── napi_geolocation_permission.cpp
├── proxycontroller/                      # 代理控制
│   ├── napi_proxy_controller.cpp
│   ├── napi_proxy_config.cpp
│   └── napi_proxy_rule.cpp
├── webadsblockmanager/                   # 广告拦截
│   └── napi_web_adsblock_manager.cpp
├── webasynccontroller/                   # 异步控制
│   └── napi_web_async_controller.cpp
└── web_native_messaging_extension/       # 原生消息扩展
    ├── extension/
    ├── extension_client/
    ├── extension_manager/
    └── ability/
```

**N-API 类注册位置**:

| JS 类 | C++ 文件 | 注册函数 |
|-------|---------|---------|
| `WebviewController` | `webviewcontroller/napi_webview_controller.cpp` | `NapiWebviewController::Init()` |
| `WebCookieManager` | `webcookiemanager/napi_web_cookie_manager.cpp` | `NapiWebCookieManager::Init()` |
| `WebStorage` | `webstorage/napi_web_storage.cpp` | `NapiWebStorage::Init()` |
| `WebDataBase` | `webdatabase/napi_web_data_base.cpp` | `NapiWebDataBase::Init()` |
| `ProxyController` | `proxycontroller/napi_proxy_controller.cpp` | `NapiProxyController::Init()` |
| `WebAdsBlockManager` | `webadsblockmanager/napi_web_adsblock_manager.cpp` | `NapiWebAdsBlockManager::Init()` |

**模块注册入口**:
- 文件: `interfaces/kits/napi/common/napi_webview_native_module.cpp:90-93`
- 模块名: `"web.webview"`

### 2.4 interfaces/native/ - NDK 接口层

```
interfaces/native/
├── arkweb_interface.h          # 统一入口头文件
├── arkweb_interface.cpp        # 实现
├── native_interface_arkweb.h   # Native API 声明
├── native_interface_arkweb.cpp # Native API 实现
├── arkweb_type.h              # 类型定义
├── arkweb_error_code.h        # 错误码定义
├── arkweb_scheme_handler.h    # Scheme Handler API
├── libohweb.ndk.json          # NDK 库描述
└── BUILD.gn                   # 构建配置
```

**NDK 入口函数**:

```cpp
// arkweb_interface.h
void* OH_ArkWeb_GetNativeAPI(ArkWeb_NativeAPIVariantKind kind);

// 使用示例
NativeControllerAPI* api = (NativeControllerAPI*)OH_ArkWeb_GetNativeAPI(
    ARKWEB_NATIVE_CONTROLLER
);
```

### 2.5 sa/ - 系统服务

#### SA 8610 - WebNativeMessagingService

```
sa/web_native_messaging/
├── service/                                # 服务端实现
│   ├── web_native_messaging_service.h/cpp # 主服务类
│   ├── web_native_messaging_manager.h/cpp # 连接管理器
│   ├── extension_ipc_connection.h/cpp     # IPC 连接
│   └── ...
├── client/                                 # 客户端实现
│   ├── web_native_messaging_client.h/cpp  # 客户端主类
│   └── web_extension_connection_callback.h
├── common/                                 # 公共代码
│   └── i_web_extension_connection_callback.h
├── IWebNativeMessagingService.idl         # IDL 接口定义
├── 8610.json                              # SA 配置文件
└── BUILD.gn                               # 构建配置
```

#### SA 8350 - AppFwkUpdateService

```
sa/app_fwk_update/
├── include/
│   ├── app_fwk_update_service.h
│   └── app_fwk_update_client.h
├── src/
│   ├── app_fwk_update_service.cpp
│   └── app_fwk_update_client.cpp
├── IAppFwkUpdateService.idl
├── 8350.json
└── BUILD.gn
```

---

## 3. 功能到代码的映射

### 3.1 核心功能映射表

| 功能 | 代码入口 | 关键文件 |
|------|---------|---------|
| **WebView 创建** | `NWebHelper::CreateNWeb()` | `ohos_nweb/src/nweb_helper.cpp` |
| **页面加载** | `WebviewController::LoadUrl()` | `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp` |
| **JS 执行** | `WebviewController::RunJavaScript()` | `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp` |
| **JS Bridge 注册** | `WebviewController::RegisterJavaScriptProxy()` | `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp` |
| **Cookie 获取** | `WebCookieManager::FetchCookie()` | `interfaces/kits/napi/webcookiemanager/napi_web_cookie_manager.cpp` |
| **Cookie 设置** | `WebCookieManager::ConfigCookie()` | `interfaces/kits/napi/webcookiemanager/napi_web_cookie_manager.cpp` |
| **消息通道** | `WebviewController::CreateWebMessagePorts()` | `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp` |
| **代理设置** | `ProxyController::SetProxyOverride()` | `interfaces/kits/napi/proxycontroller/napi_proxy_controller.cpp` |
| **下载管理** | `WebDownloadManager::OnDownloadBeforeStart()` | `interfaces/kits/napi/webviewcontroller/napi_web_download_manager.cpp` |
| **Scheme 处理** | `WebSchemeHandler` | `interfaces/kits/napi/webviewcontroller/napi_web_scheme_handler_request.cpp` |

### 3.2 安全相关代码映射

| 安全功能 | 代码位置 |
|---------|---------|
| **权限校验** | `ohos_adapter/access_token_adapter/src/access_token_adapter_impl.cpp` |
| **地理位置权限** | `ohos_adapter/distributeddatamgr_adapter/webdatabase/ohos_web_permission_data_base_adapter_impl.cpp` |
| **IPC 权限检查** | `sa/web_native_messaging/service/web_native_messaging_service.cpp` |
| **UID 校验** | `sa/app_fwk_update/src/app_fwk_update_service.cpp` |
| **参数解析** | `interfaces/kits/napi/common/napi_parse_utils.cpp` |

---

## 4. 构建系统导航

### 4.1 主要 BUILD.gn 文件

| BUILD.gn | 职责 | 关键目标 |
|---------|------|---------|
| `ohos_nweb/BUILD.gn` | 核心引擎 | `libnweb` → `arkweb_core_loader.so` |
| `ohos_adapter/BUILD.gn` | 适配器层 | `nweb_ohos_adapter` → `libnweb_ohos_adapter.so` |
| `ohos_glue/BUILD.gn` | 胶水代码 | `ohos_base_glue_source`, `ohos_nweb_glue_source` |
| `interfaces/kits/napi/BUILD.gn` | N-API 绑定 | `webview_napi` → `libwebview_napi.so` |
| `interfaces/native/BUILD.gn` | NDK 接口 | `ohweb` → `libohweb.so` |
| `sa/web_native_messaging/BUILD.gn` | SA 8610 | `web_native_messaging_service` |
| `sa/app_fwk_update/BUILD.gn` | SA 8350 | `app_fwk_update_service` |

### 4.2 配置文件位置

| 配置 | 文件路径 | 说明 |
|------|---------|------|
| **组件元数据** | `bundle.json` | 版本、依赖、构建目标 |
| **Feature Flags** | `config.gni` | 条件编译开关 |
| **WebView 配置** | `ohos_nweb/etc/web_config.xml` | 运行时行为配置 |
| **系统参数** | `ohos_nweb/etc/para/web.para` | 可调参数 |
| **DAC 配置** | `ohos_nweb/etc/para/web.para.dac` | 参数权限 |
| **SA 配置 8610** | `sa/web_native_messaging/8610.json` | NativeMessaging SA |
| **SA 配置 8350** | `sa/app_fwk_update/8350.json` | AppFwkUpdate SA |

---

## 5. 调试与日志

### 5.1 日志标签

| 模块 | 日志标签 | 查看命令 |
|------|---------|---------|
| 核心引擎 | `ArkWeb` | `hilog -T ArkWeb` |
| 适配器 | `webadapter` | `hilog -T webadapter` |
| NativeMessaging | `web_native_messaging_service` | `hilog -T web_native_messaging_service` |
| AppFwkUpdate | `app_fwk_update_service` | `hilog -T app_fwk_update_service` |

### 5.2 系统参数

```bash
# 查看 WebView 相关参数
param get web.*
param get persist.arkwebcore.*

# 常用参数
persist.arkwebcore.package_name     # 包名
persist.arkwebcore.install_path     # 安装路径
web.engine.install.completed        # 安装完成标志
```

### 5.3 Dump 信息

```bash
# 查看 SA 8610 状态
hidumper -s 8610

# 查看 SA 8350 状态
hidumper -s 8350
```

---

## 6. 代码搜索指南

### 6.1 按功能搜索

**N-API 方法实现**:
```bash
# 搜索特定 N-API 方法
grep -r "LoadUrl" interfaces/kits/napi/ --include="*.cpp"

# 搜索 napi_define_class
grep -r "napi_define_class" interfaces/kits/napi/ --include="*.cpp"
```

**权限检查**:
```bash
# 搜索 AccessToken 校验
grep -r "VerifyAccessToken" ohos_adapter/ --include="*.cpp"

# 搜索 UID 校验
grep -r "GetCallingUid" sa/ --include="*.cpp"
```

**IPC 接口**:
```bash
# 搜索 IDL 文件
find sa/ -name "*.idl"

# 搜索 IRemoteBroker
grep -r "IRemoteBroker" sa/ ohos_adapter/ --include="*.h"
```

### 6.2 快速定位模板

```
功能: <功能名称>
入口: <函数名>
文件: <文件路径>:<行号>
调用链: 
  1. <调用者>
  2. <中间层>
  3. <实现>
```

---

*文档版本: 1.0*  
*更新日期: 2026-02-07*
