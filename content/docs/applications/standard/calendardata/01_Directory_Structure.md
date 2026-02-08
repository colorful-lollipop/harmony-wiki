# 目录结构与模块职责

## 目的

本文档详细说明 CalendarData 组件的目录结构、各模块的职责和代码入口，帮助开发者快速定位代码。

## 适用范围

- 目标读者：所有开发者
- 知识级别：初级到中级
- 前置知识：了解 OpenHarmony 基础目录结构

## 顶层目录结构

```
calendardata/
├── AppScope/              # 应用级资源和配置
├── calendarmanager/       # 日历管理器核心（C++ + N-API/CJ）
├── common/                # 公共工具类（ArkTS）
├── datamanager/          # 数据管理器（ArkTS）
├── dataprovider/         # 数据提供器（ArkTS）
├── datastructure/        # 数据结构定义（ArkTS）
├── entry/                # 应用入口（ArkTS）
├── figures/              # 架构图片
├── rrule/                # 重复规则（ArkTS）
├── signature/            # 签名证书
├── bundle.json           # 组件元数据
├── build-profile.json5   # 构建配置
└── wiki/                # 工程文档
```

## 模块详解

### 1. calendarmanager/ - 日历管理器核心

**职责**: 提供 N-API 和 CJ (FFI) 绑定，实现日历数据的 C++ 核心逻辑

**语言**: C++ (主要)，JavaScript (包装层)

**关键目录**:

| 目录 | 说明 | 文件数（不含测试） |
|------|------|---------------------|
| `native/src/` | C++ 实现源文件 | ~10 |
| `native/include/` | C++ 头文件 | ~8 |
| `napi/src/` | N-API 绑定源文件 | ~9 |
| `napi/include/` | N-API 头文件 | ~8 |
| `cj/src/` | CJ FFI 绑定源文件 | ~10 |
| `cj/include/` | CJ 头文件 | ~10 |
| `common/` | 共享定义 | ~2 |
| `js/` | JS 包装层 | 1 |

**关键文件**:

| 文件 | 职责 | 证据 |
|------|------|------|
| `napi/src/module_register.cpp` | N-API 模块注册 | :45-52 |
| `napi/src/module_init.cpp` | 模块初始化 | :24-32 |
| `napi/src/calendar_napi.cpp` | Calendar N-API 绑定 | CalendarNapi 类 |
| `napi/src/calendar_manager_napi.cpp` | CalendarManager N-API 绑定 | CalendarManagerNapi 类 |
| `napi/src/event_filter_napi.cpp` | EventFilter N-API 绑定 | EventFilterNapi 类 |
| `native/src/native_calendar.cpp` | Calendar C++ 实现 | Native::Calendar 类 |
| `native/src/native_calendar_manager.cpp` | CalendarManager C++ 实现 | Native::CalendarManager 类 |
| `native/src/data_share_helper_manager.cpp` | DataShare 管理器 | DataShareHelperManager 类 |
| `js/editor.js` | JS API 包装 | 导出 CalendarManager 等 |
| `cj/src/calendar_manager_ffi.cpp` | CJ FFI 绑定 | calendarManager_ffi 导出 |

**对外导出**（来自 `js/editor.js`）:
```javascript
{
  getCalendarManager,
  CalendarManager,
  Calendar,
  CalendarAccount,
  CalendarConfig,
  Event,
  CalendarType,
  Location,
  EventFilter,
  EventType,
  RecurrenceRule,
  RecurrenceFrequency,
  Attendee,
  EventService,
  ServiceType,
  AttendeeRole,
  AttendeeStatus,
  AttendeeType
}
```

### 2. datamanager/ - 数据管理器

**职责**: 数据库操作、数据转换、处理器工厂

**语言**: ArkTS

**关键目录**:

| 目录 | 说明 |
|------|------|
| `src/main/ets/processor/` | 数据库处理器 |
| `src/main/ets/utils/` | 工具类 |
| `src/main/ets/commonevents/` | 通用事件通知 |
| `src/main/ets/constants/` | 常量定义 |

**关键文件**:

| 文件 | 职责 | 证据 |
|------|------|------|
| `processor/DatabaseProcessor.ets` | 数据库处理器接口 | 定义高/低权限操作接口 |
| `processor/DatabaseProcessorFactory.ets` | 处理器工厂 | :40-73 |
| `processor/DatabaseProcessorHelper.ets` | 数据库助手 | 辅助方法 |
| `processor/calendars/CalendarsProcessor.ets` | Calendars 表处理 | Calendars 表增删改查 |
| `processor/events/EventsProcessor.ets` | Events 表处理 | Events 表增删改查 |
| `processor/instances/InstancesProcessor.ets` | Instances 表处理 | 实例扩展和查询 |
| `processor/reminders/RemindersProcessor.ets` | Reminders 表处理 | 提醒数据处理 |
| `processor/alerts/AlertsProcessor.ets` | CalendarAlerts 表处理 | 告警数据处理 |
| `utils/CalendarDataHelper.ets` | 数据库连接管理 | RdbStore 管理 |
| `utils/CalendarUriHelper.ets` | URI 解析 | URI → 表名映射 |
| `commonevents/notify/ScheduleAlarmNotifier.ets` | 告警调度 | 通知闹钟服务 |
| `commonevents/notify/ProviderChangeNotifier.ets` | 数据变更通知 | 通知订阅者 |

**数据流**:

```
DataShareExtAbility
    ↓
DataShareAbilityAuthenticateProxy (权限验证)
    ↓
DataShareAbilityDelegate (代理)
    ↓
DatabaseProcessorFactory (工厂)
    ↓
[Calendars|Events|Instances|Reminders|Alerts]Processor (具体处理器)
    ↓
RDB 操作
```

### 3. dataprovider/ - 数据提供器

**职责**: DataShare Extension Ability 实现，权限验证代理

**语言**: ArkTS

**关键文件**:

| 文件 | 职责 | 证据 |
|------|------|------|
| `DataShareAbilityDelegate.ets` | 数据库操作代理 | 实现高/低权限 CRUD |
| `DataShareAbilityAuthenticateProxy.ets` | 权限验证代理 | :112-142 |
| `AuthenticationUriHelper.ets` | 鉴权 URI 助手 | 验证访问权限 |

**权限级别**（`DataShareAbilityAuthenticateProxy.ets:42-44`）:
```typescript
const PERMISSIONS_FLAG_HIGH: number = 2;
const PERMISSIONS_FLAG_LOW: number = 1;
const PERMISSIONS_FLAG_UNAUTHORIZED: number = -1;
```

**权限常量**（`DataShareAbilityAuthenticateProxy.ets:46-49`）:
```typescript
const PERMISSIONS_WRITE_WHOLE_CALENDAR = 'ohos.permission.WRITE_WHOLE_CALENDAR';
const PERMISSIONS_WRITE_CALENDAR = 'ohos.permission.WRITE_CALENDAR';
const PERMISSIONS_READ_WHOLE_CALENDAR = 'ohos.permission.READ_WHOLE_CALENDAR';
const PERMISSIONS_READ_CALENDAR = 'ohos.permission.READ_CALENDAR';
```

### 4. datastructure/ - 数据结构定义

**职责**: 定义所有表结构和列定义

**语言**: ArkTS

**关键目录**:

| 目录 | 说明 |
|------|------|
| `src/main/ets/calendars/` | Calendars 表 |
| `src/main/ets/events/` | Events 表 |
| `src/main/ets/instances/` | Instances 表 |
| `src/main/ets/reminders/` | Reminders 表 |
| `src/main/ets/calendaralerts/` | CalendarAlerts 表 |
| `src/main/ets/attendees/` | 参会者 |
| `src/main/ets/extendedproperties/` | 扩展属性 |
| `src/main/ets/syncstate/` | 同步状态 |

**关键表结构**:

| 表 | 主要字段 | 证据 |
|----|---------|------|
| Calendars | id, accountName, calendarColor, ownerAccount, isPrimary | `calendars/Calendars.ets:63-102` |
| Events | id, title, dtStart, dtEnd, rRule, allDay | `events/Events.ets:97-193` |
| Instances | id, eventId, begin, end | `instances/InstancesColumns.ets` |
| Reminders | id, eventId, minutes | `reminders/RemindersColumns.ets` |
| CalendarAlerts | id, eventId, alertTime, state | `calendaralerts/CalendarAlertsColumns.ets` |

### 5. entry/ - 应用入口

**职责**: Extension Ability 入口、UI Ability

**语言**: ArkTS

**关键文件**:

| 文件 | 职责 | 证据 |
|------|------|------|
| `src/main/ets/DataAbility/DataShareExtAbility.ets` | DataShare Extension | 继承 Extension |
| `src/main/ets/MainAbility/MainAbility.ets` | UI Ability | 继承 UIAbility |
| `src/main/ets/Application/Application.ets` | 应用类 | 应用生命周期 |
| `src/main/ets/pages/index.ets` | 主页面 | UI 页面 |

**DataShareExtAbility 关键方法**（`DataShareExtAbility.ets`）:
- `onCreate(want, callback)` - 创建回调
- `insert(uri, value, callback)` - 插入数据
- `update(uri, predicates, value, callback)` - 更新数据
- `query(uri, predicates, columns, callback)` - 查询数据
- `delete(uri, predicates, callback)` - 删除数据
- `batchInsert(uri, value, callback)` - 批量插入

### 6. common/ - 公共工具

**职责**: 提供跨模块的工具类

**语言**: ArkTS

**关键文件**:

| 文件 | 职责 |
|------|------|
| `utils/Log.ets` | 日志工具 |
| `utils/TimeUtils.ets` | 时间处理 |
| `utils/UrlUtils.ets` | URL/URI 处理 |
| `utils/ErrorUtils.ets` | 错误处理 |
| `utils/GlobalThis.ets` | 全局状态 |
| `utils/ValuesUtils.ets` | 值转换 |
| `utils/SystemTimerUtils.ets` | 系统定时器 |
| `utils/TextUtils.ets` | 文本处理 |
| `observer/Observer.ets` | 观察者基类 |
| `observer/Observable.ets` | 可观察对象 |
| `subscriber/Subscriber.ets` | 订阅者 |
| `broadcast/BroadcastHelper.ets` | 广播助手 |
| `time/JulianDay.ets` | 儒略日计算 |

### 7. rrule/ - 重复规则

**职责**: 日程重复规则处理

**语言**: ArkTS

**关键文件**:

| 文件 | 职责 |
|------|------|
| `src/main/ets/RecurrenceSet.ets` | 重复规则集合 |

## 数据访问层

```
┌─────────────────────────────────────────┐
│       JavaScript/ArkTS 应用层         │
└─────────────────┬───────────────────┘
                  │
                  ↓ N-API/CJ Bindings
┌─────────────────────────────────────────┐
│       calendarmanager (C++)          │
└─────────────────┬───────────────────┘
                  │ DataShare IPC
                  ↓
┌─────────────────────────────────────────┐
│     entry/DataShareExtAbility         │
└─────────────────┬───────────────────┘
                  │
                  ↓ dataprovider (ArkTS)
┌─────────────────────────────────────────┐
│  DataShareAbilityAuthenticateProxy      │ (权限验证)
└─────────────────┬───────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│    DataShareAbilityDelegate            │ (代理)
└─────────────────┬───────────────────┘
                  │
                  ↓ datamanager (ArkTS)
┌─────────────────────────────────────────┐
│    DatabaseProcessorFactory            │ (工厂)
└─────────────────┬───────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│   [Calendars|Events|...]Processor    │ (处理器)
└─────────────────┬───────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│           RDB 数据库                 │
└─────────────────────────────────────────┘
```

## 代码查找指南

### 查找 N-API 导出

1. 查看 `calendarmanager/js/editor.js` - JS API 导出
2. 查看 `calendarmanager/napi/src/module_init.cpp` - N-API 初始化
3. 查看对应的 `*_napi.cpp` - N-API 实现

### 查找数据库操作

1. 查看 `datastructure/src/main/ets/` - 表结构定义
2. 查看 `datamanager/src/main/ets/processor/` - 对应处理器
3. 查看 `dataprovider/src/main/ets/` - 权限和代理层

### 查找权限检查

1. 查看 `dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets` - ArkTS 层
2. 查看 `calendarmanager/native/src/data_share_helper_manager.cpp` - C++ 层

## 相关文档

- [项目概览](00_Overview.md) - 项目定位和核心概念
- [架构说明](02_Architecture.md) - 详细架构图和数据流
- [对外 API](03_External_API.md) - N-API 完整清单
- [内部 API](04_Internal_API.md) - 模块接口和依赖

---

返回 [目录](SUMMARY.md) | [首页](README.md)
