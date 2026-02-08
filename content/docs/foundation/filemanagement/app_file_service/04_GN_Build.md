# GN 构建配置

## 5.1 构建系统概述

应用文件服务使用 OpenHarmony 的 GN（Generate Ninja）构建系统。GN 构建系统具有声明式、高性能、可扩展等特点，广泛应用于 Chromium、OpenHarmony 等大型项目。所有构建配置使用 `.gn` 和 `.gni` 文件定义。

### 5.1.1 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 项目根构建文件，包含顶层 target 定义 |
| `app_file_service.gni` | 全局路径变量和特性开关定义 |
| `backup.gni` | 备份模块路径定义 |

### 5.1.2 路径变量

**文件**：`app_file_service.gni`

```gn
app_file_service_path = "//foundation/filemanagement/app_file_service"
```

**文件**：`backup.gni`

```gn
path_backup = "//foundation/filemanagement/app_file_service"
```

## 5.2 顶层 Targets

### 5.2.1 框架组 Targets

**文件**：`BUILD.gn`

```gn
group("tgt_backup_extension") {
  deps = [
    "frameworks/native/backup_ext:backup_extension_ability_native",
    "interfaces/api/js/napi/backup_ext:backupextensionability_napi",
    "interfaces/api/js/napi/backup_ext_context:backupextensioncontext_napi",
  ]
}

group("tgt_backup_kit_inner") {
  deps = [ "interfaces/inner_api/native/backup_kit_inner" ]
}

group("backup_tests") {
  testonly = true
  deps = [
    "tests/moduletests",
    "tests/unittests",
  ]
}

group("file_share_tests") {
  testonly = true
  deps = [ "test/unittest" ]
}
```

### 5.2.2 服务组 Targets

```gn
group("tgt_backup_sa") {
  deps = [
    "services:backup_para_etc",
    "services:backup_sa_etc",
    "services:backup_sa_profile",
    "services/backup_sa",
  ]
}
```

## 5.3 服务层构建

### 5.3.1 Backup SA 构建

**文件**：`services/backup_sn`

```gn
ohos_shared_library("backup_sa") {
  branch_protector_ret = "pac_ret"
  
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  
  sources = [
    "src/module_app_gallery/app_gallery_dispose_proxy.cpp",
    "src/module_app_gallery/app_gallery_service_connection.cpp",
    "src/module_external/bms_adapter.cpp",
    "src/module_external/sms_adapter.cpp",
    "src/module_ipc/sa_backup_connection.cpp",
    "src/module_ipc/service.cpp",
    "src/module_ipc/service_incremental.cpp",
    "src/module_ipc/sub_service.cpp",
    "src/module_ipc/svc_backup_connection.cpp",
    "src/module_ipc/svc_restore_deps_manager.cpp",
    "src/module_ipc/svc_session_manager.cpp",
    "src/module_notify/notify_work_service.cpp",
    "src/module_sched/sched_scheduler.cpp",
    "src/module_external/storage_manager_service.cpp",
  ]
  
  defines = [
    "LOG_DOMAIN=0xD004303",
    "LOG_TAG=\"BackupSA\"",
  ]
  
  include_dirs = [
    "include",
    "include/module_notify",
    "${path_backup}/interfaces/inner_api/native/backup_kit_inner/impl",
  ]
  
  deps = [
    ":backup_sa_ipc",
    "${path_backup}/interfaces/inner_api/native/backup_kit_inner:backup_kit_inner",
    "${path_backup}/utils:backup_utils",
    "${path_backup}/interfaces/innerkits/native:sandbox_helper_native",
    "${path_backup}/interfaces/innerkits/native:fileuri_native",
  ]
  
  external_deps = [
    "ability_base:want",
    "ability_runtime:ability_connect_callback_stub",
    "ability_runtime:ability_manager",
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
    "ipc:ipc_core",
    "jsoncpp:jsoncpp",
    "os_account:os_account_innerkits",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "storage_service:storage_manager_sa_proxy",
    "data_share:datashare_consumer",
    "data_share:datashare_common",
  ]
  
  if (power_mgr_enabled) {
    external_deps += ["power_manager:powermgr_client"]
    defines += ["POWER_MANAGER_ENABLED"]
  }
  
  cflags_cc = [
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-unwind-tables",
    "-fno-asynchronous-unwind-tables",
    "-Os",
  ]
  
  use_exceptions = true
  part_name = "app_file_service"
  subsystem_name = "filemanagement"
}
```

