# 04_External_CAPI - 对外 CAPI 接口

## 目的

本文档详细说明 GameController Framework 的对外 CAPI 接口，这是游戏开发者使用的主要 API。

## 适用范围

- **游戏开发者** - 本文档重点读者
- 需要集成游戏控制器能力的应用开发人员

## CAPI 概览

**重要说明**: 当前版本仅提供 CAPI（C Native API），ArkTS (N-API) 接口未来规划。

**证据**: `README.md:31` - "Note: Currently, only CAPI is available, and ArkTS interface will be planned for the future."

### CAPI 库

| 库名称 | 产物文件 | 安装路径 | 用途 |
|---------|----------|----------|------|
| libohgame_controller.z.so | libohgame_controller.z.so | ndk/ | 游戏应用链接使用 |

**证据**:
- `interfaces/kits/c/BUILD.gn:27-68`: ohgame_controller target 定义
- `interfaces/kits/c/BUILD.gn:65`: relative_install_dir = "ndk/"

## API 清单表

### 1. GameDevice APIs

| C API 名称 | C++ 实现 | 同步/异步 | 参数校验 | 错误码 | 文件:行号 |
|------------|-----------|-----------|----------|--------|-----------|
| OH_GameDevice_GetAllDeviceInfos | GameDeviceProxy::GetAllDeviceInfos | 同步 | allDeviceInfos 非空 | GAME_CONTROLLER_PARAM_ERROR, GAME_CONTROLLER_MULTIMODAL_INPUT_ERROR, GAME_CONTROLLER_NO_MEMORY | interfaces/kits/c/game_device.h:58 |
| OH_GameDevice_RegisterDeviceMonitor | GameDeviceProxy::RegisterDeviceMonitor | 同步 | deviceMonitorCallback 非空 | GAME_CONTROLLER_PARAM_ERROR | interfaces/kits/c/game_device.h:68 |
| OH_GameDevice_UnregisterDeviceMonitor | GameDeviceProxy::UnRegisterDeviceMonitor | 同步 | N/A | GAME_CONTROLLER_SUCCESS | interfaces/kits/c/game_device.h:76 |
| OH_GameDevice_DestroyAllDeviceInfos | GameDeviceProxy::DestroyAllDeviceInfos | 同步 | allDeviceInfos 非空 | GAME_CONTROLLER_PARAM_ERROR | interfaces/kits/c/game_device.h:86 |
| OH_GameDevice_AllDeviceInfos_GetCount | GameDeviceProxy::GetCountFromAllDeviceInfos | 同步 | allDeviceInfos/index 非空 | GAME_CONTROLLER_PARAM_ERROR | interfaces/kits/c/game_device.h:97 |
| OH_GameDevice_AllDeviceInfos_GetDeviceInfo | GameDeviceProxy::GetDeviceInfoFromAllDeviceInfos | 同步 | allDeviceInfos/index 非空，范围校验 | GAME_CONTROLLER_PARAM_ERROR | interfaces/kits/c/game_device.h:112 |

#### OH_GameDevice_GetAllDeviceInfos

**功能**: 获取所有在线游戏外设

**签名**:
```c
GameController_ErrorCode OH_GameDevice_GetAllDeviceInfos(
    GameDevice_AllDeviceInfos** allDeviceInfos);
```

**参数**:
- `allDeviceInfos`: 双指针，用于返回设备列表，不能为空

**返回值**:
- `GAME_CONTROLLER_SUCCESS` (0): 成功
- `GAME_CONTROLLER_PARAM_ERROR` (401): 参数为空
- `GAME_CONTROLLER_MULTIMODAL_INPUT_ERROR` (32200001): 多模态输入异常
- `GAME_CONTROLLER_NO_MEMORY` (32200002): 内存不足

**证据**: `interfaces/kits/c/game_device.h:58`

#### OH_GameDevice_RegisterDeviceMonitor

**功能**: 注册设备上线/下线监听器

**签名**:
```c
GameController_ErrorCode OH_GameDevice_RegisterDeviceMonitor(
    GameDevice_DeviceMonitorCallback deviceMonitorCallback);
```

**参数**:
- `deviceMonitorCallback`: 设备事件回调函数，不能为空

**回调函数定义**:
```c
typedef void (*GameDevice_DeviceMonitorCallback)(
    GameDevice_DeviceEvent deviceEvent);
```

**返回值**:
- `GAME_CONTROLLER_SUCCESS` (0): 成功
- `GAME_CONTROLLER_PARAM_ERROR` (401): 回调为空

