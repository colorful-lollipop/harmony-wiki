# GN 构建系统

---

## 目的

本文档说明 DBMS 项目的 GN 构建系统配置，包括所有 targets、依赖关系、编译产物和 feature flags。

---

## 适用范围

- ✅ 所有非测试 BUILD.gn 文件
- ✅ Target 详细信息
- ✅ 依赖关系图
- ✅ 编译产物清单
- ❌ 测试相关 targets（排除）

---

## BUILD.gn 文件清单

| 序号 | 文件路径 | 作用 |
|------|----------|------|
| 1 | `BUILD.gn` | 根构建入口，定义 4 个顶层 group targets |
| 2 | `services/dbms/BUILD.gn` | 核心服务共享库 libdbms 构建 |
| 3 | `interfaces/inner_api/BUILD.gn` | 内部 API 框架库 dbms_fwk 构建 |
| 4 | `interfaces/kits/js/distributedBundle/BUILD.gn` | JS N-API 接口构建 |
| 5 | `interfaces/kits/js/distributebundlemgr/BUILD.gn` | JS N-API 旧版接口构建 |
| 6 | `interfaces/kits/ani/distributed_bundle_manager/BUILD.gn` | ANI 接口和 ABC 字节码生成 |
| 7 | `services/dbms/sa_profile/BUILD.gn` | SA 配置文件和启动脚本 |

---

## .gni 配置文件

### dbms.gni

**路径**: `/Volumes/lexar/code/d/work/oh/foundation/bundlemanager/distributed_bundle_framework/dbms.gni`

**路径定义**:
```gn
bundlemanager_path = "//foundation/bundlemanager"
bundle_framework_path = "${bundlemanager_path}/bundle_framework"
dbms_inner_api_path = "${bundlemanager_path}/distributed_bundle_framework/interfaces/inner_api"
dbms_services_path = "${bundlemanager_path}/distributed_bundle_framework/services/dbms"
dbms_kits_path = "${bundlemanager_path}/distributed_bundle_framework/interfaces/kits"
```

**Feature Flags**:

| Flag | 默认值 | 说明 |
|------|--------|------|
| `distributed_bundle_framework_graphics` | true | 图形/显示功能开关 |
| `ability_runtime_enable_dbms` | true | Ability 运行时集成 |
| `account_enable_dbms` | true | 账号系统集成 |
| `distributed_bundle_framework_enable` | true | 分布式 Bundle 框架总开关 |
| `hisysevent_enable_dbms` | true | HiSysEvent 事件上报 |
| `distributed_bundle_image_framework_enable` | true | 图片框架支持 |

**证据**: `dbms.gni:28-59`

---

## Target 详细分析

### 1. 根 BUILD.gn Targets

| Target | 类型 | 依赖 |
|--------|------|------|
| `ani_dbms_packages` | group | `interfaces/kits/ani/distributed_bundle_manager:ani_dbms_packages` |
| `inner_api_target` | group | `interfaces/inner_api:dbms_fwk` |
| `jsapi_target` | group | `interfaces/kits/js/distributebundlemgr:jsapi_target`, `interfaces/kits/js/distributedBundle:jsapi_target` |
| `dbms_target` | group | `services/dbms:dbms_target`, `services/dbms/sa_profile:distributedbms` |

**证据**: `BUILD.gn:16-36`

### 2. services/dbms/libdbms

**类型**: ohos_shared_library

**输出**: `libdbms.z.so`

**Sources** (5 个):
- `src/account_manager_helper.cpp`
- `src/dbms_device_manager.cpp`
- `src/distributed_bms.cpp`
- `src/distributed_bms_host.cpp`
- `src/distributed_data_storage.cpp`
- `src/event_report.cpp` (条件)
- `src/image_compress.cpp` (条件)

**External Deps** (最多 22 个):
```
ability_base:want
access_token:libaccesstoken_sdk
access_token:libtokenid_sdk
bundle_framework:appexecfwk_base
bundle_framework:appexecfwk_core
bundle_framework:libappexecfwk_common
c_utils:utils
common_event_service:cesfwk_innerkits
device_manager:devicemanagersdk
hicollie:libhicollie
hilog:libhilog
i18n:intl_util
init:libbegetutil
ipc:ipc_core
kv_store:distributeddata_inner
resource_management:global_resmgr
safwk:system_ability_fwk
samgr:samgr_proxy
```

**证据**: `services/dbms/BUILD.gn:27-105`

### 3. interfaces/inner_api/dbms_fwk

**类型**: ohos_shared_library

**输出**: `libdbms_fwk.z.so`

**innerapi_tags**: ["platformsdk"]

**Sources**:
- `src/distributed_bms_acl_info.cpp`
- `src/distributed_bms_proxy.cpp`

**证据**: `interfaces/inner_api/BUILD.gn:21-64`

