# SecurityGuard 构建配置

**文档版本**：3.1.0  
**最后更新**：2026-02-06  
**维护者**：SecurityGuard Team

---

## 1. 概述

本文档描述 SecurityGuard 项目的 GN 构建配置，包括根构建文件、全局配置定义、各模块 targets 列表、编译产物以及依赖关系。所有配置均可追溯到代码证据。

**证据来源**：`BUILD.gn`、`security_guard.gni`、各模块 `BUILD.gn` 文件

---

## 2. 全局配置

### 2.1 配置文件位置

| 文件 | 路径 | 说明 |
|------|------|------|
| 根构建文件 | `/Volumes/lexar/code/d/work/oh/base/security/security_guard/BUILD.gn` | 项目入口构建配置 |
| 全局配置定义 | `/Volumes/lexar/code/d/work/oh/base/security/security_guard/security_guard.gni` | 全局编译参数定义 |

### 2.2 全局编译参数

**证据来源**：`security_guard.gni:14-31`

```gn
sg_root_dir = "//base/security/security_guard"
fuzz_test_output_path = "security_guard/security_guard"

declare_args() {
  security_guard_enable = true                    # 主开关
  security_guard_enable_ext = false              # 扩展功能
  security_guard_trim_model_analysis = false    # 精简模型分析
  security_guard_enable_device_id = false        # 设备 ID 功能
  security_guard_event_cfg_source = "security_guard_event.json"      # 事件配置源
  security_guard_model_cfg_source = "security_guard_model.cfg"      # 模型配置源
  security_guard_event_group_cfg_source = "security_guard_event_group.json"  # 事件组配置
  security_guard_config_update_trust_list_source = "config_update_trust_list.json"  # 配置更新信任列表
  security_guard_collector_cfg_source = "security_audit.cfg"         # 采集器配置
  security_guard_sa_profile_path = "security_guard.cfg"             # SA 配置文件路径
  security_guard_security_collector_sa_profile_path = "security_collector.cfg"  # 安全采集器 SA 配置
  security_guard_event_filter_path = "/system/lib/libsg_event_filter.so"       # 事件过滤器路径
  security_guard_event_wrapper_path = "/system/lib/libsg_event_wrapper.so"     # 事件包装器路径
}
```

### 2.3 Feature Flags 说明

| 开关 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `security_guard_enable` | true | bool | 主开关，关闭后不编译任何模块 |
| `security_guard_enable_ext` | false | bool | 扩展功能开关 |
| `security_guard_trim_model_analysis` | false | bool | 精简模型分析，关闭后不编译风险分类相关代码 |
| `security_guard_enable_device_id` | false | bool | 设备 ID 功能开关 |

---

## 3. 根构建 Targets

### 3.1 Group Targets 列表

**证据来源**：`BUILD.gn:18-103`

| Target | 类型 | 依赖模块 | 编译条件 |
|--------|------|----------|----------|
| `sg_classify_service_build_module` | group | services/risk_classify:sg_classify_service | is_standard_system |
| `sg_collect_service_build_module` | group | sg_config_manager, sg_collect_service, sg_collect_service_database | is_standard_system |
| `security_guard_fuzz_test` | group | 多个 fuzz test targets | is_standard_system |
| `security_guard_napi` | group | frameworks/js/napi:securityguard_napi | os_level == "standard" && support_jsapi |
| `security_collector_service_build_module` | group | services/security_collector:security_collector_service | is_standard_system |
| `security_collector_manager_build_module` | group | services/collector_manager:security_collector_manager | is_standard_system |
| `security_guard_unit_test` | group | 多个 unittest targets | is_standard_system |
| `security_guard_build_module_test` | group | 框架层单元测试 | is_standard_system |

### 3.2 核心 Group 定义

**证据来源**：`BUILD.gn:24-32`

```gn
group("sg_collect_service_build_module") {
  if (is_standard_system) {
    deps = [
      "${sg_root_dir}/services/config_manager:sg_config_manager",
      "${sg_root_dir}/services/data_collect:sg_collect_service",
      "${sg_root_dir}/services/data_collect:sg_collect_service_database",
    ]
  }
}

group("security_guard_napi") {
  if (os_level == "standard") {
    if (support_jsapi) {
      deps = [ "${sg_root_dir}/frameworks/js/napi:securityguard_napi" ]
    }
  }
}
```