**证据**: `interfaces/kits/c/game_device.h:68`

#### OH_GameDevice_UnregisterDeviceMonitor

**功能**: 注销设备监听器

**签名**:
```c
GameController_ErrorCode OH_GameDevice_UnregisterDeviceMonitor(void);
```

**返回值**:
- `GAME_CONTROLLER_SUCCESS` (0): 成功

**证据**: `interfaces/kits/c/game_device.h:76`

#### OH_GameDevice_DestroyAllDeviceInfos

**功能**: 销毁设备信息对象

**签名**:
```c
GameController_ErrorCode OH_GameDevice_DestroyAllDeviceInfos(
    GameDevice_AllDeviceInfos** allDeviceInfos);
```

**返回值**:
- `GAME_CONTROLLER_SUCCESS` (0): 成功
- `GAME_CONTROLLER_PARAM_ERROR` (401): 参数为空

**证据**: `interfaces/kits/c/game_device.h:86`

### 2. GamePad APIs

#### 按键监听 API

| C API 名称 | 按键类型 | C++ 实现 | 同步/异步 | 文件:行号 |
|------------|---------|-----------|-----------|-----------|
| OH_GamePad_LeftShoulder_RegisterButtonInputMonitor | 左肩键 | GamePadProxy::RegisterLeftShoulderButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:52 |
| OH_GamePad_LeftShoulder_UnregisterButtonInputMonitor | 左肩键 | GamePadProxy::UnRegisterLeftShoulderButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:60 |
| OH_GamePad_RightShoulder_RegisterButtonInputMonitor | 右肩键 | GamePadProxy::RegisterRightShoulderButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:70 |
| OH_GamePad_RightShoulder_UnregisterButtonInputMonitor | 右肩键 | GamePadProxy::UnRegisterRightShoulderButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:78 |
| OH_GamePad_LeftTrigger_RegisterButtonInputMonitor | 左扳机按钮 | GamePadProxy::RegisterLeftTriggerButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:88 |
| OH_GamePad_LeftTrigger_UnregisterButtonInputMonitor | 左扳机按钮 | GamePadProxy::UnRegisterLeftTriggerButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:96 |
| OH_GamePad_RightTrigger_RegisterButtonInputMonitor | 右扳机按钮 | GamePadProxy::RegisterRightTriggerButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:124 |
| OH_GamePad_RightTrigger_UnregisterButtonInputMonitor | 右扳机按钮 | GamePadProxy::UnRegisterRightTriggerButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:132 |
| OH_GamePad_ButtonMenu_RegisterButtonInputMonitor | 菜单键 | GamePadProxy::RegisterButtonMenuButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:160 |
| OH_GamePad_ButtonMenu_UnregisterButtonInputMonitor | 菜单键 | GamePadProxy::UnRegisterButtonMenuButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:168 |
| OH_GamePad_ButtonHome_RegisterButtonInputMonitor | Home 键 | GamePadProxy::RegisterButtonHomeButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:178 |
| OH_GamePad_ButtonHome_UnregisterButtonInputMonitor | Home 键 | GamePadProxy::UnRegisterButtonHomeButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:186 |
| OH_GamePad_ButtonA_RegisterButtonInputMonitor | A 键 | GamePadProxy::RegisterButtonAButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:196 |
| OH_GamePad_ButtonA_UnregisterButtonInputMonitor | A 键 | GamePadProxy::UnRegisterButtonAButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:204 |
| OH_GamePad_ButtonB_RegisterButtonInputMonitor | B 键 | GamePadProxy::RegisterButtonBButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:214 |
| OH_GamePad_ButtonB_UnregisterButtonInputMonitor | B 键 | GamePadProxy::UnRegisterButtonBButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:222 |
| OH_GamePad_ButtonX_RegisterButtonInputMonitor | X 键 | GamePadProxy::RegisterButtonXButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:232 |
| OH_GamePad_ButtonX_UnregisterButtonInputMonitor | X 键 | GamePadProxy::UnRegisterButtonXButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:240 |
| OH_GamePad_ButtonY_RegisterButtonInputMonitor | Y 键 | GamePadProxy::RegisterButtonYButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:250 |
| OH_GamePad_ButtonY_UnregisterButtonInputMonitor | Y 键 | GamePadProxy::UnRegisterButtonYButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:258 |
| OH_GamePad_ButtonC_RegisterButtonInputMonitor | C 键 | GamePadProxy::RegisterButtonCButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:268 |
| OH_GamePad_ButtonC_UnregisterButtonInputMonitor | C 键 | GamePadProxy::UnRegisterButtonCButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:276 |
| OH_GamePad_Dpad_LeftButton_RegisterButtonInputMonitor | 十字左键 | GamePadProxy::RegisterDpadLeftButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:286 |
| OH_GamePad_Dpad_LeftButton_UnregisterButtonInputMonitor | 十字左键 | GamePadProxy::UnRegisterDpadLeftButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:294 |
| OH_GamePad_Dpad_RightButton_RegisterButtonInputMonitor | 十字右键 | GamePadProxy::RegisterDpadRightButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:304 |
| OH_GamePad_Dpad_RightButton_UnregisterButtonInputMonitor | 十字右键 | GamePadProxy::UnRegisterDpadRightButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:312 |
| OH_GamePad_Dpad_UpButton_RegisterButtonInputMonitor | 十字上键 | GamePadProxy::RegisterDpadUpButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:322 |
| OH_GamePad_Dpad_UpButton_UnregisterButtonInputMonitor | 十字上键 | GamePadProxy::UnRegisterDpadUpButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:330 |
| OH_GamePad_Dpad_DownButton_RegisterButtonInputMonitor | 十字下键 | GamePadProxy::RegisterDpadDownButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:340 |
| OH_GamePad_Dpad_DownButton_UnregisterButtonInputMonitor | 十字下键 | GamePadProxy::UnRegisterDpadDownButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:348 |
| OH_GamePad_LeftThumbstick_RegisterButtonInputMonitor | 左摇杆按钮 | GamePadProxy::RegisterLeftThumbstickButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:376 |
| OH_GamePad_LeftThumbstick_UnregisterButtonInputMonitor | 左摇杆按钮 | GamePadProxy::UnRegisterLeftThumbstickButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:384 |
| OH_GamePad_RightThumbstick_RegisterButtonInputMonitor | 右摇杆按钮 | GamePadProxy::RegisterRightThumbstickButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:412 |
| OH_GamePad_RightThumbstick_UnregisterButtonInputMonitor | 右摇杆按钮 | GamePadProxy::UnRegisterRightThumbstickButtonInputMonitor | 同步 | interfaces/kits/c/game_pad.h:420 |

