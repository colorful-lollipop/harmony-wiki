# GN 构建配置

> 本文档描述 app_domain_verify 部件的 GN 构建配置，包括 targets、依赖关系和编译选项。

## 1. 根构建配置

**文件**: `BUILD.gn`

### 1.1 主入口

```gn
group("app_domain_verify_packages") {
  if (is_standard_system) {
    deps = [
      "etc:app_domain_verify_api_report_etc",
      "etc/init:app_domain_verify_agent.cfg",
      "frameworks/app_details_rdb:app_domain_verify_app_details_rdb",
      "frameworks/common:app_domain_verify_frameworks_common",
      "frameworks/extension:app_domain_verify_extension_framework",
      "frameworks/verifier:app_domain_verify_agent_verifier",
      "interfaces/inner_api/client:app_domain_verify_agent_client",
      "interfaces/inner_api/client:app_domain_verify_mgr_client",
      "interfaces/inner_api/common:app_domain_verify_common",
      "interfaces/kits/js/ani:appdomainverify_ani",
      "interfaces/kits/js/jsi:appdomainverify_napi",
      "profile:bundlemanager_app_domain_verify_sa_profiles",
      "services:app_domain_verify_agent_service",
      "services:app_domain_verify_mgr_service",
    ]
  }
}
```

## 2. 框架模块

### 2.1 frameworks/common

**文件**: `frameworks/common/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_frameworks_common.so` |
| **Sources** | 7 个 cpp 文件 |

**Sources**:
```
src/bms/bundle_info_query.cpp
src/config/white_list_config_mgr.cpp
src/httpsession/app_domain_verify_task_mgr.cpp
src/httpsession/i_http_task.cpp
src/permission/permission_manager.cpp
src/utils/critical_utils.cpp
src/utils/domain_url_util.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_common",
]
external_deps = [
  "ability_base",
  "access_token",
  "bundle_framework",
  "cJSON",
  "c_utils",
  "ffrt",
  "hicollie",
  "hilog",
  "hisysevent",
  "ipc",
  "memmgr",
  "netstack",
  "os_account",
  "preferences",
  "safwk",
  "samgr",
]
```

### 2.2 frameworks/verifier

**文件**: `frameworks/verifier/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_agent_verifier.so` |
| **Sources** | 4 个 cpp 文件 |

**Sources**:
```
src/domain_json_util.cpp
src/domain_verifier.cpp
src/verify_http_task.cpp
src/verify_task.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_mgr_client",
  ":app_domain_verify_common",
  ":app_domain_verify_frameworks_common",
]
```

### 2.3 frameworks/extension

**文件**: `frameworks/extension/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_extension_framework.so` |
| **Sources** | 3 个 cpp 文件 |

**Sources**:
```
src/app_domain_verify_agent_ext.cpp
src/app_domain_verify_extension_mgr.cpp
src/app_domain_verify_extension_register.cpp
```

### 2.4 frameworks/app_details_rdb

**文件**: `frameworks/app_details_rdb/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_app_details_rdb.so` |
| **Sources** | 4 个 cpp 文件 |

**Sources**:
```
src/app_details_meta_item.cpp
src/app_details_rdb_data_manager.cpp
src/app_details_rdb_item.cpp
src/app_details_rdb_open_callback.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_frameworks_common",
]
```

## 3. 服务模块

### 3.1 services

**文件**: `services/BUILD.gn`

**配置**:
```gn
config("app_domain_verify_service_config") {
  include_dirs = [
    ".",
    "$app_domain_verify_service_path/include",
    "$app_domain_verify_service_path/include/agent",
    "$app_domain_verify_service_path/include/manager",
    "$app_domain_verify_frameworks_common_path/include",
  ]
  cflags = [
    "-fvisibility=hidden",
    "-fstack-protector-strong",
  ]
  if (!is_debug) {
    cflags += [ "-Os" ]
  }
}
```

**Target 1**: `app_domain_verify_mgr_service`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_mgr_service.so` |
| **Sources** | 12 个 cpp 文件 |

**Sources**:
```
# 框架公共
bundle_info_query.cpp
white_list_config_mgr.cpp
permission_manager.cpp
domain_url_util.cpp

# 管理服务
app_details_data_mgr.cpp
app_domain_verify_data_mgr.cpp
app_domain_verify_mgr_service.cpp

# 延迟链接
ability_filter.cpp
deferred_link_mgr.cpp

# 过滤器
app_details_filter.cpp

# RDB
app_domain_verify_rdb_data_manager.cpp
app_domain_verify_rdb_open_callback.cpp
rdb_migrate_mgr.cpp

# ZIDL
app_domain_verify_mgr_service_proxy.cpp
app_domain_verify_mgr_service_stub.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_agent_client",
  ":app_domain_verify_common",
  ":app_domain_verify_app_details_rdb",
]
defines = [
  "API_EXPORT",
  if (!is_debug) "IS_RELEASE_VERSION",
]
```

**Target 2**: `app_domain_verify_agent_service`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_agent_service.so` |
| **Sources** | 3 个 cpp 文件 |

