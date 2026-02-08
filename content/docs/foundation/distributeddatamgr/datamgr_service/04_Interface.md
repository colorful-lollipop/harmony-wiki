# IPC 接口清单

**Distributed Data Manager Service (SA ID: 1301)**

---

## 目录

- [服务入口](#服务入口)
- [Feature 接口概览](#feature-接口概览)
- [RDB Feature (关系型数据库)](#rdb-feature-关系型数据库)
- [KVDB Feature (KV数据库)](#kvdb-feature-kv数据库)
- [Cloud Feature (云同步)](#cloud-feature-云同步)
- [Object Feature (分布式对象)](#object-feature-分布式对象)
- [DataShare Feature (数据共享)](#datashare-feature-数据共享)
- [UDMF Feature (统一数据管理)](#udmf-feature-统一数据管理)
- [公共错误码](#公共错误码)

---

## 服务入口

### SystemAbility 注册

```cpp
// services/distributeddataservice/app/src/kvstore_data_service.cpp:89
REGISTER_SYSTEM_ABILITY_BY_ID(KvStoreDataService, DISTRIBUTED_KV_DATA_SERVICE_ABILITY_ID, true);
```

### 核心 IPC 接口 (App 层)

**接口文件**: `services/distributeddataservice/app/src/kvstore_data_service_stub.h`

| Code | 方法名 | 说明 |
|------|-------|------|
| 0 | `GetFeatureInterfaceOnRemote` | 获取 Feature 接口 |
| 1 | `RegisterClientDeathObserverOnRemote` | 注册客户端死亡监听 |
| 2 | `ClearAppStorageOnRemote` | 清除应用存储 |
| 3 | `ExitOnRemote` | 退出服务 |
| 4 | `GetSelfBundleNameOnRemote` | 获取自身 Bundle 名 |

**Feature 接口分发**:
```cpp
// services/distributeddataservice/app/src/feature_stub_impl.h:22
class FeatureStub : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.DistributedData.ServiceProxy");
};
```

---

## Feature 接口概览

| Feature | 注册名 | RPC 方法数 | 接口描述符 |
|---------|-------|-----------|-----------|
| **RDB** | `relational_store` | 27 | `u"OHOS.DistributedRdb.IRdbService"` |
| **KVDB** | `kv_store` | 22 | 内部 Feature 接口 |
| **Cloud** | `cloud` | 21 | `u"OHOS.CloudData.CloudServer"` |
| **Object** | `data_object` | 11 | 内部 Feature 接口 |
| **DataShare** | `data_share` | 27 | `u"OHOS.DataShare.IDataShareService"` |
| **UDMF** | `udmf` | 17 | 内部 Feature 接口 |
| **总计** | - | **125** | - |

---

## RDB Feature (关系型数据库)

**实现文件**: `services/distributeddataservice/service/rdb/`

### IPC 接口定义

```cpp
// services/distributeddataservice/service/rdb/rdb_service_stub.h
class RdbServiceStub : public IRemoteStub<IRdbService> {
    static constexpr RequestHandler HANDLERS[] = {
        &RdbServiceStub::OnRemoteObtainDistributedTableName,
        &RdbServiceStub::OnDelete,
        &RdbServiceStub::OnRemoteInitNotifier,
        // ... 共 27 个方法
    };
};
```

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `OnRemoteObtainDistributedTableName` | 获取分布式表名 | 低 |
| 1 | `OnDelete` | 删除数据库 | **中** |
| 2 | `OnRemoteInitNotifier` | 初始化通知器 | 低 |
| 3 | `OnRemoteSetDistributedTables` | 设置分布式表 | **中** |
| 4 | `OnRemoteDoSync` | 执行同步 | **高** |
| 5 | `OnRemoteDoAsync` | 异步同步 | **高** |
| 6 | `OnRemoteDoSubscribe` | 订阅数据变更 | 中 |
| 7 | `OnRemoteDoUnSubscribe` | 取消订阅 | 低 |
| 8 | `OnRemoteRegisterDetailProgressObserver` | 注册进度观察者 | 低 |
| 9 | `OnRemoteUnregisterDetailProgressObserver` | 注销进度观察者 | 低 |
| 10 | `OnRemoteDoRemoteQuery` | 远程查询 | **高** |
| 11 | `OnRemoteNotifyDataChange` | 通知数据变更 | **中** |
| 12 | `OnRemoteSetSearchable` | 设置可搜索 | 低 |
| 13 | `OnRemoteQuerySharingResource` | 查询共享资源 | **中** |
| 14 | `OnBeforeOpen` | 打开前回调 | 低 |
| 15 | `OnAfterOpen` | 打开后回调 | 低 |
| 16 | `OnIsSupportSilent` | 是否支持静默访问 | 低 |
| 17 | `OnReportStatistic` | 报告统计信息 | 低 |
| 18 | `OnDisable` | 禁用功能 | **中** |
| 19 | `OnEnable` | 启用功能 | **中** |
| 20 | `OnGetPassword` | 获取密码 | **高** |
| 21 | `OnLockCloudContainer` | 锁定云容器 | **中** |
| 22 | `OnUnlockCloudContainer` | 解锁云容器 | **中** |
| 23 | `OnGetDebugInfo` | 获取调试信息 | 低 |
| 24 | `OnGetDfxInfo` | 获取 DFX 信息 | 低 |
| 25 | `OnVerifyPromiseInfo` | 验证 Promise 信息 | **中** |
| 26 | `OnSetConfig` | 设置配置 | **中** |

### IRemoteBroker 接口

```cpp
// services/distributeddataservice/service/rdb/irdb_result_set.h:23
class IRdbResultSet : public IRemoteBroker {
    // 结果集远程访问接口
};

// services/distributeddataservice/service/rdb/rdb_notifier_proxy.h:23
class RdbNotifierProxyBroker : public IRdbNotifier, public IRemoteBroker {
    // 通知器代理
};
```

---

## KVDB Feature (KV数据库)

**实现文件**: `services/distributeddataservice/service/kvdb/`

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `OnGetStoreIds` | 获取存储 ID 列表 | 低 |
| 1 | `OnBeforeCreate` | 创建前回调 | 低 |
| 2 | `OnAfterCreate` | 创建后回调 | 低 |
| 3 | `OnDelete` | 删除存储 | **中** |
| 4 | `OnClose` | 关闭存储 | 低 |
| 5 | `OnSync` | 执行同步 | **高** |
| 6 | `OnCloudSync` | 云同步 | **高** |
| 7 | `OnRegServiceNotifier` | 注册服务通知器 | 低 |
| 8 | `OnUnregServiceNotifier` | 注销服务通知器 | 低 |
| 9 | `OnSetSyncParam` | 设置同步参数 | **中** |
| 10 | `OnGetSyncParam` | 获取同步参数 | 低 |
| 11 | `OnEnableCap` | 启用能力 | **中** |
| 12 | `OnDisableCap` | 禁用能力 | **中** |
| 13 | `OnSetCapability` | 设置能力 | **中** |
| 14 | `OnAddSubInfo` | 添加订阅信息 | 中 |
| 15 | `OnRmvSubInfo` | 移除订阅信息 | 中 |
| 16 | `OnSubscribe` | 订阅数据变更 | 中 |
| 17 | `OnUnsubscribe` | 取消订阅 | 低 |
| 18 | `OnGetBackupPassword` | 获取备份密码 | **高** |
| 19 | `OnNotifyDataChange` | 通知数据变更 | **中** |
| 20 | `OnPutSwitch` | 写入开关数据 | **中** |
| 21 | `OnGetSwitch` | 读取开关数据 | 低 |
| 22 | `OnSubscribeSwitchData` | 订阅开关数据 | 中 |
| 23 | `OnUnsubscribeSwitchData` | 取消订阅开关数据 | 低 |
| 24 | `OnSetConfig` | 设置配置 | **中** |
| 25 | `OnRemoveDeviceData` | 移除设备数据 | **高** |

### 权限检查点

```cpp
// services/distributeddataservice/service/kvdb/kvdb_service_stub.cpp
bool KVDBServiceStub::CheckPermission() {
    // 检查 ohos.permission.DISTRIBUTED_DATASYNC
    // 调用 AccessTokenKit::VerifyAccessToken()
}
```

---

## Cloud Feature (云同步)

**实现文件**: `services/distributeddataservice/service/cloud/`

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `OnEnableCloud` | 启用云同步 | **高** |
| 1 | `OnDisableCloud` | 禁用云同步 | **中** |
| 2 | `OnChangeAppSwitch` | 修改应用云开关 | **中** |
| 3 | `OnClean` | 清理云数据 | **高** |
| 4 | `OnNotifyDataChange` | 通知数据变更 | **中** |
| 5 | `OnNotifyChange` | 通知变更 | **中** |
| 6 | `OnQueryStatistics` | 查询统计信息 | 低 |
| 7 | `OnQueryLastSyncInfo` | 查询上次同步信息 | 低 |
| 8 | `OnSetGlobalCloudStrategy` | 设置全局云策略 | **高** |
| 9 | `OnCloudSync` | 执行云同步 | **高** |
| 10 | `OnAllocResourceAndShare` | 分配资源并分享 | **高** |
| 11 | `OnShare` | 分享数据 | **高** |
| 12 | `OnUnshare` | 取消分享 | **高** |
| 13 | `OnExit` | 退出 | 低 |
| 14 | `OnChangePrivilege` | 修改权限 | **高** |
| 15 | `OnQuery` | 查询 | 中 |
| 16 | `OnQueryByInvitation` | 通过邀请查询 | 中 |
| 17 | `OnConfirmInvitation` | 确认邀请 | **高** |
| 18 | `OnChangeConfirmation` | 修改确认状态 | **高** |
| 19 | `OnSetCloudStrategy` | 设置云策略 | **高** |
| 20 | `OnInitNotifier` | 初始化通知器 | 低 |

### 特殊权限

```cpp
// services/distributeddataservice/service/permission/include/permission_validator.h
// 需要 ohos.permission.CLOUDDATA_CONFIG
```

---

## Object Feature (分布式对象)

**实现文件**: `services/distributeddataservice/service/object/`

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `ObjectStoreSaveOnRemote` | 远程保存对象 | **中** |
| 1 | `ObjectStoreRevokeSaveOnRemote` | 远程撤销保存 | **中** |
| 2 | `ObjectStoreRetrieveOnRemote` | 远程检索对象 | **中** |
| 3 | `OnSubscribeRequest` | 订阅请求 | 中 |
| 4 | `OnUnsubscribeRequest` | 取消订阅请求 | 低 |
| 5 | `OnAssetChangedOnRemote` | 远程资源变更 | **中** |
| 6 | `ObjectStoreBindAssetOnRemote` | 远程绑定资源 | **中** |
| 7 | `OnDeleteSnapshot` | 删除快照 | **中** |
| 8 | `OnIsContinue` | 是否继续 | 低 |
| 9 | `OnSubscribeProgress` | 订阅进度 | 低 |
| 10 | `OnUnsubscribeProgress` | 取消订阅进度 | 低 |

### Callback Broker

```cpp
// services/distributeddataservice/service/object/include/object_callback_proxy.h
class ObjectSaveCallbackProxyBroker : public IObjectSaveCallback, public IRemoteBroker;
class ObjectRevokeSaveCallbackProxyBroker : public IObjectRevokeSaveCallback, public IRemoteBroker;
class ObjectRetrieveCallbackProxyBroker : public IObjectRetrieveCallback, public IRemoteBroker;
class ObjectChangeCallbackProxyBroker : public IObjectChangeCallback, public IRemoteBroker;
class ObjectProgressCallbackProxyBroker : public IObjectProgressCallback, public IRemoteBroker;
```

---

## DataShare Feature (数据共享)

**实现文件**: `services/distributeddataservice/service/data_share/`

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `OnQuery` | 查询数据 | 中 |
| 1 | `OnAddTemplate` | 添加模板 | **中** |
| 2 | `OnDelTemplate` | 删除模板 | **中** |
| 3 | `OnPublish` | 发布数据 | **中** |
| 4 | `OnGetData` | 获取数据 | 中 |
| 5 | `OnSubscribeRdbData` | 订阅 RDB 数据 | 中 |
| 6 | `OnUnsubscribeRdbData` | 取消订阅 | 低 |
| 7 | `OnEnableRdbSubs` | 启用订阅 | **中** |
| 8 | `OnDisableRdbSubs` | 禁用订阅 | **中** |
| 9 | `OnSubscribePublishedData` | 订阅发布数据 | 中 |
| 10 | `OnUnsubscribePublishedData` | 取消订阅 | 低 |
| 11 | `OnEnablePubSubs` | 启用发布订阅 | **中** |
| 12 | `OnDisablePubSubs` | 禁用发布订阅 | **中** |
| 13 | `OnNotifyConnectDone` | 通知连接完成 | 低 |
| 14 | `OnNotifyObserver` | 通知观察者 | **中** |
| 15 | `OnSetSilentSwitch` | 设置静默开关 | **高** |
| 16 | `OnGetSilentProxyStatus` | 获取静默代理状态 | 低 |
| 17 | `OnRegisterObserver` | 注册观察者 | 中 |
| 18 | `OnUnregisterObserver` | 注销观察者 | 低 |
| 19 | `OnInsertEx` | 插入数据 | **中** |
| 20 | `OnUpdateEx` | 更新数据 | **中** |
| 21 | `OnDeleteEx` | 删除数据 | **高** |
| 22 | `OnPublishProxyData` | 发布代理数据 | **高** |
| 23 | `OnDeleteProxyData` | 删除代理数据 | **高** |
| 24 | `OnGetProxyData` | 获取代理数据 | 中 |
| 25 | `OnSubscribeProxyData` | 订阅代理数据 | 中 |
| 26 | `OnUnsubscribeProxyData` | 取消订阅 | 低 |

### 权限验证

```cpp
// services/distributeddataservice/service/data_share/data_share_service_impl.cpp
// VerifyPermission() - 验证 ohos.permission.DISTRIBUTED_DATASYNC
// VerifyAcrossAccountsPermission() - 跨账户权限
// PermitDelegate::VerifyPermission() - 代理权限
```

---

## UDMF Feature (统一数据管理)

**实现文件**: `services/distributeddataservice/service/udmf/`

### RPC 方法清单

| Code | 方法名 | 说明 | 风险等级 |
|------|-------|------|---------|
| 0 | `OnSetData` | 设置数据 | **高** |
| 1 | `OnGetData` | 获取数据 | 中 |
| 2 | `OnGetBatchData` | 批量获取数据 | 中 |
| 3 | `OnUpdateData` | 更新数据 | **高** |
| 4 | `OnDeleteData` | 删除数据 | **高** |
| 5 | `OnGetSummary` | 获取摘要 | 低 |
| 6 | `OnAddPrivilege` | 添加权限 | **高** |
| 7 | `OnSync` | 同步 | **高** |
| 8 | `OnIsRemoteData` | 是否远程数据 | 低 |
| 9 | `OnSetAppShareOption` | 设置应用分享选项 | **中** |
| 10 | `OnGetAppShareOption` | 获取应用分享选项 | 低 |
| 11 | `OnRemoveAppShareOption` | 移除应用分享选项 | **中** |
| 12 | `OnObtainAsynProcess` | 获取异步进程 | 低 |
| 13 | `OnClearAsynProcessByKey` | 清除异步进程 | 低 |
| 14 | `OnPushDelayData` | 推送延迟数据 | **中** |
| 15 | `OnSetDelayInfo` | 设置延迟信息 | **中** |
| 16 | `OnGetDataIfAvailable` | 获取可用数据 | 中 |

### 权限检查

```cpp
// services/distributeddataservice/service/udmf/permission/data_checker.cpp
// URI 权限验证
// services/distributeddataservice/service/udmf/permission/uri_permission_manager.cpp
// URI 授权/撤销
```

---

## 公共错误码

### 通用错误码 (GeneralError)

```cpp
// services/distributeddataservice/framework/include/error/general_error.h
enum GeneralError {
    E_OK = 0,                           // 成功
    E_ERROR = 1,                        // 通用错误
    E_INVALID_ARGS = 2,                 // 无效参数
    E_NOT_SUPPORT = 3,                  // 不支持
    E_NOT_FOUND = 4,                    // 未找到
    E_ALREADY_EXISTS = 5,               // 已存在
    E_NETWORK_ERROR = 6,                // 网络错误
    E_CLOUD_DISABLED = 7,               // 云同步禁用
    E_LOCKED_BY_OTHERS = 8,             // 被其他锁定
    E_VERSION_CONFLICT = 9,             // 版本冲突
    E_FULL_SYNC_IN_PROGRESS = 10,       // 全量同步中
    E_DB_ERROR = 11,                    // 数据库错误
    E_INVALID_PASSWD = 12,              // 无效密码
    E_INVALID_CIPHER = 13,              // 无效加密
    E_BUSY = 14,                        // 忙
    E_CHANGE_UNPUBLISHED = 15,          // 未发布变更
    E_SYNC_IN_PROGRESS = 16,            // 同步中
    E_IPC_ERROR = 17,                   // IPC 错误
    E_REKEY_IN_PROGRESS = 18,           // 重加密中
    E_INVALID_SCHEMA = 19,              // 无效 Schema
};
```

### Cloud 错误码

```cpp
// services/distributeddataservice/framework/include/cloud/sharing_center.h
enum Code {
    IPC_ERROR = 77,
    SHARE_INVALID_ARGS = 78,
    STRATEGY_INVALID_ARGS = 79,
    CLOUD_DISABLE = 80,
    // ...
};
```

---

## 风险等级说明

| 等级 | 定义 | 示例 |
|-----|------|------|
| **高** | 可能导致数据泄露、权限提升或拒绝服务 | 删除数据、修改权限、获取密码 |
| **中** | 可能影响数据完整性或可用性 | 同步操作、配置修改、订阅管理 |
| **低** | 主要用于查询或监控，风险较小 | 获取统计、查询状态、调试信息 |

---

## 相关文档

- [攻击面分析](05_AttackSurface.md) - 详细安全分析
- [安全评审](03_Security.md) - 风险评估
- [架构设计](01_Architecture.md) - 系统架构

---

**证据来源**:
- RDB: `services/distributeddataservice/service/rdb/rdb_service_stub.h`
- KVDB: `services/distributeddataservice/service/kvdb/kvdb_service_stub.h`
- Cloud: `services/distributeddataservice/service/cloud/cloud_service_stub.h`
- Object: `services/distributeddataservice/service/object/include/object_service_stub.h`
- DataShare: `services/distributeddataservice/service/data_share/data_share_service_stub.h`
- UDMF: `services/distributeddataservice/service/udmf/udmf_service_stub.h`
