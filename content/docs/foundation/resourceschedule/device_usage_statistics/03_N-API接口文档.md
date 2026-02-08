# N-API 接口文档

## 目的

本文档提供 device_usage_statistics 组件对外 N-API 接口的完整清单，包括 API 名称、参数、返回值、同步/异步模式、错误码等。

## 适用范围

- 新版 NAPI 模块: `@ohos.resourceschedule.usageStatistics`
- 旧版 NAPI 模块: `@ohos.bundleState`
- 所有 JS 接口和对应的 C++ 实现

---

## 接口清单

### 新版模块 (@ohos.resourceschedule.usageStatistics)

#### 模块注册

**模块名**: `resourceschedule.usageStatistics`

**注册点**: `frameworks/src/usage_statistics_init.cpp:158-161`

**证据**:
- interfaces/kits/bundlestats/napi/include/usage_statistics_init.h:42
- frameworks/src/usage_statistics_init.cpp:33-62

---

#### API 列表

##### 1. isIdleState

**功能**: 判断指定应用当前是否空闲

**签名**:
```typescript
// 异步 Callback 版本
isIdleState(bundleName: string, callback: AsyncCallback<boolean>): void

// 异步 Promise 版本
isIdleState(bundleName: string): Promise<boolean>

// 同步版本
isIdleStateSync(bundleName: string): boolean
```

**参数**:
- `bundleName` (string): 应用包名

**返回值**:
- `boolean`: true 表示空闲，false 表示非空闲

**C++ 实现**:
- `IsIdleState()` - frameworks/src/bundle_state_query_napi.cpp:96
- `IsIdleStateSync()` - frameworks/src/bundle_state_query_napi.cpp:281

**参数校验**:
- frameworks/src/bundle_state_query_napi.cpp:158-202: ParseIsIdleStateParameters()

**错误码**:
- 401003: ERR_PARAMETERS_EMPTY - bundleName 为空
- 401004: ERR_BUNDLE_NAME_TYPE - bundleName 类型错误
- 1000000304: ERR_SYSTEM_SERVICES_NOT_READY - 系统服务未就绪

---

##### 2. queryAppGroup

**功能**: 查询应用使用优先级分组

**签名**:
```typescript
// 异步 Callback 版本
queryAppGroup(bundleName: string, callback: AsyncCallback<number>): void

// 异步 Promise 版本
queryAppGroup(bundleName?: string): Promise<number>

// 同步版本
queryAppGroupSync(bundleName?: string): number
```

**参数**:
- `bundleName` (string?, 可选): 应用包名，不传则查询当前应用

**返回值**:
- `number`: 分组类型（10=活跃组, 20=每日组, 30=固定组, 40=罕见组, 50=受限组, 60=从不组）

**C++ 实现**:
- `QueryAppGroup()` - frameworks/src/bundle_active_app_group_napi.cpp:137
- `QueryAppGroupSync()` - frameworks/src/bundle_active_app_group_napi.cpp:190

**参数校验**:
- frameworks/src/bundle_active_app_group_napi.cpp:74-136: ParseQueryAppGroupParameters()

**错误码**:
- 1000000304: ERR_SYSTEM_SERVICES_NOT_READY

---

##### 3. queryCurrentBundleEvents

**功能**: 查询当前应用的事件集合

**签名**:
```typescript
queryCurrentBundleEvents(begin: number, end: number): Promise<Array<BundleActiveEvent>>
```

**参数**:
- `begin` (number): 开始时间（毫秒时间戳）
- `end` (number): 结束时间（毫秒时间戳）

**返回值**:
- `Array<BundleActiveEvent>`: 事件对象数组

**事件对象结构**:
```typescript
{
    bundleName: string,
    eventId: number,
    eventOccurredTime: number,
    stateOccurredTime: number,
    stateType: number
}
```

**C++ 实现**:
- `QueryCurrentBundleEvents()` - frameworks/src/bundle_state_query_napi.cpp:367

