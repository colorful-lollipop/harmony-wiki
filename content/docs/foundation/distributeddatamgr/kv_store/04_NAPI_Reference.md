# N-API 接口参考

## 概述

KV Store 提供两套 N-API 接口：
1. **distributeddata**：分布式数据管理基础接口
2. **distributedkvstore**：分布式 KV 存储专用接口

## 模块初始化

### distributeddata

**注册入口**：`frameworks/jskitsimpl/distributeddata/src/entry_point.cpp`

```cpp
// napi_module_register(&module);
```

**导出内容**：
- `createKVManager`：创建 KV 管理器
- `Query`：查询构建器类
- `FieldNode`：字段节点类
- `Schema`：Schema 定义类
- 常量导出（ValueType, SyncMode, SubscribeType, KVStoreType, SecurityLevel）

### distributedkvstore

**注册入口**：`frameworks/jskitsimpl/distributedkvstore/src/entry_point.cpp`

**导出内容**：
- `SingleKVStore`：单版本 KV 存储类
- `DeviceKVStore`：设备协同 KV 存储类
- `Query`：查询构建器
- `FieldNode`：字段节点
- `Schema`：Schema 定义

## API 清单

### KVManager

**创建方法**：`createKVManager(config)`

| 属性 | 类型 | 必填 | 描述 |
|-----|------|-----|------|
| config.bundleName | string | 是 | 应用 Bundle 名称 |
| config.userInfo | UserInfo | 是 | 用户信息 |

**JS 声明**：`interfaces/jskits/distributedkvstore/distributed_kvstore.js:18`

**实现文件**：`frameworks/jskitsimpl/distributeddata/src/js_kv_manager.cpp`

### SingleKVStore

| JS 方法 | C++ 实现 | 同步/异步 | 描述 |
|--------|---------|---------|------|
| `put(key, value)` | `JsSingleKVStore::Put` | 异步 | 插入键值对 |
| `get(key)` | `JsSingleKVStore::Get` | 异步 | 获取值 |
| `delete(key)` | `JsSingleKVStore::Delete` | 异步 | 删除键 |
| `putBatch(entries)` | `JsSingleKVStore::PutBatch` | 异步 | 批量插入 |
| `getEntries(query)` | `JsSingleKVStore::GetEntries` | 异步 | 获取匹配条目 |
| `getResultSet(query)` | `JsSingleKVStore::GetResultSet` | 异步 | 获取结果集 |
| `closeResultSet(resultSet)` | `JsSingleKVStore::CloseResultSet` | 异步 | 关闭结果集 |
| `startTransaction()` | `JsSingleKVStore::StartTransaction` | 异步 | 开始事务 |
| `commit()` | `JsSingleKVStore::Commit` | 异步 | 提交事务 |
| `rollback()` | `JsSingleKVStore::Rollback` | 异步 | 回滚事务 |
| `backup(file, secret)` | `JsSingleKVStore::Backup` | 异步 | 备份数据库 |
| `restore(file, secret)` | `JsSingleKVStore::Restore` | 异步 | 恢复数据库 |
| `deleteBackup(file)` | `JsSingleKVStore::DeleteBackup` | 异步 | 删除备份 |
| `on('dataChange', listener)` | `JsSingleKVStore::OnDataChange` | 事件 | 订阅数据变更 |
| `off('dataChange', listener)` | `JsSingleKVStore::OffDataChange` | 事件 | 取消订阅 |
| `sync(deviceIds, mode, query)` | `JsSingleKVStore::Sync` | 异步 | 同步数据 |

### DeviceKVStore

| JS 方法 | C++ 实现 | 同步/异步 | 描述 |
|--------|---------|---------|------|
| `get(key)` | `JsDeviceKVStore::Get` | 异步 | 获取设备数据 |
| `getEntries(query)` | `JsDeviceKVStore::GetEntries` | 异步 | 获取匹配条目 |
| `getResultSet(query)` | `JsDeviceKVStore::GetResultSet` | 异步 | 获取结果集 |
| `closeResultSet(resultSet)` | `JsDeviceKVStore::CloseResultSet` | 异步 | 关闭结果集 |
| `removeDeviceData(deviceId)` | `JsDeviceKVStore::RemoveDeviceData` | 异步 | 移除设备数据 |
| `on('syncComplete', listener)` | `JsDeviceKVStore::OnSyncComplete` | 事件 | 同步完成事件 |

