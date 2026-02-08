# N-API 接口参考

本文档提供 Power Manager 模块的 N-API（JavaScript/TypeScript）接口完整参考。

## 模块列表

| 模块名 | JS 命名空间 | 描述 |
|--------|------------|------|
| `@ohos.power` | `power` | 电源管理核心功能 |
| `@ohos.runningLock` | `runningLock` | 运行锁管理功能 |

---

## 一、Power 模块 (`@ohos.power`)

### 1.1 模块概述

**注册位置**: `frameworks/napi/power/power_module.cpp:148-162`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "power",
    .nm_register_func = PowerInit,
    .nm_modname = "power",
};

extern "C" __attribute__((constructor)) void RegisterPowerModule(void)
{
    napi_module_register(&g_module);
}
```

**导出结构**:
- 模块名: `power`
- 依赖: `@ohos.power.ndk` (Native 扩展)

### 1.2 API 清单

#### 1.2.1 关机与重启

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `shutdown(reason: string)` | `PowerNapi::Shutdown` | 异步 (Promise) | 9+ | 活跃 |
| `reboot(reason: string)` | `PowerNapi::Reboot` | 异步 (Promise) | 9+ | 活跃 |
| `shutdownDevice(reason: string)` | `Power::ShutdownDevice` | 异步 (Callback) | - | **废弃** |
| `rebootDevice(reason: string)` | `Power::RebootDevice` | 异步 (Callback) | - | **废弃** |

**代码位置**: `frameworks/napi/power/power_napi.cpp`

##### shutdown

```typescript
import { power } from '@ohos.power';

// 关机
power.shutdown('user').then(() => {
    console.log('关机请求已提交');
}).catch((error) => {
    console.error('关机失败:', error.code, error.message);
});
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `reason` | string | 是 | 关机原因，如 "user" |

**返回值**: `Promise<void>`

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数无效 |
| 4603001 | IPC 通信失败 |

##### reboot

```typescript
// 重启
power.reboot('user').then(() => {
    console.log('重启请求已提交');
}).catch((error) => {
    console.error('重启失败:', error);
});
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `reason` | string | 是 | 重启原因，如 "user" |

**返回值**: `Promise<void>`

#### 1.2.2 设备状态

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `isActive()` | `PowerNapi::IsActive` | 同步 | 9+ | 活跃 |
| `isStandby()` | `PowerNapi::IsStandby` | 同步 | 9+ | 活跃 |
| `isScreenOn()` | `Power::IsScreenOn` | 同步 | - | **废弃** |

##### isActive

```typescript
// 检查设备是否处于活跃状态
const isActive = power.isActive();
console.log('设备是否活跃:', isActive);
```

**返回值**: `boolean` - true 表示设备活跃（未休眠）

##### isStandby

```typescript
// 检查设备是否处于待机状态
const isStandby = power.isStandby();
console.log('设备是否待机:', isStandby);
```

**返回值**: `boolean` - true 表示设备处于待机状态

#### 1.2.3 唤醒与挂起

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `wakeup(deviceId: string, reason: string)` | `PowerNapi::Wakeup` | 同步 | 9+ | 活跃 |
| `suspend(immediate: boolean)` | `PowerNapi::Suspend` | 同步 | 9+ | 活跃 |
| `wakeupDevice(deviceId: string, reason: string)` | `Power::WakeupDevice` | 同步 | - | **废弃** |
| `suspendDevice(immediate: boolean)` | `Power::SuspendDevice` | 同步 | - | **废弃** |

##### wakeup

```typescript
// 唤醒设备
power.wakeup('', 'powerkey');
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `deviceId` | string | 否 | 设备 ID，空字符串表示默认设备 |
| `reason` | string | 是 | 唤醒原因，如 "powerkey", "wakeup" |

##### suspend

```typescript
// 挂起设备
power.suspend(true);  // 立即挂起

power.suspend(false); // 延迟挂起（按超时设置）
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `immediate` | boolean | 是 | true: 立即挂起; false: 按超时设置延迟挂起 |

#### 1.2.4 休眠

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `hibernate(immediate: boolean)` | `PowerNapi::Hibernate` | 同步 | 9+ | 活跃 |

##### hibernate

```typescript
// 进入休眠
power.hibernate(true);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `immediate` | boolean | 是 | true: 立即休眠; false: 延迟休眠 |