---

## 4. Frameworks 模块 Targets

### 4.1 frameworks/common/collect

**证据来源**：`frameworks/common/collect/BUILD.gn`

| Target | 类型 | 输出产物 | Inner API 标签 |
|--------|------|----------|----------------|
| `libsg_collect_sdk` | ohos_shared_library | libsg_collect_sdk.so | platformsdk, sasdk |

#### 4.1.1 Target 详细配置

```gn
ohos_shared_library("libsg_collect_sdk") {
  # 源文件列表（19 个文件）
  sources = [
    "src/i_collector_subscriber.cpp",
    "src/security_collector_manager_callback_service.cpp",
    "src/security_collector_manager_callback_stub.cpp",
    "src/security_collector_subscribe_info.cpp",
    "src/security_event.cpp",
    "src/security_event_ruler.cpp",
    "src/security_guard_utils.cpp",
    "src/acquire_data_manager_callback_service.cpp",
    "src/acquire_data_manager_callback_stub.cpp",
    "src/data_collect_manager.cpp",
    "src/data_collect_manager_callback_service.cpp",
    "src/data_collect_manager_callback_stub.cpp",
    "src/event_info.cpp",
    "src/security_event_filter.cpp",
    "src/security_event_query_callback_service.cpp",
    "src/security_event_query_callback_stub.cpp",
    "src/sg_collect_client.cpp",
    "src/sg_obtaindata_client.cpp",
    "src/event_subscribe_client.cpp",
  ]

  # 头文件搜索路径
  include_dirs = [
    "include",
    "interfaces/inner_api/collect/include",
    "interfaces/inner_api/common/include",
    "interfaces/inner_api/collector/include",
    "frameworks/common/constants/include",
    "frameworks/common/collect/include",
    "frameworks/common/collector/include",
    "frameworks/common/log/include",
    "frameworks/common/utils/include",
  ]

  # 内部依赖
  deps = [
    "//base/security/security_guard/services/data_collect/idl:data_collect_manager_idl_sa_proxy",
  ]

  # 外部依赖
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]

  # 编译配置
  configs = []
  if (is_standard_system) {
    configs += [ "//base/security/security_guard/resource/config/build:coverage_flags" ]
  }

  # 安全编译选项
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
  }

  # 版本脚本
  version_script = "sg_collect_sdk.map"

  # 安装路径
  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 4.2 frameworks/common/classify

**证据来源**：`frameworks/common/classify/BUILD.gn`

| Target | 类型 | 输出产物 | Inner API 标签 |
|--------|------|----------|----------------|
| `sg_classify_stamp` | ohos_source_set | 静态归档 | 无 |
| `libsg_classify_sdk` | ohos_shared_library | libsg_classify_sdk.so | platformsdk, sasdk |

#### 4.2.1 sg_classify_stamp 配置

```gn
ohos_source_set("sg_classify_stamp") {
  sources = [
    "src/risk_analysis_manager_callback_service.cpp",
    "src/risk_analysis_manager_callback_stub.cpp",
  ]

  deps = [
    "//base/security/security_guard/services/risk_classify/idl:risk_analysis_manager_idl_sa_proxy",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]
}
```

#### 4.2.2 libsg_classify_sdk 配置

```gn
ohos_shared_library("libsg_classify_sdk") {
  sources = [
    "src/sg_classify_client.cpp",
  ]

  deps = [
    "//base/security/security_guard/services/risk_classify/idl:risk_analysis_manager_idl_sa_proxy",
  ]

  if (!security_guard_trim_model_analysis) {
    deps += [ ":sg_classify_stamp" ]
  }

  defines = []
  if (security_guard_trim_model_analysis) {
    defines += [ "SECURITY_GUARD_TRIM_MODEL_ANALYSIS" ]
  }

  version_script = "sg_classify_sdk.map"

  subsystem_name = "security"
  part_name = "security_guard"
  innerapi_tags = ["platformsdk", "sasdk"]
}
```

### 4.3 frameworks/common/collector

**证据来源**：`frameworks/common/collector/BUILD.gn`

| Target | 类型 | 输出产物 | Inner API 标签 |
|--------|------|----------|----------------|
| `libsg_collector_sdk` | ohos_shared_library | libsg_collector_sdk.so | platformsdk, sasdk |

#### 4.3.1 Target 详细配置

```gn
ohos_shared_library("libsg_collector_sdk") {
  sources = [
    "src/collector_manager.cpp",
    "src/collector_service_loader.cpp",
    "src/i_collector.cpp",
    "src/i_collector_fwk.cpp",
    "src/i_collector_subscriber.cpp",
    "src/security_collector_event_filter.cpp",
    "src/security_collector_manager_callback_service.cpp",
    "src/security_collector_manager_callback_stub.cpp",
    "src/security_collector_manager_proxy.cpp",
    "src/security_collector_subscribe_info.cpp",
    "src/security_event.cpp",
    "src/security_event_ruler.cpp",
  ]

  include_dirs = [
    "include",
    "interfaces/inner_api/common/include",
    "interfaces/inner_api/collector/include",
    "frameworks/common/constants/include",
    "frameworks/common/log/include",
    "frameworks/common/collector/include",
    "services/security_collector/include",
  ]

  version_script = "sg_collector_sdk.map"

  subsystem_name = "security"
  part_name = "security_guard"
  innerapi_tags = ["platformsdk", "sasdk"]
}
```

### 4.4 frameworks/js/napi

**证据来源**：`frameworks/js/napi/BUILD.gn`

| Target | 类型 | 输出产物 | 安装路径 |
|--------|------|----------|----------|
| `securityguard_napi` | ohos_shared_library | securityguard_napi.z.so | module/security |

#### 4.4.1 Target 详细配置

```gn
ohos_shared_library("securityguard_napi") {
  sources = [
    "napi_request_data_manager.cpp",
    "napi_security_event_querier.cpp",
    "security_guard_napi.cpp",
    "security_guard_sdk_adaptor.cpp",
  ]

  include_dirs = [
    "frameworks/common/constants/include",
    "frameworks/js/napi",
    "frameworks/common/log/include",
  ]

  deps = [
    ":libsg_classify_sdk",
    ":libsg_collect_sdk",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "napi:ace_napi",
    "samgr:samgr_proxy",
  ]

  defines = []
  if (security_guard_trim_model_analysis) {
    defines += [ "SECURITY_GUARD_TRIM_MODEL_ANALYSIS" ]
  }

  subsystem_name = "security"
  part_name = "security_guard"
}
```

---

## 5. Services 模块 Targets

### 5.1 services/data_collect

**证据来源**：`services/data_collect/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `sg_collect_service` | ohos_shared_library | sg_collect_service.so |
| `sg_collect_service_database` | ohos_shared_library | sg_collect_service_database.so |