**统一参数校验**:
- `inputMonitorCallback`: 回调函数，不能为空
- 返回 `GAME_CONTROLLER_PARAM_ERROR` (401) 如果为空

**证据**: `interfaces/kits/c/game_pad.h:48-50` - 文档注释

#### 轴事件监听 API

| C API 名称 | 轴类型 | C++ 实现 | 同步/异步 | 文件:行号 |
|------------|---------|-----------|-----------|-----------|
| OH_GamePad_LeftTrigger_RegisterAxisInputMonitor | 左扳机轴 | GamePadProxy::RegisterLeftTriggerAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:106 |
| OH_GamePad_LeftTrigger_UnregisterAxisInputMonitor | 左扳机轴 | GamePadProxy::UnRegisterLeftTriggerAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:114 |
| OH_GamePad_RightTrigger_RegisterAxisInputMonitor | 右扳机轴 | GamePadProxy::RegisterRightTriggerAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:142 |
| OH_GamePad_RightTrigger_UnregisterAxisInputMonitor | 右扳机轴 | GamePadProxy::UnRegisterRightTriggerAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:150 |
| OH_GamePad_Dpad_RegisterAxisInputMonitor | 十字键轴 | GamePadProxy::RegisterDpadAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:358 |
| OH_GamePad_Dpad_UnregisterAxisInputMonitor | 十字键轴 | GamePadProxy::UnRegisterDpadAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:366 |
| OH_GamePad_LeftThumbstick_RegisterAxisInputMonitor | 左摇杆轴 | GamePadProxy::RegisterLeftThumbstickAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:394 |
| OH_GamePad_LeftThumbstick_UnregisterAxisInputMonitor | 左摇杆轴 | GamePadProxy::UnRegisterLeftThumbstickAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:402 |
| OH_GamePad_RightThumbstick_RegisterAxisInputMonitor | 右摇杆轴 | GamePadProxy::RegisterRightThumbstickAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:430 |
| OH_GamePad_RightThumbstick_UnregisterAxisInputMonitor | 右摇杆轴 | GamePadProxy::UnRegisterRightThumbstickAxisInputMonitor | 同步 | interfaces/kits/c/game_pad.h:438 |

