# N-API JS 接口文档

## 1. 接口总览

NetManager Base 提供 4 个 N-API 模块，共 100+ JS API：

| 模块名 | 命名空间 | API 数量 | 说明 |
|--------|----------|----------|------|
| network | `@ohos.net.network` | 3 | 网络基础能力 |
| net.connection | `@ohos.net.connection` | 60+ | 网络连接管理 |
| net.policy | `@ohos.net.policy` | 20 | 网络策略管理 |
| net.statistics | `@ohos.net.statistics` | 20 | 流量统计查询 |

---

## 2. 模块注册点

### 2.1 注册文件位置

| 模块 | 注册文件 | 注册宏 |
|------|----------|--------|
| network | `frameworks/js/napi/network/network_module/src/network_module.cpp:1` | `NAPI_MODULE(network, NetworkModule::InitNetworkModule)` |
| net.statistics | `frameworks/js/napi/netstats/src/statistics_module.cpp:1` | `napi_module_register(&g_statisticsModule)` |
| net.policy | `frameworks/js/napi/netpolicy/src/netpolicy_module.cpp:1` | `napi_module_register(&g_policyModule)` |
| net.connection | `frameworks/js/napi/connection/connection_module/src/connection_module.cpp:1` | `napi_module_register(&g_connectionModule)` |

---

## 3. network 模块

### 3.1 API 清单

| JS API | C++ 函数 | 模式 | 上下文文件 |
|--------|----------|------|------------|
| `getType()` | `NetworkModule::GetType` | 异步 | `gettype_context.cpp` |
| `subscribe()` | `NetworkModule::Subscribe` | 异步 | `subscribe_context.cpp` |
| `unsubscribe()` | `NetworkModule::Unsubscribe` | 同步 | `unsubscribe_context.cpp` |

### 3.2 调用链

```
JS: getType()
  ↓
NAPI: NetworkModule::GetType()
  ↓
Context: GetTypeContext::ParseParams()  [参数解析]
  ↓
AsyncWork: NetworkAsyncWork::ExecGetType()
  ↓
Inner: NetConnClient::GetType()
  ↓
IPC → NetConnService → NetsysController → NetsysNative → Kernel
```

---

## 4. net.connection 模块

### 4.1 核心 API 分类

#### 4.1.1 网络信息查询

| JS API | 模式 | C++ 执行函数 | 权限 |
|--------|------|--------------|------|
| `getDefaultNet()` | 异步 | `ExecGetDefaultNet` | 无 |
| `getDefaultNetSync()` | 同步 | `ExecGetDefaultNet` | 无 |
| `hasDefaultNet()` | 异步 | `ExecHasDefaultNet` | 无 |
| `hasDefaultNetSync()` | 同步 | `ExecHasDefaultNet` | 无 |
| `getAllNets()` | 异步 | `ExecGetAllNets` | 内部权限 |
| `getAllNetsSync()` | 异步 | `ExecGetAllNets` | 内部权限 |
| `getNetCapabilities(netHandle)` | 异步 | `ExecGetNetCapabilities` | 无 |
| `getNetCapabilitiesSync(netHandle)` | 同步 | `ExecGetNetCapabilities` | 无 |
| `getConnectionProperties(netHandle)` | 异步 | `ExecGetConnectionProperties` | 无 |
| `getConnectionPropertiesSync(netHandle)` | 同步 | `ExecGetConnectionProperties` | 无 |

**证据**: `frameworks/js/napi/connection/connection_module/src/connection_module.cpp:118-155`

#### 4.1.2 网络操作

| JS API | 模式 | 说明 |
|--------|------|------|
| `enableAirplaneMode()` | 异步 | 开启飞行模式 |
| `disableAirplaneMode()` | 异步 | 关闭飞行模式 |
| `reportNetConnected(netHandle)` | 异步 | 报告网络已连接 |
| `reportNetDisconnected(netHandle)` | 异步 | 报告网络已断开 |

#### 4.1.3 HTTP 代理

