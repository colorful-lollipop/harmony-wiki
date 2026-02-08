# 调用链图谱

## 概述

本文档展示 Permission Lite 的关键调用链，包括权限校验流程、IPC 认证流程和权限管理流程。

---

## 权限校验调用链

### 同步校验流程

```mermaid
sequenceDiagram
    participant App as 应用/服务
    participant Client as pms_client
    participant IPC as IPC 框架
    participant Server as pms_server
    participant Storage as 权限存储

    App->>Client: CheckPermission(uid, permName)
    Client->>IPC: IPC 调用
    IPC->>Server: 消息分发
    Server->>Storage: ReadPermission(uid)
    Storage-->>Server: PermissionData
    Server->>Server: 权限匹配校验
    Server-->>IPC: 校验结果
    IPC-->>Client: 结果返回
    Client-->>App: 1 (有权限) / 0 (无权限)
```

### 代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| 入口 | `interfaces/kits/pms_interface.h` | `CheckPermission()` |
| 客户端 | `services/pms_client/perm_client.c` | `PermClientCheckPermission()` |
| 服务端 | `services/pms/src/pms_impl.c` | `PmsCheckPermission()` |
| 存储 | `services/pms/src/pms_inner.c` | `ReadPermissionFromFile()` |

---

## 权限授予调用链

### 安装时授予

```mermaid
sequenceDiagram
    participant BMS as Bundle Manager
    participant IPC as IPC 框架
    participant Server as pms_server
    participant JSON as JSON 解析器
    participant Storage as 权限存储

    BMS->>IPC: GrantPermission(bundleName, permName)
    IPC->>Server: 消息分发
    Server->>JSON: ParsePermissionDef()
    JSON-->>Server: PermissionDef
    Server->>Server: 权限验证
    Server->>Storage: SaveOrUpdatePermissions()
    Storage-->>Server: Success
    Server-->>IPC: Result
    IPC-->>BMS: 0 (成功)
```

### 运行时授予

```mermaid
sequenceDiagram
    participant App as 应用
    participant Client as pms_client
    participant Server as pms_server
    participant Storage as 权限存储

    App->>Client: GrantRuntimePermission(uid, permName)
    Client->>Server: IPC 调用
    Server->>Storage: UpdatePermissionFlags()
    Storage-->>Server: Success
    Server-->>Client: Result
    Client-->>App: 0 (成功)
```

### 代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| 入口 | `interfaces/kits/pms_interface.h` | `GrantPermission()` |
| 实现 | `services/pms/src/pms_impl.c` | `GrantPermission()` |
| 存储 | `services/pms/src/pms_inner.c` | `SavePermission()` |

---

## IPC 认证调用链

### 策略查询流程

```mermaid
sequenceDiagram
    participant SAMGR as SAMGR
    participant IPCAuth as ipc_auth
    participant Policy as 预设策略

    SAMGR->>IPCAuth: GetCommunicationStrategy(RegParams)
    IPCAuth->>Policy: LookupPolicy(serviceName)
    Policy-->>IPCAuth: PolicyTrans[]
    IPCAuth-->>SAMGR: 策略数组
```

### 访问校验流程

```mermaid
sequenceDiagram
    participant Caller as 调用方进程
    participant SAMGR as SAMGR
    participant IPCAuth as ipc_auth
    participant Policy as 预设策略

    Caller->>SAMGR: IPC 请求 (targetService)
    SAMGR->>IPCAuth: IsCommunicationAllowed(AuthParams)
    IPCAuth->>Policy: CheckPolicy(callerUid, serviceName)
    Policy-->>IPCAuth: Allow/Deny
    IPCAuth-->>SAMGR: 1 (允许) / 0 (拒绝)
    SAMGR-->>Caller: 转发/拒绝
```

### 代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| 策略查询 | `services/ipc_auth/src/ipc_auth_impl.c` | `GetCommunicationStrategy()` |
| 策略校验 | `services/ipc_auth/src/ipc_auth_impl.c` | `IsCommunicationAllowed()` |
| 策略配置 | `services/ipc_auth/include/policy_preset.h` | `g_presetPolicies[]` |

