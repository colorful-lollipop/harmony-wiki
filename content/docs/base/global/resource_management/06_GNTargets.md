# GN Targets 文档

## 目的

本文档详细说明 OpenHarmony 资源管理组件的 GN 构建系统，包括所有 targets、类型、依赖、产物和配置选项。

## 适用范围

本文档覆盖所有非测试的 GN targets，包括主库、接口绑定和平台适配。

## 关键结论

| 类别 | Target 数量 | 主要产物 |
|------|------------|----------|
| 核心库 | 6 个 | libglobal_resmgr.so, librawfile.so, libohresmgr.so |
| JS 绑定 | 6 个 | libresourcemanager.so, libresmgr_napi_core.so |
| ETS 绑定 | 9 个 | ANI 库, .abc 文件 |
| CJ 绑定 | 1 个 | FFI 库 |
| 平台适配 | 4 个 | win_resmgr, mac_resmgr, linux_resmgr |

## 构建配置文件

### BUILD.gn 文件列表

| 文件 | 路径 | 职责 |
|------|------|------|
| **核心库** | `frameworks/resmgr/BUILD.gn` | 核心资源管理库 |
| **JS 绑定** | `interfaces/js/kits/BUILD.gn` | JS NAPI 接口 |
| **JS 核心** | `interfaces/js/innerkits/core/BUILD.gn` | NAPI 核心实现 |
| **E TS 绑定** | `interfaces/ets/ani/BUILD.gn` | ETS ANI 包组 |
| **E TS 实现** | `interfaces/ets/ani/resourceManager/BUILD.gn` | ETS ANI 实现 |
| **CJ 绑定** | `interfaces/cj/BUILD.gn` | Cangjie FFI 接口 |

### GNI 配置文件

| 文件 | 路径 | 职责 |
|------|------|------|
| **主配置** | `resmgr.gni` | 通用构建变量和特征标志 |

**证据**: Phase 1 全局扫描结果

## 核心库 Targets

### global_resmgr

**文件**: `frameworks/resmgr/BUILD.gn`

**类型**: `ohos_shared_library`

**输出名称**: `libglobal_resmgr.so`

**源文件** (21 个):
```cpp
hap_manager.cpp
hap_parser.cpp
hap_parser_v1.cpp
hap_parser_v2.cpp
hap_resource.cpp
hap_resource_manager.cpp
hap_resource_v1.cpp
hap_resource_v2.cpp
likely_subtags_key_data.cpp
likely_subtags_value_data.cpp
locale_matcher.cpp
mmap_file.cpp
native_resource_manager.cpp
raw_file_manager.cpp
res_config_impl.cpp
res_desc.cpp
res_locale.cpp
resource_manager.cpp
resource_manager_impl.cpp
system_resource_manager.cpp
theme_pack_config.cpp
theme_pack_manager.cpp
theme_pack_resource.cpp
```

**依赖**:
- `:resmgr_abc`
- 外部依赖：hilog, hisysevent, hitrace, zlib, cJSON, icu (可选) 等

**包含目录**:
- `include` (本地)
- `../../interfaces/inner_api/include`
- `../../interfaces/native/resource/include`
- `../../interfaces/js/innerkits/core/include`
- `../../dfx/hisysevent_adapter`

**证据**: `frameworks/resmgr/BUILD.gn`

### librawfile

**类型**: `ohos_shared_library`

**输出名称**: `librawfile.so`

**源文件**:
- `raw_file_manager.cpp`

**依赖**:
- `:global_resmgr`

**证据**: `frameworks/resmgr/BUILD.gn`

### ohresmgr

**类型**: `ohos_shared_library`

**输出名称**: `libohresmgr.so`

**源文件**:
- `native_resource_manager.cpp`

**依赖**:
- `:global_resmgr`

**证据**: `frameworks/resmgr/BUILD.gn`

### resmgr_abc

**类型**: `ohos_abc`

**输出名称**: `resmgr.abc`

**源文件**:
- `resmgr.js`

**依赖**: 无

**证据**: `frameworks/resmgr/BUILD.gn`

### 平台适配 Targets

| Target | 类型 | 条件 | 输出 |
|--------|------|------|------|
| `global_resmgr_win` | `ohos_shared_library` | `is_mingw == true` | libglobal_resmgr.so (Windows) |
| `global_resmgr_mac` | `ohos_shared_library` | `is_mac == true` | libglobal_resmgr.so (macOS) |
| `global_resmgr_linux` | `ohos_shared_library` | `is_linux == true` | libglobal_resmgr.so (Linux) |
| `win_resmgr` | `group` | `is_mingw == true` | global_resmgr_win |
| `mac_resmgr` | `group` | `is_mac == true` | global_resmgr_mac |
| `linux_resmgr` | `group` | `is_linux == true` | global_resmgr_linux |

**证据**: `resmgr.gni:15-23`, `frameworks/resmgr/BUILD.gn`

## JS 绑定 Targets

### resourcemanager

**文件**: `interfaces/js/kits/BUILD.gn`

**类型**: `ohos_shared_library`

**输出名称**: `libresourcemanager.so`

