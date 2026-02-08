# 构建配置

描述 GN 构建系统、feature 开关和编译产物。

---

## 1. GN 配置文件

### 1.1 关键配置文件

| 文件 | 用途 |
|-----|------|
| `datamgr_service.gni` | Feature 开关和路径定义 |
| `BUILD.gn` | 根构建入口，测试 targets |
| `bundle.json` | 模块清单，依赖声明 |
| `CLAUDE.md` | 构建命令说明 |

### 1.2 路径定义 (`datamgr_service.gni`)

```gni
# 外部依赖路径
kv_store_path = "//foundation/distributeddatamgr/kv_store"
relational_store_path = "//foundation/distributeddatamgr/relational_store"
dsoftbus_core_path = "//foundation/communication/dsoftbus/core/common/include"
datashare_path = "//foundation/distributeddatamgr/data_share"
device_manager_path = "//foundation/distributedhardware/device_manager"
udmf_path = "//foundation/distributeddatamgr/udmf"
dataobject_path = "//foundation/distributeddatamgr/data_object"

# 内部路径
data_service_path = "//foundation/.../datamgr_service/services/distributeddataservice"
```

---

## 2. Feature 开关

### 2.1 可配置开关

| 开关 | 默认值 | 依赖 | 说明 |
|-----|-------|------|------|
| `datamgr_service_cloud` | true | - | 云同步功能 |
| `datamgr_service_rdb` | true | relational_store | RDB 分布式支持 |
| `datamgr_service_kvdb` | true | kv_store | KV 分布式支持 |
| `datamgr_service_object` | true | data_object | 分布式对象 |
| `datamgr_service_data_share` | true | data_share | 数据共享 |
| `datamgr_service_udmf` | false | udmf | UDMF 服务 |
| `datamgr_service_config` | true | - | 配置支持 |
| `datamgr_service_distributed` | true | - | 分布式特性 |

### 2.2 条件开关

| 开关 | 条件 | 说明 |
|-----|------|------|
| `datamgr_service_power` | power_manager + battery_manager | 电源管理 |
| `dm_part_is_enabled` | device_manager | 设备管理 |
| `os_account_part_is_enabled` | os_account | 多用户 |
| `dataclassification_part_is_enabled` | dataclassification | 数据分类 |

### 2.3 bundle.json 配置

```json
{
  "component": {
    "name": "datamgr_service",
    "features": [
      "datamgr_service_config",
      "datamgr_service_cloud",
      "datamgr_service_rdb",
      "datamgr_service_kvdb",
      "datamgr_service_object",
      "datamgr_service_data_share",
      "datamgr_service_distributed"
    ],
    "deps": {
      "components": [
        "ability_base", "ability_runtime", "access_token",
        "bundle_framework", "common_event_service", "c_utils",
        "dataclassification", "data_share", "device_auth",
        "device_manager", "dfs_service", "dsoftbus",
        "hilog", "hisysevent", "hitrace", "huks",
        "kv_store", "ipc", "napi", "os_account",
        "relational_store", "safwk", "samgr", ...
      ]
    }
  }
}
```

---

## 3. GN Targets

### 3.1 顶层 Targets (`BUILD.gn`)

| Target | 类型 | 说明 |
|-------|------|------|
| `build_native_test` | group | 所有单元测试 |
| `fuzztest` | group | 所有模糊测试 |

### 3.2 Layer Targets

#### App Layer (`services/distributeddataservice/app/BUILD.gn`)

| Target | 类型 | 输出 | 依赖 |
|-------|------|-----|------|
| `distributeddataservice` | ohos_shared_library | libdistributeddataservice.so | framework, service |
| `distributeddata_profile` | ohos_sa_profile | 1301.json | - |

#### Framework Layer (`services/distributeddataservice/framework/BUILD.gn`)

| Target | 类型 | 输出 | 说明 |
|-------|------|-----|------|
| `distributeddatasvcfwk` | ohos_shared_library | libdistributeddatasvcfwk.so | 框架库 |

#### Service Layer (`services/distributeddataservice/service/BUILD.gn`)

| Target | 类型 | 输出 | 条件 |
|-------|------|-----|------|
| `distributeddatasvc` | ohos_shared_library | libdistributeddatasvc.so | - |

