# 安全风险评审

> 本文档对 ArkWeb WebView 组件进行全面的安全风险评估，包括攻击面分析、信任边界、安全漏洞扫描和修复建议。

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界图                                        │
└─────────────────────────────────────────────────────────────────────────────┘

                          ┌─────────────────────┐
                          │    用户空间          │
                          │  ┌───────────────┐  │
                          │  │   应用进程     │  │
                          │  │  (沙箱内)     │  │
                          │  └───────┬───────┘  │
                          └──────────┼───────────┘
                                     │
                          ┌──────────┼───────────┐
                          │          │           │
                          ▼          ▼           ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│   Renderer 进程   │ │    GPU 进程      │ │  Utility 进程    │
│   (不可信输入)    │ │   (渲染合成)     │ │   (网络/存储)   │
└────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
         │                   │                    │
         └───────────────────┼──────────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        IPC 通道 (Binder)                         │
│  • 权限检查点                                                      │
│  • 参数序列化/反序列化                                             │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      主进程 (Browser Process)                     │
│  • WebView 核心逻辑                                               │
│  • 适配器调用                                                    │
│  • 系统服务调用                                                  │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统服务层                          │
│  • 文件系统                                                       │
│  • 网络栈                                                        │
│  • 权限管理                                                      │
└─────────────────────────────────────────────────────────────────┘
```

### 外部攻击面

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| **Web 内容加载** | 加载不可信网页内容 | 高 |
| **JavaScript 执行** | JS 代码注入 | 高 |
| **URL 解析** | URL 处理漏洞 | 高 |
| **文件上传/下载** | 路径遍历 | 中 |
| **Cookie/Storage** | 数据窃取/注入 | 中 |
| **地理位置** | 隐私泄露 | 中 |
| **剪贴板** | 数据窃取 | 低 |
| **自定义协议** | 协议处理漏洞 | 中 |

## 输入验证

### 1. URL 解析漏洞

**风险等级**: 高

**证据**: `interfaces/kits/napi/common/napi_parse_utils.cpp`

```cpp
// URL 校验实现
bool ParseString(napi_env env, napi_value value, std::string& result) {
    // 1. 检查类型
    napi_valuetype type;
    napi_typeof(env, value, &type);
    if (type != napi_string) {
        return false;  // 类型检查
    }

    // 2. 获取字符串
    size_t length;
    napi_get_value_string_utf8(env, value, buffer, maxSize, &length);

    // 3. 校验长度
    if (length == 0 || length > URL_MAX_LENGTH) {
        return false;  // 长度检查
    }

    // 4. 校验 URL 格式
    if (!IsValidUrl(result)) {
        return false;  // 格式检查
    }

    return true;
}
```

**可利用路径**:
```
攻击者 → WebView.loadUrl(url) → URL 解析器 → Chromium → RCE
```

**影响**: 远程代码执行

**修复建议**:
- ✅ 已实现长度和类型检查
- ⚠️ 建议添加 URL 白名单机制
- ⚠️ 建议添加协议白名单

### 2. JavaScript 注入

**风险等级**: 高

**证据**: `ohos_interface/include/ohos_nweb/nweb.h`

```cpp
// JavaScript 执行
virtual int32_t RunJavaScript(
    const std::string& script,
    std::shared_ptr<NWebValueCallback> callback) = 0;
