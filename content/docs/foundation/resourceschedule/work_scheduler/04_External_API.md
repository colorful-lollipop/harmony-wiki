# 对外 API 文档

> **目的**: 完整列出 Work Scheduler 的所有对外 N-API 接口
> **适用范围**: 应用开发者、SDK 集成者、测试开发者

---

## API 清单总览

### 主模块 API (`resourceschedule.workScheduler`)

**模块名称**: `resourceschedule.workScheduler`
**命名空间**: `@ohos.resourceschedule.workScheduler`
**加载方式**: `import workScheduler from '@ohos.resourceschedule.workScheduler'`
**异步支持**: Promise 和 Callback 双模式

| API | 说明 | 异步 | 参数类型 |
|-----|------|-------|----------|
| `startWork(workInfo)` | 启动任务 | 否 | `WorkInfo` |
| `stopWork(workInfo, needCancel?)` | 停止任务 | 否 | `WorkInfo`, `boolean` |
| `getWorkStatus(workId)` | 获取任务状态 | ✅ | `number` |
| `obtainAllWorks()` | 获取所有任务 | ✅ | 无 |
| `stopAndClearWorks()` | 清空任务 | 否 | 无 |
| `isLastWorkTimeOut(workId)` | 检查超时 | ✅ | `number` |

### ExtensionAbility API (`WorkSchedulerExtensionAbility`)

**模块名称**: `@ohos.WorkSchedulerExtensionAbility`
**继承关系**: 继承自 `ExtensionAbility`
**用途**: 应用实现此类以接收任务回调

| 回调方法 | 说明 | 触发时机 |
|----------|------|---------|
| `onWorkStart(workInfo)` | 任务开始时回调 | 任务启动前 |
| `onWorkStop(workInfo)` | 任务停止时回调 | 任务完成后 |

### ExtensionContext API (`WorkSchedulerExtensionContext`)

**模块名称**: `application.WorkSchedulerExtensionContext`
**继承关系**: 继承自 `ExtensionContext`
**用途**: Extension 的上下文访问

| 方法 | 说明 | 异步 |
|------|------|-------|
| `startServiceExtensionAbility(want, callback)` | 启动服务扩展 | ✅ |
| `stopServiceExtensionAbility(want, callback)` | 停止服务扩展 | ✅ |

---

## 详细 API 文档

### startWork(workInfo)

**描述**: 注册一个新的延迟任务

**语法**:
```typescript
startWork(workInfo: WorkInfo): void
```

**参数**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `workId` | number | ✅ | 任务唯一标识 |
| `bundleName` | string | ✅ | 应用包名 |
| `abilityName` | string | ✅ | 执行的 Ability 名称 |
| `networkType` | NetworkType | ❌ | 网络条件 |
| `isCharging` | boolean | ❌ | 是否在充电 |
| `chargerType` | ChargingType | ❌ | 充电器类型 |
| `batteryLevel` | number | ❌ | 电池电量阈值 |
| `batteryStatus` | BatteryStatus | ❌ | 电池状态 |
| `storageRequest` | StorageRequest | ❌ | 存储状态 |
| `isRepeat` | boolean | ❌ | 是否重复 |
| `repeatCycleTime` | number | ❌ | 重复周期（分钟，最小 20） |
| `repeatCount` | number | ❌ | 重复次数 |
| `parameters` | `{[key: string]: any}` | ❌ | 自定义参数 |

**参数校验规则**:
- ✅ `workId`, `bundleName`, `abilityName` 必须非空
- ✅ 至少设置一个条件（networkType、isCharging、batteryLevel 等）
- ✅ 如果 `isRepeat` 为 true，必须设置 `repeatCycleTime`
- ✅ `repeatCycleTime` 最小 20 分钟
- ✅ `parameters` 仅支持 `number`、`string`、`boolean` 类型

**错误码**:
| 错误 | 说明 |
|------|------|
| `E_PARAM_NUMBER_ERR` | 参数数量错误 |
| `E_WORK_INFO_TYPE_ERR` | WorkInfo 类型错误 |
| `E_WORKID_ERR` | WorkId 无效 |

**C++ 实现**: `StartWork()` → `WorkSchedulerSrvClient::StartWork()`

---

### stopWork(workInfo, needCancel?)

**描述**: 停止已注册的任务

**语法**:
```typescript
stopWork(workInfo: WorkInfo, needCancel?: boolean): void
```

**参数**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `workInfo` | WorkInfo | ✅ | 任务信息（仅需 workId） |
| `needCancel` | boolean | ❌ | 是否取消任务（默认 false） |

**行为**:
- `needCancel = false`: 停止任务，下次条件满足时仍会触发
- `needCancel = true`: 取消任务，从系统中删除

**C++ 实现**: `StopWork()` → `WorkSchedulerSrvClient::StopWork()` 或 `StopAndCancelWork()`

