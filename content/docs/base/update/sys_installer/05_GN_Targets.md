# GN Targets

## 目的

本文档详细说明 `sys_installer` 的 GN 构建目标、依赖关系和配置参数。

## 适用范围

需要理解或修改 sys_installer 构建配置的开发人员。

## 构建配置概览

### 配置文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建文件，定义测试分组 |
| `sys_installer_default_cfg.gni` | 默认配置模板 |
| `frameworks/*/BUILD.gn` | 框架层构建 |
| `interfaces/*/BUILD.gn` | 接口层构建 |
| `services/*/BUILD.gn` | 服务层构建 |

### 配置模板 (sys_installer_default_cfg.gni)

```gn
# 声明参数
declare_args() {
  sys_installer_cfg_file = ""
  sys_installer_absolutely_path = "//base/update/sys_installer"
  sys_installer_feature_ohos_cfg = false
}

# 构建模板
template("sys_installer_gen") {
  # 根据配置生成 shared_library 或 static_library
}

template("module_update_gen") {
  # 模块更新库模板
}

template("check_module_update_gen") {
  # 检查模块更新工具模板
}

template("module_update_service_gen") {
  # 模块更新服务模板
}
```

## 核心 Targets

### 1. SA 服务 Targets

#### sys_installer (SA 4101)

**文件**: `frameworks/ipc_server/BUILD.gn`

```gn
ohos_shared_library("sys_installer") {
  defines = [ "SYS_INSTALLER_SERVICE" ]
  sources = [ "src/sys_installer_server.cpp" ]
  
  deps = [
    "//base/update/sys_installer/frameworks/installer_manager:libinstallermanager",
    "//base/update/sys_installer/frameworks/status_manager:libstatusmanager",
    "//base/update/sys_installer/interfaces/innerkits/ipc_client:sys_installer_stub",
    "//base/update/sys_installer/interfaces/innerkits/ipc_client:libsysinstaller_shared",
  ]
  
  external_deps = [
    "access_token:libaccesstoken_sdk",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "updater:libfsmanager",
    "updater:libmiscinfo",
    "ipc:ipc_core",
  ]
}
```

**输出**: `libsys_installer.z.so`

**用途**: SA 4101 服务主库

#### module_update_service (SA 4103)

**文件**: `services/module_update/service/BUILD.gn`

```gn
module_update_service_gen("module_update_service") {
  deps = [
    ":libmodule_update_service_static",
    "//base/update/sys_installer/services/module_update:module_update_utils",
  ]
  
  external_deps = [
    "access_token:libaccesstoken_sdk",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "hvb:libhvb_static_real",
  ]
}
```

**输出**: `libmodule_update_service.z.so`

**用途**: SA 4103 服务主库

### 2. IPC Client Targets

#### libsysinstaller_shared

**文件**: `interfaces/innerkits/ipc_client/BUILD.gn`

```gn
ohos_shared_library("libsysinstaller_shared") {
  defines = [ "SYS_INSTALLER_KITS" ]
  
  sources = [
    "src/sys_installer_kits_impl.cpp",
    "src/sys_installer_callback.cpp",
    "src/sys_installer_load_callback.cpp",
    "src/buffer_info_parcel.cpp",
  ]
  
  deps = [
    ":sysinstaller_interface",
    "//base/update/sys_installer/common:libsysinstaller_common",
  ]
  
  external_deps = [
    "ipc:ipc_core",
    "samgr:samgr_proxy",
    "hilog:libhilog",
  ]
}
```

**输出**: `libsysinstaller_shared.z.so`

**用途**: 客户端共享库

#### libsysinstallerkits

**文件**: `interfaces/innerkits/ipc_client/BUILD.gn`

```gn
ohos_static_library("libsysinstallerkits") {
  # 包含 libsysinstaller_shared 的源文件
  # 作为静态库供其他组件链接
}
```

**输出**: `libsysinstallerkits.a`

**用途**: 客户端静态库

#### libmodule_update_shared

**文件**: `interfaces/innerkits/ipc_client/BUILD.gn`

```gn
ohos_shared_library("libmodule_update_shared") {
  sources = [
    "src/module_update_kits_impl.cpp",
    "src/module_update_proxy.cpp",
    "src/module_update_load_callback.cpp",
  ]
  
  deps = [
    ":sysinstaller_interface",
    "//base/update/sys_installer/common:libsysinstaller_common",
  ]
  
  external_deps = [
    "ipc:ipc_core",
    "samgr:samgr_proxy",
    "hilog:libhilog",
  ]
}
```

**输出**: `libmodule_update_shared.z.so`

**用途**: 模块更新客户端共享库

### 3. 框架层 Targets

#### libinstallermanager

**文件**: `frameworks/installer_manager/BUILD.gn`

