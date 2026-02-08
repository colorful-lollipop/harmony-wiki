# 编译产物文档

> 本文档描述 storage_service 的编译产物，包括共享库、可执行文件、配置文件及其安装路径。

## 产物总览

### 产物分类

| 类别 | 产物类型 | 数量 | 说明 |
|------|---------|------|------|
| 共享库 (.so) | ohos_shared_library | 5+ | JS 接口库、服务库 |
| 可执行文件 | ohos_executable | 2 | 系统服务、客户端工具 |
| 配置文件 | ohos_prebuilt_etc | 10+ | 系统配置、参数 |

## 共享库 (.so)

### JS 接口库 (module/file/)

| 产物 | 来源 Target | 大小 (估计) | 说明 |
|------|-------------|-------------|------|
| `libstoragestatistics.so` | `storagestatistics` | ~200KB | 存储统计 N-API |
| `libvolumemanager.so` | `volumemanager` | ~300KB | 卷管理 N-API (+DFS) |
| `libkeymanager.so` | `keymanager` | ~150KB | 密钥管理 N-API |

### 服务库 (system/lib/)

| 产物 | 来源 Target | 说明 |
|------|-------------|------|
| `libstorage_manager.so` | `storage_manager` | Manager 服务核心库 |
| `libstorage_common_utils.so` | `storage_common_utils` | 公共工具库 |

**证据来源**：`interfaces/kits/js/storage_manager/BUILD.gn`、`services/storage_manager/BUILD.gn`

## 可执行文件

| 产物 | 来源 Target | 安装路径 | 说明 |
|------|-------------|----------|------|
| `storage_daemon` | `storage_daemon` | `system/bin/` | 存储守护进程系统服务 |
| `sdc` | `sdc` | `system/bin/` | 存储客户端工具 |

### storage_daemon

> 常驻系统服务，负责分区挂载、加密管理、内核交互。

**启动方式**：系统启动时由 init 拉起
**配置来源**：`storage_daemon.cfg`

### sdc (Storage Daemon Client)

> 客户端工具，用于查询和操作存储服务。

```bash
# 使用示例
sdc --help
```

**证据来源**：`services/storage_daemon/BUILD.gn`

## 配置文件

### 系统配置文件

| 产物 | 来源 Target | 安装路径 | 说明 |
|------|-------------|----------|------|
| `storage_daemon.cfg` | `storage_daemon_cfg` | `init/` | 守护进程启动配置 |
| `storage_manager_config.para` | `storage_manager_config.para` | `etc/param/` | Manager 参数 |
| `storage_manager_config.para.dac` | `storage_manager_config.para.dac` | `etc/param/` | DAC 权限参数 |
| `usb_config.para` | `usb_config.para` | `etc/param/` | USB 参数 |
| `usb_config.para.dac` | `usb_config.para.dac` | `etc/param/` | USB DAC 参数 |

### 存储配置

| 产物 | 来源 Target | 安装路径 | 说明 |
|------|-------------|----------|------|
| `storage_user_path.json` | `storage_daemon_user_path` | `storage_daemon/` | 用户路径配置 |
| `storage_mount_info.json` | `storage_daemon_mount_info` | `storage_daemon/` | 挂载信息 |
| `disk_config` | `storage_daemon_disk_config` | `storage_daemon/` | 磁盘配置 |

**证据来源**：`services/storage_manager/BUILD.gn`、`services/storage_daemon/BUILD.gn`

## 运行时加载关系

### 静态依赖

```
JS Application
    ↓ dlopen
libstoragestatistics.so
    ↓ deps
libvolumemanager.so
    ↓ deps
libkeymanager.so
    ↓ deps
libstorage_manager.so
    ↓ IPC
libstorage_common_utils.so
```

### 动态依赖

| 组件 | 加载方 | 加载时机 |
|------|--------|---------|
| `libstorage_manager.so` | `libstoragestatistics.so` | 首次调用 API |
| `libstorage_common_utils.so` | `libstorage_manager.so` | 服务启动 |
| `storage_daemon` | init | 系统启动 |
| `sdc` | shell | 手动执行 |

## 安装清单

### 系统分区

```
system/
├── bin/
│   ├── storage_daemon       # 守护进程
│   └── sdc                  # 客户端工具
├── lib/
│   ├── libstorage_manager.so
│   ├── libstorage_common_utils.so
│   ├── libstoragestatistics.so
│   ├── libvolumemanager.so
│   └── libkeymanager.so
├── etc/
│   └── param/
│       ├── storage_manager_config.para
│       ├── storage_manager_config.para.dac
│       ├── usb_config.para
│       └── usb_config.para.dac
├── init/
│   └── storage_daemon.cfg
└── storage_daemon/
    ├── storage_user_path.json
    ├── storage_mount_info.json
    └── disk_config
```

### 应用分区 (可选)

```
module/file/
├── libstoragestatistics.so
├── libvolumemanager.so
└── libkeymanager.so
```

**证据来源**：`interfaces/kits/js/storage_manager/BUILD.gn`、`services/storage_daemon/BUILD.gn`

## 版本与兼容性

### 产物版本

| 产物 | 版本 | 兼容性 |
|------|------|--------|
| 所有 .so | 3.1 | N-API v1 |
| storage_daemon | 3.1 | SA ID 固定 |

### 运行时依赖

| 依赖 | 最低版本 | 说明 |
|------|---------|------|
| OpenHarmony | 4.0+ | 系统版本 |
| napi | 1.0 | JS 绑定框架 |
| ipc | 1.0 | IPC 通信框架 |
| safwk | 1.0 | SA 框架 |
