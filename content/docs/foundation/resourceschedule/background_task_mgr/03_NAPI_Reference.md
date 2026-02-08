# N-API接口参考

## 模块概述

后台任务管理模块提供两个N-API模块供JS/ArkTS应用调用：

| 模块名 | 模块路径 | 说明 |
|--------|----------|------|
| `backgroundTaskManager` | `@ohos.backgroundTaskManager` | 旧版模块（兼容） |
| `resourceschedule.backgroundTaskManager` | `@ohos.resourceschedule.backgroundTaskManager` | 主模块（推荐） |

**导入方式**:
```typescript
// 推荐
import backgroundTaskManager from '@ohos.resourceschedule.backgroundTaskManager';

// 旧版兼容
import backgroundTaskManager from '@ohos.backgroundTaskManager';
```

---

## API总览

### 1. 短时任务接口 (Transient Task)

| API | 同步/异步 | 说明 | 起始版本 |
|-----|-----------|------|----------|
| `requestSuspendDelay` | 同步 | 申请短时任务延迟挂起 | 9+ |
| `cancelSuspendDelay` | 同步 | 取消短时任务 | 9+ |
| `getRemainingDelayTime` | Promise/Callback | 获取剩余延迟时间 | 9+ |
| `getTransientTaskInfo` | Promise | 获取所有短时任务信息 | 11+ |

### 2. 长时任务接口 (Continuous Task)

| API | 同步/异步 | 说明 | 起始版本 |
|-----|-----------|------|----------|
| `startBackgroundRunning` | Promise/Callback | 启动长时任务 | 9+ |
| `updateBackgroundRunning` | Promise/Callback | 更新长时任务 | 11+ |
| `stopBackgroundRunning` | Promise/Callback | 停止长时任务 | 9+ |
| `getAllContinuousTasks` | Promise | 获取所有长时任务 | 9+ |
| `obtainAllContinuousTasks` | Promise | 获取所有长时任务(含挂起) | 11+ |

### 3. 能效资源接口 (Efficiency Resources)

| API | 同步/异步 | 说明 | 起始版本 |
|-----|-----------|------|----------|
| `applyEfficiencyResources` | 同步 | 申请能效资源 | 9+ |
| `resetAllEfficiencyResources` | 同步 | 释放所有能效资源 | 9+ |
| `getAllEfficiencyResources` | Promise | 获取所有资源申请 | 9+ |

### 4. 状态管理接口

| API | 同步/异步 | 说明 | 起始版本 |
|-----|-----------|------|----------|
| `setBackgroundTaskState` | Promise | 设置任务状态 | 11+ |
| `getBackgroundTaskState` | Promise | 获取任务状态 | 11+ |
| `subscribeContinuousTaskState` | 同步 | 订阅任务状态变更 | 11+ |
| `unsubscribeContinuousTaskState` | 同步 | 取消订阅 | 11+ |
| `on` / `off` | 同步 | 事件监听 | 11+ |

---

## 短时任务接口

### requestSuspendDelay

**功能**: 申请短时任务，延迟应用挂起

**声明**:
```typescript
function requestSuspendDelay(
    reason: string, 
    callback: Callback<void>
): DelaySuspendInfo;
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| reason | string | 是 | 申请原因，最大128字符 |
| callback | Callback<void> | 是 | 超时回调函数 |

**返回值**: `DelaySuspendInfo`
| 属性 | 类型 | 说明 |
|------|------|------|
| requestId | number | 请求ID，用于取消和查询 |
| actualDelayTime | number | 实际延迟时间(毫秒) |

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误(reason为空或超长) |
| 980000401 | 系统未就绪 |
| 990000101 | 无效的UID或PID |
| 990000201 | 回调无效 |
| 990000301 | 序列化失败 |
| 990000401 | 服务未连接 |

**C++绑定位置**: `interfaces/kits/napi/src/request_suspend_delay.cpp`

**调用链**:
```
requestSuspendDelay()
    ↓ interfaces/kits/napi/src/request_suspend_delay.cpp:90
NAPI调用
    ↓ frameworks/src/background_task_manager.cpp:63
BackgroundTaskManager::RequestSuspendDelay()
    ↓ IPC
services/transient_task/src/bg_transient_task_mgr.cpp:191
BgTransientTaskMgr::RequestSuspendDelay()
```

**示例**:
```typescript
import backgroundTaskManager from '@ohos.resourceschedule.backgroundTaskManager';

