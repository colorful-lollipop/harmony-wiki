# 附录：调用链

## 目的

本文档提供关键操作的完整调用链，帮助理解从JS API到系统服务的执行路径。

## 适用范围

- 目标读者：开发者、调试工程师
- 覆盖内容：管理员激活、策略设置、插件执行等关键流程
- 不包含：所有可能的调用路径

## 关键结论

- 所有关键操作都经过多层：N-API → Proxy → IPC → Service → Plugin
- 每层都有明确的职责和错误处理机制
- 策略执行最终调用具体插件实现

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 完整架构说明
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - N-API接口定义

---

## 调用链1: 启用管理员

### 流程图

```mermaid
sequenceDiagram
    participant App as MDM应用(JS)
    participant NAPI as NAPI层
    participant Proxy as Proxy层
    participant IPC as IPC(跨进程)
    participant SA as EDM服务
    participant Perm as PermissionChecker
    participant AdminMgr as AdminManager
    participant PolicyMgr as PolicyManager
    participant RDB as RDB存储
    participant Conn as ConnectionManager
    participant Ext as 管理员Extension

    App->>NAPI: enableAdmin(want, entInfo, type, userId)
    NAPI->>NAPI: 解析参数和校验
    NAPI->>Proxy: LoadAndGetEdmService()
    Proxy->>IPC: SendRequest(ADD_DEVICE_ADMIN)
    IPC->>SA: OnRemoteRequest(code=1)
    SA->>Perm: CheckCallerPermission(bundleName, MANAGE_ENTERPRISE_DEVICE_ADMIN)
    alt 权限失败
        SA-->>IPC: ReturnError(ERR_EDM_PERMISSION_ERROR)
        IPC-->>Proxy: ReturnError
        Proxy-->>NAPI: ErrorCallback
        NAPI-->>App: ThrowError
    else 权限通过
        SA->>Perm: VerifyEnableAdminCondition(want, type, userId)
        alt 激活条件不满足
            SA->>AdminMgr: 返回ENABLE_ADMIN_FAILED
            SA-->>IPC: ReturnError
        else 条件满足
            SA->>Perm: CheckCallingUid(bundleName)
            SA->>AdminMgr: GetAdminInfo(bundleName)
            alt 管理员不存在
                SA-->>IPC: ReturnError(ADMIN_INACTIVE)
            else 管理员已存在
                SA->>AdminMgr: 获取权限列表
                SA->>AdminMgr: GetPermissionsByAdmin(bundleName, userId)
                SA->>PolicyMgr: SetAdminValue(userId, adminInfo)
                SA->>Conn: ConnectAbility(want)
                alt 连接失败
                    SA-->>IPC: ReturnError
                else 连接成功
                    SA-->>IPC: ReturnSuccess
```

### 关键函数调用

| 层级 | 函数 | 文件 | 行号 |
|------|------|------|------|
| NAPI | EnableAdmin | interfaces/kits/admin_manager/src/admin_manager_addon.cpp | 42 |
| NAPI | NativeEnableAdmin | interfaces/kits/admin_manager/src/admin_manager_addon.cpp | - |
| Proxy | EnableAdmin | interfaces/inner_api/common/src/enterprise_device_mgr_proxy.cpp | - |
| SA | EnableAdmin | services/edm/src/enterprise_device_mgr_ability.cpp | 1585 |
| SA | VerifyEnableAdminCondition | services/edm/src/admin_manager.cpp | 1351 |
| SA | CheckCallingUid | services/edm/src/permission_checker.cpp | 194 |
| SA | GetPermissionsByAdmin | services/edm/src/permission_checker.cpp | - |
| SA | SetAdminValue | services/edm/src/admin_manager.cpp | - |
| SA | ConnectAbility | services/edm/src/connection/enterprise_conn_manager.cpp | - |

---

## 调用链2: 设置策略

### 流程图