### 4. interfaces/kits/js/distributedBundle

**Targets**:

| Target | 类型 | 输出 |
|--------|------|------|
| `distributed_bundle_common` | ohos_shared_library | `libdistributed_bundle_common.z.so` |
| `distributedbundlemanager` | ohos_shared_library | `libdistributedbundlemanager.z.so` |
| `jsapi_target` | group | 条件编译 |

**证据**: `interfaces/kits/js/distributedBundle/BUILD.gn:17-125`

### 5. interfaces/kits/js/distributebundlemgr

**Targets**:

| Target | 类型 | 输出 |
|--------|------|------|
| `distributedbundle` | ohos_shared_library | `libdistributedbundle.z.so` |
| `jsapi_target` | group | 条件编译 |

**证据**: `interfaces/kits/js/distributebundlemgr/BUILD.gn` (需要读取)

### 6. interfaces/kits/ani/distributed_bundle_manager

**Targets**:

| Target | 类型 | 输出 |
|--------|------|------|
| `ani_distributed_bundle_manager` | ohos_shared_library | `libani_distributed_bundle_manager.z.so` |
| `distributed_bundle_manager` | generate_static_abc | `distributed_bundle_manager.abc` |
| `remote_ability_info` | generate_static_abc | `remote_ability_info.abc` |
| `distributed_bundle_manager_etc` | ohos_prebuilt_etc | `/system/framework/` |
| `remote_ability_info_etc` | ohos_prebuilt_etc | `/system/framework/` |
| `ani_dbms_packages` | group | 条件编译 |

**证据**: `interfaces/kits/ani/distributed_bundle_manager/BUILD.gn:17-92`

### 7. services/dbms/sa_profile

**Targets**:

| Target | 类型 | 输出 |
|--------|------|------|
| `distributedbms_sa_profile` | ohos_sa_profile | SA 配置 |
| `distributedbms.cfg` | ohos_prebuilt_etc | 启动配置 |

**证据**: `services/dbms/sa_profile/BUILD.gn`

---

## 编译产物清单

### 主要库文件

| Target | 输出文件名 | 类型 | 安装路径 | 用途 |
|--------|-----------|------|----------|------|
| libdbms | `libdbms.z.so` | shared_library (SA) | system/lib | 系统服务主库 |
| dbms_fwk | `libdbms_fwk.z.so` | shared_library | system/lib | 内部 API 框架 |
| distributed_bundle_common | `libdistributed_bundle_common.z.so` | shared_library | system/lib | JS N-API 通用库 |
| distributedbundlemanager | `libdistributedbundlemanager.z.so` | shared_library | system/lib/module/bundle | JS N-API 新版模块 |
| distributedbundle | `libdistributedbundle.z.so` | shared_library | system/lib/module | JS N-API 旧版模块 |
| ani_distributed_bundle_manager | `libani_distributed_bundle_manager.z.so` | shared_library (ANI) | system/lib | ArkTS 接口库 |

### ABC 字节码文件

| Target | 输出文件名 | 安装路径 |
|--------|-----------|----------|
| distributed_bundle_manager | `distributed_bundle_manager.abc` | system/framework | 主模块字节码 |
| remote_ability_info | `remote_ability_info.abc` | system/framework | 数据结构字节码 |

### 配置文件

| Target | 输出文件名 | 安装路径 | 用途 |
|--------|-----------|----------|------|
| distributedbms_sa_profile | `402.json` | system/profile | SA 能力配置 |
| distributedbms.cfg | `distributedbms.cfg` | system/etc/init | 启动配置 |

---

## 依赖关系图

```
根 BUILD.gn
    │
    ├─→ ani_dbms_packages
    │       └─→ interfaces/kits/ani/distributed_bundle_manager:ani_dbms_packages
    │               ├─→ ani_distributed_bundle_manager
    │               │       └→ interfaces/inner_api:dbms_fwk
    │               └─→ distributed_bundle_manager (ABC 字节码）
    ├─→ inner_api_target
    │       └─→ interfaces/inner_api:dbms_fwk
    │               └─→ libdbms_fwk.z.so
    ├─→ jsapi_target
    │       ├─→ interfaces/kits/js/distributebundlemgr:jsapi_target
    │       │       └─→ distributedbundle
    │       └─→ interfaces/kits/js/distributedBundle:jsapi_target
    │               ├─→ distributed_bundle_common
    │               │       └─→ interfaces/inner_api:dbms_fwk
    │               └─→ distributedbundlemanager
    │                       └─→ interfaces/inner_api:dbms_fwk
    └─→ dbms_target
            ├─→ services/dbms:dbms_target
            │       └─→ libdbms.z.so
            │               ├─→ interfaces/inner_api:dbms_fwk
            │               └─→ 外部 SA (bundle_framework, device_manager, 等）
            └─→ services/dbms/sa_profile:distributedbms
                    └─→ SA 配置文件
```

