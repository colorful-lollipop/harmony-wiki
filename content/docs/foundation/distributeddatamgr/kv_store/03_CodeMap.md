# 代码地图

## 概述

本文档提供 KV Store 代码库的导航地图，帮助开发者快速定位关键文件和模块。

## 顶层目录结构

```
kv_store/
├── frameworks/           # 框架层实现代码
├── interfaces/           # 接口声明（JS/C++/C）
├── databaseutils/        # 数据库工具类
├── kvstoremock/          # Mock实现（测试辅助）
├── test/                 # 测试代码（本文档不覆盖）
├── wiki/                 # Wiki文档
├── bundle.json           # 组件配置
├── BUILD.gn              # GN构建入口
└── kv_store.gni          # GN配置片段
```

---

## 接口层代码地图（interfaces/）

### JS API 声明（interfaces/jskits/）

| 模块 | 文件路径 | 职责 | 关键导出 |
|-----|---------|------|---------|
| distributedkvstore | `interfaces/jskits/distributedkvstore/distributed_kvstore.js` | JS API 声明 | `createKVManager`, 常量枚举 |
| distributeddata | `interfaces/jskits/distributeddata/distributed_data.js` | 分布式数据API | 旧版兼容接口 |

**证据**：`interfaces/jskits/distributedkvstore/distributed_kvstore.js:17-18`
```javascript
// 导出 createKVManager
module.exports = {
    createKVManager: distributedDataSo.createKVManager,
```

### Inner API 头文件（interfaces/innerkits/）

| 头文件 | 路径 | 职责 | 核心类/函数 |
|-------|------|------|-----------|
| distributed_kv_data_manager.h | `interfaces/innerkits/distributeddata/include/` | KV管理器 | `DistributedKvDataManager` |
| kvstore.h | `interfaces/innerkits/distributeddata/include/` | KV存储基类 | `KvStore` |
| single_kvstore.h | `interfaces/innerkits/distributeddata/include/` | 单版本存储 | `SingleKvStore` |
| data_query.h | `interfaces/innerkits/distributeddata/include/` | 查询构建 | `DataQuery` |
| types.h | `interfaces/innerkits/distributeddata/include/` | 类型定义 | `Key`, `Value`, `Entry` |
| store_errno.h | `interfaces/innerkits/distributeddata/include/` | 错误码 | `Status` 枚举 |
| kvstore_result_set.h | `interfaces/innerkits/distributeddata/include/` | 结果集 | `KvStoreResultSet` |
| kvstore_observer.h | `interfaces/innerkits/distributeddata/include/` | 观察者 | `KvStoreObserver` |
| distributed_data_mgr.h | `interfaces/innerkits/distributeddatamgr/include/` | 数据管理器 | `DistributedDataMgr` |

**证据**：`interfaces/innerkits/distributeddata/include/distributed_kv_data_manager.h:35`
```cpp
class API_EXPORT DistributedKvDataManager {
public:
    Status GetSingleKvStore(...);
```

### Native API（interfaces/inner_api/）

| 模块 | 路径 | 职责 | 关键文件 |
|-----|------|------|---------|
| kv_store | `interfaces/inner_api/kv_store/` | C API | `kvstore_env.h` |
| dbm_kv_store | `interfaces/inner_api/dbm_kv_store/` | DBM风格API | `dbm_kv_store.h` |

**证据**：`interfaces/inner_api/kv_store/include/kvstore_env.h`
```cpp
// C API 环境定义
```

---

## 实现层代码地图（frameworks/）

### JS API 实现（frameworks/jskitsimpl/）

#### distributedkvstore 模块

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| entry_point.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 模块注册入口 | `Init()`, `napi_module_register` |
| js_kv_manager.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | KV管理器实现 | `JsKVManager::CreateKVManager` |
| js_single_kv_store.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | SingleKVStore实现 | `JsSingleKVStore::Put/Get/Delete` |
| js_device_kv_store.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | DeviceKVStore实现 | `JsDeviceKVStore::Get/RemoveDeviceData` |
| js_query.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | Query构建器 | `JsQuery::EqualTo/GreaterThan` |
| js_field_node.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 字段节点 | `JsFieldNode` |
| js_schema.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | Schema定义 | `JsSchema` |
| js_observer.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 观察者实现 | `JsObserver` |
| js_util.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | N-API工具 | `JSUtil::GetValue/SetValue` |
| js_const_properties.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 常量导出 | `InitConstProperties` |
| js_error_utils.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 错误处理 | 错误码转换 |
| js_kv_store_resultset.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 结果集 | `JsKVStoreResultSet` |
| napi_queue.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | 异步队列 | `NapiQueue` |
| uv_queue.cpp | `frameworks/jskitsimpl/distributedkvstore/src/` | libuv队列 | `UvQueue` |

