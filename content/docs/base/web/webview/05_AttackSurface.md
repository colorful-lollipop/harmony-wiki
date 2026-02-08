# ArkWeb WebView 攻击面分析

## 1. 外部输入清单

### 1.1 N-API 参数输入

| 接口类 | 方法 | 输入类型 | 风险等级 | 代码位置 |
|--------|------|---------|---------|---------|
| **WebviewController** | `loadUrl(url)` | URL 字符串 | 🔴 高 | `napi_webview_controller.cpp:LoadUrl()` |
| **WebviewController** | `loadData(data, mimeType, encoding)` | HTML/数据字符串 | 🔴 高 | `napi_webview_controller.cpp:LoadData()` |
| **WebviewController** | `runJavaScript(script)` | JavaScript 代码 | 🔴 高 | `napi_webview_controller.cpp:RunJavaScript()` |
| **WebviewController** | `registerJavaScriptProxy(obj, name, methods)` | 对象 + 方法名数组 | 🟠 中 | `napi_webview_controller.cpp:RegisterJavaScriptProxy()` |
| **WebviewController** | `createWebMessagePorts()` | - | 🟢 低 | - |
| **WebviewController** | `postMessage(message, uri, ports)` | 消息 + URI | 🟠 中 | `napi_webview_controller.cpp:PostMessage()` |
| **WebviewController** | `setUrlTrustList(trustList)` | JSON 字符串 | 🟠 中 | `napi_webview_controller.cpp:SetUrlTrustList()` |
| **ProxyController** | `setProxyOverride(rules, bypassRules)` | 代理规则字符串 | 🟠 中 | `napi_proxy_controller.cpp:SetProxyOverride()` |
| **WebCookieManager** | `configCookie(url, value)` | Cookie 字符串 | 🟠 中 | `napi_web_cookie_manager.cpp:ConfigCookie()` |
| **WebDownloadManager** | - | 下载 URL | 🟠 中 | - |
| **WebSchemeHandler** | 各种方法 | URL/请求数据 | 🔴 高 | `napi_web_scheme_handler_request.cpp` |

### 1.2 NDK 接口输入

| 函数 | 输入类型 | 风险等级 | 代码位置 |
|------|---------|---------|---------|
| `OH_NativeArkWeb_RunJavaScript()` | JS 代码字符串 | 🔴 高 | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_RegisterJavaScriptProxy()` | 方法名数组 | 🟠 中 | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_LoadData()` | HTML/数据 | 🔴 高 | `native_interface_arkweb.cpp` |
| `OH_NativeArkWeb_SetJavaScriptProxyValidCallback()` | 回调函数 | 🟢 低 | - |
| `OH_ArkWebCookieManager_ConfigCookieSync()` | Cookie 字符串 | 🟠 中 | - |

### 1.3 IPC 接口输入

#### SA 8610 - WebNativeMessagingService

| 方法 | 输入参数 | 风险等级 | IDL 定义 |
|------|---------|---------|---------|
| `ConnectWebNativeMessagingExtension()` | Token, Want, callback, connectionId | 🔴 高 | `IWebNativeMessagingService.idl` |
| `DisconnectWebNativeMessagingExtension()` | connectionId | 🟢 低 | `IWebNativeMessagingService.idl` |
| `StartAbility()` | Token, Want, options | 🔴 高 | `IWebNativeMessagingService.idl` |
| `StopNativeConnectionFromExtension()` | connectionId | 🟢 低 | `IWebNativeMessagingService.idl` |

**风险点分析**:
- `Want` 参数包含 BundleName/AbilityName，可能被恶意构造
- `Token` 需要校验调用者身份
- `connectionId` 需防重放/伪造

#### SA 8350 - AppFwkUpdateService

| 方法 | 输入参数 | 风险等级 | 说明 |
|------|---------|---------|------|
| `VerifyPackageInstall()` | bundleName, hapPath | 🔴 高 | UID 校验限制为 FOUNDATION_UID (5523) |
| `NotifyFWKAfterBmsStart()` | - | 🟢 低 | 无参数 |
| `NotifyArkWebInstallSuccess()` | - | 🟢 低 | 无参数 |