---

### getWorkStatus(workId)

**描述**: 获取指定任务的当前状态

**语法**:
```typescript
getWorkStatus(workId: number): Promise<WorkInfo>
getWorkStatus(workId: number, callback: AsyncCallback<WorkInfo>): void
```

**返回值** (`Promise<WorkInfo>` 或 `Callback(WorkInfo)`):
| 字段 | 类型 | 说明 |
|------|------|------|
| `workId` | number | 任务 ID |
| `bundleName` | string | 应用包名 |
| `abilityName` | string | Ability 名称 |
| 状态字段 | - | 当前状态和条件 |

**异步模式**:
- **Callback**: `getWorkStatus(workId, (err, data) => { ... })`
- **Promise**: `const data = await getWorkStatus(workId)`

**C++ 实现**: `GetWorkStatus()` (使用 `napi_create_async_work`)

---

### obtainAllWorks()

**描述**: 获取调用者所有已注册的任务

**语法**:
```typescript
obtainAllWorks(): Promise<Array<WorkInfo>>
obtainAllWorks(callback: AsyncCallback<Array<WorkInfo>>): void
```

**返回值** (`Array<WorkInfo>`):
- 调用者的所有任务列表
- 按 UID 过滤（仅返回当前调用者的任务）

**异步模式**:
- **Callback**: `obtainAllWorks((err, data) => { ... })`
- **Promise**: `const data = await obtainAllWorks()`

**C++ 实现**: `ObtainAllWorks()` (使用 `napi_create_async_work`)

---

### stopAndClearWorks()

**描述**: 停止并清空调用者的所有任务

**语法**:
```typescript
stopAndClearWorks(): void
```

**行为**:
- 停止调用者所有正在执行的任务
- 从调度队列中移除所有任务
- 不影响其他应用的任务

**C++ 实现**: `StopAndClearWorks()` → `WorkSchedulerSrvClient::StopAndClearWorks()`

---

### isLastWorkTimeOut(workId)

**描述**: 检查重复任务的最后一次执行是否超时

**语法**:
```typescript
isLastWorkTimeOut(workId: number): Promise<boolean>
isLastWorkTimeOut(workId: number, callback: AsyncCallback<boolean>): void
```

**返回值** (`boolean`):
- `true`: 上次执行超时（超过 120 秒）
- `false`: 上次执行正常完成或未执行

**适用场景**:
- 仅对重复任务（`isRepeat = true`）有效
- 用于决定是否调整下次执行时间

**异步模式**:
- **Callback**: `isLastWorkTimeOut(workId, (err, data) => { ... })`
- **Promise**: `const data = await isLastWorkTimeOut(workId)`

**C++ 实现**: `IsLastWorkTimeOut()` (使用 `napi_create_async_work`)

---

## 枚举类型

### NetworkType

**描述**: 网络类型条件

**使用**: `workInfo.networkType`

**值**:
| 常量 | 值 | 说明 |
|--------|-----|------|
| `NETWORK_TYPE_ANY` | 0 | 任意网络 |
| `NETWORK_TYPE_MOBILE` | 1 | 移动数据网络 |
| `NETWORK_TYPE_WIFI` | 2 | WiFi 网络 |
| `NETWORK_TYPE_BLUETOOTH` | 3 | 蓝牙网络 |
| `NETWORK_TYPE_WIFI_P2P` | 4 | WiFi P2P 网络 |
| `NETWORK_TYPE_ETHERNET` | 5 | 以太网 |

**C++ 枚举**: `WorkCondition::Network` (`work_condition.h`)

---

### ChargingType

**描述**: 充电器类型条件

**使用**: `workInfo.chargerType`

**值**:
| 常量 | 值 | 说明 |
|--------|-----|------|
| `CHARGING_PLUGGED_ANY` | 0 | 任意充电器 |
| `CHARGING_PLUGGED_AC` | 1 | AC 充电器 |
| `CHARGING_PLUGGED_USB` | 2 | USB 充电器 |
| `CHARGING_PLUGGED_WIRELESS` | 3 | 无线充电器 |

**C++ 枚举**: `WorkCondition::Charger` (`work_condition.h`)

---

### BatteryStatus

**描述**: 电池状态条件

**使用**: `workInfo.batteryStatus`

**值**:
| 常量 | 值 | 说明 |
|--------|-----|------|
| `BATTERY_STATUS_LOW` | 0 | 低电量 |
| `BATTERY_STATUS_OKAY` | 1 | 电量正常 |
| `BATTERY_STATUS_LOW_OR_OKAY` | 2 | 低电量或正常 |

**C++ 枚举**: `WorkCondition::BatteryStatus` (`work_condition.h`)

---

### StorageRequest

**描述**: 存储状态条件

**使用**: `workInfo.storageRequest`