try {
    const delayInfo = backgroundTaskManager.requestSuspendDelay('Uploading data', () => {
        console.log('Task will expire soon');
        backgroundTaskManager.cancelSuspendDelay(delayInfo.requestId);
    });
    console.log(`Request ID: ${delayInfo.requestId}, Delay: ${delayInfo.actualDelayTime}ms`);
} catch (error) {
    console.error(`Failed: ${error.code}, ${error.message}`);
}
```

---

### cancelSuspendDelay

**功能**: 取消短时任务

**声明**:
```typescript
function cancelSuspendDelay(requestId: number): void;
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| requestId | number | 是 | requestSuspendDelay返回的ID |

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误(requestId无效) |
| 990000401 | 服务未连接 |

**C++绑定位置**: `interfaces/kits/napi/src/cancel_suspend_delay.cpp`

---

### getRemainingDelayTime

**功能**: 获取短时任务剩余时间

**声明**:
```typescript
function getRemainingDelayTime(requestId: number): Promise<number>;
function getRemainingDelayTime(requestId: number, callback: AsyncCallback<number>): void;
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| requestId | number | 是 | 请求ID |
| callback | AsyncCallback<number> | 否 | 回调形式 |

**返回值**: Promise<number> - 剩余时间(毫秒)

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 990000401 | 服务未连接 |

**C++绑定位置**: `interfaces/kits/napi/src/get_remaining_delay_time.cpp`

---

## 长时任务接口

### startBackgroundRunning

**功能**: 申请长时任务，保持后台运行

**声明**:
```typescript
function startBackgroundRunning(
    context: Context,
    bgMode: BackgroundMode,
    wantAgent: WantAgent
): Promise<void>;

function startBackgroundRunning(
    context: Context,
    bgMode: BackgroundMode,
    wantAgent: WantAgent,
    callback: AsyncCallback<void>
): void;
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| context | Context | 是 | 应用上下文 |
| bgMode | BackgroundMode | 是 | 后台模式 |
| wantAgent | WantAgent | 是 | 通知点击跳转 |
| callback | AsyncCallback<void> | 否 | 回调函数 |

**BackgroundMode枚举**:
| 名称 | 值 | 说明 | 权限 |
|------|-----|------|------|
| DATA_TRANSFER | 0 | 数据传输 | 普通 |
| AUDIO_PLAYBACK | 1 | 音频播放 | 普通 |
| AUDIO_RECORDING | 2 | 音频录制 | 普通 |
| LOCATION | 3 | 定位导航 | 普通 |
| BLUETOOTH_INTERACTION | 4 | 蓝牙传输 | 普通 |
| MULTI_DEVICE_CONNECTION | 5 | 分布式互联 | 普通 |
| WIFI_INTERACTION | 6 | WLAN传输 | SystemApi |
| VOIP | 7 | 音视频通话 | SystemApi |
| TASK_KEEPING | 8 | 计算任务 | 特定设备 |

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 权限拒绝(需要ohos.permission.KEEP_BACKGROUND_RUNNING) |
| 401 | 参数错误 |
| 980000401 | 系统未就绪 |
| 980000501 | 任务已存在 |
| 980000502 | 无效的后台模式 |
| 980000601 | 通知验证失败 |

**C++绑定位置**: `interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp:542`

**调用链**:
```
startBackgroundRunning()
    ↓ interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp
ParseContinuousTaskParam() 参数解析
    ↓ frameworks/src/background_task_manager.cpp
BackgroundTaskManager::RequestStartBackgroundRunning()
    ↓ IPC
services/continuous_task/src/bg_continuous_task_mgr.cpp
BgContinuousTaskMgr::StartBackgroundRunning()
    ↓ CheckPermission() 权限检查
StartBackgroundRunningInner()
    ↓ SendContinuousTaskNotification() 发送通知
```

**示例**:
```typescript
import backgroundTaskManager from '@ohos.resourceschedule.backgroundTaskManager';
import wantAgent from '@ohos.app.ability.wantAgent';

async function startBackgroundMode(context: Context) {
    const wantAgentInfo = {
        wants: [
            {
                bundleName: 'com.example.myapp',
                abilityName: 'EntryAbility'
            }
        ],
        operationType: wantAgent.OperationType.START_ABILITY,
        requestCode: 0
    };
    
    const agent = await wantAgent.getWantAgent(wantAgentInfo);
    
    try {
        await backgroundTaskManager.startBackgroundRunning(
            context,
            backgroundTaskManager.BackgroundMode.AUDIO_PLAYBACK,
            agent
        );
        console.log('Background running started');
    } catch (error) {
        console.error(`Failed: ${error.code}, ${error.message}`);
    }
}
```

