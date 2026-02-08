# 攻击面分析

> UI Appearance 子系统外部输入、敏感操作与信任边界

## 文档目的

本文档为安全研究员提供 UI Appearance 子系统的完整攻击面映射，包括所有外部输入入口点、敏感操作、信任边界和潜在利用路径。

**适用受众**: 安全研究员、安全审计人员
**阅读时间**: 约 15-20 分钟

---

## 攻击面总览

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            UI Appearance 攻击面                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  外部输入入口点 (Attack Surface Entry Points)              │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  1. N-API 接口 (6 个 set 方法)                          │   │
│  │  2. ANI 接口 (6 个 set 方法)                          │   │
│  │  3. IPC 通信 (IUiAppearanceAbility 接口)                 │   │
│  │  4. 系统参数读取 (persist.* keys)                      │   │
│  │  5. 公共事件订阅 (CommonEvent 接收)                     │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  敏感操作 (Sensitive Operations)                          │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  1. 系统参数写入 (SetParameterWrap)                       │   │
│  │  2. AMS 通信 (UpdateConfiguration)                        │   │
│  │  3. 权限校验 (VerifyAccessToken)                         │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. 外部输入入口点

### 1.1 N-API 接口 (JS/TS 应用层)

**位置**: `interfaces/kits/napi/src/js_ui_appearance.cpp`

| N-API 方法 | 参数类型 | 参数范围 | 异步 | 证据位置 |
|----------|----------|----------|------|----------|
| **setDarkMode()** | DarkMode (number) | 0, 1, 2+ | ✅ | Line 289 |
| **getDarkMode()** | 无 | - | ❌ | Line 334 |
| **setFontScale()** | Number (double) | 0.0 ~ 5.0 | ✅ | Line 381 |
| **getFontScale()** | 无 | - | ❌ | Line 357 |
| **setFontWeightScale()** | Number (double) | 0.0 ~ 5.0 | ✅ | Line 449 |
| **getFontWeightScale()** | 无 | - | ❌ | Line 425 |

#### 关键代码证据

**setDarkMode 入口**:
```cpp
// interfaces/kits/napi/src/js_ui_appearance.cpp:289
static napi_value JSSetDarkMode(napi_env env, napi_callback_info info)
{
    size_t argc = ARGC_WITH_TWO;
    napi_value argv[ARGC_WITH_TWO] = { 0 };
    napi_status napiStatus = napi_ok;
    napi_value result = nullptr;
    napi_get_undefined(env, &result);

    // 直接从 JS 获取参数，无额外校验
    napiStatus = napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    // ...
}
```

**输入验证位置**:
```cpp
// interfaces/kits/napi/src/js_ui_appearance.cpp:99
// FontScale 范围校验
if (asyncContext->jsFontScale <= MIN_FONT_SCALE || asyncContext->jsFontScale > MAX_FONT_SCALE) {
    resCode = UiAppearanceAbilityErrCode::INVALID_ARG;
}
```

---

### 1.2 ANI 接口 (ArkTS Native Interface)

**位置**: `interfaces/ets/ani/src/ui_appearance.cpp`

| ANI 方法 | 参数类型 | 参数范围 | 异步 | 证据位置 |
|---------|----------|----------|------|----------|
| **setDarkMode()** | ani_double (number) | 0, 1, 2+ | ✅ | Line 203 |
| **getDarkMode()** | 无 | - | ❌ | Line 242 |
| **setFontScale()** | ani_double (number) | 0.0 ~ 5.0 | ✅ | Line 314 |
| **getFontScale()** | 无 | - | ❌ | Line 298 |
| **setFontWeightScale()** | ani_double (number) | 0.0 ~ 5.0 | ✅ | Line 373 |
| **getFontWeightScale()** | 无 | - | ❌ | Line 357 |

#### 关键代码证据

```cpp
// interfaces/ets/ani/src/ui_appearance.cpp:203
ani_object SetFontScale([[maybe_unused]] ani_env* env, ani_double fontScale)
{
    // ANI 层缺少 CheckCallerIsSystemApp() 检查
    if (asyncContext->ani_FontScale <= MIN_FONT_SCALE ||
        asyncContext->ani_FontScale > MAX_FONT_SCALE) {
        resCode = UiAppearanceAbilityErrCode::INVALID_ARG;
    }
}
```

---

### 1.3 IPC 接口 (System Ability)

