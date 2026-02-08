# 04_GN 构建配置

> GN 构建系统配置详解，包含 targets、依赖关系和产物映射。

## 1. 构建入口

### 1.1 根配置

**文件**: `updateengine.gni`

```gn
updateengine_client_library_name = "update"           # N-API 客户端库名
updateengine_inner_kits = "update_service_inner_kits"
updateengine_inner_library_name = "updateservicekits" # Inner API 库名
updateengine_library_name = "updateservice"           # SA 主库名
updateengine_part_name = "update_service"             # 部件名
updateengine_root_path = "//base/update/updateservice"
updateengine_idl_path = "$updateengine_root_path/interfaces/inner_api/engine"
```

**证据**: `updateengine.gni:14-20`

### 1.2 组件描述

**文件**: `bundle.json`

```json
{
  "component": {
    "name": "update_service",
    "subsystem": "updater",
    "features": [
      "update_service_dupdate_config_path",
      "update_service_enable_run_on_demand_qos",
      "update_service_updater_sa_cfg_path",
      "update_service_sa_profile_path"
    ],
    "build": {
      "modules": [
        "//base/update/updateservice/frameworks/js/ani:update_framework_taihe",
        "//base/update/updateservice/frameworks/js/napi/update:update",
        "//base/update/updateservice/interfaces/inner_api/engine:updateservicekits",
        "//base/update/updateservice/interfaces/inner_api/modulemgr:update_module_mgr",
        "//base/update/updateservice/services/engine:dupdate_config.json",
        "//base/update/updateservice/services/engine:updater_sa.cfg",
        "//base/update/updateservice/services/engine:updateservice",
        "//base/update/updateservice/services/engine/sa_profile:updater_sa_profile"
      ]
    }
  }
}
```

**证据**: `bundle.json:13-68`

---

## 2. 主要 Build Targets

### 2.1 服务层 Targets