### 1.4 配置文件输入

| 配置文件 | 输入内容 | 风险等级 | 位置 |
|---------|---------|---------|------|
| `web_config.xml` | 渲染配置、性能参数 | 🟠 中 | `ohos_nweb/etc/` |
| `web.para` | 运行时参数 | 🟠 中 | `ohos_nweb/etc/para/` |

---

## 2. 敏感操作清单

### 2.1 系统调用

| 操作类型 | 调用位置 | 权限要求 | 风险 |
|---------|---------|---------|------|
| **网络访问** | 渲染进程 → 网络栈 | `ohos.permission.INTERNET` | 数据外泄 |
| **文件读写** | 下载管理、Cookie 存储 | 应用沙箱内 | 沙箱逃逸 |
| **进程创建** | ArkWebCore 启动 | 系统服务权限 | 权限提升 |
| **IPC 调用** | SA 客户端 | AccessToken 校验 | 越权访问 |

### 2.2 特权接口

| 接口 | 功能 | 访问控制 | 风险 |
|------|------|---------|------|
| `NWebHelper::CreateNWeb()` | 创建 WebView 实例 | 应用权限 | 资源耗尽 |
| `GetCookieManager()` | 获取 Cookie 管理器 | 同源策略 | 跨域窃取 |
| `GetDataBase()` | 获取 Web 数据库 | 应用沙箱 | 数据泄露 |
| `GetWebStorage()` | 获取 WebStorage | 同源策略 | 跨域窃取 |

### 2.3 其他服务调用

| 被调用服务 | 用途 | 代码位置 | 风险 |
|-----------|------|---------|------|
| **AbilityManager** | 启动 Extension | `sa/web_native_messaging/` | 非法启动 |
| **BundleManager** | 获取包信息 | `ohos_adapter/aafwk_adapter/` | 信息泄露 |
| **AccessTokenManager** | 权限校验 | `ohos_adapter/access_token_adapter/` | 权限绕过 |
| **AppSpawn** | 进程孵化 | `sa/app_fwk_update/` | 代码注入 |
| **Location Service** | 定位服务 | `ohos_adapter/location_adapter/` | 隐私泄露 |
| **Camera Service** | 相机访问 | `ohos_adapter/camera_adapter/` | 隐私泄露 |

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         外部 Web 内容                                │
│                    (不可信/半可信区域)                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ 恶意网页    │  │ 正常网页    │  │ 攻击者控制  │                 │
│  │ JavaScript  │  │ JavaScript  │  │ 的 iframe   │                 │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                 │
└─────────┼────────────────┼────────────────┼────────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Renderer 进程 (渲染进程)                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Chromium Blink 引擎 (沙箱内)                                │   │
│  │  • V8 JavaScript 引擎                                       │   │
│  │  • HTML/CSS 解析渲染                                        │   │
│  │  • 网络请求处理                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────┬───────────────────────────────────────────┘
          │               │               │
          │     IPC (Mojo)│               │
          ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Browser 进程 (主进程)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ libnweb.so  │  │ ohos_adapter│  │ 系统服务 SA │                 │
│  │ (NWebHelper)│  │ (30+适配器)  │  │ 8350/8610   │                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
└─────────────────────────┬───────────────────────────────────────────┘
          │               │               │
          ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    应用层 (ArkTS/C++)                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ WebView组件 │  │ N-API接口   │  │ NDK接口     │                 │
│  │ 应用代码    │  │ JS Bridge   │  │ 原生代码    │                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
└─────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    系统服务层 (System)                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ BundleManager│  │ AbilityManager│ │ AppSpawn   │                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
└─────────────────────────────────────────────────────────────────────┘

