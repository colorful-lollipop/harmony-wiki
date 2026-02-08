# API 接口文档

## 概述

miscdevice 提供多语言 API 支持：

| 语言 | 绑定层 | 头文件/模块 |
|------|--------|------------|
| JavaScript/ArkTS | JS N-API | `@ohos.vibrator` |
| C | C API | `ohvibrator.h` |
| ArkTS | Taihe | `ohos.vibrator` (IDL) |
| Cangjie | FFI | `vibrator_ffi.cpp` |

## JS/ArkTS API (N-API)

### 模块注册

**文件**: `frameworks/js/napi/vibrator/src/vibrator_js.cpp`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "vibrator",
};

extern "C" __attribute__((constructor)) void RegisterModule(void) {
    napi_module_register(&_module);
}
```

### API 清单

| JS API | C++ 实现 | 参数 | 返回 | 同步/异步 |
|--------|----------|------|------|----------|
| `startVibration(effect, attribute, callback?)` | `StartVibrate()` | VibrateEffect, VibrateAttribute | void | Async/Callback |
| `stopVibration(stopMode, callback?)` | `Stop()` | VibratorStopMode | void | Async/Callback |
| `stopVibrationSync(stopMode)` | `StopVibrationSync()` | VibratorStopMode | void | Sync |
| `isHdHapticSupported()` | `IsHdHapticSupported()` | - | boolean | Async/Callback |
| `isSupportEffect(effectId, callback)` | `IsSupportEffect()` | string | boolean | Async/Callback |
| `isSupportEffectSync(effectId)` | `IsSupportEffectSync()` | string | boolean | Sync |
| `getVibratorInfoSync()` | `GetVibratorListSync()` | - | VibratorInfo[] | Sync |
| `getEffectInfoSync(effectId)` | `GetSupportEffectInfoSync()` | string | EffectInfo | Sync |
| `on(eventType, callback)` | `On()` | string, function | void | Async |
| `off(eventType)` | `Off()` | string | void | Sync |

### VibrateEffect 类型

```typescript
// 时间模式
type: 'time',
duration: number;  // 毫秒

// 预设模式  
type: 'preset',
effectId: string;  // 如 'haptic.clock.timer'
count: number;     // 重复次数
```

### VibrateAttribute 属性

```typescript
{
    id: number;      // 振动器 ID
    usage: string;   // 用途: 'alarm' | 'notification' | 'ring' | 'unknown' | 'touch'
}
```

### VibratorStopMode 停止模式

| 模式 | 说明 |
|------|------|
| `VIBRATOR_STOP_MODE_TIME` | 停止定时振动 |
| `VIBRATOR_STOP_MODE_PRESET` | 停止预设振动 |

### 使用示例

```typescript
import vibrator from '@ohos.vibrator';

// 定时振动 (1秒)
vibrator.startVibration({
    type: 'time',
    duration: 1000,
}, {
    id: 0,
    usage: 'alarm'
}, (error) => {
    if (error) {
        console.error('振动失败:', error.code, error.message);
    }
});

// 预设振动
vibrator.startVibration({
    type: 'preset',
    effectId: 'haptic.clock.timer',
    count: 1,
}, {
    usage: 'unknown'
}).then(() => {
    console.log('振动成功');
}).catch((error) => {
    console.error('振动失败:', error);
});

// 停止振动
vibrator.stopVibration('preset');

// 查询支持的振动效果
vibrator.isSupportEffect('haptic.clock.timer', (err, supported) => {
    console.log('是否支持:', supported);
});
```

## C API (NDK)

### 头文件

- `interfaces/kits/c/vibrator.h` - C 接口声明
- `interfaces/kits/c/vibrator_type.h` - 类型定义

### C API 清单

| C API | 说明 | 参数 |
|-------|------|------|
| `OH_Vibrator_StartVibration()` | 启动振动 | effect, attribute |
| `OH_Vibrator_StopVibration()` | 停止振动 | stopMode |
| `OH_Vibrator_IsHdHapticSupported()` | 检查 HD 触觉支持 | - |
| `OH_Vibrator_IsSupportEffect()` | 检查效果支持 | effectId |
| `OH_Vibrator_GetVibratorInfo()` | 获取振动器信息 | vibratorInfo |
| `OH_Vibrator_GetEffectInfo()` | 获取效果信息 | effectId, effectInfo |

### 错误码

```c
// interfaces/kits/c/vibrator_type.h
#define VIBRATOR_SUCCESS             0
#define VIBRATOR_ERROR               -1
#define PERMISSION_DENIED            201
#define PARAMETER_ERROR              401
#define NO_SUPPORT                   202
```

### C 使用示例

```c
#include "ohvibrator.h"