**统一参数校验**:
- `inputMonitorCallback`: 回调函数，不能为空
- 返回 `GAME_CONTROLLER_PARAM_ERROR` (401) 如果为空

### 3. 数据模型与 Getter APIs

#### GameDeviceEventData APIs

| C API 名称 | C++ 实现 | 文件:行号 |
|------------|-----------|-----------|
| OH_GameDevice_DeviceEvent_GetChangedType | GameDeviceEventProxy::GetChangedType | interfaces/kits/c/game_device_event.h:18 |
| OH_GameDevice_DeviceEvent_GetDeviceInfo | GameDeviceEventProxy::GetDeviceInfo | interfaces/kits/c/game_device_event.h:25 |
| OH_GameDevice_DestroyDeviceInfo | GameDeviceEventProxy::DestroyDeviceInfo | interfaces/kits/c/game_device_event.h:32 |
| OH_GameDevice_DeviceInfo_GetDeviceId | GameDeviceEventProxy::GetDeviceIdFromDeviceInfo | interfaces/kits/c/game_device_event.h:37 |
| OH_GameDevice_DeviceInfo_GetName | GameDeviceEventProxy::GetNameFromDeviceInfo | interfaces/kits/c/game_device_event.h:43 |
| OH_GameDevice_DeviceInfo_GetProduct | GameDeviceEventProxy::GetProductFromDeviceInfo | interfaces/kits/c/game_device_event.h:49 |
| OH_GameDevice_DeviceInfo_GetVersion | GameDeviceEventProxy::GetVersionFromDeviceInfo | interfaces/kits/c/game_device_event.h:55 |
| OH_GameDevice_DeviceInfo_GetPhysicalAddress | GameDeviceEventProxy::GetPhysFromDeviceInfo | interfaces/kits/c/game_device_event.h:61 |
| OH_GameDevice_DeviceInfo_GetDeviceType | GameDeviceEventProxy::GetDeviceTypeFromDeviceInfo | interfaces/kits/c/game_device_event.h:67 |

**证据**: `interfaces/kits/c/game_device_event.h`

#### GamePadEventData APIs

**Button Event Getters**:
| C API 名称 | 文件:行号 |
|------------|-----------|
| OH_GamePad_ButtonEvent_GetDeviceId | interfaces/kits/c/game_pad_event.h:20 |
| OH_GamePad_ButtonEvent_GetButtonAction | interfaces/kits/c/game_pad_event.h:26 |
| OH_GamePad_ButtonEvent_GetButtonCode | interfaces/kits/c/game_pad_event.h:33 |
| OH_GamePad_ButtonEvent_GetButtonCodeName | interfaces/kits/c/game_pad_event.h:39 |
| OH_GamePad_ButtonEvent_GetActionTime | interfaces/kits/c/game_pad_event.h:78 |
| OH_GamePad_PressedButtons_GetCount | interfaces/kits/c/game_pad_event.h:46 |
| OH_GamePad_PressedButtons_GetButtonInfo | interfaces/kits/c/game_pad_event.h:52 |
| OH_GamePad_DestroyPressedButton | interfaces/kits/c/game_pad_event.h:60 |
| OH_GamePad_PressedButton_GetButtonCode | interfaces/kits/c/game_pad_event.h:65 |
| OH_GamePad_PressedButton_GetButtonCodeName | interfaces/kits/c/game_pad_event.h:71 |

**Axis Event Getters**:
| C API 名称 | 文件:行号 |
|------------|-----------|
| OH_GamePad_AxisEvent_GetDeviceId | interfaces/kits/c/game_pad_event.h:84 |
| OH_GamePad_AxisEvent_GetAxisSourceType | interfaces/kits/c/game_pad_event.h:89 |
| OH_GamePad_AxisEvent_GetXAxisValue | interfaces/kits/c/game_pad_event.h:96 |
| OH_GamePad_AxisEvent_GetYAxisValue | interfaces/kits/c/game_pad_event.h:102 |
| OH_GamePad_AxisEvent_GetZAxisValue | interfaces/kits/c/game_pad_event.h:108 |
| OH_GamePad_AxisEvent_GetRZAxisValue | interfaces/kits/c/game_pad_event.h:114 |
| OH_GamePad_AxisEvent_GetHatXAxisValue | interfaces/kits/c/game_pad_event.h:120 |
| OH_GamePad_AxisEvent_GetHatYAxisValue | interfaces/kits/c/game_pad_event.h:126 |
| OH_GamePad_AxisEvent_GetBrakeAxisValue | interfaces/kits/c/game_pad_event.h:132 |
| OH_GamePad_AxisEvent_GetGasAxisValue | interfaces/kits/c/game_pad_event.h:138 |
| OH_GamePad_AxisEvent_GetActionTime | interfaces/kits/c/game_pad_event.h:144 |

