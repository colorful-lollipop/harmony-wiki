# 关键调用链示例

## 目的

本文档展示 relational_store 组件中关键操作的调用链，从入口点到核心逻辑的完整路径。

## 适用范围

- 关键操作的完整调用链
- 跨模块调用关系
- 数据流和转换过程

## 1. 数据库打开流程

### 1.1 RdbStore 创建（JS → N-API → Core）

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as NAPI 层
    participant Helper as RdbHelper
    participant Impl as RdbStoreImpl
    participant Pool as ConnectionPool
    participant Sec as SecurityManager
    participant SQLite as SQLite 引擎

    App->>NAPI: rdbStoreHelper.getRdbStore(config)
    Note over NAPI: config 包含 name, securityLevel 等

    NAPI->>Helper: GetRdbStore(config, callback)
    Helper->>Sec: CheckDatabaseSecurity(config)
    Sec-->>Helper: 返回加密密钥（如果已加密）
    Helper->>Impl: CreateRdbStoreImpl(config, key)
    Impl->>Pool: Initialize(4)
    Note over Pool: 最多 4 个连接

    Pool->>SQLite: sqlite3_open_v2(path, key)
    Note over SQLite: 如果已加密，使用 key 参数

    SQLite-->>Pool: 返回 sqlite3* 连接句柄
    Pool-->>Impl: 连接池初始化完成

    Impl->>Helper: 回调 callback.onCreate(version)
    Helper-->>NAPI: 转换为 JS RdbStore 对象
    NAPI-->>App: 返回 RdbStore 实例

    Note over App: 现在 App 可以调用 insert/query 等方法
```

**关键文件与代码位置**：
- N-API 入口：`frameworks/js/napi/relationalstore/src/napi_rdb_store_helper.cpp:145-180`
- Helper 实现：`frameworks/native/rdb/src/rdb_helper.cpp:40-85`
- Core 实现：`frameworks/native/rdb/src/rdb_store_impl.cpp:1-200`
- 连接池：`frameworks/native/rdb/src/connection_pool.cpp:23-178`

## 2. 查询操作流程

### 2.1 Query 谓用（JS → N-API → Core → SQLite）

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as NAPI 层
    participant Predicates as RdbPredicates
    participant Impl as RdbStoreImpl
    participant Builder as SqliteSqlBuilder
    participant Pool as ConnectionPool
    participant SQLite as SQLite 引擎

    App->>NAPI: rdbStore.query(predicates, columns)
    NAPI->>Predicates: 初始化 Predicates 对象
    Predicates->>Predicates: equalTo("name", "value")
    Predicates->>Predicates: and()
    Predicates->>Predicates: greaterThan("age", 18)

    NAPI->>Impl: Query(predicates, columns)
    Impl->>Builder: BuildSql(predicates, columns)
    Note over Builder: 将谓词转换为 SQL 语句

    Builder->>Builder: AppendConditions()
    Builder->>Builder: AppendColumns()
    Builder->>Builder: AppendOrderBy()
    Builder-->>Impl: 返回 SQL 语句："SELECT id, name FROM test_table WHERE name = ? AND age > ? ORDER BY id DESC"

    Impl->>Pool: GetConnection(READ_ONLY)
    Note over Pool: 读操作可以并发

    Pool-->>Impl: 返回读连接（可能复用）
    Impl->>SQLite: sqlite3_prepare_v2(connection, sql)
    SQLite-->>Impl: 返回准备好的语句句柄

    Impl->>SQLite: sqlite3_bind_text(stmt, 1, "value")
    Impl->>SQLite: sqlite3_bind_int(stmt, 2, 18)
    Note over SQLite: 参数化查询，防止 SQL 注入

    Impl->>SQLite: sqlite3_step(stmt)
    SQLite-->>Impl: 返回行数据
    Impl->>SQLite: sqlite3_finalize(stmt)
    Note over SQLite: 释放语句资源

    Impl->>Pool: ReleaseConnection(connection)
    Note over Pool: 连接归还到池中

    Impl->>Impl: 创建 StepResultSet 对象
    Impl->>Impl: 用查询结果初始化结果集

    Impl-->>NAPI: 封装为 JS ResultSet 对象
    NAPI-->>App: 返回 Promise<ResultSet>
```

