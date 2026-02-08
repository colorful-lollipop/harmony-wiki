# User File Service - GN 构建系统

## 概述

user_file_service 使用 OpenHarmony 的 GN (Generate Ninja) 构建系统进行编译管理。

**构建入口**：`//foundation/filemanagement/user_file_service/BUILD.gn`

**构建系统**：`//build/ohos.gni`

---

## 构建配置

### 根配置

```gn
import("//build/ohos.gni")
```

**证据**：`BUILD.gn:14`

### 特性开关 (filemanagement_aafwk.gni)

**文件**：`filemanagement_aafwk.gni`

```gn
declare_args() {
    picker_udmf_enabled = true
    user_file_service_cloud_disk_enable = false
    ufs_sandbox_manarer = false
}
```

| 开关 | 默认值 | 作用 |
|------|--------|------|
| `picker_udmf_enabled` | true | 启用 Picker 的 UMD 支持 |
| `user_file_service_cloud_disk_enable` | false | 启用云盘管理 |
| `ufs_sandbox_manarer` | false | 启用沙箱管理器 |

---

## 主要 Targets

### 服务层 Targets

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `user_file_managers` | group | - | 聚合目标 |
| `file_access_service` | ohos_shared_library | libfile_access_service.z.so | 文件访问服务 |
| `external_file_manager_hap` | ohos_hap | external_file_manager.hap | 外部文件管理器 |

**证据**：`services/BUILD.gn:18-25`

#### file_access_service 详细配置

```gn
ohos_shared_library("file_access_service") {
    branch_protector_ret = "pac_ret"
    sanitize = {
        integer_overflow = true
        ubsan = true
        boundary_sanitize = true
        cfi = true
        cfi_cross_dso = true
    }

    shlib_type = "sa"  // System Ability

    include_dirs = [
        "${user_file_service_path}/interfaces/inner_api/cloud_disk_kit_inner/include",
        "${user_file_service_path}/interfaces/inner_api/file_access/include",
        "${user_file_service_path}/services/native/notify_event/include",
        "${user_file_service_path}/services/rdb_adapter/include",
        "${user_file_service_path}/services/native/file_access_service/include",
    ]

    sources = [
        "${user_file_service_path}/interfaces/inner_api/cloud_disk_kit_inner/src/cloud_disk_comm.cpp",
        "${user_file_service_path}/interfaces/inner_api/file_access/src/uri_ext.cpp",
        "native/file_access_service/src/bundle_observer.cpp",
        "native/file_access_service/src/file_access_ext_connection.cpp",
        "native/file_access_service/src/file_access_service.cpp",
        "native/file_access_service/src/file_access_service_client.cpp",
        "native/notify_event/src/notify_work_service.cpp",
        "native/file_access_service/src/ufs_access_token_helper.cpp",
        "rdb_adapter/src/ufs_rdb_adapter.cpp",
        "rdb_adapter/src/ufs_db_services_constants.cpp",
        "native/cloud_disk_service/src/cloud_disk_synchronous_root_manager.cpp",
        "native/cloud_disk_service/src/cloud_disk_service.cpp"
    ]

    external_deps = [
        "ability_runtime:extension_manager",
        "access_token:libaccesstoken_sdk",
        "ipc:ipc_core",
        "safwk:system_ability_fwk",
        "samgr:samgr_proxy",
        "relational_store:native_rdb",
    ]

    if (user_file_service_cloud_disk_enable) {
        defines += ["SUPPORT_CLOUD_DISK_MANAGER"]
        external_deps += ["dfs_service:clouddiskservice_kit_inner"]
    }

    subsystem_name = "filemanagement"
    part_name = "user_file_service"
}
```

### N-API 层 Targets

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `fileaccess` | ohos_shared_library | libfileaccess.z.so | 文件访问 N-API |
| `picker` | ohos_shared_library | libpicker.z.so | 文件选择器 |
| `cj_picker_ffi` | ohos_shared_library | libcj_picker_ffi.z.so | CJ FFI |

**证据**：`frameworks/js/napi/file_access_module/BUILD.gn`

#### fileaccess 配置

