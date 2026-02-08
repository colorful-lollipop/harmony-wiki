# 架构设计

> 本文档描述 storage_service 的整体架构，包括分层设计、组件关系、IPC 通信和线程模型。

## 整体架构

### 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Application)                      │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           JS API (@ohos.storageManager)               │  │
│  │   file.storageStatistics  │  file.volumeManager  │   │  │
│  │   file.keyManager         │                       │   │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓ IPC (N-API)
┌─────────────────────────────────────────────────────────────┐
│              storage_manager (SA Manager)                    │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              StorageManagerProvider                  │  │
│  │  (STORAGE_MANAGER_MANAGER_ID)                      │  │
│  │         业务逻辑层 - 用户/卷/统计/加密管理           │  │
│  └─────────────────────────────────────────────────────┘  │
│                              ↓ IPC (Binder)                  │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           StorageDaemonCommunication                │  │
│  │         IPC 客户端 - 与 storage_daemon 通信           │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓ IPC (Binder)
┌─────────────────────────────────────────────────────────────┐
│              storage_daemon (SA Daemon)                       │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           StorageDaemonProvider                     │  │
│  │  (STORAGE_MANAGER_DAEMON_ID)                       │  │
│  │       底层操作 - 挂载/加密/配额/分区                  │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    内核层 (Kernel)                           │
│  • Uevent (设备热插拔)                                       │
│  • Vold (卷管理)                                            │
│  • FsCrypt (文件加密)                                       │
└─────────────────────────────────────────────────────────────┘
```

**证据来源**：`README_zh.md:6-15`、`bundle.json:97-115`

## 组件职责

### 1. storage_api (JS 层)

> 为应用提供一套查询、管理存储和用户的接口 API。

| 模块 | 模块名 | 入口文件 | 说明 |
|------|-------|---------|------|
| 存储统计 | `file.storageStatistics` | `storage_statistics_napi.cpp` | 空间统计、Bundle 统计 |
| 卷管理 | `file.volumeManager` | `volumemanager_napi.cpp` | 挂载/卸载/格式化 |
| 密钥管理 | `file.keyManager` | `keymanager_napi.cpp` | 用户密钥激活/停用 |
| 分布式文件 | (DFS) | `napi_module_dfs_service.cpp` | 分布式文件服务 |

**证据来源**：`services/storage_manager/kits_impl/src/*_napi.cpp`

### 2. storage_manager (服务层)

> 提供卷、磁盘的相关查询能力和管理能力，多用户数据目录管理接口及以应用或用户为维度的存储空间统计查询能力。

| 功能模块 | 目录 | 说明 |
|---------|------|------|
| IPC | `services/storage_manager/ipc/` | SA Provider、Stub |
| Volume | `services/storage_manager/volume/` | 卷管理逻辑 |
| Disk | `services/storage_manager/disk/` | 磁盘管理 |
| Storage | `services/storage_manager/storage/` | 存储统计 |
| Account | `services/storage_manager/account_subscriber/` | 账户订阅 |
| Communication | `services/storage_manager/storage_daemon_communication/` | IPC 客户端 |

**证据来源**：`bundle.json:97-115`、`services/storage_manager/BUILD.gn`

### 3. storage_daemon (守护进程层)

> 提供分区挂载能力，与内核层的交互能力、设备上下线监听能力及目录加解密能力。

| 功能模块 | 目录 | 说明 |
|---------|------|------|
| IPC | `services/storage_daemon/ipc/` | SA Provider、Stub |
| User | `services/storage_daemon/user/` | 用户目录管理 |
| Volume | `services/storage_daemon/volume/` | 卷操作 |
| Disk | `services/storage_daemon/disk/` | 磁盘操作 |
| Crypto | `services/storage_daemon/crypto/` | 加密管理 |
| File Sharing | `services/storage_daemon/file_sharing/` | 共享文件 |
| MTP | `services/storage_daemon/mtp/` | MTP 设备支持 |
| Netlink | `services/storage_daemon/netlink/` | 内核事件监听 |

**证据来源**：`services/storage_daemon/BUILD.gn`

## SA (System Ability) 设计

### 两层 SA 架构

| SA ID | 服务名 | 进程 | 说明 |
|-------|--------|------|------|
| `STORAGE_MANAGER_MANAGER_ID` | storage_manager | 独立进程 | 上层服务，对外暴露 API |
| `STORAGE_MANAGER_DAEMON_ID` | storage_daemon | 独立进程 | 底层守护进程，执行实际操作 |

**SA 注册位置**：
- `storage_manager`: `services/storage_manager/ipc/src/storage_manager_provider.cpp:56`
- `storage_daemon`: `services/storage_daemon/main.cpp:125`

**关键代码**：
```cpp
// storage_manager 注册
REGISTER_SYSTEM_ABILITY_BY_ID(StorageManagerProvider, STORAGE_MANAGER_MANAGER_ID, true);

// storage_daemon 注册
sptr<StorageDaemon::StorageDaemonProvider> sd(new StorageDaemon::StorageDaemonProvider());
samgr->AddSystemAbility(STORAGE_MANAGER_DAEMON_ID, sd);
```

## IPC 通信架构

### 通信流程

```
JS App
    ↓ napi_call_function
storage_statistics_napi.cpp (N-API 层)
    ↓
StorageManagerConnect::GetXxx() (IPC 客户端)
    ↓ IPCSkeleton::GetSystemAbility()
SAMgr → storage_manager (SA Manager)
    ↓ IPC 调用
StorageDaemonCommunication (IPC 客户端)
    ↓ IPCSkeleton::GetSystemAbility()
SAMgr → storage_daemon (SA Daemon)
    ↓
StorageDaemon (核心实现)
    ↓
Kernel (Vold/FsCrypt/Uevent)
```

**IPC 客户端连接代码**：
- `services/storage_manager/kits_impl/src/storage_manager_connect.cpp:34-65`
- `services/storage_manager/storage_daemon_communication/src/storage_daemon_communication.cpp:43-72`

### IPC 接口定义

| 接口 | 继承 | 说明 |
|------|------|------|
| `IStorageManager` | IRemoteBroker | storage_manager 对外接口 |
| `IStorageDaemon` | IRemoteBroker | storage_daemon 对外接口 |
| `StorageManagerProvider` | SystemAbility + StorageManagerStub | SA Provider |
| `StorageDaemonProvider` | StorageDaemonStub | SA Provider |

**证据来源**：
- `services/storage_manager/include/ipc/storage_manager_provider.h`
- `services/storage_daemon/include/ipc/storage_daemon_provider.h`

## 线程模型

### 主线程

- **N-API 层**：JS 调用在主线程执行，异步操作通过 `NAsyncWorkPromise` 派发到线程池
- **IPC 消息循环**：通过 `IPCSkeleton::JoinWorkThread()` 启动 IPC 消息处理线程

### 线程池

- **异步任务**：存储统计、卷查询等耗时操作在线程池执行
- **NAsyncWork**：使用 `NAsyncWorkPromise` / `NAsyncWorkCallback` 模式

**证据来源**：`services/storage_manager/kits_impl/src/storage_statistics_n_exporter.cpp:54-70`

## 依赖关系

### 外部依赖方向

```
storage_service
    ├── ability_runtime (Extension 能力)
    ├── access_token (权限校验)
    ├── bundle_framework (应用信息)
    ├── crypto_framework (加密)
    ├── ipc (IPC 通信)
    ├── napi (JS 绑定)
    ├── os_account (多用户)
    ├── safwk (SA 框架)
    └── samgr (服务管理)
```

### 内部依赖方向

```
interfaces/kits/js/
    ↓ deps
storage_manager_sa_proxy (innerkits)
    ↓ calls
services/storage_manager/
    ↓ IPC
services/storage_daemon/
```

**证据来源**：`bundle.json:39-88`
