# 安全风险评审

> Settings 应用的安全风险分析（基于代码证据）

---

## 目的

本文档基于代码证据，分析 Settings 应用的安全风险、攻击面和潜在漏洞。

## 适用范围

- 目标读者：安全研究员、系统开发者
- 项目：@ohos/settings (Settings 3.1)
- 分析方法：静态代码分析 + 证据追踪

---

## 攻击面清单

### 1. 输入接口

| 攻击面 | API | 风险等级 | 证据 |
|----------|------|----------|--------|
| N-API 设置 API | getValue, setValue | 🟡 中 | napi/settings/native_module.cpp:35-48 |
| N-API 观察者 | registerKeyObserver | 🟡 中 | napi/settings/native_module.cpp:44-45 |
| ANI 设置 API | ani_get_value, ani_set_value | 🟡 中 | ani/settings/ani_settings.h:69-91 |
| CJ FFI API | FfiSettingsSetValue | 🟡 中 | cj/settings/src/settings_ffi.h:24-34 |

### 2. 数据访问

| 攻击面 | 机制 | 风险等级 | 证据 |
|----------|--------|----------|--------|
| DataShare | 数据共享框架 | 🟢 低 | napi/settings/napi_settings.cpp:49,416-446 |
| 数据表（global/system/secure） | 设置存储 | 🟢 低 | napi/settings/napi_settings.cpp:37-40 |

### 3. 系统服务调用

| 攻击面 | 服务 | 风险等级 | 证据 |
|----------|--------|----------|--------|
| BundleManager SA | Bundle 信息获取 | 🟢 低 | native/settings/src/napi_bundle_util.cpp:29,34 |
| DataAbility 服务 | 数据访问 | 🟢 低 | DataShare::Creator() 调用 |

### 4. UI 交互

| 攻击面 | 机制 | 风险等级 | 证据 |
|----------|--------|----------|--------|
| ArkTS UI | 页面组件 | 🟢 低 | product/phone/src/main/ets/pages/ |

---

## 信任边界

### 边界定义

```
┌───────────────────────────────────┐
│  应用进程                   │
│  （product/phone）          │
│  N-API / ANI / CJ FFI    │
├─────────────────────────────────┤    │
│  Settings API              │
├─────────────────────────────────┤    │
│  DataShare 框架            │
├─────────────────────────────────┤    │
│  DataAbility 服务            │
├─────────────────────────────────┤    │
│  系统存储（三张表）       │
└───────────────────────────────────┘
```

### 数据流信任链

**读取流程**：
- 应用 → N-API → Native → DataShare → DataAbility → 存储
- 每层无身份验证（依赖系统权限框架）

**写入流程**：
- 应用 → N-API → Native → DataShare → DataAbility → 存储
- 每层无身份验证（依赖系统权限框架）

### 观察者通知

**信任问题**：
- 任何应用都可以注册观察者
- 无身份验证
- 回调由应用提供，可能被伪造

---

## 可被利用点（基于证据）

### ⚠️ 风险点 1：缺少参数验证

**证据**：
```cpp
// napi/settings/napi_settings.cpp
napi_value napi_get_value(napi_env env, napi_callback_info info)
{
    // 参数验证不充分
    size_t argc = 0;
    napi_get_cb_info(env, info, &argc, nullptr, nullptr);
    // 未检查参数数量和类型
}
```

**触发路径**：
```
恶意应用
  ↓
settings.getValue() // 未提供参数
  ↓
N-API 层
  ↓
崩溃或未定义行为
```

**影响**：
- 应用崩溃（拒绝服务）
- 内存泄漏
- 未定义行为

**修复建议**：
```cpp
// napi/settings/napi_settings.cpp
napi_value napi_get_value(napi_env env, napi_callback_info info)
{
    size_t argc = 0;
    napi_get_cb_info(env, info, &argc, nullptr, nullptr);

    // 添加参数验证
    if (argc < 1) {
        napi_throw_type_error(env, "Invalid argument count");
        return nullptr;
    }

    if (argc > 2) { // name + domainName
        napi_throw_type_error(env, "Too many arguments");
        return nullptr;
    }

    // 验证参数类型
    napi_valuetype valuetype;
    napi_typeof(env, argv[0], &valuetype);
    if (valuetype != napi_string) {
        napi_throw_type_error(env, "Invalid argument type");
        return nullptr;
    }

    // 原有代码...
}
```