```gn
ohos_shared_library("fileaccess") {
    branch_protector_ret = "pac_ret"
    sanitize = {
        integer_overflow = true
        ubsan = true
        boundary_sanitize = true
        cfi = true
        cfi_cross_dso = true
    }

    relative_install_dir = "module/file"

    include_dirs = [
        "${user_file_service_path}/frameworks/js/napi/common",
        "${user_file_service_path}/frameworks/js/napi/file_access_module",
        "${user_file_service_path}/frameworks/js/napi/file_access_module/file_info",
        "${user_file_service_path}/frameworks/js/napi/file_access_module/root_info",
        "${user_file_service_path}/utils",
        "${user_file_service_path}/services/native/file_access_service/include",
        "${user_file_service_path}/interfaces/kits/js/src/common",
    ]

    sources = [
        "${user_file_service_path}/frameworks/js/napi/common/file_extension_info_napi.cpp",
        "file_info/napi_file_info_exporter.cpp",
        "file_info/napi_file_iterator_exporter.cpp",
        "napi_fileaccess_helper.cpp",
        "napi_observer_callback.cpp",
        "napi_utils.cpp",
        "native_fileaccess_module.cpp",
        "root_info/napi_root_info_exporter.cpp",
        "root_info/napi_root_iterator_exporter.cpp",
    ]

    deps = [
        "${user_file_service_path}/interfaces/inner_api/file_access:file_access_ext_base_include",
        "${user_file_service_path}/interfaces/inner_api/file_access:file_access_extension_ability_kit",
        "${user_file_service_path}/services:file_access_service",
        "${user_file_service_path}/services:file_access_service_base_include",
    ]

    external_deps = [
        "ability_runtime:abilitykit_native",
        "bundle_framework:appexecfwk_core",
        "c_utils:utils",
        "file_api:filemgmt_libn",
        "napi:ace_napi",
    ]

    subsystem_name = "filemanagement"
    part_name = "user_file_service"
}
```

### Picker Targets

**证据**：`interfaces/kits/picker/BUILD.gn`

```gn
ohos_shared_library("picker") {
    # ... 配置 ...

    if (picker_udmf_enabled) {
        defines = [ "UDMF_ENABLED" ]
        external_deps += [ "udmf:udmf_client" ]
    }
}

ohos_shared_library("cj_picker_ffi") {
    # CJ FFI 配置
    innerapi_tags = [ "platformsdk" ]
}
```

---

## 接口 Targets (Inner Kits)

| Target | 类型 | 头文件目录 |
|--------|------|-----------|
| `file_access_extension_ability_kit` | - | `interfaces/inner_api/file_access/include` |
| `file_access_ext_base_include` | - | IDL 生成 |
| `cloud_disk_manager_kit` | - | `interfaces/inner_api/cloud_disk_kit_inner/include` |

**证据**：`bundle.json:79-110`

---

## 构建依赖

### 外部依赖

| 依赖组件 | 子组件 | 用途 |
|----------|--------|------|
| `ability_runtime` | extension_manager, napi_common | Ability 框架 |
| `access_token` | libaccesstoken_sdk | 权限管理 |
| `ipc` | ipc_core, ipc_single | 进程通信 |
| `safwk` | system_ability_fwk | SA 框架 |
| `samgr` | samgr_proxy | 服务管理 |
| `relational_store` | native_rdb | 关系数据库 |
| `napi` | ace_napi | N-API 框架 |
| `bundle_framework` | appexecfwk_core | 包管理 |
| `hilog` | libhilog | 日志 |
| `hitrace` | hitrace_meter | 追踪 |

### 内部依赖

```
fileaccess ──────► services:file_access_service
                 ├─► file_access_service_base_include
                 └─► file_access_ext_base_include

picker ──────────► file_access_service
cj_picker_ffi ───► picker
```

---

## 构建命令

### 全量编译

```bash
# 编译 user_file_service
./build.sh --product-name {product} --parts user_file_service

# 或使用 gn + ninja
gn gen out/{product}
ninja -C out/{product} user_file_service
```

### 单独编译

```bash
# 编译文件访问服务
ninja -C out/{product} //foundation/filemanagement/user_file_service/services:file_access_service

# 编译 N-API
ninja -C out/{product} //foundation/filemanagement/user_file_service/frameworks/js/napi/file_access_module:fileaccess

# 编译 Picker
ninja -C out/{product} //foundation/filemanagement/user_file_service/interfaces/kits/picker:picker
```

---

## 构建产物

| 产物类型 | 格式 | 说明 |
|----------|------|------|
| 共享库 | `.z.so` | 系统能力或 N-API |
| HAP | `.hap` | 外部文件管理器 |
| 配置 | `.cfg` | SA 配置 |
| Profile | `.json` | SA Profile |

---

## 安全编译选项

所有共享库目标启用以下安全选项：

```gn
sanitize = {
    integer_overflow = true    # 整数溢出检测
    ubsan = true               # 未定义行为检测
    boundary_sanitize = true   # 边界检测
    cfi = true                  # 控制流完整性
    cfi_cross_dso = true        # 跨 DSO CFI
}

branch_protector_ret = "pac_ret"  # 返回地址保护 (PAC)
```

---

## 编译配置示例

### 启用云盘管理

```bash
# 构建时启用
./build.sh --product-name {product} \
    --parts user_file_service \
    --gn-args user_file_service_cloud_disk_enable=true
```

### 启用沙箱管理器

```bash
./build.sh --product-name {product} \
    --parts user_file_service \
    --gn-args ufs_sandbox_manarer=true
```
