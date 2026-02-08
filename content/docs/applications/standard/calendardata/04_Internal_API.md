# 内部 API

## 目的

本文档说明 CalendarData 组件的内部模块接口、依赖方向、稳定性标记和可替换点，帮助组件开发者理解内部设计。

## 适用范围

- 目标读者：组件开发者、架构师
- 知识级别：中级到高级
- 前置知识：了解 OpenHarmony 组件化架构

## 模块依赖图

### 依赖方向

```
datastructure
    ↑
    │
common ─┐
    │    │
    │    │
datamanager (依赖 common, datastructure)
    │
    │
dataprovider (依赖 datamanager, common, datastructure)
    │
    │
entry (依赖 dataprovider)
    │
    │
calendarmanager (依赖 datastructure, common)
```

### 依赖说明

| 模块 | 依赖 | 说明 |
|------|------|------|
| calendarmanager | datastructure | 使用表结构定义 |
| calendarmanager | common | 使用日志等工具类 |
| datamanager | common | 使用日志、时间工具等 |
| datamanager | datastructure | 使用表结构定义 |
| dataprovider | datamanager | 使用数据库处理器 |
| dataprovider | common | 使用日志、错误工具等 |
| dataprovider | datastructure | 使用表结构定义 |
| entry | dataprovider | 使用 DataShareAbility |

**无循环依赖**：所有模块依赖形成 DAG（有向无环图）

## DataProvider 内部 API

### DatabaseProcessor 接口

**位置**: datamanager/src/main/ets/processor/DatabaseProcessor.ets

**稳定性**: ✅ 稳定

**接口定义**:

```typescript
export interface DatabaseProcessor {
  insertByHighAuthority(rdbStore, uri, values, callback): void;
  batchInsertByHighAuthority(rdbStore, uri, values, callback): void;
  insertByLowAuthority(rdbStore, uri, values, callback): void;
  batchInsertByLowAuthority(rdbStore, uri, values, callback): void;
  deleteByHighAuthority(rdbStore, uri, predicates, callback): void;
  deleteByLowAuthority(rdbStore, uri, predicates, callback): void;
  updateByHighAuthority(rdbStore, uri, values, predicates, callback): void;
  updateByLowAuthority(rdbStore, uri, values, predicates, callback): void;
  queryByHighAuthority(rdbStore, uri, columns, predicates, callback): void;
  queryByLowAuthority(rdbStore, uri, columns, predicates, callback): void;
}
```

**实现类**:
- CalendarsProcessor
- EventsProcessor
- InstancesProcessor
- RemindersProcessor
- AlertsProcessor

### DatabaseProcessorFactory

**位置**: datamanager/src/main/ets/processor/DatabaseProcessorFactory.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
class DatabaseProcessorFactory {
  getDatabaseProcessor(tableName: string): DatabaseProcessor;
}
```

**支持的表名**:
- CalendarsColumns.TABLE_NAME
- EventColumns.TABLE_NAME
- InstancesColumns.TABLE_NAME
- RemindersColumns.TABLE_NAME
- CalendarAlertsColumns.TABLE_NAME

### DataShareAbilityDelegate

**位置**: dataprovider/src/main/ets/DataShareAbilityDelegate.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
class DataShareAbilityDelegate {
  init(): Promise<boolean>;
  insertByHighAuthority(uri, value, callback): void;
  batchInsertByHighAuthority(uri, value, callback): void;
  insertByLowAuthority(uri, value, callback): void;
  batchInsertByLowAuthority(uri, value, callback): void;
  deleteByHighAuthority(uri, predicates, callback): void;
  deleteByLowAuthority(uri, predicates, callback): void;
  updateByHighAuthority(uri, value, predicates, callback): void;
  updateByLowAuthority(uri, value, predicates, callback): void;
  queryByHighAuthority(uri, columns, predicates, callback): void;
  queryByLowAuthority(uri, columns, predicates, callback): void;
}
```

### DataShareAbilityAuthenticateProxy

**位置**: dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
class DataShareAbilityAuthenticateProxy {
  init(): Promise<boolean>;
  insertByProxy(uri, value, permission, callback): void;
  deleteByProxy(uri, predicates, permission, callback): void;
  updateByProxy(uri, value, predicates, permission, callback): void;
  queryByProxy(uri, columns, predicates, permission, callback): void;
}
```

**权限级别**:
- PERMISSIONS_FLAG_HIGH: 2
- PERMISSIONS_FLAG_LOW: 1
- PERMISSIONS_FLAG_UNAUTHORIZED: -1

## Native 层内部 API

### DataShareHelperManager

**位置**: calendarmanager/native/src/data_share_helper_manager.cpp

**稳定性**: ✅ 稳定

**接口**:

```cpp
class DataShareHelperManager {
public:
    DataShareHelperManager() = default;
    ~DataShareHelperManager() = default;

    // 创建 DataShareHelper 实例
    std::shared_ptr<DataShareHelper> CreateInnerDataShareHelper(const std::string &permissionUri);

    // 插入操作
    int Insert(const std::string &uri, const Native::ValuesBucket &values, const std::string &bundleName);
    int BatchInsert(const std::string &uri, const std::vector<Native::ValuesBucket> &values, const std::string &bundleName);