**位置**: `services/IUiAppearanceAbility.idl`

| IPC 方法 | IDL 签名 | 参数 | 返回 | 证据位置 |
|---------|----------|------|------|----------|
| **SetDarkMode()** | `int SetDarkMode([in] int darkMode)` | int | Line 17 |
| **GetDarkMode()** | `int GetDarkMode()` | int | Line 18 |
| **SetFontScale()** | `int SetFontScale([in] String fontScale)` | int | Line 20 |
| **GetFontScale()** | `int GetFontScale([out] String fontScale)` | int | Line 19 |
| **SetFontWeightScale()** | `int SetFontWeightScale([in] String fontWeightScale)` | int | Line 22 |
| **GetFontWeightScale()** | `int GetFontWeightScale([out] String fontWeightScale)` | int | Line 21 |

#### IDL 接口定义

```cpp
// services/IUiAppearanceAbility.idl:16-23
interface OHOS.ArkUi.UiAppearance.IUiAppearanceAbility {
    int SetDarkMode([in] int darkMode);
    int GetDarkMode();
    int GetFontScale([out] String fontScale);
    int SetFontScale([in] String fontScale);
    int GetFontWeightScale([out] String fontWeightScale);
    int SetFontWeightScale([in] String fontWeightScale);
}
```

---

### 1.4 系统参数输入

**位置**: `services/utils/src/parameter_wrap.cpp`

| 参数键 | 类型 | 访问 | 证据位置 |
|-------|------|------|----------|
| `persist.ace.darkmode.{userId}` | String | 读/写 | Line 20-47 |
| `persist.sys.font_scale_for_user.{userId}` | String | 读/写 | Line 20-47 |
| `persist.sys.font_wght_scale_for_user.{userId}` | String | 读/写 | Line 20-47 |
| `persist.uiAppearance.first_initialization` | String | 读/写 | Line 20-47 |

#### 参数访问 API

```cpp
// services/utils/src/parameter_wrap.cpp:37
bool SetParameterWrap(const std::string& name, const std::string& value)
{
    // 直接调用系统参数 API，无额外校验
    return SetParameter(name.c_str(), value.c_str());
}
```

---

### 1.5 公共事件输入

**位置**: `services/src/ui_appearance_ability.cpp`

| 事件名称 | 用途 | 证据位置 |
|---------|------|----------|
| **COMMON_EVENT_USER_SWITCHED** | 用户切换 | Line 329 |
| **COMMON_EVENT_TIME_CHANGED** | 系统时间变化 | Line 331 |
| **COMMON_EVENT_TIMEZONE_CHANGED** | 时区变化 | Line 332 |
| **COMMON_EVENT_BOOT_COMPLETED** | 系统启动完成 | Line 329 |
| **COMMON_EVENT_SCREEN_OFF** | 屏幕关闭 | Line 333 |
| **COMMON_EVENT_SCREEN_ON** | 屏幕打开 | Line 334 |

#### 事件订阅代码

```cpp
// services/src/ui_appearance_ability.cpp:326
void UiAppearanceAbility::SubscribeCommonEvent()
{
    EventFwk::MatchingSkills matchingSkills;
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_BOOT_COMPLETED);
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_USER_SWITCHED);
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_TIME_CHANGED);
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_TIMEZONE_CHANGED);
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_SCREEN_ON);
    matchingSkills.AddEvent(EventFwk::CommonEventSupport::COMMON_EVENT_SCREEN_OFF);
    // 无事件数据校验
    EventFwk::CommonEventManager::SubscribeCommonEvent(uiAppearanceEventSubscriber_);
}
```

---

## 2. 敏感操作清单

### 2.1 系统参数写入

**操作**: 持久化系统参数
**证据位置**: `services/utils/src/parameter_wrap.cpp:37`

```cpp
bool SetParameterWrap(const std::string& name, const std::string& value)
{
    // 直接写入系统参数，可被任何有权限的进程修改
    return SetParameter(name.c_str(), value.c_str());
}
```

**风险**:
- 参数值未验证合法性
- 可能覆盖关键系统配置

### 2.2 AMS 通信

**操作**: 更新应用配置
**证据位置**: `services/src/ui_appearance_ability.cpp:119-140`