**参数校验**:
- frameworks/src/bundle_state_query_napi.cpp:296-366: ParseQueryCurrentBundleEventsParameters()

**错误码**:
- 401005: ERR_BEGIN_TIME_TYPE - beginTime 类型错误
- 401006: ERR_BEGIN_TIME_LESS_THEN_ZERO - beginTime < 0
- 401007: ERR_END_TIME_TYPE - endTime 类型错误
- 401008: ERR_END_TIME_LESS_THEN_BEGIN_TIME - endTime <= beginTime

---

##### 4. queryBundleEvents

**功能**: 查询所有应用的事件集合

**签名**:
```typescript
queryBundleEvents(begin: number, end: number): Promise<Array<BundleActiveEvent>>
```

**参数**:
- `begin` (number): 开始时间（毫秒时间戳）
- `end` (number): 结束时间（毫秒时间戳）

**返回值**:
- `Array<BundleActiveEvent>`: 事件对象数组

**C++ 实现**:
- `QueryBundleEvents()` - frameworks/src/bundle_state_query_napi.cpp:424

**参数校验**:
- frameworks/src/bundle_state_query_napi.cpp:367-423: 参数校验

**错误码**:
- 同 queryCurrentBundleEvents

---

##### 5. queryBundleStatsInfoByInterval

**功能**: 按时间间隔查询应用使用时长统计

**签名**:
```typescript
queryBundleStatsInfoByInterval(
    byInterval: IntervalType,
    begin: number,
    end: number
): Promise<Array<BundleStatsInfo>>
```

**参数**:
- `byInterval` (IntervalType): 时间间隔类型
  - `IntervalType.BY_OPTIMIZED` - 自动优化
  - `IntervalType.BY_DAILY` - 每日
  - `IntervalType.BY_WEEKLY` - 每周
  - `IntervalType.BY_MONTHLY` - 每月
  - `IntervalType.BY_ANNUALLY` - 每年
- `begin` (number): 开始时间
- `end` (number): 结束时间

**返回值**:
- `Array<BundleStatsInfo>`: 统计信息数组

**统计对象结构**:
```typescript
{
    bundleName: string,
    abilityPrevAccessTime: number,
    abilityInFgTotalTime: number,
    id: number
}
```

**C++ 实现**:
- `QueryBundleStatsInfoByInterval()` - frameworks/src/bundle_state_query_napi.cpp:553

**参数校验**:
- frameworks/src/bundle_state_query_napi.cpp:475-552: ParseQueryBundleStatsInfoByInterval()

**错误码**:
- 401009: ERR_INTERVAL_TYPE - intervalType 类型错误
- 401010: ERR_INTERVAL_OUT_OF_RANGE - intervalType 范围错误
- 同时间参数错误码

---

##### 6. queryBundleStatsInfos

**功能**: 查询应用使用时长统计

**签名**:
```typescript
queryBundleStatsInfos(begin: number, end: number): Promise<BundleStatsMap>
```

**参数**:
- `begin` (number): 开始时间
- `end` (number): 结束时间

**返回值**:
- `BundleStatsMap`: 统计信息映射

**C++ 实现**:
- `QueryBundleStatsInfos()` - frameworks/src/bundle_state_query_napi.cpp:651

**错误码**:
- 同时间参数错误码

---

##### 7. queryModuleUsageRecords

**功能**: 查询 FA/Ability 模块使用记录

**签名**:
```typescript
queryModuleUsageRecords(maxNum?: number): Promise<Array<HapModuleInfo>>
```

**参数**:
- `maxNum` (number?, 可选): 最大返回数量，默认 1000，最大 1000

**返回值**:
- `Array<HapModuleInfo>`: 模块使用记录数组

