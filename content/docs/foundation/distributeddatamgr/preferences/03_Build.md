# 构建配置

本文档描述 Preferences 模块的 GN 构建配置、Targets 列表和编译产物说明。

## 1 构建系统概述

### 1.1 构建工具

- **构建系统**：GN (Generate Ninja)
- **构建命令**：hb (OpenHarmony Build)

### 1.2 构建入口

**根配置文件**：`preferences.gni`

**内容**：

```gni
preferences_base_path = "//foundation/distributeddatamgr/preferences"
preferences_napi_path = "${preferences_base_path}/frameworks/js/napi"
preferences_cj_path = "${preferences_base_path}/frameworks/cj/src"
preferences_native_path = "${preferences_base_path}/frameworks/native"
preferences_innerapi_path = "${preferences_base_path}/interfaces/inner_api"
preferences_ndk_path = "${preferences_base_path}/frameworks/ndk"
preferences_interfaces_path = "${preferences_base_path}/interfaces"
```

---

## 2 模块配置

### 2.1 bundle.json 配置

**文件位置**：`bundle.json`

**组件信息**：

```json
{
  "name": "preferences",
  "subsystem": "distributeddatamgr",
  "syscap": [
    "SystemCapability.DistributedDataManager.Preferences.Core",
    "SystemCapability.DistributedDataManager.Preferences.Core.Lite"
  ],
  "features": [],
  "adapted_system_type": ["standard"],
  "rom": "512KB",
  "ram": "1024KB"
}
```

---

## 3 Sub Components

**bundle.json 位置**：`bundle.json:70-79`

### 3.1 子组件列表

| # | 子组件路径 | 类型 | 说明 |
|---|-----------|------|------|
| 1 | `//foundation/distributeddatamgr/preferences/interfaces/inner_api:native_preferences` | inner_kit | Native 内部接口 |
| 2 | `//foundation/distributeddatamgr/preferences/interfaces/ndk:libohpreferences` | inner_kit | NDK C 接口 |
| 3 | `//foundation/distributeddatamgr/preferences/frameworks/js/napi/common:preferences_jscommon` | sub_component | JS 公共组件 |
| 4 | `//foundation/distributeddatamgr/preferences/frameworks/js/napi/preferences:preferences` | sub_component | Preferences JS API |
| 5 | `//foundation/distributeddatamgr/preferences/frameworks/js/napi/sendable_preferences:sendablepreferences` | sub_component | 可迁移首选项 |
| 6 | `//foundation/distributeddatamgr/preferences/frameworks/js/napi/storage:storage` | sub_component | Storage JS API |
| 7 | `//foundation/distributeddatamgr/preferences/frameworks/js/napi/system_storage:storage_napi` | sub_component | System Storage JS API |
| 8 | `//foundation/distributeddatamgr/preferences/frameworks/ets/taihe/preferences:preferences_taihe` | sub_component | ArkTS/Taihe 接口 |

---

## 4 GN Targets 详解

### 4.1 native_preferences (Inner API)

**BUILD.gn 位置**：`interfaces/inner_api/BUILD.gn`

**目标类型**：ohos_shared_library / ohos_static_library

**产物类型**：
- 动态库：`libnative_preferences.so`（OHOS）
- 静态库：`libnative_preferences_static.a`

**源码文件**：

```gn
base_sources = [
  "${preferences_native_path}/platform/src/preferences_dfx_adapter.cpp",
  "${preferences_native_path}/platform/src/preferences_file_lock.cpp",
  "${preferences_native_path}/platform/src/preferences_task_processor.cpp",
  "${preferences_native_path}/platform/src/preferences_thread.cpp",
  "${preferences_native_path}/src/base64_helper.cpp",
  "${preferences_native_path}/src/preferences_base.cpp",
  "${preferences_native_path}/src/preferences_helper.cpp",
  "${preferences_native_path}/src/preferences_impl.cpp",
  "${preferences_native_path}/src/preferences_observer.cpp",
  "${preferences_native_path}/src/preferences_utils.cpp",
  "${preferences_native_path}/src/preferences_value.cpp",
  "${preferences_native_path}/src/preferences_xml_utils.cpp",
]
```

**平台差异**（OHOS 特有）：

```gn
sources += [
  "${preferences_native_path}/platform/src/preferences_db_adapter.cpp",
  "${preferences_native_path}/src/preferences_enhance_impl.cpp",
  "${preferences_native_path}/src/preferences_value_parcel.cpp",
  "${preferences_native_path}/platform/src/preferences_ffrt_task_processor.cpp",
]
```