**Sources**:
```
src/agent/core/app_domain_verify_agent_service.cpp
src/agent/zidl/app_domain_verify_agent_service_proxy.cpp
src/agent/zidl/app_domain_verify_agent_service_stub.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_mgr_client",
  ":app_domain_verify_common",
  ":app_domain_verify_app_details_rdb",
  ":app_domain_verify_frameworks_common",
  ":app_domain_verify_extension_framework",
  ":app_domain_verify_agent_verifier",
]
```

## 4. 接口模块

### 4.1 interfaces/inner_api/common

**文件**: `interfaces/inner_api/common/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libapp_domain_verify_common.so` |
| **Sources** | 7 个 cpp 文件 |

### 4.2 interfaces/inner_api/client

**文件**: `interfaces/inner_api/client/BUILD.gn`

**Target 1**: `app_domain_verify_mgr_client`

| 属性 | 值 |
|-----|------|
| **输出文件** | `libapp_domain_verify_mgr_client.so` |

**Target 2**: `app_domain_verify_agent_client`

| 属性 | 值 |
|-----|------|
| **输出文件** | `libapp_domain_verify_agent_client.so` |

### 4.3 interfaces/kits/js/jsi

**文件**: `interfaces/kits/js/jsi/BUILD.gn`

| 属性 | 值 |
|-----|------|
| **Target 类型** | `ohos_shared_library` |
| **输出文件** | `libappdomainverify_napi.so` |
| **Sources** | 4 个 cpp 文件 |

**Sources**:
```
common/src/api_event_reporter.cpp
common/src/config_parser.cpp
src/app_domain_verify_manager_napi.cpp
src/native_module.cpp
```

**依赖**:
```gn
deps = [
  ":app_domain_verify_mgr_client",
  ":app_domain_verify_common",
]
```

**安装路径**: `module/bundle/`

## 5. 配置文件

### 5.1 etc/BUILD.gn

```gn
ohos_prebuilt_etc("app_domain_verify_api_report_etc") {
  source = "api_report.conf"
  install_dir = "app_domain_verify"
}
```

### 5.2 etc/init/BUILD.gn

```gn
ohos_prebuilt_etc("app_domain_verify_agent.cfg") {
  source = "init/app_domain_verify_agent.cfg"
}
```

### 5.3 profile/BUILD.gn

```gn
ohos_sa_profile("bundlemanager_app_domain_verify_sa_profiles") {
  sources = [ "6200.json", "6201.json" ]
}
```

## 6. 依赖图

```
app_domain_verify_packages
├── libapp_domain_verify_common.so ───────────┐
├── libapp_domain_verify_frameworks_common.so ─┤
│   └── libapp_domain_verify_common.so ◄─────┘
├── libapp_domain_verify_app_details_rdb.so ──┤
│   └── libapp_domain_verify_frameworks_common.so
├── libapp_domain_verify_extension_framework.so
├── libapp_domain_verify_agent_verifier.so ──┤
│   ├── libapp_domain_verify_mgr_client.so ───┤
│   ├── libapp_domain_verify_common.so ◄──────┤
│   └── libapp_domain_verify_frameworks_common.so
├── libapp_domain_verify_mgr_client.so ───────┤
│   └── libapp_domain_verify_common.so ◄──────┤
├── libapp_domain_verify_agent_client.so ─────┤
│   └── libapp_domain_verify_common.so ◄──────┤
├── libapp_domain_verify_mgr_service.so ──────┤
│   ├── libapp_domain_verify_agent_client.so
│   ├── libapp_domain_verify_common.so ◄──────┤
│   └── libapp_domain_verify_app_details_rdb.so
├── libapp_domain_verify_agent_service.so ────┤
│   ├── libapp_domain_verify_mgr_client.so ───┤
│   ├── libapp_domain_verify_common.so ◄──────┤
│   ├── libapp_domain_verify_app_details_rdb.so
│   ├── libapp_domain_verify_frameworks_common.so
│   ├── libapp_domain_verify_extension_framework.so
│   └── libapp_domain_verify_agent_verifier.so
├── libappdomainverify_napi.so ───────────────┤
│   ├── libapp_domain_verify_mgr_client.so ───┤
│   └── libapp_domain_verify_common.so ◄──────┤
├── appdomainverify_ani (ANI)
├── app_domain_verify_api_report_etc
├── app_domain_verify_agent.cfg
└── bundlemanager_app_domain_verify_sa_profiles
```

## 7. 构建命令

### 7.1 编译部件

```bash
./build.sh --product-name rk3568 --ccache --build-target app_domain_verify
```

### 7.2 编译测试

```bash
# 单元测试
./build.sh --product-name rk3568 --build-target app_domain_verify_unit_test

# 模糊测试
./build.sh --product-name rk3568 --build-target app_domain_verify_fuzz_test
```

## 8. 相关文档

| 文档 | 链接 |
|-----|------|
| 编译产物清单 | [05_Outputs.md](./05_Outputs.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 常见问题 | [07_FAQ.md](./07_FAQ.md) |
