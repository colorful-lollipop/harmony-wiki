# N-API 接口文档

> UI Appearance N-API 接口详细说明

## 接口概览

### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | uiAppearance |
| **模块路径** | @ohos.uiAppearance |
| **注册方式** | napi_module_register |
| **注册函数** | UiAppearanceExports |
| **头文件** | `interfaces/kits/napi/include/js_ui_appearance.h` |
| **实现文件** | `interfaces/kits/napi/src/js_ui_appearance.cpp` |

### 导出接口列表

| 接口 | 类型 | 同步/异步 | 权限要求 |
|------|------|----------|----------|
| setDarkMode | 函数 | 异步 (Promise/Callback) | UPDATE_CONFIGURATION |
| getDarkMode | 函数 | 同步 | 无 |
| setFontScale | 函数 | 异步 (Promise/Callback) | UPDATE_CONFIGURATION |
| getFontScale | 函数 | 同步 | 无 |
| setFontWeightScale | 函数 | 异步 (Promise/Callback) | UPDATE_CONFIGURATION |
| getFontWeightScale | 函数 | 同步 | 无 |
| DarkMode | 属性 | - | 常量枚举 |

## DarkMode 枚举

```typescript
enum DarkMode {
  // 始终深色模式
  ALWAYS_DARK = 0,
  // 始终浅色模式
  ALWAYS_LIGHT = 1,
}
```

**代码位置**: `interfaces/kits/napi/src/js_ui_appearance.cpp:496-504`

```cpp
napi_value DarkMode = nullptr;
napi_value alwaysDark = nullptr;
napi_value alwaysLight = nullptr;
NAPI_CALL(env, napi_create_int32(env, 0, &alwaysDark));
NAPI_CALL(env, napi_create_int32(env, 1, &alwaysLight));
NAPI_CALL(env, napi_create_object(env, &DarkMode));
NAPI_CALL(env, napi_set_named_property(env, DarkMode, "ALWAYS_DARK", alwaysDark));
NAPI_CALL(env, napi_set_named_property(env, DarkMode, "ALWAYS_LIGHT", alwaysLight));
```

---

## 接口详细说明

### setDarkMode

设置系统深色模式。

#### 函数签名

```typescript
function setDarkMode(mode: DarkMode, callback?: AsyncCallback<void>): void
function setDarkMode(mode: DarkMode): Promise<void>
```

#### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| mode | DarkMode | 是 | 深色模式 (ALWAYS_DARK: 0, ALWAYS_LIGHT: 1) |
| callback | AsyncCallback\<void\> | 否 | 异步回调，不传则返回 Promise |

#### 返回值

| 类型 | 说明 |
|------|------|
| void | 异步操作，无直接返回值 |

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 201 | PERMISSION_ERR - 权限不足 |
| 202 | NOT_SYSTEM_APP - 非系统应用 |
| 401 | INVALID_ARG - 参数错误 |
| 500001 | SYS_ERR - 系统错误 |

#### 使用示例

```typescript
import uiAppearance from '@ohos.uiAppearance';

// Promise 方式
uiAppearance.setDarkMode(uiAppearance.DarkMode.ALWAYS_DARK)
  .then(() => {
    console.info('深色模式设置成功');
  })
  .catch((error) => {
    console.error('深色模式设置失败: ' + JSON.stringify(error));
  });

// Callback 方式
uiAppearance.setDarkMode(uiAppearance.DarkMode.ALWAYS_LIGHT, (err) => {
  if (err) {
    console.error('深色模式设置失败: ' + JSON.stringify(err));
  } else {
    console.info('深色模式设置成功');
  }
});
```

#### 实现分析

**C++ 实现**: `interfaces/kits/napi/src/js_ui_appearance.cpp:289-332`

```cpp
static napi_value JSSetDarkMode(napi_env env, napi_callback_info info)
{
    // 1. 获取参数
    size_t argc = ARGC_WITH_TWO;
    napi_value argv[ARGC_WITH_TWO] = { 0 };
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    
    // 2. 参数校验
    JsUiAppearance::CheckArgs(env, argc, argv);
    
    // 3. 创建异步上下文
    auto asyncContext = new (std::nothrow) AsyncContext();
    napi_get_value_int32(env, argv[0], &asyncContext->jsSetArg);
    asyncContext->mode = JsUiAppearance::ConvertJsDarkMode2Enum(asyncContext->jsSetArg);
    
    // 4. 创建 Promise 或 Callback
    if (argc == ARGC_WITH_TWO) {
        napi_create_reference(env, argv[1], 1, &asyncContext->callbackRef);
    } else {
        napi_create_promise(env, &asyncContext->deferred, &result);
    }
    
    // 5. 创建异步工作
    napi_create_async_work(env, nullptr, resource, 
        JsUiAppearance::OnExecute,  // 执行 IPC 调用
        JsUiAppearance::OnComplete, // 完成回调
        reinterpret_cast<void*>(asyncContext), &asyncContext->work);
    napi_queue_async_work(env, asyncContext->work);
}
```