**依赖配置**：

```gn
external_deps = [
  "ability_base:zuri",
  "ability_runtime:dataobs_manager",
  "access_token:libaccesstoken_sdk",
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "ffrt:libffrt",
  "hilog:libhilog",
  "hisysevent:libhisysevent",
  "hitrace:hitrace_meter",
  "ipc:ipc_single",
  "libxml2:libxml2",
]
```

**配置信息**：

```gn
configs = [ ":native_preferences_config" ]
public_configs = [ ":native_preferences_public_config" ]

include_dirs = [
  "include",
  "${preferences_base_path}/frameworks/common/include",
  "${preferences_native_path}/include",
  "${preferences_native_path}/platform/include/",
]
```

**Sanitizer 配置**（OHOS）：

```gn
sanitize = {
  boundary_sanitize = true
  ubsan = true
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

**内联标签**：

```gn
innerapi_tags = [
  "platformsdk",
  "sasdk",
]
```

---

### 4.2 libohpreferences (NDK)

**BUILD.gn 位置**：`interfaces/ndk/BUILD.gn`

**目标类型**：ohos_shared_library

**产物类型**：动态库 `libohpreferences.so`

**源码**：

```gn
sources = [
  "${preferences_ndk_path}/src/oh_convertor.cpp",
  "${preferences_ndk_path}/src/oh_preferences.cpp",
  "${preferences_ndk_path}/src/oh_preferences_option.cpp",
  "${preferences_ndk_path}/src/oh_preferences_value.cpp",
]
```

**依赖**：

```gn
deps = [
  "${preferences_base_path}/interfaces/inner_api:native_preferences",
]
```

---

### 4.3 preferences (JS API)

**BUILD.gn 位置**：`frameworks/js/napi/preferences/BUILD.gn`

**目标类型**：ohos_shared_library

**产物类型**：动态库 `libpreferences.so`

**源码文件**：

```gn
base_sources = [
  "${preferences_napi_path}/preferences/src/entry_point.cpp",
  "${preferences_napi_path}/preferences/src/napi_preferences.cpp",
  "${preferences_napi_path}/preferences/src/napi_preferences_helper.cpp",
]
```

**依赖配置**：

```gn
deps = [
  "${preferences_base_path}/interfaces/inner_api:native_preferences",
  "${preferences_napi_path}/common:preferences_jscommon",
]

external_deps = [
  "ability_runtime:abilitykit_native",
  "ability_runtime:extensionkit_native",
  "ability_runtime:napi_base_context",
  "ability_runtime:napi_common",
  "c_utils:utils",
  "hilog:libhilog",
  "ipc:ipc_single",
  "napi:ace_napi",
]
```

**安装路径**：

```gn
relative_install_dir = "module/data"
subsystem_name = "distributeddatamgr"
part_name = "preferences"
```

---

### 4.4 preferences_jscommon (JS Common)

**BUILD.gn 位置**：`frameworks/js/napi/common/BUILD.gn`

**目标类型**：ohos_shared_library

**产物类型**：动态库 `libpreferences_jscommon.so`

**源码文件**：

```gn
sources = [
  "${preferences_napi_path}/common/src/js_common_utils.cpp",
  "${preferences_napi_path}/common/src/js_observer.cpp",
  "${preferences_napi_path}/common/src/js_sendable_utils.cpp",
  "${preferences_napi_path}/common/src/napi_async_call.cpp",
  "${preferences_base_path}/frameworks/common/src/preferences_error.cpp",
  "${preferences_napi_path}/common/src/napi_preferences_observer.cpp",
  "${preferences_napi_path}/common/src/uv_queue.cpp",
]
```

---

### 4.5 其他 JS Targets

| Target | 路径 | 产物 |
|--------|------|------|
| sendablepreferences | `frameworks/js/napi/sendable_preferences/BUILD.gn` | libsendablepreferences.so |
| storage | `frameworks/js/napi/storage/BUILD.gn` | libstorage.so |
| storage_napi | `frameworks/js/napi/system_storage/BUILD.gn` | libstorage_napi.so |

---

## 5 依赖关系

### 5.1 模块依赖图

```
┌─────────────────────────────────────────────────────────┐
│                   libpreferences.so                      │
│                    (JS API)                              │
├─────────────────────────────────────────────────────────┤
│                        ↓                                 │
│         ┌─────────────────────────┐                      │
│         │  libpreferences_jscommon│                      │
│         │      (公共组件)          │                      │
│         └─────────────────────────┘                      │
│                        ↓                                 │
│         ┌─────────────────────────┐                      │
│         │   libnative_preferences  │                      │
│         │    (Native 核心)         │                      │
│         └─────────────────────────┘                      │
│                        ↓                                 │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐│
│  │libxml2 │ │ipc    │ │ffrt    │ │hilog  │ │access_ ││
│  │        │ │       │ │        │ │       │ │token   ││
│  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘┘
└─────────────────────────────────────────────────────────┘
```

### 5.2 外部依赖列表

| 依赖组件 | 用途 | 类型 |
|----------|------|------|
| ability_runtime | 应用运行时 | external_deps |
| ability_base | 基础能力 | external_deps |
| access_token | 访问令牌 | external_deps |
| napi | Node API | external_deps |
| hilog | 日志 | external_deps |
| c_utils | C 工具库 | external_deps |
| ipc | 进程间通信 | external_deps |
| libxml2 | XML 解析 | external_deps |
| ffrt | 任务调度 | external_deps |
| hitrace | 性能追踪 | external_deps |
| hisysevent | 事件上报 | external_deps |
| bounds_checking_function | 边界检查 | external_deps |

---

## 6 编译产物

### 6.1 产物清单

| 产物名称 | 类型 | 说明 | 安装路径 |
|----------|------|------|----------|
| `libnative_preferences.so` | 动态库 | Native 内部接口 | system/lib/module/ |
| `libnative_preferences_static.a` | 静态库 | Native 内部接口 | - |
| `libohpreferences.so` | 动态库 | NDK C 接口 | system/lib/ |
| `libpreferences.so` | 动态库 | JS API | module/data/ |
| `libpreferences_jscommon.so` | 动态库 | JS 公共组件 | module/data/ |
| `libsendablepreferences.so` | 动态库 | 可迁移首选项 | module/data/ |
| `libstorage.so` | 动态库 | Storage API | module/data/ |
| `libstorage_napi.so` | 动态库 | System Storage | module/data/ |

### 6.2 产物关系

```
应用 (JS/NDK/CJ)
     ↓
