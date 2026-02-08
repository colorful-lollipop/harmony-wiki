# 架构设计

描述分布式数据管理服务的整体架构、组件关系和数据流。

---

## 1. 三层架构概览

```
┌─────────────────────────────────────────────────────────────────────┐
│                         App Layer (app/)                            │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    KvStoreDataService                          │  │
│  │  - SystemAbility 实现 (SA ID: 1301)                           │  │
│  │  - FeatureStubImpl: 客户端接口存根                             │  │
│  │  - 会话管理、安全检查、备份规则                                │  │
│  └───────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│                         Service Layer (service/)                     │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │    KVDB      │ │     RDB      │ │    Cloud     │              │
│  │  Feature     │ │  Feature     │ │  Feature     │              │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │   Object     │ │  DataShare   │ │    UDMF      │              │
│  │  Feature     │ │  Feature     │ │  Feature     │              │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  公共模块: bootstrap, backup, permission, dumper, matrix     │  │
│  └─────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│                      Framework Layer (framework/)                  │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  核心基础设施:                                                  │  │
│  │  - AutoCache: 数据库句柄管理                                  │  │
│  │  - MetaDataManager: 元数据管理                               │  │
│  │  - FeatureSystem: 插件注册系统                               │  │
│  │  - EventCenter: 事件订阅/发布                                │  │
│  │  - CryptoManager: 加密管理                                   │  │
│  │  - DirectoryManager: 目录管理                                │  │
│  └─────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│                       Adapter Layer (adapter/)                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │  Account     │ │ Communicator │ │   Network   │              │
│  │  Adapter     │ │   Adapter    │ │   Adapter   │              │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │    DFX       │ │  ScreenLock  │ │     QoS     │              │
│  │  Adapter     │ │   Adapter    │ │   Adapter   │              │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. 目录结构

### 2.1 App Layer (`services/distributeddataservice/app/`)

| 目录/文件 | 职责 |
|----------|------|
| `src/kvstore_data_service.h` | SystemAbility 主类定义 |
| `src/kvstore_data_service.cpp` | SystemAbility 实现 |
| `src/feature_stub_impl.h` | Feature 存根实现 |
| `src/session_manager/` | 会话管理 |
| `src/security/` | 安全相关 |
| `src/checker/` | 签名/权限检查器 |
| `src/backup_rule/` | 备份规则 |
| `src/installer/` | 安装器 |

**证据**: `bundle.json` 定义子组件 `//foundation/.../app:build_module`

### 2.2 Service Layer (`services/distributeddataservice/service/`)

| 模块 | 目录 | 职责 | 关键类 |
|-----|------|------|--------|
| **kvdb** | `service/kvdb/` | KV 数据库服务 | `KvDbServiceImpl`, `KvDbGeneralStore` |
| **rdb** | `service/rdb/` | 关系数据库服务 | `RdbServiceImpl`, `RdbGeneralStore` |
| **cloud** | `service/cloud/` | 云同步服务 | `CloudServiceImpl`, `SyncManager` |
| **object** | `service/object/` | 分布式对象 | `ObjectServiceImpl`, `ObjectManager` |
| **data_share** | `service/data_share/` | 跨应用数据共享 | `DataShareServiceImpl` |
| **udmf** | `service/udmf/` | 统一数据管理 | `UdmfServiceImpl` |
| **utd** | `service/utd/` | 统一类型描述 | `UtdServiceImpl` |
| **bootstrap** | `service/bootstrap/` | 组件初始化 | `Bootstrap` |
| **backup** | `service/backup/` | 备份恢复 | `BackupManager` |
| **permission** | `service/permission/` | 权限检查 | `PermitDelegate` |
| **dumper** | `service/dumper/` | 调试 dump | `DumpHelper` |
| **matrix** | `service/matrix/` | 设备矩阵 | `DeviceMatrix` |
| **common** | `service/common/` | 公共工具 | - |
| **config** | `service/config/` | 配置管理 | - |

### 2.3 Framework Layer (`services/distributeddataservice/framework/`)

| 模块 | 职责 | 说明 |
|-----|------|------|
| **account** | 账户抽象 | 多用户支持 |
| **app_id_mapping** | App ID 映射 | 配置管理 |
| **access_check** | 访问检查配置 | 权限相关 |
| **backuprule** | 备份规则管理 | - |
| **changeevent** | 远程变更事件 | - |
| **checker** | 检查器框架 | 权限/验证 |
| **cloud** | 云服务抽象 | `cloud_db`, `cloud_server` |
| **communication** | 通信管理 | ConnectManager |
| **crypto** | 加密管理 | CryptoManager |
| **device_manager** | 设备管理抽象 | DeviceManagerDelegate |
| **dfx** | 可观测性 | Reporter |
| **directory** | 目录管理 | DirectoryManager |
| **dump** | dump 管理 | DumpManager |
| **eventcenter** | 事件中心 | EventCenter, Event |
| **feature** | Feature 系统 | FeatureSystem |
| **flow_control** | 流控管理 | FlowControlManager |
| **metadata** | 元数据管理 | MetaDataManager, StoreMetaData |
| **network** | 网络抽象 | NetworkDelegate |
| **screen** | 屏幕状态 | ScreenManager |
| **serializable** | 序列化工具 | Serializable |
| **snapshot** | 快照管理 | Snapshot |
| **store** | 存储抽象 | AutoCache, GeneralStore |
| **sync_mgr** | 同步管理 | SyncManager |
| **thread** | 线程管理 | ThreadManager |
| **utils** | 公共工具 | 加密、匿名化等 |

