# CalendarData 架构说明

## 目的

本文档详细说明 CalendarData 组件的架构设计，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- 目标读者：架构师、高级开发者
- 知识级别：中级到高级
- 前置知识：了解 OpenHarmony 组件化架构

## 整体架构

### 系统边界

```
┌─────────────────────────────────────────────────────────────┐
│                   OpenHarmony 系统                  │
├─────────────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐      ┌──────────────┐         │
│  │  Calendar   │      │  第三方应用   │         │
│  │  (UI)      │◄────►│ (调用者)     │         │
│  └──────────────┘      └──────────────┘         │
│         │                  │                   │
│         │ N-API          │                   │
│         ▼                  │                   │
│  ┌──────────────────────────────────┐          │
│  │   CalendarData 组件          │          │
│  │  ┌──────────────────────┐    │          │
│  │  │ calendarmanager     │    │          │
│  │  │ (N-API/CJ/C++)    │    │          │
│  │  └────────┬─────────────┘    │          │
│  │           │ DataShare         │          │
│  │           ▼                  │          │
│  │  ┌──────────────────────┐    │          │
│  │  │ entry               │    │          │
│  │  │ (DataShareAbility)  │    │          │
│  │  └────────┬─────────────┘    │          │
│  │           │ dataprovider       │          │
│  │           ▼                  │          │
│  │  ┌──────────────────────┐    │          │
│  │  │ DataShareDelegate   │    │          │
│  │  └────────┬─────────────┘    │          │
│  │           │ datamanager        │          │
│  │           ▼                  │          │
│  │  ┌──────────────────────┐    │          │
│  │  │ ProcessorFactory   │    │          │
│  │  └────────┬─────────────┘    │          │
│  │           │                  │          │
│  │           ▼                  │          │
│  │  ┌──────────────────────┐    │          │
│  │  │ RDB 数据库        │    │          │
│  │  └──────────────────────┘    │          │
│  └──────────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
```

### 核心组件关系

```
calendarmanager          dataprovider            datamanager
     │                        │                       │
     │ N-API Bindings       │ DataShare IPC         │
     │                        │                       │
     ▼                        ▼                       ▼
┌─────────┐            ┌─────────────────┐      ┌──────────────┐
│ Calendar│            │ DataShareExt   │      │ Processor    │
│ NAPI    │──────────►│ Ability         │─────►│ Factory      │
└─────────┘            └────────┬────────┘      └──────┬───────┘
                               │                       │
                               ▼                       ▼
                      ┌─────────────────┐      ┌──────────────┐
                      │ Authenticate    │      │ Calendar     │
                      │ Proxy          │─────►│ Processor    │
                      └─────────────────┘      └──────────────┘
                      ┌─────────────────┐      ┌──────────────┐
                      │ Delegate        │      │ Events       │
                      │ Proxy          │─────►│ Processor    │
                      └─────────────────┘      └──────────────┘
                                               ...
```

## 数据流

### 读操作流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant CM as CalendarManager
    participant DSH as DataShareHelper
    participant Ability as DataShareExtAbility
    participant Auth as AuthenticateProxy
    participant Delegate as DataShareDelegate
    participant Factory as ProcessorFactory
    participant Processor as CalendarProcessor
    participant RDB as RDB

    App->>NAPI: getCalendar()
    NAPI->>CM: 获取日历
    CM->>DSH: CreateInnerDataShareHelper(READ_URI)
    DSH->>Ability: query()
    Ability->>Auth: queryByProxy(uri, predicates, READ_CALENDAR)
    Auth->>Delegate: verifyByUri(uri, predicates, LOW_PERMISSION)
    Delegate->>Factory: getDatabaseProcessor(table)
    Factory->>Processor: new CalendarsProcessor()
    Processor->>Delegate: queryByLowAuthority()
    Delegate->>RDB: query(predicates)
    RDB-->>Delegate: result
    Delegate-->>Auth: result
    Auth-->>Ability: result
    Ability-->>DSH: result
    DSH-->>CM: result
    CM-->>NAPI: result
    NAPI-->>App: Calendar[]
```

### 写操作流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant CM as CalendarManager
    participant DSH as DataShareHelper
    participant Ability as DataShareExtAbility
    participant Auth as AuthenticateProxy
    participant Delegate as DataShareDelegate
    participant Factory as ProcessorFactory
    participant Processor as CalendarProcessor
    participant RDB as RDB

    App->>NAPI: createCalendar(calendarAccount)
    NAPI->>CM: 创建日历
    CM->>DSH: CreateInnerDataShareHelper(WRITE_URI)
    DSH->>Ability: insert()
    Ability->>Auth: insertByProxy(uri, value, WRITE_CALENDAR)
    Auth->>Delegate: verifyByUri(uri, value, LOW_PERMISSION)
    Delegate->>Factory: getDatabaseProcessor(table)
    Factory->>Processor: new CalendarsProcessor()
    Processor->>Delegate: insertByLowAuthority()
    Delegate->>RDB: insert(valuesBucket)
    RDB-->>Delegate: rowId
    Delegate-->>Auth: rowId
    Auth-->>Ability: rowId
    Ability-->>DSH: rowId
    DSH-->>CM: rowId
    CM-->>NAPI: calendarId
    NAPI-->>App: 成功
```