#### 5.1.1 sg_collect_service 配置

```gn
ohos_shared_library("sg_collect_service") {
  sources = [
    # 框架层代码
    "frameworks/common/collect/src/security_event_filter.cpp",
    "frameworks/common/collector/src/security_collector_event_filter.cpp",
    "frameworks/common/collector/src/security_collector_subscribe_info.cpp",
    "frameworks/common/json/src/json_cfg.cpp",
    "frameworks/common/utils/src/security_guard_utils.cpp",
    # 服务层代码
    "services/data_collect/sa/acquire_data_callback_proxy.cpp",
    "services/data_collect/sa/acquire_data_subscribe_manager.cpp",
    "services/data_collect/sa/data_collect_manager_callback_proxy.cpp",
    "services/data_collect/sa/data_collect_manager_service.cpp",
    "services/data_collect/sa/data_format.cpp",
    "services/data_collect/sa/security_event_query_callback_proxy.cpp",
    # 存储层代码
    "services/data_collect/store/src/database_helper.cpp",
    "services/data_collect/store/src/database_manager.cpp",
    "services/data_collect/store/src/file_system_store_helper.cpp",
    "services/data_collect/store/src/data_statistics.cpp",
    "services/data_collect/store/src/risk_event_rdb_helper.cpp",
    # 采集器代码
    "services/security_collector/src/security_collector_run_manager.cpp",
  ]

  defines = [
    "SECURITY_GUARD_EVENT_CFG_SOURCE",
    "SECURITY_GUARD_MODEL_CFG_SOURCE",
    "SECURITY_GUARD_EVENT_GROUP_CFG_SOURCE",
    "SECURITY_GUARD_CONFIG_UPDATE_TRUST_LIST_SOURCE",
    "SECURITY_GUARD_COLLECTOR_CFG_SOURCE",
    "SECURITY_GUARD_EVENT_FILTER_PATH",
    "SECURITY_GUARD_EVENT_WRAPPER_PATH",
  ]

  if (security_guard_trim_model_analysis) {
    defines += [ "SECURITY_GUARD_TRIM_MODEL_ANALYSIS" ]
  }

  if (security_guard_enable_device_id) {
    defines += [ "SECURITY_GUARD_ENABLE_DEVICE_ID" ]
  }

  deps = [
    ":libsg_collector_sdk",
    ":sg_bigdata_stamp",
    ":security_collector_manager",
    ":sg_config_data_manager",
    ":sg_collect_service_database",
    ":data_collect_manager_idl_sa_stub",
    ":sg_model_manager_stamp",
    ":libsg_collect_sdk",
  ]

  version_script = "sg_collect_service.map"

  subsystem_name = "security"
  part_name = "security_guard"
}
```

