# GN构建与编译产物

## 概述

本文档详细说明后台任务管理模块的GN构建配置、编译产物和安装路径。

**根构建文件**: `BUILD.gn`  
**配置定义文件**: `bgtaskmgr.gni`

---

## 顶层Target组

### 根BUILD.gn定义

```gn
# 框架组 - 包含所有对外接口库
group("fwk_group_background_task_mgr_all") {
  if (background_task_mgr_device_enable) {
    deps = [ "${bgtaskmgr_interfaces_path}:bgtaskmgr_interfaces" ]
  }
}

# 服务组 - 包含服务、资源、SA配置
group("service_group_background_task_mgr_all") {
  if (background_task_mgr_device_enable) {
    deps = [
      "${bgtaskmgr_root_path}/resources:bgtaskmgr_resources",
      "${bgtaskmgr_root_path}/sa_profile:bgtaskmgr_sa_profile",
      "${bgtaskmgr_root_path}/services:bgtaskmgr_service",
    ]
  }
}

# 测试组 - 包含所有测试
group("test_background_task_mgr_all") {
  testonly = true
  if (background_task_mgr_device_enable) {
    deps = [ /* 所有测试target */ ]
  }
}
```

---

## Feature Flags

**文件**: `bgtaskmgr.gni`

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `background_task_mgr_graphics` | bool | true | 启用图形支持 |
| `background_task_mgr_jsstack` | bool | true | 启用JS栈支持 |
| `background_task_mgr_device_enable` | bool | true | 设备使能开关 |
| `has_os_account_part` | bool | auto | OS账号组件可用性 |
| `distributed_notification_enable` | bool | auto | 分布式通知服务可用性 |

---

## 服务层Targets

### services/BUILD.gn

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `bgtaskmgr_service` | ohos_shared_library | `libbgtaskmgr_service.z.so` | 主服务动态库 |
| `bgtaskmgr_service_static` | ohos_static_library | `.a`文件 | 静态库（用于测试） |

**bgtaskmgr_service配置**:

```gn
ohos_shared_library("bgtaskmgr_service") {
  source_set = [ "bgtaskmgr_service_sources" ]
  
  # SA类型，安装到/system/lib/
  shlib_type = "sa"
  
  # 安全加固
  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true
    cfi_cross_dso = true
  }
  cflags_cc = [
    "-fstack-protector-strong",
    "-fvisibility=hidden"
  ]
  
  # 公共依赖
  public_deps = [
    "//foundation/ability/ability_runtime:ability_manager",
    "//foundation/ability/ability_runtime:app_manager",
  ]
  
  # 外部依赖
  external_deps = [
    "ability_base:want",
    "ability_runtime:abilitykit_native",
    "access_token:libaccesstoken_sdk",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "eventhandler:libeventhandler",
    "hicollie:libhicollie",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "jsoncpp:jsoncpp",
    "relational_store:native_rdb",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
  ]
  
  # 定义宏
  defines = [
    "HAS_OS_ACCOUNT_PART",
    "DISTRIBUTED_NOTIFICATION_ENABLE",
  ]
}
```

**源文件列表** (29个):

| 目录 | 文件数 | 主要文件 |
|------|--------|----------|
| `common/src/` | 9 | app_mgr_helper.cpp, bundle_manager_helper.cpp, bgtask_config.cpp等 |
| `continuous_task/src/` | 5 | bg_continuous_task_mgr.cpp, continuous_task_record.cpp等 |
| `core/src/` | 1 | background_task_mgr_service.cpp |
| `efficiency_resources/src/` | 3 | bg_efficiency_resources_mgr.cpp等 |
| `transient_task/src/` | 11 | bg_transient_task_mgr.cpp, timer_manager.cpp, watchdog.cpp等 |

---

## 接口层Targets

### interfaces/innerkits/BUILD.gn

| Target | 类型 | 说明 |
|--------|------|------|
| `background_task_mgr_interface` | idl_gen_interface | IDL代码生成 |
| `bgtaskmgr_innerkits` | ohos_shared_library | 内部API动态库 |
| `background_task_mgr_stub` | ohos_source_set | 服务端Stub |
| `background_task_mgr_proxy` | ohos_source_set | 客户端Proxy |
| `background_task_subscriber_interface` | idl_gen_interface | 订阅者IDL |
| `background_task_subscriber_stub` | ohos_source_set | 订阅者Stub |
| `background_task_subscriber_proxy` | ohos_source_set | 订阅者Proxy |
| `expired_callback_interface` | idl_gen_interface | 回调IDL |
| `expired_callback_stub` | ohos_source_set | 回调Stub |
| `expired_callback_proxy` | ohos_source_set | 回调Proxy |

