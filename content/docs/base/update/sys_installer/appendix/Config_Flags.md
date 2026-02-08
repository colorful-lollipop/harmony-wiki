# 配置参数

## 目的

本文档汇总 `sys_installer` 的关键配置参数、宏定义和 Feature Flags。

## 适用范围

开发人员、系统集成人员。

## GN 配置参数

### sys_installer_default_cfg.gni

**文件**: `sys_installer_default_cfg.gni`

```gn
# 配置文件路径
declare_args() {
  # 可选的配置文件路径
  sys_installer_cfg_file = ""
  
  # 基础路径
  sys_installer_absolutely_path = "//base/update/sys_installer"
  
  # OHOS 配置特性开关
  sys_installer_feature_ohos_cfg = false
}
```

### Feature Flags

| Flag | 默认值 | 说明 |
|------|--------|------|
| `sys_installer_cfg_file` | "" | 自定义配置文件路径 |
| `sys_installer_feature_ohos_cfg` | false | OHOS 配置特性 |

## 编译宏定义

### 服务端宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `SYS_INSTALLER_SERVICE` | ipc_server/BUILD.gn | 编译服务端代码 |

### 客户端宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `SYS_INSTALLER_KITS` | ipc_client/BUILD.gn | 编译客户端代码 |

### 功能宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `SUPPORT_HVB` | module_update/BUILD.gn | 启用 HVB 支持 |
| `WITH_SELINUX` | module_update/service/BUILD.gn | 启用 SELinux 支持 |

## SA ID 定义

### 接口代码

**文件**: `interfaces/inner_api/include/sys_installer_sa_ipc_interface_code.h`

```cpp
enum SysInstallerInterfaceCode {
    // SA 4101 命令码
    SYS_INSTALLER_INIT = 1,
    START_UPDATE_PACKAGE_ZIP,
    SET_UPDATE_CALLBACK,
    GET_UPDATE_STATUS,
    // ...
};

enum ModuleUpdateInterfaceCode {
    // SA 4103 命令码
    INSTALL_MODULE_PACKAGE = 1,
    UNINSTALL_MODULE_PACKAGE,
    GET_MODULE_PACKAGE_INFO,
    REPORT_MODULE_UPDATE_STATUS,
    EXIT_MODULE_UPDATE,
    GET_HMP_VERSION_INFO,
    START_UPDATE_HMP_PACKAGE,
    GET_HMP_UPDATE_RESULT
};

enum SysInstallerCallbackInterfaceCode {
    UPDATE_RESULT = 1,
    STREAM_UPDATE_RESULT = 2,
};
```

### SA ID

| 名称 | 值 | 说明 |
|------|-----|------|
| SYS_INSTALLER_DISTRIBUTED_SERVICE_ID | 4101 | 系统安装器 SA |
| MODULE_UPDATE_SERVICE_ID | 4103 | 模块更新 SA |

## 权限常量

### 权限名称

**文件**: `frameworks/ipc_server/src/sys_installer_server.cpp`

```cpp
const std::string PERMISSION_UPDATE_SYSTEM = "ohos.permission.UPDATE_SYSTEM";
```

### UID/GID

| 名称 | 值 | 说明 |
|------|-----|------|
| USER_UPDATE_AUTHORITY | 6666 | Update 服务 UID |
| ROOT_UID | 0 | Root UID |

## 路径常量

### 模块更新路径

**文件**: `services/module_update/util/include/module_constants.h`

```cpp
// 模块更新根目录
const std::string MODULE_UPDATE_BASE_DIR = "/data/module_update/";

// 子目录
const std::string MODULE_UPDATE_PACKAGE_DIR = "/data/module_update_package/";
const std::string MODULE_ACTIVE_DIR = "/data/module_update/active/";
const std::string MODULE_BACKUP_DIR = "/data/module_update/backup/";
const std::string MODULE_PREINSTALL_DIR = "/system/module_update/";

// 日志路径
const std::string UPDATER_LOG_DIR = "/data/updater/log/";
```

## 错误码定义

