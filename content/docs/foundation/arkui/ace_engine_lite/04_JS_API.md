# JS API 参考 (JerryScript 绑定)

## 目的

本文档列出 ace_engine_lite 所有对 JavaScript 应用开发者暴露的 API，包括模块、方法、参数、错误码和调用链。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 使用 **JerryScript 原生 API**（非 N-API）

---

## 模块加载机制

### requireNative 函数

**语法**：
```javascript
requireNative("category.moduleName")
```

**示例**：
```javascript
const app = requireNative("system.app");
const router = requireNative("system.router");
```

**实现位置**：`frameworks/src/core/modules/presets/require_module.cpp:23-33`

**证据**：`frameworks/src/core/modules/presets/require_module.cpp:23`（Require 函数实现）

---

## 内置模块 API

### 1. App 模块

**模块名**：`system.app`  
**初始化函数**：`InitAppModule()`  
**位置**：`frameworks/src/core/modules/app_module.cpp`

#### API 清单

| API 名称 | 参数 | 返回值 | 同步/异步 | 错误码 |
|---------|------|--------|----------|
| `getInfo()` | 无 | `Object` | 同步 | 202 (manifest 解析失败) |
| `terminate()` | 无 | `undefined` | 同步 | - |

**实现位置**：`frameworks/src/core/modules/app_module.cpp:48-75`

#### API 详细说明

##### app.getInfo()

**描述**：获取应用的基本信息（应用名、版本名、版本号）

**返回值**：
```javascript
{
    appName: string,    // 应用名称
    versionName: string, // 版本名称（如 1.0.0）
    versionCode: number  // 版本代码（如 1）
}
```

**调用链**：
```
JS: app.getInfo()
  → AppModule::GetInfo()
  → ReadManifest()
  → cJSON_Parse(manifest.json)
  → JSI::CreateObject()
  → JSI::SetStringProperty(..., appName)
  → JSI::SetStringProperty(..., versionName)
  → JSI::SetNumberProperty(..., versionCode)
```

**证据**：`frameworks/src/core/modules/app_module.cpp:48-75`

##### app.terminate()

**描述**：终止当前 Ability

**调用链**：
```
JS: app.terminate()
  → AppModule::Terminate()
  → JsAppContext::TerminateAbility()
  → AceAbility::OnStop()
```

**证据**：`frameworks/src/core/modules/app_module.cpp:122-129`

---

### 2. Router 模块

**模块名**：`system.router`  
**初始化函数**：`InitRouterModule()`  
**位置**：`frameworks/src/core/modules/router_module.cpp`

#### API 清单

| API 名称 | 参数 | 返回值 | 同步/异步 | 错误码 |
|---------|------|--------|----------|
| `replace(object)` | `{uri: string, params: Object}` | `undefined` | 异步 | 202 (参数检查失败) |

**实现位置**：`frameworks/src/core/modules/router_module.cpp:32-54`

#### API 详细说明

##### router.replace(options)

**描述**：替换当前页面并传递参数

**参数**：
```javascript
{
    uri: string,      // 目标页面 URI（必填）
    params: Object,    // 页面参数（可选）
}
```

**参数校验**：
- 必须为 Object 类型
- 必须包含 `uri` 属性（字符串）
- `params` 属性为可选

**错误码**：
- 202：参数无效（argsNum != 1 或 args == nullptr）

**调用链**：
```
JS: router.replace({uri: 'About', params: {id: '1'}})
  → RouterModule::Replace(object)
  → JsAppContext::GetTopJSAbilityImpl()
  → Router::Replace(jerry_value_t object, bool async = true)
  → JsPageStateMachine::Init(object, jsRes)
  → StateMachine::SetViewModel(viewModel)
  → StateMachine::RegisterUriAndParamsToPage(uri, params)
  → jerry_get_property(viewModel, ROUTER_PAGE)
  → jerry_parse(page.js)
  → jerry_run(render function)
```

**证据**：`frameworks/src/core/modules/router_module.cpp:32-54`

---

### 3. Console 模块

**模块名**：`console`（全局对象，无需 require）  
**初始化函数**：`ConsoleModule::Load()`  
**位置**：`frameworks/src/core/modules/presets/console_module.cpp`

#### API 清单

