# 对外接口文档

> N-API、IPC 及配置接口完整说明

## 目的

本文档详细列出分布式数据对象组件的所有对外接口，包括 JS API、IPC 接口和配置项，供开发者和安全研究员参考。

## 适用范围

- OpenHarmony 标准系统
- 组件版本 3.1.0
- 对外暴露面：JS API、IPC

---

## N-API 接口清单

### 模块注册

**模块名**: `data.distributedDataObject`

**N-API 导出函数**:

| JS API | C++ 实现 | 文件位置 | 功能描述 |
|---------|-----------|----------|----------|
| `createObjectSync()` | `JSDistributedObjectStore::JSCreateObjectSync` | `js_module_init.cpp:32` | 同步创建分布式对象 |
| `destroyObjectSync()` | `JSDistributedObjectStore::JSDestroyObjectSync` | `js_module_init.cpp:33` | 销毁分布式对象 |
| `on()` | `JSDistributedObjectStore::JSOn` | `js_module_init.cpp:34` | 注册事件监听 |
| `off()` | `JSDistributedObjectStore::JSOff` | `js_module_init.cpp:35` | 取消事件监听 |
| `recordCallback()` | `JSDistributedObjectStore::JSRecordCallback` | `js_module_init.cpp:36` | 记录回调 |
| `deleteCallback()` | `JSDistributedObjectStore::JSDeleteCallback` | `js_module_init.cpp:37` | 删除回调 |
| `sequenceNum()` | `JSDistributedObjectStore::JSEquenceNum` | `js_module_init.cpp:38` | 获取序列号 |

**证据**: `frameworks/jskitsimpl/src/adaptor/js_module_init.cpp:31-39`

### JS 导出接口

**导出对象** (`distributed_data_object.js:573-577`):

| 导出名 | 功能描述 |
|--------|----------|
| `createDistributedObject` | 创建分布式对象实例 |
| `create` | 创建增强版分布式对象（v9） |
| `genSessionId` | 生成随机会话 ID |

**证据**: `interfaces/jskits/distributed_data_object.js:573-577`

### JS 对象方法

**DistributedV9 类方法**:

| 方法 | 同步/异步 | 参数 | 返回值 | 描述 |
|------|-----------|------|--------|------|
| `setSessionId(sessionId, callback?)` | 异步 | sessionId: string, callback: function | boolean | 设置/切换会话 |
| `on(type, callback)` | 同步 | type: 'change'\|'status', callback: function | void | 注册事件监听 |
| `off(type, callback?)` | 同步 | type: string, callback?: function | void | 取消事件监听 |
| `save(deviceId, callback)` | 异步 | deviceId: string, callback: function | void | 持久化到本地 |
| `revokeSave(callback)` | 异步 | callback: function | void | 撤回持久化 |
| `bindAssetStore(assetKey, bindInfo, callback)` | 异步 | assetKey: string, bindInfo: object, callback: function | void | 绑定资产存储 |

**证据**: `interfaces/jskits/distributed_data_object.js`

---

## IPC 接口

### 服务名称

**接口标识**: `OHOS.DistributedObject.IObjectService`

**证据**: `frameworks/innerkitsimpl/include/iobject_service.h:25`

### 接口码

| 码值 | 接口名称 | 功能描述 | 参数 |
|------|----------|----------|------|
| 0 | OBJECTSTORE_SAVE | 保存对象到远程设备 | bundleName, sessionId, deviceId |
| 1 | OBJECTSTORE_REVOKE_SAVE | 撤回保存 | bundleName, sessionId |
| 2 | OBJECTSTORE_RETRIEVE | 检索对象 | bundleName, sessionId, deviceId |
| 3 | OBJECTSTORE_REGISTER_OBSERVER | 注册数据变更观察者 | bundleName, sessionId, deviceList |
| 4 | OBJECTSTORE_UNREGISTER_OBSERVER | 取消注册观察者 | bundleName, sessionId |
| 5 | OBJECTSTORE_ON_ASSET_CHANGED | 资产变更通知 | assetKey |
| 6 | OBJECTSTORE_BIND_ASSET_STORE | 绑定资产存储 | bindInfo |
| 7 | OBJECTSTORE_DELETE_SNAPSHOT | 删除快照 | snapshotId |
| 9 | OBJECTSTORE_REGISTER_PROGRESS | 注册进度观察者 | sessionId |
| 10 | OBJECTSTORE_UNREGISTER_PROGRESS | 取消注册进度 | sessionId |

**证据**: `frameworks/innerkitsimpl/include/distributeddata_object_store_ipc_interface_code.h`

---

## 错误码

| 错误码 | 名称 | 含义 | 触发场景 |
|--------|------|------|----------|
| 0 | SUCCESS | 成功 | - |
| 1651 | ERR_DB_SET_PROCESS | 数据库设置失败 | 存储引擎初始化 |
| 1653 | ERR_DATA_LEN | 非法数据长度 | 对象大小超过 500KB |
| 1673 | ERR_NO_PERMISSION | 无权限 | 缺少 DISTRIBUTED_DATASYNC 权限 |

**完整错误码列表**: 参考 `interfaces/innerkits/objectstore_errors.h`

**证据**: `interfaces/innerkits/objectstore_errors.h`

---

## 证据

- N-API 注册: `js_module_init.cpp:28-44`
- JS API 导出: `distributed_data_object.js:573-577`
- IPC 接口: `distributeddata_object_store_ipc_interface_code.h`
- 错误码: `objectstore_errors.h`

## 相关链接

- [项目概览](./00_Overview.md)
- [架构详解](./01_Architecture.md)
- [攻击面分析](./04_AttackSurface.md)