#### 5.1.2 sg_collect_service_database 配置

```gn
ohos_shared_library("sg_collect_service_database") {
  sources = [
    "services/data_collect/store/src/database.cpp",
    "services/data_collect/store/src/generic_values.cpp",
    "services/data_collect/store/src/sg_sqlite_helper.cpp",
    "services/data_collect/store/src/sqlite_helper.cpp",
    "services/data_collect/store/src/statement.cpp",
    "services/data_collect/store/src/variant_value.cpp",
  ]

  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 5.2 services/risk_classify

**证据来源**：`services/risk_classify/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `sg_classify_service` | ohos_shared_library | sg_classify_service.so |

```gn
ohos_shared_library("sg_classify_service") {
  sources = [
    "frameworks/common/utils/src/file_util.cpp",
    "frameworks/common/utils/src/json_util.cpp",
    "frameworks/common/utils/src/security_guard_utils.cpp",
    "services/risk_classify/plugin_manager/src/detect_plugin_manager.cpp",
    "services/risk_classify/src/risk_analysis_manager_callback_proxy.cpp",
    "services/risk_classify/src/risk_analysis_manager_service.cpp",
  ]

  deps = [
    ":libsg_collect_sdk",
    ":sg_bigdata_stamp",
    ":sg_config_manager",
    ":sg_collect_service",
    ":risk_analysis_manager_idl_sa_stub",
    ":sg_model_manager_stamp",
  ]

  version_script = "sg_classify_service.map"

  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 5.3 services/security_collector

**证据来源**：`services/security_collector/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `security_collector_service` | ohos_shared_library | security_collector_service.so |

```gn
ohos_shared_library("security_collector_service") {
  sources = [
    "frameworks/common/collector/src/security_collector_event_filter.cpp",
    "frameworks/common/collector/src/security_collector_subscribe_info.cpp",
    "services/security_collector/src/security_collector_manager_callback_proxy.cpp",
    "services/security_collector/src/security_collector_manager_service.cpp",
    "services/security_collector/src/security_collector_manager_stub.cpp",
    "services/security_collector/src/security_collector_run_manager.cpp",
    "services/security_collector/src/security_collector_subscriber_manager.cpp",
  ]

  deps = [
    ":libsg_collect_sdk",
    ":libsg_collector_sdk",
    ":security_collector_manager",
  ]

  version_script = "security_collector_service.map"

  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 5.4 services/collector_manager

**证据来源**：`services/collector_manager/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `security_collector_manager` | ohos_shared_library | security_collector_manager.so |