**模块对象结构**:
```typescript
{
    bundleName: string,
    appLabelId: number,
    moduleName: string,
    labelId: number,
    descriptionId: number,
    abilityName: string,
    abilityLableId: number,
    abilityDescriptionId: number,
    abilityIconId: number,
    launchedCount: number,
    lastModuleUsedTime: number,
    formRecords: Array<{
        formName: string,
        formDimension: number,
        formId: number,
        formLastUsedTime: number,
        count: number
    }>
}
```

**C++ 实现**:
- `QueryModuleUsageRecords()` - frameworks/src/bundle_state_query_napi.cpp:131

**参数校验**:
- frameworks/src/bundle_state_query_napi.cpp:60-130: ParseQueryModuleUsageRecords()

**错误码**:
- 401013: ERR_MAX_RECORDS_NUM_TYPE - maxNum 类型错误
- 401014: ERR_MAX_RECORDS_NUM_BIGER_THEN_ONE_THOUSAND - maxNum > 1000 或 < 1

---

##### 8. setAppGroup

**功能**: 设置应用分组

**签名**:
```typescript
setAppGroup(bundleName: string, newGroup: GroupType): Promise<void>
```

**参数**:
- `bundleName` (string): 应用包名
- `newGroup` (GroupType): 新分组类型
  - `GroupType.ALIVE_GROUP` (10)
  - `GroupType.DAILY_GROUP` (20)
  - `GroupType.FIXED_GROUP` (30)
  - `GroupType.RARE_GROUP` (40)
  - `GroupType.LIMITED_GROUP` (50)
  - `GroupType.NEVER_GROUP` (60)

**返回值**:
- `Promise<void>`

**权限要求**:
- `ohos.permission.BUNDLE_ACTIVE_INFO`
- 系统应用

**C++ 实现**:
- `SetAppGroup()` - frameworks/src/bundle_active_app_group_napi.cpp:312

**参数校验**:
- frameworks/src/bundle_active_app_group_napi.cpp:257-311: ParseSetAppGroupParameters()

**错误码**:
- 201: ERR_PERMISSION_DENIED - 权限被拒绝
- 202: ERR_NOT_SYSTEM_APP - 非系统应用
- 401011: ERR_NEW_GROUP_TYPE - newGroup 类型错误
- 401012: ERR_NEW_GROUP_OUT_OF_RANGE - newGroup 范围错误

---

##### 9. registerAppGroupCallBack

**功能**: 注册应用分组变更回调

**签名**:
```typescript
registerAppGroupCallBack(callback: Callback<AppGroupCallbackInfo>): Promise<void>
```

**参数**:
- `callback` (Callback): 回调函数

**回调参数结构**:
```typescript
{
    userId: number,
    bundleName: string,
    newGroup: number
}
```

**返回值**:
- `Promise<void>`

**C++ 实现**:
- `RegisterAppGroupCallBack()` - frameworks/src/bundle_active_app_group_napi.cpp:421

**参数校验**:
- frameworks/src/bundle_active_app_group_napi.cpp:357-420: ParseRegisterAppGroupCallBackParameters()

**错误码**:
- 401015: ERR_APP_GROUP_OBSERVER_CALLBACK_TYPE - callback 类型错误
- 401016: ERR_REPEAT_REGISTER_APP_GROUP_OBSERVER - 重复注册

---

##### 10. unregisterAppGroupCallBack

**功能**: 注销应用分组变更回调

**签名**:
```typescript
unregisterAppGroupCallBack(): Promise<void>
```

**返回值**:
- `Promise<void>`

**C++ 实现**:
- `UnRegisterAppGroupCallBack()` - frameworks/src/bundle_active_app_group_napi.cpp:503

**错误码**:
- 401017: ERR_APP_GROUP_OBSERVER_IS_NULLPTR - observer 为 null

---

##### 11. queryDeviceEventStats

**功能**: 查询系统事件统计（休眠、唤醒、解锁、锁屏）

**签名**:
```typescript
queryDeviceEventStats(begin: number, end: number): Promise<Array<DeviceEventStats>>
```

