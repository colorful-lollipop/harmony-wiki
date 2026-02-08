# GN 构建配置

## 文档信息

| 项目 | 内容 |
|------|------|
| 目标读者 | 构建开发者、系统集成者 |
| 目的 | 理解项目的 GN 构建配置 |
| 前置知识 | GN 构建系统基础 |
| 代码证据 | `BUILD.gn`、`services/BUILD.gn` 等 |

## 构建文件概览

### 主要构建文件

| 文件路径 | 职责 |
|----------|------|
| `BUILD.gn` | 根构建入口，定义顶层 targets |
| `bundle.json` | 组件声明文件 |
| `distributedfile.gni` | 分布式文件通用配置 |
| `services/BUILD.gn` | 服务层构建配置 |
| `interfaces/*/BUILD.gn` | 接口层构建配置 |

### 子目录 BUILD.gn 统计

| 子目录 | 数量 | 说明 |
|--------|------|------|
| `services/` | 6 | 各服务模块构建 |
| `interfaces/` | 5 | 接口层构建 |
| `utils/` | 2 | 工具库构建 |
| `adapter/` | 1 | 适配器构建 |

---

## 根目录 BUILD.gn

**文件**：`BUILD.gn`

### 顶层 Group Targets

| Target | 类型 | 职责 |
|--------|------|------|
| `services_target` | group | 主服务入口，聚合所有服务 |
| `cloudsync_kit_inner_target` | group | 云同步 Inner Kit |
| `cloud_daemon_kit_inner_target` | group | 云守护 Inner Kit |
| `cloud_file_kit_inner_target` | group | 云文件 Inner Kit |
| `clouddiskservice_kit_inner_target` | group | 云盘服务 Inner Kit |
| `cloudsync_asset_kit_inner_target` | group | 云同步资源 Inner Kit |
| `distributed_file_daemon_kit_inner_target` | group | 分布式文件守护 Inner Kit |

**services_target 依赖配置**：

```gn
// BUILD.gn:17-44
group("services_target") {
  deps = [
    "${services_path}:cloudsyncservice.para",
    "${services_path}:cloudsyncservice.para.dac",
    "${services_path}:distributed_file.para",
    "${services_path}:distributedfile_etc",
    "${services_path}:distributedfile_sa_profile",
    "${services_path}/clouddisk_database:clouddisk_database",
    "${services_path}/cloudfiledaemon:cloudfiledaemon",
    "${services_path}/cloudsyncservice:cloudsync_sa",
  ]

  if (dfs_service_feature_enable_dist_file_daemon && dfs_service_feature_enable_distributed_ability) {
    deps += [ "${services_path}/distributedfiledaemon:libdistributedfiledaemon" ]
  }
  if (dfs_service_feature_enable_cloud_adapter) {
    deps += [ "${services_path}:cloudfiledaemon_etc" ]
  }
  if (dfs_service_feature_enable_cloud_disk) {
    deps += [
      "${services_path}:clouddiskservice_sa_profile",
      "${services_path}/clouddiskservice:clouddiskservice_sa",
      "${services_path}:clouddiskservice_etc",
      "${services_path}/clouddiskservice/seccomp_policy:disk_monitor_seccomp_filter",
    ]
  }
}
```

---

## services/ 构建配置

**文件**：`services/BUILD.gn`

### SA Profile 配置

| Target | 类型 | 输入文件 | 说明 |
|--------|------|----------|------|
| `distributedfile_sa_profile` | ohos_sa_profile | `5204.json` [+5201.json, +5205.json] | 分布式文件 SA |
| `clouddiskservice_sa_profile` | ohos_sa_profile | `5207.json` | 云盘服务 SA |

### 预置配置文件