| API 名称 | 参数 | 返回值 | 同步/异步 |
|---------|------|--------|----------|
| `log(message)` | `string` | `undefined` | 同步 |
| `debug(message)` | `string` | `undefined` | 同步 |
| `info(message)` | `string` | `undefined` | 同步 |
| `warn(message)` | `string` | `undefined` | 同步 |
| `error(message)` | `string` | `undefined` | 同步 |

**实现位置**：`frameworks/src/core/modules/presets/console_module.cpp`

#### API 详细说明

##### console.log(message)

**描述**：输出日志信息到控制台

**参数**：
```javascript
message: any  // 要输出的消息
```

**调用链**：
```
JS: console.log("Hello")
  → ConsoleModule::Log(...)
  → HILOG_INFO(..., message)
```

**证据**：`frameworks/src/core/modules/presets/console_log_impl.cpp`

---

### 4. Timer 模块

**模块名**：`timer`  
**初始化函数**：`InitTimersModule()`  
**位置**：`frameworks/src/core/modules/presets/timer_module.cpp`

#### API 清单

| API 名称 | 参数 | 返回值 | 同步/异步 |
|---------|------|--------|----------|
| `setTimeout(callback, delay)` | `number` (timerId) | 异步 | - |
| `clearTimeout(timerId)` | `void` | 同步 | - |
| `setInterval(callback, delay)` | `number` (timerId) | 异步 | - |
| `clearInterval(timerId)` | `void` | 同步 | - |

**实现位置**：`frameworks/src/core/modules/presets/timer_module.cpp`

#### API 详细说明

##### timer.setTimeout(callback, delay)

**描述**：设置延迟执行一次的定时器

**参数**：
```javascript
callback: Function,  // 回调函数
delay: number,       // 延迟时间（毫秒）
```

**返回值**：`number` - 定时器 ID，用于清除定时器

**调用链**：
```
JS: setTimeout(() => { console.log('Hello'); }, 1000)
  → TimerModule::SetTimeout(...)
  → JsAsyncWork::DispatchAsyncWork(...)
  → JsTimerList::AddTimer(timerId, ...)
```

**证据**：`frameworks/src/core/modules/presets/timer_module.cpp`

---

### 5. Feature Ability 模块（跨设备通信）

**模块名**：`featureAbility`  
**初始化函数**：`InitFeatureAbilityModule()`  
**位置**：`frameworks/src/core/modules/presets/feature_ability_module.cpp`

#### API 清单

| API 名称 | 参数 | 返回值 | 同步/异步 | 错误码 |
|---------|------|--------|----------|
| `subscribeMsg(options)` | `void` | 同步 | 2060 (注册失败) |
| `unsubscribeMsg()` | `void` | 同步 | - |
| `sendMsg(options)` | `void` | 同步 | 2060 (发送失败) |
| `detect(options)` | `void` | 同步 | 202 (参数检查失败) |

**实现位置**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:53-171`

#### API 详细说明

##### featureAbility.subscribeMsg(options)

**描述**：注册消息接收器

**参数**：
```javascript
{
    success: Function,  // 成功回调
    fail: Function,     // 失败回调
    complete: Function  // 完成回调
}
```

**调用链**：
```
JS: featureAbility.subscribeMsg({...})
  → FeatureAbilityModule::SubscribeMessage(...)
  → AbilityKit::RegisterReceiver(bundleName)
  → AMS (Ability Manager Service)
  → 注册消息接收器
```

**证据**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:230-269`

##### featureAbility.sendMsg(options)

**描述**：发送消息到对端设备或应用

**参数**：
```javascript
{
    success: Function,      // 成功回调
    fail: Function,         // 失败回调
    complete: Function,     // 完成回调
    deviceId: string,      // 目标设备 ID
    bundleName: string,   // 目标应用包名
    abilityName: string,  // 目标 Ability 名称
    message: string        // 消息内容
}
```

**调用链**：
```
JS: featureAbility.sendMsg({...})
  → FeatureAbilityModule::SendMsgToPeer(...)
  → AbilityKit::SendMsgToPeerApp(...)
  → AMS (Ability Manager Service)
  → 发送到目标 Ability
```

**证据**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:174-227`

##### featureAbility.detect(options)

**描述**：检测设备（如手机）是否在线

**参数**：
```javascript
{
    success: Function,  // 成功回调
    fail: Function,     // 失败回调
}
```

**调用链**：
```
JS: featureAbility.detect({...})
  → FeatureAbilityModule::Detect(...)
  → AbilityKit::DetectPhoneApp(...)
  → AMS (Ability Manager Service)
  → 检测目标 Ability