| JS API | 模式 | 说明 |
|--------|------|------|
| `getDefaultHttpProxy()` | 异步 | 获取默认代理 |
| `getGlobalHttpProxy()` | 异步 | 获取全局代理 |
| `setGlobalHttpProxy(proxy)` | 异步 | 设置全局代理 |
| `setAppHttpProxy(proxy)` | 同步 | 设置应用代理 |
| `setPacUrl(url)` | 同步 | 设置 PAC URL |
| `getPacUrl()` | 同步 | 获取 PAC URL |
| `getProxyMode()` | 异步 | 获取代理模式 |
| `setProxyMode(mode)` | 异步 | 设置代理模式 |

#### 4.1.4 DNS 相关

| JS API | 模式 | 说明 |
|--------|------|------|
| `getAddressesByName(host)` | 异步 | DNS 解析 |
| `getAddressesByNameWithOptions(host, opts)` | 异步 | DNS 解析(带选项) |
| `addCustomDnsRule(host, ip)` | 异步 | 添加自定义 DNS 规则 |
| `removeCustomDnsRule(host)` | 异步 | 移除自定义 DNS 规则 |
| `clearCustomDnsRules()` | 异步 | 清除自定义 DNS 规则 |

#### 4.1.5 网络接口管理

| JS API | 模式 | 权限 |
|--------|------|------|
| `setInterfaceUp(iface)` | 异步 | CONNECTIVITY_INTERNAL |
| `setNetInterfaceIpAddress(iface, ip)` | 异步 | CONNECTIVITY_INTERNAL |
| `addNetworkRoute(netId, dest, nextHop, iface)` | 异步 | CONNECTIVITY_INTERNAL |
| `createVlanInterface(iface, vlanId)` | 异步 | CONNECTIVITY_INTERNAL |
| `destroyVlanInterface(iface, vlanId)` | 异步 | CONNECTIVITY_INTERNAL |

#### 4.1.6 监听/事件

| JS API | 模式 | 说明 |
|--------|------|------|
| `createNetConnection()` | 构造器 | 创建 NetConnection 对象 |
| `NetConnection.register()` | 异步 | 注册网络监听 |
| `NetConnection.unregister()` | 异步 | 注销网络监听 |
| `NetConnection.on(type, callback)` | 同步 | 注册事件监听 |
| `NetConnection.off(type, callback)` | 同步 | 移除事件监听 |

**事件类型**:
- `netAvailable` - 网络可用
- `netLost` - 网络丢失
- `netCapabilitiesChange` - 网络能力变更
- `netConnectionPropertiesChange` - 连接属性变更
- `netUnavailable` - 网络不可用

### 4.2 参数校验

#### 4.2.1 参数解析位置

| 参数类型 | 解析文件 | 关键函数 |
|----------|----------|----------|
| NetHandle | `parse_nethandle_context.cpp` | `ParseNetHandle()` |
| NetSpecifier | `connection_module.cpp:87` | `ParseNetSpecifier()` |
| HttpProxy | `setglobalhttpproxy_context.cpp` | `ParseHttpProxy()` |
| 字符串 | `getaddressbyname_context.cpp` | `ParseHostName()` |

#### 4.2.2 边界检查

```cpp
// 示例: 数组长度限制
// frameworks/js/napi/connection/connection_module.cpp:69
uint32_t arrayLength = NapiUtils::GetArrayLength(env, obj) > MAX_ARRAY_LENGTH 
    ? MAX_ARRAY_LENGTH 
    : NapiUtils::GetArrayLength(env, obj);

// 示例: 枚举值校验
// frameworks/js/napi/connection/connection_module.cpp:97-103
bool ret = ParseTypesArray<NetBearType>(env, bearerTypes, capabilities.bearerTypes_, 
    [](uint32_t value) {
        return value >= 0 && value <= static_cast<uint32_t>(NetBearType::BEARER_DEFAULT);
    });
```

