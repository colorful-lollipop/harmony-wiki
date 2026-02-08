# GN 构建系统说明

本文档描述扩展外部设备管理模块的 GN 构建配置，包括 Targets 清单、依赖关系、编译产物和构建参数。这些信息对于理解模块的构建过程、进行定制化编译和问题定位具有重要意义。

## 构建系统概览

扩展外部设备管理模块使用 GN（Generate Ninja）作为构建系统，构建配置文件分布在多个 `BUILD.gn` 文件中。模块共包含 56 个 BUILD.gn 文件，分为核心框架、服务实现、DDK 库、JS 绑定、测试等类别。

### 关键配置参数

| 参数 | 值 | 说明 |
|------|-----|------|
| subsystem_name | `hdf` | 所属子系统 |
| part_name | `external_device_manager` | 部件名称 |
| install_enable | `true` | 启用安装 |
| shlib_type | `sa` | 共享库类型（System Ability） |

## 主要 Targets 清单

### 服务层 Targets

| Target 名称 | 类型 | 源文件 | 输出产物 |
|------------|------|--------|----------|
| `driver_extension_manager` | ohos_shared_library | driver_ext_mgr.cpp, event_config.cpp, ext_permission_manager.cpp | libdriver_extension_manager.z.so |
| `driver_extension_manager_test` | ohos_static_library | 同上 | 静态库（测试用） |
| `driver_extension` | ohos_shared_library | js_driver_extension.cpp | libdriver_extension.z.so |
| `driver_extension_module` | ohos_shared_library | driver_extension_module_loader.cpp | libdriver_extension_module.z.so |

### 设备管理 Targets

| Target 名称 | 类型 | 源文件 | 输出产物 |
|------------|------|--------|----------|
| `driver_extension_device_manager` | ohos_shared_library | etx_device_mgr.cpp, device.cpp, driver_extension_controller.cpp | libdriver_extension_device_manager.z.so |
| `drivers_pkg_manager` | ohos_shared_library | driver_pkg_manager.cpp, pkg_database.cpp | libdrivers_pkg_manager.z.so |
| `driver_extension_bus_core` | ohos_shared_library | bus_extension_core.cpp | libdriver_extension_bus_core.z.so |
| `driver_extension_usb_bus` | ohos_shared_library | usb_bus_extension.cpp, usb_dev_subscriber.cpp | libdriver_extension_usb_bus.z.so |

### DDK Targets

| Target 名称 | 类型 | 源文件 | 输出产物 |
|------------|------|--------|----------|
| `ddk_base` | ohos_shared_library | ddk_api.cpp | libddk_base.z.so |
| `usb_ndk` | ohos_shared_library | usb_ddk_api.cpp, usb_config_desc_parser.cpp | libusb_ndk.z.so |
| `hid` | ohos_shared_library | input_emit_event.cpp | libhid.z.so |
| `scsi` | ohos_shared_library | scsi_ddk_api.cpp | libscsi.z.so |
| `usb_serial_ndk` | ohos_shared_library | usb_serial_ddk_api.cpp | libusb_serial_ndk.z.so |

### JS/NAPI Targets

| Target 名称 | 类型 | 源文件 | 输出产物 |
|------------|------|--------|----------|
| `devicemanager_napi` | ohos_shared_library | device_manager_middle.cpp | libdevicemanager_napi.z.so |
| `driverextensionability` | ohos_shared_library | driver_extension_ability_module.cpp | libdriverextensionability.z.so |
| `driverextensioncontext_napi` | ohos_shared_library | driver_extension_context_module.cpp | libdriverextensioncontext_napi.z.so |

## 核心构建配置

### 主服务构建配置