**证据**：`frameworks/jskitsimpl/distributedkvstore/src/entry_point.cpp:28-31`
```cpp
static napi_value Init(napi_env env, napi_value exports)
{
    const napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("createKVManager", JsKVManager::CreateKVManager)
    };
```

#### distributeddata 模块

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| entry_point.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 模块注册入口 | `napi_module_register` |
| js_kv_manager.cpp | `frameworks/jskitsimpl/distributeddata/src/` | KV管理器 | `JsKVManager` |
| js_kv_store.cpp | `frameworks/jskitsimpl/distributeddata/src/` | KV存储基类 | `JsKVStore` |
| js_single_kv_store.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 单版本存储 | `JsSingleKVStore` |
| js_device_kv_store.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 设备存储 | `JsDeviceKVStore` |
| js_kv_store_resultset.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 结果集 | `JsKVStoreResultSet` |
| js_query.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 查询 | `JsQuery` |
| js_schema.cpp | `frameworks/jskitsimpl/distributeddata/src/` | Schema | `JsSchema` |
| js_field_node.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 字段节点 | `JsFieldNode` |
| js_observer.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 观察者 | `JsObserver` |
| js_util.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 工具函数 | `JSUtil` |
| js_const_properties.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 常量 | `InitConstProperties` |
| napi_queue.cpp | `frameworks/jskitsimpl/distributeddata/src/` | 异步队列 | `NapiQueue` |
| uv_queue.cpp | `frameworks/jskitsimpl/distributeddata/src/` | libuv队列 | `UvQueue` |

### 内部实现（frameworks/innerkitsimpl/）

#### distributeddatafwk

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| distributed_kv_data_manager.cpp | `frameworks/innerkitsimpl/distributeddatafwk/src/` | 管理器实现 | `DistributedKvDataManager` |
| kvdb_notifier_client.cpp | `frameworks/innerkitsimpl/distributeddatafwk/src/` | 通知客户端 | `KvDBNotifierClient` |
| kvdb_notifier_stub.cpp | `frameworks/innerkitsimpl/distributeddatafwk/src/` | IPC存根 | `KvDBNotifierStub` |

#### kvdb

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| kvdb_service_impl.cpp | `frameworks/innerkitsimpl/kvdb/src/` | KVDB服务实现 | `KvDBServiceImpl` |
| kvdb_service_client.cpp | `frameworks/innerkitsimpl/kvdb/src/` | 服务客户端 | `KvDBServiceClient` |
| kv_types_util.cpp | `frameworks/innerkitsimpl/kvdb/src/` | 类型工具 | 序列化/反序列化 |

### 核心数据库库（frameworks/libs/distributeddb/）

#### 接口层（interfaces/）

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| kv_store_delegate_manager.h | `frameworks/libs/distributeddb/interfaces/include/` | 委托管理器 | `KvStoreDelegateManager` |
| kv_store_nb_delegate.h | `frameworks/libs/distributeddb/interfaces/include/` | 非阻塞委托 | `KvStoreNbDelegate` |
| kv_store_delegate.h | `frameworks/libs/distributeddb/interfaces/include/` | 委托基类 | `KvStoreDelegate` |
| kv_store_observer.h | `frameworks/libs/distributeddb/interfaces/include/` | 观察者 | `KvStoreObserver` |
| kv_store_result_set.h | `frameworks/libs/distributeddb/interfaces/include/` | 结果集 | `KvStoreResultSet` |
| kv_store_errno.h | `frameworks/libs/distributeddb/interfaces/include/` | 错误码 | `DBStatus` |

