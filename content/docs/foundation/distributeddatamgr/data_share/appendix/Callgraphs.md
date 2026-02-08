# 附录：关键调用链

## 目的

本文档梳理 Data Share 中关键操作的完整调用链，帮助开发者理解代码执行流程。

## createDataShareHelper 调用链

```
JS: dataShare.createDataShareHelper(context, uri, options)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_CreateDataShareHelper() 
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:186]
    │
    ├── ValidateCreateParam()
    │   ├── IsSystemApp()  [line 41-45]
    │   ├── GetStageModeContext()
    │   ├── GetUri()
    │   └── GetOptions()
    │       └── GetIsProxy() / GetWaitTime()
    │
    ├── napi_new_instance() 创建 JS 对象
    │
    └── ExecuteCreator()
        │
        ▼
Native: DataShareHelper::Creator()
    [frameworks/native/consumer/src/datashare_helper.cpp:97-128]
    │
    ├── IsProxy(uri) ?
    │   ├── YES: CreateServiceHelper() ──► Silent Mode
    │   └── NO: CreateExtHelper() ──► Non-Silent Mode
    │
    └── CreateServiceHelper()
        │
        ▼
        DataShareHelper::Create()
            [frameworks/native/consumer/src/datashare_helper.cpp:186-196]
            │
            ├── GetSilentProxyStatus()
            ├── CreateProxyHelper() ──► datashareproxy://
            └── CreateServiceHelper()
                │
                ▼
                DataShareHelperImpl 创建
                    ├── generalCtl_ = GeneralControllerServiceImpl
                    ├── extSpCtl_ = ExtSpecialController
                    ├── persistentDataCtl_ = PersistentDataController
                    └── publishedDataCtl_ = PublishedDataController
```

## Insert 调用链

```
JS: helper.insert(uri, valuesBucket)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_Insert()
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:301-347]
    │
    ├── GetUri()
    ├── GetValueBucketObject() 解析 ValuesBucket
    └── AsyncCall 异步执行
        │
        ▼
Native: DataShareHelperImpl::Insert()
    [frameworks/native/consumer/src/datashare_helper_impl.cpp:123-135]
    │
    └── generalCtl_->Insert()
        │
        ├── (Silent) GeneralControllerServiceImpl::Insert()
        │   [frameworks/native/consumer/controller/service/src/general_controller_service_impl.cpp:42-60]
        │   │
        │   └── TimedQuery() / Execute()
        │       │
        │       ▼
        │       DataShareManagerImpl::Insert()
        │           │
        │           ▼
        │           DataShareServiceProxy::Insert() ──► IPC
        │
        └── (Non-Silent) GeneralControllerProviderImpl::Insert()
            [frameworks/native/consumer/controller/provider/src/general_controller_provider_impl.cpp:25-38]
            │
            └── DataShareProxy::Insert()
                │
                ▼
                SendRequest(CMD_INSERT) ──► IPC

IPC: SendRequest(CMD_INSERT)
    │
    ▼
Provider: DataShareStub::OnRemoteRequest()
    [frameworks/native/provider/src/datashare_stub.cpp:78-113]
    │
    └── 分发到 CmdInsert()
        [frameworks/native/provider/src/datashare_stub.cpp:207-228]
        │
        └── DataShareStubImpl::Insert()
            [frameworks/native/provider/src/datashare_stub_impl.cpp:218-266]
            │
            ├── VerifyProvider()  [line 229] ⚠️ 返回值被忽略
            ├── CheckCallingPermission(writePermission)  [line 239]
            │   └── AccessTokenKit::VerifyAccessToken()
            └── InsertInner()
                │
                └── uvQueue_->JsSyncCall()
                    │
                    ▼
                    JsDataShareExtAbility::Insert()
                        [JS Extension 执行]
```

## Query 调用链 (含共享内存)