**bgtaskmgr_innerkits输出**: `libbgtaskmgr_innerkits.z.so`

### interfaces/kits/BUILD.gn

| Target | 类型 | 安装路径 | 说明 |
|--------|------|----------|------|
| `backgroundtaskmanager` | ohos_shared_library | `module/` | 旧版NAPI模块 |
| `backgroundtaskmanager_napi` | ohos_shared_library | `module/resourceschedule/` | 主NAPI模块 |
| `cj_background_task_mgr_ffi` | ohos_shared_library | - | Cangjie FFI |
| `transient_task` | ohos_shared_library | `ndk/` | C API (NDK) |

**backgroundtaskmanager_napi配置**:
```gn
ohos_shared_library("backgroundtaskmanager_napi") {
  sources = [
    "napi/src/init_bgtaskmgr.cpp",
    "napi/src/bg_continuous_task_napi_module.cpp",
    "napi/src/request_suspend_delay.cpp",
    "napi/src/cancel_suspend_delay.cpp",
    "napi/src/get_remaining_delay_time.cpp",
    "napi/src/efficiency_resources_operation.cpp",
    "napi/src/common.cpp",
    # ... 其他NAPI源文件
  ]
  
  install_images = [ "system" ]
  relative_install_dir = "module/resourceschedule"
  part_name = "background_task_mgr"
  subsystem_name = "resourceschedule"
}
```

### interfaces/kits/ets/taihe/BUILD.gn (ArkTS 1.2)

| Target | 类型 | 输出 |
|--------|------|------|
| `copy_backgroundTaskManager` | copy_taihe_idl | IDL文件复制 |
| `run_taihe` | ohos_taihe | ANI代码生成 |
| `background_task_manager_ani` | taihe_shared_library | ANI运行时库 |
| `background_task_manager_abc` | generate_static_abc | `background_task_manager_abc.abc` |
| `background_task_manager_etc` | ohos_prebuilt_etc | 配置文件 |

**安装路径**: `/system/framework/background_task_manager_abc.abc`

---

## 资源Targets

### resources/BUILD.gn

| Target | 类型 | 输出路径 |
|--------|------|----------|
| `backgroundtask_res` | ohos_resources | - |
| `backgroundtaskresources_hap` | ohos_hap | `app/com.ohos.backgroundtaskmgr.resources/` |

**HAP配置**:
```gn
ohos_hap("backgroundtaskresources_hap") {
  deps = [ ":backgroundtask_res" ]
  hap_profile = "./main/config.json"
  hap_name = "BackgroundTaskResources"
  module_install_dir = "app/com.ohos.backgroundtaskmgr.resources"
  certificate_profile = "./BackgroundTaskResources.p7b"
  subsystem_name = "resourceschedule"
  part_name = "background_task_mgr"
}
```

**多语言资源** (70+语言):
- `resources/main/resources/ar/` - 阿拉伯语
- `resources/main/resources/de/` - 德语
- `resources/main/resources/en_GB/` - 英式英语
- `resources/main/resources/zh_CN/` - 简体中文
- `resources/main/resources/zh_TW/` - 繁体中文
- ... (更多语言)

---

## SA配置Target

### sa_profile/BUILD.gn

| Target | 类型 | 源文件 |
|--------|------|--------|
| `bgtaskmgr_sa_profile` | ohos_sa_profile | `1903.json` |

**SA配置** (`1903.json`):
```json
{
    "process": "resource_schedule_service",
    "systemability": [
        {
            "name": 1903,
            "libpath": "libbgtaskmgr_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "extension": ["backup", "restore"]
        }
    ]
}
```

---

## 编译产物清单