信任边界:
① Web内容 ↔ Renderer: 沙箱隔离
② Renderer ↔ Browser: Mojo IPC + 权限校验
③ Browser ↔ 应用层: N-API/NDK 接口
④ 应用层 ↔ 系统服务: AccessToken 校验
```

### 关键信任边界说明

| 边界 | 保护机制 | 绕过风险 |
|------|---------|---------|
| **① Web 沙箱** | Chromium 沙箱 + 同源策略 | 沙箱逃逸漏洞 |
| **② IPC 边界** | Mojo Capability | IPC 劫持 |
| **③ API 边界** | N-API 类型检查 | 类型混淆 |
| **④ 系统边界** | AccessToken + SELinux | 权限提升 |

---

## 4. 攻击向量汇总

### 4.1 远程攻击向量

| 向量 | 攻击方式 | 前提条件 | 潜在影响 |
|------|---------|---------|---------|
| **恶意网页** | 利用 WebView 漏洞 | 用户访问恶意 URL | 代码执行、信息窃取 |
| **XSS 注入** | 通过 JS Bridge 注入 | 应用未校验 JS 输入 | 原生代码执行 |
| **URL 欺骗** | 构造恶意 URL Scheme | 自定义 Scheme 处理缺陷 | 钓鱼、跳转攻击 |
| **中间人攻击** | SSL 证书绕过 | SSL 错误处理不当 | 流量劫持 |

### 4.2 本地攻击向量

| 向量 | 攻击方式 | 前提条件 | 潜在影响 |
|------|---------|---------|---------|
| **配置文件篡改** | 修改 web_config.xml | 需要 root 权限 | 配置注入 |
| **HAP 替换** | 替换 ArkWebCore.hap | 系统签名绕过 | 恶意引擎注入 |
| **IPC 劫持** | 伪造 IPC 调用 | SA 权限校验绕过 | 服务滥用 |
| **共享内存攻击** | 利用 Surface/Buffer | 内存管理缺陷 | 信息泄露 |

### 4.3 供应链攻击向量

| 向量 | 攻击方式 | 前提条件 | 潜在影响 |
|------|---------|---------|---------|
| **Chromium 漏洞** | 利用 CVE | 未及时更新引擎 | 远程代码执行 |
| **第三方库** | 依赖库漏洞 | 依赖管理缺陷 | 各种漏洞 |

---

## 5. 历史 CVE 参考

| CVE ID | CVSS | 组件 | 漏洞类型 | 影响版本 |
|--------|------|------|---------|---------|
| CVE-2025-54607 | 7.7 (High) | ArkWeb | 认证管理漏洞 | - |
| CVE-2025-27536 | 5.5 (Medium) | arkcompiler | Type Confusion | - |
| CVE-2025-23414 | 7.8 (High) | ArkWeb | Use-After-Free | - |
| CVE-2025-20024 | - | ArkWeb | 本地代码执行 | v5.0.2 |

---

## 6. 安全测试建议

### 6.1 静态分析重点

```cpp
// 重点关注以下代码模式:

// 1. URL 处理
LoadUrl(url);                          // 检查 URL 校验
SetUrlTrustList(trustList);            // 检查 JSON 解析安全

// 2. JavaScript 执行
RunJavaScript(script);                 // 检查代码注入
RegisterJavaScriptProxy(obj, name);    // 检查对象暴露范围

// 3. IPC 处理
ConnectWebNativeMessagingExtension();  // 检查 Token 校验
StartAbility(want);                    // 检查 Want 参数

// 4. 文件操作
VerifyPackageInstall(bundleName, path); // 检查路径遍历
```

### 6.2 动态测试重点

1. **Fuzzing 测试**
   - 对 N-API 接口进行参数 Fuzzing
   - 对 IPC 接口进行消息 Fuzzing
   - 已有 Fuzz 测试: `test/fuzztest/` 目录

2. **渗透测试**
   - 构造恶意 HTML 页面
   - 测试 JS Bridge 边界
   - 测试 URL Scheme 处理

3. **权限测试**
   - 跨应用 IPC 调用
   - 沙箱逃逸尝试
   - 权限提升测试

---

*文档版本: 1.0*
*更新日期: 2026-02-07*