```
JS: helper.query(uri, predicates, columns)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_Query()
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:394-442]
    │
    ├── GetUri()
    ├── UnwrapDataSharePredicates()
    └── AsyncCall 异步执行
        │
        ▼
Native: DataShareHelperImpl::Query()
    [frameworks/native/consumer/src/datashare_helper_impl.cpp:238-275]
    │
    └── generalCtl_->Query()
        │
        └── DataShareProxy::Query()
            [frameworks/native/consumer/src/datashare_proxy.cpp:397-429]
            │
            ├── MessageParcel 封装
            └── SendRequest(CMD_QUERY) ──► IPC

IPC: SendRequest(CMD_QUERY)
    │
    ▼
Provider: DataShareStub::CmdQuery()
    [frameworks/native/provider/src/datashare_stub.cpp:374-397]
    │
    └── DataShareStubImpl::Query()
        [frameworks/native/provider/src/datashare_stub_impl.cpp:542-590]
        │
        ├── VerifyProvider()
        ├── CheckCallingPermission(readPermission)
        └── QueryInner()
            │
            ├── Create SharedMemory (Ashmem)
            │   └── ISharedResultSet::CreateAshmem()
            │
            └── uvQueue_->JsSyncCall()
                │
                └── JsDataShareExtAbility::Query()
                    │
                    └── ResultSetBridge::WriteToAshmem()
                        [数据写入共享内存]

Return: IPC Response
    │
    ▼
Consumer: DataShareProxy::Query() 返回
    │
    └── ISharedResultSet::ReadFromAshmem()
        [从共享内存读取数据]
        │
        ▼
        DataShareResultSet 创建
            │
            ▼
NAPI: ResultSet 包装为 JS 对象返回
```

## RegisterObserver 调用链

```
JS: helper.on('dataChange', uri, callback)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_On()
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:877-902]
    │
    ├── 解析 eventType
    └── 分发处理
        │
        ├── 'rdbDataChange' ──► Napi_SubscribeRdbObserver()
        ├── 'publishedDataChange' ──► Napi_SubscribePublishedObserver()
        └── 'dataChange' ──► Napi_RegisterObserver()
            │
            ▼
            AsyncCall 执行
                │
                ▼
Native: DataShareHelperImpl::RegisterObserver()
    [frameworks/native/consumer/src/datashare_helper_impl.cpp:308-327]
    │
    └── generalCtl_->RegisterObserver()
        │
        └── DataShareProxy::RegisterObserver()
            [frameworks/native/consumer/src/datashare_proxy.cpp:525-548]
            │
            └── SendRequest(CMD_REGISTER_OBSERVER) ──► IPC

IPC: SendRequest(CMD_REGISTER_OBSERVER)
    │
    ▼
Provider: DataShareStub::CmdRegisterObserver()
    [frameworks/native/provider/src/datashare_stub.cpp:500-522]
    │
    └── DataShareStubImpl::RegisterObserver()
        [frameworks/native/provider/src/datashare_stub_impl.cpp:683-695]
        │
        ├── VerifyProvider()
        ├── CheckCallingPermission(readPermission)  [line 691]
        └── DataObsMgrClient::RegisterObserver()
            │
            ▼
            系统服务 DataObsMgr
                │
                └── 观察者注册完成
```

## NotifyChange 调用链

```
JS: helper.notifyChange(uri)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_NotifyChange()
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:700-714]
    │
    └── AsyncCall 执行
        │
        ▼
Native: DataShareHelperImpl::NotifyChange()
    │
    └── generalCtl_->NotifyChange()
        │
        └── DataShareProxy::NotifyChange() ──► IPC

IPC: SendRequest(CMD_NOTIFY_CHANGE)
    │
    ▼
Provider: DataShareStub::CmdNotifyChange()
    │
    └── DataShareStubImpl::NotifyChange()
        [frameworks/native/provider/src/datashare_stub_impl.cpp:753-776]
        │
        └── DataObsMgrClient::NotifyChange()
            │
            ▼
            系统服务 DataObsMgr
                │
                └── 通知所有注册的观察者
                    │
                    ▼
                    Observer Callback 触发
                        │
                        ▼
                        JS: callback(changeInfo)
```

## Publish 调用链

```
JS: helper.publish(data, bundleName)
    │
    ▼
NAPI: NapiDataShareHelper::Napi_Publish()
    [frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:735-751]
    │
    ├── GetBundleName()
    └── AsyncCall 执行
        │
        ▼
Native: DataShareHelperImpl::Publish()
    [frameworks/native/consumer/src/datashare_helper_impl.cpp:395-418]
    │
    └── publishedDataCtl_->Publish()
        │
        └── PublishedDataController::Publish()
            │
            └── DataShareManagerImpl::Publish()
                │
                ▼
                DataShareServiceProxy::Publish() ──► IPC

IPC: SendRequest(CMD_PUBLISH)
    │
    ▼
Provider: DataShareServiceProxy ──► datamgr_service
    │
    └── 系统服务处理发布数据
        │
        ▼
        数据存储 + 通知订阅者
```

