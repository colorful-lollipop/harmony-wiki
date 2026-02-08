# GN 构建配置

## 1. 构建系统概述

DeviceManager 使用 **GN (Generate Ninja)** 作为构建系统，输出 Ninja 构建文件。

### 1.1 构建入口

| 文件 | 说明 |
|-----|------|
| `BUILD.gn` | 根构建入口 |
| `device_manager.gni` | 项目全局配置 |

### 1.2 全局配置

**文件**：`device_manager.gni`

```gn
# 路径变量定义
devicemanager_path = "//foundation/distributedhardware/device_manager"
innerkits_path = "${devicemanager_path}/interfaces/inner_kits"
services_path = "${devicemanager_path}/services"
common_path = "${devicemanager_path}/common"
utils_path = "${devicemanager_path}/utils"
json_path = "${devicemanager_path}/json"
softbuscache_parh = "${devicemanager_path}/services/softbuscache"

# 功能开关
support_jsapi = true
device_manager_feature_product = "default"
device_manager_common = false
```

> 证据来源：`device_manager.gni`

## 2. 根 BUILD.gn

**文件**：`BUILD.gn`

### 2.1 轻量系统构建

```gn
if (defined(ohos_lite)) {
  if (ohos_kernel_type == "liteos_m") {
    lite_component("device_manager") {
      features = [
        "interfaces/inner_kits/native_cpp:devicemanagersdk"
      ]
    }
  } else {
    lite_component("device_manager") {
      features = [
        "utils:devicemanagerutils",
        "services/service:devicemanagerservice",
        "services/implementation:devicemanagerserviceimpl",
        "interfaces/inner_kits/native_cpp:devicemanagersdk",
        "test/smallunittest:lite_devicemanager_test",
        "services/softbuscache:dmdevicecache",
      ]
    }
  }
}
```

### 2.2 标准系统构建

```gn
group("device_manager") {
  deps = [
    "ext:ext_modules",
    "sa_profile:dm_sa_profile",
    "services/etc:ohos.para.dac",
    "services/implementation:devicemanagerserviceimpl",
    "services/service:devicemanagerservice",
    "services/softbuscache:dmdevicecache",
    "dbimpl/kvdbimpl/bykvstore:dmdb_kvstore",
  ]

  if (!is_lite_system && product_name != "qemu-arm-linux-min") {
    deps += [
      "commondependency:devicemanagerdependencytest",
      "radar:devicemanagerradartest",
      "services/service:devicemanagerservicetest",
      "utils:devicemanagerutilstest",
    ]
  }

  if (product_name != "qemu-arm-linux-min") {
    deps += [ "display/entry:DeviceManager_UI" ]
  }
}
```

### 2.3 框架构建

```gn
group("device_manager_fwk") {
  deps = [
    "interfaces/cj:cj_distributed_device_manager_ffi_group",
    "interfaces/inner_kits/native_cpp:devicemanagersdk",
    "interfaces/kits:devicemanager_native_js",
    "interfaces/mini_tools_kits/native_cpp:devicemanagerminisdk",
  ]
}
```

> 证据来源：`BUILD.gn:14-85`

## 3. Interfaces 模块

### 3.1 kits 模块

**文件**：`interfaces/kits/BUILD.gn`

```gn
group("devicemanager_native_js") {
  deps = []
  if (support_jsapi) {
    deps += [
      "./js:devicemanager",
      "./js4.0:distributeddevicemanager",
      "./ndk:devicemanager_ndk",
      "./taihe:devicemanager_ani",
    ]
  }
}
```

**输出产物**：
- `libdevicemanager.so`：传统 JS 接口
- `libdistributeddevicemanager.so`：4.0+ JS 接口
- `libdevicemanager_ndk.so`：NDK 接口
- `libdevicemanager_ani.so`：太和界面库

### 3.2 js4.0 模块

**文件**：`interfaces/kits/js4.0/BUILD.gn`