**证据**：`frameworks/libs/distributeddb/interfaces/include/kv_store_delegate_manager.h:35`
```cpp
class KvStoreDelegateManager {
public:
    DBStatus GetKvStore(...);
    DBStatus CloseKvStore(...);
```

#### 接口实现（interfaces/src/）

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| kv_store_delegate_manager.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 管理器实现 | `KvStoreDelegateManager` |
| kv_store_delegate_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 委托实现 | `KvStoreDelegateImpl` |
| kv_store_nb_delegate_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 非阻塞委托 | `KvStoreNbDelegateImpl` |
| kv_store_result_set_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 结果集实现 | `KvStoreResultSetImpl` |
| kv_store_snapshot_delegate_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 快照委托 | `KvStoreSnapshotDelegateImpl` |
| kv_store_changed_data_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 变更数据 | `KvStoreChangedDataImpl` |
| intercepted_data_impl.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 拦截数据 | `InterceptedDataImpl` |
| kv_store_errno.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 错误码 | 错误转换 |
| runtime_config.cpp | `frameworks/libs/distributeddb/interfaces/src/` | 运行时配置 | `RuntimeConfig` |

#### 存储引擎（storage/）

```
frameworks/libs/distributeddb/storage/
└── src/
    ├── kv/                 # KV存储实现
    ├── sqlite/             # SQLite适配层
    └── runtime/            # 运行时存储
```

#### 同步模块（syncer/）

```
frameworks/libs/distributeddb/syncer/
└── src/
    ├── device/             # 设备间同步
    └── cloud/              # 云端同步（可选）
```

### 公共工具（frameworks/common/）

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| concurrent_map.h | `frameworks/common/` | 并发Map | `ConcurrentMap` |
| concurrent_striped_map.h | `frameworks/common/` | 分片并发Map | `ConcurrentStripedMap` |
| executor.h | `frameworks/common/` | 执行器 | `Executor` |
| executor_pool.h | `frameworks/common/` | 执行器池 | `ExecutorPool` |
| pool.h | `frameworks/common/` | 对象池 | `ObjectPool` |
| priority_queue.h | `frameworks/common/` | 优先队列 | `PriorityQueue` |
| task_scheduler.h | `frameworks/common/` | 任务调度器 | `TaskScheduler` |
| block_data.h | `frameworks/common/` | 阻塞数据 | `BlockData` |
| itypes_util.h | `frameworks/common/` | IPC类型工具 | `ITypesUtil` |
| js_proxy.h | `frameworks/common/` | JS代理 | `JSProxy` |
| lru_bucket.h | `frameworks/common/` | LRU缓存 | `LRUBucket` |
| log_print.h | `frameworks/common/` | 日志打印 | `ZLOGI/ZLOGE` |
| dds_trace.h | `frameworks/common/` | 追踪 | `DDS_TRACE` |

### Native 实现（frameworks/native/）

| 模块 | 路径 | 职责 | 关键文件 |
|-----|------|------|---------|
| kv_store | `frameworks/native/kv_store/` | C API实现 | `kv_store.c` |
| dbm_kv_store | `frameworks/native/dbm_kv_store/` | DBM风格实现 | `dbm_kv_store.c` |

### ETS/ArkTS 实现（frameworks/ets/）

| 模块 | 路径 | 职责 | 关键文件 |
|-----|------|------|---------|
| taihe/kv_store | `frameworks/ets/taihe/kv_store/` | ArkTS绑定 | `ani_*.cpp` |

### CJ FFI（frameworks/cj/）

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| distributed_kv_store_ffi.cpp | `frameworks/cj/src/` | FFI接口 | `CJ_*` 函数 |
| distributed_kv_store_impl.cpp | `frameworks/cj/src/` | 实现 | 内部实现 |

---

## 数据库工具（databaseutils/）

| 文件 | 路径 | 职责 | 关键类/函数 |
|-----|------|------|-----------|
| acl.h | `databaseutils/include/` | ACL定义 | `Acl` 结构 |
| acl.cpp | `databaseutils/src/` | ACL实现 | ACL操作 |

---

## Mock组件（kvstoremock/）

| 模块 | 路径 | 用途 | 说明 |
|-----|------|------|------|
| interfaces/mock | `kvstoremock/interfaces/mock/` | Mock接口 | 测试用 |
| distributeddb | `kvstoremock/distributeddb/` | DB Mock | 测试用 |