## 权限检查调用链

```
Entry: DataShareStubImpl::CheckCallingPermission(permission)
    [frameworks/native/provider/src/datashare_stub_impl.cpp:55-64]
    │
    ├── IPCSkeleton::GetCallingTokenID()
    │
    ├── if permission.empty() ──► return true
    │
    └── AccessTokenKit::VerifyAccessToken(token, permission)
        │
        ▼
        系统服务 AccessToken
            │
            └── 权限检查结果返回

Entry: DataSharePermission::VerifyPermission(tokenID, uri, isRead)
    [frameworks/native/permission/src/data_share_permission.cpp:63-91]
    │
    ├── 检查 URI 为空
    │
    ├── DataShareCalledConfig::GetProviderInfo()
    │   │
    │   ├── GetFromProxyData()
    │   │   └── GetBundleInfoFromBMS()
    │   │       └── BundleMgrClient::GetBundleInfo()
    │   │           └── 系统服务 BMS
    │   │
    │   └── 解析 proxyData 获取 readPermission/writePermission
    │
    ├── if permission.empty() ──► return ERR_PERMISSION_DENIED
    │
    └── AccessTokenKit::VerifyAccessToken(tokenID, permission)
        │
        └── 权限检查结果返回
```

## 关键文件路径汇总

| 功能 | 文件路径 | 关键行号 |
|------|----------|----------|
| NAPI 入口 | `frameworks/js/napi/dataShare/src/native_datashare_module.cpp` | 30-44 |
| Helper NAPI | `frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp` | 186-236 |
| Helper 创建 | `frameworks/native/consumer/src/datashare_helper.cpp` | 97-196 |
| Helper 实现 | `frameworks/native/consumer/src/datashare_helper_impl.cpp` | 123-418 |
| Proxy 实现 | `frameworks/native/consumer/src/datashare_proxy.cpp` | 158-548 |
| Stub 实现 | `frameworks/native/provider/src/datashare_stub.cpp` | 78-500 |
| StubImpl | `frameworks/native/provider/src/datashare_stub_impl.cpp` | 55-776 |
| 权限检查 | `frameworks/native/permission/src/data_share_permission.cpp` | 63-345 |
| 共享内存 | `frameworks/native/common/src/ishared_result_set.cpp` | - |

## 时序图

```
Client (JS)              NAPI              Native Consumer         IPC              Native Provider
    │                       │                      │                  │                     │
    │ insert(uri, data)     │                      │                  │                     │
    │──────────────────────>│                      │                  │                     │
    │                       │ Napi_Insert()        │                  │                     │
    │                       │─────────────────────>│                  │                     │
    │                       │                      │ Insert()         │                     │
    │                       │                      │─────────────────>│                     │
    │                       │                      │                  │ SendRequest()       │
    │                       │                      │                  │────────────────────>│
    │                       │                      │                  │                     │ OnRemoteRequest()
    │                       │                      │                  │                     │────────┐
    │                       │                      │                  │                     │        │
    │                       │                      │                  │                     │        ▼
    │                       │                      │                  │                     │ CmdInsert()
    │                       │                      │                  │                     │────────┤
    │                       │                      │                  │                     │        │
    │                       │                      │                  │                     │        ▼
    │                       │                      │                  │                     │ CheckPermission()
    │                       │                      │                  │                     │────────┤
    │                       │                      │                  │                     │        │
    │                       │                      │                  │                     │        ▼
    │                       │                      │                  │                     │ JsSyncCall()
    │                       │                      │                  │                     │────────┤
    │                       │                      │                  │                     │        │
    │                       │                      │                  │                     │◄───────┘
    │                       │                      │                  │                     │ (JS执行)
    │                       │                      │                  │                     │
    │                       │                      │                  │ Return              │
    │                       │                      │                  │◄────────────────────│
    │                       │                      │ Return           │                     │
    │                       │                      │◄─────────────────│                     │
    │                       │ Return               │                  │                     │
    │                       │◀─────────────────────│                  │                     │
    │ Promise.resolve()     │                      │                  │                     │
    │◀──────────────────────│                      │                  │                     │
    │                       │                      │                  │                     │
```

## 相关文档

- [架构设计](../05_Architecture.md) - 组件关系和数据流
- [N-API 参考](../04_NAPI_Reference.md) - JS API 详情
- [内部 API](../06_Inner_API.md) - Native 接口详情