**条件依赖**:
```gn
deps = [
  ":distributeddata_cloud",   # if datamgr_service_cloud
  ":distributeddata_kvdb",     # if datamgr_service_kvdb
  ":distributeddata_rdb",     # if datamgr_service_rdb
  ":distributeddata_object",   # if datamgr_service_object
  ":data_share_service",       # if datamgr_service_data_share
  # ...
]
```

### 3.3 Feature Targets

| Feature | Target | 类型 |
|---------|--------|------|
| **Cloud** | `distributeddata_cloud` | ohos_source_set |
| **RDB** | `distributeddata_rdb` | ohos_source_set |
| **KVDB** | `distributeddata_kvdb` | ohos_source_set |
| **Object** | `distributeddata_object` | ohos_source_set |
| **DataShare** | `data_share_service` | ohos_source_set |
| **UDMF** | `udmf_server`, `utd_server` | ohos_source_set |

### 3.4 Adapter Targets

| Adapter | Target | 类型 |
|---------|--------|------|
| Account | `distributeddata_account` | ohos_source_set |
| Communicator | `distributeddata_communicator` | ohos_source_set |
| DFX | `distributeddata_dfx` | ohos_source_set |
| Network | `distributeddata_network` | ohos_source_set |
| QoS | `distributeddata_qos` | ohos_source_set |
| ScreenLock | `distributeddata_screenlock` | ohos_source_set |

---

## 4. 编译产物

### 4.1 共享库

| 产物 | 路径 | 说明 |
|-----|------|------|
| `libdistributeddataservice.so` | - | App 层主库 |
| `libdistributeddatasvcfwk.so` | - | Framework 层库 |
| `libdistributeddatasvc.so` | - | Service 层库 |

### 4.2 配置文件

| 产物 | 路径 | 说明 |
|-----|------|------|
| `1301.json` | `sa_profile/` | SA 配置文件 |
| `distributed_data.cfg` | `conf/` | Init 配置 |

### 4.3 SA Profile (`sa_profile/1301.json`)

```json
{
  "process": "distributeddata",
  "systemability": [{
    "name": 1301,
    "libpath": "libdistributeddataservice.z.so",
    "run-on-create": true,
    "distributed": false,
    "dump_level": 1,
    "extension": ["backup", "restore"]
  }]
}
```

---

## 5. 构建命令

### 5.1 标准构建

```bash
# 构建所有模块
./build.sh --product-name <product> --build-target datamgr_service

# 快速构建
./build.sh --product-name <product> --build-target datamgr_service --fast-rebuild
```

### 5.2 分层构建

```bash
# App 层
./build.sh --product-name <product> --build-target \
  //foundation/.../datamgr_service/services/distributeddataservice/app:build_module

# Framework 层
./build.sh --product-name <product> --build-target \
  //foundation/.../datamgr_service/services/distributeddataservice/framework:build_module

# Service 层
./build.sh --product-name <product> --build-target \
  //foundation/.../datamgr_service/services/distributeddataservice/service:build_module
```

### 5.3 测试构建

```bash
# 单元测试
./build.sh --product-name <product> --build-target datamgr_service_test
```

---

## 6. 产物运行时加载

```
系统启动
    │
    ▼
/system/bin/distributeddata (SA 进程)
    │
    ├─ dlopen ── libdistributeddataservice.z.so
    │                 │
    │                 ├─ libdistributeddatasvcfwk.so (Framework)
    │                 │
    │                 └─ libdistributeddatasvc.so (Service)
    │                               │
    │                               ├─ Feature Modules
    │                               │   ├─ KVDB
    │                               │   ├─ RDB
    │                               │   └─ Cloud
    │                               │
    │                               └─ Adapters
    │                                   ├─ Account
    │                                   ├─ Communicator
    │                                   └─ ...
    │
    └── 配置文件
        ├─ /system/etc/init/distributed_data.cfg
        └─ /system/etc/distributeddata/conf/config.json
```

---

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| 概览 | [00_Overview.md](00_Overview.md) |
| 架构 | [01_Architecture.md](01_Architecture.md) |
| 安全评审 | [03_Security.md](03_Security.md) |