### 4.3 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `NETMANAGER_SUCCESS` | 0 | 成功 |
| `NETMANAGER_ERR_PARAMETER_ERROR` | 401 | 参数错误 |
| `NETMANAGER_ERR_CAPABILITY_NOT_SUPPORTED` | 801 | 能力不支持 |
| `NETMANAGER_ERR_INTERNAL_ERROR` | 2100001 | 内部错误 |
| `NETMANAGER_ERR_OPERATION_FAILED` | 2100002 | 操作失败 |
| `NETMANAGER_ERR_NETWORK_NOT_FOUND` | 2100003 | 网络不存在 |
| `NET_CONN_ERR_NO_HTTP_PROXY` | 2100005 | 无代理配置 |
| `NET_CONN_ERR_HTTP_PROXY_INVALID` | 2100006 | 代理配置无效 |

---

## 5. net.policy 模块

### 5.1 API 清单

| JS API | 模式 | C++ 执行函数 | 权限 |
|--------|------|--------------|------|
| `setPolicyByUid(uid, policy)` | 异步 | `ExecSetPolicyByUid` | MANAGE_NET_STRATEGY |
| `getPolicyByUid(uid)` | 异步 | `ExecGetPolicyByUid` | 无 |
| `getUidsByPolicy(policy)` | 异步 | `ExecGetUidsByPolicy` | 无 |
| `setBackgroundPolicy(allow)` | 异步 | `ExecSetBackgroundPolicy` | MANAGE_NET_STRATEGY |
| `getBackgroundPolicy()` | 异步 | `ExecGetBackgroundPolicy` | 无 |
| `getBackgroundPolicyByUid(uid)` | 异步 | `ExecGetBackgroundPolicyByUid` | 无 |
| `setNetQuotaPolicies(policies)` | 异步 | `ExecSetNetQuotaPolicies` | MANAGE_NET_STRATEGY |
| `getNetQuotaPolicies()` | 异步 | `ExecGetNetQuotaPolicies` | 无 |
| `isUidNetAllowed(uid, isMetered)` | 异步 | `ExecIsUidNetAllowed` | 无 |
| `setDeviceIdleTrustlist(uid, isAllow)` | 异步 | `ExecSetDeviceIdleTrustlist` | MANAGE_NET_STRATEGY |
| `getDeviceIdleTrustlist()` | 异步 | `ExecGetDeviceIdleTrustlist` | 无 |
| `resetPolicies(simId)` | 异步 | `ExecResetPolicies` | MANAGE_NET_STRATEGY |
| `restoreAllPolicies(simId)` | 异步 | `ExecRestoreAllPolicies` | MANAGE_NET_STRATEGY |
| `updateRemindPolicy(netType, simId, remindType)` | 异步 | `ExecUpdateRemindPolicy` | MANAGE_NET_STRATEGY |
| `setNetworkAccessPolicy(uid, policy)` | 异步 | `ExecSetNetworkAccessPolicy` | MANAGE_NET_STRATEGY |
| `getNetworkAccessPolicy(uid)` | 异步 | `ExecGetNetworkAccessPolicy` | 无 |
| `on(type, callback)` | 异步 | `PolicyObserverWrapper::On` | 无 |
| `off(type, callback)` | 异步 | `PolicyObserverWrapper::Off` | 无 |

**证据**: `frameworks/js/napi/netpolicy/src/netpolicy_module.cpp`

### 5.2 事件类型

- `netUidPolicyChange` - UID 策略变更
- `netUidRuleChange` - UID 规则变更
- `netMeteredIfacesChange` - 计费接口变更
- `netQuotaPolicyChange` - 配额策略变更
- `netBackgroundPolicyChange` - 后台策略变更

---

## 6. net.statistics 模块

### 6.1 API 清单