---

### stopBackgroundRunning

**功能**: 停止长时任务

**声明**:
```typescript
function stopBackgroundRunning(context: Context): Promise<void>;
function stopBackgroundRunning(context: Context, callback: AsyncCallback<void>): void;
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| context | Context | 是 | 应用上下文 |
| callback | AsyncCallback<void> | 否 | 回调函数 |

**C++绑定位置**: `interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp:703`

---

### updateBackgroundRunning

**功能**: 更新长时任务（支持批量模式/子模式）

**声明**:
```typescript
function updateBackgroundRunning(
    context: Context,
    bgMode: BackgroundMode,
    wantAgent: WantAgent
): Promise<void>;
```

**C++绑定位置**: `interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp:619`

---

## 能效资源接口

### applyEfficiencyResources

**功能**: 申请能效资源特权

**声明**:
```typescript
function applyEfficiencyResources(request: EfficiencyResourcesRequest): void;
```

**参数** - `EfficiencyResourcesRequest`:
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceTypes | number | 是 | 资源类型位掩码 |
| isApply | boolean | 是 | true申请/false释放 |
| timeOut | number | 是 | 超时时间(毫秒) |
| reason | string | 是 | 申请原因 |
| isPersist | boolean | 否 | 是否持久化 |
| isProcess | boolean | 否 | 是否进程级 |
| cpuLevel | number | 否 | CPU级别(1=小,2=中,3=大) |

**ResourceType枚举**:
| 名称 | 值 | 说明 |
|------|-----|------|
| CPU | 1 | CPU资源，申请后不被挂起 |
| COMMON_EVENT | 2 | 公共事件挂起不被代理 |
| TIMER | 4 | 计时器挂起不被代理 |
| WORK_SCHEDULER | 8 | 延迟任务更长执行时间 |
| BLUETOOTH | 16 | 蓝牙挂起不被代理 |
| GPS | 32 | GPS挂起不被代理 |
| AUDIO | 64 | 音频挂起不被代理 |
| RUNNING_LOCK | 128 | 运行锁 |
| SENSOR | 256 | 传感器 |

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 1870000101 | 资源申请超限 |
| 1870000102 | 无效PID/UID |

**C++绑定位置**: `interfaces/kits/napi/src/efficiency_resources_operation.cpp:165`

**调用链**:
```
applyEfficiencyResources()
    ↓ interfaces/kits/napi/src/efficiency_resources_operation.cpp:165
ParseParameters() 参数解析与校验
    ↓ frameworks/src/background_task_manager.cpp
BackgroundTaskManager::ApplyEfficiencyResources()
    ↓ IPC
services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp:160
BgEfficiencyResourcesMgr::ApplyEfficiencyResources()
```

**示例**:
```typescript
import backgroundTaskManager from '@ohos.resourceschedule.backgroundTaskManager';