**参数**:
- `begin` (number): 开始时间
- `end` (number): 结束时间

**返回值**:
- `Array<DeviceEventStats>`: 事件统计数组

**事件统计对象结构**:
```typescript
{
    name: string,
    eventId: number,
    count: number
}
```

**C++ 实现**:
- `QueryDeviceEventStats()` - frameworks/src/bundle_state_query_napi.cpp:945

**错误码**:
- 同时间参数错误码

---

##### 12. queryNotificationEventStats

**功能**: 查询应用通知次数

**签名**:
```typescript
queryNotificationEventStats(begin: number, end: number): Promise<Array<DeviceEventStats>>
```

**参数**:
- `begin` (number): 开始时间
- `end` (number): 结束时间

**返回值**:
- `Array<DeviceEventStats>`: 通知统计数组

**C++ 实现**:
- `QueryNotificationEventStats()` - frameworks/src/bundle_state_query_napi.cpp:999

**错误码**:
- 同时间参数错误码

---

##### 13. queryAppStatsInfos

**功能**: 查询应用统计信息

**签名**:
```typescript
queryAppStatsInfos(begin: number, end: number): Promise<Array<AppStatsInfo>>
```

**参数**:
- `begin` (number): 开始时间
- `end` (number): 结束时间

**返回值**:
- `Array<AppStatsInfo>`: 应用统计数组

**C++ 实现**:
- `QueryAppStatsInfos()` - frameworks/src/bundle_state_query_napi.cpp:751

**错误码**:
- 同时间参数错误码

---

##### 14. queryLastUseTime

**功能**: 查询应用最后使用时间

**签名**:
```typescript
queryLastUseTime(bundleNames: Array<string>): Promise<Map<string, number>>
```

**参数**:
- `bundleNames` (Array<string>): 应用包名数组

**返回值**:
- `Map<string, number>`: 应用包名到最后使用时间的映射

**C++ 实现**:
- `QueryLastUseTime()` - frameworks/src/bundle_state_query_napi.cpp:843

**错误码**:
- 同参数错误码

---

### 旧版模块 (@ohos.bundleState)

**模块名**: `bundleState`

**注册点**: `frameworks/src/bundle_state_init.cpp:106`

**API 列表**（兼容旧接口）:

| API 名称 | 新版对应 | 说明 |
|---------|----------|------|
| queryBundleActiveStates | queryBundleEvents | 查询应用事件集合 |
| queryBundleStateInfos | queryBundleStatsInfos | 查询应用使用时长 |
| queryCurrentBundleActiveStates | queryCurrentBundleEvents | 查询当前应用事件 |
| queryBundleStateInfoByInterval | queryBundleStatsInfoByInterval | 按间隔查询统计 |
| queryAppUsagePriorityGroup | queryAppGroup | 查询应用分组 |
| isIdleState | isIdleState | 判断应用空闲状态 |

**证据**:
- frameworks/src/bundle_state_init.cpp:46-50: 旧版 API 导出

---

## 错误码完整列表

### 参数错误码 (401001-401017)

| 错误码 | 值 | 含义 |
|--------|-----|------|
| ERR_PARAMETERS_NUMBER | 401001 | 参数数量错误 |
| ERR_CALL_BACK_TYPE | 401002 | callback 类型错误 |
| ERR_PARAMETERS_EMPTY | 401003 | 参数为空 |
| ERR_BUNDLE_NAME_TYPE | 401004 | bundleName 类型错误 |
| ERR_BEGIN_TIME_TYPE | 401005 | beginTime 类型错误 |
| ERR_BEGIN_TIME_LESS_THEN_ZERO | 401006 | beginTime < 0 |
| ERR_END_TIME_TYPE | 401007 | endTime 类型错误 |
| ERR_END_TIME_LESS_THEN_BEGIN_TIME | 401008 | endTime <= beginTime |
| ERR_INTERVAL_TYPE | 401009 | intervalType 类型错误 |
| ERR_INTERVAL_OUT_OF_RANGE | 401010 | intervalType 范围错误 |
| ERR_NEW_GROUP_TYPE | 401011 | newGroup 类型错误 |
| ERR_NEW_GROUP_OUT_OF_RANGE | 401012 | newGroup 范围错误 |
| ERR_MAX_RECORDS_NUM_TYPE | 401013 | maxNum 类型错误 |
| ERR_MAX_RECORDS_NUM_BIGER_THEN_ONE_THOUSAND | 401014 | maxNum > 1000 或 < 1 |
| ERR_APP_GROUP_OBSERVER_CALLBACK_TYPE | 401015 | observer callback 类型错误 |
| ERR_REPEAT_REGISTER_APP_GROUP_OBSERVER | 401016 | 重复注册 observer |
| ERR_APP_GROUP_OBSERVER_IS_NULLPTR | 401017 | observer 为 null |

