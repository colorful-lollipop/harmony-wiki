# 对外 N-API（JS API）- display_manager

> 本文档说明 display_manager 模块提供的对外 JavaScript Native API（N-API）

---

## 文档目的

本文档说明：
- 所有 N-API 模块的注册与导出
- 每个 API 的参数、返回值、同步/异步模式
- JS 方法到 C++ 实现的映射
- 权限要求与错误码

## 适用范围

- **N-API 模块**：display_manager 提供 2 个 N-API 模块
- **JS 层面**：应用开发者调用
- **C++ 实现层**：Frameworks 层的 N-API 绑定

---

## N-API 模块概览

display_manager 提供 **2 个 N-API 模块**（证据：`state_manager/frameworks/`）：

| 模块 | 命名空间 | 生成库 | 安装位置 |
|------|---------|---------|---------|
| **brightness** | `@ohos.display.brightness` | libbrightness.so | system/lib/module/ |
| **@ohos.brightness** | `@ohos.brightness` | libdisplay_manager_brightness_taihe_native.so | system/lib/ |

**注意**：两个模块提供类似功能，但接口不同：
- `brightness`（传统 N-API）：5 个方法
- `@ohos.brightness`（新版 ETS/ANI）：2 个 setValue 重载

---

## 模块 1：brightness（传统 N-API）

### 模块注册

**注册点**：`state_manager/frameworks/napi/brightness_module.cpp:214-217`

| 属性 | 值 | 证据 |
|------|------|------|
| **nm_version** | 1 | brightness_module.cpp:205 |
| **nm_flags** | 0 | brightness_module.cpp:206 |
| **nm_filename** | "brightness" | brightness_module.cpp:207 |
| **nm_modname** | "brightness" | brightness_module.cpp:209 |
| **nm_register_func** | Init | brightness_module.cpp:208 |

**注册函数**：
```cpp
static napi_value Init(napi_env env, napi_value exports)
{
    DISPLAY_HILOGD(COMP_FWK, "brightness init");
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("getValue", GetValue),
        DECLARE_NAPI_FUNCTION("setValue", SetValue),
        DECLARE_NAPI_FUNCTION("getMode", GetMode),
        DECLARE_NAPI_FUNCTION("setMode", SetMode),
        DECLARE_NAPI_FUNCTION("setKeepScreenOn", SetKeepScreenOn)
    };
    NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc));
    DISPLAY_HILOGD(COMP_FWK, "brightness init end");
    return exports;
}
```

**证据**：`state_manager/frameworks/napi/brightness_module.cpp:187-201`

---

### API 清单

#### 1. getValue

| 属性 | 值 |
|------|------|
| **JS 函数名** | `getValue` |
| **C++ 入口函数** | `GetValue()` |
| **绑定位置** | brightness_module.cpp:191 |
| **实现文件** | brightness.cpp |
| **参数** | 无 |
| **返回值** | `number` - 当前亮度值（0-255） |
| **同步/异步** | 异步（通过 napi_send_event） |
| **错误码** | ERR_OK（成功）、ERR_CONNECTION_FAIL（服务未连接） |
| **权限要求** | 无 |

**实现位置**：`brightness.cpp:70-93`

```cpp
static napi_value GetValue(napi_env env, napi_callback_info info)
{
    // 实现...
}
```

**调用链**：
```
JS: brightness.getValue()
  → NAPI Glue: GetValue()
    → Brightness::GetValue()
      → Brightness::GetValueCallback()
        → DisplayPowerMgrClient::GetBrightness()
          → IPC: GetBrightness(displayId)
            → DisplayPowerMgrService::GetBrightnessInner()
              → ScreenController::GetBrightness()
```

---

#### 2. setValue