| Target | 类型 | 输入 | 安装路径 |
|--------|------|------|----------|
| `distributedfile_etc` | ohos_prebuilt_etc | `distributedfile.cfg` | `/system/etc/init/` |
| `clouddiskservice_etc` | ohos_prebuilt_etc | `clouddiskservice.cfg` | `/system/etc/init/` |
| `cloudfiledaemon_etc` | ohos_prebuilt_etc | `cloudfiledaemon.cfg` | `/system/etc/init/` |
| `distributed_file.para` | ohos_prebuilt_etc | `distributed_file.para` | `/system/etc/param/` |
| `cloudsyncservice.para` | ohos_prebuilt_etc | `cloudsyncservice.para` | `/system/etc/param/` |
| `cloudsyncservice.para.dac` | ohos_prebuilt_etc | `cloudsyncservice.para.dac` | `/system/etc/param/` |

---

## services/distributedfiledaemon/ 构建

**文件**：`services/distributedfiledaemon/BUILD.gn`

### 主库 Target

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libdistributedfiledaemon` | ohos_shared_library | `libdistributedfiledaemon.z.so` | 分布式文件守护主库 |

**关键配置**：

```gn
// BUILD.gn:16-163
ohos_shared_library("libdistributedfiledaemon") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
  }
  include_dirs = [
    "include",
    "include/network/softbus",
    "${distributedfile_path}/frameworks/native/distributed_file_inner/include",
  ]
  defines = [
    "LOG_DOMAIN=0xD00430B",
    "LOG_TAG=\"distributedfile_daemon\"",
    "DFS_ENABLE_RADAR",
  ]
  external_deps = [
    "app_file_service:fileuri_native",
    "ability_base:want",
    "access_token:libaccesstoken_sdk",
    "dsoftbus:softbus_client",
    "hilog:libhilog",
    "ipc:ipc_single",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    // ... 更多
  ]
}
```

### Inner Kit Target

| Target | 类型 | 说明 |
|--------|------|------|
| `distributed_file_daemon_kit_inner` | ohos_shared_library | Inner API 库 |

**关键 Sources**：

- IPC：`src/ipc/daemon.cpp`、`src/ipc/daemon_stub.cpp`
- 网络：`src/network/softbus/softbus_handler.cpp`
- 挂载：`src/mountpoint/mount_manager.cpp`
- 通道：`src/channel_manager/channel_manager.cpp`

---

## services/cloudsyncservice/ 构建

**文件**：`services/cloudsyncservice/BUILD.gn`

### 主服务 Target

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cloudsync_sa` | ohos_shared_library | `libcloudsync_sa.z.so` | 云同步服务 |
| `cloudsync_sa_static` | ohos_static_library | `.a` | 静态库 |

**关键配置**：

```gn
// BUILD.gn:101-222
ohos_shared_library("cloudsync_sa") {
  branch_protector_ret = "pac_ret"
  include_dirs = [
    "include",
    "include/cycle_task",
    "include/transport/softbus",
    "${innerkits_native_path}/cloudsync_kit_inner",
  ]
  defines = [
    "LOG_DOMAIN=0xD004307",
    "LOG_TAG=\"CLOUDSYNC_SA\"",
  ]
  deps = [
    ":cloud_sync_service_interface",
    "${clouddisk_database_path}:clouddisk_database",
    "${innerkits_native_path}/cloud_file_kit_inner:cloudfile_kit",
  ]
}
```

### 周期任务模块

| 模块 | 源文件 |
|------|--------|
| 周期任务 | `src/cycle_task/cycle_task.cpp` |
| 任务运行器 | `src/cycle_task/cycle_task_runner.cpp` |
| 优化缓存任务 | `src/cycle_task/tasks/optimize_cache_task.cpp` |
| 优化存储任务 | `src/cycle_task/tasks/optimize_storage_task.cpp` |
| 定期检查任务 | `src/cycle_task/tasks/periodic_check_task.cpp` |
| 定期清理任务 | `src/cycle_task/tasks/periodic_clean_task.cpp` |
| 报告统计任务 | `src/cycle_task/tasks/report_statistics_task.cpp` |
| 保存订阅任务 | `src/cycle_task/tasks/save_subscription_task.cpp` |
| 数据库备份任务 | `src/cycle_task/tasks/database_backup_task.cpp` |

