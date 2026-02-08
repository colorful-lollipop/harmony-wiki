# 关键调用链

## 概述

本文档展示 miscdevice 子系统的关键 API 调用链，帮助理解从应用层到驱动的完整调用路径。

## 振动 API 调用链

### startVibration (JS API)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: Application                                                      │
└─────────────────────────────────────────────────────────────────────────┘
  JS: startVibration({type: 'preset', effectId: 'haptic.clock.timer'}, attr)
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: JS N-API (vibrator_js.cpp)                                       │
└─────────────────────────────────────────────────────────────────────────┘
  napi_value StartVibrate(napi_env env, napi_callback_info info)
    │
    ├─ Parse parameters: effectId, duration, count
    │    │
    │    ▼
    │  vibrator_napi_utils.cpp::ParseString()  // effectId 解析
    │  vibrator_napi_utils.cpp::ParseInt32()   // duration/count 解析
    │
    ├─ Validate permissions
    │    │
    │    ▼
    │  CheckVibratePermission()  // 权限检查 (权限不足返回 201)
    │
    └─ Call Native Client
         │
         ▼
      vibrator_interface_native::StartVibrator()
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: Native Client (vibrator_service_client.cpp)                      │
└─────────────────────────────────────────────────────────────────────────┘
  auto proxy = GetMiscdeviceServiceProxy();
  if (proxy != nullptr) {
      proxy->StartVibrator(effect, attribute);  // Binder IPC
  }
         │
         ▼ IPC (Binder)
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: System Ability (miscdevice_service.cpp)                           │
└─────────────────────────────────────────────────────────────────────────┘
  int32_t MiscdeviceService::StartVibrator(
      const VibrateEffect &effect,
      const VibrateAttribute &attribute)
    │
    ├─ [1] Permission Check: CheckVibratePermission()
    │    │
    │    ▼
    │  AccessTokenKit::VerifyAccessToken()
    │
    ├─ [2] Priority Check: VibrationPriorityManager::CheckVibration()
    │    │
    │    ▼
    │  GetCallingPackageName()  // 获取调用者包名
    │  IsNewVibrationHighPriority()  // 优先级判断
    │
    ├─ [3] Create VibratorThread (if needed)
    │    │
    │    ▼
    │  VibratorThread::Start()
    │
    └─ [4] Call HDI
         │
         ▼
      HdiConnection::StartVibrator(effect)
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: HDI Adapter (hdi_connection.cpp)                                 │
└─────────────────────────────────────────────────────────────────────────┘
  VibratorHdiConnection::StartVibrator(effect)
    │
    ├─ [1] Parse Effect
    │    │
    │    ▼
    │  ParseHapticPattern()  // 解析振动模式
    │
    ├─ [2] Get HDI (from HDI Pool)
    │    │
    │    ▼
    │  GetVibratorHdi()
    │
    └─ [3] Call Driver
         │
         ▼
      VibratorHdi::StartVibration(pattern)
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Layer: HDF Driver (libvibrator_proxy_2.0.z.so)                          │
└─────────────────────────────────────────────────────────────────────────┘
  Driver calls kernel ioctl()
         │
         ▼
    Kernel vibrator driver
```

### stopVibration (JS API)

```
JS: stopVibration('preset')
    │
    ▼
napi_value Stop(napi_callback_info)
    │
    ▼
vibrator_service_client::StopVibrator(mode)
    │
    ▼ IPC
MiscdeviceService::StopVibratorByMode(mode)
    │
    ├─ Check VibrationPriorityManager
    │    │
    │    ▼
    │  IsNewVibrationHighPriority()
    │
    ├─ Call VibratorThread
    │    │
    │    ▼
    │  VibratorThread::Stop()
    │
    └─ Call HDI
         │
         ▼
      HdiConnection::StopVibrator(mode)
         │
         ▼
      VibratorHdi::StopVibration(mode)
```

### isSupportEffect (JS API)

```
JS: isSupportEffect('haptic.clock.timer', callback)
    │
    ▼