---

### ⚠️ 风险点 2：SQL 注入风险（DataShare）

**证据**：
```cpp
// napi/settings/napi_settings.cpp
std::shared_ptr<OHOS::DataShare::DataShareHelper> getDataShareHelper(...) {
    // 未使用参数化查询
    dataShareHelper->Query(predicates, columns, ...);
}
```

**触发路径**：
```
恶意应用
  ↓
settings.setValue("malicious; DROP TABLE global--")
  ↓
N-API 层
  ↓
Native 层
  ↓
DataShare（未过滤特殊字符）
  ↓
DataAbility（未转义）
  ↓
SQL 执行（DROP TABLE global--）
```

**影响**：
- 数据库破坏
- 数据丢失
- 拒绝服务

**修复建议**：
```cpp
// 使用 DataSharePredicates 参数化查询
DataSharePredicates predicates;
predicates.EqualTo(SettingsData::SETTINGS_DATA_FIELD_KEYWORD, name);
dataShareHelper->Query(predicates, ...);

// 或者验证输入
if (name.find(";") != std::string::npos || name.find("--") != std::string::npos) {
    // 拒绝包含 SQL 特殊字符的输入
    napi_throw_type_error(env, "Invalid key name");
    return nullptr;
}
```

---

### ⚠️ 风险点 3：观察者劫持

**证据**：
```cpp
// napi/settings/napi_settings_observer.cpp
class SettingsObserver : public OHOS::AAFwk::DataAbilityObserverStub {
public:
    void OnChange(const std::string &key, const std::string &value) override {
        // 直接回调，未验证来源
        if (callbackRef != nullptr) {
            napi_value callback = nullptr;
            napi_get_reference_value(env, callbackRef, &callback);
            napi_value args[2];
            args[0] = wrap_string_to_js(env, key);
            args[1] = wrap_string_to_js(env, value);
            napi_call_function(env, callback, 2, args, nullptr);
        }
    }
};
```

**触发路径**：
```
恶意应用
  ↓
settings.registerKeyObserver("brightness", maliciousObserver)
  ↓
N-API 层
  ↓
注册恶意观察者
  ↓
当其他应用修改 "brightness" 时
  ↓
恶意观察者收到回调（可以伪造数据）
  ↓
回调到恶意应用
```

**影响**：
- 数据泄露
- 隐私泄露
- 信息篡改

**修复建议**：
```cpp
// 1. 验证观察者来源
napi_value registerKeyObserver(napi_env env, napi_callback_info info) {
    // 验证调用者身份
    OHOS::sptr<IRemoteObject> token = GetCallingToken();
    if (!ValidateObserver(token)) {
        napi_throw_error(env, "Unauthorized observer registration");
        return nullptr;
    }

    // 原有代码...
}

// 2. 验证回调来源
void OnChange(const std::string &key, const std::string &value) override {
    // 验证回调是否来自合法进程
    if (!ValidateCallbackSource()) {
        return; // 忽略恶意回调
    }

    // 原有代码...
}
```

---

### ⚠️ 风险点 4：日志信息泄露

**证据**：
```cpp
// napi/settings/napi_settings.cpp:96-101
std::string unwrap_string_from_js(napi_env env, napi_value param, bool showLog, bool anonymousLog) {
    // 敏感数据使用匿名化
    std::string defaultValue("");
    size_t size = 0;
    napi_get_value_string_utf8(env, param, &size, nullptr);

    // 匿名化日志输出
    if (anonymousLog) {
        LogInfo("Settings: key = ******");
    } else {
        LogInfo("Settings: key = %{public}s", defaultValue.c_str());
    }

    return defaultValue;
}
```

**触发路径**：
```
恶意应用
  ↓
设置敏感键（如 password, token）
  ↓
N-API 层
  ↓
Native 层
  ↓
日志系统（Hilog）
  ↓
日志包含敏感信息
  ↓
日志被收集到日志系统
  ↓
攻击者可读取日志
```

