# appendix/Callgraphs.md - 关键调用链

## 概述

本文档描述 ecological_rule_manager 模块的关键调用链，包括从调用方到 SA 的完整路径。

## 调用链 1: QueryStartExperience

### 调用方 → Client SDK → SA

```mermaid
sequenceDiagram
    participant Caller as AbilityManagerService
    participant Client as ERMS Client
    participant Proxy as ERMS Proxy
    participant Stub as ERMS Stub
    participant Service as ERMS Service

    Caller->>Client: QueryStartExperience(want, callerInfo, rule)
    Client->>Client: CheckConnectService()
    Client->>Proxy: QueryStartExperience()
    
    Note over Proxy: 1. WriteInterfaceToken<br>2. WriteParcelable(want)<br>3. WriteParcelable(callerInfo)
    Proxy->>Stub: SendRequest(QUERY_START_EXPERIENCE_CMD)
    
    Note over Stub: 1. EnforceInterceToken()<br>2. VerifySystemApp()
    Stub->>Stub: ReadParcelable(want)
    Stub->>Stub: ReadParcelable(callerInfo)
    Stub->>Service: QueryStartExperience()
    
    Service-->>Stub: ExperienceRule
    Stub-->>Proxy: AmsExperienceRule
    Proxy-->>Client: ExperienceRule
    Client-->>Caller: ERR_OK / ERR_*
```

### 代码入口

| 步骤 | 文件 | 函数 |
|------|------|------|
| 1 | `ecological_rule_mgr_service_client.cpp` | `QueryStartExperience()` (第 135 行) |
| 2 | `ecological_rule_mgr_service_proxy.cpp` | `QueryStartExperience()` (第 107 行) |
| 3 | `ecological_rule_mgr_service_stub.cpp` | `OnQueryStartExperienceResult()` (第 159 行) |
| 4 | `ecologic_rule_mgr_service.cpp` | `QueryStartExperience()` (第 73 行) |

## 调用链 2: QueryFreeInstallExperience

### 调用方 → Client SDK → SA

```mermaid
sequenceDiagram
    participant Caller as BundleManagerService
    participant Client as ERMS Client
    participant Proxy as ERMS Proxy
    participant Stub as ERMS Stub
    participant Service as ERMS Service

    Caller->>Client: QueryFreeInstallExperience(want, callerInfo, rule)
    Client->>Proxy: QueryFreeInstallExperience()
    Proxy->>Stub: SendRequest(QUERY_FREE_INSTALL_EXPERIENCE_CMD)
    Stub->>Service: QueryFreeInstallExperience()
    Service-->>Stub: ExperienceRule
    Stub-->>Proxy: BmsExperienceRule
    Proxy-->>Client: ExperienceRule
    Client-->>Caller: ERR_OK / ERR_*
```

### 代码入口

| 步骤 | 文件 | 函数 |
|------|------|------|
| 1 | `ecological_rule_mgr_service_client.cpp` | `QueryFreeInstallExperience()` (第 100 行) |
| 2 | `ecological_rule_mgr_service_proxy.cpp` | `QueryFreeInstallExperience()` (第 30 行) |
| 3 | `ecological_rule_mgr_service_stub.cpp` | `OnQueryFreeInstallExperienceResult()` (第 67 行) |
| 4 | `ecologic_rule_mgr_service.cpp` | `QueryFreeInstallExperience()` (第 57 行) |

## 调用链 3: SA 启动流程

### SystemAbilityFramework 加载

```mermaid
flowchart TD
    A[系统启动] --> B[foundation 进程启动]
    B --> C[SAMgr 加载 SA]
    C --> D[EcologicalRuleMgrService 构造]
    D --> E[OnStart 调用]
    E --> F[Publish SA]
    F --> G[SA 就绪]
    
    G --> H[Client 调用 GetInstance]
    H --> I[ConnectService]
    I --> J[CheckSystemAbility]
    J --> K[返回 Proxy]
```

### 关键代码

| 步骤 | 文件 | 函数/宏 |
|------|------|----------|
| SA 注册 | `ecologic_rule_mgr_service.cpp` | `REGISTER_SYSTEM_ABILITY_BY_ID(EcologicalRuleMgrService, 6105, true)` |
| OnStart | `ecologic_rule_mgr_service.cpp` | `OnStart()` (第 92 行) |
| GetInstance | `ecologic_rule_mgr_service.cpp` | `GetInstance()` (第 46 行) |
| Publish | `ecologic_rule_mgr_service.cpp` | `Publish()` (第 100 行) |

## 调用链 4: SA 死亡重连

```mermaid
sequenceDiagram
    participant Client as ERMS Client
    participant Death as DeathRecipient
    participant SAMgr as SystemAbilityManager
    participant SA as ERMS SA

    Client->>SAMgr: AddDeathRecipient()
    SA-->>Death: OnRemoteDied()
    Death->>Client: OnRemoteSaDied()
    Client->>SAMgr: ConnectService()
    SAMgr-->>Client: New Proxy
```

### 关键代码

| 步骤 | 文件 | 函数 |
|------|------|------|
| 添加 DeathRecipient | `ecological_rule_mgr_service_client.cpp` | `ConnectService()` (第 77 行) |
| 死亡回调 | `ecological_rule_mgr_service_client.cpp` | `OnRemoteSaDied()` (第 95 行) |
| 重连逻辑 | `ecological_rule_mgr_service_client.cpp` | `ConnectService()` (第 65 行) |

## 调用链 5: 权限校验流程

```mermaid
flowchart TD
    A[IPC 调用到达] --> B{EnforceInterceToken?}
    B -->|失败| C[返回 ERR_PERMISSION_DENIED]
    B -->|成功| D{VerifySystemApp?}
    D -->|失败| E[返回 ERR_FAILED]
    D -->|成功| F[执行业务逻辑]
    
    D --> D1{Native/Shell?}
    D1 -->|是| F
    D1 -->|否| D2{ROOT_UID?}
    D2 -->|是| F
    D2 -->|否| D3{Foundation?}
    D3 -->|是| F
    D3 -->|否| D4{IsSystemApp?}
    D4 -->|是| F
    D4 -->|否| E
```

## 附录：IPC Code 映射

| Code | 枚举值 | 接口 | Stub Handler |
|------|--------|------|--------------|
| 0 | QUERY_FREE_INSTALL_EXPERIENCE_CMD | QueryFreeInstallExperience | OnQueryFreeInstallExperienceResult |
| 1 | QUERY_START_EXPERIENCE_CMD | QueryStartExperience | OnQueryStartExperienceResult |
| 2 | EVALUATE_RESOLVE_INFO_CMD | EvaluateResolveInfos | OnEvaluateResolveInfosResult |
| 3 | IS_SUPPORT_PUBLISH_FORM_CMD | IsSupportPublishForm | OnIsSupportPublishFormResult |

**参考文件**: `interfaces/innerkits/include/ecological_rule_mgr_service_interface.h:43-48`