**证据**: `interfaces/kits/c/game_pad_event.h`

## 调用链示例

### 设备监听链示例

```c
#include <OHGameController.h>

// 1. 定义设备事件回调
void OnDeviceEvent(GameDevice_DeviceEvent event) {
    GameDevice_ChangedType type = OH_GameDevice_DeviceEvent_GetChangedType(event);

    if (type == GAME_DEVICE_ADDED) {
        // 设备上线
        GameDevice_DeviceInfo* info = NULL;
        OH_GameDevice_DeviceEvent_GetDeviceInfo(event, &info);
        // 处理新设备
    } else if (type == GAME_DEVICE_REMOVED) {
        // 设备下线
    }
}

// 2. 注册设备监听
GameController_ErrorCode ret = OH_GameDevice_RegisterDeviceMonitor(OnDeviceEvent);
if (ret != GAME_CONTROLLER_SUCCESS) {
    // 处理错误
}

// 3. 获取所有设备
GameDevice_AllDeviceInfos* allDevices = NULL;
ret = OH_GameDevice_GetAllDeviceInfos(&allDevices);
if (ret == GAME_CONTROLLER_SUCCESS) {
    int32_t count = 0;
    OH_GameDevice_AllDeviceInfos_GetCount(allDevices, &count);

    for (int32_t i = 0; i < count; i++) {
        GameDevice_DeviceInfo* device = NULL;
        OH_GameDevice_AllDeviceInfos_GetDeviceInfo(allDevices, i, &device);
        // 处理每个设备
    }

    // 释放资源
    OH_GameDevice_DestroyAllDeviceInfos(&allDevices);
}

// 4. 应用退出时注销监听
OH_GameDevice_UnregisterDeviceMonitor();
```

**证据**: `interfaces/kits/c/game_device.h`, `interfaces/kits/c/game_device_event.h`

### 按键监听链示例

```c
#include <OHGameController.h>

// 1. 定义按键事件回调
void OnButtonEvent(GamePad_ButtonEvent event) {
    GamePad_ButtonCode buttonCode = OH_GamePad_ButtonEvent_GetButtonCode(event);
    GamePad_ButtonAction action = OH_GamePad_ButtonEvent_GetButtonAction(event);

    if (buttonCode == GAME_PAD_BUTTON_A && action == GAME_PAD_BUTTON_PRESS) {
        // A 键按下
        printf("A button pressed\n");
    }
}

// 2. 注册按键监听
GameController_ErrorCode ret = OH_GamePad_ButtonA_RegisterButtonInputMonitor(OnButtonEvent);
if (ret != GAME_CONTROLLER_SUCCESS) {
    // 处理错误
}

// 3. 应用退出时注销监听
OH_GamePad_ButtonA_UnregisterButtonInputMonitor();
```

**证据**: `interfaces/kits/c/game_pad.h`, `interfaces/kits/c/game_pad_event.h`

## 完整调用链

### 设备查询调用链

```
游戏应用 C 代码
    │
    ▼ OH_GameDevice_GetAllDeviceInfos()
    │
    ▼ game_device.cpp (C wrapper)
    │
    ▼ GameDeviceProxy::GetAllDeviceInfos()
    │
    ▼ frameworks/capi/src/game_device_proxy.cpp
    │
    ▼ MultiModalInputMonitor::GetAllDeviceInfos()
    │
    ▼ frameworks/native/multi_modal_input/src/multi_modal_input_monitor.cpp
    │
    ▼ DeviceInfoService::GetAllDevices()
    │
    ▼ frameworks/native/multi_modal_input/src/device_info_service.cpp (async MMI query)
    │
    ▼ MMI Service (MultiModalInput)
    │
    └─► 返回设备列表
```

**证据**:
- `interfaces/kits/c/game_device.h:58`: OH_GameDevice_GetAllDeviceInfos 定义
- `interfaces/kits/c/game_device.cpp:19`: GameDeviceProxy::GetAllDeviceInfos 调用
- `frameworks/native/multi_modal_input/include/multi_modal_input_monitor.h`: MultiModalInputMonitor 接口

### 输入事件调用链

