# 03_Inner_API - 内部API文档

> 模块接口、依赖方向与稳定性

---

## 1. 模块接口概览

### 1.1 接口分层

```
┌─────────────────────────────────────────────────────────────┐
│  接口层 (Framework)                                          │
│  - JS/ETS API (N-API/ANI)                                    │
│  - Native Client API                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (IPC)
┌─────────────────────────────────────────────────────────────┐
│  服务接口层 (Service Interface)                               │
│  - ICompanionDeviceAuth (IDL)                                │
│  - 回调接口 (IIpc*Callback)                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  内部模块接口 (Inner API)                                      │
│  - Manager接口                                               │
│  - Adapter接口                                               │
│  - SecurityAgent接口                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. IPC接口定义

### 2.1 ICompanionDeviceAuth

**文件**: `frameworks/native/ipc/idl/ICompanionDeviceAuth.idl`

**接口方法**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| SubscribeAvailableDeviceStatus | localUserId, callback | resultCode | 订阅可用设备状态 |
| UnsubscribeAvailableDeviceStatus | callback | resultCode | 取消订阅 |
| SubscribeTemplateStatusChange | localUserId, callback | resultCode | 订阅模板状态变化 |
| UnsubscribeTemplateStatusChange | callback | resultCode | 取消订阅 |
| SubscribeContinuousAuthStatusChange | param, callback | resultCode | 订阅持续认证状态 |
| UnsubscribeContinuousAuthStatusChange | callback | resultCode | 取消订阅 |
| GetTemplateStatus | localUserId | templateList, resultCode | 获取模板状态 |
| RegisterDeviceSelectCallback | callback | resultCode | 注册设备选择回调 |
| UnregisterDeviceSelectCallback | - | resultCode | 注销回调 |
| UpdateTemplateEnabledBusinessIds | templateId, businessIds | resultCode | 更新业务ID |
| CheckLocalUserIdValid | localUserId | isValid, resultCode | 检查用户ID |

### 2.2 回调接口

**IIpcTemplateStatusCallback**:
```idl
interface IIpcTemplateStatusCallback {
    void OnTemplateStatusChange(IpcTemplateStatus[] templateStatusList);
}
```

**IIpcAvailableDeviceStatusCallback**:
```idl
interface IIpcAvailableDeviceStatusCallback {
    void OnAvailableDeviceStatusChange(IpcDeviceStatus[] deviceStatusList);
}
```

**IIpcContinuousAuthStatusCallback**:
```idl
interface IIpcContinuousAuthStatusCallback {
    void OnContinuousAuthStatusChange(IpcContinuousAuthStatus status);
}
```

**IIpcDeviceSelectCallback**:
```idl
interface IIpcDeviceSelectCallback {
    void OnDeviceSelect(int selectPurpose, IIpcSetDeviceSelectResultCallback callback);
}
```

---

## 3. Manager接口

### 3.1 CompanionManager

**文件**: `services/singleton/inc/companion/companion_manager_impl.h`

**职责**: 伴随设备生命周期管理

**主要方法**:
```cpp
class CompanionManager {
public:
    ResultCode AddCompanion(const DeviceKey& deviceKey, ...);
    ResultCode RemoveCompanion(uint64_t templateId);
    ResultCode GetCompanionStatus(uint64_t templateId, ...);
    ResultCode UpdateCompanionStatus(uint64_t templateId, ...);
};
```

### 3.2 HostBindingManager

**文件**: `services/singleton/inc/host_binding/host_binding_manager_impl.h`

**职责**: 主设备绑定关系管理

**主要方法**:
```cpp
class HostBindingManager {
public:
    ResultCode AddHostBinding(const DeviceKey& hostDeviceKey, ...);
    ResultCode RemoveHostBinding(const DeviceKey& hostDeviceKey);
    ResultCode GetHostBindingStatus(...);
};
```

### 3.3 RequestManager

**文件**: `services/singleton/inc/request/request_manager_impl.h`

**职责**: 认证请求生命周期管理

**主要方法**:
```cpp
class RequestManager {
public:
    ResultCode CreateRequest(RequestType type, ...);
    ResultCode CancelRequest(uint32_t requestId);
    ResultCode GetRequestStatus(uint32_t requestId, ...);
};
```

---

## 4. Adapter接口

### 4.1 AccessTokenKitAdapter

**文件**: `services/external_adapters/access_token/src/access_token_kit_adapter_impl.cpp`

**职责**: 权限检查适配

**接口**:
```cpp
class AccessTokenKitAdapter {
public:
    bool CheckPermission(IPCObjectStub& stub, const std::string& permissionName);
    bool CheckSystemPermission(IPCObjectStub& stub);
    uint32_t GetAccessTokenId(IPCObjectStub& stub);
};
```

### 4.2 UserAuthAdapter

**文件**: `services/external_adapters/user_auth/src/user_auth_adapter_impl.cpp`

**职责**: UserAuth框架适配

**接口**:
```cpp
class UserAuthAdapter {
public:
    ResultCode BeginDelegateAuth(...);
    ResultCode CancelAuthentication(...);
};
```

### 4.3 SoftBusAdapter

**文件**: `services/cross_device_channels/soft_bus/src/soft_bus_adapter_impl.cpp`

**职责**: 软总线通道适配

**接口**:
```cpp
class SoftBusAdapter {
public:
    ResultCode SendMessage(const DeviceKey& deviceKey, const std::vector<uint8_t>& message);
    ResultCode RegisterChannelListener(...);
};
```

---

## 5. SecurityAgent接口

### 5.1 ISecurityAgent

**文件**: `services/singleton/inc/security_agent/security_agent.h`

**稳定性**: 核心安全边界，接口变更影响大

**主要方法分类**:

**框架交互**:
```cpp
ResultCode Init();
ResultCode SetActiveUser(const SetActiveUserInput& input);
ResultCode HostGetExecutorInfo(HostGetExecutorInfoOutput& output);
ResultCode HostOnRegisterFinish(const RegisterFinishInput& input);
```

**添加伴随设备**:
```cpp
ResultCode HostBeginAddCompanion(const HostBeginAddCompanionInput& input, ...);
ResultCode HostEndAddCompanion(const HostEndAddCompanionInput& input, ...);
ResultCode CompanionInitKeyNegotiation(const CompanionInitKeyNegotiationInput& input, ...);
```

**Token认证**:
```cpp
ResultCode HostBeginTokenAuth(const HostBeginTokenAuthInput& input, ...);
ResultCode HostEndTokenAuth(const HostEndTokenAuthInput& input, ...);
ResultCode CompanionProcessTokenAuth(const CompanionProcessTokenAuthInput& input, ...);
```

---

## 6. 模块依赖图

```
┌─────────────────────────────────────────────────────────────┐
│  frameworks/js/napi                                          │
│  └── depends on: frameworks/native/client                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/client                                    │
│  └── depends on: frameworks/native/ipc                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/ipc                                       │
│  └── depends on: common                                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (IPC)
┌─────────────────────────────────────────────────────────────┐
│  services/service_entry                                      │
│  ├── depends on: common                                     │
│  ├── depends on: singleton/*                                │
│  ├── depends on: cross_device_comm                          │
│  └── depends on: external_adapters/*                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. 接口稳定性标注

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| N-API/JS API | 稳定 | 对外公开API，向后兼容 |
| ICompanionDeviceAuth | 稳定 | IPC接口定义，变更需同步 |
| ISecurityAgent | 不稳定 | 内部接口，厂商可定制 |
| Manager接口 | 内部 | 不推荐外部依赖 |
| Adapter接口 | 内部 | 外部系统适配层 |

---

*文档生成时间: 2025-02-06*