| JS API | 模式 | C++ 执行函数 | 说明 |
|--------|------|--------------|------|
| `getIfaceRxBytes(iface)` | 异步 | `ExecGetIfaceRxBytes` | 接口下行流量 |
| `getIfaceTxBytes(iface)` | 异步 | `ExecGetIfaceTxBytes` | 接口上行流量 |
| `getCellularRxBytes()` | 异步 | `ExecGetCellularRxBytes` | 蜂窝下行流量 |
| `getCellularTxBytes()` | 异步 | `ExecGetCellularTxBytes` | 蜂窝上行流量 |
| `getAllRxBytes()` | 异步 | `ExecGetAllRxBytes` | 总下行流量 |
| `getAllTxBytes()` | 异步 | `ExecGetAllTxBytes` | 总上行流量 |
| `getUidRxBytes(uid)` | 异步 | `ExecGetUidRxBytes` | 应用下行流量 |
| `getUidTxBytes(uid)` | 异步 | `ExecGetUidTxBytes` | 应用上行流量 |
| `getSockfdRxBytes(sockfd)` | 异步 | `ExecGetSockfdRxBytes` | 套接字下行流量 |
| `getSockfdTxBytes(sockfd)` | 异步 | `ExecGetSockfdTxBytes` | 套接字上行流量 |
| `getTrafficStatsByIface(iface)` | 异步 | `ExecGetIfaceStats` | 接口详细统计 |
| `getTrafficStatsByUid(uid)` | 异步 | `ExecGetIfaceUidStats` | 应用详细统计 |
| `updateIfacesStats()` | 异步 | `ExecUpdateIfacesStats` | 更新接口统计 |
| `updateStatsData()` | 异步 | `ExecUpdateStatsData` | 更新统计数据 |
| `on(type, callback)` | 异步 | `StatisticsObserverWrapper::On` | 注册监听 |
| `off(type, callback)` | 异步 | `StatisticsObserverWrapper::Off` | 移除监听 |

**证据**: `frameworks/js/napi/netstats/src/statistics_module.cpp`

---

## 7. 异步工作实现

### 7.1 异步工作文件位置

| 模块 | 异步工作文件 |
|------|--------------|
| connection | `frameworks/js/napi/connection/async_work/src/connection_async_work.cpp` |
| network | `frameworks/js/napi/network/async_work/src/network_async_work.cpp` |
| netpolicy | `frameworks/js/napi/netpolicy/src/netpolicy_async_work.cpp` |
| netstats | `frameworks/js/napi/netstats/src/statistics_async_work.cpp` |

### 7.2 异步执行模式

```cpp
// 标准异步模式
napi_create_async_work(
    env,                    // napi_env
    nullptr,                // async_resource
    resourceName,           // 资源名称
    ExecuteCallback,        // 执行函数 (工作线程)
    CompleteCallback,       // 完成函数 (主线程)
    context,                // 上下文数据
    &asyncWork              // 输出: async_work
);

// 示例: connection 模块
// frameworks/js/napi/connection/async_work/src/connection_async_work.cpp
void ConnectionAsyncWork::ExecGetDefaultNet(napi_env env, void *data) {
    auto context = static_cast<GetDefaultNetContext *>(data);
    context->netHandle_ = NetConnClient::GetInstance().GetDefaultNet();
    context->errorCode_ = (context->netHandle_.GetNetId() == 0) 
        ? NETMANAGER_ERR_INTERNAL_ERROR 
        : NETMANAGER_SUCCESS;
}
```

### 7.3 同步执行模式

```cpp
// 同步模式直接执行，不创建 async_work
napi_value ConnectionExec::ExecGetDefaultNet(napi_env env, napi_callback_info info) {
    auto context = std::make_unique<GetDefaultNetContext>(env, info);
    // 直接调用内部接口
    NetHandle netHandle = NetConnClient::GetInstance().GetDefaultNet();
    // 返回结果
    return CreateNetHandleJsObject(env, netHandle);
}
```

---

## 8. 上下文 (Context) 模式

### 8.1 Context 职责

每个 N-API 方法都有对应的 Context 类，负责：
1. **参数解析**: 从 JS 参数提取 C++ 数据
2. **参数校验**: 类型检查、范围检查、空值检查
3. **错误处理**: 记录错误信息
4. **结果存储**: 存储执行结果

