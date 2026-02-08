# 关键调用链

---

## 目的

本文档提供 DBMS 服务的关键调用链分析，从入口到核心逻辑的完整执行路径。

---

## 适用范围

- ✅ JS API 调用链
- ✅ IPC 调用链
- ✅ 服务内部调用链
- ❌ 详细代码实现（请参考源文件）

---

## JS API 到 SA 的完整调用链

### 调用链：getRemoteAbilityInfo（单个）

```
[应用] distributedBundle.getRemoteAbilityInfo(elementName, locale)
    │
    ▼
[N-API 绑定层] interfaces/kits/js/distributedBundle/native_module.cpp:31-43
    │
    │  Init() → napi_define_properties()
    │      → "getRemoteAbilityInfo" 映射到 GetRemoteAbilityInfo()
    │
    ▼
[JS 实现] interfaces/kits/js/distributedBundle/distributed_bundle.cpp:230-275
    │
    │ GetRemoteAbilityInfo(env, info)
    │
    │  ├─→ ParseElementName()
    │  │    └─→ napi_get_named_property() × 4
    │  │        (deviceId, bundleName, abilityName, moduleName)
    │
    │  ├─→ napi_create_async_work()
    │  │    ├─→ GetRemoteAbilityInfoExec() (Execute)
    │  │    │    └─→ DistributedHelper::InnerGetRemoteAbilityInfo()
    │  │    │
    │  │    │    └─→ GetDistributedBundleMgr()
    │  │    │
    │  │    │    │    └─→ SystemAbilityManagerClient::GetSystemAbilityManager()
    │  │    │    │    │    └─→ GetSystemAbility(SA_ID=402)
    │  │    │    │    │    └─→ iface_cast<IDistributedBms>()
    │  │    │    │
    │  │    │    └─→ Proxy::GetRemoteAbilityInfo()
    │  │    │
    │  │    │    ├─→ SendRequest()
    │  │    │    │    └─→ IPC (Binder/HIDL)
    │  │    │
    │  └─→ GetRemoteAbilityInfoComplete() (Complete)
    │         └─→ ConvertRemoteAbilityInfo()
    │              ├─→ napi_create_object()
    │              ├─→ napi_create_string_utf8() × 4 (elementName)
    │              ├─→ napi_create_string_utf8() (label)
    │              └─→ napi_create_string_utf8() (icon)
    │
    └─→ napi_resolve_deferred() 或 napi_call_function()
```

### 关键代码位置

| 步骤 | 文件 | 行号 |
|------|------|------|
| N-API 注册 | native_module.cpp | 49-64 |
| 参数解析 | distributed_bundle.cpp | 211-255 |
| 异步工作创建 | distributed_bundle.cpp | 267-272 |
| 执行函数 | distributed_bundle.cpp | 182-191 |
| Helper 调用 | distributed_helper.cpp | 30-49 |
| IPC 代理获取 | distributed_bundle_mgr.cpp | 71-81 |
| Proxy 发送 | distributed_bms_proxy.cpp | 123-131 |
| SA 接收 | distributed_bms_host.cpp | 31-43 |
| SA 业务逻辑 | distributed_bms.cpp | 257-673 |
| 结果转换 | distributed_bundle.cpp | 152-189 |
| Promise 解析 | distributed_bundle.cpp | 202-228 |

---

## 服务内部调用链

### 调用链：本地 Ability 查询

```
[Proxy] DistributedBmsProxy::GetAbilityInfo()
    │
    ▼ SendRequest(GET_ABILITY_INFO, data, reply)
    │
    │ [IPC] MessageParcel / MessageOption
    │
    ▼
[Stub] DistributedBmsHost::OnRemoteRequest()
    │
    │  ├─→ ReadInterfaceToken()
    │  ├─→ HandleGetAbilityInfo()
    │
    │  ▼
    │  ┌─────────────────────────────────────────────────┐
    │  │ distributed_bms.cpp:390-475        │
    │  │ GetAbilityInfo(elementName, remoteAbilityInfo)  │
    │  │                                         │
    │  │  ├─→ VerifyCallingPermissionOrAclCheck()   │
    │  │  │    ├─→ GetLocalDevice(dmDeviceInfo)      │
    │  │  │    ├─→ IPCSkeleton::GetCallingDeviceID() │
    │  │  │    └─→ 验证设备是否本地       │
    │  │  │                                         │
    │  │  ├─→ VerifyCallingPermission() (如需要）      │
    │  │  │    ├─→ IPCSkeleton::GetCallingTokenID() │
    │  │  │    └─→ VerifyAccessToken()            │
    │  │  │                                         │
    │  │  └─→ GetBundleMgr()->GetAbilityInfo()    │
    │  │           │                                 │
    │  │           └─→ IBundleMgr (SA = 401)          │
    │  │                                             │
    │  └─→ ReplyParcel.WriteParcelableInfo()   │
    │
    └─────────────────────────────────────────────────────────┘
```

### 调用链：跨设备 Ability 查询