libpreferences.so (JS API)
     ↓
libpreferences_jscommon.so (公共组件)
     ↓
libnative_preferences.so (Native 核心)
     ↓
system libs (ipc, libxml2, ffrt, etc.)
```

### 6.3 头文件位置

| 头文件类型 | 路径 |
|-----------|------|
| NDK C 头文件 | `interfaces/ndk/include/` |
| Inner API 头文件 | `interfaces/inner_api/include/` |
| Native 头文件 | `frameworks/native/include/` |
| JS N-API 头文件 | `frameworks/js/napi/*/include/` |

---

## 7 平台差异

### 7.1 OHOS (OpenHarmony)

- **native_preferences**: 动态库 + 静态库
- **Sanitizer**: 启用 CFI、UBSan、Boundary Sanitize
- **任务调度**: 使用 FFRT

### 7.2 Android/iOS

- **native_preferences**: 静态库
- **任务调度**: 使用 ExecutorPool
- **Sanitizer**: 默认不启用

### 7.3 Windows/macOS

- **native_preferences**: 动态库
- **cross_platform**: 定义 CROSS_PLATFORM
- **任务调度**: 使用 ExecutorPool

---

## 8 构建命令

### 8.1 全量构建

```bash
# 使用 hb 构建整个系统
hb build -f
```

### 8.2 单模块构建

```bash
# 构建 preferences 模块
hb build -p distributeddatamgr -t preferences
```

### 8.3 单独构建 GN

```bash
# 使用 gn 和 ninja
gn gen out/preferences
ninja -C out/preferences preferences
```

---

## 9 构建配置说明

### 9.1 编译选项

```gn
cflags_cc = [
  "-std=c++17",
  "-stdlib=libc++",
]

// 可见性控制
config("preferences_public_config") {
  visibility = [ ":*" ]
  include_dirs = [ "include" ]
}
```

### 9.2 符号导出

```gn
cflags_cc = [
  "-std=c++17",
  "-stdlib=libc++",
  "-fvisibility=hidden",  // 隐藏符号
]
```

### 9.3 Subsystem/Part 配置

```gn
subsystem_name = "distributeddatamgr"
part_name = "preferences"
```

---

## 10 相关文档

| 文档 | 说明 |
|------|------|
| [概览](./00_Overview.md) | 模块定位和能力边界 |
| [架构设计](./01_Architecture.md) | 内部架构详解 |
| [API 参考](./02_API_Reference.md) | 完整 API 清单 |
| [安全评审](./04_Security_Review.md) | 安全风险分析 |