```cpp
sptr<AppExecFwk::IAppMgr> UiAppearanceAbility::GetAppManagerInstance()
{
    sptr<ISystemAbilityManager> systemAbilityManager =
        SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    sptr<IRemoteObject> appObject = systemAbilityManager->GetSystemAbility(APP_MGR_SERVICE_ID);
    sptr<AppExecFwk::IAppMgr> systemAbility = iface_cast<AppExecFwk::IAppMgr>(appObject);
    return systemAbility;
}
```

**调用链**: `UpdateConfiguration() → AMS → WMS → AceEngine`
**风险**: 通过 IPC 向 AMS 发送不可信的配置对象

### 2.3 权限校验操作

**操作**: 验证调用者权限
**证据位置**: `services/src/ui_appearance_ability.cpp:142-151`

```cpp
bool UiAppearanceAbility::VerifyAccessToken(const std::string& permissionName)
{
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return true;
    }
    LOGE("permission %{private}s denied, callerToken : %{public}u", permissionName.c_str(), callerToken);
    return false;
}
```

**风险**: Token ID 篡改、权限绕过

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            信任边界 (Trust Boundary)                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  不可信区域 (Untrusted Zone)                                     │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  • JS/TS 应用 (任何第三方应用)                           │   │
│  │  • N-API 参数 (来自应用层)                                     │   │
│  │  • ANI 参数 (来自 ArkTS 层)                                    │   │
│  │  • IPC 调用者 (外部进程)                                   │   │
│  │  • 公共事件 (来自 EventFwk)                                   │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│                              ══════════════════════════════════════════════════   │
│                              权限校验边界 (Permission Check Boundary)          │
│                              ══════════════════════════════════════════════   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  半可信区域 (Semi-trusted Zone)                               │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  • UiAppearanceAbilityClient (IPC Proxy)                     │   │
│  │  • UiAppearanceAbility (SA 实现，已校验权限)                │   │
│  │  • DarkModeManager (受 SA 控制)                             │   │
│  │  • SmartGestureManager (受 SA 控制)                             │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│                              ════════════════════════════════════════════════════   │
│                              系统服务边界 (System Service Boundary)                 │
│                              ════════════════════════════════════════════════   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  可信区域 (Trusted Zone)                                          │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  • AMS (Ability Manager Service)                               │   │
│  │  • WMS (Window Manager Service)                               │   │
│  │  • ResourceManager (系统资源管理)                            │   │
│  │  • System Parameter Service (系统参数服务)                     │   │
│  │  • OsAccountManager (系统账户服务)                             │   │
│  │  • CommonEventService (系统事件服务)                            │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 边界穿越点

| 边界 | 穿越位置 | 校验机制 | 证据 |
|------|----------|----------|------|
| **应用层 → N-API** | JSSetDarkMode | 参数类型 + 范围 | `js_ui_appearance.cpp:204` |
| **N-API → IPC** | UiAppearanceAbilityClient | 系统应用检查 (仅 N-API) | `js_ui_appearance.cpp:280` |
| **IPC → SA** | VerifyAccessToken | AccessToken 校验 | `ui_appearance_ability.cpp:142` |
| **SA → AMS** | UpdateConfiguration | 系统内部调用 (已受控) | `ui_appearance_ability.cpp:455` |
| **SA → 参数系统** | SetParameterWrap | DAC 权限 | `ui_appearance.para.dac` |

---

## 4. 攻击向量分析

### 4.1 参数注入攻击

**攻击点**: N-API `setDarkMode()` / `setFontScale()` / `setFontWeightScale()`
**触发路径**:
```
恶意应用 → JSSetDarkMode(2) → OnExecute() → SetDarkMode() → UpdateConfiguration()
```

**风险分析**:
- **DarkMode 值 2+**: 转换为 `DarkMode::UNKNOWN`，传入服务层
- **fontScale 字符串注入**: 服务层接受字符串，未校验格式

**影响**: 可能导致系统配置不一致、逻辑漏洞

---

### 4.2 权限绕过攻击

**攻击点**: ANI 层缺少 SystemApp 检查
**触发路径**:
```
ArkTS 应用 → ANI::setDarkMode() → IPC 调用 (无 SystemApp 检查)
```