napi_value IsSupportEffect(napi_callback_info)
    │
    ├─ Parse effectId
    │    │
    │    ▼
    │  vibrator_napi_utils.cpp::ParseString()
    │
    └─ Call Native
         │
         ▼
      vibrator_service_client::IsSupportEffect(effectId)
         │
         ▼ IPC
MiscdeviceService::IsSupportEffect(effectId)
    │
    ├─ Permission Check (VIBRATE permission)
    │
    ├─ [Check Feature Flag]
    │    │
    │    ▼ (OHOS_BUILD_ENABLE_VIBRATOR_CUSTOM)
    │  CustomVibrationMatcher::IsEffectSupported()
    │    │
    │    ▼ (else)
    │  LoadPresetEffectInfo()  // 预设效果检查
    │
    └─ Return result
         │
         ▼ IPC Result
    return true/false
```

## Light API 调用链

### TurnOn (Native API)

```
Light Client: light_client.cpp::TurnOn()
    │
    ▼ IPC
MiscdeviceService::TurnOn(lightId, effect)
    │
    ├─ Permission Check
    │    │
    │    ▼
    │  CheckLightPermission()
    │
    ├─ [Check Feature: HDF_DRIVERS_INTERFACE_LIGHT]
    │    │
    │    ▼
    │  HdiLightConnection::TurnOn(lightId)
    │
    └─ Return result
```

## IPC 消息序列化

### MessageParcel Structure

```cpp
// Client (Proxy)
MessageParcel data;
data.WriteInt32(effect.type);
data.WriteInt64(effect.duration);
data.WriteString(effect.effectId);
data.WriteInt32(attribute.usage);
remote->SendRequest(CODE_START_VIBRATE, data, reply, option);

// Server (Stub)
MessageParcel reply;
switch (code) {
    case CODE_START_VIBRATE: {
        int32_t type = data.ReadInt32();
        int64_t duration = data.ReadInt64();
        std::string effectId = data.ReadString();
        // ...
        break;
    }
}
```

## 回调注册流程

### SubscribeVibratorPlugInfo

```
JS: vibrator.on('hapticPlug', callback)
    │
    ▼
napi_value On(napi_callback_info)
    │
    ├─ Parse eventType: 'hapticPlug'
    │
    └─ Call Native
         │
         ▼
      vibrator_service_client::SubscribeVibratorPlugInfo(callback)
         │
         ├─ Create DeathRecipient
         │    │
         │    ▼
         │  RemoteObject::AddDeathRecipient()
         │
         └─ IPC: TransferClientRemoteObject()
              │
              ▼
           MiscdeviceService::TransferClientRemoteObject()
               │
               └─ Store client proxy
                    │
                    ▼
                 VibratorPlugCallback::OnReceive()
                      │
                      ▼
                   NotifyAllClients()  // 回调通知
```

## 调用链关键路径总结

| API | 调用深度 | 关键检查点 |
|-----|---------|-----------|
| startVibration | 6 层 | 权限 → 优先级 → HDI |
| stopVibration | 5 层 | 优先级 → HDI |
| isSupportEffect | 5 层 | 权限 → 效果匹配器 |
| TurnOn | 4 层 | 权限 → HDI |
| Subscribe | 4 层 | DeathRecipient → 远程对象传递 |

## 性能关键路径

### 振动启动延迟

```
延迟来源:
1. N-API 参数解析: ~10-100μs
2. 权限检查 (IPC): ~50-200μs
3. Binder IPC (跨进程): ~100-500μs
4. 优先级判断: ~10-50μs
5. HDI 调用: ~50-200μs

预计总延迟: ~220-1050μs
```

### 建议优化点

| 优化项 | 位置 | 预期收益 |
|-------|------|----------|
| 缓存权限检查结果 | permission_util.cpp | -50-100μs |
| 批量 HDI 调用 | hdi_connection.cpp | -50-100μs |
| 预解析振动模式 | vibrator_napi_utils.cpp | -20-50μs |
