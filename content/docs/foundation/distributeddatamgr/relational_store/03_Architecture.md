# 系统架构说明

## 目的

本文档详细描述 relational_store 组件的架构设计，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- relational_store 架构设计
- 组件间交互关系
- 数据流转路径
- 并发模型与线程安全
- 关键时序与调用链

## 关键结论

### 1. 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层（Application Layer）           │
│  - JS 应用（ArkTS）                                    │
│  - ETS 应用（ArkTS）                                 │
│  - 仓颉应用（Cangjie）                              │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                N-API 绑定层（N-API Layer）          │
│  - data.rdb (Legacy)                                    │
│  - data.relationalStore (New)                              │
│  - data.cloudData                                          │
│  - data.dataAbility                                         │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              Inner API 接口层（Inner API Layer）        │
│  - RdbStore, RdbHelper (核心接口）                      │
│  - CloudManager, CloudService (云同步接口）                │
│  - ISharedResultSet, DataAbilityPredicates（适配接口）       │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               核心实现层（Core Implementation）        │
│  - RdbStoreImpl (数据库操作）                            │
│  - ConnectionPool (连接池）                               │
│  - RdbSecurityManager (加密管理）                         │
│  - CloudManager (云同步管理）                              │
└──────────────────────────┬────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │   SQLite     │  │    IPC       │  │    HUKS       │
    │   引擎        │  │   通信        │  │   加密         │
    └──────────────┘  └──────────────┘  └──────────────┘
```

### 2. 组件图

#### 2.1 核心组件

| 组件 | 职责 | 位置 | 证据 |
|------|------|------|------|
| **RdbStore** | 数据库核心接口，定义 CRUD、事务、同步操作 | `interfaces/inner_api/rdb/include/rdb_store.h` |
| **RdbStoreImpl** | RdbStore 实现，封装 SQLite 操作 | `frameworks/native/rdb/src/rdb_store_impl.cpp` |
| **ConnectionPool** | SQLite 连接池，管理最多 4 个连接 | `frameworks/native/rdb/src/connection_pool.cpp` |
| **RdbHelper** | 数据库创建和升级助手 | `frameworks/native/rdb/src/rdb_helper.cpp` |
| **RdbPredicates** | 类型安全的查询谓词构建器 | `frameworks/native/rdb/src/rdb_predicates.cpp` |
| **ResultSet** | 查询结果集，支持迭代访问 | `frameworks/native/rdb/src/step_result_set.cpp` |
| **RdbSecurityManager** | 加密密钥管理和数据库加解密 | `frameworks/native/rdb/src/rdb_security_manager.cpp` |
| **CloudManager** | 云同步管理器，协调设备间和云端同步 | `frameworks/native/cloud_data/src/cloud_manager.cpp` |
| **RdbNotifierStub** | 数据变更通知的 IPC 存根 | `frameworks/native/rdb/src/rdb_notifier_stub.cpp` |

#### 2.2 IPC 组件

| 组件 | 职责 | 接口 | 证据 |
|------|------|------|------|
| **IRdbService** | RDB 服务接口 | IRemoteBroker | `frameworks/native/rdb/include/irdb_service.h` |
| **RdbServiceProxy** | RDB 服务客户端代理 | IRemoteProxy | `frameworks/native/rdb/include/rdb_service_proxy.h` |
| **ICloudService** | 云服务接口 | IRemoteBroker | `frameworks/native/cloud_data/include/icloud_service.h` |
| **CloudServiceProxy** | 云服务客户端代理 | IRemoteProxy | `frameworks/native/cloud_data/include/cloud_service_proxy.h` |
| **ISharedResultSet** | 跨进程共享结果集接口 | IRemoteBroker | `interfaces/inner_api/dataability/include/ishared_result_set.h` |

### 3. 数据流

#### 3.1 查询流程

```mermaid
sequenceDiagram
    participant App as 应用层
    participant NAPI as N-API 绑定
    participant Inner as Inner API
    participant Core as 核心实现
    participant Pool as 连接池
    participant SQLite as SQLite 引擎

    App->>NAPI: 调用 query(predicates, columns)
    NAPI->>Inner: Query(predicates, columns)
    Inner->>Core: Query(predicates, columns)
    Core->>Pool: GetConnection()
    Pool-->>Core: 返回 SQLite 连接
    Core->>SQLite: 执行 SQL 语句
    SQLite-->>Core: 返回结果集
    Core->>Pool: ReleaseConnection()
    Core-->>Inner: 返回 ResultSet
    Inner-->>NAPI: 封装为 JS ResultSet 对象
    NAPI-->>App: 返回结果集

    Note over SQLite: SQL 语句由 Predicates 构建
    Note over Pool: 连接池复用 SQLite 连接
```

#### 3.2 写操作流程

```mermaid
sequenceDiagram
    participant App as 应用层
    participant NAPI as N-API 绑定
    participant Inner as Inner API
    participant Core as 核心实现
    participant Pool as 连接池
    participant SQLite as SQLite 引擎
    participant Sec as 加密管理

    App->>NAPI: 调用 insert(table, values)
    NAPI->>Inner: Insert(table, values)
    Inner->>Core: Insert(table, values)
    Core->>Pool: GetConnection()
    Pool-->>Core: 返回写连接（EXCLUSIVE）
    Core->>Sec: 获取加密密钥
    Sec-->>Core: 返回密钥
    Core->>SQLite: 执行 INSERT 语句
    SQLite-->>Core: 返回行 ID
    Core->>Pool: ReleaseConnection()
    Core-->>Inner: 返回插入结果
    Inner-->>NAPI: 封装为 JS 结果
    NAPI-->>App: 返回行 ID

    Note over Pool: 写操作获取 EXCLUSIVE 连接
    Note over Sec: 如果数据库已加密，使用密钥解密
```

#### 3.3 云同步流程

```mermaid
sequenceDiagram
    participant App as 应用层
    participant CloudMgr as CloudManager
    participant RdbStore as RdbStore
    participant CloudSvc as CloudService Proxy
    participant DataMgr as DataMgr Service

    App->>CloudMgr: sync(option, tables, asyncCallback)
    CloudMgr->>RdbStore: 查询本地待同步数据
    RdbStore-->>CloudMgr: 返回待同步数据
    CloudMgr->>CloudSvc: 上传数据到云端
    CloudSvc->>DataMgr: 通过 IPC 调用云服务
    DataMgr-->>CloudSvc: 返回云端结果
    CloudSvc-->>CloudMgr: 返回同步结果
    CloudMgr->>RdbStore: 更新本地数据（冲突解决）
    CloudMgr->>App: 通过回调通知同步完成

    Note over CloudSvc: 通过 SystemAbility 加载云服务
    Note over RdbStore: 支持分布式表标记和查询
```

### 4. 线程模型

#### 4.1 连接池模型

| 特性 | 说明 | 证据 |
|------|------|------|
| **连接池大小** | 最大 4 个 SQLite 连接 | `README_zh.md:46` |
| **读连接** | 支持多个并发读操作 | `connection_pool.cpp` |
| **写连接** | 同一时间只支持一个写操作 | `README_zh.md:47` |
| **连接复用** | 连接用完归还池中，不立即关闭 | `connection_pool.cpp` |
| **线程安全** | 使用互斥锁保护连接池 | `connection_pool.cpp` |

**连接池状态机**：
```
空闲状态 ←→ 借出状态 ←→ 使用中 ←→ 归还状态 ←→ 空闲状态
                      ↓
                     关闭状态（异常）
```

#### 4.2 事务模型

| 事务类型 | 隔离级别 | 说明 | 证据 |
|----------|----------|------|------|
| **EXCLUSIVE** | 独占式 | 开始事务时锁定数据库 | `rdb_store.h:597-614` |
| **IMMEDIATE** | 即时式 | 有其他 EXCLUSIVE 事务时失败 | SQLite 默认 |
| **DEFERRED** | 延迟式 | 第一次读操作时锁定 | SQLite 默认 |

**事务生命周期**：
```mermaid
stateDiagram-v2
    [*] --> BEGIN: BeginTrans()
    BEGIN --> EXECUTING: 执行 SQL 语句
    EXECUTING --> EXECUTING: 继续执行
    EXECUTING --> COMMIT: Commit()
    EXECUTING --> ROLLBACK: RollBack()
    COMMIT --> [*]: 事务提交
    ROLLBACK --> [*]: 事务回滚
```

#### 4.3 观察者模式

| 组件 | 职责 | 线程 | 证据 |
|------|------|------|------|
| **RdbStoreObserver** | 数据变更订阅 | 用户线程 | `rdb_store.h:714-729` |
| **SubscribeMode** | 订阅模式：LOCAL、REMOTE、LOCAL_DETAIL | - | `rdb_types.h` |
| **DetailProgressObserver** | 云同步进度观察者 | 工作线程 | `rdb_types.h:80-82` |
| **RdbNotifierStub** | IPC 存根，处理跨进程通知 | Binder 线程 | `rdb_notifier_stub.h` |

**观察者注册流程**：
```mermaid
sequenceDiagram
    participant App as 应用
    participant Rdb as RdbStore
    participant Obs as 观察者

    App->>Rdb: Subscribe(option, observer)
    Rdb->>Rdb: 注册观察者到内部列表
    Rdb-->>App: 返回注册结果

    Note over Rdb: 数据变更时触发回调

    数据变更->>Rdb: 检测到数据变化
    Rdb->>Obs: OnChange(data)
    Obs->>App: 应用处理数据变更
```

### 5. 关键时序

#### 5.1 数据库打开流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as RdbHelper
    participant Impl as RdbStoreImpl
    participant Sec as SecurityManager
    participant Pool as ConnectionPool
    participant SQLite as SQLite

    App->>Helper: getRdbStore(config, callback)
    Helper->>Sec: 检查数据库加密
    Sec-->>Helper: 返回加密状态和密钥
    Helper->>Impl: 创建 RdbStoreImpl 实例
    Impl->>Pool: 初始化连接池
    Pool->>SQLite: 打开数据库文件（带加密密钥）
    SQLite-->>Pool: 返回 SQLite 连接
    Pool-->>Impl: 连接池初始化完成
    Impl->>Helper: 回调 onCreate/onUpgrade
    Helper-->>App: 返回 RdbStore 实例
```

#### 5.2 分布式查询流程

```mermaid
sequenceDiagram
    participant LocalApp as 本地应用
    participant LocalRdb as 本地 RdbStore
    participant IPCMgr as IPC Manager
    participant RemoteSvc as 远程 RdbService
    participant RemoteRdb as 远程 RdbStore

    LocalApp->>LocalRdb: remoteQuery(device, predicates, columns)
    LocalRdb->>IPCMgr: 获取远程设备服务代理
    IPCMgr-->>LocalRdb: 返回 RdbServiceProxy
    LocalRdb->>RemoteSvc: Query(predicates, columns)
    RemoteSvc->>RemoteRdb: 调用远程 Query
    RemoteRdb-->>RemoteSvc: 返回结果集
    RemoteSvc-->>LocalRdb: 返回查询结果
    LocalRdb-->>LocalApp: 返回远程数据

    Note over IPCMgr: 通过 SystemAbilityManager 加载
```

#### 5.3 云同步详细流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant CloudMgr as CloudManager
    participant Rdb as 本地 RdbStore
    participant Proxy as CloudServiceProxy
    participant CloudSvc as 云服务
    participant DataMgr as DataMgr Service

    App->>CloudMgr: Sync(SyncOption, tables, asyncCallback)
    CloudMgr->>Rdb: 获取分布式表数据
    Rdb-->>CloudMgr: 返回待同步数据
    CloudMgr->>Proxy: 同步数据到云端
    Proxy->>CloudSvc: UploadData(data)
    CloudSvc->>DataMgr: 调用云数据上传接口
    DataMgr-->>CloudSvc: 返回云端数据
    CloudSvc-->>CloudMgr: 返回云端数据
    CloudMgr->>Rdb: 更新本地数据（冲突解决）
    CloudMgr->>App: asyncCallback.onComplete(result)

    Note over CloudSvc: 需要权限 ohos.permission.CLOUDDATA_CONFIG
```

### 6. 内存管理与资源生命周期

| 资源类型 | 创建方式 | 销毁方式 | 所有权 | 证据 |
|----------|----------|----------|--------|------|
| **ResultSet** | Query() 返回共享指针 | Close() 或析构 | RdbStore | `result_set.h` |
| **Transaction** | CreateTransaction() 返回 | Commit()、RollBack() 或析构 | RdbStore | `transaction.h` |
| **Connection** | 连接池借出 | 归还连接池 | ConnectionPool | `connection_pool.cpp` |
| **观察者** | Subscribe() 注册 | Unsubscribe() 注销 | 用户代码 | `rdb_store.h` |

### 7. 错误传播机制

```mermaid
graph LR
    SQLite[SQLite 错误] --> RdbStore[RdbStore 转换<br/>rdb_errno.h]
    RdbStore --> InnerAPI[Inner API<br/>传播错误码]
    InnerAPI --> NAPI[N-API 绑定<br/>转换为 JS Error]
    NAPI --> App[应用层<br/>抛出异常]

    RdbStore[安全检查失败] --> RdbStore[RdbSecurityManager<br/>加密失败]
    RdbStore[权限检查失败] --> InnerAPI[AccessToken<br/>权限拒绝]

    style SQLite fill:#ff9999
    style RdbStore fill:#ffcc00
    style InnerAPI fill:#ff6600
    style NAPI fill:#ff3300
    style App fill:#ff0000
```

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构与模块职责
- [05_Inner_API.md](./05_Inner_API.md) - 内部接口定义
- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 详细调用链

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: rdb_store.h, connection_pool.cpp, cloud_manager.cpp, rdb_notifier_stub.h, transaction.h