**关键文件与代码位置**：
- N-API Query：`frameworks/js/napi/relationalstore/src/napi_rdb_store.cpp:398-456`
- Predicates：`frameworks/js/napi/relationalstore/src/napi_rdb_predicates.cpp`
- SQL Builder：`frameworks/native/rdb/src/sqlite_sql_builder.cpp:89-234`
- Core Query：`frameworks/native/rdb/src/rdb_store_impl.cpp:523-654`

## 3. 插入操作流程

### 3.1 Insert 调用链（JS → N-API → Core → SQLite）

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as NAPI 层
    participant Impl as RdbStoreImpl
    participant Pool as ConnectionPool
    participant Sec as SecurityManager
    participant SQLite as SQLite 引擎

    App->>NAPI: rdbStore.insert(table, values)
    Note over NAPI: values 是 ValuesBucket 对象

    NAPI->>Impl: Insert(table, values)
    Impl->>Sec: GetEncryptKey()
    Sec-->>Impl: 返回加密密钥（如果数据库已加密）

    Impl->>Pool: GetConnection(EXCLUSIVE)
    Note over Pool: 写操作需要 EXCLUSIVE 锁

    Pool-->>Impl: 返回独占连接

    Impl->>Sec: EncryptValues(values, key)
    Note over Sec: 加密敏感数据（如果配置了加密级别）

    Impl->>Builder: BuildInsertSql(table, values)
    Impl-->>Impl: 返回 INSERT 语句："INSERT INTO test_table (name, age) VALUES (?, ?)"

    Impl->>SQLite: sqlite3_prepare_v2(connection, sql)
    SQLite-->>Impl: 返回语句句柄

    Impl->>SQLite: sqlite3_bind_text(stmt, 1, "John")
    Impl->>SQLite: sqlite3_bind_int(stmt, 2, 25)
    Note over SQLite: 参数化绑定

    Impl->>SQLite: sqlite3_step(stmt)
    SQLite-->>Impl: 返回 SQLITE_DONE

    Impl->>SQLite: sqlite3_finalize(stmt)
    Impl->>SQLite: sqlite3_last_insert_rowid(connection, &rowId)
    Note over SQLite: 获取插入的行 ID

    Impl->>Pool: ReleaseConnection(connection)
    Note over Pool: 释放写连接

    Impl-->>NAPI: 返回 Promise.resolve(rowId)
    NAPI-->>App: 返回插入的行 ID
```

**关键文件与代码位置**：
- N-API Insert：`frameworks/js/napi/relationalstore/src/napi_rdb_store.cpp:227-352`
- Core Insert：`frameworks/native/rdb/src/rdb_store_impl.cpp:166-218`
- Security Manager：`frameworks/native/rdb/src/rdb_security_manager.cpp:89-134`

## 4. 云同步流程

### 4.1 云同步调用链（JS → NAPI → CloudManager → CloudService）

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as CloudData NAPI
    participant CloudMgr as CloudManager
    participant RdbStore as 本地 RdbStore
    participant Proxy as CloudServiceProxy
    participant Service as DataMgr Service
    participant Cloud as 云端服务

    App->>NAPI: cloudData.cloudSync(tables, mode)
    NAPI->>CloudMgr: Sync(tables, mode, asyncCallback)
    Note over NAPI: mode 可选 LOCAL、REMOTE、CLOUD

    CloudMgr->>RdbStore: GetDistributedTables()
    RdbStore-->>CloudMgr: 返回标记为可同步的表列表

    CloudMgr->>RdbStore: QueryEachTable(predicates)
    RdbStore-->>CloudMgr: 返回每个表的待同步数据

    CloudMgr->>Proxy: CheckCloudConnection()
    Proxy->>Service: Ping()
    Service-->>Proxy: 返回在线状态

    CloudMgr->>Proxy: UploadToCloud(data)
    Proxy->>Service: Upload(data)
    Note over Service: 通过 IPC 调用云端存储

    Service->>Cloud: 处理上传
    Cloud-->>Service: 返回上传结果（版本、hash 等）

    Service-->>Proxy: 返回上传结果
    Proxy-->>CloudMgr: 返回云端数据

    CloudMgr->>Proxy: DownloadFromCloud()
    Proxy->>Service: Download()
    Service->>Cloud: 处理下载
    Cloud-->>Service: 返回云端数据
    Service-->>Proxy: 返回云端数据
    Proxy-->>CloudMgr: 返回云端数据

    CloudMgr->>RdbStore: MergeCloudData(localData, cloudData)
    Note over RdbStore: 冲突解决和数据合并

    RdbStore-->>CloudMgr: 返回合并结果

    CloudMgr->>App: asyncCallback.onComplete(result)
    NAPI-->>App: 返回 Promise.resolve(result)
```