```gn
ohos_static_library("libinstallermanager") {
  sources = [
    "src/sys_installer_manager.cpp",
    "src/sys_installer_manager_helper.cpp",
    "src/stream_installer_manager.cpp",
    "src/stream_installer_manager_helper.cpp",
  ]
  
  include_dirs = [
    "${sys_installer_path}/frameworks/installer_manager/include",
    "${sys_installer_path}/common/include",
  ]
  
  deps = [
    "//base/update/sys_installer/frameworks/action_processer:libactionprocesser",
    "//base/update/sys_installer/frameworks/status_manager:libstatusmanager",
  ]
  
  external_deps = [
    "updater:libfsmanager",
    "updater:libupdaterlog",
    "hilog:libhilog",
  ]
}
```

**输出**: `libinstallermanager.a`

#### libstatusmanager

**文件**: `frameworks/status_manager/BUILD.gn`

```gn
ohos_static_library("libstatusmanager") {
  sources = [
    "src/status_manager.cpp",
    "src/stream_status_manager.cpp",
  ]
  
  external_deps = [
    "hilog:libhilog",
    "c_utils:utils",
  ]
}
```

**输出**: `libstatusmanager.a`

#### libactionprocesser

**文件**: `frameworks/action_processer/BUILD.gn`

```gn
ohos_static_library("libactionprocesser") {
  sources = [ "src/action_processer.cpp" ]
  
  external_deps = [
    "hilog:libhilog",
  ]
}
```

**输出**: `libactionprocesser.a`

#### libverifyaction

**文件**: `frameworks/actions/verify_action/BUILD.gn`

```gn
ohos_static_library("libverifyaction") {
  sources = [ "src/pkg_verify.cpp" ]
  
  external_deps = [
    "updater:libupdaterpackage",
    "hilog:libhilog",
  ]
}
```

**输出**: `libverifyaction.a`

### 4. 服务层 Targets

#### module_update_utils

**文件**: `services/module_update/BUILD.gn`

```gn
ohos_shared_library("module_update_utils") {
  sources = [
    "util/src/module_file.cpp",
    "util/src/module_update_verify.cpp",
    "util/src/module_utils.cpp",
    "util/src/module_zip_helper.cpp",
  ]
  
  if (defined(ohos_part_enabled) && ohos_part_enabled("startup_hvb")) {
    sources += [
      "util/src/module_hvb_ops.cpp",
      "util/src/module_hvb_utils.cpp",
    ]
    defines = [ "SUPPORT_HVB" ]
  }
  
  external_deps = [
    "lz4:liblz4_static",
    "zlib:shared_libz",
    "bzip2:libbz2",
    "openssl:libcrypto_shared",
    "hilog:libhilog",
  ]
}
```

**输出**: `libmodule_update_utils.z.so`

**用途**: 模块更新工具库

#### libmodule_update_service_static

**文件**: `services/module_update/service/BUILD.gn`

```gn
ohos_static_library("libmodule_update_service_static") {
  sources = [
    "src/module_update_service.cpp",
    "src/module_update_stub.cpp",
    "src/module_update_consumer.cpp",
    "src/module_update_main.cpp",
    "src/module_update_producer.cpp",
    "src/module_update_queue.cpp",
  ]
  
  defines = [ "WITH_SELINUX" ]
  
  external_deps = [
    "selinux_adapter:librestorecon",
    "hilog:libhilog",
  ]
}
```

**输出**: `libmodule_update_service_static.a`

#### libabupdate

**文件**: `services/ab_update/BUILD.gn`

```gn
ohos_static_library("libabupdate") {
  sources = [ "src/ab_update.cpp" ]
  
  external_deps = [
    "updater:libfsmanager",
    "hilog:libhilog",
  ]
}
```

**输出**: `libabupdate.a`

#### libstreamupdate

**文件**: `services/stream_update/BUILD.gn`

```gn
ohos_static_library("libstreamupdate") {
  sources = [ "src/stream_update.cpp" ]
  
  external_deps = [
    "updater:libringbuffer",
    "hilog:libhilog",
  ]
}
```

**输出**: `libstreamupdate.a`

### 5. 可执行文件 Targets

#### sys_installer_client

**文件**: `interfaces/innerkits/ipc_client/BUILD.gn`

```gn
ohos_executable("sys_installer_client") {
  sources = [ "src/sys_installer_client.cpp" ]
  deps = [ ":libsysinstaller_shared" ]
}
```

**输出**: `sys_installer_client`

**用途**: 测试客户端

#### module_update_client

**文件**: `interfaces/innerkits/ipc_client/BUILD.gn`

```gn
ohos_executable("module_update_client") {
  sources = [ "services/module_update/service/main.cpp" ]
  deps = [ ":module_update" ]
}
```

**输出**: `module_update_client`

**用途**: 模块更新客户端

#### check_module_update_init

**文件**: `services/module_update/src/BUILD.gn`