**参数校验**: `interfaces/kits/napi/src/js_ui_appearance.cpp:204-234`

```cpp
napi_status JsUiAppearance::CheckArgs(napi_env env, size_t argc, napi_value* argv)
{
    if (argc != ARGC_WITH_ONE && argc != ARGC_WITH_TWO) {
        NapiThrow(env, "the number of parameters can only be 1 or 2.", 
            UiAppearanceAbilityErrCode::INVALID_ARG);
        return napi_invalid_arg;
    }
    
    napi_valuetype valueType = napi_undefined;
    switch (argc) {
        case ARGC_WITH_TWO:
            napi_typeof(env, argv[1], &valueType);
            if (valueType != napi_function) {
                NapiThrow(env, "the second parameter must be a function.",
                    UiAppearanceAbilityErrCode::INVALID_ARG);
                return napi_invalid_arg;
            }
            [[fallthrough]];
        case ARGC_WITH_ONE:
            napi_typeof(env, argv[0], &valueType);
            if (valueType != napi_number) {
                NapiThrow(env, "the first parameter must be DarkMode.", 
                    UiAppearanceAbilityErrCode::INVALID_ARG);
                return napi_invalid_arg;
            }
    }
    return napi_ok;
}
```

---

### getDarkMode

获取当前系统深色模式。

#### 函数签名

```typescript
function getDarkMode(): DarkMode
```

#### 参数说明

无参数。

#### 返回值

| 类型 | 说明 |
|------|------|
| DarkMode | 当前深色模式 (ALWAYS_DARK: 0, ALWAYS_LIGHT: 1) |

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 500001 | SYS_ERR - 系统错误 |

#### 使用示例

```typescript
import uiAppearance from '@ohos.uiAppearance';

const currentMode = uiAppearance.getDarkMode();
if (currentMode === uiAppearance.DarkMode.ALWAYS_DARK) {
    console.info('当前为深色模式');
} else {
    console.info('当前为浅色模式');
}
```

#### 实现分析

**C++ 实现**: `interfaces/kits/napi/src/js_ui_appearance.cpp:334-355`

```cpp
static napi_value JSGetDarkMode(napi_env env, napi_callback_info info)
{
    napi_value result = nullptr;
    napi_get_undefined(env, &result);
    size_t argc = 0;
    
    // 检查无参数
    NAPI_CALL(env, napi_get_cb_info(env, info, &argc, nullptr, nullptr, nullptr));
    if (argc != 0) {
        NapiThrow(env, "requires no parameter.", UiAppearanceAbilityErrCode::INVALID_ARG);
        return result;
    }
    
    // IPC 调用获取
    auto mode = UiAppearanceAbilityClient::GetInstance()->GetDarkMode();
    
    // 错误处理
    if (mode == UiAppearanceAbilityErrCode::SYS_ERR) {
        NapiThrow(env, "get dark-mode failed.", UiAppearanceAbilityErrCode::SYS_ERR);
        return result;
    }
    
    // 返回结果
    NAPI_CALL(env, napi_create_int32(env, mode, &result));
    return result;
}
```

---

### setFontScale

设置系统字体缩放比例。

#### 函数签名

```typescript
function setFontScale(fontScale: number, callback?: AsyncCallback<void>): void
function setFontScale(fontScale: number): Promise<void>
```

#### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| fontScale | number | 是 | 字体缩放比例，范围 0 ~ 5，精度 0.01 |
| callback | AsyncCallback\<void\> | 否 | 异步回调，不传则返回 Promise |

#### 返回值

| 类型 | 说明 |
|------|------|
| void | 异步操作，无直接返回值 |

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 201 | PERMISSION_ERR - 权限不足 |
| 202 | NOT_SYSTEM_APP - 非系统应用 |
| 401 | INVALID_ARG - 参数错误（fontScale 不在 0~5 范围内） |
| 500001 | SYS_ERR - 系统错误 |

#### 使用示例

```typescript
import uiAppearance from '@ohos.uiAppearance';

// 设置字体缩放为 1.5 倍
uiAppearance.setFontScale(1.5)
  .then(() => {
    console.info('字体缩放设置成功');
  })
  .catch((error) => {
    console.error('字体缩放设置失败: ' + JSON.stringify(error));
  });
```

#### 实现分析

**C++ 实现**: `interfaces/kits/napi/src/js_ui_appearance.cpp:381-423`

```cpp
static napi_value JSSetFontScale(napi_env env, napi_callback_info info)
{
    // ... 参数获取 ...
    
    // 参数校验
    napiStatus = JsUiAppearance::CheckFontScaleArgs(env, argc, argv);
    
    // 创建异步上下文
    auto asyncContext = new (std::nothrow) AsyncContext();
    napi_get_value_double(env, argv[0], &asyncContext->jsFontScale);
    asyncContext->fontScale = std::to_string(asyncContext->jsFontScale);
    
    // 创建异步工作
    napi_create_async_work(env, nullptr, resource, 
        JsUiAppearance::OnSetFontScale,  // 执行
        JsUiAppearance::OnComplete,     // 完成
        reinterpret_cast<void*>(asyncContext), &asyncContext->work);
    napi_queue_async_work(env, asyncContext->work);
}
```