**关键文件与代码位置**：
- N-API Cloud Sync：`frameworks/js/napi/cloud_data/src/js_config.cpp:145-189`
- Cloud Manager：`frameworks/native/cloud_data/src/cloud_manager.cpp:1-300`
- Service Proxy：`frameworks/native/cloud_data/src/cloud_service_proxy.cpp`

## 5. 事务操作流程

### 5.1 事务生命周期（JS → N-API → Core）

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as NAPI 层
    participant Trans as Transaction
    participant Impl as RdbStoreImpl
    participant Pool as ConnectionPool

    App->>NAPI: rdbStore.beginTransaction()
    NAPI->>Impl: CreateTransaction(EXCLUSIVE)
    Impl->>Pool: GetConnection(EXCLUSIVE)
    Pool-->>Impl: 返回独占连接

    Impl->>Impl: 创建 TransactionImpl 对象
    Impl-->>NAPI: 返回 JS Transaction 对象

    App->>NAPI: transaction.insert(table, values)
    NAPI->>Trans: Insert(table, values)
    Trans->>Impl: Insert(table, values, trxId)
    Note over Impl: 使用保存的事务 ID

    Impl->>Impl: 执行 INSERT（在事务内）
    Trans-->>NAPI: 返回 Promise.resolve(rowId)

    App->>NAPI: transaction.commit()
    NAPI->>Trans: Commit(trxId)
    Trans->>Impl: Commit(trxId)
    Impl->>Impl: 执行 COMMIT SQL
    Impl-->>Trans: 返回提交结果
    Trans-->>NAPI: 返回 Promise.resolve(void)

    App->>NAPI: transaction.rollBack() [如果出错]
    NAPI->>Trans: RollBack(trxId)
    Trans->>Impl: RollBack(trxId)
    Impl->>Impl: 执行 ROLLBACK SQL
    Trans-->>NAPI: 返回 Promise.resolve(void)
```

**关键文件与代码位置**：
- N-API Transaction：`frameworks/js/napi/relationalstore/src/napi_transaction.cpp`
- Core Transaction：`frameworks/native/rdb/src/transaction_impl.cpp`
- RdbStore Transaction：`frameworks/native/rdb/src/rdb_store_impl.cpp:593-614`

## 6. 观察者通知流程

### 6.1 数据变更通知（Core → IPC Stub → Remote）

```mermaid
sequenceDiagram
    participant LocalApp as 本地应用 A
    participant RdbStore as 本地 RdbStore
    participant Notifier as RdbNotifierStub
    participant Binder as Binder IPC
    participant RemoteApp as 远程应用 B

    LocalApp->>RdbStore: Subscribe(option, observer)
    RdbStore->>RdbStore: 注册观察者到内部列表
    RdbStore-->>LocalApp: 返回注册结果

    Note over RdbStore: 数据发生变化

    RdbStore->>Notifier: OnDataChange(table, data)
    Notifier->>Notifier: 构造通知数据

    Notifier->>Binder: SendRequest(code, data)
    Note over Binder: 通过 Binder IPC 发送跨进程通知

    Binder->>Binder: 转发到远程进程
    Binder->>RemoteApp: Deliver(data)

    RemoteApp->>RemoteApp: 接收通知
    RemoteApp->>RemoteApp: OnChange(data)
    RemoteApp->>RemoteApp: 更新 UI