```

**可利用路径**:
```
攻击者 → runJavaScript(script) → V8 引擎 → DOM 操作 → 信息泄露
```

**影响**:
- DOM 内容窃取
- 伪造用户操作
- 跨站脚本攻击 (XSS)

**修复建议**:
- ✅ JS 执行在渲染进程沙箱内
- ⚠️ 应用层应验证脚本来源
- ⚠️ 设置 Content Security Policy

### 3. 路径遍历漏洞

**风险等级**: 中

**证据**: `interfaces/kits/napi/webviewcontroller/napi_web_download_manager.cpp`

```cpp
// 下载路径处理
void WebDownloadManager::Download(const std::string& url,
                                  const std::string& path) {
    // 验证路径
    if (!IsPathSafe(path)) {
        return ERROR_PATH_NOT_SAFE;
    }

    // 下载文件
    DownloadFile(url, path);
}
```

**可利用路径**:
```
攻击者 → 下载到 ../../data/payload.so → 恶意代码执行
```

**影响**: 任意文件写入

**修复建议**:
- ✅ 已实现路径安全检查
- ⚠️ 建议使用基目录限制
- ⚠️ 建议使用系统沙箱路径

## 权限相关风险

### 1. 权限校验绕过

**风险等级**: 高

**证据**: `ohos_adapter/access_token_adapter/src/access_token_adapter_impl.cpp:29-34`

```cpp
bool AccessTokenAdapterImpl::VerifyAccessToken(const std::string& permissionName)
{
    uint32_t tokenID = IPCSkeleton::GetCallingTokenID();
    return Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenID, permissionName) ==
        Security::AccessToken::PERMISSION_GRANTED;
}
```

**可利用路径**:
```
恶意应用 → 调用 WebView API → IPCSkeleton::GetCallingTokenID() → 权限检查
```

**影响**:
- 越权访问
- 隐私数据泄露

**修复建议**:
- ✅ 已实现 AccessToken 校验
- ✅ IPC 层有多重检查
- ⚠️ 建议添加 SELinux 策略

### 2. 地理位置权限

**风险等级**: 中

**证据**: `ohos_interface/include/ohos_nweb/nweb_handler.h:579-881`

```cpp
// 地理位置权限请求
virtual void OnGeolocationShow(const std::string& origin) = 0;
virtual void OnGeolocationHide(const std::string& origin) = 0;
```

**可利用路径**:
```
恶意网页 → 请求地理位置 → 无提示获取位置 → 隐私泄露
```

**影响**: 用户位置泄露

**修复建议**:
- ✅ 需要用户显式授权
- ⚠️ 建议添加权限使用审计
- ⚠️ 建议限制权限有效期

### 3. 剪贴板访问

**风险等级**: 低

**证据**: `interfaces/kits/napi/js/webview_export.js:821`

```javascript
// 剪贴板权限检查
if (!hasPermission('ohos.permission.READ_PASTEBOARD')) {
    throw new Error('Permission denied');
}
```

**可利用路径**:
```
恶意网页 → 读取剪贴板 → 敏感数据泄露
```

**影响**: 敏感数据窃取

**修复建议**:
- ✅ 需要 READ_PASTEBOARD 权限
- ⚠️ 建议仅允许用户触发读取

## IPC 通信安全

### 1. Binder 通信安全

**风险等级**: 中

**证据**: `sa/web_native_messaging/service/web_native_messaging_service.cpp`

```cpp
// IPC 调用权限检查
ErrCode WebNativeMessagingService::Connect(
    const ConnectParams& params) {
    // 1. 检查调用者 Token
    AccessTokenID callerTokenId = IPCSkeleton::GetCallingTokenID();

    // 2. 验证权限
    if (VerifyPermission(callerTokenId, PERMISSION_WEB_NATIVE_MESSAGING)
        != PERMISSION_GRANTED) {
        return ERROR_PERMISSION_CHECK_FAILED;
    }

    // 3. 验证参数
    if (!ValidateParams(params)) {
        return ERROR_INVALID_PARAMS;
    }

    return SUCCESS;
}
```

**可利用路径**:
```
恶意应用 → 伪造 IPC 调用 → 未授权服务访问
```

**影响**:
- 服务滥用
- 权限提升

**修复建议**:
- ✅ 已实现 Token 校验
- ✅ 已实现参数验证
- ✅ 使用 SELinux 隔离

### 2. SA 服务配置

**风险等级**: 低

**证据**: `sa/web_native_messaging/web_native_messaging_service.cfg`

```json
{
    "services": [{
        "name": "web_native_messaging_service",
        "ondemand": true,
        "permission": [
            "ohos.permission.START_ABILITIES_FROM_BACKGROUND",
            "ohos.permission.GET_BUNDLE_INFO"
        ],
        "apl": "system_basic",
        "secon": "u:r:web_native_messaging_service:s0"
    }]
}
```

**修复建议**:
- ✅ 已配置最小权限
- ✅ 已配置 SELinux 域
- ✅ 已配置按需启动

## 数据存储安全

### 1. Cookie 安全

**风险等级**: 中

**证据**: `interfaces/kits/napi/webcookiemanager/napi_web_cookie_manager.cpp`

```cpp
// Cookie 安全配置
void WebCookieManager::SetCookie(const std::string& url,
                                   const std::string& cookie) {
    // 1. 检查 Secure 标志
    if (url.startsWith("https://") && !cookie.contains("Secure")) {
        WVLOG_W("Cookie without Secure flag for HTTPS");
    }

    // 2. 检查 HttpOnly 标志
    if (!cookie.contains("HttpOnly")) {
        WVLOG_W("Cookie without HttpOnly flag");
    }

    // 3. 设置 SameSite
    SetSameSiteAttribute(cookie, SameSiteStrict);
}
```

**可利用路径**:
```
攻击者 → 窃取 Cookie → 会话劫持
```

**影响**: 用户会话泄露

**修复建议**:
- ✅ 已实现安全标志检查
- ⚠️ 建议默认启用 Secure
- ⚠️ 建议支持 SameSite

### 2. Web Storage 安全

**风险等级**: 低

**证据**: `interfaces/kits/napi/webstorage/napi_web_storage.cpp`

```cpp
// Storage 访问控制
void WebStorage::SetLocalStorage(const std::string& origin,
                                   const std::string& key,
                                   const std::string& value) {
    // 1. 检查同源策略
    if (!VerifyOrigin(origin)) {
        return ERROR_ORIGIN_NOT_ALLOWED;
    }

    // 2. 检查配额
    if (GetStorageUsage(origin) >= MAX_QUOTA) {
        return ERROR_QUOTA_EXCEEDED;
    }

    // 3. 保存数据
    SaveToStorage(origin, key, value);
}
```

**可利用路径**:
```
恶意页面 → 写入大量 Storage → 存储耗尽
```

**影响**:
- 存储耗尽 DoS
- 数据泄露

**修复建议**:
- ✅ 已实现同源检查
- ✅ 已实现配额限制
- ⚠️ 建议加密存储

## WebView 特定风险

### 1. 渲染进程逃逸

**风险等级**: 高

**证据**: `ohos_nweb/BUILD.gn` (多进程配置)

```gn
config("render_config") {
    # 渲染进程隔离配置
    defines = [
        "MULTI_PROCESS_RENDER",
        "SITE_ISOLATION",
    ]
}
```

**可利用路径**:
```
渲染进程漏洞 → 逃逸到主进程 → 完整系统权限
```

**影响**:
- 完整系统权限
- 恶意代码执行

**修复建议**:
- ✅ 已启用多进程隔离
- ✅ 已启用站点隔离
- ⚠️ 建议启用进程沙箱

### 2. 渲染进程崩溃

**风险等级**: 低

**证据**: `ohos_nweb/src/nweb_hisysevent.cpp`

```cpp
// 崩溃事件上报
void NWebHisysevent::ReportRenderProcessGone(
    const RenderProcessGoneDetail& detail) {
    // 记录崩溃信息
    WVLOG_E("Render process gone: %{public}d, %{public}s",
            detail.reason, detail.stack.c_str());

    // 上报 HiSysEvent
    HiSysEvent::Write(WEBVIEW, "RENDER_PROCESS_GONE",
        HiSysEvent::EventType::FAULT,
        "pid", detail.pid,
        "reason", detail.reason,
        "stack", detail.stack);
}
```

**可利用路径**:
```
恶意网页 → 触发渲染崩溃 → 服务降级
```

**影响**:
- 服务不可用
- 用户体验下降

**修复建议**:
- ✅ 已实现崩溃恢复
- ✅ 已实现崩溃上报
- ⚠️ 建议添加崩溃隔离

### 3. 内存安全

**风险等级**: 高

**证据**: `arkweb_utils/arkweb_utils.h`

```cpp
// 内存安全工具
class SafeString {
public:
    // 防止缓冲区溢出
    static std::string Truncate(const std::string& str, size_t maxLen) {
        return str.length() > maxLen ?
            str.substr(0, maxLen) : str;
    }

