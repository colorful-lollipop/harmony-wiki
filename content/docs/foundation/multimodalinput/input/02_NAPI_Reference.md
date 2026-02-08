# 02_NAPI_Reference - N-API 接口参考

## 概述

本文档详细描述 multimodalinput_input 子系统对外暴露的所有 N-API 接口。

## 代码证据

**N-API 模块注册位置**: `frameworks/napi/*/src/native_register_module.cpp`

---

## 2.1 inputEventClient 模块

### 2.1.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.inputEventClient` |
| **注册文件** | `frameworks/napi/input_event_client/src/js_register_module.cpp` |
| **系统能力** | `SystemCapability.MultimodalInput.Input.Core` |
| **权限要求** | `ohos.permission.INJECT_INPUT_EVENT` |

### 2.1.2 API 清单

#### 2.1.2.1 injectEvent

```typescript
function injectEvent(options: { KeyEvent: KeyEvent }): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| options | object | 是 | 注入事件选项 |
| options.KeyEvent | KeyEvent | 是 | 按键事件对象 |

**示例**:
```typescript
import input from '@ohos.multimodalInput.inputEventClient'

var keyEvent = {
    isPressed: true,
    code: 2,  // BACK key
    keyDownDuration: 10
};
input.injectEvent({ KeyEvent: keyEvent });
```

**C++ 实现**: `js_register_module.cpp:169-213`

---

#### 2.1.2.2 injectKeyEvent

```typescript
function injectKeyEvent(keyEvent: KeyEvent): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keyEvent | KeyEvent | 是 | 按键事件对象 |

**C++ 实现**: `js_register_module.cpp:215-259`

---

#### 2.1.2.3 injectMouseEvent

```typescript
function injectMouseEvent(options: { mouseEvent: MouseEvent; useGlobalCoordinate?: boolean }): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| options | object | 是 | 注入选项 |
| options.mouseEvent | MouseEvent | 是 | 鼠标事件 |
| options.useGlobalCoordinate | boolean | 否 | 是否使用全局坐标 |

**C++ 实现**: `js_register_module.cpp:401-457`

---

#### 2.1.2.4 injectTouchEvent

```typescript
function injectTouchEvent(options: { touchEvent: TouchEvent; useGlobalCoordinate?: boolean }): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| options | object | 是 | 注入选项 |
| options.touchEvent | TouchEvent | 是 | 触摸事件 |
| options.useGlobalCoordinate | boolean | 否 | 是否使用全局坐标 |

**C++ 实现**: `js_register_module.cpp:640-698`

---

#### 2.1.2.5 injectJoystickEvent

```typescript
function injectJoystickEvent(options: { joystickEvent: JoystickEvent }): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| options | object | 是 | 注入选项 |
| options.joystickEvent | JoystickEvent | 是 | 游戏手柄事件 |

**C++ 实现**: `js_register_module.cpp:802-843`

---

#### 2.1.2.6 permitInjection

```typescript
function permitInjection(enable: boolean): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| enable | boolean | 是 | 是否授权事件注入 |

**C++ 实现**: `js_register_module.cpp:845-871`

---

## 2.2 inputDevice 模块

### 2.2.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.inputDevice` |
| **注册文件** | `frameworks/napi/input_device/src/native_register_module.cpp` |
| **系统能力** | `SystemCapability.MultimodalInput.Input.InputDevice` |

### 2.2.2 API 清单

| API | 功能 | 同步/异步 |
|-----|------|----------|
| `getDevice(deviceId)` | 获取设备信息 | 异步 |
| `getDeviceIds()` | 获取所有设备 ID | 同步 |
| `supportKeys(deviceId, keys)` | 查询设备支持的按键 | 异步 |
| `supportKeysSync(deviceId, keys)` | 查询设备支持的按键 | 同步 |
| `getKeyboardType(deviceId)` | 获取键盘类型 | 异步 |
| `getKeyboardTypeSync(deviceId)` | 获取键盘类型 | 同步 |
| `getDeviceList()` | 获取设备列表 | 同步 |
| `getDeviceInfo(deviceId)` | 获取设备信息 | 异步 |
| `getDeviceInfoSync(deviceId)` | 获取设备信息 | 同步 |
| `setKeyboardRepeatDelay(delay)` | 设置按键重复延迟 | 同步 |
| `setKeyboardRepeatRate(rate)` | 设置按键重复速率 | 同步 |
| `getKeyboardRepeatDelay()` | 获取按键重复延迟 | 同步 |
| `getKeyboardRepeatRate()` | 获取按键重复速率 | 同步 |
| `getIntervalSinceLastInput()` | 获取最后输入间隔 | 同步 |
| `setInputDeviceEnabled(enabled)` | 设置设备使能状态 | 同步 |
| `setFunctionKeyEnabled(enabled)` | 设置功能键使能 | 同步 |
| `isFunctionKeyEnabled()` | 查询功能键状态 | 同步 |
| `on('change', callback)` | 监听设备变化 | 异步 |
| `off('change', callback?)` | 取消设备变化监听 | 异步 |

### 2.2.3 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 201 | 权限不足 |
| 202 | 非系统应用 |

---

## 2.3 inputMonitor 模块

### 2.3.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.inputMonitor` |
| **注册文件** | `frameworks/napi/input_monitor/src/js_input_monitor_module.cpp` |
| **系统能力** | `SystemCapability.MultimodalInput.Input.InputMonitor` |
| **权限要求** | `ohos.permission.INPUT_MONITORING` |

### 2.3.2 API 清单

#### 2.3.2.1 on