**证据**:
- interfaces/kits/bundlestats/napi/include/bundle_state_inner_errors.h: 完整错误码定义

---

### 系统错误码

| 错误码 | 值 | 含义 |
|--------|-----|------|
| ERR_PERMISSION_DENIED | 201 | 权限被拒绝 |
| ERR_NOT_SYSTEM_APP | 202 | 非系统应用 |
| ERR_PARAM_ERROR | 401 | 参数错误 |
| ERR_MEMORY_OPERATION_FAILED | 10000001 | 内存操作失败 |
| ERR_IPC_COMMUNICATION_FAILED | 10000004 | IPC 通信失败 |
| ERR_APPLICATION_IS_NOT_INSTALLED | 10000005 | 应用未安装 |
| ERR_APPLICATION_GROUP_OPERATION_REPEATED | 10100001 | 应用分组操作重复 |
| ERR_GET_SYSTEM_ABILITY_MANAGER_FAILED | 1000000301 | 获取系统能力管理器失败 |
| ERR_GET_SYSTEM_ABILITY_FAILED | 1000000302 | 获取系统能力失败 |
| ERR_SYSTEM_SERVICES_NOT_READY | 1000000304 | 系统服务未就绪 |

---

## 异步处理模式

### Promise vs Callback

**Promise 模式**:
```typescript
// 调用时不传 callback
const result = await queryBundleEvents(begin, end);
```

**Callback 模式**:
```typescript
// 调用时传入 callback
queryBundleEvents(begin, end, (err, data) => {
    if (err) {
        console.error(err);
        return;
    }
    console.log(data);
});
```

**C++ 实现**:
- frameworks/src/bundle_state_common.cpp:82-90: GetCallbackPromiseResult()

---

### 同步 API

**同步 API**:
- `isIdleStateSync()` - frameworks/src/bundle_state_query_napi.cpp:281
- `queryAppGroupSync()` - frameworks/src/bundle_active_app_group_napi.cpp:190

**注意**: 同步 API 可能阻塞主线程，谨慎使用。

---

## 权限要求

| API | 权限 | 系统应用 |
|-----|-------|----------|
| setAppGroup | ohos.permission.BUNDLE_ACTIVE_INFO | 必须 |
| registerAppGroupCallBack | ohos.permission.BUNDLE_ACTIVE_INFO | 必须 |
| 其他查询 API | 无 | 无 |

**证据**:
- services/common/src/bundle_active_service.cpp:54: NEEDED_PERMISSION 定义
- services/common/src/bundle_active_service.cpp:717-766: 权限检查实现

---

## 相关跳转链接

- [00_项目概览.md](00_项目概览.md) - 了解项目定位
- [01_目录结构与模块职责.md](01_目录结构与模块职责.md) - 了解模块组织
- [02_架构说明.md](02_架构说明.md) - 了解数据流和线程模型
- [06_安全风险评审.md](06_安全风险评审.md) - 了解安全机制

---

## 版本信息

- **生成时间**: 2026-02-06
- **文档版本**: v1.0
