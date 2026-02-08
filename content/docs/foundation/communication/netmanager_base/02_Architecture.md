# 架构设计与组件关系

## 1. 整体架构图

```mermaid
graph TB
    subgraph ApplicationLayer["应用层"]
        JS[JS/TS 应用]
        ArkTS[ArkTS 应用]
        CppApp[C++ 应用]
        CApp[C 应用]
    end

    subgraph FrameworkLayer["框架层"]
        NAPI[JS N-API]
        ANI[ETS ANI]
        Native[Native API]
        CAPI[C API]
    end

    subgraph InterfaceLayer["接口层"]
        InnerKits[InnerKits<br/>C++ 接口]
    end

    subgraph ServiceLayer["服务层"]
        NetConn[NetConnService<br/>SA 1151]
        NetPolicy[NetPolicyService<br/>SA 1152]
        NetStats[NetStatsService<br/>SA 1153]
    end

    subgraph ControllerLayer["控制器层"]
        NetsysCtrl[NetsysController]
    end

    subgraph NativeLayer["Native层"]
        NetsysNative[NetsysNativeService<br/>SA 1158]
    end

    subgraph KernelLayer["内核层"]
        Kernel[Linux Kernel]
        BPF[BPF/eBPF]
        Netlink[Netlink]
        Iptables[iptables/nftables]
    end

    JS --> NAPI
    ArkTS --> ANI
    CppApp --> Native
    CApp --> CAPI

    NAPI --> InnerKits
    ANI --> InnerKits
    Native --> InnerKits
    CAPI --> InnerKits

    InnerKits --> NetConn
    InnerKits --> NetPolicy
    InnerKits --> NetStats

    NetConn --> NetsysCtrl
    NetPolicy --> NetsysCtrl
    NetStats --> NetsysCtrl

    NetsysCtrl --> NetsysNative

    NetsysNative --> Kernel
    NetsysNative --> BPF
    NetsysNative --> Netlink
    NetsysNative --> Iptables
```

---

## 2. 组件详细说明

### 2.1 服务层组件

| 组件 | SA ID | 进程 | 核心职责 |
|------|-------|------|----------|
| NetConnService | 1151 | netmanager | 网络连接管理、网络选择、代理配置 |
| NetPolicyService | 1152 | netmanager | 网络策略、防火墙、配额管理 |
| NetStatsService | 1153 | netmanager | 流量统计、历史数据、告警 |
| NetsysNativeService | 1158 | netsysnative | 底层网络操作、DNS、路由、防火墙 |

### 2.2 进程模型

```
┌─────────────────────────────────────────────────────────────┐
│                     netmanager 进程                          │
│                                                              │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
│  │ NetConnService│  │NetPolicyService│  │ NetStatsService│   │
│  │    (SA 1151)  │  │    (SA 1152)   │  │    (SA 1153)   │   │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘   │
│          │                  │                  │            │
│          └──────────────────┴──────────────────┘            │
│                             │                               │
│                    ┌────────┴────────┐                      │
│                    │  NetsysController │                      │
│                    └────────┬────────┘                      │
└─────────────────────────────┼───────────────────────────────┘
                              │
                              │ IPC (Binder)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   netsysnative 进程                          │
│                                                              │
│              ┌──────────────────────────┐                   │
│              │   NetsysNativeService    │                   │
│              │        (SA 1158)         │                   │
│              └──────────────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 数据流

### 3.1 网络连接建立流

```mermaid
sequenceDiagram
    participant App as JS应用
    participant NAPI as N-API
    participant Client as NetConnClient
    participant Service as NetConnService
    participant Controller as NetsysController
    participant Native as NetsysNative
    participant Kernel as Kernel

    App->>NAPI: registerNetSupplier()
    NAPI->>Client: 调用内部接口
    Client->>Service: IPC: RegisterNetSupplier
    Service->>Service: 创建 NetSupplier
    
    Note over Service: 网络供应商注册
    
    App->>NAPI: updateNetLinkInfo()
    NAPI->>Client: 调用内部接口
    Client->>Service: IPC: UpdateNetLinkInfo
    Service->>Service: 更新网络链路信息
    Service->>Controller: 网络配置操作
    Controller->>Native: IPC: 底层网络操作
    Native->>Kernel: netlink 系统调用
    Kernel-->>Native: 返回结果
    Native-->>Controller: 返回结果
    Controller-->>Service: 返回结果
    Service->>Service: 评估最佳网络
    Service-->>Client: 回调通知
    Client-->>NAPI: 回调通知
    NAPI-->>App: 网络可用事件