### 2.4 Adapter Layer (`services/distributeddataservice/adapter/`)

| 模块 | 职责 | 系统服务调用 |
|-----|------|--------------|
| **account** | 账户适配器 | AccountManagerService |
| **communicator** | 通信适配器 | SoftBus |
| **network** | 网络适配器 | NetConnManager |
| **dfx** | DFX 适配器 | Hiview, Hisysevent |
| **screenlock** | 屏幕锁适配器 | ScreenLockManager |
| **qos** | QoS 适配器 | QoSManager |
| **utils** | 工具适配器 | - |

---

## 3. Feature 系统

### 3.1 架构

```
┌─────────────────────────────────────────────────────────┐
│              FeatureSystem (插件注册中心)                │
│  ┌─────────────────────────────────────────────────┐   │
│  │  RegisterCreator(name, creator, bindMode)      │   │
│  │  GetCreator(name) → Feature*                   │   │
│  │  RegisterStaticActs(name, acts)                │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    ┌─────────┐     ┌─────────┐     ┌─────────┐
    │  KVDB   │     │   RDB   │     │  Cloud  │
    │ Feature │     │ Feature │     │ Feature │
    └─────────┘     └─────────┘     └─────────┘
```

### 3.2 Feature 注册点

| Feature | 注册文件 | Feature Name |
|---------|----------|--------------|
| **KVDB** | `service/kvdb/kvdb_service_impl.cpp` | `"kv_store"` |
| **RDB** | `service/rdb/rdb_service_impl.cpp` | `"relational_store"` |
| **Cloud** | `service/cloud/cloud_service_impl.cpp` | `"cloud"` |
| **Object** | `service/object/src/object_service_impl.cpp` | `"data_object"` |
| **DataShare** | `service/data_share/data_share_service_impl.cpp` | `"data_share"` |
| **UDMF** | `service/udmf/udmf_service_impl.cpp` | `"udmf"` |
| **UTD** | `service/utd/utd_service_impl.cpp` | `"utd"` |

### 3.3 注册模式

```cpp
// Feature 实现类继承 Feature 接口
class KvDbFeature : public Feature {
    // 实现 Feature 生命周期方法
};

// 注册到 FeatureSystem
FeatureSystem::GetInstance().RegisterCreator(
    "kv_store",
    []() { return std::shared_ptr<Feature>(new KvDbFeature()); },
    FeatureSystem::BIND_NOW  // 或 LAZY
);
```

**证据**: `framework/include/feature/feature_system.h`

---

## 4. 数据流

### 4.1 客户端请求处理流程

```
┌─────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  JS API │────▶│ IPC Stub    │────▶│ FeatureStub │────▶│ Feature     │
│ (Client)│     │ (App进程)   │     │ Impl        │     │ (Service)   │
└─────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                  │
                                                                  ▼
                                                           ┌─────────────┐
                                                           │   AutoCache │
                                                           │  (句柄管理)  │
                                                           └─────────────┘
                                                                  │
                                    ┌─────────────────────────────┘
                                    ▼
                             ┌─────────────┐
                             │ 底层存储     │
                             │ KV/RDB      │
                             └─────────────┘
```

### 4.2 同步流程

```
┌──────────┐     ┌─────────────┐     ┌─────────────┐     ┌──────────┐
│ 本地写入  │────▶│ AutoCache   │────▶│  SyncMgr    │────▶│ SoftBus  │
│          │     │ (句柄管理)   │     │ (同步协调)   │     │ (传输)   │
└──────────┘     └─────────────┘     └─────────────┘     └──────────┘
                                               │
                    ┌──────────────────────────┘
                    ▼
             ┌─────────────┐
             │  远端设备    │
             │  接收同步    │
             └─────────────┘
```

---

## 5. 线程模型

### 5.1 线程池配置

| 线程类型 | 说明 | 配置来源 |
|---------|------|---------|
| **IPC 线程** | 处理 IPC 请求 | `ThreadManager::Init ipcThreadNum` |
| **工作线程** | 业务逻辑处理 | `ThreadManager::Init min/maxThreadNum` |
| **定时器线程** | 备份/同步调度 | `BackupManager` |

**证据**: `service/bootstrap/src/bootstrap.cpp` `LoadThread()` 方法

### 5.2 线程安全

| 组件 | 线程安全机制 |
|-----|-------------|
| **AutoCache** | 内部同步，线程安全 |
| **MetaDataManager** | 内部同步，线程安全 |
| **EventCenter** | 事件订阅/发布线程安全 |
| **ConcurrentMap** | 并发安全 Map |

---

## 6. 生命周期

### 6.1 服务启动流程

```
OnStart()
├── LoadConfigs()           Bootstrap 加载配置
│   ├── LoadThread()        线程配置
│   ├── LoadDirectory()     目录配置
│   ├── LoadComponents()    动态组件 (dlopen)
│   ├── LoadCheckers()      检查器配置
│   └── LoadBackup()        备份配置
├── Initialize()            核心初始化
│   ├── InitExecutor()      线程池
│   ├── Init Security       安全模块
│   └── 订阅系统事件        Account/Screen/Device
└── StartService()         发布服务
    └── LoadFeatures()      加载 Features
```

**证据**: `app/src/kvstore_data_service.cpp` `OnStart()` 方法

### 6.2 Feature 生命周期

```
Create() → OnInitialize() → OnBind() → [业务运行] → OnUnbind() → OnRelease()
```

---

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| 概览 | [00_Overview.md](00_Overview.md) |
| 构建配置 | [02_Build.md](02_Build.md) |
| 安全评审 | [03_Security.md](03_Security.md) |