```gn
ohos_shared_library("distributeddevicemanager") {
  sources = [
    "src/native_devicemanager_js.cpp",
    # ... 其他源文件
  ]

  include_dirs = [
    "include",
    "${innerkits_path}/native_cpp/include",
    "${devicemanager_path}/services/implementation/include",
    # ... 其他头文件目录
  ]

  defines = [
    "OHOS_NAPI",
    "DM_NAPI_EXPORT",
  ]

  deps = [
    "${innerkits_path}/native_cpp:devicemanagersdk",
    "${utils_path}:devicemanagerutils",
  ]

  external_deps = [
    "hilog:libhilog",
    "ipc:ipc_core",
  ]
}
```

### 3.3 inner_kits 模块

**文件**：`interfaces/inner_kits/native_cpp/BUILD.gn`

```gn
ohos_shared_library("devicemanagersdk") {
  sources = [
    "src/ipc/standard/ipc_client.cpp",
    "src/ipc/standard/ipc_client_skeleton.cpp",
    # ... 其他源文件
  ]

  include_dirs = [
    "include",
    "${services_path}/implantation/include",
    "${common_path}/include",
    "${utils_path}/include",
  ]

  defines = [
    "OHOS_SDK",
    "DM_SDK_EXPORT",
  ]

  deps = [
    "${utils_path}:devicemanagerutils",
    "${services_path}/implantation:devicemanagerserviceimpl",
  ]
}
```

## 4. Services 模块

### 4.1 implementation 模块

**文件**：`services/implementation/BUILD.gn`

#### 轻量系统版本

```gn
shared_library("devicemanagerserviceimpl") {
  include_dirs = [
    "include",
    "include/ability",
    "include/adapter",
    "include/credential",
    "include/dependency/hichain",
    "include/dependency/softbus",
    "include/devicestate",
    # ... 其他头文件
  ]

  sources = [
    "src/ability/lite/dm_ability_manager.cpp",
    "src/credential/dm_credential_manager.cpp",
    "src/dependency/hichain/hichain_auth_connector.cpp",
    "src/dependency/hichain/hichain_connector.cpp",
    "src/dependency/softbus/softbus_connector.cpp",
    "src/device_manager_service_impl_lite.cpp",
    "src/devicestate/dm_device_state_manager.cpp",
  ]

  defines = [
    "LITE_DEVICE",
    "DH_LOG_ENABLE",
    "DH_LOG_TAG=\"devicemanagerserviceimpl\"",
    "LOG_DOMAIN=0xD004110",
  ]

  deps = [
    "${devicemanager_path}/radar:devicemanagerradar",
    "${json_path}:devicemanagerjson",
    "${softbuscache_parh}:dmdevicecache",
    "${utils_path}:devicemanagerutils",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//base/security/device_auth/services:deviceauth_sdk",
    "//foundation/communication/dsoftbus:dsoftbus",
  ]
}
```

#### 标准系统版本

```gn
ohos_shared_library("devicemanagerserviceimpl") {
  branch_protector_ret = "pac_ret"

  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
    integer_overflow = true
    ubsan = true
  }

  cflags = [
    "-Werror",
    "-fPIC",
    "-fstack-protector-strong",
  ]

  ldflags = [
    "-Wl,-z,relro",
    "-Wl,-z,now",
  ]

  sources = [
    "src/authentication/dm_auth_manager.cpp",
    "src/credential/dm_credential_manager.cpp",
    "src/dependency/hichain/hichain_connector.cpp",
    "src/dependency/softbus/softbus_connector.cpp",
    "src/devicestate/dm_device_state_manager.cpp",
    # ... 其他源文件（共 25+ 文件）
  ]

  defines = [
    "HI_LOG_ENABLE",
    "DH_LOG_TAG=\"devicemanagerserviceimpl\"",
    "LOG_DOMAIN=0xD004110",
  ]

  deps = [
    "${devicemanager_path}/commondependency:devicemanagerdependency",
    "${devicemanager_path}/radar:devicemanagerradar",
    "${innerkits_path}/native_cpp:devicemanagersdk",
    "${json_path}:devicemanagerjson",
    "${softbuscache_parh}:dmdevicecache",
    "${utils_path}:devicemanagerutils",
  ]

  external_deps = [
    "ability_base:session_info",
    "ability_base:want",
    "device_auth:deviceauth_sdk",
    "dsoftbus:softbus_client",
    "dsoftbus:softbus_utils",
    "ipc:ipc_core",
    # ... 其他外部依赖
  ]
}
```

