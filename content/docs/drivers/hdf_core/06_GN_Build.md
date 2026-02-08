# HDF Core GN 构建系统

## 1. 概述

HDF Core 使用 **GN**（Generate Ninja）构建系统，支持多平台、多系统类型的构建。

**构建入口**: `//drivers/hdf_core/adapter/BUILD.gn`

## 2. 构建目标

### 2.1 主构建目标

```gn
# adapter/BUILD.gn
group("uhdf_entry") {
  deps = [
    "//drivers/hdf_core/adapter/uhdf2/hdi:libhdi",
    "//drivers/hdf_core/adapter/uhdf2/hdi:libhdi_base",
    "//drivers/hdf_core/adapter/uhdf2/host:libhdf_host",
    "//drivers/hdf_core/adapter/uhdf2/ipc:libhdf_ipc_adapter",
    "//drivers/hdf_core/adapter/uhdf2/manager:hdf_devmgr",
    "//drivers/hdf_core/adapter/uhdf2/platform:libhdf_platform",
    "//drivers/hdf_core/adapter/uhdf2/pub_utils:libpub_utils",
    "//drivers/hdf_core/adapter/uhdf2/utils:libhdf_utils",
    "//drivers/hdf_core/framework/tools/hdf_dbg:hdf_dbg",
  ]
}
```

### 2.2 UHDF 共享库

| Target | 类型 | 输出 | 路径 | 说明 |
|--------|------|------|------|------|
| libhdi | ohos_shared_library | libhdi.z.so | uhdf2/hdi | HDI 主库 |
| libhdi_base | ohos_shared_library | libhdi_base.z.so | uhdf2/hdi | HDI 基础库 |
| libhdf_host | ohos_shared_library | libhdf_host.z.so | uhdf2/host | Host 环境库 |
| libhdf_ipc_adapter | ohos_shared_library | libhdf_ipc_adapter.z.so | uhdf2/ipc | IPC 适配库 |
| libhdf_platform | ohos_shared_library | libhdf_platform.z.so | uhdf2/platform | 平台驱动库 |
| libhdf_utils | ohos_shared_library | libhdf_utils.z.so | uhdf2/utils | 工具库 |
| libpub_utils | ohos_shared_library | libpub_utils.z.so | uhdf2/pub_utils | 公共工具库 |
| libhdf_sec | ohos_shared_library | libhdf_sec.z.so | uhdf2/security | 安全库 |

### 2.3 UHDF 可执行文件

| Target | 类型 | 输出 | 路径 | 说明 |
|--------|------|------|------|------|
| hdf_devmgr | ohos_executable | hdf_devmgr | uhdf2/manager | 设备管理器 |
| hdf_dbg | ohos_executable | hdf_dbg | tools/hdf_dbg | 调试工具 |

### 2.4 UHDF 配置文件

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| hdf_devmgr.cfg | ohos_prebuilt_etc | init/hdf_devmgr.cfg | Init 配置 |
| hdf_pnp.cfg | ohos_prebuilt_etc | hdfconfig/hdf_pnp.cfg | 即插即用配置 |
| hdf_devmgr.para.dac | ohos_prebuilt_etc | param/hdf_devmgr.para.dac | 参数配置 |
| hdf_peripheral.cfg | ohos_prebuilt_etc | init/hdf_peripheral.cfg | 外设配置（可选） |

## 3. BUILD.gn 详解

### 3.1 libhdi 示例

```gn
# adapter/uhdf2/hdi/BUILD.gn
ohos_shared_library("libhdi") {
  sources = [
    "src/servmgr_client.c",
    "src/devmgr_client.c",
    "src/iservmgr_client.cpp",
    "src/idevmgr_client.cpp",
    "src/stub_collector.cpp",
    "src/hdi_support.cpp",
  ]

  include_dirs = [
    "$hdf_uhdf_path/hdi/include",
    "$hdf_framework_path/include",
    "$hdf_framework_path/include/core",
    "$hdf_framework_path/include/utils",
    "$hdf_framework_path/include/osal",
    "$hdf_uhdf_path/ipc/include",
    "$hdf_uhdf_path/utils/include",
    "$hdf_uhdf_path/pub_utils/include",
    "$hdf_interfaces_path/inner_api",
    "$hdf_interfaces_path/inner_api/hdi",
    "$hdf_interfaces_path/inner_api/core",
    "$hdf_interfaces_path/inner_api/utils",
    "$hdf_interfaces_path/inner_api/ipc",
    "$hdf_interfaces_path/inner_api/osal/uhdf",
  ]

  deps = [
    "//drivers/hdf_core/adapter/uhdf2/ipc:libhdf_ipc_adapter",
    "//drivers/hdf_core/adapter/uhdf2/pub_utils:libpub_utils",
    "//drivers/hdf_core/adapter/uhdf2/hdi:libhdi_base",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "samgr:samgr_proxy",
  ]

  defines = [ "__USER__" ]

  install_images = [ "system", "updater" ]
  subsystem_name = "hdf"
  part_name = "hdf_core"
}
```

### 3.2 hdf_devmgr 示例

