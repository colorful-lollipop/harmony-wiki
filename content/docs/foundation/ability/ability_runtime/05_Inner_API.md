# Inner API（内部组件间接口）

## 概述

Inner API（InnerKit）是 ability_runtime 内部组件间使用的编程接口，仅限系统组件使用，不对普通应用开放。这些接口通过 IPC 机制在进程间通信。

## Inner API 清单

### 1. AbilityManager 接口

**头文件目录**：`interfaces/inner_api/ability_manager/`

| 接口文件 | 说明 |
|---------|------|
| `ability_manager_client.h` | AbilityManager 客户端 |
| `ability_manager_stub.h` | AbilityManager 服务端存根 |
| `ability_connect_callback_stub.h` | 连接回调存根 |
| `launch_param.h` | 启动参数 |
| `mission_info.h` | 任务信息 |
| `mission_snapshot.h` | 任务快照 |
| `start_params_by_SCB.h` | SCB 启动参数 |

**主要类**：

```cpp
// ability_manager_client.h
class AbilityManagerClient {
public:
    static AbilityManagerClient &GetInstance();
    
    int32_t StartAbility(const Want &want, int32_t userId = -1, 
                         int requestCode = -1, const StartOptions &options = {});
    
    int32_t TerminateAbility(const sptr<IRemoteObject> &token);
    
    int32_t ConnectAbility(const Want &want, const sptr<IAbilityConnection> &connect,
                           const sptr<IRemoteObject> &callerToken, int32_t userId = -1);
    
    int32_t DisconnectAbility(const sptr<IAbilityConnection> &connect);
    
    // ... 更多方法
};
```

**代码证据**：`interfaces/inner_api/ability_manager/include/ability_manager_client.h`

### 2. AppManager 接口

**头文件目录**：`interfaces/inner_api/app_manager/`

| 接口文件 | 说明 |
|---------|------|
| `app_mgr_client.h` | AppManager 客户端 |
| `page_state_data.h` | 页面状态数据 |
| `app_state_data.h` | 应用状态数据 |

**主要类**：

```cpp
// app_mgr_client.h
class AppMgrClient {
public:
    static AppMgrClient &GetInstance();
    
    int32_t GetAppRunningInfoByBundleName(const std::string &bundleName,
                                          std::vector<AppRunningInfo> &infos);
    
    int32_t RegisterApplicationStateObserver(const sptr<IApplicationStateObserver> &observer);
    
    // ... 更多方法
};
```

**代码证据**：`interfaces/inner_api/app_manager/include/appmgr/app_mgr_client.h`

### 3. ExtensionManager 接口

**头文件目录**：`interfaces/inner_api/extension_manager/`

```cpp
// extension_manager_client.h
class ExtensionManagerClient {
public:
    static ExtensionManagerClient &GetInstance();
    
    int32_t CreateExtensionContext(const std::string &extensionAbilityName,
                                   const std::shared_ptr<AbilityContext> &owner);
    
    // ... 更多方法
};
```

### 4. MissionManager 接口

**头文件目录**：`interfaces/inner_api/mission_manager/`

```cpp
// mission_manager_client.h
class MissionManagerClient {
public:
    static MissionManagerClient &GetInstance();
    
    int32_t RegisterMissionListener(const sptr<IMissionListener> &listener);
    
    int32_t GetMissionInfo(int32_t missionId, MissionInfo &missionInfo);
    
    // ... 更多方法
};
```

### 5. UriPermissionManager 接口

**头文件目录**：`interfaces/inner_api/uri_permission/`

```cpp
// uri_permission_manager_client.h
class UriPermissionManagerClient {
public:
    static UriPermissionManagerClient &GetInstance();
    
    int32_t GrantUriPermission(const std::string &uri, unsigned int flag,
                               const std::string &bundleName);
    
    int32_t RevokeUriPermission(const std::string &uri, const std::string &bundleName);
    
    // ... 更多方法
};
```

### 6. QuickFixManager 接口