### Query

| JS 方法 | C++ 实现 | 描述 |
|--------|---------|------|
| `equalTo(field, value)` | `JsQuery::EqualTo` | 等值匹配 |
| `notEqualTo(field, value)` | `JsQuery::NotEqualTo` | 不等匹配 |
| `greaterThan(field, value)` | `JsQuery::GreaterThan` | 大于 |
| `lessThan(field, value)` | `JsQuery::LessThan` | 小于 |
| `greaterThanOrEqualTo(field, value)` | `JsQuery::GreaterThanOrEqualTo` | 大于等于 |
| `lessThanOrEqualTo(field, value)` | `JsQuery::LessThanOrEqualTo` | 小于等于 |
| `isNull(field)` | `JsQuery::IsNull` | 为空 |
| `isNotNull(field)` | `JsQuery::IsNotNull` | 非空 |
| `inNumber(field, valueList)` | `JsQuery::InNumber` | 在数值列表中 |
| `notInNumber(field, valueList)` | `JsQuery::NotInNumber` | 不在数值列表中 |
| `inString(field, valueList)` | `JsQuery::InString` | 在字符串列表中 |
| `notInString(field, valueList)` | `JsQuery::NotInString` | 不在字符串列表中 |
| `like(field, value)` | `JsQuery::Like` | LIKE 匹配 |
| `unlike(field, value)` | `JsQuery::Unlike` | NOT LIKE 匹配 |
| `and()` | `JsQuery::And` | AND 条件 |
| `or()` | `JsQuery::Or` | OR 条件 |
| `orderByAsc(field)` | `JsSingleKVStore::OrderByAsc` | 升序排序 |
| `orderByDesc(field)` | `JsSingleKVStore::OrderByDesc` | 降序排序 |
| `limit(total, offset)` | `JsSingleKVStore::Limit` | 结果限制 |
| `beginGroup()` | `JsQuery::BeginGroup` | 条件分组开始 |
| `endGroup()` | `JsQuery::EndGroup` | 条件分组结束 |
| `reset()` | `JsQuery::Reset` | 重置查询 |
| `prefixKey(key)` | `JsQuery::PrefixKey` | 键前缀 |
| `deviceId(deviceId)` | `JsQuery::DeviceId` | 设备 ID |

## 常量定义

**文件**：`interfaces/jskits/distributedkvstore/distributed_kvstore.js:21-63`

### ValueType

| 常量 | 值 | 描述 |
|-----|---|------|
| ValueType.STRING | 0 | 字符串 |
| ValueType.INTEGER | 1 | 整数 |
| ValueType.FLOAT | 2 | 浮点数 |
| ValueType.BYTE_ARRAY | 3 | 字节数组 |
| ValueType.BOOLEAN | 4 | 布尔值 |
| ValueType.DOUBLE | 5 | 双精度浮点 |

### SyncMode

| 常量 | 值 | 描述 |
|-----|---|------|
| SyncMode.PULL_ONLY | 0 | 仅拉取 |
| SyncMode.PUSH_ONLY | 1 | 仅推送 |
| SyncMode.PUSH_PULL | 2 | 推送+拉取 |

### SubscribeType

| 常量 | 值 | 描述 |
|-----|---|------|
| SubscribeType.SUBSCRIBE_TYPE_LOCAL | 0 | 本地订阅 |
| SubscribeType.SUBSCRIBE_TYPE_REMOTE | 1 | 远程订阅 |
| SubscribeType.SUBSCRIBE_TYPE_ALL | 2 | 全部订阅 |

### KVStoreType