```mermaid
sequenceDiagram
    participant App as MDM应用(JS)
    participant NAPI as NAPI层
    participant Proxy as Proxy层
    participant IPC as IPC(跨进程)
    participant SA as EDM服务
    participant Perm as PermissionChecker
    participant PluginMgr as PluginManager
    participant Plugin as IPlugin实现
    participant RDB as RDB存储

    App->>NAPI: setPolicy(policyName, policyData)
    NAPI->>NAPI: 解析参数
    NAPI->>Proxy: LoadAndGetEdmService()
    Proxy->>IPC: SendRequest(SET_DATETIME, etc.)
    IPC->>SA: OnRemoteRequest(funcCode)
    SA->>Perm: CheckCallerPermission(bundleName, ENTERPRISE_SET_DATETIME)
    alt 权限失败
        SA-->>IPC: ReturnError(ERR_EDM_PERMISSION_ERROR)
        IPC-->>Proxy: ReturnError
        Proxy-->>NAPI: ErrorCallback
        NAPI-->>App: ThrowError
    else 权限通过
        SA->>Perm: CheckCallingUid(bundleName)
        SA->>Perm: CheckHandlePolicyPermission(SET, admin, policyName)
        alt 权限不足
            SA-->>IPC: ReturnError(PERMISSION_DENIED)
            IPC-->>Proxy: ReturnError
            Proxy-->>NAPI: ErrorCallback
            NAPI-->>App: ThrowError
        else 权限充足
            SA->>PluginMgr: HandlePolicy(funcCode, data, reply, userId)
            PluginMgr->>PluginMgr: GetPluginByFuncCode(funcCode)
            PluginMgr->>PluginMgr: LoadPluginByFuncCode(funcCode) [首次]
            alt 插件未加载
                PluginMgr->>DL: dlopen(plugin.so)
                PluginMgr->>PluginMgr: AddPlugin(funcCode, plugin)
            else 插件已加载
                PluginMgr->>PluginMgr: 直接调用已加载插件
            PluginMgr->>Plugin: OnHandlePolicy(funcCode, data, reply, userId)
            Plugin->>Plugin: Deserialize policyData
            Plugin->>Plugin: OnGetPolicy() [查询策略]
            Plugin->>Plugin: 合并策略数据(Merge)
            Plugin->>RDB: QueryAndMerge(mergedData)
            Plugin->>Plugin: OnHandlePolicyDone(funcCode, adminName, changed, userId)
            Plugin->>Plugin: Save to RDB(PolicyManager)
            Plugin-->>IPC: ReturnSuccess
            IPC-->>Proxy: ReturnSuccess
            Proxy-->>NAPI: SuccessCallback
            NAPI-->>App: Return(result)
```

### 关键函数调用