| 属性 | 值 |
|------|------|
| **JS 函数名** | `setValue` |
| **C++ 入口函数** | `SetValue()` |
| **绑定位置** | brightness_module.cpp:192 |
| **实现文件** | brightness.cpp |
| **参数** | `number` - 亮度值（0-255） |
| **返回值** | `void` |
| **同步/异步** | 混合模式：
  - 立即调用 `Brightness::SetValue()`（同步，如果不需要渐变）
  - 异步调用 `Brightness::SystemSetValue()`（通过 napi_send_event），如果有渐变 |
| **错误码** | ERR_OK（成功）、ERR_INVALID_VALUE（值超出范围） |
| **权限要求** | 无 |

**实现位置**：`brightness.cpp:81-156`

```cpp
// 同步路径（无渐变）
napi_value SetValue(napi_env env, napi_callback_info info)
{
    Brightness brightness(env);
    brightness.SetValue(info);  // 直接设置
}

// 异步路径（有渐变）
static napi_value SetValue(napi_env env, napi_callback_info info)
{
    return SyncWork(env, "SetValue", BRIGHTNESS_VALUE, info, [](void *data) {
        Brightness *asyncBrightness = reinterpret_cast<Brightness*>(data);
        asyncBrightness->SystemSetValue();  // 通过事件异步执行
        delete asyncBrightness;
    });
}
```

**调用链**：
```
JS: brightness.setValue(128)
  → NAPI Glue: SetValue()
    → Brightness::SetValue() / SystemSetValue()
      → DisplayPowerMgrClient::SetBrightness(128)
        → IPC: SetBrightness(128, displayId, continuous, result, retCode)
          → DisplayPowerMgrService::SetBrightnessInner(128)
            → ScreenController::SetBrightness(128)
              → Window Manager: SetBrightness()
```

---

#### 3. getMode

| 属性 | 值 |
|------|------|
| **JS 函数名** | `getMode` |
| **C++ 入口函数** | `GetMode()` |
| **绑定位置** | brightness_module.cpp:193 |
| **实现文件** | brightness.cpp |
| **参数** | 无 |
| **返回值** | `number` - 亮度模式（0=手动，1=自动） |
| **同步/异步** | 异步（通过 napi_send_event） |
| **错误码** | ERR_OK（成功）、ERR_CONNECTION_FAIL |
| **权限要求** | 无 |

**实现位置**：`brightness.cpp:154-166`

**调用链**：
```
JS: brightness.getMode()
  → NAPI Glue: GetMode()
    → Brightness::GetMode()
      → DisplayPowerMgrClient::IsAutoAdjustBrightness()
        → IPC: IsAutoAdjustBrightness(result)
          → DisplayPowerMgrService::IsAutoAdjustBrightnessInner()
            → BrightnessManager::IsAutoAdjustBrightness()
```

---

#### 4. setMode

| 属性 | 值 |
|------|------|
| **JS 函数名** | `setMode` |
| **C++ 入口函数** | `SetMode()` |
| **绑定位置** | brightness_module.cpp:194 |
| **实现文件** | brightness.cpp |
| **参数** | `number` - 亮度模式（0=手动，1=自动） |
| **返回值** | `void` |
| **同步/异步** | 异步（通过 napi_send_event） |
| **错误码** | ERR_OK（成功）、ERR_INVALID_VALUE |
| **权限要求** | 无 |

**实现位置**：`brightness.cpp:139-153`

**调用链**：
```
JS: brightness.setMode(1)  // 自动亮度
  → NAPI Glue: SetMode()
    → Brightness::SetMode()
      → DisplayPowerMgrClient::AutoAdjustBrightness(true)
        → IPC: AutoAdjustBrightness(enable, result)
          → DisplayPowerMgrService::AutoAdjustBrightnessInner(true)
            → BrightnessManager::AutoAdjustBrightness(true)
```

---

#### 5. setKeepScreenOn