**头文件目录**：`interfaces/inner_api/quick_fix/`

```cpp
// quick_fix_manager_client.h
class QuickFixManagerClient {
public:
    static QuickFixManagerClient &GetInstance();
    
    int32_t ApplyQuickFix(const std::vector<std::string> &quickFixFiles);
    
    int32_t GetQuickFixInfo(const std::string &bundleName, QuickFixInfo &quickFixInfo);
    
    // ... 更多方法
};
```

### 7. Runtime 接口

**头文件目录**：`interfaces/inner_api/runtime/`

```cpp
// js_runtime.h
class JsRuntime {
public:
    static std::shared_ptr<JsRuntime> Create(const RuntimeOption &option);
    
    bool LoadModule(const std::string &modulePath, const std::string &moduleName);
    
    bool RunScript(const std::string &filePath);
    
    // ... 更多方法
};
```

### 8. 错误码工具

**头文件目录**：`interfaces/inner_api/error_utils/`

```cpp
// ability_runtime_error_util.h
class AbilityRuntimeErrorUtil {
public:
    static int32_t ConvertToAbilityRuntimeError(int32_t systemErrorCode);
    
    static std::string GetErrorMessage(int32_t errorCode);
};
```

## 权限验证 API

**头文件**：`services/common/include/permission_verification.h`

```cpp
class PermissionVerification {
public:
    bool VerifyCallingPermission(const std::string &permissionName, 
                                 const uint32_t specifyTokenId = 0);
    
    bool IsSACall() const;                    // 是否系统能力调用
    bool IsShellCall() const;                 // 是否 Shell 调用
    bool IsSystemAppCall() const;             // 是否系统应用调用
    
    bool VerifyMissionPermission() const;     // 任务管理权限
    bool VerifyAccountPermission() const;     // 账户权限
    
    int CheckCallServiceAbilityPermission(const VerificationInfo &verificationInfo,
                                          uint32_t specifyTokenId = 0);
    
    int CheckCallDataAbilityPermission(const VerificationInfo &verificationInfo,
                                       bool isShell);
    
    // ... 更多权限校验方法
};
```

**代码证据**：`services/common/include/permission_verification.h`

## 稳定性标注

### 接口稳定性分类

| 标注 | 含义 | 使用建议 |
|------|------|---------|
| **SysCap** | 系统能力接口，稳定 | 可直接使用 |
| **InnerKit** | 内部组件接口，稳定 | 系统组件使用 |
| **Internal** | 内部实现，可能变化 | 避免使用 |

### 接口稳定性证据

通过以下方式判断接口稳定性：

1. **头文件位置**：inner_api > kits > frameworks/native
2. **命名约定**：
   - `*_stub.h`, `*_proxy.h`：IPC 接口，稳定
   - `*_client.h`：客户端接口，稳定
   - 内部 `impl.h`, `inner*.h`：内部实现，可能变化

## 使用示例

### 进程内调用

```cpp
// 同一进程内获取 AbilityManagerClient
auto &abilityMgrClient = AbilityManagerClient::GetInstance();
int32_t result = abilityMgrClient.StartAbility(want);

// 获取 AppManagerClient
auto &appMgrClient = AppMgrClient::GetInstance();
std::vector<AppRunningInfo> infos;
appMgrClient.GetAppRunningInfoByBundleName("com.example.app", infos);
```

### 跨进程调用

```cpp
// 通过 IPC 调用 AbilityManagerService
sptr<ISystemAbilityManager> samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
sptr<IRemoteObject> amsObj = samgr->GetSystemAbility(ABILITY_MGR_SERVICE_ID);
sptr<IAbilityManager> abilityMgr = iface_cast<IAbilityManager>(amsObj);

Want want;
want.SetElementName("com.example.app", "EntryAbility");
abilityMgr->StartAbility(want);
```

## 相关文档

- [N-API 参考](04_NAPI_Reference.md)
- [架构说明](03_Architecture.md)
- [安全风险评审](08_Security_Review.md)
