# 内部 API 文档

> 本文档描述 storage_service 的内部 C++ API，包括 Inner API 模块职责、依赖方向和关键接口定义。

## Inner API 概述

Inner API 是供 storage_service 内部组件使用的 C++ 接口，不对外暴露给应用开发者。

### 目录结构

```
interfaces/innerkits/
├── storage_manager/native/          # 存储管理 Inner API
│   ├── bundle_stats.h               # 应用存储统计
│   ├── disk.h                       # 磁盘信息
│   ├── ext_bundle_stats.h           # 扩展应用统计
│   ├── statistic_info.h             # 统计信息
│   ├── storage_stats.h              # 存储统计
│   ├── storage_service_errno.h      # 错误码 (18KB)
│   ├── storage_service_constants.h  # 常量定义
│   ├── storage_file_raw_data.h      # 文件原始数据
│   ├── userdata_dir_info.h          # 用户数据目录
│   ├── volume_core.h                # 卷核心结构
│   └── volume_external.h            # 外部卷接口
│
└── acl/native/                      # ACL Inner API
    └── storage_acl.h                 # ACL 权限控制
```

**证据来源**：`interfaces/innerkits/storage_manager/native/*.h`

## 核心数据结构

### VolumeCore

卷的核心数据结构，定义在 `volume_core.h`：

```cpp
struct VolumeCore {
    std::string id_;
    std::string uuid_;
    std::string diskId_;
    std::string description_;
    bool removable_;
    int state_;        // VolumeState 枚举
    std::string path_;
    std::string fsType_;
    int flags_;
};
```

**证据来源**：`interfaces/innerkits/storage_manager/native/volume_core.h`

### VolumeState

卷状态枚举：

| 值 | 状态 | 说明 |
|---|------|------|
| 0 | UNMOUNTED | 未挂载 |
| 1 | CHECKING | 检查中 |
| 2 | MOUNTED | 已挂满 |
| 3 | UNMOUNTING | 卸载中 |
| 4 | FAILED | 失败 |

**证据来源**：`interfaces/innerkits/storage_manager/native/volume_core.h`

### BundleStats

应用存储统计结构：

```cpp
struct BundleStats {
    int64_t appSize_;          // 应用大小
    int64_t dataSize_;         // 数据大小
    int64_t cacheSize_;        // 缓存大小
    int64_t diskCache_;        // 磁盘缓存
    int64_t total_;            // 总大小
};
```

**证据来源**：`interfaces/innerkits/storage_manager/native/bundle_stats.h`

### Disk

磁盘信息结构：

```cpp
struct Disk {
    std::string diskId_;
    std::string vendor_;
    std::string product_;
    std::string path_;
    int flags_;               // DISK_* 标志
};
```

**证据来源**：`interfaces/innerkits/storage_manager/native/disk.h`

## 模块职责

### 1. storage_manager 模块

> 提供卷、磁盘查询管理、多用户目录管理、空间统计。

| 子模块 | 目录 | 职责 |
|--------|------|------|
| IPC | `services/storage_manager/ipc/` | SA Provider、Stub、权限校验 |
| Volume | `services/storage_manager/volume/` | 卷管理业务逻辑 |
| Disk | `services/storage_manager/disk/` | 磁盘管理 |
| Storage | `services/storage_manager/storage/` | 存储统计 |
| Account | `services/storage_manager/account_subscriber/` | 账户订阅 |
| Communication | `services/storage_manager/storage_daemon_communication/` | 与 daemon IPC |

**证据来源**：`services/storage_manager/BUILD.gn`

### 2. storage_daemon 模块

> 提供分区挂载、内核交互、目录加解密。

| 子模块 | 目录 | 职责 |
|--------|------|------|
| IPC | `services/storage_daemon/ipc/` | SA Provider、Stub |
| User | `services/storage_daemon/user/` | 用户目录操作 |
| Volume | `services/storage_daemon/volume/` | 卷操作（挂载/卸载） |
| Disk | `services/storage_daemon/disk/` | 磁盘操作 |
| Crypto | `services/storage_daemon/crypto/` | 加密管理（FsCrypt/Huks） |
| File Sharing | `services/storage_daemon/file_sharing/` | 共享文件 |
| MTP | `services/storage_daemon/mtp/` | MTP 设备 |
| Netlink | `services/storage_daemon/netlink/` | 内核事件监听 |

**证据来源**：`services/storage_daemon/BUILD.gn`

## 依赖方向

### 模块依赖图

```
┌─────────────────────────────────────────────────────────┐
│                   interfaces/kits/js/                    │
│              (storageStatistics, volumeManager)          │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│              services/storage_manager/                   │
│  ┌─────────────────────────────────────────────────┐   │
│  │              StorageManagerProvider              │   │
│  │  (SA Provider + 业务逻辑 + 权限校验)              │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                            ↓ IPC
┌─────────────────────────────────────────────────────────┐
│              services/storage_manager/                   │
│      storage_daemon_communication (IPC 客户端)           │
└─────────────────────────────────────────────────────────┘
                            ↓ IPC
┌─────────────────────────────────────────────────────────┐
│              services/storage_daemon/                    │
│  ┌─────────────────────────────────────────────────┐   │
│  │             StorageDaemonProvider               │   │
│  │  (SA Provider + 底层操作)                        │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                         Kernel                          │
│               (Vold, Uevent, FsCrypt)                   │
└─────────────────────────────────────────────────────────┘
```

