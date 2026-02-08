# Inner API 参考

## 概述

Inner API 是面向系统级组件的 C++ 接口，主要供系统服务和其他子系统使用。

## 接口清单

### 分布式数据管理接口

| 接口文件 | 路径 | 职责 |
|---------|------|------|
| `distributed_kv_data_manager.h` | `interfaces/innerkits/distributeddata/include/` | 分布式 KV 数据管理器 |
| `kvstore.h` | `interfaces/innerkits/distributeddata/include/` | KV 存储抽象基类 |
| `single_kvstore.h` | `interfaces/innerkits/distributeddata/include/` | 单版本 KV 存储接口 |
| `data_query.h` | `interfaces/innerkits/distributeddata/include/` | 数据查询接口 |
| `types.h` | `interfaces/innerkits/distributeddata/include/` | 类型定义 |

### 观察者与回调接口

| 接口文件 | 路径 | 职责 |
|---------|------|------|
| `kvstore_observer.h` | `interfaces/innerkits/distributeddata/include/` | KV 存储观察者 |
| `kvstore_sync_callback.h` | `interfaces/innerkits/distributeddata/include/` | 同步回调 |
| `kvstore_result_set.h` | `interfaces/innerkits/distributeddata/include/` | 结果集 |
| `change_notification.h` | `interfaces/innerkits/distributeddata/include/` | 变更通知 |
| `kvstore_death_recipient.h` | `interfaces/innerkits/distributeddata/include/` | 死亡接收者 |

### 工具类型接口

| 接口文件 | 路径 | 职责 |
|---------|------|------|
| `blob.h` | `interfaces/innerkits/distributeddata/include/` | 二进制大对象 |
| `end_point.h` | `interfaces/innerkits/distributeddata/include/` | 端点定义 |
| `executor.h` | `interfaces/innerkits/distributeddata/include/` | 执行器 |
| `executor_pool.h` | `interfaces/innerkits/distributeddata/include/` | 执行器池 |
| `pool.h` | `interfaces/innerkits/distributeddata/include/` | 线程池 |
| `priority_queue.h` | `interfaces/innerkits/distributeddata/include/` | 优先级队列 |
| `visibility.h` | `interfaces/innerkits/distributeddata/include/` | 可见性定义 |

## 核心接口详解

### KvStore 抽象基类

**文件**：`interfaces/innerkits/distributeddata/include/kvstore.h`

**证据**：`KvStore` 类定义（第 25 行）

```cpp
class API_EXPORT KvStore {
public:
    virtual StoreId GetStoreId() const = 0;
    virtual Status Put(const Key &key, const Value &value) = 0;
    virtual Status PutBatch(const std::vector<Entry> &entries) = 0;
    virtual Status Delete(const Key &key) = 0;
    virtual Status DeleteBatch(const std::vector<Key> &keys) = 0;
    // ... 更多方法
};
```

### SingleKVStore 接口

**文件**：`interfaces/innerkits/distributeddata/include/single_kvstore.h`

**关键方法**：

| 方法 | 参数 | 返回值 | 描述 |
|-----|------|-------|------|
| `GetStoreId()` | - | StoreId | 获取存储 ID |
| `Put()` | const Key&, const Value& | Status | 插入键值对 |
| `Get()` | const Key&, Value& | Status | 获取值 |
| `Delete()` | const Key& | Status | 删除键 |
| `PutBatch()` | const std::vector<Entry>& | Status | 批量插入 |
| `GetEntries()` | const Query&, std::vector<Entry>& | Status | 获取匹配条目 |
| `GetResultSet()` | const Query&, KvStoreResultSet*& | Status | 获取结果集 |
| `CloseResultSet()` | KvStoreResultSet* | Status | 关闭结果集 |
| `GetResultSize()` | const Query&, int& | Status | 获取结果大小 |
| `RemoveDeviceData()` | const std::string& | Status | 移除设备数据 |
| `Sync()` | const SyncParam& | Status | 同步数据 |

### DistributedKvDataManager

**文件**：`interfaces/innerkits/distributeddata/include/distributed_kv_data_manager.h`

**职责**：分布式 KV 数据管理器，负责创建和管理 KV 存储实例

```cpp
class API_EXPORT DistributedKvDataManager {
public:
    // 获取单例实例
    static DistributedKvDataManager &GetInstance();

    // 创建/获取 KV 存储
    Status GetKvStore(const Options &options, const StoreId &storeId,
                       std::function<void(Status, std::unique_ptr<SingleKVStore>)> callback);

    // 关闭 KV 存储
    Status CloseKvStore(SingleKVStore *kvStore);

    // 注册观察者
    Status RegisterObserver(const ObserverKey &key, KvStoreObserver *observer);

    // 取消注册观察者
    Status UnRegisterObserver(const ObserverKey &key);
};
```

### DataQuery 接口

**文件**：`interfaces/innerkits/distributeddata/include/data_query.h`

**关键方法**：

| 方法 | 描述 |
|-----|------|
| `EqualTo()` | 等值条件 |
| `NotEqualTo()` | 不等条件 |
| `GreaterThan()` | 大于 |
| `LessThan()` | 小于 |
| `GreaterThanOrEqualTo()` | 大于等于 |
| `LessThanOrEqualTo()` | 小于等于 |
| `In()` | 在集合中 |
| `NotIn()` | 不在集合中 |
| `Like()` | LIKE 匹配 |
| `OrderByAsc()` | 升序排序 |
| `OrderByDesc()` | 降序排序 |
| `Limit()` | 结果限制 |
| `PrefixKey()` | 键前缀 |
| `DeviceId()` | 设备 ID |

## 类型定义

### Options

```cpp
struct Options {
    KvStoreType kvStoreType;
    SecurityLevel securityLevel;
    bool encrypt;
    Schema schema;
    bool isAutoSync;
    // ...
};
```

### Key / Value

```cpp
using Key = Blob;      // 二进制键
using Value = Blob;    // 二进制值
```

### StoreId

```cpp
struct StoreId {
    std::string storeId;
};
```

### Status

```cpp
enum class Status {
    SUCCESS = 0,
    INVALID_ARGUMENT = 1,
    NOT_FOUND = 2,
    ALREADY_EXISTS = 3,
    KEY_NOT_FOUND = 4,
    // ...
};
```

## 依赖方向

```
innerkits (接口声明)
    │
    ├── frameworks/innerkitsimpl/ (实现)
    │   ├── distributeddatafwk/
    │   └── kvdb/
    │
    └── frameworks/libs/distributeddb/ (核心库)
```

## 使用示例

```cpp
#include "distributed_kv_data_manager.h"
#include "single_kvstore.h"

using namespace OHOS::DistributedKv;

void Example() {
    auto &mgr = DistributedKvDataManager::GetInstance();

    Options options;
    options.kvStoreType = KvStoreType::SINGLE_VERSION;
    options.securityLevel = SecurityLevel::S2;

    StoreId storeId{"mystore"};

    mgr.GetKvStore(options, storeId,
        [&](Status status, std::unique_ptr<SingleKVStore> kvStore) {
            if (status != Status::SUCCESS) {
                return;
            }

            Key key{"testKey"};
            Value value{"testValue"};
            kvStore->Put(key, value);

            Value outValue;
            kvStore->Get(key, outValue);
        });
}
```

## 相关文档

- [代码地图](03_CodeMap.md) - 代码文件导航
- [N-API 接口参考](04_NAPI_Reference.md)
- [架构设计](02_Architecture.md)
- [GN 构建指南](06_Build_GN.md)