```

**证据**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:120-171`

---

### 6. 其他模块

#### 6.1 Render 模块

**模块名**：`@ohos.render`（默认加载）  
**API**：`render(element)` - 渲染 JS 元素

**证据**：`frameworks/src/core/modules/presets/render_module.cpp`

#### 6.2 国际化模块

**模块名**：`system.intl` / `system.locale`  
**API**：数字格式化、日期格式化等

**证据**：`frameworks/src/core/modules/presets/intl_module.cpp`, `localization_module.cpp`

#### 6.3 系统能力模块

**模块名**：`system.syscap`  
**API**：查询系统能力是否支持

**证据**：`frameworks/src/core/modules/presets/syscap_module.cpp`

---

## 参数校验机制

### 类型检查

**实现**：使用 JerryScript 的类型检查 API

```cpp
// frameworks/src/core/base/js_fwk_common.cpp
bool ValueIsUndefined(JSIValue value);
bool ValueIsFunction(JSIValue value);
bool ValueIsObject(JSIValue value);
bool ValueIsBoolean(JSIValue value);
```

**证据**：`frameworks/src/core/base/js_fwk_common.h:177-199`

### 参数数量检查

**示例**：
```cpp
// frameworks/src/core/modules/router_module.cpp:34
if (argsNum != 1 || args == nullptr) {
    return JSI::CreateErrorWithCode(JSI_ERR_CODE_PARAM_CHECK_FAILED, "params should only be one object.");
}
```

**证据**：`frameworks/src/core/modules/router_module.cpp:34-36`

---

## 错误码定义

### ACE Framework 错误码

**位置**：`frameworks/src/core/context/ace_event_error_code.h`

| 错误码 | 值 | 说明 |
|---------|------|------|
| `JSI_ERR_CODE_PARAM_CHECK_FAILED` | 201 | 参数校验失败 |
| `JSI_ERR_CODE_OPERATION_FAILED` | 202 | 操作失败 |
| `JSI_ERR_CODE_FILE_OPERATION_FAILED` | 203 | 文件操作失败 |
| `JSI_ERR_CODE_FILE_MAX_SIZE_REACHED` | 204 | 文件大小超限 |
| `JSI_ERR_CODE_OUT_OF_MEMORY` | 205 | 内存不足 |
| `JSI_ERR_CODE_FILE_NOT_FOUND` | 206 | 文件未找到 |
| `JSI_ERR_CODE_NOT_SUPPORTED` | 207 | 不支持的操作 |

**证据**：`frameworks/src/core/context/ace_event_error_code.h`

### Feature Ability 错误码