```gn
ohos_shared_library("security_collector_manager") {
  sources = [
    "frameworks/common/json/src/json_cfg.cpp",
    "frameworks/common/utils/src/security_guard_utils.cpp",
    "services/collector_manager/src/collector_cfg_marshalling.cpp",
    "services/collector_manager/src/data_collection.cpp",
    "services/collector_manager/src/lib_loader.cpp",
  ]

  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 5.5 services/config_manager

**证据来源**：`services/config_manager/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `sg_config_data_manager` | ohos_shared_library | sg_config_data_manager.so |
| `sg_config_manager` | ohos_shared_library | sg_config_manager.so |

```gn
ohos_shared_library("sg_config_data_manager") {
  sources = [
    "services/config_manager/src/config_data_manager.cpp",
  ]
}

ohos_shared_library("sg_config_manager") {
  sources = [
    "frameworks/common/json/src/json_cfg.cpp",
    "frameworks/common/utils/src/file_util.cpp",
    "frameworks/common/utils/src/json_util.cpp",
    "frameworks/common/utils/src/security_guard_utils.cpp",
    "services/config_manager/src/base_config.cpp",
    "services/config_manager/src/config_manager.cpp",
    "services/config_manager/src/config_operator.cpp",
    "services/config_manager/src/config_subscriber.cpp",
    "services/config_manager/src/event_config.cpp",
    "services/config_manager/src/event_group_config.cpp",
    "services/config_manager/src/interface.cpp",
    "services/config_manager/src/model_cfg_marshalling.cpp",
    "services/config_manager/src/model_config.cpp",
  ]

  version_script = "sg_config_manager.map"

  subsystem_name = "security"
  part_name = "security_guard"
}
```

### 5.6 services/bigdata

**证据来源**：`services/bigdata/BUILD.gn`

| Target | 类型 | 输出产物 |
|--------|------|----------|
| `sg_bigdata_stamp` | ohos_source_set | 静态归档 |

```gn
ohos_source_set("sg_bigdata_stamp") {
  sources = [
    "src/bigdata.cpp",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
  ]
}
```

---

## 6. IDL 接口 Targets

### 6.1 Risk Analysis Manager IDL

**证据来源**：`services/risk_classify/idl/BUILD.gn`

| Target | 类型 | 说明 |
|--------|------|------|
| `risk_analysis_manager_service_interface` | idl_gen_interface | IDL 接口生成 |
| `risk_analysis_manager_idl_sa_stub` | ohos_source_set | Server Stub |
| `risk_analysis_manager_idl_sa_proxy` | ohos_source_set | Client Proxy |
| `risk_analysis_manager_idl_sa_stub_tdd` | ohos_source_set | TDD Stub |

**IDL 文件**：`RiskAnalysisManager.idl`

### 6.2 Data Collect Manager IDL

**证据来源**：`services/data_collect/idl/BUILD.gn`

| Target | 类型 | 说明 |
|--------|------|------|
| `data_collect_manager_service_interface` | idl_gen_interface | IDL 接口生成 |
| `data_collect_manager_idl_sa_stub` | ohos_source_set | Server Stub |
| `data_collect_manager_idl_sa_proxy` | ohos_source_set | Client Proxy |

**IDL 文件**：`DataCollectManagerIdl.idl`

---

## 7. SA Profile 与配置 Targets

### 7.1 SA Profile 配置

**证据来源**：`sa_profile/BUILD.gn`

| Target | 类型 | 源文件 | 安装路径 |
|--------|------|--------|----------|
| `sg_sa_profile_standard` | ohos_sa_profile | 3523.json, 3524.json | 无 |
| `security_guard.init` | ohos_prebuilt_etc | security_guard.cfg | init |
| `security_collector_sa_profile_standard` | ohos_sa_profile | 3525.json | 无 |
| `security_collector.init` | ohos_prebuilt_etc | security_collector.cfg | init |

### 7.2 OEM 配置

**证据来源**：`oem_property/BUILD.gn`