### 内部依赖

| 依赖方 | 被依赖方 | 依赖类型 |
|--------|---------|---------|
| `storage_manager` | `storage_common_utils` | public_deps |
| `storage_manager` | `storage_daemon_ipc` | deps |
| `storage_daemon` | `sdc` | deps |
| `storage_daemon` | `storage_common_utils` | deps |
| JS kits | `storage_manager_sa_proxy` | deps |

**证据来源**：`services/storage_manager/BUILD.gn`、`services/storage_daemon/BUILD.gn`

## 关键内部 API

### StorageManagerProvider

storage_manager 的 SA Provider，继承 `SystemAbility` 和 `StorageManagerStub`：

```cpp
class StorageManagerProvider : public SystemAbility,
                               public StorageManagerStub {
public:
    // 用户管理
    int32_t PrepareAddUser(int32_t userId);
    int32_t RemoveUser(int32_t userId);
    int32_t PrepareStartUser(int32_t userId);
    int32_t StopUser(int32_t userId);
    int32_t CompleteAddUser(int32_t userId);
    
    // 存储统计
    int32_t GetBundleStats(const std::string& pkgName, 
                           BundleStats& bundleStats,
                           int32_t appIndex = 0,
                           uint32_t statFlag = 0);
    int32_t GetUserStorageStats(int32_t userId, 
                                StorageStats& storageStats);
    
    // 卷管理
    int32_t Mount(const std::string& volumeId);
    int32_t Unmount(const std::string& volumeId);
    int32_t GetAllVolumes(std::vector<VolumeExternal>& volumes);
    int32_t Format(const std::string& volumeId, const std::string& fsType);
    int32_t Partition(const std::string& diskId, int32_t type);
    
    // 加密
    int32_t ActiveUserKey(int32_t userId, 
                         const std::vector<uint8_t>& token,
                         const std::vector<uint8_t>& secret);
    int32_t InactiveUserKey(int32_t userId);
    
    // 共享文件
    int32_t CreateShareFile(const std::vector<std::string>& srcPaths,
                            int32_t tokenId,
                            std::vector<std::string>& destPaths);
    int32_t DeleteShareFile(const std::vector<std::string>& paths,
                            int32_t tokenId);
};
```

**证据来源**：`services/storage_manager/include/ipc/storage_manager_provider.h`

### StorageDaemonProvider

storage_daemon 的 SA Provider，继承 `StorageDaemonStub`：

```cpp
class StorageDaemonProvider : public StorageDaemonStub {
public:
    // 用户目录
    int32_t PrepareUserDirs(int32_t userId, 
                            const std::vector<uint8_t>& token,
                            const std::vector<uint8_t>& secret);
    int32_t DestroyUserDirs(int32_t userId, bool isForce);
    int32_t StartUser(int32_t userId);
    int32_t StopUser(int32_t userId);
    int32_t CompleteAddUser(int32_t userId);
    
    // 卷操作
    int32_t Mount(const std::string& volumeId);
    int32_t Unmount(const std::string& volumeId);
    int32_t Check(const std::string& volumeId);
    int32_t Format(const std::string& volumeId, const std::string& fsType);
    int32_t Partition(const std::string& diskId, int32_t type);
    
    // 密钥管理
    int32_t InitGlobalKey();
    int32_t ActiveUserKey(int32_t userId,
                          const std::vector<uint8_t>& token,
                          const std::vector<uint8_t>& secret);
    int32_t InactiveUserKey(int32_t userId);
    
    // 配额
    int32_t SetBundleQuota(const std::string& bundleName,
                           const std::string& volumeId,
                           int64_t quota);
    int32_t GetOccupiedSpace(int32_t userId,
                             int64_t& occupiedSize);
};
```

**证据来源**：`services/storage_daemon/include/ipc/storage_daemon_provider.h`

## 权限校验

### 权限校验函数

| 函数 | 位置 | 权限 | 用途 |
|------|------|------|------|
| `CheckClientPermission()` | `storage_manager_provider.cpp:76-94` | `ohos.permission.STORAGE_MANAGER` 等 | 通用权限校验 |
| `CheckClientPermissionForCrypt()` | `storage_manager_provider.cpp:107-117` | `ohos.permission.STORAGE_MANAGER_CRYPT` | 加密权限 |
| `IsSystemApp()` | `storage_manager_provider.cpp:96-105` | - | 系统应用检查 |

### 关键 UID 常量

```cpp
constexpr pid_t ACCOUNT_UID = 3058;      // 账户服务
constexpr pid_t BACKUP_SA_UID = 1089;    // 备份 SA
constexpr pid_t FOUNDATION_UID = 5523;   // foundation 进程
constexpr pid_t DFS_UID = 1009;          // DFS 服务
constexpr pid_t ROOT_UID = 0;            // root
```

**证据来源**：`services/storage_manager/ipc/src/storage_manager_provider.cpp:58-65`

## 稳定性标注

### 稳定接口

| 接口 | 层级 | 稳定性依据 |
|------|------|-----------|
| Inner API (innerkits/) | Inner | 仅供内部组件使用 |
| IPC 接口 | Stable | IDL 定义，版本稳定 |

### 不稳定接口

| 接口/头文件 | 层级 | 风险说明 |
|-------------|------|---------|
| Test API | Internal | 测试用途，可能变更 |
| DFX 报告 | Internal | 诊断功能，可能变更 |