```gn
# adapter/uhdf2/manager/BUILD.gn
ohos_executable("hdf_devmgr") {
  sources = [ "src/devmgr_main.c" ]

  include_dirs = [
    "$hdf_uhdf_path/manager/include",
    "$hdf_framework_path/include/core",
    "$hdf_framework_path/include/utils",
    "$hdf_framework_path/include/osal",
    "$hdf_interfaces_path/inner_api",
  ]

  deps = [
    ":libhdf_ipc_adapter",
    ":libhdf_utils",
    ":hdf_devmgr.cfg",
    ":hdf_devmgr.para.dac",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "init:libbegetutil",
  ]

  install_images = [ "system", "updater" ]
  subsystem_name = "hdf"
  part_name = "hdf_core"
}
```

## 4. 构建配置

### 4.1 uhdf.gni 配置

**文件**: `adapter/uhdf2/uhdf.gni`

```gn
hdf_framework_path = "//drivers/hdf_core/framework"
hdf_uhdf_path = "//drivers/hdf_core/adapter/uhdf2"

# 参数声明
declare_args() {
  hdf_core_default_hicollie_config = false
}

declare_args() {
  hicollie_enabled = false
  
  if (hdf_core_default_hicollie_config) {
    hicollie_enabled = hdf_core_default_hicollie_config
  }
  
  with_sample = false                    # 包含示例驱动
  hdf_core_default_peripheral_config = true   # 默认外设配置
}
```

### 4.2 Feature 开关

| Feature | 默认值 | 说明 |
|---------|--------|------|
| hdf_core_default_hicollie_config | false | 默认 hicollie 配置 |
| hicollie_enabled | false | 启用 hicollie 看门狗 |
| with_sample | false | 包含示例驱动 |
| hdf_core_default_peripheral_config | true | 包含外设配置 |
| hdf_core_khdf_test_support | - | KHDF 测试支持 |
| hdf_core_platform_test_support | - | 平台测试支持 |

### 4.3 编译定义

| 定义 | 说明 |
|------|------|
| `__USER__` | 用户态构建 |
| `__OHOS_USER__` | OpenHarmony 用户态 |
| `__OHOS_STANDARD_SYS__` | 标准系统 |
| `HDFHICOLLIE_ENABLE` | 启用 hicollie |
| `WITH_SELINUX` | 启用 SELinux |

## 5. 依赖关系

### 5.1 UHDF 依赖图

```
libhdi
├── libhdf_ipc_adapter
│   └── libpub_utils
├── libpub_utils
└── libhdi_base

libhdf_host
├── libhdf_ipc_adapter
└── libhdf_utils

hdf_devmgr
├── libhdf_ipc_adapter
└── libhdf_utils

hdf_dbg
├── libhdi
└── libhdf_utils
```

### 5.2 外部依赖

```
bounds_checking_function:libsec_shared
c_utils:utils
hilog:libhilog / hilog_lite:hilog_shared
ipc:ipc_single / ipc:ipc_core
samgr:samgr_proxy
init:libbegetutil
selinux_adapter:libservice_checker (optional)
hicollie:libhicollie (optional)
```

## 6. 编译产物

### 6.1 输出目录结构

```
out/
└── [target]/
    ├── system/
    │   ├── lib/
    │   │   ├── libhdi.z.so
    │   │   ├── libhdi_base.z.so
    │   │   ├── libhdf_host.z.so
    │   │   ├── libhdf_ipc_adapter.z.so
    │   │   ├── libhdf_platform.z.so
    │   │   ├── libhdf_utils.z.so
    │   │   └── libpub_utils.z.so
    │   ├── bin/
    │   │   ├── hdf_devmgr
    │   │   └── hdf_dbg
    │   └── etc/
    │       ├── init/hdf_devmgr.cfg
    │       └── hdfconfig/
    └── updater/...
```

### 6.2 运行时加载关系

```
hdf_devmgr (进程)
├── libhdf_ipc_adapter.z.so (IPC 适配)
├── libhdf_utils.z.so (工具库)
└── 加载驱动 .so 文件

系统服务
├── libhdi.z.so (HDI 接口)
├── libhdf_ipc_adapter.z.so (IPC 适配)
└── 通过 IPC 调用 hdf_devmgr
```

## 7. KHDF 构建

### 7.1 LiteOS-A

**文件**: `adapter/khdf/liteos/BUILD.gn`

```gn
import("hdf.gni")

# HDF 驱动模板
hdf_driver("hdf_driver") {
  sources = [ ... ]
  include_dirs = [ ... ]
}
```

### 7.2 LiteOS-M

**文件**: `adapter/khdf/liteos_m/BUILD.gn`

```gn
import("hdf.gni")

hdf_driver("hdf_lite") {
  sources = [ ... ]
  macro_switch = true  # 使用宏配置
}
```

### 7.3 KHDF 目标

| Target | 系统 | 输出 |
|--------|------|------|
| hdf | LiteOS-A | 内核模块 |
| hdf_lite | LiteOS-M | 内核模块 |
| hdf_lite | UniProton | 内核模块 |

## 8. 构建命令

### 8.1 完整构建

```bash
# 构建 UHDF
gn gen out --args='...'
ninja -C out drivers/hdf_core/adapter:uhdf_entry

# 构建 KHDF (LiteOS-A)
ninja -C out drivers/hdf_core/adapter/khdf/liteos:hdf
```

### 8.2 单独构建

```bash
# 构建 libhdi
ninja -C out drivers/hdf_core/adapter/uhdf2/hdi:libhdi

# 构建 hdf_devmgr
ninja -C out drivers/hdf_core/adapter/uhdf2/manager:hdf_devmgr
```

## 9. 相关文档

- [项目概览](./01_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [内部 API](./05_Inner_API.md)