| 常量 | 值 | 描述 |
|-----|---|------|
| KVStoreType.DEVICE_COLLABORATION | 0 | 设备协同模式 |
| KVStoreType.SINGLE_VERSION | 1 | 单版本模式 |
| KVStoreType.MULTI_VERSION | 2 | 多版本模式 |

### SecurityLevel

| 常量 | 值 | 描述 |
|-----|---|------|
| SecurityLevel.NO_LEVEL | 0 | 无安全级别 |
| SecurityLevel.S0 | 1 | S0 |
| SecurityLevel.S1 | 2 | S1 |
| SecurityLevel.S2 | 3 | S2 |
| SecurityLevel.S3 | 5 | S3 |
| SecurityLevel.S4 | 6 | S4 |

### Limits

| 常量 | 值 | 描述 |
|-----|---|------|
| Constants.MAX_KEY_LENGTH | 1024 | 最大键长度 |
| Constants.MAX_VALUE_LENGTH | 4194303 | 最大值长度 |
| Constants.MAX_KEY_LENGTH_DEVICE | 896 | 设备键最大长度 |
| Constants.MAX_STORE_ID_LENGTH | 128 | Store ID 最大长度 |
| Constants.MAX_QUERY_LENGTH | 512000 | 查询最大长度 |
| Constants.MAX_BATCH_SIZE | 128 | 批量操作最大数量 |

## 参数校验

### JSUtil 工具类

**文件**：`frameworks/jskitsimpl/distributedkvstore/src/js_util.cpp`

**职责**：N-API 类型转换和参数校验

| 方法 | 参数类型 | 校验逻辑 |
|-----|---------|---------|
| `GetValue(env, napi_value, std::string&)` | string | 检查 `napi_string` 类型 |
| `GetValue(env, napi_value, bool&)` | boolean | 检查 `napi_boolean` 类型 |
| `GetValue(env, napi_value, int32_t&)` | int32 | 检查 `napi_number` 类型 |
| `GetValue(env, napi_value, std::vector<uint8_t>&)` | Uint8Array | 检查 `napi_uint8_array` 类型 |
| `GetValue(env, napi_value, Entry&)` | object | 检查对象属性完整性 |

### 错误码

参考：`interfaces/innerkits/distributeddata/include/store_errno.h`

| 错误码 | 描述 |
|-------|------|
| STATUS_OK | 成功 |
| INVALID_ARGUMENT | 无效参数 |
| NOT_FOUND | 未找到 |
| ALREADY_EXISTS | 已存在 |
| ERROR | 通用错误 |

## 使用示例

### 创建 KVManager

```javascript
import distributedKVStore from '@ohos.distributed.kvStore';

const kvManager = await distributedKVStore.createKVManager({
    bundleName: 'com.example.myapp',
    userInfo: {
        userId: 0,
        userType: distributedKVStore.UserType.SAME_USER_ID
    }
});
```

### 打开 KV Store

```javascript
const options = {
    kvStoreType: distributedKVStore.KVStoreType.SINGLE_VERSION,
    securityLevel: distributedKVStore.SecurityLevel.S2
};

const kvStore = await kvManager.getKVStore(options, 'mystore');
```

### 读写操作

```javascript
// 写入
await kvStore.put('key1', 'value1');
await kvStore.put('key2', new Uint8Array([1, 2, 3]));

// 读取
const value = await kvStore.get('key1');

// 批量写入
await kvStore.putBatch([
    { key: 'key1', value: 'value1' },
    { key: 'key2', value: 'value2' }
]);

// 条件查询
const entries = await kvStore.getEntries(
    new distributedKVStore.Query().prefixKey('prefix')
);
```

### 订阅数据变更

```javascript
kvStore.on('dataChange', distributedKVStore.SubscribeType.SUBSCRIBE_TYPE_LOCAL, (data) => {
    console.log('Data changed:', data);
});
```

## 相关文档

- [项目概述](01_Overview.md)
- [代码地图](03_CodeMap.md) - 代码文件导航
- [架构设计](02_Architecture.md)
- [Inner API 参考](05_Inner_API.md)
