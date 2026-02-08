# sensors_miscdevice_lite API 文档

## ⚠️ 重要声明

**本文档基于标准系统实现推断**，Lite 版本当前无代码实现。

实际 N-API 实现请参考：
- **主实现仓库**: https://github.com/openharmony/sensors_miscdevice
- **API 文档**: https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-vibrator

---

## 3.1 Vibrator API

### 3.1.1 API 清单

基于 OpenHarmony 标准系统 `vibrator` 模块，推断的 API 如下：

| JS API | 命名空间 | 类型 | 同步/异步 | 实现状态 |
|--------|----------|------|-----------|----------|
| `startVibration()` | vibrator | 方法 | 异步 | ❌ 未实现 |
| `stopVibration()` | vibrator | 方法 | 同步 | ❌ 未实现 |
| `isSupportEffect()` | vibrator | 方法 | 异步 | ❌ 未实现 |

**命名空间**: `@kit.SensorServiceKit` 或 `vibrator`

---

### 3.1.2 startVibration

启动设备振动。

#### 函数签名

```typescript
// 异步回调模式
startVibration(effect: VibrateEffect, attribute: VibratorAttribute, callback: AsyncCallback<void>): void

// Promise 模式
startVibration(effect: VibrateEffect, attribute: VibratorAttribute): Promise<void>
```

#### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `effect` | `VibrateEffect` | 是 | 振动效果配置 |
| `attribute` | `VibratorAttribute` | 是 | 振动属性（用途、ID等） |
| `callback` | `AsyncCallback<void>` | 否 | 异步回调 |

#### VibrateEffect 类型

```typescript
type VibrateEffect = VibrateTime | VibratePreset | VibrateFromFile

// 定时振动
interface VibrateTime {
    type: 'time';
    duration: number;  // 振动时长 (ms)
}

// 预设振动
interface VibratePreset {
    type: 'preset';
    effectId: string;  // 预设效果ID，如 'haptic.clock.timer'
}

// 自定义振动（从文件）
interface VibrateFromFile {
    type: 'file';
    hapticFd: number;  // Haptic 配置文件描述符
}
```

#### VibratorAttribute 类型

```typescript
interface VibratorAttribute {
    usage?: {           // 振动用途（影响行为）
        scenario: VibratorScenario;
        flags?: number;
    };
    id?: number;        // 振动器 ID
}
```

#### VibratorScenario 枚举

| 枚举值 | 说明 |
|--------|------|
| `VIBRATOR_SCENARIO_UNKNOWN` | 未知场景 |
| `VIBRATOR_SCENARIO_NOTIFICATION` | 通知 |
| `VIBRATOR_SCENARIO_ALARM` | 闹钟 |
| `VIBRATOR_SCENARIO_RING` | 来电响铃 |
| `VIBRATOR_SCENARIO_INTERACTION` | 交互反馈 |
| `VIBRATOR_SCENARIO_GAME` | 游戏 |

#### 使用示例

```typescript
import { vibrator } from '@kit.SensorServiceKit';
import { BusinessError } from '@kit.BasicServicesKit';

// 定时振动 (Promise)
try {
    await vibrator.startVibration(
        { type: 'time', duration: 1000 },
        { usage: { scenario: vibrator.VibratorScenario.VIBRATOR_SCENARIO_NOTIFICATION } }
    );
    console.info('Vibration started successfully');
} catch (error) {
    const err = error as BusinessError;
    console.error(`Failed to start vibration: ${err.code}, ${err.message}`);
}

// 预设振动效果 (Callback)
vibrator.startVibration(
    { type: 'preset', effectId: 'haptic.clock.timer' },
    { usage: { scenario: vibrator.VibratorScenario.VIBRATOR_SCENARIO_ALARM } },
    (error: BusinessError) => {
        if (error) {
            console.error(`Failed: ${error.code}, ${error.message}`);
            return;
        }
        console.info('Preset vibration started');
    }
);
```

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 14600101 | 设备不支持 |
| 14600102 | 振动器忙碌 |
| 14600103 | 权限不足 |

---

### 3.1.3 stopVibration

停止设备振动。

#### 函数签名

```typescript
// 指定模式停止
stopVibration(stopMode: VibratorStopMode): void

// 停止所有
stopVibration(): void
```

#### VibratorStopMode 枚举

| 枚举值 | 说明 |
|--------|------|
| `VIBRATOR_STOP_MODE_TIME` | 停止定时振动 |
| `VIBRATOR_STOP_MODE_PRESET` | 停止预设振动 |
| `VIBRATOR_STOP_MODE_ALL` | 停止所有振动 |

#### 使用示例

```typescript
import { vibrator } from '@kit.SensorServiceKit';

// 停止定时振动
vibrator.stopVibration(vibrator.VibratorStopMode.VIBRATOR_STOP_MODE_TIME);

// 停止所有振动
vibrator.stopVibration();
```

---

### 3.1.4 isSupportEffect

查询指定振动效果是否支持。

#### 函数签名

```typescript
isSupportEffect(effectId: string, callback: AsyncCallback<boolean>): void
isSupportEffect(effectId: string): Promise<boolean>
```

#### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `effectId` | string | 是 | 要查询的效果 ID |
| `callback` | AsyncCallback | 否 | 回调函数 |

#### 使用示例