**范围校验** (`interfaces/kits/napi/src/js_ui_appearance.cpp:99`):

```cpp
} else if (asyncContext->jsFontScale <= MIN_FONT_SCALE || 
           asyncContext->jsFontScale > MAX_FONT_SCALE) {
    resCode = UiAppearanceAbilityErrCode::INVALID_ARG;
}
```

> MIN_FONT_SCALE = 0, MAX_FONT_SCALE = 5

---

### getFontScale

获取当前系统字体缩放比例。

#### 函数签名

```typescript
function getFontScale(): number
```

#### 参数说明

无参数。

#### 返回值

| 类型 | 说明 |
|------|------|
| number | 当前字体缩放比例 |

#### 使用示例

```typescript
import uiAppearance from '@ohos.uiAppearance';

const fontScale = uiAppearance.getFontScale();
console.info('当前字体缩放比例: ' + fontScale);
```

---

### setFontWeightScale

设置系统字重缩放比例。

#### 函数签名

```typescript
function setFontWeightScale(fontWeightScale: number, callback?: AsyncCallback<void>): void
function setFontWeightScale(fontWeightScale: number): Promise<void>
```

#### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| fontWeightScale | number | 是 | 字重缩放比例，范围 0 ~ 5，精度 0.01 |
| callback | AsyncCallback\<void\> | 否 | 异步回调，不传则返回 Promise |

#### 返回值

| 类型 | 说明 |
|------|------|
| void | 异步操作，无直接返回值 |

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 201 | PERMISSION_ERR - 权限不足 |
| 202 | NOT_SYSTEM_APP - 非系统应用 |
| 401 | INVALID_ARG - 参数错误 |
| 500001 | SYS_ERR - 系统错误 |

#### 使用示例

```typescript
import uiAppearance from '@ohos.uiAppearance';

uiAppearance.setFontWeightScale(1.2)
  .then(() => {
    console.info('字重缩放设置成功');
  })
  .catch((error) => {
    console.error('字重缩放设置失败: ' + JSON.stringify(error));
  });
```

---

### getFontWeightScale

获取当前系统字重缩放比例。

#### 函数签名

```typescript
function getFontWeightScale(): number
```

#### 参数说明

无参数。

#### 返回值

| 类型 | 说明 |
|------|------|
| number | 当前字重缩放比例 |

---

## 权限说明

### 权限要求

| API | 权限 | 说明 |
|-----|------|------|
| setDarkMode | ohos.permission.UPDATE_CONFIGURATION | 必须具有系统配置更新权限 |
| getDarkMode | 无 | 任何应用可调用 |
| setFontScale | ohos.permission.UPDATE_CONFIGURATION | 必须具有系统配置更新权限 |
| getFontScale | 无 | 任何应用可调用 |
| setFontWeightScale | ohos.permission.UPDATE_CONFIGURATION | 必须具有系统配置更新权限 |
| getFontWeightScale | 无 | 任何应用可调用 |

### 权限检查位置

**N-API 层** (`interfaces/kits/napi/src/js_ui_appearance.cpp:280-287`):

```cpp
bool JsUiAppearance::CheckCallerIsSystemApp()
{
    auto selfToken = IPCSkeleton::GetSelfTokenID();
    if (!Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(selfToken)) {
        return false;
    }
    return true;
}
```

**SA 层** (`services/src/ui_appearance_ability.cpp:142-151`):

```cpp
bool UiAppearanceAbility::VerifyAccessToken(const std::string& permissionName)
{
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, permissionName);
    if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return true;
    }
    return false;
}
```

---

## 错误码参考

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | SUCCEEDED | 成功 |
| 201 | PERMISSION_ERR | 权限不足 |
| 202 | NOT_SYSTEM_APP | 非系统应用 |
| 401 | INVALID_ARG | 参数错误 |
| 500001 | SYS_ERR | 系统内部错误 |

**代码位置**: `services/include/ui_appearance_types.h:28-34`

---

## 调用链总结

```
JS 调用
    ↓
N-API (js_ui_appearance.cpp)
    ├── 参数解析 (napi_get_cb_info)
    ├── 参数校验 (CheckArgs/CheckFontScaleArgs)
    ├── 系统应用检查 (CheckCallerIsSystemApp)
    ├── 异步工作创建 (napi_create_async_work)
    └── IPC 调用 (UiAppearanceAbilityClient)
                                    ↓
                            SA (ui_appearance_ability.cpp)
                                ├── 权限验证 (VerifyAccessToken)
                                ├── 配置更新 (UpdateConfiguration)
                                └── 持久化 (ConfigurePersistence)
```

---

*相关内容: [架构设计](02_Architecture.md) | [内部 API](04_InnerAPI.md)*