```
游戏应用 C 代码
    │
    ▼ OH_GamePad_ButtonA_RegisterButtonInputMonitor(callback)
    │
    ▼ game_pad.cpp (C wrapper)
    │
    ▼ GamePadProxy::RegisterButtonInputMonitor()
    │
    ▼ frameworks/capi/src/game_pad_proxy.cpp
    │
    ▼ WindowInputIntercept::RegisterGamePadInputEventCallback()
    │
    ▼ frameworks/native/window/include/window_input_intercept.h
    │
    ▼ InputEventCallback::RegisterGamePadButtonEventCallback()
    │
    ▼ frameworks/native/window/src/input_event_callback.cpp
    │
    ▼ Window Framework (注册输入拦截）
    │
    ▼ 当按键事件发生时
    │
    └─► 回调到游戏应用
```

**证据**:
- `interfaces/kits/c/game_pad.h:196`: OH_GamePad_ButtonA_RegisterButtonInputMonitor 定义
- `frameworks/native/window/include/window_input_intercept.h`: WindowInputIntercept 接口
- `frameworks/native/window/src/input_event_callback.cpp`: InputEventCallback 实现

## 参数校验与错误处理

### 参数校验模式

**1. Null 指针检查**:
```cpp
// frameworks/capi/src/game_device_proxy.cpp:21
if (deviceMonitorCallback == nullptr) {
    HILOGE("[CAPI][RegisterDeviceMonitor]deviceMonitorCallback is nullptr");
    return GameController_ErrorCode::GAME_CONTROLLER_PARAM_ERROR;
}
```

**证据**: `frameworks/capi/src/game_device_proxy.cpp:21`

**2. 范围校验**:
```cpp
// frameworks/capi/src/game_device_proxy.cpp:47
if (index >= count || index < 0) {
    HILOGE("[CAPI][GetDeviceInfoFromAllDeviceInfos]index is out of range");
    return GameController_ErrorCode::GAME_CONTROLLER_PARAM_ERROR;
}
```

**证据**: `frameworks/capi/src/game_device_proxy.cpp:47`

**3. 内存分配检查**:
```cpp
// frameworks/capi/src/game_device_proxy.cpp:29
int32_t ret = StringUtils::ConvertToCharPtrArray(
    ((BasicDeviceInfo*)deviceInfo)->uniq, deviceId);
if (ret == GAME_CONTROLLER_SUCCESS) {
    return GameController_ErrorCode::GAME_CONTROLLER_SUCCESS;
}
return GameController_ErrorCode::GAME_CONTROLLER_NO_MEMORY;
```

**证据**: `frameworks/capi/src/game_device_proxy.cpp:29`

### 错误码定义

| 错误码 | 值 | 说明 |
|---------|-----|------|
| GAME_CONTROLLER_SUCCESS | 0 | 成功 |
| GAME_CONTROLLER_PARAM_ERROR | 401 | 参数错误（null、超出范围等）|
| GAME_CONTROLLER_MULTIMODAL_INPUT_ERROR | 32200001 | 多模态输入服务异常 |
| GAME_CONTROLLER_NO_MEMORY | 32200002 | 内存分配失败 |

**证据**: `interfaces/kits/c/game_controller_type.h`

## 同步/异步模式

### 同步 API

**所有 CAPI 均为同步调用**，立即返回结果。

**证据**: 所有 API 定义均返回 `GameController_ErrorCode`，无 Promise/Callback 模式。

### 异步回调模式

**事件监听使用回调注册模式**:
- 设备事件: `GameDevice_DeviceMonitorCallback`
- 按键事件: `GamePad_ButtonInputMonitorCallback`
- 轴事件: `GamePad_AxisInputMonitorCallback`

**回调触发时机**:
- 设备上线/下线时立即触发
- 输入事件发生时立即触发

## 关键结论

1. **当前仅 CAPI**，无 N-API（ArkTS 未来规划）
2. **两套主要 API**：
   - GameDevice: 设备管理和监听
   - GamePad: 手柄输入监听（按键/轴）
3. **同步 API 调用**，异步回调模式用于事件监听
4. **参数校验**：Null 检查、范围校验、内存检查
5. **完整调用链**：C API → C++ Proxy → Framework → MMI/Window Framework

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [03_Architecture.md](./03_Architecture.md) - 完整架构说明
- [05_Inner_API.md](./05_Inner_API.md) - 内部 API 详解

---

**版本**: 1.0 | **更新时间**: 2026-02-06