---

## 权限存储调用链

### 读取权限

```mermaid
flowchart TD
    A[ReadPermission] --> B{缓存存在?}
    B -->|是| C[返回缓存]
    B -->|否| D[打开文件]
    D --> E[解析 JSON]
    E --> F[更新缓存]
    F --> G[返回数据]
    G --> C
```

### 写入权限

```mermaid
flowchart TD
    A[WritePermission] --> B[构建 JSON]
    B --> C[写入临时文件]
    C --> D{写入成功?}
    D -->|是| E[重命名文件]
    D -->|否| F[删除临时文件]
    E --> G[更新缓存]
    F --> H[返回错误]
```

### 代码路径

| 操作 | 文件 | 函数 |
|------|------|------|
| 读取 | `services/pms/src/pms_inner.c` | `ReadPermissionFromFile()` |
| 写入 | `services/pms/src/pms_inner.c` | `WritePermissionToFile()` |
| 解析 | `services/pms/src/pms_inner.c` | `ParsePermissionJson()` |

---

## JS API 调用链

### ACE Lite JSI 调用

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant JSI as JSI Runtime
    participant Module as PermModule
    participant PMS as PMS Client

    JS->>JSI: check(permissionName)
    JSI->>Module: CheckSelfPerm(thisVal, args)
    Module->>PMS: CheckSelfPermission(permName)
    PMS->>PMS: IPC 调用
    PMS-->>Module: Result
    Module-->>JSI: boolean
    JSI-->>JS: true/false
```

### 代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| 入口 | `services/js_api/src/perm_module.cpp` | `CheckSelfPerm()` |
| 实现 | `services/js_api/src/perm_module.cpp` | `PermModule::CheckSelfPerm()` |

---

## 服务初始化调用链

```mermaid
sequenceDiagram
    participant Init as 系统初始化
    participant Base as pms_base
    participant Server as pms_server
    participant SAMGR as SAMGR

    Init->>Base: APP_FEATURE_INIT()
    Base->>Server: CreateInstance()
    Server->>Server: Initialize()
    Server->>Server: LoadAllPermissions()
    Server->>SAMGR: RegisterFeature()
    Server-->>Base: Success
    Base-->>Init: 完成
```

### 代码路径

| 步骤 | 文件 | 函数 |
|------|------|------|
| 初始化 | `services/pms_base/src/permission_service.c` | `Init()` |
| 实例创建 | `services/pms_base/src/permission_service.c` | `PermissionService::GetInstance()` |
| 服务注册 | `services/pms/src/pms_server.c` | `Init()` |

---

## 完整调用链视图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              权限校验完整调用链                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐                                                               │
│  │   App    │  (调用方)                                                     │
│  └───┬──────┘                                                               │
│      │                                                                      │
│      ▼  C API 调用                                                         │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐          │
│  │   C API  │────▶│Client    │────▶│   IPC    │────▶│  Server  │          │
│  │ Interface│     │Stub      │     │Framework │     │Dispatcher│          │
│  └───┬──────┘     └──────────┘     └──────────┘     └────┬─────┘          │
│      │                                                   │                 │
│      │                                                   ▼                 │
│      │                                           ┌──────────────┐          │
│      │                                           │ 权限校验逻辑 │          │
│      │                                           │(PmsCheck)    │          │
│      │                                           └──────┬───────┘          │
│      │                                                  │                  │
│      │                                                  ▼                  │
│      │                                          ┌──────────────┐          │
│      │                                          │  权限存储    │          │
│      │                                          │(JSON 文件)   │          │
│      │                                          └──────────────┘          │
│      │                                                                          │
│      ◀────────────────────────────────────────────────────────────────────────┘
│                                      │
│                              返回校验结果 (1/0)
│
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 相关文档

- 架构说明 → `02_Architecture.md`
- API 接口 → `03_APIs.md`
- 内部 API → `appendix/06_Inner_APIs.md`