### 5.3.2 IDL 生成

```gn
idl_gen_interface("backup_idl") {
  sources = [
    "IExtension.idl",
    "IService.idl",
    "IServiceReverse.idl",
  ]
  sources_common = [
    "ServiceReverseType.idl",
    "ServiceType.idl",
  ]
  hitrace = "HITRACE_TAG_FILEMANAGEMENT"
  log_domainid = "0xD004313"
  log_tag = "AppFileService"
}
```

## 5.4 接口层构建

### 5.4.1 JS N-API 构建

**文件**：`interfaces/kits/js/BUILD.gn`

```gn
ohos_shared_library("fileshare") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    integer_overflow = true
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  
  include_dirs = [
    "${app_file_service_path}/interfaces",
    "${app_file_service_path}/interfaces/common/include",
    "${app_file_service_path}/interfaces/innerkits/native/file_share/include",
  ]
  
  sources = [
    "${app_file_service_path}/interfaces/common/src/json_utils.cpp",
    "${app_file_service_path}/interfaces/common/src/sandbox_helper.cpp",
    "${app_file_service_path}/interfaces/innerkits/native/file_share/src/file_permission.cpp",
    "file_share/fileshare_n_exporter.cpp",
    "file_share/grant_permissions.cpp",
    "file_share/grant_uri_permission.cpp",
  ]
  
  deps = [ "${app_file_service_path}/interfaces/innerkits/native:fileuri_native" ]
  
  external_deps = [
    "ability_base:want",
    "ability_base:zuri",
    "ability_runtime:abilitykit_native",
    "ability_runtime:extensionkit_native",
    "ability_runtime:uri_permission_mgr",
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "bundle_framework:appexecfwk_base",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "data_share:datashare_common",
    "data_share:datashare_consumer",
    "file_api:filemgmt_libhilog",
    "file_api:filemgmt_libn",
    "file_api:remote_uri_native",
    "hilog:libhilog",
    "init:libbegetutil",
    "ipc:ipc_core",
    "napi:ace_napi",
  ]
  
  if (sandbox_manarer) {
    external_deps += [ "sandbox_manager:libsandbox_manager_sdk" ]
    defines += [
      "SANDBOX_MANAGER",
      "ABILITY_RUNTIME_FEATURE_SANDBOXMANAGER",
    ]
  }
  
  relative_install_dir = "module"
  part_name = "app_file_service"
  subsystem_name = "filemanagement"
}
```

### 5.4.2 InnerKit 构建

```gn
ohos_shared_library("fileshare_native") {
  sources = [
    "file_permission.cpp",
    "file_share.cpp",
  ]
  
  deps = [
    "${app_file_service_path}/utils:backup_utils",
    "${app_file_service_path}/interfaces/common/src:common_func",
  ]
  
  external_deps = [
    "ability_base:zuri",
    "bundle_framework:appexecfwk_base",
    "c_utils:utils",
    "file_api:filemgmt_libhilog",
    "file_api:filemgmt_libn",
    "ipc:ipc_core",
  ]
}

ohos_shared_library("fileuri_native") {
  sources = [
    "file_uri.cpp",
    "common_func.cpp",
  ]
  
  external_deps = [
    "ability_base:zuri",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "c_utils:utils",
    "file_api:filemgmt_libhilog",
    "file_api:filemgmt_libn",
    "hilog:libhilog",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]
}
```

## 5.5 工具库构建

### 5.5.1 静态库依赖

```gn
ohos_static_library("backup_cxx_cppdeps") {
  defines = [
    "LOG_DOMAIN=0xD004305",
    "LOG_TAG=\"BackupUtils\"",
  ]
  
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
}
```

### 5.5.2 动态库