```typescript
function on(type: string, callback: Callback<TouchEvent>): void;
function on(type: string, fingers: number, callback: Callback<TouchEvent>): void;
function on(type: 'mouse', rects: Rect[], callback: Callback<MouseEvent>): void;
function on(type: 'keyPressed', keys: number[], callback: Callback<KeyEvent>): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 是 | 事件类型 |
| callback | Callback | 是 | 回调函数 |
| fingers | number | 否 | 手指数量 |
| rects | Rect[] | 否 | 监听区域 |
| keys | number[] | 否 | 按键列表 |

**支持的事件类型**:
- `touch` - 触摸事件
- `mouse` - 鼠标事件
- `pinch` - 捏合手势
- `threeFingersSwipe` - 三指滑动
- `fourFingersSwipe` - 四指滑动
- `rotate` - 旋转手势
- `keyPressed` - 按键事件

**C++ 实现**: `js_input_monitor_module.cpp:42-76` (Api9) + `js_input_monitor_module.cpp:141-191` (标准)

---

#### 2.3.2.2 off

```typescript
function off(type: string): void;
function off(type: string, callback: Callback): void;
function off(type: string, fingers: number): void;
function off(type: string, fingers: number, callback: Callback): void;
```

取消事件监听。

**C++ 实现**: `js_input_monitor_module.cpp:215-334`

---

#### 2.3.2.3 queryTouchEvents

```typescript
function queryTouchEvents(count: number): TouchEvent[];
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| count | number | 是 | 查询事件数量 |

**C++ 实现**: `js_input_monitor_module.cpp:336-356`

---

## 2.4 pointer 模块

### 2.4.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.pointer` |
| **注册文件** | `frameworks/napi/pointer/src/js_pointer_module.cpp` |

### 2.4.2 API 清单 (40+ API)

| 类别 | API | 功能 |
|------|-----|------|
| 可见性 | `setPointerVisible(visible)` | 设置指针可见性 |
| | `isPointerVisible()` | 查询指针可见性 |
| 颜色 | `setPointerColor(color)` | 设置指针颜色 |
| | `getPointerColor()` | 查询指针颜色 |
| 速度 | `setPointerSpeed(speed)` | 设置指针速度 |
| | `getPointerSpeed()` | 查询指针速度 |
| 样式 | `setPointerStyle(style)` | 设置指针样式 |
| | `getPointerStyle()` | 查询指针样式 |
| 大小 | `setPointerSize(size)` | 设置指针大小 |
| | `getPointerSize()` | 查询指针大小 |
| 触摸板 | `setTouchpadScrollSwitch(switch)` | 设置触摸板滚动开关 |
| | `getTouchpadScrollSwitch()` | 查询触摸板滚动开关 |
| | `setTouchpadPointerSpeed(speed)` | 设置触摸板速度 |
| | `getTouchpadPointerSpeed()` | 查询触摸板速度 |
| 鼠标 | `setMousePrimaryButton(btn)` | 设置主鼠标键 |
| | `getMousePrimaryButton()` | 查询主鼠标键 |
| | `setMouseScrollRows(rows)` | 设置滚轮行数 |
| | `getMouseScrollRows()` | 查询滚轮行数 |

---

## 2.5 inputConsumer 模块

### 2.5.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.inputConsumer` |
| **权限要求** | `ohos.permission.INPUT_MONITORING` |

### 2.5.2 API 清单

| API | 功能 |
|-----|------|
| `on(type, callback)` | 注册事件监听 |
| `off(type, callback?)` | 取消事件监听 |
| `setShieldStatus(status)` | 设置盾牌状态 |
| `getShieldStatus()` | 查询盾牌状态 |
| `getAllSystemHotkeys()` | 获取所有系统热键 |

---

## 2.6 infraredEmitter 模块

### 2.6.1 模块信息

| 属性 | 值 |
|------|-----|
| **JS 模块名** | `multimodalInput.infraredEmitter` |
| **权限要求** | `ohos.permission.MANAGE_INPUT_INFRARED_EMITTER` |

### 2.6.2 API 清单

| API | 功能 |
|-----|------|
| `hasIrEmitter()` | 检查是否支持红外 |
| `getInfraredFrequencies()` | 获取红外频率列表 |
| `transmitInfrared(frequency, pattern)` | 发射红外信号 |

---

## 2.7 其他事件模块

以下模块主要提供事件类型定义和常量：

| 模块 | 功能 |
|------|------|
| `keyEvent` | 按键事件类型定义 |
| `mouseEvent` | 鼠标事件类型定义 |
| `touchEvent` | 触摸事件类型定义 |
| `gestureEvent` | 手势事件类型定义 |
| `joystickEvent` | 游戏手柄事件类型定义 |
| `keyCode` | 按键码常量 |
| `intentionCode` | 意图码常量 |
| `shortKey` | 快捷键配置 (`setKeyDownDuration`) |

---

## 2.8 错误码参考

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | RET_OK | 成功 |
| 201 | COMMON_PERMISSION_CHECK_ERROR | 权限检查失败 |
| 202 | ERROR_NOT_SYSAPI | 非系统应用 |
| 401 | COMMON_PARAMETER_ERROR | 参数错误 |

**代码位置**: `util/common/include/error_multimodal.h`

---

## 2.9 新建 N-API 模块指南

### 2.9.1 目录结构

```
frameworks/napi/{module_name}/
├── BUILD.gn
├── src/
│   ├── native_register_module.cpp    # 模块注册
│   ├── js_{module_name}_context.cpp  # 上下文管理
│   └── js_{module_name}_manager.cpp  # 管理器实现
└── index.d.ts                        # 类型定义
```

### 2.9.2 注册模板

```cpp
static napi_module {ModuleName}Module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = {ModuleName}Context::Export,
    .nm_modname = "multimodalInput.{moduleName}",
    .nm_priv = ((void*)0),
    .reserved = { 0 },
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&{ModuleName}Module);
}
```