try {
    backgroundTaskManager.applyEfficiencyResources({
        resourceTypes: backgroundTaskManager.ResourceType.CPU | 
                       backgroundTaskManager.ResourceType.GPS,
        isApply: true,
        timeOut: 600000,  // 10分钟
        reason: 'Navigation task',
        isPersist: false,
        isProcess: false,
        cpuLevel: backgroundTaskManager.EfficiencyResourcesCpuLevel.MEDIUM_CPU
    });
} catch (error) {
    console.error(`Failed: ${error.code}, ${error.message}`);
}
```

---

### resetAllEfficiencyResources

**功能**: 释放所有能效资源

**声明**:
```typescript
function resetAllEfficiencyResources(): void;
```

**C++绑定位置**: `interfaces/kits/napi/src/efficiency_resources_operation.cpp:178`

---

### getAllEfficiencyResources

**功能**: 获取所有能效资源申请信息

**声明**:
```typescript
function getAllEfficiencyResources(): Promise<Array<EfficiencyResourcesInfo>>;
```

**返回值**: `EfficiencyResourcesInfo[]`
| 属性 | 类型 | 说明 |
|------|------|------|
| resourceNumber | number | 资源类型掩码 |
| isApply | boolean | 是否申请 |
| timeOut | number | 超时时间 |
| reason | string | 原因 |
| isPersist | boolean | 是否持久化 |
| isProcess | boolean | 是否进程级 |

**C++绑定位置**: `interfaces/kits/napi/src/efficiency_resources_operation.cpp:227`

---

## 枚举与常量

### BackgroundMode

```typescript
enum BackgroundMode {
    DATA_TRANSFER = 0,           // 数据传输
    AUDIO_PLAYBACK = 1,          // 音频播放
    AUDIO_RECORDING = 2,         // 音频录制
    LOCATION = 3,                // 定位导航
    BLUETOOTH_INTERACTION = 4,   // 蓝牙传输
    MULTI_DEVICE_CONNECTION = 5, // 分布式互联
    WIFI_INTERACTION = 6,        // WLAN传输 (SystemApi)
    VOIP = 7,                    // 音视频通话 (SystemApi)
    TASK_KEEPING = 8             // 计算任务
}
```

**导出位置**: `interfaces/kits/napi/src/init_bgtaskmgr.cpp:107-133`

### ResourceType

```typescript
enum ResourceType {
    CPU = 1,              // 0b000000001
    COMMON_EVENT = 2,     // 0b000000010
    TIMER = 4,            // 0b00000100
    WORK_SCHEDULER = 8,   // 0b00001000
    BLUETOOTH = 16,       // 0b00010000
    GPS = 32,             // 0b00100000
    AUDIO = 64,           // 0b01000000
    RUNNING_LOCK = 128,   // 0b10000000
    SENSOR = 256          // 0b100000000
}
```

**导出位置**: `interfaces/kits/napi/src/init_bgtaskmgr.cpp:126-133`

### ContinuousTaskCancelReason

```typescript
enum ContinuousTaskCancelReason {
    USER_CANCEL = 0,
    SYSTEM_CANCEL = 1,
    USER_CANCEL_REMOVE_NOTIFICATION = 2,
    SYSTEM_CANCEL_DATA_TRANSFER_LOW_SPEED = 3,
    SYSTEM_CANCEL_AUDIO_PLAYBACK_NOT_USE_AVSESSION = 4,
    SYSTEM_CANCEL_AUDIO_PLAYBACK_NOT_RUNNING = 5,
    SYSTEM_CANCEL_AUDIO_RECORDING_NOT_RUNNING = 6,
    SYSTEM_CANCEL_NOT_USE_LOCATION = 7,
    SYSTEM_CANCEL_NOT_USE_BLUETOOTH = 8,
    SYSTEM_CANCEL_NOT_USE_MULTI_DEVICE = 9,
    SYSTEM_CANCEL_USE_ILLEGALLY = 10
}
```

**导出位置**: `interfaces/kits/napi/src/init_bgtaskmgr.cpp:155-188`

### ContinuousTaskSuspendReason

```typescript
enum ContinuousTaskSuspendReason {
    SYSTEM_SUSPEND_DATA_TRANSFER_LOW_SPEED = 0,
    SYSTEM_SUSPEND_AUDIO_PLAYBACK_NOT_USE_AVSESSION = 1,
    SYSTEM_SUSPEND_AUDIO_PLAYBACK_NOT_RUNNING = 2,
    SYSTEM_SUSPEND_AUDIO_RECORDING_NOT_RUNNING = 3,
    SYSTEM_SUSPEND_LOCATION_NOT_USED = 4,
    SYSTEM_SUSPEND_BLUETOOTH_NOT_USED = 5,
    SYSTEM_SUSPEND_MULTI_DEVICE_NOT_USED = 6,
    SYSTEM_SUSPEND_USED_ILLEGALLY = 7,
    SYSTEM_SUSPEND_SYSTEM_LOAD_WARNING = 8,
    SYSTEM_SUSPEND_VOIP_NOT_USED = 9
}
```

**导出位置**: `interfaces/kits/napi/src/init_bgtaskmgr.cpp:190-223`

### EfficiencyResourcesCpuLevel

```typescript
enum EfficiencyResourcesCpuLevel {
    SMALL_CPU = 1,   // 小核
    MEDIUM_CPU = 2,  // 中核
    LARGE_CPU = 3    // 大核
}
```

**导出位置**: `interfaces/kits/napi/src/init_bgtaskmgr.cpp:139-141`

---

## 错误码汇总

### 通用错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_BGTASK_PERMISSION_DENIED | 201 | 权限被拒绝 |
| ERR_BGTASK_NOT_SYSTEM_APP | 202 | 非系统应用 |
| ERR_BGTASK_INVALID_PARAM | 401 | 参数错误 |
| ERR_BGTASK_NO_MEMORY | 980000101 | 内存不足 |
| ERR_BGTASK_PARCELABLE_FAILED | 980000201 | 序列化失败 |
| ERR_BGTASK_TRANSACT_FAILED | 980000301 | IPC调用失败 |
| ERR_BGTASK_SYS_NOT_READY | 980000401 | 系统未就绪 |
| ERR_BGTASK_SERVICE_NOT_CONNECTED | 980000402 | 服务未连接 |

### 长时任务错误码 (9800005xx/9800006xx)

| 错误码 | 说明 |
|--------|------|
| ERR_BGTASK_OBJECT_EXISTS | 对象已存在 |
| ERR_BGTASK_OBJECT_NOT_EXIST | 对象不存在 |
| ERR_BGTASK_INVALID_BGMODE | 无效的后台模式 |
| ERR_BGTASK_CONTINUOUS_REQUEST_NULL_OR_TYPE | 请求参数为空或类型错误 |
| ERR_BGTASK_NOTIFICATION_VERIFY_FAILED | 通知验证失败 |
| ERR_BGTASK_CONTINUOUS_AUTH_NOT_PERMITTED | 授权未通过 |

### 短时任务错误码 (9900001xx/9900002xx)

| 错误码 | 说明 |
|--------|------|
| ERR_BGTASK_INVALID_PID_OR_UID | 无效的PID或UID |
| ERR_BGTASK_INVALID_BUNDLE_NAME | 无效的Bundle名称 |
| ERR_BGTASK_EXCEEDS_THRESHOLD | 超出配额限制 |
| ERR_BGTASK_TIME_INSUFFICIENT | 时间不足 |

### 能效资源错误码 (1870000xxx)

| 错误码 | 说明 |
|--------|------|
| ERR_BGTASK_RESOURCES_EXCEEDS_MAX | 资源申请超限 |
| ERR_BGTASK_RESOURCES_INVALID_PID_OR_UID | 无效PID/UID |

---

## 参数校验规则

### requestSuspendDelay参数校验

| 参数 | 校验规则 | 错误码 |
|------|----------|--------|
| reason | 非空，长度≤128字符 | ERR_REASON_NULL_OR_TYPE_ERR |
| callback | 必须为函数类型 | ERR_CALLBACK_NULL_OR_TYPE_ERR |

### startBackgroundRunning参数校验

| 参数 | 校验规则 | 错误码 |
|------|----------|--------|
| context | 非空，必须为Context类型 | ERR_CONTEXT_NULL_OR_TYPE_ERR |
| bgMode | 必须为有效BackgroundMode值 | ERR_BGMODE_NULL_OR_TYPE_ERR |
| wantAgent | 非空，必须为WantAgent类型 | ERR_WANTAGENT_NULL_OR_TYPE_ERR |

### applyEfficiencyResources参数校验

| 参数 | 校验规则 | 错误码 |
|------|----------|--------|
| resourceTypes | 非零，有效位掩码 | ERR_RESOURCE_TYPES_INVALID |
| isApply | 必须为布尔值 | ERR_ISAPPLY_NULL_OR_TYPE_ERR |
| timeOut | 非负整数 | ERR_TIMEOUT_INVALID |
| reason | 非空字符串 | ERR_REASON_NULL_OR_TYPE_ERR |
| isPersist | 布尔值（可选） | ERR_ISPERSIST_NULL_OR_TYPE_ERR |
| isProcess | 布尔值（可选） | ERR_ISPROCESS_NULL_OR_TYPE_ERR |

---

## 权限要求

| API | 权限 | 权限级别 |
|-----|------|----------|
| requestSuspendDelay | 无 | - |
| startBackgroundRunning | ohos.permission.KEEP_BACKGROUND_RUNNING | normal |
| stopBackgroundRunning | 无 | - |
| applyEfficiencyResources | 无 | - |

---

## 相关文档

- [架构说明](02_Architecture.md) - 调用链详细说明
- [内部API](04_Inner_API.md) - C++接口层
- [安全风险](06_Security.md) - 接口安全分析
