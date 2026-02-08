# Callgraphs - 关键调用链

> 重要流程的调用链图示

---

## 1. 服务启动调用链

```
系统启动
    ↓
MakeAndRegisterAbility(CompanionDeviceAuthService)
    ↓
CompanionDeviceAuthService::OnStart()
    ├── Init() - 初始化日志
    ├── RegisterAdapters() - 注册适配器
    ├── SingletonManager::GetInstance() - 获取单例
    │   ├── CompanionManager::GetInstance()
    │   ├── HostBindingManager::GetInstance()
    │   ├── CrossDeviceCommManager::GetInstance()
    │   └── SecurityAgentImpl::Create()
    │       └── Rust FFI Init
    ├── SubscribeSystemAbility() - 订阅依赖SA
    └── Publish() - 发布服务
```

---

## 2. getStatusMonitor调用链

```
JS: getStatusMonitor(localUserId)
    ↓
NAPI: GetStatusMonitor()
    ├── CheckUseUserIdmPermission()
    │   └── AccessTokenKit::VerifyAccessToken()
    ├── CheckCallerIsSystemApp()
    │   └── TokenIdKit::IsSystemAppByFullTokenID()
    ├── StatusMonitorClass() - 创建StatusMonitor类
    ├── napi_new_instance() - 创建实例
    ├── napi_wrap() - 包装C++对象
    └── statusMonitor->SetLocalUserId()
        └── Client: GetStatusMonitor()
            └── IPC: GetServiceProxy()
                └── SystemAbilityManager::CheckSystemAbility(945)
```

---

## 3. 添加伴随设备调用链

```
主设备侧:
    ↓
User操作: 开始添加设备
    ↓
HostAddCompanionRequest::Start()
    ├── SecurityAgent::HostBeginAddCompanion()
    │   └── Rust FFI: host_begin_add_companion()
    │       └── 生成密钥协商请求
    └── SoftBusChannel::SendMessage()
        └── 发送给伴随设备

伴随设备侧:
    ↓
SoftBusChannel::OnMessageReceived()
    ↓
MessageRouter::Route()
    ↓
CompanionInitKeyNegotiationHandler::Handle()
    ├── SecurityAgent::CompanionInitKeyNegotiation()
    │   └── Rust FFI: companion_init_key_negotiation()
    └── 发送响应回主设备

主设备侧(响应):
    ↓
HostEndAddCompanion::Handle()
    ├── SecurityAgent::HostEndAddCompanion()
    │   └── Rust FFI: 完成绑定，生成凭证
    └── CompanionManager::AddCompanion()
        └── 存储模板信息
```

---

## 4. Token认证调用链

```
认证触发:
    ↓
UserAuthFramework::BeginAuthentication()
    ↓
CompanionDeviceAuthDriver::OnBeginExecute()
    ↓
HostTokenAuthRequest::Start()
    ├── SecurityAgent::HostBeginTokenAuth()
    │   └── Rust FFI: host_begin_token_auth()
    │       └── 生成认证Token
    └── SoftBusChannel::SendMessage()
        └── 发送Token给伴随设备

伴随设备验证:
    ↓
CompanionTokenAuthHandler::Handle()
    ├── SecurityAgent::CompanionProcessTokenAuth()
    │   └── Rust FFI: companion_process_token_auth()
    │       ├── 验证Token签名
    │       └── 确认用户身份(佩戴检测等)
    └── 返回认证结果

主设备完成:
    ↓
HostEndTokenAuth::Handle()
    ├── SecurityAgent::HostEndTokenAuth()
    └── 通知UserAuthFramework结果
```

---

## 5. 回调通知调用链

```
服务层状态变化:
    ↓
CompanionManager::UpdateStatus()
    ↓
SubscriptionManager::NotifyTemplateStatusChange()
    ↓
IIpcTemplateStatusCallback::OnTemplateStatusChange()
    ↓
IPC回调到客户端:
        ↓
    IpcTemplateStatusCallbackProxy::OnTemplateStatusChange()
        ↓
    客户端进程:
        ↓
    IpcTemplateStatusCallbackService::OnTemplateStatusChange()
        ↓
    NapiTemplateStatusCallback::OnTemplateStatusChange()
        ├── napi_get_uv_event_loop()
        └── napi_send_event()
            ↓
    JS线程:
        ↓
    执行JS回调函数
```

---

## 6. 权限检查调用链

```
API调用入口:
    ↓
CompanionDeviceAuthService::Subscribe*() / Unsubscribe*() / Get*() / Update*()
    ↓
CheckPermission()
    ├── AccessTokenKitAdapter::CheckPermission()
    │   ├── stub.GetCallingTokenID()
    │   └── AccessTokenKit::VerifyAccessToken(tokenId, USE_USER_IDM_PERMISSION)
    └── AccessTokenKitAdapter::CheckSystemPermission()
        ├── IPCSkeleton::GetCallingFullTokenID()
        ├── TokenIdKit::IsSystemAppByFullTokenID()
        └── AccessTokenKit::GetTokenTypeFlag() == TOKEN_HAP
```

---

## 7. 请求生命周期调用链

```
请求创建:
    ↓
RequestFactory::CreateRequest(RequestType type)
    ↓
RequestManager::AddRequest()
    └── 生成requestId

请求执行:
    ↓
Request::Start()
    ├── 执行具体业务逻辑
    └── TaskRunner::PostTask()

请求完成/取消:
    ↓
Request::OnComplete() / Request::Cancel()
    ↓
RequestManager::RemoveRequest()
    └── 清理资源
```

---

## 8. 跨设备消息路由调用链

```
消息接收:
    ↓
SoftBusChannel::OnDataReceived()
    ↓
ConnectionManager::DispatchMessage()
    ↓
MessageRouter::Route()
    ├── 解析消息类型
    ├── 查找对应Handler
    └── Handler::Handle()
        ├── add_companion handlers
        ├── token_auth handlers
        ├── delegate_auth handlers
        └── ...

消息发送:
    ↓
OutboundRequest::Send()
    ↓
CrossDeviceCommManager::SendMessage()
    ↓
SoftBusChannel::Send()
    └── SoftBus::SendMsg()
```

---

*文档生成时间: 2025-02-06*