**源文件**:
- `resource_manager_napi.cpp`
- `hisysevent_adapter.cpp`

**依赖**:
- `../innerkits/core:resmgr_napi_core`

**证据**: `interfaces/js/kits/BUILD.gn`

### resourcemanager_preview

**类型**: `ohos_shared_library`

**输出名称**: `resourcemanager`

**源文件**:
- `resource_manager_napi.cpp`

**依赖**:
- `../innerkits/core:resmgr_napi_core_preview`

**证据**: `interfaces/js/kits/BUILD.gn`

### sendableresourcemanager

**类型**: `ohos_shared_library`

**输出名称**: `libsendableresourcemanager.so`

**源文件**:
- `sendable_resource_manager_napi.cpp`

**依赖**: 无

**证据**: `interfaces/js/kits/BUILD.gn`

### sendableresourcemanager_preview

**类型**: `ohos_shared_library`

**输出名称**: `sendableresourcemanager`

**源文件**:
- `sendable_resource_manager_napi.cpp`

**依赖**: 无

**证据**: `interfaces/js/kits/BUILD.gn`

## JS 核心 Targets

### resmgr_napi_core

**文件**: `interfaces/js/innerkits/core/BUILD.gn`

**类型**: `ohos_shared_library`

**输出名称**: `libresmgr_napi_core.so`

**源文件** (6 个):
```cpp
resource_manager_addon.cpp
resource_manager_napi_async_impl.cpp
resource_manager_napi_context.cpp
resource_manager_napi_sync_impl.cpp
resource_manager_napi_utils.cpp
resource_table_loader.cpp
```

**依赖**:
- `../../../../frameworks/resmgr:global_resmgr`
- 外部依赖：hilog, hisysevent, hitrace, napi, cJSON

**证据**: `interfaces/js/innerkits/core/BUILD.gn`

### resmgr_napi_core_preview

**类型**: `group`

**依赖**:
- `:resmgr_napi_core_preview_inner`

**证据**: `interfaces/js/innerkits/core/BUILD.gn`

## ETS/ANI Targets

### ani_package_resmgr

**文件**: `interfaces/ets/ani/BUILD.gn`

**类型**: `group`

**依赖**:
- `./resourceManager:rawFileDescriptor_etc`
- `./resourceManager:resmgr_ani`
- `./resourceManager:resourceManager_ani`
- `./resourceManager:resourceManager_etc`
- `./resourceManager:resource_etc`

**证据**: `interfaces/ets/ani/BUILD.gn`

### resourceManager_ani

**文件**: `interfaces/ets/ani/resourceManager/BUILD.gn`

**类型**: `ohos_shared_library`

**源文件**:
- `resourceManager.cpp`

**依赖**:
- `../../../../frameworks/resmgr:global_resmgr`
- `global_resmgr` (外部)

**证据**: `interfaces/ets/ani/resourceManager/BUILD.gn`

### resmgr_ani

**类型**: `ohos_shared_library`

**源文件**:
- `ani_utils.cpp`
- `resmgr_ani.cpp`

**依赖**:
- `:resourceManager_ani`
- `global_resmgr`
- `../../../js/innerkits/core:resmgr_napi_core`

**证据**: `interfaces/ets/ani/resourceManager/BUILD.gn`

### ABC 生成 Targets

| Target | 类型 | 输出 | 源文件 |
|--------|------|------|--------|
| `resourceManager` | `generate_static_abc` | `system/framework/resourceManager.abc` | `@ohos.resourceManager.ets` |
| `resource` | `generate_static_abc` | `system/framework/resource.abc` | `global/resource.ets`, `global/resourceInner.ets` |
| `rawFileDescriptor` | `generate_static_abc` | `system/framework/rawFileDescriptor.abc` | `global/rawFileDescriptor.ets`, `global/rawFileDescriptorInner.ets` |
| `resourceManager_etc` | `ohos_prebuilt_etc` | - | `:resourceManager` |
| `resource_etc` | `ohos_prebuilt_etc` | - | `:resource` |
| `rawFileDescriptor_etc` | `ohos_prebuilt_etc` | - | `:rawFileDescriptor` |

**证据**: `interfaces/ets/ani/resourceManager/BUILD.gn`

## Cangjie/FFI Targets

### cj_resource_manager_ffi

**文件**: `interfaces/cj/BUILD.gn`

**类型**: `ohos_shared_library`

**源文件** (SDK 构建):
- `resource_manager_ffi.cpp`
- `resource_manager_impl.cpp`
- `utils.cpp`

**源文件** (非 SDK 构建):
- `resource_manager_mock.cpp` (替代 resource_manager_impl.cpp)

**依赖**:
- `../../frameworks/resmgr:global_resmgr`

**证据**: `interfaces/cj/BUILD.gn`