    // 防止格式化字符串
    static std::string Sanitize(const std::string& str) {
        std::string result;
        for (char c : str) {
            if (IsSafeChar(c)) {
                result += c;
            }
        }
        return result;
    }
};
```

**可利用路径**:
```
恶意输入 → 缓冲区溢出 → 代码执行
```

**影响**:
- 远程代码执行
- 内存损坏

**修复建议**:
- ✅ 使用安全字符串操作
- ✅ ASan/TSan 支持
- ⚠️ 建议启用 Control Flow Guard

## 安全机制总结

### 已实现安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| **多进程隔离** | ohos_nweb/ | 高 |
| **站点隔离** | Chromium | 高 |
| **AccessToken 校验** | access_token_adapter/ | 高 |
| **URL 校验** | napi_parse_utils.cpp | 高 |
| **参数验证** | 各 N-API | 高 |
| **SELinux** | sa/*.cfg | 中 |
| **崩溃隔离** | nweb_helper.cpp | 中 |
| **安全存储** | webstorage/ | 中 |

### 待改进安全机制

| 机制 | 当前状态 | 建议 |
|------|----------|------|
| **内容安全策略** | 部分支持 | 完善 CSP 头检查 |
| **安全浏览** | 无 | 集成安全浏览 API |
| **隐私保护** | 基础 | 添加隐私模式 |
| **加密存储** | 无 | 实现存储加密 |
| **进程沙箱** | 基础 | 启用严格沙箱 |

## 安全检查清单

### 开发阶段

- [ ] 所有外部输入必须验证
- [ ] 使用安全字符串操作
- [ ] 遵循最小权限原则
- [ ] 启用编译器安全选项
- [ ] 添加安全测试用例

### 代码审查

- [ ] 检查权限调用
- [ ] 检查输入验证
- [ ] 检查敏感数据处理
- [ ] 检查错误处理
- [ ] 检查日志泄露

### 测试阶段

- [ ] 模糊测试输入处理
- [ ] 权限测试
- [ ] 渗透测试
- [ ] 崩溃恢复测试
- [ ] 性能安全测试

## 相关文档

| 主题 | 文档 |
|------|------|
| 架构设计 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_N-API.md](./02_N-API.md) |
| 内部模块 | [03_InnerAPI.md](./03_InnerAPI.md) |
| 构建系统 | [04_Build.md](./04_Build.md) |

---

[返回 SUMMARY.md](./SUMMARY.md) | [上一章: 构建系统](./04_Build.md)
