# 04_GN_Build - GN 构建配置

## 概述

本文档描述 ecological_rule_manager 模块的 GN 构建配置、Targets 列表及编译产物。

## 构建入口

### 根构建文件

**文件**: `BUILD.gn` (根目录)

```gn
group("ecological_rule_mgr_packages") {
  if (is_standard_system) {
    deps = [
      "interfaces/innerkits:erms_client",
      "profile:ecologicalrulemgrservice_sa_profiles",
      "services:ecologicalrulemgr_service",
    ]
  }
}
```

**说明**: 仅在 Standard System 构建时包含所有模块。

## GN Targets 详解

### 1. erms_client

**类型**: `ohos_shared_library` (动态库)

**位置**: `interfaces/innerkits/BUILD.gn`

#### Sources

| 文件 | 职责 |
|------|------|
| `ecological_rule_mgr_service_client.cpp` | Client 单例、SA 连接管理 |
| `ecological_rule_mgr_service_param.cpp` | Parcelable 结构序列化 |
| `ecological_rule_mgr_service_proxy.cpp` | IPC Proxy 实现 |

#### Config

```gn
config("ecologicalrulemgrservice_client_config") {
  include_dirs = [
    "include",
    "${ecologicalrulemgrservice_path}/manager/include",
    "${ecologicalrulemgrservice_utils_path}/include",
  ]
}
```

#### Dependencies

```gn
external_deps = [
  "ability_base:want",
  "bundle_framework:appexecfwk_base",
  "c_utils:utils",
  "eventhandler:libeventhandler",
  "hilog:libhilog",
  "ipc:ipc_core",
  "samgr:samgr_proxy",
]
```

#### Metadata

```gn
innerapi_tags = [ "platformsdk" ]
subsystem_name = "bundlemanager"
part_name = "ecological_rule_manager"
```

**产物**: `liberms_client.z.so`

### 2. ecologicalrulemgr_service

**类型**: `ohos_shared_library` (动态库 - SA 实现)

**位置**: `services/BUILD.gn`

#### Sources

| 文件 | 职责 |
|------|------|
| `ecological_rule_mgr_service_param.cpp` | 参数序列化 |
| `ecologic_rule_mgr_service.cpp` | SA 主实现（OnStart/OnStop） |
| `ecologic_rule_mgr_service_stub.cpp` | IPC Stub 实现 |

#### Config

```gn
config("ecologicalrulemgrservice_config") {
  include_dirs = [
    "include",
    "./manager/include",
    "../utils/include",
    "${innerkits_path}/include",
  ]
}
```

#### Dependencies

```gn
external_deps = [
  "ability_base:want",
  "ability_base:zuri",
  "ability_runtime:abilitykit_native",
  "ability_runtime:runtime",
  "ability_runtime:wantagent_innerkits",
  "access_token:libaccesstoken_sdk",
  "access_token:libtokenid_sdk",
  "bundle_framework:appexecfwk_base",
  "bundle_framework:appexecfwk_core",
  "c_utils:utils",
  "hilog:libhilog",
  "ipc:ipc_core",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
]
```

#### Compiler Flags

```gn
cflags = [
  "-fvisibility=hidden",
  "-fdata-sections",
  "-ffunction-sections",
  "-Os",
]
cflags_cc = [
  "-fvisibility-inlines-hidden",
  "-Os",
]
```

**产物**: `libecologicalrulemgr_service.z.so`

### 3. ecologicalrulemgrservice_sa_profiles

**类型**: `ohos_sa_profile`

**位置**: `profile/BUILD.gn`

**Sources**: `6105.json`

**产物**: SA Profile 配置

### 4. ecologicalrulemgrservice_utils

**类型**: `ohos_source_set`

**位置**: `utils/BUILD.gn`

**依赖**: `hilog:libhilog`

## 产物清单

| 产物 | 类型 | 描述 |
|------|------|------|
| `liberms_client.z.so` | 动态库 | Inner Kit Client SDK |
| `libecologicalrulemgr_service.z.so` | 动态库 | System Ability 实现 |

## 产物路径

### 构建输出路径

```
out/{product}/libs/
├── liberms_client.z.so
└── libecologicalrulemgr_service.z.so
```

### 安装路径

| 产物 | 安装路径 |
|------|----------|
| SA 库 | `/system/lib64/platformsdk/libecologicalrulemgr_service.z.so` |
| Client 库 | `/system/lib64/platformsdk/liberms_client.z.so` |

**参考**: `profile/6105.json` 中 `libpath: "libecologicalrulemgr_service.z.so"`

## bundle.json 配置

```json
{
  "component": {
    "name": "ecological_rule_manager",
    "subsystem": "bundlemanager",
    "syscap": ["SystemCapability.BundleManager.EcologicalRuleManager"],
    "inner_kits": [
      {
        "name": "//foundation/bundlemanager/ecological_rule_manager/services:ecologicalrulemgr_service",
        "header": {
          "header_files": ["ecological_rule_mgr_service_interface.h"],
          "header_base": "//foundation/bundlemanager/ecological_rule_manager/interfaces/innerkits/include"
        }
      },
      {
        "name": "//foundation/bundlemanager/ecological_rule_manager/interfaces/innerkits:erms_client",
        "header": {
          "header_files": [
            "ecological_rule_mgr_service_interface.h",
            "ecological_rule_mgr_service_param.h"
          ],
          "header_base": "//foundation/bundlemanager/ecological_rule_manager/interfaces/innerkits/include"
        }
      }
    ]
  }
}
```

## 编译命令

```bash
# 单独编译 ecological_rule_manager
./build.sh --product-name rk3568 --ccache --build-target ecological_rule_manager

# 参数说明
# --product-name: 产品名称 (如 rk3568, Hi3516DV300)
# --ccache: 使用编译缓存
# --build-target: 编译目标 (ecological_rule_manager)
```

## 相关文档

- 目录结构: [01_Directory_Structure.md](01_Directory_Structure.md)
- 接口: [03_Inner_API.md](03_Inner_API.md)