**返回值**: `boolean` - true 表示休眠请求已发送

#### 1.2.5 电源模式

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `setPowerMode(mode: DevicePowerMode)` | `PowerNapi::SetPowerMode` | 同步 | 9+ | 活跃 |
| `getPowerMode()` | `PowerNapi::GetPowerMode` | 同步 | 9+ | 活跃 |

##### setPowerMode

```typescript
import { power, DevicePowerMode } from '@ohos.power';

// 设置电源模式
power.setPowerMode(DevicePowerMode.MODE_POWER_SAVE);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `mode` | DevicePowerMode | 是 | 电源模式枚举 |

##### getPowerMode

```typescript
// 获取当前电源模式
const mode = power.getPowerMode();
console.log('当前电源模式:', mode);
```

**返回值**: `DevicePowerMode`

#### 1.2.6 屏幕控制

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `setScreenOffTime(time: number)` | `PowerNapi::SetScreenOffTime` | 同步 | 9+ | 活跃 |
| `refreshActivity(reason: string)` | `PowerNapi::RefreshActivity` | 同步 | 9+ | 活跃 |

##### setScreenOffTime

```typescript
// 设置屏幕自动关闭时间（毫秒）
power.setScreenOffTime(30000);  // 30秒

// 恢复默认设置
power.setScreenOffTime(-1);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `time` | number | 是 | 超时时间(毫秒)，-1 表示恢复默认 |

##### refreshActivity

```typescript
// 刷新用户活动（阻止屏幕关闭）
power.refreshActivity('user_activity');
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `reason` | string | 是 | 刷新原因，如 "user_activity" |

#### 1.2.7 电源键过滤策略

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `setPowerKeyFilteringStrategy(strategy: PowerKeyFilteringStrategy)` | `PowerNapi::SetPowerKeyFilteringStrategy` | 同步 | 9+ | 活跃 |

##### setPowerKeyFilteringStrategy

```typescript
import { power, PowerKeyFilteringStrategy } from '@ohos.power';

// 设置电源键长按过滤策略
power.setPowerKeyFilteringStrategy(
    PowerKeyFilteringStrategy.DISABLE_LONG_PRESS_FILTERING
);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `strategy` | PowerKeyFilteringStrategy | 是 | 过滤策略枚举 |

#### 1.2.8 关机回调

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `registerShutdownCallback(callback: AsyncShutdownCallback)` | `PowerNapi::RegisterShutdownCallback` | 异步 | 9+ | 活跃 |
| `unregisterShutdownCallback(callbackId: number)` | `PowerNapi::UnRegisterShutdownCallback` | 异步 | 9+ | 活跃 |

##### registerShutdownCallback

```typescript
// 注册关机回调
const callbackId = await power.registerShutdownCallback((reason) => {
    console.log('设备即将关机:', reason);
});
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `callback` | AsyncShutdownCallback | 是 | 关机回调函数 |

**返回值**: `Promise<number>` - 回调 ID

##### unregisterShutdownCallback

```typescript
// 取消关机回调
await power.unregisterShutdownCallback(callbackId);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `callbackId` | number | 是 | 注册时返回的回调 ID |

### 1.3 枚举类型

#### DevicePowerMode

```typescript
enum DevicePowerMode {
    MODE_NORMAL = 0,           // 正常模式
    MODE_POWER_SAVE = 1,      // 省电模式
    MODE_PERFORMANCE = 2,     // 性能模式
    MODE_EXTREME_POWER_SAVE = 3, // 超级省电模式
    MODE_CUSTOM_POWER_SAVE = 4, // 自定义省电模式
}
```

**代码位置**: `frameworks/napi/power/power_module.cpp:41-69`

#### PowerKeyFilteringStrategy

```typescript
enum PowerKeyFilteringStrategy {
    DISABLE_LONG_PRESS_FILTERING = 0,  // 禁用长按过滤
    LONG_PRESS_FILTERING_ONCE = 1,     // 长按过滤一次
}
```

**代码位置**: `frameworks/napi/power/power_module.cpp:71-104`

### 1.4 错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 401 | `ERR_PARAM_INVALID` | 参数无效 |
| 4603001 | `ERR_IPC` | IPC 通信失败 |
| 4603002 | `ERR_SERVICE_NOT_READY` | 服务未就绪 |