| Target | 类型 | 源文件 |
|--------|------|--------|
| `security_guard_cfg` | ohos_prebuilt_etc | hos/security_guard.cfg |
| `security_guard_model_cfg` | ohos_prebuilt_etc | hos/security_guard_model.cfg |
| `security_guard_event_cfg` | ohos_prebuilt_etc | hos/security_guard_event.json |
| `security_audit_cfg` | ohos_prebuilt_etc | hos/security_audit.cfg |
| `config_update_trust_list_cfg` | ohos_prebuilt_etc | hos/config_update_trust_list.json |
| `security_guard_event_group_cfg` | ohos_prebuilt_etc | hos/security_guard_event_group.json |

---

## 8. 编译产物清单

### 8.1 主要产物列表

| 产物名称 | 类型 | 源模块 | 用途 |
|---------|------|--------|------|
| securityguard_napi.z.so | 共享库 | frameworks/js/napi | N-API 接口模块 |
| libsg_collect_sdk.so | 共享库 | frameworks/common/collect | 数据采集 SDK（Inner API） |
| libsg_classify_sdk.so | 共享库 | frameworks/common/classify | 风险分类 SDK（Inner API） |
| libsg_collector_sdk.so | 共享库 | frameworks/common/collector | 采集器 SDK（Inner API） |
| sg_collect_service.so | 服务 | services/data_collect | 数据收集服务（SA 3524） |
| sg_collect_service_database.so | 服务 | services/data_collect | 数据库服务模块 |
| sg_classify_service.so | 服务 | services/risk_classify | 风险分类服务（SA 3523） |
| security_collector_service.so | 服务 | services/security_collector | 采集器服务（SA 3525） |
| security_collector_manager.so | 共享库 | services/collector_manager | 采集器管理模块 |
| sg_config_manager.so | 共享库 | services/config_manager | 配置管理模块 |
| sg_config_data_manager.so | 共享库 | services/config_manager | 配置数据管理模块 |

### 8.2 产物安装路径

| 产物 | 预期安装路径 |
|------|-------------|
| securityguard_napi.z.so | /system/lib/module/security/ |
| libsg_collect_sdk.so | /system/lib/ |
| libsg_classify_sdk.so | /system/lib/ |
| libsg_collector_sdk.so | /system/lib/ |
| sg_collect_service.so | /system/lib/ |
| sg_classify_service.so | /system/lib/ |
| security_collector_service.so | /system/lib/ |

### 8.3 数据存储路径

**证据来源**：`sa_profile/security_guard.cfg:6-10`

```json
{
  "jobs": [{
    "name": "services:security_guard",
    "cmds": [
      "mkdir /data/service/el1/public/database 0711 ddms ddms",
      "mkdir /data/service/el1/public/database/security_guard_service 02770 security_guard ddms",
      "mkdir /data/service/el1/public/database/security_guard_service/file_store 0700 security_guard security_guard",
      "mkdir /data/service/el1/public/security_guard 0700 security_guard security_guard",
      "mkdir /data/service/el1/public/security_guard/tmp 0700 security_guard security_guard"
    ]
  }]
}
```

---

## 9. 依赖关系总览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              构建依赖图                                      │
│                                                                              │
│  应用层                                                                      │
│  ┌─────────────────────────────┐                                            │
│  │ securityguard_napi.z.so    │                                            │
│  └──────────────┬──────────────┘                                            │
│                 │ deps                                                         │
│  ┌──────────────▼──────────────┐                                            │
│  │ libsg_classify_sdk.so      │                                            │
│  │ libsg_collect_sdk.so       │                                            │
│  │ libsg_collector_sdk.so     │                                            │
│  └──────────────┬──────────────┘                                            │
│                 │ deps                                                         │
│  ┌──────────────▼──────────────────────────────────────────────────────┐   │
│  │                     IDL IPC 接口层                                  │   │
│  │  risk_analysis_manager_idl_sa_proxy    data_collect_manager_idl_sa_proxy│
│  └───────────────────────────────┬──────────────────────────────────────┘   │
└──────────────────────────────────┼───────────────────────────────────────────┘
                                   │ IPC