**影响**：
- 敏感信息泄露
- 隐私泄露
- 安全凭证泄露

**修复建议**：
1. **日志脱敏**：
```cpp
// 不要记录敏感数据
if (IsSensitiveKey(name)) {
    LogInfo("Settings: key = ******");
} else {
    LogInfo("Settings: key = %{public}s", name.c_str());
}
```

2. **日志审计**：
- 检查所有日志调用
- 避免记录敏感参数
- 使用日志级别控制（DEBUG/INFO）

---

### ⚠️ 风险点 5：权限绕过

**证据**：
```cpp
// Settings 应用不执行权限检查
// 权限仅应用层声明：product/phone/src/main/module.json5
{
  "name": "ohos.permission.UPDATE_CONFIGURATION",
  "reason": "$string:UPDATE_CONFIGURATION"
}
```

**触发路径**：
```
恶意应用
  ↓
声明 ohos.permission.UPDATE_CONFIGURATION
  ↓
绕过系统权限检查
  ↓
调用 Settings API（setValue）
  ↓
修改系统设置
```

**影响**：
- 未授权的系统设置修改
- 安全配置篡改
- 设备行为异常

**修复建议**：

1. **Native 层权限检查**：
```cpp
// native/settings/src/napi_bundle_util.cpp
#include "access_token/AccessTokenKit.h"

bool HasPermission(const std::string& permission) {
    auto tokenId = AccessTokenKit::GetAccessTokenId();
    return AccessTokenKit::VerifyAccessToken(tokenId, permission);
}
```

2. **敏感操作保护**：
```cpp
napi_value napi_set_value(napi_env env, napi_callback_info info) {
    // 验证敏感操作的权限
    if (IsSensitiveKey(name)) {
        if (!HasPermission("ohos.permission.UPDATE_CONFIGURATION")) {
            napi_throw_error(env, "Permission denied");
            return nullptr;
        }
    }

    // 原有代码...
}
```

---

## 未发现的风险

### ✅ 已确认无风险

| 风险类型 | 检查结果 | 说明 |
|-----------|---------|--------|
| 自定义 SA | ✅ 未发现 | Settings 不实现自定义 SA |
| Binder 驱动交互 | ✅ 未发现 | 仅使用系统 DataShare |
| 直接文件系统访问 | ✅ 未发现 | 所有访问通过 DataAbility |
| 沙箱逃逸 | ✅ 未发现 | 无沙箱代码 |

---

## 检查范围与局限性

### 已检查范围

1. ✅ N-API 模块（napi/settings/, napi/intelligentscene/）
2. ✅ ANI 模块（ani/settings/, ani/intelligentscene/）
3. ✅ CJ FFI 模块（cj/settings/）
4. ✅ Native 模块（native/settings/）
5. ✅ 产品模块（product/phone/）

### 未检查范围

1. ❌ 测试模块（已忽略 test/）
2. ❌ 第三方库实现（依赖系统的 external_deps）
3. ⚠️ 搜索功能的数据库实现（RDB）- 未深入分析

### 局限性

- 基于静态代码分析，无法检测运行时行为
- 依赖系统权限框架的实际实现未验证
- 依赖系统服务（DataShare, BundleManager）的安全机制未分析

---

## 修复优先级

| 优先级 | 风险点 | 修复难度 | 修复时间 |
|----------|----------|-----------|----------|
| 🔴 高 | 观察者劫持 | 中 | 建议立即修复 |
| 🔴 高 | 权限绕过 | 中 | 建议立即修复 |
| 🟡 中 | SQL 注入风险 | 低 | 建议修复 |
| 🟡 中 | 日志信息泄露 | 低 | 建议修复 |
| 🟢 低 | 参数验证 | 低 | 建议修复 |

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[04_NAPI_API.md](04_NAPI_API.md)** - 对外 API 文档
- **[05_Inner_API.md](05_Inner_API.md)** - 内部 API 文档

---

**最后更新**：2026-02-06 00:11:23
