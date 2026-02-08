# 故障排查

## 概述

本文档收集 KV Store 使用过程中的常见问题和调试方法。

## 编译问题

### 问题 1：找不到头文件

**错误信息**：
```
fatal error: 'distributed_kv_data_manager.h' file not found
```

**原因**：未正确配置 include 路径

**解决方案**：
1. 确认 `bundle.json` 中已声明依赖
2. 检查 `BUILD.gn` 中的 `include_dirs`

**证据**：`interfaces/jskits/distributedkvstore/BUILD.gn`

```gn
external_deps = [
  "//foundation/distributeddatamgr/kv_store/frameworks/libs/distributeddb:distributeddb",
]
```

### 问题 2：链接错误 - 找不到符号

**错误信息**：
```
undefined reference to 'DistributedDB::KvStoreDelegateManager::GetKvStore'
```

**原因**：未链接 `distributeddb` 库

**解决方案**：
1. 确认 `deps` 中包含 `distributeddb`
2. 检查 `public_external_deps` 配置

**证据**：`frameworks/libs/distributeddb/BUILD.gn:132-135`

```gn
public_external_deps = [
  "openssl:libcrypto_shared",
  "sqlite:sqlite",
]
```

### 问题 3：SQLite 编译错误

**错误信息**：
```
'Sqlite' file not found
```

**原因**：未声明 SQLite 依赖

**解决方案**：添加 SQLite 到 `external_deps`

```gn
external_deps = [
  "sqlite:sqlite",
]
```

## 运行时问题

### 问题 1：KVStore 创建失败

**错误信息**：
```
E0001: Invalid argument
```

**常见原因**：

| 错误码 | 原因 | 解决方案 |
|-------|------|---------|
| INVALID_ARGUMENT | 参数无效 | 检查 bundleName、storeId 格式 |
| NOT_FOUND | storeId 为空 | 提供有效的 storeId |
| ALREADY_EXISTS | store 已存在 | 使用不同的 storeId 或先关闭 |

**排查步骤**：

```javascript
// 1. 检查配置参数
const options = {
    kvStoreType: distributedKVStore.KVStoreType.SINGLE_VERSION,
    securityLevel: distributedKVStore.SecurityLevel.S2
};

// 2. 检查 storeId
const storeId = 'valid_store_id'; // 长度 <= 128

// 3. 添加错误处理
try {
    const kvStore = await kvManager.getKVStore(options, storeId);
} catch (error) {
    console.error('Error:', error.code, error.message);
}
```

### 问题 2：数据同步失败

**错误信息**：
```
Sync failed: device not found
```

**排查步骤**：

1. 检查设备发现状态

```javascript
// 检查设备是否在线
const devices = await deviceManager.getTrustedDeviceList();
console.log('Available devices:', devices);
```

2. 检查同步权限

```javascript
// 确认 SyncMode 配置
const syncMode = distributedKVStore.SyncMode.PUSH_PULL;
await kvStore.sync(deviceIds, syncMode, query);
```

3. 检查网络连接

```bash
# 查看网络状态
hilog | grep -E "DistributedData|KVStore"
```

### 问题 3：数据库损坏

**错误信息**：
```
Database corrupted
```

**排查步骤**：

1. 检查 SQLite 完整性

```cpp
// 使用 SQLite 完整性检查
PRAGMA integrity_check;
```

2. 从备份恢复

```javascript
// 备份
await kvStore.backup('backup.db', secret);

// 恢复
await kvStore.restore('backup.db', secret);
```

3. 删除并重建

```cpp
// 删除数据库
kvManager.deleteKvStore(storeId);
```

### 问题 4：性能问题

**症状**：操作响应慢

**排查方向**：

| 方向 | 检查项 |
|-----|--------|
| 批量操作 | 是否使用 PutBatch 替代多次 Put |
| 查询优化 | 是否使用合适的索引 |
| 事务使用 | 是否合理使用事务 |
| 数据量 | 单个数据库是否过大 |

**优化建议**：

```javascript
// ❌ 错误：逐条插入
for (const item of items) {
    await kvStore.put(item.key, item.value);
}

// ✅ 正确：批量插入
const entries = items.map(item => ({
    key: item.key,
    value: item.value
}));
await kvStore.putBatch(entries);
```

## 调试方法

### 1. 日志查看

**命令**：
```bash
# 查看 KV Store 相关日志
hilog | grep -E "KvStore|JS_KVManager|SingleKVStore"

# 查看详细日志
hilog -D 0x0020 | grep -E "DistributedData"
```

**日志级别**：
- D - Debug
- I - Info
- W - Warning
- E - Error
- F - Fatal

### 2. 追踪工具

**使用 HiTrace**：

```cpp
#include "hitrace/hitrace_meter.h"

// 开始追踪
uint64_t traceId = HiTraceBegin("KVOperation", HiTraceFlag::INPUT_PARAM);

// 执行操作
kvStore->Put(key, value);

// 结束追踪
HiTraceEnd(traceId);
```

### 3. 内存检查

**使用 AddressSanitizer**：

```gn
# 在 BUILD.gn 中启用
sanitize = {
  asan = true
}
```

### 4. 线程分析

**使用线程安全检测**：

```cpp
// 检查是否有线程冲突
// 编译时添加 -fsanitize=thread
```

## 常见错误码

### N-API 层错误

| 错误码 | 描述 | 常见原因 |
|-------|------|---------|
| 1 | INVALID_ARGUMENT | 参数类型错误 |
| 2 | NOT_FOUND | 对象不存在 |
| 3 | ALREADY_EXISTS | 对象已存在 |
| 4 | ERROR | 通用错误 |

### KV Store 错误

| 错误码 | 描述 | 解决方案 |
|-------|------|---------|
| 401 | Parameter error | 检查参数类型 |
| 1001 | Database not found | 检查 storeId |
| 1002 | Database already exists | 使用其他 storeId |
| 1003 | Database corrupted | 恢复备份 |
| 1004 | Permission denied | 检查权限配置 |

## 调试技巧

### 1. 启用详细日志

```javascript
// 在应用启动时设置
const logger = console;
logger.level = 'debug';
```

### 2. 使用 Try-Catch 包装操作

```javascript
async function safePut(kvStore, key, value) {
    try {
        await kvStore.put(key, value);
        console.log('Put success');
    } catch (error) {
        console.error('Put failed:', error.code, error.message);
        // 记录错误上下文
        console.error('Key:', key);
        console.error('Value type:', typeof value);
    }
}
```

### 3. 监控数据库状态

```javascript
// 获取数据库大小
const diskSize = await kvStore.getDiskSize();

// 获取记录数量
const resultSet = await kvStore.getResultSet(query);
const count = await resultSet.getCount();
```

## 相关文档

- [代码地图](03_CodeMap.md) - 代码文件导航
- [N-API 接口参考](04_NAPI_Reference.md)
- [安全风险评审](08_Security_Review.md)
- [GN 构建指南](06_Build_GN.md)