### 8.2 Context 文件组织

```
frameworks/js/napi/connection/async_context/
├── include/
│   ├── getdefaultnet_context.h
│   ├── setglobalhttpproxy_context.h
│   ├── bindsocket_context.h
│   └── ... (30+ 个 context)
└── src/
    ├── getdefaultnet_context.cpp
    ├── setglobalhttpproxy_context.cpp
    └── ...
```

### 8.3 Context 示例

```cpp
// frameworks/js/napi/connection/async_context/include/getdefaultnet_context.h
class GetDefaultNetContext : public BaseContext {
public:
    GetDefaultNetContext(napi_env env, napi_callback_info info);
    
    void ParseParams() override;  // 解析参数
    
    NetHandle netHandle_;         // 输出结果
    int32_t errorCode_;
};

// 参数解析实现
void GetDefaultNetContext::ParseParams() {
    // 检查参数个数
    if (paramsCount != PARAM_COUNT) {
        SetErrorCode(NETMANAGER_ERR_PARAMETER_ERROR);
        SetErrorMessage("Invalid parameter count");
        return;
    }
    // 解析 callback 或创建 Promise
    ParseCallbackOrPromise();
}
```

---

## 9. 权限声明

### 9.1 权限列表

| 权限 | 英文 | 说明 | 使用 API |
|------|------|------|----------|
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | 查询网络状态 | getDefaultNet, hasDefaultNet |
| `ohos.permission.SET_NETWORK_INFO` | 设置网络信息 | 配置网络 | setGlobalHttpProxy |
| `ohos.permission.MANAGE_NET_STRATEGY` | 管理网络策略 | 策略管理 | setPolicyByUid |
| `ohos.permission.GET_NETWORK_STATS` | 获取网络统计 | 流量查询 | getUidRxBytes |
| `ohos.permission.CONNECTIVITY_INTERNAL` | 内部连接权限 | 系统级操作 | setInterfaceUp |

### 9.2 权限检查实现

```cpp
// utils/common_utils/src/netmanager_base_permission.cpp:33
int32_t NetManagerPermission::CheckPermission(const std::string &permission) {
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    auto ret = AccessTokenKit::VerifyAccessToken(callerToken, permission);
    return (ret == PERMISSION_GRANTED) ? NETMANAGER_SUCCESS : NETMANAGER_ERROR;
}
```

---

## 10. 调用链完整示例

### 10.1 JS → Kernel 完整调用链

```
【JS 层】
net.connection.getDefaultNet()
  ↓
【N-API 层】frameworks/js/napi/connection/connection_module/src/connection_module.cpp
ConnectionModule::GetDefaultNet()
  ├─ ParseParams()  → GetDefaultNetContext::ParseParams()
  └─ ModuleTemplate::Interface<GetDefaultNetContext>()
        ↓
【Async Work】frameworks/js/napi/connection/async_work/src/connection_async_work.cpp
ConnectionAsyncWork::ExecGetDefaultNet()
  ↓
【Inner API】frameworks/native/netconnclient/src/net_conn_client.cpp
NetConnClient::GetDefaultNet()
  ↓
【IPC Client】frameworks/native/netconnclient/src/proxy/net_conn_service_proxy.cpp
NetConnServiceProxy::GetDefaultNet()
  → Remote()->SendRequest(ConnInterfaceCode::CMD_NM_GET_DEFAULT_NET)
        ↓
【IPC Server】services/netconnmanager/src/stub/net_conn_service_stub.cpp
NetConnServiceStub::OnRemoteRequest()
  → NetConnServiceStub::OnGetDefaultNet()
        ↓
【Service】services/netconnmanager/src/net_conn_service.cpp
NetConnService::GetDefaultNet()
  → 返回 defaultNetSupplier_->GetNetId()
        ↓
【返回】NetHandle → napi_value → JS Object
```

---

*生成时间: 2025-02-06*