**位置**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:40-41`

| 错误码 | 值 | 说明 |
|---------|------|------|
| `ERR_CODE_INVALID_PARAMETER` | 202 | 无效参数 |
| `ERR_CODE_SEND_MSG_FAILED` | 2060 | 消息发送失败 |

**证据**：`frameworks/src/core/modules/presets/feature_ability_module.cpp:40-41`

---

## 异步处理机制

### 异步 API

以下 API 支持异步回调：

| API | 异步方式 | 回调类型 |
|------|----------|----------|
| `router.replace` | Promise/Callback | 自定义回调 |
| `featureAbility.subscribeMsg` | Event-based | success/fail/complete |
| `featureAbility.sendMsg` | Callback | success/fail/complete |
| `featureAbility.detect` | Callback | success/fail |
| `setTimeout` | Callback | 延迟执行 |
| `setInterval` | Callback | 周期执行 |

**异步实现**：`frameworks/native_engine/async/js_async_work.cpp`

**证据**：`frameworks/native_engine/async/js_async_work.cpp:62-135`

---

## 完整模块列表

### 19+ 内置模块总览

| 序号 | 模块名 | Category | 初始化函数 | 文件位置 | 功能描述 |
|------|---------|----------|----------|----------|----------|
| 1 | **app** | system | InitAppModule | `frameworks/src/core/modules/app_module.cpp` | 应用信息查询 |
| 2 | **router** | system | InitRouterModule | `frameworks/src/core/modules/router_module.cpp` | 页面路由导航 |
| 3 | **console** | - (全局) | ConsoleModule::Load | `frameworks/src/core/modules/presets/console_module.cpp` | 日志输出 |
| 4 | **timer** | - (全局) | InitTimersModule | `frameworks/src/core/modules/presets/timer_module.cpp` | 定时器管理 |
| 5 | **featureAbility** | ohos | InitFeatureAbilityModule | `frameworks/src/core/modules/presets/feature_ability_module.cpp` | 跨设备通信 |
| 6 | **render** | @ohos | RenderModule::Load | `frameworks/src/core/modules/presets/render_module.cpp` | 渲染控制 |
| 7 | **intl** | system | InitIntlModule | `frameworks/src/core/modules/presets/intl_module.cpp` | 国际化支持 |
| 8 | **locale** | system | InitLocaleModule | `frameworks/src/core/modules/presets/localization_module.cpp` | 本地化支持 |
| 9 | **syscap** | system | InitSyscapsModule | `frameworks/src/core/modules/presets/syscap_module.cpp` | 系统能力查询 |
| 10 | **version** | - | AceVersionModule::Load | `frameworks/src/core/modules/presets/version_module.cpp` | 版本信息 |
| 11 | **appData** | - | AppDataModule::Load | `frameworks/src/core/modules/presets/app_data_module.cpp` | 应用数据 |
| 12 | **dfx** | system | InitDfxModule | `frameworks/src/core/modules/dfx_module.cpp` | 调试诊断 |
| 13 | **dialog** | system | InitDialogModule | `frameworks/src/core/modules/dialog_module.cpp` | 对话框 |
| 14 | **fetch** | system | InitFetchModule | *(条件编译)* | HTTP 请求 |
| 15 | **audio** | system | InitAudioModule | *(条件编译)* | 音频播放 |
| 16 | **file** | system | InitNativeApiFs | *(条件编译)* | 文件系统访问 |
| 17 | **storage** | system | InitNativeApiKv | *(条件编译)* | KV 存储 |
| 18 | **device** | system | InitDeviceModule | *(条件编译)* | 设备信息 |
| 19 | **deviceInfo** | system | InitDeviceInfoModule | *(条件编译)* | 设备详细信息 |
| 20 | **geolocation** | system | InitLocationModule | *(条件编译)* | 地理位置 |
| 21 | **vibrator** | system | InitVibratorModule | *(条件编译)* | 振动控制 |
| 22 | **sensor** | system | InitSensorModule | *(条件编译)* | 传感器 |
| 23 | **brightness** | system | InitBrightnessModule | *(条件编译)* | 亮度控制 |
| 24 | **battery** | system | InitBatteryModule | *(条件编译)* | 电池信息 |

### 模块分类说明

| Category | 说明 | 示例 |
|----------|------|------|
| `system.*` | 系统模块 | system.app, system.router |
| `ohos.*` | OHOS 特定模块 | featureAbility |
| 全局对象 | 自动加载，无需 require | console, timer |
| 条件编译 | 通过 Feature Flag 控制 | fetch, audio, file |

### 模块注册表

所有模块在 `OHOS_MODULES[]` 数组中注册：

```cpp
// frameworks/module_manager/ohos_module_config.h:97-158
const Module OHOS_MODULES[] = {
    {"app", InitAppModule},
    {"router", InitRouterModule},
#if (FEATURE_SUPPORT_HTTP == 1)
    {"fetch", InitFetchModule},
#endif
#if (FEATURE_MODULE_AUDIO == 1)
    {"audio", InitAudioModule},
#endif
    {"dfx", InitDfxModule},
    {"prompt", InitDialogModule},
    // ... 更多模块
};
```

**证据**：`frameworks/module_manager/ohos_module_config.h:97-158`

### 条件编译模块（Feature Flags）

| Feature Flag | 模块 | 说明 |
|--------------|------|------|
| `FEATURE_SUPPORT_HTTP` | fetch | HTTP 网络请求 |
| `FEATURE_MODULE_AUDIO` | audio | 音频播放 |
| `FEATURE_MODULE_STORAGE` | file, storage | 文件和存储 |
| `FEATURE_MODULE_DEVICE` | device, deviceInfo | 设备信息 |
| `FEATURE_MODULE_GEO` | geolocation | 地理位置 |
| `FEATURE_MODULE_SENSOR` | vibrator, sensor | 传感器 |
| `FEATURE_MODULE_BRIGHTNESS` | brightness | 亮度控制 |
| `FEATURE_MODULE_BATTERY` | battery | 电池信息 |

**证据**：`frameworks/module_manager/ohos_module_config.h`

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - JSI 封装层
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [05_Inner_API.md](05_Inner_API.md) - 内部 API
