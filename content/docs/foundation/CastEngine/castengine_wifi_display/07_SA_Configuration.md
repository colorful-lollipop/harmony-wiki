# SA 配置与服务

## System Ability 概述

castengine_wifi_display 提供了两个 System Ability (SA)：

| SA ID | 进程 | 用途 |
|-------|------|------|
| 5527 | sharing_service | WFD Sink 能力 |
| 5528 | sharing_service | WFD Source 能力 |

## SA 配置文件

### 5527.json

**文件位置**: `sa_profile/5527.json`

```json
{
    "process": "sharing_service",
    "systemability": [
        {
            "name": 5527,
            "libpath": "libsharing_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": []
        }
    ]
}
```

### 5528.json

**文件位置**: `sa_profile/5528.json`

```json
{
    "process": "sharing_service",
    "systemability": [
        {
            "name": 5528,
            "libpath": "libsharing_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": []
        }
    ]
}
```

### 配置文件字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | 5527 / 5528 | SA ID |
| `process` | sharing_service | 所属进程 |
| `libpath` | libsharing_service.z.so | 实现库路径 |
| `run-on-create` | true | 按需启动 |
| `distributed` | false | 非分布式 SA |
| `dump_level` | 1 | dump 级别 |

## 服务进程配置

**文件位置**: `services/etc/sharing_service.cfg`

```json
{
    "jobs" : [{
        "name" : "services:sharing_service",
        "cmds" : [
            "mkdir /data/service/el1/public/database/sharingcodec 0777 audio audio",
            "mkdir /data/service/el1/public/database/sharingcodec/cache 0777 audio audio",
            "mkdir /data/service/el1/public/database/sharingcodec/meta 0777 audio audio",
            "mkdir /data/service/el1/public/database/sharingcodec/meta/backup 0777 audio audio",
            "mkdir /data/service/el1/public/database/sharingcodec/kvdb 0777 audio audio",
            "mkdir /data/service/el1/public/database/sharingcodec/key 0777 audio audio",
            "mkdir /data/service/el1/public/sharing_service 0700 sharing_service sharing_service",
            "syncexec /system/bin/chmod 0711 /data/service/el1/public/database",
            "syncexec /system/bin/chown -R ddms:ddms /data/service/el1/public/database/sharingcodec/meta",
            "syncexec /system/bin/chmod -R 2770 /data/service/el1/public/database/sharingcodec/meta"
        ]
    }],
    "services" : [{
        "name" : "sharing_service",
        "path" : ["/system/bin/sa_main", "/system/profile/sharing_service.json"],
        "uid" : "audio",
        "gid" : ["system", "audio", "root", "vendor_mpp_driver"],
        "ondemand" : false,
        "apl" : "system_basic",
        "permission" : [
            "ohos.permission.DISTRIBUTED_DATASYNC",
            "ohos.permission.DISTRIBUTED_SOFTBUS_CENTER",
            "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
            "ohos.permission.CAMERA",
            "ohos.permission.MICROPHONE",
            "ohos.permission.ACCESS_SERVICE_DM",
            "ohos.permission.CAPTURE_SCREEN",
            "ohos.permission.GET_WIFI_PEERS_MAC",
            "ohos.permission.GET_WIFI_INFO",
            "ohos.permission.SET_WIFI_INFO",
            "ohos.permission.GET_WIFI_LOCAL_MAC",
            "ohos.permission.ACCESS_CAST_ENGINE_MIRROR"
        ],
        "permission_acls" : ["ohos.permission.CAPTURE_SCREEN"],
        "secon" : "u:r:sharing_service:s0"
    }]
}
```

### 进程配置字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | sharing_service | 服务名称 |
| `uid` | audio | 用户 ID |
| `gid` | system, audio, root, vendor_mpp_driver | 组 ID |
| `ondemand` | false | 非按需启动 |
| `apl` | system_basic | 权限级别 |
| `secon` | u:r:sharing_service:s0 | SELinux 上下文 |

### 权限列表

| 权限 | 用途 | 敏感度 |
|------|------|--------|
| `DISTRIBUTED_DATASYNC` | 分布式数据同步 | 高 |
| `DISTRIBUTED_SOFTBUS_CENTER` | 软总线中心 | 高 |
| `GET_BUNDLE_INFO_PRIVILEGED` | 获取包信息（特权） | 高 |
| `CAMERA` | 相机访问 | 高 |
| `MICROPHONE` | 麦克风访问 | 高 |
| `ACCESS_SERVICE_DM` | 访问设备管理 | 高 |
| `CAPTURE_SCREEN` | 屏幕捕获 | 高 |
| `GET_WIFI_PEERS_MAC` | 获取 WiFi 对等设备 MAC | 高 |
| `GET_WIFI_INFO` | 获取 WiFi 信息 | 中 |
| `SET_WIFI_INFO` | 设置 WiFi 信息 | 高 |
| `GET_WIFI_LOCAL_MAC` | 获取本地 WiFi MAC | 高 |
| `ACCESS_CAST_ENGINE_MIRROR` | 访问投屏引擎镜像 | 高 |

### ACL 权限

| 权限 | 说明 |
|------|------|
| `ohos.permission.CAPTURE_SCREEN` | 屏幕捕获权限可通过 ACL 授予 |

## SELinux 配置

**SELinux 上下文**: `u:r:sharing_service:s0`

**配置文件**: `services/etc/sharing_service.cfg:43`

## 相关文档

- [GN 构建配置](05_GN_Build.md)
- [编译产物](06_Build_Artifacts.md)
- [安全风险评审](08_Security_Review.md)