### 动态库 (.so)

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libbgtaskmgr_service.z.so` | `/system/lib/` | 主服务库 (SA类型) |
| `libbgtaskmgr_innerkits.z.so` | `/system/lib/` | 内部API库 |
| `libbackgroundtaskmanager.z.so` | `/system/lib/module/` | 旧版NAPI |
| `libbackgroundtaskmanager_napi.z.so` | `/system/lib/module/resourceschedule/` | 主NAPI模块 |
| `libcj_background_task_mgr_ffi.z.so` | `/system/lib/` | Cangjie FFI |
| `libtransient_task.z.so` | `/system/lib/ndk/` | C API (NDK) |

### HAP包

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `BackgroundTaskResources.hap` | `/system/app/com.ohos.backgroundtaskmgr.resources/` | 资源HAP |

### ABC文件

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `background_task_manager_abc.abc` | `/system/framework/` | ArkTS 1.2字节码 |

### SA配置文件

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `1903.json` | `/system/profile/` | SA ID 1903配置 |

---

## 头文件安装

**inner_kits头文件** (bundle.json定义):

```json
"inner_kits": [
  {
    "header": {
      "header_base": "//foundation/resourceschedule/background_task_mgr/interfaces/innerkits/include",
      "header_files": [
        "background_mode.h",
        "background_task_mgr_helper.h",
        "background_task_subscriber.h",
        "continuous_task_callback_info.h",
        "continuous_task_param.h",
        "delay_suspend_info.h",
        "efficiency_resource_info.h",
        "expired_callback.h",
        "resource_callback_info.h",
        "resource_type.h",
        "transient_task_app_info.h"
      ]
    },
    "name": "//foundation/resourceschedule/background_task_mgr/interfaces/innerkits:bgtaskmgr_innerkits"
  }
]
```

**安装路径**: 编译时自动安装到SDK头文件目录

---

## 依赖关系图

```
fwk_group_background_task_mgr_all
    ↓
bgtaskmgr_interfaces (group)
    ├── bgtaskmgr_innerkits (shared_library)
    │   ├── background_task_mgr_stub (source_set)
    │   ├── background_task_mgr_proxy (source_set)
    │   ├── background_task_subscriber_stub (source_set)
    │   ├── background_task_subscriber_proxy (source_set)
    │   └── expired_callback_* (source_set)
    ├── backgroundtaskmanager (shared_library)
    │   └── depends: bgtaskmgr_innerkits
    ├── backgroundtaskmanager_napi (shared_library)
    │   └── depends: bgtaskmgr_innerkits, background_task_mgr_proxy
    ├── cj_background_task_mgr_ffi (shared_library)
    └── background_task_manager_taihe (group)
        ├── background_task_manager_ani (taihe_shared_library)
        └── background_task_manager_etc (prebuilt_etc)

service_group_background_task_mgr_all
    ├── bgtaskmgr_resources (group)
    │   └── backgroundtaskresources_hap (ohos_hap)
    ├── bgtaskmgr_sa_profile (ohos_sa_profile)
    └── bgtaskmgr_service (ohos_shared_library)
        ├── depends: bgtaskmgr_innerkits
        ├── depends: background_task_mgr_stub
        └── external_deps: [ability_runtime, access_token, ...]
```

---

## 构建命令示例

```bash
# 构建整个模块
gn gen out --args='...'
ninja -C out background_task_mgr:fwk_group_background_task_mgr_all
ninja -C out background_task_mgr:service_group_background_task_mgr_all

# 构建特定target
ninja -C out background_task_mgr/services:bgtaskmgr_service
ninja -C out background_task_mgr/interfaces/innerkits:bgtaskmgr_innerkits
ninja -C out background_task_mgr/interfaces/kits:backgroundtaskmanager_napi

# 构建测试
ninja -C out background_task_mgr:test_background_task_mgr_all
```

---

## Platform Defines

构建时自动定义的宏:

| 宏 | 说明 |
|-----|------|
| `HAS_OS_ACCOUNT_PART` | OS账号组件可用 |
| `DISTRIBUTED_NOTIFICATION_ENABLE` | 分布式通知服务可用 |
| `SUPPORT_GRAPHICS` | 图形支持 |
| `SUPPORT_AUTH` | 认证支持 |
| `FEATURE_PRODUCT_PHONE` | 手机产品 |
| `FEATURE_PRODUCT_WATCH` | 手表产品 |
| `FEATURE_PRODUCT_PC` | PC产品 |
| `FEATURE_PRODUCT_TABLET` | 平板产品 |

---

## 相关文档

- [目录结构](01_Directory_Structure.md) - 源码组织
- [内部API](04_Inner_API.md) - 接口说明
- [架构说明](02_Architecture.md) - 组件依赖