## 权限验证流程

### 权限级别判定

```mermaid
flowchart TD
    A[收到请求] --> B{权限类型}
    B -->|*_WHOLE_CALENDAR| C[返回 HIGH_LEVEL = 2]
    B -->|*_CALENDAR| D[返回 LOW_LEVEL = 1]
    B -->|其他| E[返回 UNAUTHORIZED = -1]

    C --> F{权限级别}
    D --> F
    E --> F

    F -->|HIGH| G[dataOperateSelectorByHighAuthority]
    F -->|LOW| H[dataOperateSelectorByLowAuthority]
    F -->|UNAUTHORIZED| I[返回权限错误]

    G --> J[执行高权限操作]
    H --> K[执行低权限操作]
```

### C++ 层权限检查

calendarmanager/native/src/data_share_helper_manager.cpp:28-86

```cpp
// 权限名称常量
const std::string READ_PERMISSION_NAME  = "ohos.permission.READ_WHOLE_CALENDAR";
const std::string WRITE_PERMISSION_NAME = "ohos.permission.WRITE_WHOLE_CALENDAR";

// 创建 DataShareHelper 时检查权限
std::shared_ptr<DataShareHelper> DataShareHelperManager::CreateInnerDataShareHelper(
    const std::string &permissionUri) {
    // ...

    // 根据操作类型选择权限名称
    std::string permissionName = isRead ? READ_PERMISSION_NAME : WRITE_PERMISSION_NAME;

    // 验证调用者权限
    ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        IPCSkeleton::GetCallingTokenID(), permissionName);

    if (ret == Security::AccessToken::PERMISSION_GRANTED) {
        LOG_INFO("DataShareHelper in high permission");
        // 创建高权限 DataShareHelper
    } else {
        LOG_INFO("DataShareHelper in low permission");
        // 创建低权限 DataShareHelper
    }
}
```

### ArkTS 层权限检查

dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:112-165

```typescript
async function verifyByUri(
    uri: string,
    dataParameter: DataParameter,
    highPermission: Permissions,
    lowPermission: Permissions,
    permission: Permissions,
    callback: Function): Promise<number> {
    
    let bundleNameAndTokenId = getBundleNameAndTokenIDByUri(uri);

    // 验证高权限
    if (permission === PERMISSIONS_READ_WHOLE_CALENDAR ||
        permission === PERMISSIONS_WRITE_WHOLE_CALENDAR) {
        // 封装回调添加权限使用记录
        dataParameter.callback = (err: BusinessError, rowId: number) => {
            callback(err, rowId);
            if (isNeedAddPermissionUsedRecord(uri, dataParameter)) {
                addPermissionUsedRecordInCallBack(err, bundleNameAndTokenId.tokenId, highPermission);
            }
        };
        return PERMISSIONS_FLAG_HIGH;  // 2
    }

    // 验证低权限
    if (permission === PERMISSIONS_READ_CALENDAR ||
        permission === PERMISSIONS_WRITE_CALENDAR) {
        // 封装回调添加权限使用记录
        dataParameter.callback = (err: BusinessError, rowId: number) => {
            callback(err, rowId);
            if (isNeedAddPermissionUsedRecord(uri, dataParameter)) {
                addPermissionUsedRecordInCallBack(err, bundleNameAndTokenId.tokenId, lowPermission);
            }
        };
        return PERMISSIONS_FLAG_LOW;  // 1
    }

    return PERMISSIONS_FLAG_UNAUTHORIZED;  // -1
}
```

## 线程模型

### 主线程

- **calendarmanager**: N-API 绑定在主线程执行
- **entry/DataShareExtAbility**: ArkTS 代码在主线程执行

### 异步处理

- 数据库操作通过异步 Promise/Callback 模式执行
- 权限检查通过 async/await 模式

### 线程同步

- 使用 Mutex 保护共享资源
- RDB 操作通过关系数据库内部锁机制保证一致性

## 关键时序

### 初始化时序

```mermaid
sequenceDiagram
    participant System as 系统启动
    participant Entry as Entry Ability
    participant Proxy as AuthenticateProxy
    participant Delegate as DataShareDelegate
    participant RDB as RDB

    System->>Entry: 启动 DataShareExtAbility
    Entry->>Entry: onCreate()
    Entry->>Entry: 初始化 GlobalThis
    Entry->>Proxy: init()
    Proxy->>Delegate: init()
    Delegate->>Delegate: 获取 RdbStore
    Delegate->>Delegate: insertDefaultCalendar()
    Delegate->>RDB: 插入默认日历
    RDB-->>Delegate: 成功
    Delegate-->>Proxy: true
    Proxy-->>Entry: true
    Entry-->>System: 就绪
```

## 模块依赖

### 依赖方向

```
datastructure (无依赖)
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

### 无环依赖

所有模块依赖形成 DAG（有向无环图），不存在循环依赖。

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [对外 API](03_External_API.md) - N-API 完整清单
- [内部 API](04_Internal_API.md) - 模块接口详情
- [安全评审](07_Security_Review.md) - 权限机制深入分析

---

返回 [目录](SUMMARY.md) | [首页](README.md)