### 系统错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_OK | 0 | 成功 |
| ERR_INVALID_VALUE | -1 | 无效参数 |
| ERR_INVALID_OPERATION | -2 | 无效操作 |
| ERR_PERMISSION_DENIED | -3 | 权限拒绝 |
| ERR_NOT_SUPPORTED | -4 | 不支持的操作 |

### 模块更新错误码

**文件**: `services/module_update/util/include/module_error_code.h`

```cpp
enum ModuleErrorCode {
    ERR_OK = 0,
    ERR_VERIFY_FAIL = 1,
    ERR_EXTRACT_FAIL = 2,
    ERR_MOUNT_FAIL = 3,
    ERR_DM_CREATE_FAIL = 4,
    ERR_LOOP_CREATE_FAIL = 5,
    ERR_FILE_OPERATION_FAIL = 6,
    ERR_HVB_VERIFY_FAIL = 7,
    ERR_INVALID_PACKAGE = 8,
    ERR_NO_SPACE = 9,
    ERR_TASK_QUEUE_FULL = 10,
    ERR_TASK_TIMEOUT = 11,
    ERR_ALREADY_INSTALLED = 12,
    ERR_NOT_INSTALLED = 13,
};
```

## 更新状态码

**文件**: `interfaces/innerkits/ipc_client/Types.idl`

```cpp
enum UpdateStatus {
    UPDATE_STATE_SUCCESS = 0,
    UPDATE_STATE_FAIL = 1,
    UPDATE_STATE_ONGOING = 2,
    UPDATE_STATE_SUB_SUCCESS = 3,
    UPDATE_STATE_SUB_FAIL = 4,
    UPDATE_STATE_ALREADY_UP_TO_DATE = 5
};
```

## 配置文件

### sys_installer.cfg

**文件**: `frameworks/ipc_server/etc/sys_installer.cfg`

```json
{
    "name": "sys_installer_sa",
    "uid": "root",
    "gid": ["update", "system", "root"],
    "permission": [
        "ohos.permission.DEVICE_STANDBY_EXEMPTION",
        "ohos.permission.RUNNING_LOCK"
    ],
    "secon": "u:r:sys_installer_sa:s0"
}
```

### module_update_sa.cfg

**文件**: `frameworks/ipc_server/etc/module_update_sa.cfg`

```json
{
    "name": "module_update_sa",
    "uid": "update",
    "gid": ["update", "system", "root"],
    "permission": [
        "ohos.permission.READ_DFX_SYSEVENT"
    ],
    "secon": "u:r:module_update_service:s0"
}
```

### SA Profile

**文件**: `frameworks/ipc_server/sa_profile/4101.json`

```json
{
    "process": "sys_installer_sa",
    "name": "sys_installer_sa",
    "saId": 4101,
    "libpath": "libsys_installer.z.so",
    "run-on-create": false,
    "distributed": false,
    "dump-level": 1
}
```

**文件**: `frameworks/ipc_server/sa_profile/4103.json`

```json
{
    "process": "module_update_sa",
    "name": "module_update_sa",
    "saId": 4103,
    "libpath": "libmodule_update_service.z.so",
    "run-on-create": false,
    "start-on-demand": {
        "persist.samgr.moduleupdate.start": "true",
        "persist.moduleupdate.bms.scan": "revert"
    },
    "stop-on-demand": {
        "bootevent.boot.completed": "true"
    }
}
```

## 系统参数

### sys_installer.para

**文件**: `frameworks/ipc_server/etc/sys_installer.para`

```
persist.samgr.moduleupdate.start = false
persist.moduleupdate.bms.scan = normal
```

## 相关链接

- [GN Targets](../05_GN_Targets.md)
- [编译产物](../07_Build_Products.md)
- [安全风险分析](../06_Security_Analysis.md)

---

*证据来源*:
- `sys_installer_default_cfg.gni`: GN 配置
- `interfaces/inner_api/include/sys_installer_sa_ipc_interface_code.h`: SA ID
- `services/module_update/util/include/module_constants.h`: 路径常量
- `services/module_update/util/include/module_error_code.h`: 错误码
- `frameworks/ipc_server/etc/`: 配置文件