**代码位置**: `frameworks/napi/utils/napi_errors.h`

---

## 二、RunningLock 模块 (`@ohos.runningLock`)

### 2.1 模块概述

**注册位置**: `frameworks/napi/runninglock/runninglock_module.cpp:150-164`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "runningLock",
    .nm_register_func = RunningLockInit,
    .nm_modname = "runningLock",
};

extern "C" __attribute__((constructor)) void RegisterRunninglockModule(void)
{
    napi_module_register(&g_module);
}
```

**导出结构**:
- 模块名: `runningLock`
- 导出类: `RunningLock`
- 导出枚举: `RunningLockType`

### 2.2 API 清单

#### 2.2.1 运行锁创建

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `create(name: string, type: RunningLockType)` | `Create` | 异步 (Promise) | 9+ | 活跃 |
| `createRunningLock(name: string, type: RunningLockType)` | `CreateRunningLock` | 异步 (Callback) | - | **废弃** |

##### create

```typescript
import { runningLock, RunningLock, RunningLockType } from '@ohos.runningLock';

// 创建运行锁
const lock = await runningLock.create('MyBackgroundTask', RunningLockType.BACKGROUND);

// 使用运行锁
lock.hold(0);  // 无限期保持
// ... 业务逻辑
lock.unhold();  // 释放锁
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 运行锁名称 |
| `type` | RunningLockType | 是 | 运行锁类型 |

**返回值**: `Promise<RunningLock>` - 运行锁实例

#### 2.2.2 类型检查

| JS API | C++ 实现 | 同步/异步 | API 级别 | 状态 |
|--------|----------|----------|---------|------|
| `isSupported(type: RunningLockType)` | `RunningLockNapi::IsSupported` | 同步 | 9+ | 活跃 |
| `isRunningLockTypeSupported(type: RunningLockType)` | `RunningLockInterface::IsRunningLockTypeSupported` | 同步 | - | **废弃** |

##### isSupported

```typescript
// 检查运行锁类型是否支持
const isSupported = runningLock.isSupported(RunningLockType.BACKGROUND_USER_IDLE);
console.log('是否支持:', isSupported);
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | RunningLockType | 是 | 运行锁类型 |

**返回值**: `boolean`

### 2.3 RunningLock 类

**类定义位置**: `frameworks/napi/runninglock/runninglock_module.cpp:56-76`

```typescript
class RunningLock {
    // 新 API (API 9+)
    hold(timeout: number): boolean;
    isHolding(): boolean;
    unhold(): void;

    // 旧 API (废弃)
    lock(timeout: number): boolean;
    isUsed(): boolean;
    unlock(): void;
}
```

#### hold

```typescript
// 无限期保持运行锁
lock.hold(0);

// 限时保持（毫秒）
lock.hold(30000);  // 30秒后自动释放
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `timeout` | number | 是 | 超时时间(毫秒)，0 表示无限期 |

**返回值**: `boolean` - true 表示成功

#### isHolding

```typescript
// 检查锁是否仍保持
const isHolding = lock.isHolding();
console.log('锁是否保持:', isHolding);
```

**返回值**: `boolean`

#### unhold

```typescript
// 释放运行锁
lock.unhold();
```

### 2.4 枚举类型

#### RunningLockType

```typescript
enum RunningLockType {
    BACKGROUND = 1,                    // 后台保活锁
    PROXIMITY_SCREEN_CONTROL = 2,      // 距离感应锁（控制屏幕）
    BACKGROUND_USER_IDLE = 3,          // 后台用户空闲检测锁
}
```

**代码位置**: `frameworks/napi/runninglock/runninglock_module.cpp:88-110`

---

## 三、异步操作模式

### 3.1 Promise 模式

新 API (API 9+) 使用 Promise 模式：

```typescript
// Promise 链式调用
power.shutdown('user')
    .then(() => console.log('关机成功'))
    .catch((error) => console.error('失败:', error));

// async/await 模式
async function shutdownDevice() {
    try {
        await power.shutdown('user');
        console.log('关机请求已发送');
    } catch (error) {
        console.error('关机失败:', error);
    }
}
```

### 3.2 Callback 模式

旧 API (已废弃) 使用 Callback 模式：