## Target 依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ani_package_resmgr (group)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │resmgr_ani   │  │resourceMgr_ │  │resource_etc │  │rawFileDescriptor_etc│ │
│  │  (shlib)    │  │  ani (shlib)│  │  (prebuilt) │  │    (prebuilt)       │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         │                │                │                    │            │
│         └────────────────┴────────────────┴────────────────────┘            │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    global_resmgr (core shared library)               │    │
│  │  ┌──────────────────────────────────────────────────────────────┐   │    │
│  │  │ Sources: hap_manager.cpp, resource_manager_impl.cpp, etc.    │   │    │
│  │  │ Deps: resmgr_abc (JS bytecode)                               │   │    │
│  │  └──────────────────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘

                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  librawfile   │         │    ohresmgr     │         │ resmgr_napi_core│
│  (shlib)      │         │    (shlib)      │         │    (shlib)      │
│               │         │                 │         │                 │
│ raw_file_mgr  │         │ native_res_mgr  │         │ napi addon impl │
│               │         │                 │         │                 │
│ Deps:         │         │ Deps:           │         │ Deps:           │
│ global_resmgr │         │ global_resmgr   │         │ global_resmgr   │
└───────────────┘         └─────────────────┘         └────────┬────────┘
                                                               │
                    ┌──────────────────────────────────────────┘
                    │
                    ▼
        ┌─────────────────────┐
        │   resourcemanager   │
        │     (shlib)         │
        │                     │
        │  JS NAPI interface  │
        │  for applications   │
        └─────────────────────┘
```

## 根构建入口

### bundle.json

**路径**: `/bundle.json`

**sub_component** 数组定义了所有主要构建 target：

```json
"sub_component": [
  "//base/global/resource_management/interfaces/js/kits:resourcemanager",
  "//base/global/resource_management/interfaces/js/kits:sendableresourcemanager",
  "//base/global/resource_management/interfaces/cj:cj_resource_manager_ffi",
  "//base/global/resource_management/frameworks/resmgr:global_resmgr",
  "//base/global/resource_management/frameworks/resmgr:librawfile",
  "//base/global/resource_management/frameworks/resmgr:win_resmgr",
  "//base/global/resource_management/frameworks/resmgr:linux_resmgr",
  "//base/global/resource_management/frameworks/resmgr:mac_resmgr",
  "//base/global/resource_management/frameworks/resmgr:ohresmgr",
  "//base/global/resource_management/interfaces/ets/ani:ani_package_resmgr",
  "//base/global/resource_management/interfaces/js/innerkits/core:resmgr_napi_core_preview"
]
```

**证据**: `bundle.json:76-88`

## 特征标志

### resmgr.gni

**路径**: `/resmgr.gni`

**定义的特征标志**:
```gn
declare_args() {
  resource_management_support_icu = true
}
```

**平台检测变量**:
```gn
is_mingw = "${current_os}_${current_cpu}" == "mingw_x86_64"
is_linux = "${current_os}_${current_cpu}" == "linux_x64"
is_mac = "${current_os}_${current_cpu}" == "mac_x64" ||
         "${current_os}_${host_cpu}" == "mac_arm64"
```

**证据**: `resmgr.gni:26-28`

## 外部依赖

### 核心依赖

| 依赖 | 用途 |
|------|------|
| `hilog:libhilog` | 日志系统 |
| `hisysevent:libhisysevent` | 系统事件 |
| `hitrace:hitrace_meter` | 性能追踪 |
| `zlib:shared_libz` | 压缩解压 |
| `cJSON:cjson` | JSON 解析 |
| `icu:shared_icui18n/icuuc` | 国际化支持 (可选) |

### 绑定层依赖

| 依赖 | 用途 |
|------|------|
| `napi:ace_napi` | N-API 框架 |
| `bundle_framework:appexecfwk_core_headers` | Bundle 框架 |
| `ability_base:extractortool` | 能力基础库 |
| `ace_engine:drawable_descriptor` | ArkUI 引擎 |
| `runtime_core:ani` | ANI 运行时 |

**证据**: `frameworks/resmgr/BUILD.gn`, `interfaces/js/innerkits/core/BUILD.gn`

## 配置选项

### ICU 支持

**选项**: `resource_management_support_icu`

**默认值**: `true`

**影响**: 是否启用 ICU 国际化库支持

**使用位置**: `frameworks/resmgr/BUILD.gn`

**证据**: `resmgr.gni:27`

## 平台适配

### Windows (MinGW)

**条件**: `is_mingw == true`

**Target**: `global_resmgr_win`, `win_resmgr`

**用途**: IDE 预览环境

**证据**: `resmgr.gni:15`

### macOS

**条件**: `is_mac == true`

**Target**: `global_resmgr_mac`, `mac_resmgr`

**用途**: IDE 预览环境

**证据**: `resmgr.gni:21-22`

### Linux

**条件**: `is_linux == true`

**Target**: `global_resmgr_linux`, `linux_resmgr`

**用途**: IDE 预览环境

**证据**: `resmgr.gni:18`

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [编译产物](07_BuildArtifacts.md) - 编译产物和部署
- [目录结构](02_DirectoryStructure.md) - 代码组织和模块职责

---

**生成时间**: 2026-02-06
**证据来源**: bundle.json, resmgr.gni, frameworks/resmgr/BUILD.gn, interfaces/*/BUILD.gn