┌──────────────────────────────────▼───────────────────────────────────────────┐
│                              服务层                                          │
│  ┌────────────────────────────┐  ┌────────────────────────────────────────┐│
│  │ sg_classify_service.so    │  │ sg_collect_service.so                 ││
│  │ 依赖:                       │  │ 依赖:                                  ││
│  │ • libsg_collect_sdk.so    │  │ • libsg_collector_sdk.so              ││
│  │ • sg_config_manager.so    │  │ • security_collector_manager.so       ││
│  │ • sg_collect_service.so   │  │ • sg_config_manager.so               ││
│  │ • risk_analysis_manager_ │  │ • sg_collect_service_database.so     ││
│  │   idl_sa_stub             │  │ • data_collect_manager_idl_sa_stub   ││
│  │ • sg_model_manager_stamp │  │ • sg_model_manager_stamp             ││
│  │ • sg_bigdata_stamp       │  │ • sg_bigdata_stamp                   ││
│  └────────────────────────────┘  └────────────────────────────────────────┘││
│                                                                              │
│  ┌────────────────────────────┐  ┌────────────────────────────────────────┐│
│  │ security_collector_       │  │ security_collector_manager.so         ││
│  │ service.so                │  │                                        ││
│  │ 依赖:                       │  │                                        ││
│  │ • libsg_collect_sdk.so    │  │                                        ││
│  │ • libsg_collector_sdk.so │  │                                        ││
│  │ • security_collector_     │  │                                        ││
│  │   manager.so              │  │                                        ││
│  └────────────────────────────┘  └────────────────────────────────────────┘││
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. 测试 Targets

### 10.1 单元测试

**证据来源**：`BUILD.gn:77-91`

| 模块 | Target | 类型 |
|------|--------|------|
| inner_api | inner_api_test | ohos_unittest |
| config_manager | SecurityGuardConfigManagerTest | ohos_unittest |
| data_collect | data_collect_test | ohos_unittest |
| data_collect | SecurityGuardDataCollectTest | ohos_unittest |
| data_collect | SecurityGuardDatabaseManagerTest | ohos_unittest |
| data_collect | SecurityGuardDataCollectSaTest | ohos_unittest |
| risk_classify | SecurityGuardRiskAnalysisTest | ohos_unittest |
| model_manager | SecurityGuardModelManagerTest | ohos_unittest |
| security_collector | security_collector_test | ohos_unittest |

### 10.2 Fuzz 测试

**证据来源**：`BUILD.gn:34-51`

| 模块 | Fuzz Test Targets |
|------|-------------------|
| collect | ReportSecurityInfoFuzzTest |
| classify | RequestSecurityModelResultAsyncFuzzTest, RequestSecurityModelResultSyncFuzzTest |
| data_collect | AcquireDataSubscribeManagerFuzzTest, DatabaseFuzzTest, DataCollectManagerServiceFuzzTest |
| ipc | DataCollectManagerFuzzTest, RiskAnalysisManagerFuzzTest, SecurityCollectorManagerFuzzTest |
| security_collector | SecurityCollectorFuzzTest |

---

## 11. 安全编译选项

### 11.1 启用的安全编译选项

所有 targets 默认启用了以下安全编译选项：

| 选项 | 说明 |
|------|------|
| `sanitize.integer_overflow` | 整数溢出检测 |
| `sanitize.ubsan` | 未定义行为检测 |
| `sanitize.boundary_sanitize` | 边界检测 |
| `sanitize.cfi` | 控制流完整性 |
| `sanitize.cfi_cross_dso` | 跨 DSO CFI |
| `branch_protector_ret = "pac_ret"` | 返回地址保护（PAC） |
| `-D_FORTIFY_SOURCE=2` | 强化源码宏 |
| `-Os` | 优化大小 |

**证据来源**：各模块 `BUILD.gn` 文件中的 `sanitize` 配置

---

## 12. 相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API 参考 |
| [03_Architecture.md](./03_Architecture.md) | 架构详解 |
| [05_Services.md](./05_Services.md) | SA 服务详解 |
| [06_Security_Review.md](./06_Security_Review.md) | 安全风险分析 |
