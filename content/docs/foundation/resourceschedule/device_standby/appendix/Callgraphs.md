# 关键调用链

## 概述

本文档描述 Device Standby 部件的关键调用链路，从入口到核心逻辑的完整流程。

## 1. 豁免申请调用链

### JS/N-API 调用链

```
JS 应用
    │
    ▼
interfaces/kits/napi/src/init.cpp:31
    │
    ▼
napi_function: ApplyAllowResource
    │
    ▼
interfaces/kits/napi/src/standby_napi_module.cpp:283
    │
    ▼
StandbyServiceClient::GetInstance().ApplyAllowResource()
    │
    ▼
services/core/src/standby_service.cpp:231
    │
    ▼
StandbyServiceImpl::GetInstance().ApplyAllowResource()
    │
    ├──► CheckCallerPermission()       // 权限校验
    ├──► CheckAllowTypeInfo()          // 参数校验
    └──► UpdateRecord()                // 更新豁免记录
```

### 关键代码路径

| 阶段 | 文件:行号 | 说明 |
|------|----------|------|
| N-API 入口 | `standby_napi_module.cpp:283` | ApplyAllowResource |
| 客户端代理 | `standby_service_client.cpp:68` | StandbyServiceClient |
| 服务入口 | `standby_service.cpp:223` | StandbyService |
| 服务实现 | `standby_service_impl.cpp:743` | StandbyServiceImpl |
| 权限校验 | `standby_service_impl.cpp:627-641` | CheckCallerPermission |
| 参数校验 | `standby_service_impl.cpp:759-765` | CheckAllowTypeInfo |
| 记录更新 | `standby_service_impl.cpp:772-808` | UpdateRecord |

## 2. 设备待机查询调用链

```
JS 应用
    │
    ▼
napi_function: IsDeviceInStandby
    │
    ▼
interfaces/kits/napi/src/standby_napi_module.cpp:98
    │
    ▼
napi_create_async_work()  // 异步工作
    │
    ▼
StandbyServiceClient::GetInstance().IsDeviceInStandby()
    │
    ▼
IPC 调用 (Stub → Proxy)
    │
    ▼
StandbyServiceImpl::GetInstance().IsDeviceInStandby()
    │
    ▼
查询当前状态机状态
```

## 3. 状态机切换调用链

```
事件源 (用户交互/定时器/传感器)
    │
    ▼
StandbyServiceImpl::HandleEvent()
    │
    ▼
StandbyServiceImpl::DispatchEvent()
    │
    ▼
plugins/standby_state/src/state_manager_adapter.cpp
    │
    ▼
StateManagerAdapter::CheckStateTransition()
    │
    ├──► 旧状态 OnExit()
    └──► 新状态 OnEnter()
    │
    ▼
StandbyStateSubscriber::NotifyStateChanged()
    │
    ▼
IStandbyServiceSubscriber::OnStandbyStateChanged()
```

## 4. 插件注册调用链

```
StandbyServiceImpl::Init()
    │
    ▼
StandbyServiceImpl::RegisterPlugin()
    │
    ▼
Plugin Manager
    │
    ├──► RegisterPluginInner(ConstraintManager)
    ├──► RegisterPluginInner(ListenerManager)
    ├──► RegisterPluginInner(StrategyManager)
    └──► RegisterPluginInner(StateManager)
```

## 5. IPC 通信调用链

### 客户端 → 服务

```
StandbyServiceClient
    │
    ▼
GetStandbyServiceProxy()  // 获取 SA 代理
    │
    ▼
IPCSkeleton::GetContextObject()
    │
    ▼
Stub ↔ Proxy 序列化/反序列化
    │
    ▼
StandbyServiceImpl::OnRemoteRequest()
    │
    ▼
分发到具体方法
```

### 服务 → 订阅者

```
StandbyServiceImpl
    │
    ▼
StandbyStateSubscriber::ReportXXX()
    │
    ▼
IRemoteProxy::SendRequest()
    │
    ▼
订阅者回调
```

## 6. 权限校验调用链

```
ApplyAllowResource()
    │
    ▼
CheckCallerPermission(reasonCode)
    │
    ▼
IPCSkeleton::GetCallingTokenID()
    │
    ▼
AccessTokenKit::GetTokenType(tokenId)
    │
    ├──► TOKEN_HAP
    │        │
    │        ▼
    │    AccessTokenKit::VerifyAccessToken(tokenId, PERMISSION)
    │
    └──► TOKEN_NATIVE
             │
             ▼
         CheckNativePermission()
```

## 关键数据结构传递

```
ResourceRequest
    ├── allowType_: uint32_t
    ├── uid_: int32_t
    ├── name_: std::string
    ├── duration_: int32_t
    ├── reason_: std::string
    └── reasonCode_: uint32_t

AllowRecord
    ├── uid_: int32_t
    ├── pid_: int32_t
    ├── name_: std::string
    ├── allowType_: uint32_t
    ├── allowTimeList_: std::vector<AllowTimeInfo>
    └── reasonCode_: uint32_t
```