```gn
ohos_shared_library("backup_utils") {
  sources = [
    "src/b_anony/",
    "src/b_encryption/",
    "src/b_error/",
    "src/b_filesystem/",
    "src/b_hiaudit/",
    "src/b_json/",
    "src/b_jsonutil/",
    "src/b_ohos/",
    "src/b_process/",
    "src/b_radar/",
    "src/b_sa/",
    "src/b_tarball/",
    "src/b_utils/",
  ]
  
  configs += [ ":utils_private_config" ]
  public_configs += [ ":utils_public_config" ]
  
  deps = [
    ":backup_cxx_cppdeps",
    "${path_backup}/interfaces/innerkits/native:sandbox_helper_native",
  ]
  
  external_deps = [
    "libaccesstoken_sdk",
    "libtokenid_sdk",
    "cjson:cjson",
    "c_utils:utils",
    "libdfx_dumpcatcher:libdfx_dumpcatcher",
    "hilog:libhilog",
    "hisysevent:hisysevent_inner",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
    "ipc:ipc_core",
    "jsoncpp:jsoncpp",
    "openssl:libcrypto_shared",
    "zlib:shared_libz",
  ]
  
  part_name = "app_file_service"
  subsystem_name = "filemanagement"
}
```

## 5.6 产物映射

### 5.6.1 产物清单

| Target | 类型 | 输出文件 | 安装路径 |
|--------|------|----------|----------|
| `backup_sa` | SA | `libbackup_sa.so` | `system/lib/${arch}/` |
| `backup_utils` | Utils | `libbackup_utils.so` | `system/lib/${arch}/` |
| `fileshare` | JS N-API | `libfileshare.so` | `system/etc/module/` |
| `fileuri` | JS N-API | `libfileuri.so` | `system/etc/module/file/` |
| `backup` | JS N-API | `libbackup.so` | `system/etc/module/file/` |
| `fileshare_native` | InnerKit | `libfileshare_native.so` | `system/lib/${arch}/` |
| `fileuri_native` | InnerKit | `libfileuri_native.so` | `system/lib/${arch}/` |
| `backup_kit_inner` | InnerAPI | `libbackup_kit_inner.so` | `system/lib/${arch}/` |
| `ohfileshare` | NDK | `libohfileshare.so` | `system/lib/${arch}/` |
| `ohfileuri` | NDK | `libohfileuri.so` | `system/lib/${arch}/` |
| `backup_extension_ability_native` | Extension | `libbackup_extension_ability_native.so` | `system/extensionability/` |

### 5.6.2 配置文件

| 配置类型 | 文件 | 用途 |
|----------|------|------|
| SA Profile | `5203.json` | SA 注册配置 |
| Init Config | `backup.cfg` | 服务启动参数 |
| 参数配置 | `backup.para` | 系统参数 |
| Sandbox 配置 | `file_share_sandbox.json` | 文件分享沙箱 |
| Sandbox 配置 | `backup_sandbox.json` | 备份沙箱 |

## 5.7 构建命令

### 5.7.1 完整构建

```bash
# 构建整个 subsystem
hb build app_file_service

# 或使用 ninja 直接构建
ninja -C out/<product> app_file_service:fwk_group
ninja -C out/<product> app_file_service:service_group
```

### 5.7.2 单模块构建

```bash
# 构建 JS N-API
ninja -C out/<product> //foundation/filemanagement/app_file_service/interfaces/kits/js:fileshare
ninja -C out/<product> //foundation/filemanagement/app_file_service/interfaces/kits/js:fileuri
ninja -C out/<product> //foundation/filemanagement/app_file_service/interfaces/kits/js:backup

# 构建 SA
ninja -C out/<product> //foundation/filemanagement/app_file_service/services/backup_sa:backup_sa

# 构建工具库
ninja -C out/<product> //foundation/filemanagement/app_file_service/utils:backup_utils
```

## 5.8 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](00_Overview.md) | 构建产物用途 |
| [系统架构](01_Architecture.md) | 组件依赖关系 |
| [备份服务 SA](02_Service_SA.md) | SA 构建配置 |
| [工具库](03_Utils.md) | Utils 构建配置 |
| [JS N-API 接口](10_NAPI_JS.md) | JS 接口构建 |
| [NDK 接口](11_NAPI_NDK.md) | NDK 接口构建 |