> 证据来源：`services/implementation/BUILD.gn:14-276`

## 5. Utils 模块

**文件**：`utils/BUILD.gn`

```gn
ohos_shared_library("devicemanagerutils") {
  sources = [
    "src/log/dm_log.cpp",
    "src/cipher/dm_cipher.cpp",
    "src/ipc/standard/dm_ipc_client.cpp",
    "src/ipc/standard/dm_ipc_server.cpp",
  ]

  include_dirs = [
    "include",
    "include/log",
    "include/cipher",
    "include/ipc/standard",
  ]

  defines = [
    "OHOS_UTILS",
    "DH_LOG_TAG=\"device_manager_utils\"",
  ]

  external_deps = [
    "hilog:libhilog",
    "ipc:ipc_core",
    "utils:utils",
  ]
}
```

## 6. SA 配置模块

**文件**：`sa_profile/BUILD.gn`

```gn
ohos_sa_profile("dm_sa_profile") {
  srcs = [
    "device_manager.cfg",
  ]
  subsystem_name = "distributedhardware"
  part_name = "device_manager"
}
```

> 证据来源：`sa_profile/BUILD.gn`

## 7. 编译产物模块

### 7.1 产物清单

| 模块 | 产物类型 | 产物名称 | 路径 |
|-----|---------|---------|------|
| interfaces/kits/js4.0 | .so | libdistributeddevicemanager.so | out/... |
| interfaces/inner_kits | .so | libdevicemanagersdk.so | out/... |
| services/implementation | .so | libdevicemanagerserviceimpl.so | out/... |
| services/service | .so | libdevicemanagerservice.so | out/... |
| utils | .so | libdevicemanagerutils.so | out/... |
| display/entry | .hap | DeviceManager_UI.hap | out/... |
| sa_profile | .cfg | device_manager.cfg | system/etc/ |

### 7.2 产物依赖关系

```
libdevicemanagerservice.so
    │
    ├── libdevicemanagerserviceimpl.so
    │       │
    │       ├── libdevicemanagersdk.so
    │       │       │
    │       │       └── libdevicemanagerutils.so
    │       │
    │       └── libdmdevicecache.so
    │
    └── libdeviceauth.so (外部依赖)
            │
            └── libdsoftbus.so (外部依赖)
```

## 8. 构建开关

### 8.1 功能开关

| 开关 | 默认值 | 说明 |
|-----|-------|------|
| `support_jsapi` | true | 支持 JS API |
| `device_manager_feature_product` | "default" | 功能产品配置 |
| `device_manager_common` | false | 通用功能开关 |
| `support_screenlock` | true | 锁屏支持（默认产品） |
| `support_msdp` | false | MSDP 空间感知支持 |

### 8.2 安全加固

```gn
sanitize = {
  boundary_sanitize = true    // 边界检查
  cfi = true                   // 控制流完整性
  cfi_cross_dso = true         // 跨 DSO CFI
  integer_overflow = true      // 整数溢出检查
  ubsan = true                 // 未定义行为检查
}

ldflags = [
  "-Wl,-z,relro",   // 只读重定位
  "-Wl,-z,now",     // 立即绑定
]
```

> 证据来源：`services/implementation/BUILD.gn:153-171`

## 9. 构建命令

### 9.1 全量构建

```bash
./build.sh --product-name <product> --ccache
```

### 9.2 单独构建

```bash
# 构建 DeviceManager
hb build -p distributedhardware/device_manager

# 构建 DeviceManager UI HAP
hb build -p distributedhardware/device_manager --build-target display/entry:DeviceManager_UI
```

### 9.3 单独模块构建

```bash
# 构建 JS SDK
ninja -C out/<product>/packages/phone distributeddevicemanager

# 构建 Service
ninja -C out/<product>/packages/phone libdevicemanagerserviceimpl.so
```