```

### 3.2 网络策略控制流

```mermaid
sequenceDiagram
    participant Settings as 设置应用
    participant NAPI as N-API
    participant Client as NetPolicyClient
    participant Service as NetPolicyService
    participant Controller as NetsysController
    participant Native as NetsysNative
    participant Iptables as iptables

    Settings->>NAPI: setPolicyByUid(uid, policy)
    NAPI->>Client: 调用内部接口
    Client->>Service: IPC: SetPolicyByUid
    Service->>Service: 存储策略到数据库
    Service->>Service: 通知策略变更
    Service->>Controller: 下发防火墙规则
    Controller->>Native: IPC: 防火墙配置
    Native->>Iptables: 执行 iptables 命令
    Iptables-->>Native: 返回结果
    Native-->>Controller: 返回结果
    Controller-->>Service: 返回结果
    Service-->>Client: 返回结果
    Client-->>NAPI: 返回结果
    NAPI-->>Settings: 返回结果
```

### 3.3 流量统计查询流

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant Client as NetStatsClient
    participant Service as NetStatsService
    participant Controller as NetsysController
    participant Native as NetsysNative
    participant BPF as BPF

    App->>NAPI: getUidRxBytes(uid)
    NAPI->>Client: 调用内部接口
    Client->>Service: IPC: GetUidRxBytes
    Service->>Service: 查询缓存/数据库
    
    alt 缓存未命中
        Service->>Controller: 查询实时流量
        Controller->>Native: IPC: 获取统计
        Native->>BPF: 读取 BPF Map
        BPF-->>Native: 返回原始数据
        Native-->>Controller: 返回统计值
        Controller-->>Service: 返回统计值
        Service->>Service: 更新缓存
    end
    
    Service-->>Client: 返回流量值
    Client-->>NAPI: 返回结果
    NAPI-->>App: 返回流量值
```

---

## 4. 线程模型

### 4.1 NetConnService 线程模型

```
┌─────────────────────────────────────────────────────────────────┐
│                      NetConnService                             │
│                                                                 │
│  ┌──────────────────┐    ┌─────────────────────────────────┐  │
│  │  EventHandler    │    │         工作线程池              │  │
│  │  (主线程)        │    │  ┌─────┐ ┌─────┐ ┌─────┐       │  │
│  │                  │    │  │Worker│ │Worker│ │Worker│       │  │
│  │  - 处理IPC请求   │    │  └─────┘ └─────┘ └─────┘       │  │
│  │  - 网络事件分发  │    │                                 │  │
│  │  - 定时任务      │    │  处理阻塞操作：                 │  │
│  │  - 回调通知      │    │  - HTTP探测                     │  │
│  │                  │    │  - 数据库查询                   │  │
│  └──────────────────┘    │  - 网络配置                     │  │
│                          └─────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**关键线程**:
- **主线程 (EventHandler)**: 处理 IPC 请求、网络状态变更事件
- **HTTP 探测线程**: 执行网络连通性探测
- **代理检测线程**: 检测 PAC 代理配置

### 4.2 NetsysNativeService 线程模型

```
┌─────────────────────────────────────────────────────────────────┐
│                   NetsysNativeService                           │
│                                                                 │
│  ┌──────────────────┐    ┌─────────────────────────────────┐  │
│  │  IPC 线程        │    │        DNS 线程池               │  │
│  │                  │    │  ┌─────┐ ┌─────┐ ┌─────┐       │  │
│  │  - Binder 通信   │    │  │DNS  │ │DNS  │ │DNS  │       │  │
│  │  - 接口分发      │    │  │Worker│ │Worker│ │Worker│       │  │
│  │                  │    │  └─────┘ └─────┘ └─────┘       │  │
│  └──────────────────┘    │                                 │  │
│                          │  并发 DNS 解析                  │  │
│  ┌──────────────────┐    └─────────────────────────────────┘  │
│  │  Netlink 线程    │                                          │
│  │                  │    ┌─────────────────────────────────┐  │
│  │  - 监听内核事件  │    │        BPF 轮询线程             │  │
│  │  - 接口变更      │    │                                 │  │
│  │  - 路由变更      │    │  - 读取 BPF Ring Buffer         │  │
│  │                  │    │  - 流量统计更新                 │  │
│  └──────────────────┘    │  - 防火墙事件                   │  │
│                          └─────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. 关键时序图

### 5.1 网络连接注册与激活时序

```mermaid
sequenceDiagram
    participant Supplier as 网络供应商<br/>(WiFi/蜂窝)
    participant Service as NetConnService
    participant Network as Network对象
    participant Native as NetsysNative
    participant App as 应用

    Note over Supplier,App: 网络连接建立流程

    Supplier->>Service: RegisterNetSupplier()
    Service->>Service: 创建 NetSupplier 对象
    Service-->>Supplier: 返回 supplierId

    Supplier->>Service: UpdateNetSupplierInfo()
    Service->>Service: 更新供应商状态

    Supplier->>Service: UpdateNetLinkInfo()
    Service->>Network: 创建 Network 对象
    Service->>Service: 评估最佳网络
    
    Service->>Native: 配置网络 (IP/路由/DNS)
    Native-->>Service: 配置完成
    
    Service->>Service: 标记网络可用
    Service->>App: 回调: onNetAvailable()
    Service->>App: 回调: onNetCapabilitiesChange()
    Service->>App: 回调: onConnectionPropertiesChange()
```

