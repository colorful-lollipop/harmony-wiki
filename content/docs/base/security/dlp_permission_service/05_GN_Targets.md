# GN Targets 与编译配置

## 目的

本文档描述 DLP 权限管理服务的 GN 构建系统，包括关键 targets、类型、依赖、编译产物和特性开关。

## 适用范围

- 目标读者：构建工程师、平台开发者
- 覆盖内容：GN targets、类型、依赖、产物、开关

---

## GN 配置概览

### GN 文件统计

| 文件类型 | 数量 |
|----------|------|
| BUILD.gn | 86 |
| .gni | 2 |

### 根配置文件

| 文件 | 路径 | 用途 |
|------|------|------|
| `BUILD.gn` | 根目录 | 根构建入口 |
| `dlp_permission_service.gni` | 根目录 | 定义根目录和 feature flags |
| `identify_sensitive_content.gni` | 根目录 | 敏感内容识别 feature flag |
| `config/BUILD.gn` | config/ | 公共构建配置（覆盖率、fortify） |

---

## 根 Target

### dlp_permission_build_module

**文件**：`BUILD.gn:17-33

**类型**：group

**依赖**：

| Target | 路径 | 说明 |
|--------|-------|------|
| `clone_app_permission_config` | frameworks/access_config | 克隆应用权限配置 |
| `libdlp_fuse` | interfaces/inner_api/dlp_fuse | FUSE 文件系统库 |
| `libdlp_permission_common_interface` | interfaces/inner_api/dlp_permission | 通用接口库 |
| `libdlp_permission_sdk` | interfaces/inner_api/dlp_permission | 主 SDK |
| `libdlp_setconfig_sdk` | interfaces/inner_api/dlp_set_config | 配置 SDK |
| `ohdlp_permission` | interfaces/kits/c | C API / NDK 库 |
| `dlp_permission_service` | services/dlp_permission/sa | System Ability 服务 |
| `dlp_permission_sa_profile_standard` | services/dlp_permission/sa/sa_profile | SA 配置 |
| `napi_packages` | interfaces/kits | N-API 包（条件编译） |

**条件编译**：
```gn
if (is_standard_system) {
    deps = [...]
}
if (support_jsapi) {
    deps += [napi_packages]
}
```

---

## Frameworks 层 Targets

### frameworks/access_config:clone_app_permission_config

**文件**：`frameworks/access_config/BUILD.gn`

**类型**：ohos_prebuilt_etc

**输出**：`clone_app_permission.json`

**安装位置**：`dlp_permission/`

**依赖**：无

### frameworks/dlp_permission

**说明**：框架层代码通常作为 source_set 集成到服务中，无独立 target。

---

## Inner API 层 Targets

### interfaces/inner_api/dlp_permission:libdlp_permission_sdk

**文件**：`interfaces/inner_api/dlp_permission/BUILD.gn`

**类型**：ohos_shared_library

**输出**：`libdlp_permission_sdk.so`

**依赖**：

| Target | 类型 | 说明 |
|--------|------|------|
| `dlp_permission_stub` | ohos_source_set | IPC stub/proxy |
| `dlp_permission_interface` | idl_gen_interface | IDL 接口生成 |
| `dlp_permission_sdk_config` | config | SDK 配置 |

**外部依赖**：
- ipc:ipc_core
- samgr:samgr_proxy
- os_account:libaccountkits
- access_token:libaccesstoken_sdk
- hilog:libhilog
- 等

### interfaces/inner_api/dlp_permission:libdlp_permission_common_interface

**类型**：ohos_shared_library

**输出**：`libdlp_permission_common_interface.so`

**依赖**：`dlp_permission_interface`

**用途**：公共接口库（platformsdk）

### interfaces/inner_api/dlp_fuse:libdlp_fuse

**文件**：`interfaces/inner_api/dlp_fuse/BUILD.gn`

**类型**：ohos_shared_library

**输出**：`libdlp_fuse.so`

**依赖**：

| Target | 类型 | 说明 |
|--------|------|------|
| `libdlpparse_inner` | ohos_shared_library | 内部解析库 |
| `libfuse` | ohos_prebuilt_etc | FUSE 库 |

**Sources**：
- `dlp_fuse_fd.c`
- `dlp_fuse_helper.cpp`
- `dlp_fuse_utils.cpp`
- `dlp_link_file.cpp`
- `dlp_link_manager.cpp`
- `fuse_daemon.cpp`

**Include Dirs**：
```
${dlp_root_dir}/frameworks/common/include
${dlp_root_dir}/interfaces/inner_api/dlp_parse/include
${dlp_root_dir}/interfaces/inner_api/dlp_permission/include/
include
```

### interfaces/inner_api/dlp_parse:libdlpparse

**类型**：ohos_shared_library

**输出**：`libdlpparse.so`

**依赖**：
- `dlpparse_public_config`
- `os_account:libaccountkits` (条件)
- `security_dlp_credential_service:libcredential_service_sdk` (条件)

### interfaces/inner_api/dlp_parse:libdlpparse_inner

**类型**：ohos_shared_library

**输出**：`libdlpparse_inner.so`

**依赖**：
- `libdlpparse`
- `os_account:libaccountkits` (条件)
- `security_dlp_credential_service:libcredential_service_sdk` (条件)

**Defines**：
- `DLP_PARSE_INNER` (当 `dlp_parse_inner` = true)
- `SUPPORT_DLP_CREDENTIAL` (当 `dlp_credential_enable` = true)

### interfaces/inner_api/dlp_set_config:libdlp_setconfig_sdk

**类型**：ohos_shared_library

**输出**：`libdlp_setconfig_sdk.so`

**依赖**：
- `libdlp_permission_sdk`
- `dlpsetconfig_public_config`

---

## Kits 层 Targets

### interfaces/kits/c:ohdlp_permission

**文件**：`interfaces/kits/c/BUILD.gn`

**类型**：ohos_shared_library

**输出**：`ohdlp_permission.so`

**Inner API Tags**：[ndk]

**依赖**：`libdlp_permission_sdk`

### interfaces/kits/dlp_permission/napi:libdlppermission_napi

**文件**：`interfaces/kits/dlp_permission/BUILD.gn`

**类型**：ohos_shared_library

**输出**：`libdlppermission_napi.so`

**安装位置**：`module/`

**Sanitize**：
- integer_overflow = true
- cfi = true
- cfi_cross_dso = true

**Defines**：
- `HILOG_ENABLE`
- `IS_EMULATOR` (当 `is_emulator` = true)
- `SUPPORT_DLP_CREDENTIAL` (当 `dlp_credential_enable` = true)

**Sources**：
```
${dlp_root_dir}/interfaces/kits/dlp_permission/napi/src/napi_dlp_permission.cpp
${dlp_root_dir}/interfaces/kits/dlp_permission/napi/src/napi_dlp_permission_manager.cpp
${dlp_root_dir}/interfaces/kits/dlp_permission/napi/src/napi_dlp_connection_plugin.cpp
${dlp_root_dir}/interfaces/kits/napi_common/src/napi_common.cpp
${dlp_root_dir}/interfaces/kits/napi_common/src/napi_error_msg.cpp
${dlp_root_dir}/interfaces/kits/dlp_permission/napi/src/dlp_connection_static_mock.cpp (当 SUPPORT_DLP_CREDENTIAL = false)
```

**依赖**：
- `libdlp_fuse`
- `libdlpparse_inner`
- `libdlp_permission_sdk`

**外部依赖**：
- ability_runtime:napi_base_context, napi_common
- access_token:libaccesstoken_sdk
- napi:ace_napi
- 等

### interfaces/kits/dlp_permission/napi:libdlpsetdlpfeature_napi

**类型**：ohos_shared_library

**输出**：`libdlpsetdlpfeature_napi.so`

**安装位置**：`module/`

**Sanitize**：与 libdlppermission_napi 相同

**Sources**：
```
napi_dlp_feature.cpp
napi_common.cpp
napi_error_msg.cpp
```

### interfaces/kits:identify_sensitive_content:napi/identifysensitivecontent_napi

**类型**：ohos_shared_library

**输出**：`identifysensitivecontent_napi.so`

**安装位置**：`module/security/`

**条件编译**：`FILE_IDENTIFY_ENABLE` (当 `target_platform == "pc" && data_identify_anonymize_service_enable`)

---

## Services 层 Targets

### services/dlp_permission/sa:dlp_permission_service

**文件**：`services/dlp_permission/sa/BUILD.gn`

**类型**：ohos_shared_library

**输出**：`libdlp_permission_service.z.so`

**依赖**：

| Target | 类型 | 说明 |
|--------|------|------|
| `dlp_hex_string_static` | ohos_static_library | 十六进制字符串工具 |
| `dlp_permission_serializer_static` | ohos_static_library | 序列化器 |
| `dlp_permission_stub` | ohos_source_set | IPC stub |
| `param_files` | group | 参数配置文件 |

**External Dependencies** (部分）：
- ability_base:want, ability_base:zuri
- ability_runtime:app_manager, extension_manager
- access_token:libaccesstoken_sdk
- bundle_framework:appexecfwk_base, appexecfwk_core
- ipc:ipc_core
- huks:libhukssdk
- safwk:system_ability_fwk
- samgr:samgr_proxy
- os_account:domain_account_innerkits, libaccountkits, os_account_innerkits
- hilog:libhilog
- hisysevent:libhisysevent
- config_policy:configpolicy_util
- eventhandler:libeventhandler
- kv_store:distributeddata_inner
- json:nlohmann_json_static
- openssl:openssl_shared
- libfuse:libfuse

**Include Dirs**：
```
adapt_utils/account_adapt
adapt_utils/alg_adapt/alg_manager/include
adapt_utils/alg_adapt/huks_adapt_manager/include
adapt_utils/app_observer
adapt_utils/critical_handler
adapt_utils/file_manager
callback/dlp_sandbox_change_callback
callback/open_dlp_file_callback
sa_common
sa_main
storage/include
${dlp_root_dir}/frameworks/common/include
${dlp_root_dir}/frameworks/dlp_permission/include
${dlp_root_dir}/interfaces/inner_api/dlp_parse/include
${dlp_root_dir}/interfaces/inner_api/dlp_permission/include
```

### services/dlp_permission/sa/sa_profile:dlp_permission_sa_profile_standard

**类型**：ohos_sa_profile

**输出**：SA 配置

**Sources**：`3521.json`

---

## 特性开关 (Feature Flags)

### dlp_permission_service.gni

**文件路径**：根目录

**全局变量**：

| Flag | 默认值 | 条件 | 说明 |
|------|---------|------|------|
| `dlp_permission_service_gathering_policy` | false | - | 聚合沙箱策略 |
| `dlp_parse_inner` | true | 当 `global_parts_info.account_os_account` 存在 | 内部解析特性 |
| `dlp_credential_enable` | true | 当 `global_parts_info.security_dlp_credential_service` 存在 | 凭证服务支持 |
| `dlp_permission_service_credential_connection_enable` | true | - | 凭证连接支持 |
| `dlp_file_version_inner` | true | - | 内部文件版本 |

### define_flags

**根据 feature flags 生成的编译宏**：

| Define | 条件 | 用途 |
|--------|------|------|
| `DLP_GATHERING_SANDBOX` | `dlp_permission_service_gathering_policy` = true | 聚合策略功能 |
| `DLP_PARSE_INNER` | `dlp_parse_inner` = true | 内部解析功能 |
| `SUPPORT_DLP_CREDENTIAL` | `dlp_credential_enable` = true | 凭证支持 |
| `DLP_FILE_VERSION_INNER` | `dlp_file_version_inner` = true | 内部文件版本 |

### config/BUILD.gn

**覆盖率 flag**：

| Config | 条件 | 效果 |
|--------|------|------|
| `coverage_flags` | `dlp_permission_service_feature_coverage` = true | 添加 `--coverage` cflags/ldflags |

**安全加固 flag**：

| Config | 效果 |
|--------|------|
| `common_build_options_flags` | 设置 `-D_FORTIFY_SOURCE=2` |

---

## Target 类型说明

| 类型 | 说明 | 后缀 |
|------|------|------|
| `group` | 目标组，仅聚合依赖 | 无 |
| `ohos_shared_library` | OpenHarmony 共享库 | .so |
| `ohos_static_library` | OpenHarmony 静态库 | .a |
| `ohos_source_set` | 源码集合，不生成独立产物 | 无 |
| `ohos_prebuilt_etc` | 预构建配置文件 | 无 |
| `ohos_sa_profile` | System Ability 配置 | 无 |
| `ohos_unittest` | 单元测试 | - |
| `ohos_shared_library` (N-API) | N-API 共享库 | .so (module/) |
| `idl_gen_interface` | IDL 接口生成 | 生成的 .h/.cpp |

---

## 依赖关系图

### 主要依赖流

```
libdlppermission_napi.so
    ↓ 依赖
libdlp_fuse.so
    ↓ 依赖
libdlpparse_inner.so
    ↓ 依赖
libdlp_permission_sdk.so
    ↓ 依赖
libdlp_permission_service.z.so
    ↓ 依赖
外部服务 (samgr, access_token, huks, etc.)
```

### 内部依赖关系

```
dlp_permission_service (服务)
    ├─ 依赖 dlp_hex_string_static
    ├─ 依赖 dlp_permission_serializer_static
    │       ├─ 依赖 dlp_hex_string_static
    │       └─ 依赖 dlp_permission_interface (IDL)
    └─ 依赖 dlp_permission_stub
            └─ 依赖 dlp_permission_interface (IDL)
```

---

## 相关跳转链接

- [编译产物](06_Build_Artifacts.md) - 查看最终产物和安装路径
- [目录结构与模块职责](01_Directory_Structure.md) - 查看代码组织

---

最后更新时间：2026-02-06