#### updateservice (SA 主库)

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` (shlib_type = "sa") |
| **输出** | `libupdateservice.z.so` |
| **位置** | `/system/lib/` |
| **Sanitize** | boundary_sanitize, cfi, cfi_cross_dso |

**BUILD.gn**: `services/engine/BUILD.gn`

**sources**:
- `services/engine/src/` - 主服务实现
- `services/core/ability/` - 核心能力
- `services/firmware/` - OTA 升级
- `services/startup/` - 启动管理

**deps**:
```gn
deps = [
  "//base/update/updateservice/foundations:update_foundations",
  "//base/update/updateservice/interfaces/inner_api/engine:updateservicekits",
  "//base/update/updateservice/interfaces/inner_api/modulemgr:update_module_mgr",
]
```

**external_deps**:
```gn
external_deps = [
  "ability_base:configuration",
  "ability_base:session_info",
  "access_token:libaccesstoken_sdk",
  "access_token:libtokenid_sdk",
  "bundle_framework:appexecfwk_core",
  "cJSON:cjson",
  "c_utils:utils",
  "curl:curl_shared",
  "hilog:libhilog",
  "hisysevent:libhisysevent",
  "ipc:ipc_core",
  "openssl:libcrypto_shared",
  "openssl:libssl_shared",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
  "updater:libfsmanager",
  "updater:libpackage_shared",
  "updater:libupdater_shared",
  "storage_service:storage_manager_sa_proxy",
]
```

**defines**:
```gn
-DUAL_ADAPTER
-DUPDATE_SERVICE
# 更多条件定义见 engine_sa.gni
```

**证据**: `services/engine/BUILD.gn:51-72`

---

### 2.2 接口层 Targets

#### updateservicekits (Inner API 客户端)

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **输出** | `libupdateservicekits.z.so` |
| **位置** | `/system/lib/` |
| **Sanitize** | integer_overflow, ubsan, boundary_sanitize, cfi |
| **分支保护** | pac_ret |
| **InnerAPI** | platformsdk |

**BUILD.gn**: `interfaces/inner_api/engine/BUILD.gn`

**deps**:
```gn
deps = [
  ":update_service_interface",  # IDL 生成
  "//base/update/updateservice/foundations:update_foundations",
  "//base/update/updateservice/interfaces/inner_api/modulemgr:update_module_mgr",
]
```

**证据**: `interfaces/inner_api/engine/BUILD.gn`

---

#### update_module_mgr (模块管理器)

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **输出** | `libupdate_module_mgr.z.so` |
| **位置** | `/system/lib/` |
| **InnerAPI** | sasdk |

**BUILD.gn**: `interfaces/inner_api/modulemgr/BUILD.gn`

**sources**:
- `update_service_module.cpp`
- `module_manager.cpp`

**证据**: `interfaces/inner_api/modulemgr/BUILD.gn`

---

### 2.3 框架层 Targets

#### update (N-API 客户端)

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **输出** | `libupdate.z.so` |
| **位置** | `/system/lib/module/` |
| **NAPI Version** | 8 |
| **条件编译** | `ability_ability_runtime_enable` |

**BUILD.gn**: `frameworks/js/napi/update/BUILD.gn`

**sources**:
```gn
sources = [
  "common/src/client_helper.cpp",
  "common/src/iupdater.cpp",
  "src/define_property.cpp",
  "src/local_updater.cpp",
  "src/restorer.cpp",
  "src/session_manager.cpp",
  "src/update_client.cpp",
  "src/update_module.cpp",
  "src/update_session.cpp",
]
```

**deps**:
```gn
deps = [
  "//base/update/updateservice/foundations:update_foundations",
  "//base/update/updateservice/interfaces/inner_api/engine:updateservicekits",
]
```

**external_deps**:
```gn
external_deps = [
  "access_token:libaccesstoken_sdk",
  "access_token:libtokenid_sdk",
  "c_utils:utils",
  "cJSON:cjson",
  "hilog:libhilog",
  "ipc:ipc_core",
  "napi:ace_napi",
]
```

**cflags**:
```gn
"-DNAPI_VERSION=8"
"-fstack-protector-strong"
```

**证据**: `frameworks/js/napi/update/BUILD.gn:18-75`

---

#### update_ani (ANI 框架)

| 属性 | 值 |
|------|-----|
| **类型** | `taihe_shared_library` |
| **输出** | `libupdate_ani.z.so` |
| **ABC 输出** | `update_ani.abc` |
| **位置** | `/system/framework/` |

**BUILD.gn**: `frameworks/js/ani/BUILD.gn`

**证据**: `frameworks/js/ani/BUILD.gn`

---

### 2.4 基础层 Targets

#### update_foundations (基础库)

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **输出** | `libupdate_foundations.z.so` |
| **位置** | `/system/lib/` |
| **InnerAPI** | platformsdk |

**BUILD.gn**: `foundations/BUILD.gn`

**聚合模块**:
- `define` - 通用定义
- `log` - 日志基础设施
- `sa_loader` - SA 加载器
- `sys_event` - 系统事件
- `utils` - 工具函数
- `base_model` - 基础模型

**证据**: `foundations/BUILD.gn`, `foundations/foundations.gni`

---

## 3. 配置文件 Targets

### 3.1 配置文件清单

| Target | 类型 | 源文件 | 安装位置 |
|--------|------|--------|----------|
| `dupdate_config.json` | prebuilt_etc | `services/engine/etc/dupdate_config.json` | `update/` |
| `updater_sa.cfg` | prebuilt_etc | `services/engine/etc/updater_sa.cfg` | `init/` |
| `updater_sa.rc` | prebuilt_etc | `interfaces/inner_api/engine/etc/updater_sa.rc` | `init/` |
| `updater_sa_profile` | sa_profile | `services/engine/sa_profile/3006.json` | SA 目录 |

**证据**: `services/engine/BUILD.gn:33-49`, `services/engine/sa_profile/BUILD.gn`

---

## 4. Feature Flags

### 4.1 全局 Feature

**文件**: `adapter/default_config/feature_config/standard/config.gni`

```gn
ability_ability_runtime_enable = true/false       # 是否启用 ability runtime
communication_netmanager_base_enable = true/false # 是否启用网络管理器
```

### 4.2 服务 Feature

**文件**: `services/engine/engine_sa.gni`

```gn
ability_ability_base_enable = true/false
preference_native_preferences_enable = true/false
update_service_enable_run_on_demand_qos = true/false
```

### 4.3 数据库 Feature

**文件**: `services/core/ability/sqlite/sqlite.gni`

```gn
relational_store_native_rdb_enable = true/false  # RDB vs 空实现
```

### 4.4 条件 Defines

| Define | 条件 |
|--------|------|
| `DUAL_ADAPTER` | 始终启用 |
| `UPDATE_SERVICE` | 始终启用 |
| `ABILITY_BASE_ENABLE` | `ability_ability_base_enable` |
| `ABILITY_RUNTIME_ENABLE` | `ability_ability_runtime_enable` |
| `NETMANAGER_BASE_ENABLE` | `communication_netmanager_base_enable` |
| `UPDATE_SERVICE_ENABLE_RUN_ON_DEMAND_QOS` | 对应 feature |

**证据**: `updateengine.gni`, 各 `.gni` 文件

---

## 5. 依赖关系图

```mermaid
graph TD
    subgraph "Framework Layer"
        NAPI[update (N-API)]
        ANI[update_ani (ANI)]
    end
    
    subgraph "Interface Layer"
        KITS[updateservicekits]
        MGR[update_module_mgr]
    end
    
    subgraph "Service Layer"
        SA[updateservice (SA)]
    end
    
    subgraph "Foundation Layer"
        FOUND[update_foundations]
    end
    
    NAPI --> KITS
    NAPI --> FOUND
    ANI --> NAPI
    KITS --> MGR
    KITS --> FOUND
    SA --> KITS
    SA --> MGR
    SA --> FOUND