---

## 条件编译

### Feature Flags 影响矩阵

| Flag | 影响的 Source | 影响的 Deps | 定义的宏 |
|------|-------------|-----------|---------|
| `hisysevent_enable_dbms` | event_report.cpp | hisysevent:libhisysevent | HISYSEVENT_ENABLE |
| `account_enable_dbms` | - | os_account:libaccountkits, os_account:os_account_innerkits | ACCOUNT_ENABLE |
| `distributed_bundle_image_framework_enable` | image_compress.cpp | image_framework:image_native | DISTRIBUTED_BUNDLE_IMAGE_ENABLE |
| `distributed_bundle_framework_enable` | distributed_bundle.cpp / distributed_bundle_unsupported.cpp | - | - |
| `distributed_bundle_framework_graphics` | 整个 jsapi_target 包含的库 | - | - |

**证据**: `dbms.gni:28-59`, `services/dbms/BUILD.gn:84-100`

---

## 构建产物推导

### 预期输出路径

| Target | 输出路径 | 说明 |
|--------|----------|------|
| libdbms.z.so | `${out_dir}/system/lib/libdbms.z.so` | SA 共享库 |
| libdbms_fwk.z.so | `${out_dir}/system/lib/libdbms_fwk.z.so` | 平台 SDK 库 |
| libdistributed_bundle_common.z.so | `${out_dir}/system/lib/libdistributed_bundle_common.z.so` | N-API 通用库 |
| libdistributedbundlemanager.z.so | `${out_dir}/system/lib/module/bundle/libdistributedbundlemanager.z.so` | JS N-API 模块 |
| libdistributedbundle.z.so | `${out_dir}/system/lib/module/libdistributedbundle.z.so` | JS N-API 模块 |
| libani_distributed_bundle_manager.z.so | `${out_dir}/system/lib/libani_distributed_bundle_manager.z.so` | ANI 库 |
| distributed_bundle_manager.abc | `${out_dir}/system/framework/distributed_bundle_manager.abc` | ABC 字节码 |
| 402.json | `${out_dir}/system/profile/402.json` | SA 配置 |
| distributedbms.cfg | `${out_dir}/system/etc/init/distributedbms.cfg` | 启动配置 |

---

## SA 配置详解

### SA 配置 (402.json)

**文件**: `services/dbms/sa_profile/402.json`

**配置**:
```json
{
    "process": "d-bms",
    "systemability": [{
        "name": 402,
        "libpath": "libdbms.z.so",
        "run-on-create": false,
        "distributed": true,
        "start-on-demand": {
            "deviceonline": [{"name": "deviceonline", "value": "on"}]
        },
        "stop-on-demand": {
            "deviceonline": [{"name": "deviceonline", "value": "off"}]
        }
    }]
}
```

**证据**: `services/dbms/sa_profile/402.json:1-28`

### 启动配置 (distributedbms.cfg)

**文件**: `services/dbms/sa_profile/distributedbms.cfg`

**配置**:
```cfg
jobs : [{
    name: "services:d-bms",
    cmds: [
        "mkdir /data/service/el1/public/database 0711 ddms ddms",
        "mkdir /data/service/el1/public/database/bundle_manager_service 02770 dbms ddms"
    ]
}]

services: [{
    name: "d-bms",
    path: ["/system/bin/sa_main", "/system/profile/d-bms.json"],
    ondemand: true,
    uid: "dbms",
    gid: ["dbms", "shell"],
    permission: [
        "ohos.permission.DISTRIBUTED_DATASYNC",
        "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
        "ohos.permission.GET_INSTALLED_BUNDLE_LIST",
        "ohos.permission.ACCESS_SERVICE_DM",
        "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
        "ohos.permission.GET_BUNDLE_RESOURCES"
    ],
    start-mode: "condition",
}]
```

**证据**: `services/dbms/sa_profile/distributedbms.cfg:1-31`

---

## 安全配置

### 统一安全选项（所有共享库）

**证据**: `services/dbms/BUILD.gn:28-37`, `interfaces/inner_api/BUILD.gn:23-31`

**配置**:
- **分支保护**: `branch_protector_ret = "pac_ret"`
- **控制流完整性**: `cfi = true`, `cfi_cross_dso = true`
- **边界检查**: `boundary_sanitize = true`
- **整型溢出**: `integer_overflow = true`
- **未定义行为**: `ubsan = true`
- **栈保护**: `cflags = ["-fstack-protector-strong"]`

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| 根 targets | BUILD.gn:16-36 |
| Feature flags | dbms.gni:28-59 |
| libdbms target | services/dbms/BUILD.gn:27-105 |
| dbms_fwk target | interfaces/inner_api/BUILD.gn:21-64 |
| SA 配置 | services/dbms/sa_profile/402.json:1-28 |