```typescript
// 旧 API 模式（不推荐）
power.shutdownDevice('user', (error) => {
    if (error) {
        console.error('关机失败:', error);
    } else {
        console.log('关机成功');
    }
});
```

### 3.3 线程模型

N-API 异步操作使用 UV 线程池：

```
JS 线程 → NAPI 异步工作 → FFRT 线程池 → IPC → Service
```

**代码位置**: `frameworks/napi/utils/async_callback_info.cpp`

---

## 四、权限要求

### 4.1 必需权限

| API | 权限 | 说明 |
|-----|------|------|
| `shutdown` | `ohos.permission.SHUTDOWN` | 关机权限 |
| `reboot` | `ohos.permission.REBOOT` | 重启权限 |
| `wakeup` | 无 | - |
| `suspend` | 无 | - |
| `setPowerMode` | `ohos.permission.POWER_MANAGER` | 电源管理权限 |
| `create(RunningLock)` | 无 | 自动检查调用方 |

### 4.2 权限检查位置

**代码位置**: `utils/permission/permission.cpp`

```cpp
// 权限检查示例
int32_t CheckPermission(const std::string& permissionName)
{
    // 使用 access_token 框架检查权限
}
```

---

## 五、使用示例

### 5.1 完整的电源管理示例

```typescript
import { power, runningLock, RunningLock, RunningLockType, DevicePowerMode } from '@ohos.power';

class PowerManagerDemo {
    private backgroundLock: RunningLock | null = null;

    // 1. 保持后台任务运行
    async holdBackgroundTask(taskName: string) {
        this.backgroundLock = await runningLock.create(
            taskName,
            RunningLockType.BACKGROUND
        );
        this.backgroundLock.hold(0);  // 无限期保持
        console.log('后台任务锁已获取');
    }

    // 2. 释放后台任务锁
    releaseBackgroundTask() {
        this.backgroundLock?.unhold();
        this.backgroundLock = null;
        console.log('后台任务锁已释放');
    }

    // 3. 检查设备状态
    async checkDeviceStatus() {
        const isActive = power.isActive();
        const isStandby = power.isStandby();
        console.log(`活跃: ${isActive}, 待机: ${isStandby}`);
    }

    // 4. 设置省电模式
    async enablePowerSave() {
        await power.setPowerMode(DevicePowerMode.MODE_POWER_SAVE);
        console.log('已切换到省电模式');
    }
}
```

### 5.2 监听电源状态变化

```typescript
// 注册电源状态回调
import { power } from '@ohos.power';

class PowerStateMonitor {
    async monitor() {
        // 注册关机回调
        const callbackId = await power.registerShutdownCallback((reason) => {
            console.log('即将关机:', reason);
            // 保存数据
        });

        // 监听运行锁变化（通过其他模块）
        // ...
    }
}
```

---

## 六、API 变更历史

| API | 变更版本 | 变更内容 |
|-----|----------|----------|
| `shutdownDevice` | API 9 | 标记废弃，推荐使用 `shutdown` |
| `rebootDevice` | API 9 | 标记废弃，推荐使用 `reboot` |
| `createRunningLock` | API 9 | 标记废弃，推荐使用 `create` |
| `shutdown` | API 9 | 新增 Promise 模式 |
| `reboot` | API 9 | 新增 Promise 模式 |
| `hibernate` | API 9 | 新增 |
| `setPowerMode` | API 9 | 新增 |
| `getPowerMode` | API 9 | 新增 |
| `isStandby` | API 9 | 新增 |
| `setScreenOffTime` | API 9 | 新增 |
| `refreshActivity` | API 9 | 新增 |
| `setPowerKeyFilteringStrategy` | API 9 | 新增 |
| `registerShutdownCallback` | API 9 | 新增 |

---

## 七、注意事项

1. **异步操作**: 关机、重启等操作是异步的，需要通过 Promise 处理结果
2. **权限要求**: 关机、重启、电源模式设置需要相应权限
3. **运行锁**: 使用完毕后必须释放，否则会导致设备无法休眠
4. **废弃 API**: 旧 API 在 API 9 标记废弃，但仍然可用

---

## 八、相关文档

- [内部 C++ API](04_Inner_API.md)
- [SA/IPC 架构](05_SA_IPC.md)
- [构建配置](06_Build.md)