---

## 关键调用链导航

### 1. 创建 KVManager 调用链

```
JS: createKVManager(config)
  └── frameworks/jskitsimpl/distributedkvstore/src/entry_point.cpp:29
      └── JsKVManager::CreateKVManager
          └── frameworks/jskitsimpl/distributedkvstore/src/js_kv_manager.cpp
              └── DistributedKvDataManager
                  └── frameworks/innerkitsimpl/distributeddatafwk/
```

### 2. Put 操作调用链

```
JS: kvStore.put(key, value)
  └── frameworks/jskitsimpl/distributedkvstore/src/js_single_kv_store.cpp
      └── JsSingleKVStore::Put
          └── SingleKvStore::Put
              └── interfaces/innerkits/distributeddata/include/single_kvstore.h
                  └── KvStoreNbDelegate::Put
                      └── frameworks/libs/distributeddb/interfaces/include/kv_store_nb_delegate.h
```

### 3. 数据同步调用链

```
JS: kvStore.sync(deviceIds, mode, query)
  └── frameworks/jskitsimpl/distributedkvstore/src/js_single_kv_store.cpp
      └── JsSingleKVStore::Sync
          └── SingleKvStore::Sync
              └── frameworks/libs/distributeddb/syncer/
```

### 4. 查询构建调用链

```
JS: new Query().equalTo('field', value)
  └── frameworks/jskitsimpl/distributedkvstore/src/js_query.cpp
      └── JsQuery::EqualTo
          └── DataQuery
              └── interfaces/innerkits/distributeddata/include/data_query.h
```

---

## 按功能查找代码

### 想要实现/查找... | 查看这些文件

| 功能需求 | 推荐文件路径 |
|---------|-------------|
| **添加新的 JS API** | `interfaces/jskits/*/distributed_*.js` 声明<br>`frameworks/jskitsimpl/*/src/entry_point.cpp` 注册<br>`frameworks/jskitsimpl/*/src/js_*.cpp` 实现 |
| **修改存储逻辑** | `frameworks/libs/distributeddb/storage/src/` |
| **修改同步逻辑** | `frameworks/libs/distributeddb/syncer/src/` |
| **添加新的 Inner API** | `interfaces/innerkits/*/include/*.h` 声明<br>`frameworks/innerkitsimpl/*/src/*.cpp` 实现 |
| **修改错误码** | `interfaces/innerkits/distributeddata/include/store_errno.h`<br>`frameworks/libs/distributeddb/interfaces/include/kv_store_errno.h` |
| **修改线程模型** | `frameworks/common/executor_pool.h`<br>`frameworks/jskitsimpl/*/src/napi_queue.cpp`<br>`frameworks/jskitsimpl/*/src/uv_queue.cpp` |
| **修改常量定义** | `interfaces/jskits/*/distributed_*.js`<br>`frameworks/jskitsimpl/*/src/js_const_properties.cpp` |

---

## 术语对照表

| 术语 | 代码中的命名 | 所在文件 |
|-----|-------------|---------|
| KV管理器 | `DistributedKvDataManager` | `distributed_kv_data_manager.h/cpp` |
| 单版本存储 | `SingleKvStore` | `single_kvstore.h`, `js_single_kv_store.cpp` |
| 设备协同存储 | `DeviceKvStore` | `js_device_kv_store.cpp` |
| 查询构建器 | `DataQuery` / `JsQuery` | `data_query.h`, `js_query.cpp` |
| 结果集 | `KvStoreResultSet` | `kvstore_result_set.h` |
| 观察者 | `KvStoreObserver` | `kvstore_observer.h`, `js_observer.cpp` |
| 委托管理器 | `KvStoreDelegateManager` | `kv_store_delegate_manager.h/cpp` |
| 非阻塞委托 | `KvStoreNbDelegate` | `kv_store_nb_delegate.h` |

---

## 相关文档

- [项目概述](01_Overview.md) - 项目定位与核心概念
- [架构设计](02_Architecture.md) - 模块架构与数据流
- [N-API 接口参考](04_NAPI_Reference.md) - JS API 详细文档
- [Inner API 参考](05_Inner_API.md) - C++ 接口文档