| 层级 | 函数 | 文件 | 说明 |
|------|------|------|------|
| NAPI | SetPowerPolicy | interfaces/kits/device_settings/src/device_settings_addon.cpp | 97 |
| Proxy | HandleDevicePolicy | interfaces/inner_api/common/src/enterprise_device_mgr_proxy.cpp | - |
| SA | HandleDevicePolicy | services/edm/src/enterprise_device_mgr_ability.cpp | 2060 |
| SA | CheckHandlePolicyPermission | services/edm/src/permission_checker.cpp | 249 |
| PluginMgr | HandlePolicy | services/edm/src/plugin_manager.cpp | - |
| PluginMgr | GetPluginByFuncCode | services/edm/src/plugin_manager.cpp | - |
| PluginMgr | LoadPluginByFuncCode | services/edm/src/plugin_manager.cpp | - |
| Plugin | OnHandlePolicy | services/edm_plugin/src/*_plugin.cpp | 各插件实现 |
| Plugin | GetOthersMergePolicyData | services/edm_plugin/src/*_plugin.cpp | 各插件实现 |
| Plugin | OnHandlePolicyDone | services/edm_plugin/src/*_plugin.cpp | 各插件实现 |
| PolicyMgr | SetPolicy | services/edm/src/policy_manager.cpp | - |
| PolicyMgr | GetPolicy | services/edm/src/policy_manager.cpp | - |
| RDB | Insert | services/edm/src/database/edm_rdb_data_manager.cpp | - |

---

## 调用链3: 查询策略

### 流程图

```mermaid
sequenceDiagram
    participant App as MDM应用(JS)
    participant NAPI as NAPI层
    participant Proxy as Proxy层
    participant IPC as IPC(跨进程)
    participant SA as EDM服务
    participant Perm as PermissionChecker
    participant PluginMgr as PluginManager
    participant Plugin as IPlugin实现
    participant Query as 查询策略

    App->>NAPI: getPolicy(policyName)
    NAPI->>NAPI: 解析参数
    NAPI->>Proxy: LoadAndGetEdmService()
    Proxy->>IPC: SendRequest(funcCode)
    IPC->>SA: OnRemoteRequest(funcCode)
    SA->>Perm: CheckCallerPermission(bundleName, ENTERPRISE_GET_SETTINGS)
    alt 权限失败
        SA-->>IPC: ReturnError
        IPC-->>Proxy: ReturnError
        Proxy-->>NAPI: ErrorCallback
        NAPI-->>App: ThrowError
    else 权限通过
        SA->>PluginMgr: GetPolicy(funcCode, reply, userId)
        PluginMgr->>PluginMgr: GetPluginByFuncCode(funcCode)
        PluginMgr->>PluginMgr: 直接调用已加载插件
        PluginMgr->>PluginMgr: GetPolicy(funcCode, data, reply, userId)
        Plugin->>Query: QueryPolicy(pluginName) [调用查询策略]
            Query->>RDB: Query from database
            RDB-->>Query: Return policyData
            Query-->>Plugin: Return(policyData)
        Plugin-->>IPC: ReturnSuccess
        IPC-->>Proxy: ReturnSuccess
        Proxy-->>NAPI: SuccessCallback
        NAPI-->>App: Return(policyValue)
```

### 关键函数调用

| 层级 | 函数 | 文件 | 说明 |
|------|------|------|------|
| NAPI | GetPowerPolicy | interfaces/kits/device_settings/src/device_settings_addon.cpp | - |
| Proxy | GetDevicePolicy | interfaces/inner_api/common/src/enterprise_device_mgr_proxy.cpp | - |
| SA | GetDevicePolicy | services/edm/src/enterprise_device_mgr_ability.cpp | 2126 |
| SA | CheckSystemCalling | services/edm/src/permission_checker.cpp | 210 |
| PluginMgr | GetPolicy | services/edm/src/plugin_manager.cpp | - |
| Plugin | OnGetPolicy | services/edm_plugin/src/*_plugin.cpp | 各插件实现 |
| Query | GetPolicy | services/edm/include/query_policy/*_query.cpp | 各查询实现 |
| RDB | Query | services/edm/src/database/edm_rdb_data_manager.cpp | - |

---

## 调用链4: 插件动态加载

### 流程图

```mermaid
sequenceDiagram
    participant SA as EDM服务
    participant PluginMgr as PluginManager
    participant FuncCode as 功能码映射
    participant DL as dlopen
    participant SO as 插件SO
    participant Plugin as IPlugin实现

    SA->>PluginMgr: HandlePolicy(funcCode=xxx)
    PluginMgr->>FuncCode: GetSoNameByCode(funcCode)
    alt 插件未注册
        FuncCode-->>PluginMgr: 映射不存在
        PluginMgr->>FuncCode: GetSoNameByCode(code) [使用默认映射]
        FuncCode-->>PluginMgr: 返回soName
    PluginMgr->>PluginMgr: pluginsCode_.count(plugin) < 1 ?
    alt 首次加载
        PluginMgr->>DL: dlopen(soPath)
        DL->>DL: dlsym("CreatePlugin")
        DL->>SO: 返回Plugin对象
        SO->>Plugin: OnHandlePolicy() [首次调用初始化]
        Plugin->>PluginMgr: AddPlugin(funcCode, plugin)
    else 已加载
        PluginMgr->>PluginMgr: pluginsCode_.count(plugin) >= 1
        PluginMgr->>PluginMgr: 直接调用插件
```

### 关键函数调用

| 层级 | 函数 | 文件 | 说明 |
|------|------|------|------|
| PluginMgr | HandlePolicy | services/edm/src/plugin_manager.cpp | - |
| PluginMgr | GetPluginByFuncCode | services/edm/src/plugin_manager.cpp | - |
| PluginMgr | LoadPluginByFuncCode | services/edm/src/plugin_manager.cpp | - |
| Plugin | AddPlugin | services/edm_plugin/src/*_plugin.cpp | 插件自注册 |
| Plugin | CreatePlugin | services/edm_plugin/src/*_plugin.cpp | 插件构造函数 |

---

## 调用链5: 权限检查流程

### 流程图

```mermaid
sequenceDiagram
    participant IPC as IPC(跨进程)
    participant Perm as PermissionChecker
    participant Token as AccessTokenKit
    participant Bundle as BundleManager
    participant SA as EDM服务

    IPC->>Perm: CheckCallerPermission(bundleName, permission)
    alt Token验证失败
        Perm-->>IPC: Return(false)
        IPC-->>SA: ReturnError(ERR_EDM_PERMISSION_ERROR)
    else Token验证通过
        Perm->>Token: GetCallingTokenID()
        Token-->>Perm: Return(tokenId)
        Perm->>Token: GetTokenTypeFlag(tokenId)
        alt 不是系统应用
            Perm->>Token: VerifyAccessToken(tokenId, permission)
            Token-->>Perm: Return(granted?)
            Perm-->>IPC: Return(false)
            IPC-->>SA: ReturnError(SYSTEM_API_DENIED)
        else 是系统应用
            Perm->>Token: IsSystemAppByFullTokenID(IPCSkeleton::GetCallingFullTokenID())
            alt 不是系统应用
                Perm-->>IPC: Return(false)
                IPC-->>SA: ReturnError(SYSTEM_API_DENIED)
            else 是系统应用
                Perm->>Bundle: GetNameForUid(uid, bundleName)
                alt Bundle名称不匹配
                    Perm-->>IPC: Return(false)
                    IPC-->>SA: ReturnError(ERR_EDM_PERMISSION_ERROR)
                else Bundle名称匹配
                    Perm-->>IPC: Return(true)
                    IPC-->>SA: ReturnSuccess
```

### 关键函数调用

| 层级 | 函数 | 文件 | 说明 |
|------|------|------|------|
| Perm | CheckCallerPermission | services/edm/src/permission_checker.cpp | 180 |
| Perm | VerifyCallingPermission | services/edm/src/permission_checker.cpp | 368 |
| Perm | IsSystemAppOrNative | common/external/src/edm_access_token_manager_impl.cpp | 41 |

---

## 调用链说明

### N-API异步工作模式

所有N-API API都使用异步工作模式，通过以下步骤：

1. **参数解析**：NAPI层从JS参数中提取值
2. **参数校验**：检查参数类型、范围、合法性
3. **Proxy创建**：延迟创建Proxy对象，按需加载服务
4. **IPC调用**：跨进程发送请求到EDM服务
5. **服务执行**：EDM服务处理请求，可能涉及：
   - 权限检查
   - 管理员查询
   - 策略处理
   - 插件执行
   - 数据库操作
6. **回调执行**：通过Promise或Callback返回结果给JS

### 错误传播机制

```
NAPI层 → 返回错误码 + 错误消息
    ↓
Proxy层 → 解析错误，转换为N-API错误对象
    ↓
JS应用 → 捕获异常，显示错误信息
```

---

## 相关跳转

- [04_External_API_NAPI.md](04_External_API_NAPI.md) - 完整API定义
- [03_Architecture.md](03_Architecture.md) - 架构和数据流
- [09_Troubleshooting.md](09_Troubleshooting.md) - 错误处理和调试