```typescript
import { vibrator } from '@kit.SensorServiceKit';
import { BusinessError } from '@kit.BasicServicesKit';

// 使用 Promise
const isSupported = await vibrator.isSupportEffect('haptic.clock.timer');
if (isSupported) {
    console.info('Effect is supported');
} else {
    console.info('Effect is not supported');
}

// 使用回调
vibrator.isSupportEffect('haptic.clock.timer', (err: BusinessError, state: boolean) => {
    if (err) {
        console.error(`Query failed: ${err.code}, ${err.message}`);
        return;
    }
    console.info(`Effect support state: ${state}`);
});
```

---

## 3.2 LED API（推断）

### 3.2.1 API 清单

| JS API | 描述 | 实现状态 |
|--------|------|----------|
| `turnOnLed()` | 打开 LED 灯 | ❌ 未实现 |
| `turnOffLed()` | 关闭 LED 灯 | ❌ 未实现 |
| `setLedBrightness()` | 设置 LED 亮度 | ❌ 未实现 |
| `getLedList()` | 获取 LED 列表 | ❌ 未实现 |

**注意**: LED API 详细规范请参考主实现仓库 `sensors_miscdevice`。

---

## 3.3 权限配置

### 3.3.1 权限声明

| 权限名 | 级别 | 用途 |
|--------|------|------|
| `ohos.permission.VIBRATE` | system_grant | 使用振动功能 |

#### 权限申请方式

**方式一：声明式（module.json5）**

```json
{
    "requestPermissions": [
        {
            "name": "ohos.permission.VIBRATE",
            "reason": "Need vibrator for notification feedback",
            "usedScene": {
                "abilities": ["EntryAbility"],
                "when": "inuse"
            }
        }
    ]
}
```

**方式二：动态申请（API 9+）**

```typescript
import { abilityAccessCtrl, bundleManager, Permissions } from '@kit.AbilityKit';

// 检查权限状态
const atManager = abilityAccessCtrl.createAtManager();
const tokenId = bundleManager.getApplicationInfoSync('com.example.app').accessTokenId;
const permissionState = atManager.checkAccessTokenSync(tokenId, 'ohos.permission.VIBRATE');

if (permissionState === abilityAccessCtrl.GrantState.PERMISSION_GRANTED) {
    // 已有权限
} else {
    // 申请权限
    const permissions: Permissions[] = ['ohos.permission.VIBRATE'];
    atManager.requestPermissionFromUser(permissions, (err, data) => {
        // 处理结果
    });
}
```

---

## 3.4 C/C++ 绑定入口（推断）

### 3.4.1 N-API 注册点

基于 OpenHarmony 标准实现，N-API 注册应位于：

| 路径（推断） | 文件 | 函数 |
|-------------|------|------|
| `interfaces/plugin/` | `napi_vibrator.cpp` | `NAPI_MODULE()` 宏注册 |
| `interfaces/plugin/` | `napi_led.cpp` | `NAPI_MODULE()` 宏注册 |

### 3.4.2 N-API 典型结构

```cpp
// 推断的 N-API 注册结构
EXTERN_C_START
static napi_module_register(napi_env env, napi_value exports) {
    napi_value vibratorModule = nullptr;
    napi_create_object(env, &vibratorModule);
    
    // 注册 startVibration
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("startVibration", StartVibration),
        DECLARE_NAPI_FUNCTION("stopVibration", StopVibration),
        DECLARE_NAPI_FUNCTION("isSupportEffect", IsSupportEffect),
    };
    
    napi_define_properties(env, vibratorModule, sizeof(desc) / sizeof(desc[0]), desc);
    
    // 导出模块
    napi_set_namedProperty(env, exports, "vibrator", vibratorModule);
}
EXTERN_C_END

NAPI_MODULE(vibrator, vibrator_module_register)
```

---

## 3.5 错误码汇总

### 3.5.1 通用错误码

| 错误码 | 宏定义（推断） | 说明 |
|--------|---------------|------|
| 401 | `PARAMETER_ERROR` | 参数错误 |
| 14600101 | `SENSOR_NOT_SUPPORTED` | 设备不支持 |
| 14600102 | `SENSOR_BUSY` | 振动器忙碌 |
| 14600103 | `PERMISSION_DENIED` | 权限不足 |
| 14600104 | `SYSTEM_ERROR` | 系统错误 |

### 3.5.2 错误处理最佳实践

```typescript
import { vibrator } from '@kit.SensorServiceKit';
import { BusinessError } from '@kit.BasicServicesKit';

try {
    vibrator.startVibration({ type: 'time', duration: 1000 }, {});
} catch (error) {
    const err = error as BusinessError;
    switch (err.code) {
        case 401:
            console.error('Invalid parameters');
            break;
        case 14600101:
            console.error('Vibrator not supported');
            break;
        case 14600103:
            console.error('Permission denied');
            break;
        default:
            console.error(`Unknown error: ${err.code}`);
    }
}
```

---

## 3.6 相关文档

### 官方资源
- [Vibrator API 参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-vibrator)
- [sensors_miscdevice GitHub](https://github.com/openharmony/sensors_miscdevice)
- [权限管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/runtime-permissions-0000001774280914)

### 本地文档
- [02_Architecture.md](./02_Architecture.md) - 架构说明
- [05_Security.md](./05_Security.md) - 安全评审
- [06_Troubleshooting.md](./06_Troubleshooting.md) - 常见问题