```

**关键文件与代码位置**：
- Notifier Stub：`frameworks/native/rdb/src/rdb_notifier_stub.cpp`
- RdbStore Observer：`frameworks/native/rdb/src/rdb_store_impl.cpp:714-729`

## 7. 加密数据库打开流程

### 7.1 带加密的数据库打开（Helper → SecurityManager → SQLite）

```mermaid
sequenceDiagram
    participant NAPI as NAPI 层
    participant Helper as RdbHelper
    participant Sec as SecurityManager
    participant SQLite as SQLite 引擎

    NAPI->>Helper: GetRdbStore(config, callback)
    Note over NAPI: config.securityLevel = S1-S4

    Helper->>Sec: CheckDatabaseEncryption(path)
    Sec->>Sec: 读取加密文件头
    Sec-->>Helper: 返回加密状态和版本号

    Helper->>Sec: GetEncryptKey()
    Note over Sec: 从 HUKS 获取密钥或从密钥文件读取

    Sec-->>Helper: 返回密钥对象（key, version）

    Helper->>Helper: OpenDatabaseWithKey(path, key)
    Note over Helper: 调用 SQLite 的加密 API

    Helper->>SQLite: sqlite3_open_v2(path, &db, SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE, key)
    Note over SQLite: 使用 key 参数打开加密数据库

    SQLite-->>Helper: 返回数据库句柄

    Helper->>Helper: 检查数据库版本
    Helper->>Helper: 如果需要升级，调用 callback.onUpgrade()

    Helper->>Helper: 创建 RdbStoreImpl 实例
    Helper-->>NAPI: 返回 RdbStore 实例（已加密）
```

**关键文件与代码位置**：
- RdbHelper：`frameworks/native/rdb/src/rdb_helper.cpp:40-85`
- Security Manager：`frameworks/native/rdb/src/rdb_security_manager.cpp:145-180`

## 8. 分布式表查询流程

### 8.1 远程设备查询（JS → RdbStore → RemoteService）

```mermaid
sequenceDiagram
    participant LocalApp as 本地应用
    participant RdbStore as 本地 RdbStore
    participant IPCMgr as IPC Manager
    participant RemoteSvc as 远程 RdbService
    participant RemoteRdb as 远程 RdbStore

    LocalApp->>RdbStore: remoteQuery(deviceId, predicates, columns)
    Note over LocalApp: 查询远程设备的表数据

    RdbStore->>IPCMgr: GetRemoteRdbService(deviceId)
    IPCMgr->>IPCMgr: 通过 SystemAbilityManager 获取远程服务代理

    IPCMgr-->>RdbStore: 返回 Remote RdbServiceProxy

    RdbStore->>RemoteSvc: Query(predicates, columns)
    Note over RemoteSvc: 跨 IPC 调用

    RemoteSvc->>RemoteRdb: Query(predicates, columns)
    Note over RemoteRdb: 执行本地查询

    RemoteRdb-->>RemoteSvc: 返回 ResultSet

    RemoteSvc-->>RdbStore: 返回查询结果

    RdbStore->>RemoteSvc: ShareResultSet(resultSet)
    Note over RdbStore: 将 ResultSet 转换为共享内存块

    RemoteSvc-->>RdbStore: 返回共享结果

    RdbStore-->>LocalApp: 返回 ResultSet（包含远程数据）
```

**关键文件与代码位置**：
- Remote Query：`frameworks/native/rdb/src/rdb_store_impl.cpp:465-483`
- IPC Manager：`frameworks/native/rdb/src/rdb_manager_impl.cpp:1-145`
- Service Proxy：`frameworks/native/rdb/src/rdb_service_proxy.cpp`

## 调用链总结

| 操作 | 入口点 | 核心逻辑 | 数据存储 | 关键文件 |
|------|----------|----------|----------|------------|
| **打开数据库** | RdbHelper::GetRdbStore() | SecurityManager + SQLite | 本地文件 | rdb_helper.cpp<br/>rdb_security_manager.cpp |
| **查询数据** | RdbStore::Query() | SqliteSqlBuilder → SQLite | 本地表 | sqlite_sql_builder.cpp<br/>rdb_store_impl.cpp |
| **插入数据** | RdbStore::Insert() | SecurityManager → SQLite | 本地表 | rdb_security_manager.cpp<br/>rdb_store_impl.cpp |
| **云同步** | CloudManager::Sync() | CloudServiceProxy → 云端 | 本地 + 云端 | cloud_manager.cpp<br/>cloud_service_proxy.cpp |
| **事务** | RdbStore::CreateTransaction() | TransactionImpl → SQLite | 本地表 | transaction_impl.cpp<br/>rdb_store_impl.cpp |
| **观察者** | RdbNotifierStub::OnDataChange() | Binder IPC | 跨进程 | rdb_notifier_stub.cpp |

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: 各核心实现文件、头文件定义
