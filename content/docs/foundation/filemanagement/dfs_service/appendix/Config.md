# 配置说明

## 文档信息

| 项目 | 内容 |
|------|------|
| 目标读者 | 系统配置者、开发者 |
| 目的 | 理解各配置文件的作用和格式 |

## 配置文件列表

| 文件 | 位置 | 格式 | 职责 |
|------|------|------|------|
| bundle.json | 根目录 | JSON | 组件声明 |
| distributedfile.gni | 根目录 | GNI | 通用配置 |
| 5201.json | services/ | JSON | SA 5201 配置 |
| 5204.json | services/ | JSON | SA 5204 配置 |
| 5205.json | services/ | JSON | SA 5205 配置 |
| 5207.json | services/ | JSON | SA 5207 配置 |
| distributedfile.cfg | services/ | JSON | 守护进程配置 |
| cloudfiledaemon.cfg | services/ | JSON | 云文件守护配置 |
| clouddiskservice.cfg | services/ | JSON | 云盘服务配置 |

---

## bundle.json

**路径**：`bundle.json`

**职责**：组件声明，定义子系统、组件、依赖、构建配置。

### 关键字段

```json
{
    "name": "@ohos/dfs_service",      // 组件名
    "version": "3.1",                  // 版本
    "subsystem": "filemanagement",     // 子系统
    "component": {
        "name": "dfs_service",        // 组件名
        "syscap": [...],              // 系统能力
        "features": [...],             // 功能开关
        "deps": {...},                 // 组件依赖
        "build": {...}                 // 构建配置
    }
}
```

### 系统能力

| 能力名 | 说明 |
|--------|------|
| SystemCapability.FileManagement.DistributedFileService.CloudSync.Core | 云同步核心能力 |
| SystemCapability.FileManagement.DistributedFileService.CloudSyncManager | 云同步管理能力 |

### 功能开关

| 开关 | 说明 |
|------|------|
| dfs_service_feature_enable_cloud_adapter | 启用云适配器 |
| dfs_service_feature_enable_cloud_disk | 启用云盘 |
| dfs_service_feature_enable_dist_file_daemon | 启用分布式文件守护 |
| dfs_service_feature_enable_distributed_ability | 启用分布式能力 |

---

## SA 配置文件（5201.json 等）

### 通用字段

| 字段 | 类型 | 说明 |
|------|------|------|
| process | string | 进程名 |
| systemability | array | SA 列表 |
| [].name | number | SA ID |
| [].libpath | string | 库路径 |
| [].run-on-create | boolean | 是否随系统启动 |
| [].distributed | boolean | 是否分布式 SA |
| [].start-on-demand | object | 按需启动配置 |

### 5201.json（DistributedFileDaemon）

```json
{
    "process": "distributedfiledaemon",
    "systemability": [{
        "name": 5201,
        "libpath": "libdistributedfiledaemon.z.so",
        "run-on-create": false,
        "depend": [4700],
        "distributed": true
    }]
}
```

### 5204.json（CloudSyncService）

```json
{
    "process": "cloudfileservice",
    "systemability": [{
        "name": 5204,
        "libpath": "libcloudsync_sa.z.so",
        "run-on-create": false,
        "start-on-demand": {
            "commonevent": [...],
            "timedevent": {...}
        }
    }]
}
```

---

## 服务配置文件（.cfg）

### distributedfile.cfg

**路径**：`services/distributedfile.cfg`

**职责**：定义分布式文件守护进程运行参数。

### 关键配置

| 字段 | 说明 |
|------|------|
| jobs | 启动时执行的作业 |
| [].name | 作业名 |
| [].cmds | 命令列表 |
| services | 服务配置列表 |
| [].name | 服务名 |
| [].path | 启动路径 |
| [].uid | 用户 ID |
| [].gid | 组 ID |
| [].caps | Linux 能力 |
| [].secon | SELinux 上下文 |
| [].apl | 权限等级 |
| [].ondemand | 是否按需启动 |
| [].permission | 权限列表 |

### 配置示例

```json
{
    "jobs": [{
        "name": "services:cloudfileservice",
        "cmds": [
            "mkdir /data/service/el1/public/cloudfile 0711 dfs dfs",
            "restorecon /data/service/el1/public/cloudfile"
        ]
    }],
    "services": [{
        "name": "distributedfiledaemon",
        "path": ["/system/bin/sa_main", "/system/profile/distributedfiledaemon.json"],
        "uid": "1009",
        "gid": ["system", "dfs", 1006, 1008, "log"],
        "caps": ["DAC_READ_SEARCH", "CHOWN", "NET_RAW"],
        "secon": "u:r:distributedfiledaemon:s0",
        "apl": "system_basic",
        "permission": [
            "ohos.permission.ACCESS_SERVICE_DP",
            "ohos.permission.DISTRIBUTED_DATASYNC"
        ]
    }]
}
```

---

## GNI 配置文件

### distributedfile.gni

**路径**：`distributedfile.gni`

**职责**：定义 GN 构建的通用变量和配置。

### 关键变量

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `support_same_account` | bool | `true` | 是否支持同账号设备互联 |
| `support_device_profile` | bool | `true` | 是否启用设备画像 |
| `cloudsync_service_power` | bool | 动态 | 电池状态同步 |
| `cloudsync_service_hicollie_enable` | bool | 动态 | 性能监控使能 |
| `dfs_service_feature_enable_cloud_adapter` | bool | `false` | 云适配器开关 |
| `dfs_service_feature_enable_cloud_disk` | bool | `false` | 云盘功能开关 |
| `dfs_service_feature_enable_dist_file_daemon` | bool | `true` | 分布式守护开关 |
| `dfs_service_feature_enable_distributed_ability` | bool | `true` | 分布式能力开关 |

### 路径变量

```gni
distributedfile_path = "//foundation/filemanagement/dfs_service"
services_path = "${distributedfile_path}/services"
utils_path = "${distributedfile_path}/utils"
clouddisk_database_path = "${services_path}/clouddisk_database"
```

### 条件编译示例

```gni
if (dfs_service_feature_enable_dist_file_daemon && dfs_service_feature_enable_distributed_ability) {
    deps += [ "${services_path}/distributedfiledaemon:libdistributedfiledaemon" ]
}
if (dfs_service_feature_enable_cloud_disk) {
    deps += [
      "${services_path}:clouddiskservice_sa_profile",
      "${services_path}/clouddiskservice:clouddiskservice_sa",
    ]
}
```

---

## 相关跳转

- 架构设计：[01_Architecture.md](../01_Architecture.md)
- 构建配置：[04_Build.md](../04_Build.md)
- 编译产物：[05_Artifacts.md](../05_Artifacts.md)
