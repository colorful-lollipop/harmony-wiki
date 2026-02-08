# 关键调用链

**目的**: 记录模块的关键调用链，便于理解和调试

---

## 拨号流程调用链

### 入口 → 核心处理

```
JS 层
  │
  ▼
@ohos.telephony.call.d.ts
  - dialCall(phoneNumber, options)
  │
  ▼
N-API 绑定层
  │
  ├── 位置: frameworks/js/napi/src/native_module.cpp
  └── RegisterCallManagerFunc()
  │
  ▼
CallManagerService (SA 4005)
  │
  ├── 位置: services/call_manager_service/src/call_manager_service.cpp
  ├── OnStart() [行 151]
  ├── Dial() [行 303]
  │
  ▼
CallControlManager
  │
  ├── 位置: services/call/src/call_control_manager.cpp
  ├── ProcessDialRequest()
  ├── CheckPermission() [PLACE_CALL]
  ├── ValidatePhoneNumber()
  │
  ▼
Telephony Core Service
  │
  ├── 位置: telephony_interaction/src/core_service_connection.cpp
  └── Dial(slotId, phoneNumber, videoState)
```

### 权限检查路径

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  JS API     │────▶│  N-API      │────▶│  SA Service │
│ dialCall()  │     │  Binding    │     │  Handler    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                              ┌────────────────┼────────────────┐
                              ▼                ▼                ▼
                    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
                    │ PLACE_CALL   │ │ ANSWER_CALL  │ │ GET_TELEPHONY│
                    │ 拨号权限      │ │ 接听/挂断权限 │ │ STATE 查询权限│
                    └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
                           │                │                │
                           └────────────────┼────────────────┘
                                            ▼
                              ┌───────────────────────┐
                              │ TelephonyPermission   │
                              │ ::CheckPermission()   │
                              └───────────────────────┘
```

---

## 通话状态变更流程

```
通话事件发生
    │
    ▼
telephony_core_service
    │
    ├── CallStateChanged()
    │
    ▼
CallStatusListener
    │
    ├── 位置: services/call/src/call_state_listener.cpp
    └── OnCallStateChanged()
    │
    ▼
CallStatusManager
    │
    ├── 位置: services/call/src/call_status_manager.cpp
    ├── UpdateCallState()
    └── NotifyCallback()
    │
    ▼
CallAbilityCallbackProxy
    │
    ├── 位置: services/call_report/src/call_ability_callback_proxy.cpp
    └── ReportCallStateChanged()
    │
    ▼
App (UI 回调)
```

---

## 音频设备切换流程

```
音频事件
    │
    ▼
Audio Framework
    │
    ├── AudioDeviceChangeCallback
    │
    ▼
AudioProxy
    │
    ├── 位置: services/audio/src/audio_proxy.cpp
    └── OnAudioDeviceChanged()
    │
    ▼
AudioDeviceManager
    │
    ├── 位置: services/audio/src/audio_device_manager.cpp
    └── HandleDeviceChange()
    │
    ▼
AudioStateProcessor
    │
    └── 切换音频状态
        │
        ▼
    CallManager
        │
        └── 调整通话音频路由
```

---

## 蓝牙通话处理流程

```
蓝牙设备操作
    │
    ▼
Bluetooth Framework
    │
    ├── HFP 回调
    │
    ▼
BluetoothCallService
    │
    ├── 位置: services/bluetooth/src/bluetooth_call_service.cpp
    └── ProcessBluetoothEvent()
    │
    ▼
BluetoothCallManager
    │
    ├── 位置: services/bluetooth/src/bluetooth_call_manager.cpp
    └── OnBluetoothScoConnected()
    │
    ▼
Audio Manager
    │
    └── 切换音频到蓝牙设备
```

---

## 相关文档

- [架构说明](../01_Architecture.md)
- [API 参考](../02_API_Reference.md)