```gn
ohos_shared_library("driver_extension_manager") {
  install_enable = true
  sources = [
    "${ext_mgr_path}/services/native/driver_extension_manager/src/driver_ext_mgr_types.cpp",
    "native/driver_extension_manager/src/driver_ext_mgr.cpp",
    "native/driver_extension_manager/src/event_config.cpp",
    "native/driver_extension_manager/src/ext_permission_manager.cpp",
  ]
  
  include_dirs = [
    "${ext_mgr_path}/services/native/driver_extension_manager/include",
    "${ext_mgr_path}/interfaces/ddk/usb/",
    "${ext_mgr_path}/interfaces/innerkits/",
    # ... 更多 include 目录
  ]
  
  configs = [ "${utils_path}:utils_config" ]
  
  deps = [
    "${ext_mgr_path}/interfaces/innerkits:external_device_manager_stub",
    "${ext_mgr_path}/services/native/driver_extension_manager/src/bus_extension/core:driver_extension_bus_core",
    "${ext_mgr_path}/services/native/driver_extension_manager/src/device_manager:driver_extension_device_manager",
    "${ext_mgr_path}/services/native/driver_extension_manager/src/device_notification:notification_peripheral",
    "${ext_mgr_path}/services/native/driver_extension_manager/src/drivers_hisysevent:report_sys_event",
    "${ext_mgr_path}/services/native/driver_extension_manager/src/drivers_pkg_manager:drivers_pkg_manager",
    # ... 更多依赖
  ]
  
  external_deps = [
    "ability_runtime:ability_manager",
    "access_token:libaccesstoken_sdk",
    "bundle_framework:appexecfwk_base",
    "cJSON:cjson",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    # ... 更多外部依赖
  ]
  
  cflags_cc = [
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
    "-Os",
  ]
  
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    cfi_no_nvcall = true
    cfi_vcall_icall_only = true
    debug = false
  }
  
  shlib_type = "sa"
  subsystem_name = "hdf"
  part_name = "external_device_manager"
}
```

## 依赖关系

### 内部依赖

```
driver_extension_manager
├── driver_extension_manager (SA)
│   ├── device_manager (设备管理)
│   │   ├── device
│   │   ├── driver_extension_controller
│   │   └── bundle_update_callback
│   ├── drivers_pkg_manager (包管理)
│   │   ├── pkg_database
│   │   ├── pkg_db_helper
│   │   └── drv_bundle_state_callback
│   ├── bus_extension (总线扩展)
│   │   ├── core (总线核心)
│   │   │   └── bus_extension_core
│   │   └── usb (USB 总线)
│   │       ├── usb_bus_extension
│   │       ├── usb_dev_subscriber
│   │       └── usb_driver_change_callback
│   ├── device_notification (通知)
│   │   └── notification_peripheral
│   ├── drivers_hisysevent (系统事件)
│   │   └── report_sys_event
│   └── ext_permission_manager (权限)
└── external_device_manager_stub (IPC 存根)
```

### 外部依赖

| 外部组件 | 用途 |
|---------|------|
| ability_runtime | Ability 生命周期管理 |
| access_token | 权限验证 |
| bundle_framework | 包管理服务接口 |
| cJSON | JSON 解析 |
| hilog | 日志输出 |
| ipc | 进程间通信 |
| safwk | System Ability 框架 |
| samgr | 系统能力管理器 |
| distributed_notification_service | 分布式通知 |

## 编译产物

### 共享库产物

| 产物路径 | 安装路径 | 说明 |
|---------|---------|------|
| libdriver_extension_manager.z.so | system/lib | 主服务库 |
| libdriver_extension_device_manager.z.so | system/lib | 设备管理库 |
| libdrivers_pkg_manager.z.so | system/lib | 包管理库 |
| libdriver_extension_bus_core.z.so | system/lib | 总线核心库 |
| libdriver_extension_usb_bus.z.so | system/lib | USB 总线库 |
| libddk_base.z.so | system/lib/ndk/ | 基础 DDK |
| libusb_ndk.z.so | system/lib/ndk/ | USB DDK |
| libhid.z.so | system/lib/ndk/ | HID DDK |
| libscsi.z.so | system/lib/ndk/ | SCSI DDK |
| libusb_serial_ndk.z.so | system/lib/ndk/ | USB Serial DDK |
| libdevicemanager_napi.z.so | system/lib/module/driver/ | N-API 库 |

### 配置文件产物

| 产物路径 | 安装路径 | 说明 |
|---------|---------|------|
| 5110.json | system/profile/ | SA 配置文件 |
| hdf_ext_devmgr.cfg | system/etc/init/ | 初始化配置 |
| peripheral_fault_notifier_config.json | system/etc/peripheral/ | 故障通知配置 |
| string.json (多语言) | system/etc/peripheral/resources/ | 本地化字符串 |

### NDK 产物