```gn
check_module_update_gen("check_module_update") {
  sources = [ "main.cpp" ]
  deps = [
    ":module_update_static",
    "//base/update/sys_installer/services/module_update:module_update_utils",
  ]
}
```

**输出**: `check_module_update_init`

**用途**: 启动时模块检查

#### module_update_tool

**文件**: `tools/module_update_tool/BUILD.gn`

```gn
ohos_executable("module_update_tool") {
  sources = [ "main.cpp" ]
  deps = [
    "//base/update/sys_installer/interfaces/innerkits/ipc_client:libmodule_update_shared",
  ]
}
```

**输出**: `module_update_tool`

**用途**: 模块更新 CLI 工具

### 6. SA Profile Targets

#### sys_installer_sa_profile

**文件**: `frameworks/ipc_server/sa_profile/BUILD.gn`

```gn
ohos_sa_profile("sys_installer_sa_profile") {
  sources = [ "4101.json" ]
}
```

**配置内容**:
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

#### module_update_sa_profile

**文件**: `frameworks/ipc_server/sa_profile/BUILD.gn`

```gn
ohos_sa_profile("module_update_sa_profile") {
  sources = [ "4103.json" ]
}
```

**配置内容**:
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

### 7. ETC 配置 Targets

#### sys_installer_etc

**文件**: `frameworks/ipc_server/etc/BUILD.gn`

```gn
ohos_prebuilt_etc("sys_installer_etc") {
  source = "sys_installer.cfg"
  relative_install_dir = "init"
}

ohos_prebuilt_etc("sys_installer_para") {
  source = "sys_installer.para"
  relative_install_dir = "param"
}

ohos_prebuilt_etc("sys_installer_sa_rc") {
  source = "sys_installer_sa.rc"
  relative_install_dir = "init"
}
```

## 依赖关系图

```
libsysinstaller_shared
    ├── sysinstaller_interface (IDL)
    ├── libsysinstaller_common
    ├── ipc:ipc_core
    ├── samgr:samgr_proxy
    └── hilog:libhilog

sys_installer
    ├── libinstallermanager
    │   ├── libactionprocesser
    │   └── libstatusmanager
    ├── libstatusmanager
    ├── sys_installer_stub
    ├── libsysinstaller_shared
    ├── access_token:libaccesstoken_sdk
    ├── safwk:system_ability_fwk
    ├── samgr:samgr_proxy
    └── updater:libfsmanager

module_update_service
    ├── libmodule_update_service_static
    ├── module_update_utils
    ├── access_token:libaccesstoken_sdk
    ├── safwk:system_ability_fwk
    └── hvb:libhvb_static_real

module_update_utils
    ├── lz4:liblz4_static
    ├── zlib:shared_libz
    ├── bzip2:libbz2
    ├── openssl:libcrypto_shared
    └── hilog:libhilog
```

## 关键 Defines

| Define | 定义位置 | 用途 |
|--------|----------|------|
| `SYS_INSTALLER_SERVICE` | ipc_server | 服务端代码编译 |
| `SYS_INSTALLER_KITS` | ipc_client | 客户端代码编译 |
| `SUPPORT_HVB` | module_update | HVB 支持 |
| `WITH_SELINUX` | module_update/service | SELinux 支持 |

## 产物清单

| Target | 类型 | 输出文件名 | 安装路径 |
|--------|------|------------|----------|
| sys_installer | shared_library | libsys_installer.z.so | /system/lib/ |
| module_update_service | shared_library | libmodule_update_service.z.so | /system/lib/ |
| libsysinstaller_shared | shared_library | libsysinstaller_shared.z.so | /system/lib/ |
| libsysinstallerkits | static_library | libsysinstallerkits.a | N/A (编译时) |
| libmodule_update_shared | shared_library | libmodule_update_shared.z.so | /system/lib/ |
| libinstallermanager | static_library | libinstallermanager.a | N/A (编译时) |
| libstatusmanager | static_library | libstatusmanager.a | N/A (编译时) |
| module_update_utils | shared_library | libmodule_update_utils.z.so | /system/lib/ |
| sys_installer_client | executable | sys_installer_client | /system/bin/ |
| module_update_client | executable | module_update_client | /system/bin/ |
| check_module_update | executable | check_module_update_init | /system/bin/ |
| module_update_tool | executable | module_update_tool | /system/bin/ |

## 相关链接

- [目录结构](01_Directory_Structure.md)
- [内部 API](04_Internal_APIs.md)
- [编译产物](07_Build_Products.md)

---

*证据来源*:
- `BUILD.gn`: 根构建配置
- `sys_installer_default_cfg.gni`: 模板配置
- `frameworks/ipc_server/BUILD.gn`: SA 服务构建
- `interfaces/innerkits/ipc_client/BUILD.gn`: IPC Client 构建
- `bundle.json`: 产物定义