    // 查询操作
    int Query(const std::string &uri, const std::vector<std::string> &columns, const Native::DataSharePredicates &predicates,
             std::vector<std::string> &outRecords, const std::string &bundleName);

    // 更新操作
    int Update(const std::string &uri, const Native::ValuesBucket &values, const Native::DataSharePredicates &predicates,
             const std::string &bundleName);

    // 删除操作
    int Delete(const std::string &uri, const Native::DataSharePredicates &predicates, const std::string &bundleName);
};
```

**权限检查**:
- CreateInnerDataShareHelper 时验证权限
- 使用 AccessTokenKit::VerifyAccessToken()
- 返回高权限或低权限 DataShareHelper

### Native::Calendar

**位置**: calendarmanager/native/src/native_calendar.cpp

**稳定性**: ✅ 稳定

**接口**（部分）:
- AddEvent()
- AddEvents()
- DeleteEvent()
- DeleteEvents()
- UpdateEvent()
- UpdateEvents()
- GetEvents()
- GetConfig()
- SetConfig()
- QueryEventInstances()

### Native::CalendarManager

**位置**: calendarmanager/native/src/native_calendar_manager.cpp

**稳定性**: ✅ 稳定

**接口**:
- CreateCalendar()
- DeleteCalendar()
- GetCalendar()
- GetAllCalendars()

## 公共工具 API

### CalendarDataHelper

**位置**: datamanager/src/main/ets/utils/CalendarDataHelper.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
class CalendarDataHelper {
  static getInstance(): CalendarDataHelper;
  getRdbStore(): Promise<RdbStore>;
}
```

### CalendarUriHelper

**位置**: datamanager/src/main/ets/utils/CalendarUriHelper.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
function getTableByUri(uri: string): string;
function getBundleNameAndTokenIDByUri(uri: string): BundleNameAndTokenId;
```

**URI 格式**: `datashare:///com.ohos.calendarData/<table>/<bundleName>/<tokenId>`

### Log

**位置**: common/src/main/ets/utils/Log.ets

**稳定性**: ✅ 稳定

**接口**:

```typescript
class Log {
  static log(tag: string, message: string): void;
  static info(tag: string, message: string): void;
  static warn(tag: string, message: string): void;
  static error(tag: string, message: string): void;
}
```

## 可替换点

### 数据库抽象层

**位置**: datamanager/src/main/ets/processor/DatabaseProcessor.ets

**说明**: DatabaseProcessor 接口定义了统一的数据库操作抽象，可以实现不同的数据库后端。

**稳定性**: ✅ 稳定接口，✅ 稳定实现

**可替换方案**:
- 当前实现：RDB
- 可替换：其他关系数据库或 NoSQL

### DataShare 层

**位置**: dataprovider/src/main/ets/DataShareAbilityDelegate.ets

**说明**: DataShareAbilityDelegate 封装了 DataShare 调用，可以替换为其他 IPC 机制。

**稳定性**: ⚠️ 依赖 OpenHarmony DataShare 框架

### 权限验证层

**位置**: dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets

**说明**: 权限验证逻辑集中在此，可以调整验证策略。

**稳定性**: ✅ 稳定实现

## 不稳定接口

### 内部实现细节

以下接口是内部实现细节，不应依赖：

1. **RDB 表结构**（datastructure/）:
   - 具体列名可能变更
   - 通过 API 而非直接访问数据库

2. **数据库处理器实现**（datamanager/processor/）:
   - 具体处理器可能重构
   - 通过 DatabaseProcessorFactory 获取

3. **权限常量**（dataprovider/）:
   - 可能新增权限级别
   - 使用常量而非硬编码

4. **Native 层实现**（calendarmanager/native/）:
   - C++ 实现细节
   - 通过 N-API 接口调用

## 线程安全

### 主线程操作

- N-API 绑定：主线程
- DataShareExtAbility：主线程

### 异步操作

- 数据库操作：异步 Promise/Callback
- 权限验证：async/await

### 共享资源

- RdbStore：通过 CalendarDataHelper 单例访问
- DataShareHelper：按需创建

## 错误传播

### ArkTS 层

**位置**: common/src/main/ets/utils/ErrorUtils.ets

```typescript
export function hasNoError(err: BusinessError): boolean;
```

**传播方式**:
- 通过 callback 第一个参数传播错误
- 通过 Promise reject 传播错误

### C++ 层

**位置**: calendarmanager/native/src/native_util.cpp

**错误码**:
- ERROR = -1
- SUCCESS = 0

## 性能考虑

### 批量操作

优先使用批量 API：
- `batchInsert` 而非多次 `insert`
- `addEvents` 而非多次 `addEvent`

### 查询优化

- 使用 `EventFilter` 过滤
- 限制查询范围（时间范围）
- 仅查询需要的列

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [架构说明](02_Architecture.md) - 详细架构和数据流
- [对外 API](03_External_API.md) - N-API 完整清单
- [GN 构建](05_GN_Build.md) - 构建系统详解

---

返回 [目录](SUMMARY.md) | [首页](README.md)