### 传输模块

| 模块 | 源文件 |
|------|--------|
| 文件传输管理器 | `src/transport/file_transfer_manager.cpp` |
| 消息处理器 | `src/transport/message_handler.cpp` |
| 会话管理器 | `src/transport/softbus/session_manager.cpp` |
| SoftBus 适配器 | `src/transport/softbus/softbus_adapter.cpp` |
| SoftBus 会话 | `src/transport/softbus/softbus_session.cpp` |

---

## interfaces/kits/js/cloudfilesync/ 构建

**文件**：`interfaces/kits/js/cloudfilesync/BUILD.gn`

### N-API Target

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cloudsync` | ohos_shared_library | `libcloudsync.ndk.so` | JS 云同步 N-API |

**关键配置**：

```gn
// BUILD.gn:29-78
ohos_shared_library("cloudsync") {
  branch_protector_ret = "pac_ret"
  configs = [ ":optimize-size" ]
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
  }
  sources = [
    "cloud_file_cache_napi.cpp",
    "cloud_file_napi.cpp",
    "cloud_file_version_napi.cpp",
    "cloud_sync_n_exporter.cpp",
    "cloud_sync_napi.cpp",
    "file_sync_napi.cpp",
    "gallery_sync_napi.cpp",
    "multi_download_progress_napi.cpp",
  ]
  deps = [
    "${innerkits_native_path}/cloudsync_kit_inner:cloudsync_kit_inner",
    "${utils_path}:libdistributedfileutils",
  ]
  external_deps = [
    "ability_runtime:dataobs_manager",
    "hilog:libhilog",
    "ipc:ipc_single",
    "napi:ace_napi",
  ]
  defines = [
    "LOG_DOMAIN=0xD004309",
    "LOG_TAG=\"CLOUD_FILE_SYNC\"",
  ]
  relative_install_dir = "module/file"
}
```

---

## 关键依赖

### 外部组件依赖

| 组件 | 用途 |
|------|------|
| `ability_base` | 基础能力 |
| `ability_runtime` | 运行时能力 |
| `access_token` | 访问控制 |
| `ipc` | 进程间通信 |
| `napi` | Node API |
| `hilog` | 日志系统 |
| `safwk` | 系统能力框架 |
| `samgr` | 服务管理 |
| `dsoftbus` | 分布式软总线 |
| `libfuse` | FUSE 文件系统 |
| `ffrt` | 函数运行时 |

### 内部模块依赖

| 依赖路径 | 用途 |
|----------|------|
| `cloudsync_kit_inner` | 云同步 Inner Kit |
| `cloud_file_kit_inner` | 云文件 Inner Kit |
| `clouddisk_database` | 云盘数据库 |
| `libdistributedfileutils` | 分布式文件工具 |
| `libdistributedfiledentry` | 分布式文件目录项 |

---

## 编译开关

### Feature Flags

| 开关 | 位置 | 默认值 | 影响 |
|------|------|--------|------|
| `dfs_service_feature_enable_cloud_adapter` | BUILD.gn | - | 云适配器编译 |
| `dfs_service_feature_enable_cloud_disk` | BUILD.gn | - | 云盘功能编译 |
| `dfs_service_feature_enable_dist_file_daemon` | BUILD.gn | - | 分布式文件守护编译 |
| `dfs_service_feature_enable_distributed_ability` | BUILD.gn | - | 分布式能力编译 |

### 条件编译示例

```gn
// BUILD.gn:29-32
if (dfs_service_feature_enable_dist_file_daemon && dfs_service_feature_enable_distributed_ability) {
  deps += [ "${services_path}/distributedfiledaemon:libdistributedfiledaemon" ]
}
```

---

## 相关跳转

- 编译产物：[05_Artifacts.md](./05_Artifacts.md)
- 架构设计：[01_Architecture.md](./01_Architecture.md)
- 对外接口：[02_N-API.md](./02_N-API.md)