| 属性 | 值 |
|------|------|
| **JS 函数名** | `setKeepScreenOn` |
| **C++ 入口函数** | `SetKeepScreenOn()` |
| **绑定位置** | brightness_module.cpp:195 |
| **实现文件** | brightness.cpp |
| **参数** | `boolean` - 是否保持屏幕常亮 |
| **返回值** | `void` |
| **同步/异步** | 异步（通过 napi_send_event） |
| **错误码** | ERR_OK（成功）、ERR_CONNECTION_FAIL |
| **权限要求** | 无 |

**实现位置**：`brightness.cpp:173-218`

**调用链**：
```
JS: brightness.setKeepScreenOn(true)
  → NAPI Glue: SetKeepScreenOn()
    → Brightness::SetKeepScreenOn()
      → RunningLock::Lock() / UnLock()
        → Power Manager: Create/Release RunningLock
```

---

## 模块 2：@ohos.brightness（新版 ETS/ANI）

### 模块注册

**注册点**：`state_manager/frameworks/ets/taihe/brightness/src/ani_constructor.cpp:19,26`

| 属性 | 值 | 证据 |
|------|------|------|
| **命名空间** | `@ohos.brightness` | ohos.brightness.taihe:16 |
| **库名称** | `display_manager_brightness_taihe_native` | BUILD.gn |
| **构造函数** | `ANI_Constructor()` | ani_constructor.cpp:19 |
| **注册调用** | `ohos::brightness::ANIRegister(env)` | ani_constructor.cpp:26 |

**加载调用**（证据：`ohos.brightness.impl.cpp:loadLibraryWithPermissionCheck`）：
```cpp
loadLibraryWithPermissionCheck("display_manager_brightness_taihe_native.z", "@ohos.brightness")
```

**证据**：`state_manager/frameworks/ets/taihe/brightness/src/ohos.brightness.impl.cpp:loadLibraryWithPermissionCheck`

---

### API 清单

#### 1. setValue(value: i32, continuous: bool)

| 属性 | 值 |
|------|------|
| **ETS 函数名** | `setValue` |
| **ANI 入口函数** | `SetValueContinuous()` |
| **绑定位置** | ohos.brightness.impl.cpp:64 |
| **实现文件** | ohos.brightness.impl.cpp |
| **参数** | `value: i32` - 亮度值（0-255），`continuous: bool` - 是否持续更新 |
| **返回值** | `void` |
| **同步/异步** | TODO(需确认) |
| **错误码** | ERR_OK（成功）、ERR_PERMISSION_DENIED（权限拒绝）、ERR_SYSTEM_API_DENIED（系统 API 拒绝） |
| **权限要求** | 无 |

**实现位置**：`ohos.brightness.impl.cpp:39-52`

```cpp
static void SetValueContinuous(int32_t value, bool continuous)
{
    BrightnessInfo brightnessInfo;
    brightnessInfo.brightness = value;
    brightnessInfo.continuous = continuous;
    // 实现细节...
}
```

**调用链**：
```
ETS: @ohos.brightness.setValue(128, true)
  → ANI: SetValueContinuous()
    → BrightnessInfo
      → DisplayPowerMgrClient::SetBrightness()
        → IPC: SetBrightness()
          → DisplayPowerMgrService::SetBrightnessInner()
```

---

#### 2. setValue(value: i32)

| 属性 | 值 |
|------|------|
| **ETS 函数名** | `setValue` |
| **ANI 入口函数** | `SetValueInt()` |
| **绑定位置** | ohos.brightness.impl.cpp:65 |
| **实现文件** | ohos.brightness.impl.cpp |
| **参数** | `value: i32` - 亮度值（0-255） |
| **返回值** | `void` |
| **同步/异步** | TODO(需确认) |
| **错误码** | ERR_OK（成功）、ERR_PERMISSION_DENIED、ERR_SYSTEM_API_DENIED |
| **权限要求** | 无 |

**实现位置**：`ohos.brightness.impl.cpp:55-63`