**证据**:
```cpp
// interfaces/ets/ani/src/ui_appearance.cpp - ANI 层无 SystemApp 检查
ani_object SetFontScale([[maybe_unused]] ani_env* env, ani_double fontScale)
{
    // 直接调用 IPC，无 CheckCallerIsSystemApp()
    // ...
}

// 对比 N-API 层有检查
// interfaces/kits/napi/src/js_ui_appearance.cpp:280
bool JsUiAppearance::CheckCallerIsSystemApp()
{
    auto selfToken = IPCSkeleton::GetSelfTokenID();
    if (!Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(selfToken)) {
        return false;
    }
    return true;
}
```

**影响**: 非系统应用可通过 ANI 绕过 N-API 的额外保护

---

### 4.3 测试 Mock 风险

**攻击点**: 测试代码错误链接到生产环境
**证据位置**: `test/mock/mock_accesstoken_kit.cpp`

```cpp
// test/mock/mock_accesstoken_kit.cpp - 永远返回允许
int AccessTokenKit::VerifyAccessToken(AccessTokenID tokenID, const std::string& permissionName)
{
    return PERMISSION_GRANTED;  // 永远返回允许！
}
```

**影响**: 如果测试 Mock 错误链接到生产环境，所有权限检查都将失效

---

### 4.4 竞态条件攻击

**攻击点**: `GetCallingUserId()` 用户 ID 计算
**证据位置**: `services/src/ui_appearance_ability.cpp:391-406`

```cpp
int32_t UiAppearanceAbility::GetCallingUserId()
{
    const static int32_t UID_TRANSFORM_DIVISOR = 200000;
    int32_t userId = OHOS::IPCSkeleton::GetCallingUid() / UID_TRANSFORM_DIVISOR;
    if (userId == 0) {
        // 如果 userId=0 时，重新查询前台账户，存在 TOCTOU 窗口
        auto errNo = AccountSA::OsAccountManager::GetForegroundOsAccountLocalId(userId);
        // ...
    }
    return userId;
}
```

**影响**: 用户切换期间可能获取错误的用户 ID

---

## 5. 快速索引

### 按攻击面查找

| 攻击类型 | 相关文档 | 关键位置 |
|----------|----------|----------|
| **参数注入** | 06_SecurityReview.md | N-API/ANI 输入点 |
| **权限绕过** | 06_SecurityReview.md | VerifyAccessToken, CheckCallerIsSystemApp |
| **竞态条件** | 06_SecurityReview.md | GetCallingUserId |
| **测试 Mock** | 06_SecurityReview.md | mock_accesstoken_kit.cpp |

### 按代码位置查找

| 文件 | 攻击面 | 说明 |
|------|--------|------|
| `js_ui_appearance.cpp` | N-API 参数注入 | 6 个 set 方法的输入点 |
| `ui_appearance.cpp` | ANI 参数注入 | 6 个 ANI 方法的输入点 |
| `ui_appearance_ability.cpp` | 权限校验, IPC 调用 | VerifyAccessToken, GetCallingUserId |
| `parameter_wrap.cpp` | 系统参数注入 | SetParameterWrap 调用点 |
| `ui_appearance_ability.h` | 信任边界 | usersParam_ 的 mutex 保护 |

---

## 6. 证据库索引

| 攻击点 | 文件路径 | 行号 | 符号 |
|--------|----------|------|------|
| **N-API setDarkMode** | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 289 | JSSetDarkMode |
| **N-API CheckCallerIsSystemApp** | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 280-287 | CheckCallerIsSystemApp |
| **ANI setDarkMode (无 SystemApp 检查)** | `interfaces/ets/ani/src/ui_appearance.cpp` | 203-226 | SetFontScale, SetFontWeightScale |
| **SA VerifyAccessToken** | `services/src/ui_appearance_ability.cpp` | 142-151 | VerifyAccessToken |
| **SA UpdateConfiguration** | `services/src/ui_appearance_ability.cpp` | 455-501 | UpdateConfiguration |
| **SA GetCallingUserId** | `services/src/ui_appearance_ability.cpp` | 391-406 | GetCallingUserId |
| **参数持久化** | `services/utils/src/parameter_wrap.cpp` | 37 | SetParameterWrap |
| **测试 Mock** | `test/mock/mock_accesstoken_kit.cpp` | - | VerifyAccessToken (永远允许) |

---

**相关文档**:
- [架构设计](02_Architecture.md) - 理解信任边界和数据流
- [安全评审](06_Security.md) - 详细风险评估和修复建议
- [N-API 接口](03_NAPI.md) - API 详细文档