| 产物路径 | 安装路径 | 说明 |
|---------|---------|------|
| libddk_base.z.so | system/lib/ndk/ | 共享库 |
| libddk_base.z.a | system/lib/ndk/ | 静态库 |
| libusb_ndk.z.so | system/lib/ndk/ | 共享库 |
| libusb_ndk.z.a | system/lib/ndk/ | 静态库 |
| libscsi.z.so | system/lib/ndk/ | 共享库 |
| libscsi.z.a | system/lib/ndk/ | 静态库 |
| libusb_serial_ndk.z.so | system/lib/ndk/ | 共享库 |
| libusb_serial_ndk.z.a | system/lib/ndk/ | 静态库 |

## 构建命令

### 编译 32 位 ARM 系统

```bash
./build.sh --product-name {product_name} --ccache --build-target external_device_manager
```

### 编译 64 位 ARM 系统

```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target external_device_manager
```

参数说明：

| 参数 | 说明 |
|------|------|
| --product-name | 产品名称（如 rk3568） |
| --ccache | 启用 ccache 加速编译 |
| --target-cpu | 目标 CPU 架构（arm 或 arm64） |
| --build-target | 构建目标 |

## 条件编译

### USB Pass-Through 模式

```gn
declare_args() {
  extdevmgr_usb_pass_through = true
}

if (extdevmgr_usb_pass_through) {
  defines += [ "EXTDEVMGR_USB_PASS_THROUGH" ]
}
```

### 32 位 Binder IPC

```gn
if (target_cpu == "arm") {
  cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

## 覆盖率支持

```gn
config("coverage_flags") {
  if (external_device_manager_coverage) {
    cflags = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}
```

启用覆盖率编译时，需要在构建参数中添加：

```bash
--gn-args external_device_manager_coverage=true
```

## bundle.json 配置

```json
{
  "name": "@ohos/external_device_manager",
  "version": "4.0",
  "component": {
    "name": "external_device_manager",
    "subsystem": "hdf",
    "syscap": [
      "SystemCapability.Driver.HID.Extension",
      "SystemCapability.Driver.ExternalDevice",
      "SystemCapability.Driver.USB.Extension",
      "SystemCapability.Driver.DDK.Extension",
      "SystemCapability.Driver.UsbSerial.Extension",
      "SystemCapability.Driver.SCSI.Extension"
    ],
    "adapted_system_type": ["standard"],
    "rom": "735KB",
    "ram": "8000KB",
    "deps": {
      "components": [
        "hilog", "init", "ipc", "samgr", "ability_base",
        "common_event_service", "c_utils", "os_account",
        "drivers_interface_usb", "bundle_framework",
        "ability_runtime", "hisysevent", "hitrace",
        "napi", "safwk", "eventhandler", "ace_engine",
        "access_token", "relational_store",
        "drivers_interface_input", "cJSON",
        "distributed_notification_service",
        "i18n", "image_framework", "runtime_core",
        "selinux_adapter"
      ]
    },
    "build": {
      "sub_component": [
        "//drivers/external_device_manager/frameworks:ext_devmgr_frameworks",
        "//drivers/external_device_manager/services/native/driver_extension:driver_extension_module",
        "//drivers/external_device_manager/sa_profile:ext_dev_mgr_sa",
        "//drivers/external_device_manager/services:driver_extension_manager",
        "//drivers/external_device_manager/services:driver_extension_manager_test",
        "//drivers/external_device_manager/services/native/driver_extension_manager/src/drivers_pkg_manager:drivers_pkg_manager",
        "//drivers/external_device_manager/services/native/driver_extension_manager/src/device_manager:driver_extension_device_manager",
        "//drivers/external_device_manager/services/native/driver_extension_manager/src/bus_extension/core:driver_extension_bus_core",
        "//drivers/external_device_manager/services/native/driver_extension_manager/src/bus_extension/usb:driver_extension_usb_bus",
        "//drivers/external_device_manager/services/native/driver_extension_manager/src/drivers_hisysevent:report_sys_event"
      ],
      "inner_kits": [
        {
          "name": "//drivers/external_device_manager/interfaces/innerkits:driver_ext_mgr_client",
          "header": {
            "header_files": ["driver_ext_mgr_client.h"],
            "header_base": "//drivers/external_device_manager/interfaces/innerkits"
          }
        }
      ]
    }
  }
}
```

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