### 5.2 网络切换时序

```mermaid
sequenceDiagram
    participant WiFi as WiFi网络
    participant Cellular as 蜂窝网络
    participant Service as NetConnService
    participant App as 应用

    Note over WiFi,App: 从蜂窝切换到WiFi

    WiFi->>Service: UpdateNetLinkInfo()
    Service->>Service: 评估网络得分
    Service->>Service: WiFi得分 > 蜂窝得分
    
    Service->>Service: 切换默认网络
    Service->>Cellular: 降级为非默认
    Service->>WiFi: 提升为默认
    
    Service->>App: 回调: onNetLost(蜂窝)
    Service->>App: 回调: onNetAvailable(WiFi)
    Service->>App: 回调: onNetCapabilitiesChange()
```

### 5.3 应用网络策略生效时序

```mermaid
sequenceDiagram
    participant Settings as 设置应用
    participant PolicyService as NetPolicyService
    participant Database as 策略数据库
    participant ConnService as NetConnService
    participant Native as NetsysNative
    participant App as 目标应用

    Settings->>PolicyService: setPolicyByUid(uid, POLICY_REJECT)
    PolicyService->>Database: 存储策略
    PolicyService->>ConnService: 通知策略变更
    PolicyService->>Native: 下发防火墙规则
    Native->>Native: 更新 iptables 规则
    
    Note over App: 应用尝试网络访问
    App->>Native: socket 连接请求
    Native->>Native: 检查 UID 策略
    Native->>App: 拒绝连接
```

---

## 6. 模块交互关系

### 6.1 核心交互矩阵

| 调用方 | 被调用方 | 通信方式 | 典型场景 |
|--------|----------|----------|----------|
| NetConnService | NetsysController | IPC | 网络配置、路由设置 |
| NetPolicyService | NetsysController | IPC | 防火墙规则下发 |
| NetStatsService | NetsysController | IPC | 流量统计查询 |
| NetConnService | NetPolicyService | IPC | 策略同步通知 |
| NetsysController | NetsysNativeService | IPC | 底层网络操作 |
| NetConnService | NetConnClient | Callback | 网络状态通知 |
| NetsysNativeService | Kernel | Netlink | 网络配置 |
| NetsysNativeService | Kernel | BPF | 流量统计 |

### 6.2 回调机制

```
┌─────────────────────────────────────────────────────────────────┐
│                        回调架构                                  │
│                                                                 │
│  应用层 (JS/C++/C)                                              │
│       │                                                         │
│       │ 注册回调                                                 │
│       ▼                                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  INetConnCallback / INetPolicyCallback / ...            │  │
│  │  (IPC Callback Interface)                                │  │
│  └──────────────────────────────────────────────────────────┘  │
│       │                                                         │
│       │ IPC (DeathRecipient 处理断线)                          │
│       ▼                                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  NetConnService / NetPolicyService / ...                 │  │
│  │  (Service 发起回调)                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  回调类型:                                                      │
│  - onNetAvailable / onNetLost                                  │
│  - onNetCapabilitiesChange                                     │
│  - onConnectionPropertiesChange                                │
│  - onUidPolicyChange                                           │
│  - onQuotaPolicyChange                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. 设计模式应用

### 7.1 使用的关键设计模式

| 模式 | 应用场景 | 文件 |
|------|----------|------|
| **单例模式** | NetsysController | `services/netsyscontroller/include/netsys_controller.h` |
| **观察者模式** | 网络状态监听 | `services/netconnmanager/include/net_activate.h` |
| **策略模式** | 网络选择策略 | `services/netconnmanager/src/net_conn_service.cpp` |
| **工厂模式** | Network 对象创建 | `services/netconnmanager/include/network.h` |
| **代理模式** | IPC Proxy/Stub | `interfaces/innerkits/*/include/proxy/` |
| **桥接模式** | N-API 封装 | `frameworks/js/napi/connection/` |

### 7.2 类层次结构

```
SystemAbility (OpenHarmony基类)
    │
    ├── NetConnService
    │       ├── INetActivateCallback
    │       └── NetConnServiceStub
    │
    ├── NetPolicyService
    │       └── NetPolicyCallbackStub
    │
    ├── NetStatsService
    │       └── NetStatsServiceStub
    │
    └── NetsysNativeService
            └── NetsysNativeServiceStub

IRemoteBroker (IPC接口基类)
    │
    ├── INetConnService
    │       ├── NetConnServiceStub (服务端)
    │       └── NetConnServiceProxy (客户端)
    │
    ├── INetPolicyService
    │       ├── NetPolicyServiceStub
    │       └── NetPolicyServiceProxy
    │
    └── INetsysService
            ├── NetsysNativeServiceStub
            └── NetsysNativeServiceProxy
```

---

*生成时间: 2025-02-06*