```

---

## 6. 产物映射表

| Target | 输出文件 | 安装路径 | 用途 |
|--------|----------|----------|------|
| `updateservice` | `libupdateservice.z.so` | `/system/lib/` | SA 主库 |
| `updateservicekits` | `libupdateservicekits.z.so` | `/system/lib/` | Inner API |
| `update_module_mgr` | `libupdate_module_mgr.z.so` | `/system/lib/` | 模块管理 |
| `update` | `libupdate.z.so` | `/system/lib/module/` | N-API 客户端 |
| `update_ani` | `libupdate_ani.z.so` | `/system/lib/` | ANI 库 |
| `update_ani` | `update_ani.abc` | `/system/framework/` | ABC 字节码 |
| `dupdate_config.json` | `dupdate_config.json` | `/system/etc/update/` | 运行时配置 |
| `updater_sa.cfg` | `updater_sa.cfg` | `/system/etc/init/` | SA 配置 |
| `updater_sa.rc` | `updater_sa.rc` | `/system/etc/init/` | RC 脚本 |
| `updater_sa_profile` | `3006.json` | `/system/sa/` | SA Profile |

---

## 7. IDL 生成

### 7.1 IDL 配置文件

**文件**: `services/engine/BUILD.gn`

```gn
idl_gen_interface("update_service_interface") {
  src_idl = rebase_path(updateengine_idl_path + "/" + "IUpdateService.idl")
  sources_callback = [ "callback/IUpdateCallback.idl" ]
  dst_file = string_join(",", idl_interface_sources)
  log_domainid = "0xD002E00"
  log_tag = "UPDATE_SERVICE_KITS"
}
```

**IDL 输入**:
- `IUpdateService.idl` - 主服务接口
- `IUpdateCallback.idl` - 回调接口

**IDL 输出**:
- `update_service_stub.cpp` - 服务端存根
- `update_service_proxy.cpp` - 客户端代理

**证据**: `services/engine/BUILD.gn:25-31`

---

## 8. 安全编译选项

### 8.1 生产库编译选项

所有生产库启用以下安全特性：

| 选项 | 说明 |
|------|------|
| `-fPIC` | 位置无关代码 |
| `-Os` | 尺寸优化 |
| `-fstack-protector-strong` | 栈保护 |
| `boundary_sanitize` | 边界检查 |
| `cfi` | 控制流完整性 |
| `cfi_cross_dso` | 跨 DSO CFI |
| `pac_ret` | PAC 返回地址保护 |
| `integer_overflow` | 整数溢出检测 |
| `ubsan` | 未定义行为检测 |

**证据**: `services/engine/BUILD.gn:52-58`, `frameworks/js/napi/update/BUILD.gn:68-74`

---

## 9. 测试 Targets

### 9.1 单元测试

| Target | 类型 | 描述 |
|--------|------|------|
| `UpdateLogTest` | ohos_unittest | 日志测试 |
| `UpdateServiceJsonUtilsTest` | ohos_unittest | JSON 工具测试 |
| `firmware_stream_installer_install_test` | ohos_unittest | 流式安装测试 |
| `stream_progress_thread_test` | ohos_unittest | 进度线程测试 |

### 9.2 模糊测试

| Target | 测试 API |
|--------|----------|
| `UpdateServiceCheckNewVersionFuzzTest` | checkNewVersion |
| `UpdateServiceDownloadFuzzTest` | download |
| `UpdateServiceCancelFuzzTest` | cancel |
| `UpdateServiceGetNewVersionFuzzTest` | getNewVersion |
| `UpdateServiceGetUpgradePolicyFuzzTest` | getUpgradePolicy |
| `UpdateServiceSetUpgradePolicyFuzzTest` | setUpgradePolicy |
| `UpdateServiceRegisterUpdateCallbackFuzzTest` | registerUpdateCallback |
| `UpdateServiceUnregisterUpdateCallbackFuzzTest` | unregisterUpdateCallback |

**证据**: `test/unittest/BUILD.gn`, `test/fuzztest/*/BUILD.gn`

---

## 10. 构建命令

### 10.1 全量构建

```bash
# 构建整个 update_service 部件
hb set
hb build -f
```

### 10.2 单独构建

```bash
# 构建单个 target
gn gen out/default --args="..."
ninja -C out/default //base/update/updateservice/services/engine:updateservice
```

### 10.3 查看产物

```bash
# 查看构建产物
ls -la out/default/libs/ | grep update
ls -la out/default/system/lib/ | grep update
```

---

## 11. 下一步

- **架构设计**: [01_Architecture.md](./01_Architecture.md)
- **N-API 参考**: [02_N-API.md](./02_N-API.md)
- **安全评审**: [05_Security.md](./05_Security.md)
- **故障排查**: [06_Troubleshooting.md](./06_Troubleshooting.md)