// 启动振动
Vibrator_Parameter param = {
    .type = VIBRATOR_TYPE_TIME,
    .duration = 1000,
};
Vibrator_Attribute attr = {
    .usage = VIBRATOR_USAGE_ALARM,
};
int32_t ret = OH_Vibrator_StartVibration(&param, &attr);
if (ret != VIBRATOR_SUCCESS) {
    // 处理错误
}

// 停止振动
OH_Vibrator_StopVibration(VIBRATOR_STOP_MODE_TIME);
```

## Taihe (ArkTS)

### IDL 定义

**文件**: `frameworks/ets/taihe/idl/ohos.vibrator.taihe`

```typescript
// 命名空间
@!namespace("@ohos.vibrator", "vibrator")

// 核心 API
declare function startVibrationSync(effect: VibrateEffect, attribute: VibrateAttribute): void;
declare function stopVibrationByModeSync(mode: VibratorStopMode): void;
declare function isHdHapticSupported(): boolean;
declare function getVibratorInfoSync(): VibratorInfo[];
declare function getVibratorPatternBuilder(): VibratorPatternBuilder;
```

### VibratorPatternBuilder

```typescript
class VibratorPatternBuilder {
    constructor(duration: number)
    setIntensity(intensity: number): VibratorPatternBuilder
    setDuration(duration: number): VibratorPatternBuilder
    setLoopCount(count: number): VibratorPatternBuilder
    build(): VibrateEffect
}
```

## Cangjie FFI

### 导出函数

**文件**: `frameworks/cj/src/vibrator_ffi.cpp`

```cpp
extern "C" {
    void FfiVibratorStartVibrationTime(RetVibrateTime effect, RetVibrateAttribute attribute, int32_t &code);
    void FfiVibratorStartVibrationPreset(RetVibratePreset effect, RetVibrateAttribute attribute, int32_t &code);
    void FfiVibratorStartVibrationFile(RetVibrateFromFile effect, RetVibrateAttribute attribute, int32_t &code);
    void FfiVibratorStopVibration(int32_t &code);
    void FfiVibratorStopVibrationMode(char* vibMode, int32_t &code);
    bool FfiVibratorSupportEffect(char* id, int32_t &code);
    bool FfiVibratorIsHdHapticSupported();
}
```

## 参数校验

### JS API 参数校验

**文件**: `frameworks/js/napi/vibrator/src/vibrator_napi_utils.cpp`

| API | 校验项 | 错误码 |
|-----|--------|--------|
| `startVibration` | effect 不可为 null, duration > 0 | 401 (PARAMETER_ERROR) |
| `stopVibration` | stopMode 有效枚举 | 401 |
| `isSupportEffect` | effectId 长度限制 | 401 |

### 权限检查

```cpp
// services/miscdevice_service/src/miscdevice_service.cpp
PermissionUtil &permissionUtil = PermissionUtil::GetInstance();
int32_t ret = permissionUtil.CheckVibratePermission(callerToken, "ohos.permission.VIBRATE");
if (ret != PERMISSION_GRANTED) {
    return PERMISSION_DENIED;  // 201
}
```

## 内部 API (Inner API)

### Vibrator Agent

**文件**: `interfaces/inner_api/vibrator/vibrator_agent.h`

| 接口 | 说明 |
|------|------|
| `StartVibrator()` | 启动振动 |
| `StopVibrator()` | 停止振动 |
| `StopVibratorByMode()` | 按模式停止 |
| `GetVibratorInfo()` | 获取振动器列表 |
| `GetEffectInfo()` | 获取效果信息 |
| `SubscribeVibratorPlugInfo()` | 订阅插拔事件 |
| `TransferClientRemoteObject()` | 传递客户端远程对象 |

### Light Agent

**文件**: `interfaces/inner_api/light/light_agent.h`

| 接口 | 说明 |
|------|------|
| `TurnOn()` | 开启灯光 |
| `TurnOff()` | 关闭灯光 |
| `GetLightInfo()` | 获取灯光信息 |
| `PlayLightEffect()` | 播放灯光效果 |
| `StopLightEffect()` | 停止灯光效果 |