```
[Proxy] DistributedBmsProxy::GetRemoteAbilityInfo()
    │
    ▼ SendRequest(GET_REMOTE_ABILITY_INFO, data, reply)
    │
    │ [IPC] MessageParcel / MessageOption
    │
    ▼
[Stub] DistributedBmsHost::OnRemoteRequest()
    │
    │  ├─→ ReadInterfaceToken()
    │  ├─→ HandleGetRemoteAbilityInfo()
    │
    │  ▼
    │  ┌─────────────────────────────────────────────────┐
    │  │ distributed_bms.cpp:230-289        │
    │  │ GetRemoteAbilityInfo(elementName, remoteAbilityInfo)  │
    │  │                                         │
    │  │  ├─→ VerifyCallingPermissionOrAclCheck()   │
    │  │  │    ├─→ GetLocalDevice(dmDeviceInfo)      │
    │  │  │    ├─→ IPCSkeleton::GetCallingDeviceID() │
    │  │  │    └─→ 检查是否需要本地验证         │
    │  │  │                                         │
    │  │  ├─→ CheckAclData(info)                  │
    │  │  │    └─→ DbmsDeviceManager::CheckAclData() │
    │  │  │          ├─→ GetOsAccountData()             │
    │  │  │          ├─→ GetLocalDeviceInfo()          │
    │  │  │          └─→ DeviceManager::CheckAccessControl()  │
    │  │  │                                         │
    │  │  └─→ CheckSystemAbility(networkId)         │
    │  │           └─→ 远程 DBMS SA → GetRemoteAbilityInfo()
    │  │                          └─→ IBundleMgr SA → GetRemoteAbilityInfo()
    │  │                                             │
    │  └─→ ReplyParcel.WriteParcelableInfo()   │
    │
    └─────────────────────────────────────────────────────────┘
```

---

## 关键时序图

### 时序 1：跨设备查询完整流程

```
应用                  N-API               Proxy                DBMS SA               DeviceManager         远程DBMS
  │                      │                   │                     │                     │              │
  │ getRemoteAbilityInfo  │                   │                     │                     │              │
  ├─ napi_create_async─┼─ ParseElementName ─┼─ SendRequest ───────────┼────── OnStart ──┤
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │   VerifyCallingPermissionOrAclCheck              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │   ┌───────────────────┐  │
  │                      │                   │                     │   │ GetLocalDevice      │  │
  │                      │                   │                     │   │ GetOsAccountData    │  │
  │                      │                   │                     │   └───────────────────┘  │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │  ├─ CheckSystemAbility   ────┼──────────────┤
  │                      │                   │                     │  │                     │              │
  │                      │                   │                     │  │                     │              │
  │                      │                   │                     │  │   CheckAccessControl   │
  │                      │                   │                     │  │   └────────────────────┘  │
  │                      │                   │                     │                     │              │
  │                      ◄─────────────────── ReplyMessage ────┼───────┼────── napi_resolve    │
  │                      │                   │                     │                     │              │
  │                      │                   │               ConvertRemoteAbilityInfo              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      │                   │                     │                     │              │
  │                      ◄── callback/Promise resolve ────────────┼─────────────────────────────┘
  │                      │                   │                     │
```

---

## 依赖服务调用链

### Bundle Manager SA 调用

**调用点**: `services/dbms/src/distributed_bms.cpp:183-197`

```
DBMS (SA 402)
    │
    ▼ GetBundleMgr()
    │
    ├─→ SystemAbilityManagerClient::GetInstance()
    ├─→ GetSystemAbilityManager()
    └─→ GetSystemAbility(BUNDLE_MGR_SERVICE_SYS_ABILITY_ID)
        │
        ▼
    Bundle Manager SA (401)
        ├─→ GetAbilityInfo()
        │   └─→ 查询本地 Bundle 数据库
        └─→ 返回 AbilityInfo
```

### Device Manager 调用

**调用点**: `services/dbms/src/dbms_device_manager.cpp:69-85`

```
DBMS (SA 402)
    │
    ▼ GetUdidByNetworkId() / GetUuidByNetworkId()
    │
    ├─→ InitDeviceManager()
    │   └─→ DeviceManager::InitDeviceManager()
    │
    └─→ DeviceManager::GetInstance()
        └─→ GetUdidByNetworkId() / GetUuidByNetworkId()
            └─→ 查询设备数据库
```

---

## 性能关键点

### 瓶颈分析

| 瓶颈 | 位置 | 影响 |
|------|------|------|
| IPC 调用 | 分布式_bms_proxy.cpp:123-131 | 每次查询需要多次 IPC 调用 |
| 权限验证 | distributed_bms.cpp:638-651 | 每次查询都验证权限 |
| 跨设备查询 | dbms_device_manager.cpp:102-132 | 依赖 Device Manager，增加延迟 |
| 图标编码 | distributed_bms.cpp:502-508 | Base64 编码消耗 CPU |

### 优化建议

```javascript
// 使用批量查询替代多次单条查询
const results = await distributedBundle.getRemoteAbilityInfo([
    elementName1, elementName2, elementName3, ...
]);
```

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:407-492`

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| N-API 注册 | interfaces/kits/js/distributedBundle/native_module.cpp:49-64 |
| 参数解析 | interfaces/kits/js/distributedBundle/distributed_bundle.cpp:211-255 |
| IPC 代理 | interfaces/inner_api/src/distributed_bms_proxy.cpp:123-131 |
| SA 处理 | services/dbms/src/distributed_bms_host.cpp:31-43 |
| 权限验证 | services/dbms/src/distributed_bms.cpp:638-671 |
| Bundle Manager 调用 | services/dbms/src/distributed_bms.cpp:183-197 |
| Device Manager 调用 | services/dbms/src/dbms_device_manager.cpp:69-85 |