**调用链**：
```
ETS: @ohos.brightness.setValue(128)
  → ANI: SetValueInt()
    → BrightnessInfo
      → DisplayPowerMgrClient::SetBrightness()
        → IPC: SetBrightness()
          → DisplayPowerMgrService::SetBrightnessInner()
```

---

## 错误码映射

| 错误名称 | 值 | 说明 | 证据 |
|---------|------|------|------|
| ERR_OK | 0 | 成功 | display_mgr_errors.h |
| ERR_CONNECTION_FAIL | 4700101 | 服务连接失败 | display_mgr_errors.h |
| ERR_PERMISSION_DENIED | 201 | 权限被拒绝 | display_mgr_errors.h:46 |
| ERR_SYSTEM_API_DENIED | 202 | 系统 API 被拒绝 | ohos.brightness.impl.cpp:34 |
| ERR_INVALID_VALUE | 401 | 无效参数 | display_mgr_errors.h |

**ANI 错误映射**（证据：`ohos.brightness.impl.cpp:32-37`）：
```cpp
{DisplayErrors::ERR_PERMISSION_DENIED, "Permission is denied"},
{DisplayErrors::ERR_SYSTEM_API_DENIED, "System permission is denied"}
```

---

## 异步模型

### N-API 异步机制

brightness 模块使用 `napi_send_event` 进行异步调用：

| 函数 | 异步方式 | 优先级 | 证据 |
|------|---------|--------|------|
| GetValue | `napi_send_event` | napi_eprio_low | brightness_module.cpp:75 |
| SetValue | 条件异步（SyncWork） | napi_eprio_low | brightness_module.cpp:100 |
| GetMode | `napi_send_event` | napi_eprio_low | brightness_module.cpp:125 |
| SetMode | `napi_send_event` | napi_eprio_low | brightness_module.cpp:142 |
| SetKeepScreenOn | `napi_send_event` | napi_eprio_low | brightness_module.cpp:167 |

**证据**：`brightness_module.cpp:56-76`

```cpp
static void SyncWorkSendEvent(napi_env env, Brightness *asyncContext,
    brightness_callback complete, napi_event_priority prio, const std::string& resName)
{
    auto task = [asyncContext, complete]() {
        // 执行回调
    };
    napi_send_event(env, task, prio, resName.c_str());
}
```

---

## 参数校验

| 参数 | 校验项 | 证据 |
|------|--------|------|
| **亮度值（setValue）** | 范围检查（0-255） | brightness.cpp |
| **亮度模式（setMode）** | 0 或 1 | brightness.cpp |
| **KeepScreenOn** | boolean 值 | brightness.cpp |
| **参数解析** | napi_get_value_* | brightness.cpp |

---

## 线程模型

| 模块 | 线程 | 说明 |
|------|------|------|
| **brightness** | 主线程 + 事件队列 | `napi_send_event` 使用事件队列 |
| **IPC 调用** | Binder 线程 | 标准 Binder IPC 模型 |
| **服务端** | FFRT 线程 | DisplayPowerMgrService 使用 FFRTQueue（证据：display_power_mgr_service.cpp:68） |

**证据**：
```cpp
// 服务端
queue_ = std::make_shared<FFRTQueue> ("display_power_mgr_service");

// 客户端异步
napi_send_event(env, task, napi_eprio_low, "GetValue");
```

---

## 相关链接

- **内部 API**：[05_Internal_API.md](05_Internal_API.md) - DisplayPowerMgrClient 接口
- **IPC 协议**：[03_Architecture.md](03_Architecture.md#zidl-通信架构)
- **GN 构建**：[06_GN_Targets.md](06_GN_Targets.md) - brightness 目标

---

## 文档更新记录

- **2026-02-06**：初始版本 v1.0，基于代码扫描生成
- **证据来源**：`state_manager/frameworks/napi/`、`state_manager/frameworks/ets/taihe/`