**值**:
| 常量 | 值 | 说明 |
|--------|-----|------|
| `STORAGE_LEVEL_LOW` | 0 | 存储空间低 |
| `STORAGE_LEVEL_OKAY` | 1 | 存储空间正常 |
| `STORAGE_LEVEL_LOW_OR_OKAY` | 2 | 存储空间低或正常 |

**C++ 枚举**: `WorkCondition::Storage` (`work_condition.h`)

---

## Extension 回调

### onWorkStart(workInfo)

**触发时机**: 任务即将执行前

**用途**:
- 初始化任务资源
- 记录任务开始时间
- 准备任务参数

**参数**:
- `workInfo: WorkInfo` - 完整的任务信息（包含 parameters）

### onWorkStop(workInfo)

**触发时机**: 任务完成后

**用途**:
- 释放任务资源
- 记录任务结束时间
- 更新任务状态

**参数**:
- `workInfo: WorkInfo` - 完整的任务信息

---

## 错误处理

### 错误码映射

| JS 错误 | C++ 错误码 | 说明 |
|-----------|-----------|------|
| 参数错误 | `E_PARAM_NUMBER_ERR` | 参数数量不正确 |
| 参数类型错误 | `E_WORK_INFO_TYPE_ERR` | WorkInfo 类型错误 |
| WorkId 错误 | `E_WORKID_ERR` | WorkId 无效 |
| 回调类型错误 | `E_CALLBACK_TYPE_ERR` | 回调函数类型错误 |
| 系统错误 | `ERR_OK` | 成功 |

### 错误抛出方式

所有错误通过 `Common::HandleErrCode()` 或 `Common::HandleParamErr()` 抛出：
- 同步方法：直接抛出异常
- 异步方法：通过回调的第一个参数传递错误

---

## C++ 入口映射

| JS API | C++ 函数 | 文件 | 行号 |
|---------|-----------|------|------|
| `startWork` | `StartWork()` | `start_work.cpp:27` | 27 |
| `stopWork` | `StopWork()` | `stop_work.cpp:28` | 28 |
| `getWorkStatus` | `GetWorkStatus()` | `get_work_status.cpp:72` | 72 |
| `obtainAllWorks` | `ObtainAllWorks()` | `obtain_all_works.cpp:59` | 59 |
| `stopAndClearWorks` | `StopAndClearWorks()` | `stop_and_clear_works.cpp:23` | 23 |
| `isLastWorkTimeOut` | `IsLastWorkTimeOut()` | `is_last_work_time_out.cpp:105` | 105 |

**模块注册点**:
- **InitApi()** (`init.cpp:41`) - 定义所有导出方法
- **RegisterModule()** (`init.cpp:254-257`) - 构造函数注册

**NAPI 模块定义**:
```cpp
napi_module g_apiModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitApi,
    .nm_modname = "resourceschedule.workScheduler",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

---

## 使用示例

### 基础任务

```typescript
import workScheduler from '@ohos.resourceschedule.workScheduler'

// 创建任务
const workInfo = {
  workId: 1,
  bundleName: 'com.example.app',
  abilityName: 'WorkAbility',
  networkType: workScheduler.NetworkType.NETWORK_TYPE_WIFI,
  isCharging: true
}

// 启动任务
workScheduler.startWork(workInfo)
```

### 重复任务

```typescript
const workInfo = {
  workId: 2,
  bundleName: 'com.example.app',
  abilityName: 'WorkAbility',
  networkType: workScheduler.NetworkType.NETWORK_TYPE_WIFI,
  isRepeat: true,
  repeatCycleTime: 30,  // 每30分钟
  repeatCount: 5       // 重复5次
}

workScheduler.startWork(workInfo)
```

### 异步查询

```typescript
// Promise 模式
const workInfo = await workScheduler.getWorkStatus(1)
console.log(`Task status: ${JSON.stringify(workInfo)}`)

// Callback 模式
workScheduler.getWorkStatus(1, (err, data) => {
  if (err) {
    console.error(`Error: ${err}`)
    return
  }
  console.log(`Task status: ${JSON.stringify(data)}`)
})
```

### 清空任务

```typescript
workScheduler.stopAndClearWorks()
```

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 核心概念和使用场景
- [01_Positioning.md](01_Positioning.md) - 项目定位和边界
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链详解

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| API 注册点 | `interfaces/kits/js/napi/src/init.cpp:256` (`napi_module_register`) |
| API 方法定义 | `interfaces/kits/js/napi/src/init.cpp:44-51` (`napi_define_properties`) |
| 异步实现 | `interfaces/kits/js/napi/src/get_work_status.cpp:100-121` (`napi_create_async_work`) |
| 枚举类型 | `interfaces/kits/js/napi/src/init.cpp:64-102` (`InitNetworkType` 等) |
| BUILD.gn 配置 | `interfaces/kits/js/BUILD.gn:22-68` (workscheduler target) |
