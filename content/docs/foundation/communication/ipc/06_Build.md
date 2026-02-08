# 构建配置

## 6.1 GN 构建系统概述

OpenHarmony IPC 组件使用 **GN (Generate Ninja)** 作为构建系统。

### 构建入口

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 根构建入口，定义 `ipc_components` 组 |
| `config.gni` | 全局配置变量 |
| `config/BUILD.gn` | 公共配置目标 |

### 配置变量 (config.gni)

| 变量 | 默认值 | 用途 |
|------|--------|------|
| `ipc_feature_rpc_enabled` | `false` | 启用 RPC 功能 |
| `ipc_feature_test_enabled` | `false` | 启用测试功能 |
| `ipc_feature_trace_enabled` | `false` | 启用追踪功能 |
| `ipc_feature_freeze_enabled` | `false` | 启用进程冻结 |
| `ipc_feature_memory_usage_enabled` | `false` | 启用内存统计 |
| `resourceschedule_ffrt_support` | `false` | FFRT 支持 |
| `hiviewdfx_hisysevent_support` | `false` | HiSysEvent 支持 |
| `hiviewdfx_backtrace_support` | `false` | Backtrace 支持 |

**证据**: `config.gni:20-29`

## 6.2 主构建目标

### 根目标 (BUILD.gn)

```gn
group("ipc_components") {
  if (os_level == "standard") {
    deps = [
      "$SUBSYSTEM_DIR/interfaces/innerkits/c_api:ipc_capi",
      "$SUBSYSTEM_DIR/interfaces/innerkits/cj:cj_ipc_ffi",
      "$SUBSYSTEM_DIR/interfaces/innerkits/ipc_core:ipc_core",
      "$SUBSYSTEM_DIR/interfaces/innerkits/ipc_single:ipc_single",
      "$SUBSYSTEM_DIR/interfaces/kits/ndk:ipc_capi",
      "$SUBSYSTEM_DIR/ipc/native/src/core:ipc_common",
    ]
    if (support_jsapi) {
      deps += [
        "$SUBSYSTEM_DIR/interfaces/innerkits/ipc_napi_common:ipc_napi",
        "$SUBSYSTEM_DIR/interfaces/kits/js/napi:rpc",
      ]
    }
    if (!build_ohos_sdk) {
      deps += [
        "$SUBSYSTEM_DIR/interfaces/innerkits/libdbinder:libdbinder",
        "$SUBSYSTEM_DIR/interfaces/innerkits/rust:rust_ipc_component",
      ]
    }
  } else {
    deps = [ "$SUBSYSTEM_DIR/interfaces/innerkits/c:rpc" ]
  }
}
```

**证据**: `BUILD.gn:20-44`

## 6.3 核心库 Targets

### ipc_common（基础库）

| 属性 | 值 |
|------|-----|
| **目标** | `ipc_common` |
| **类型** | `ohos_shared_library` |
| **产物** | `libipc_common.z.so` |
| **路径** | `ipc/native/src/core:BUILD.gn` |

**关键 Sources**:
```gn
sources = [
  "framework/source/ipc_payload_statistics_impl.cpp",
  "framework/source/process_skeleton.cpp",
  "invoker/source/binder_connector.cpp",
]
```

**关键 Deps**:
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "hilog:libhilog",
]
```

**Defines**:
```gn
defines = [ "FFRT_IPC_ENABLE" ]
```

### ipc_single（主 IPC 实现）

| 属性 | 值 |
|------|-----|
| **目标** | `ipc_single` |
| **类型** | `ohos_shared_library` |
| **产物** | `libipc_single.z.so` |
| **路径** | `interfaces/innerkits/ipc_single:BUILD.gn` |

**关键 Sources** (38+ 文件):
- DBinder: `databus_socket_listener.cpp`, `dbinder_databus_invoker.cpp`
- Framework: `ipc_object_proxy.cpp`, `ipc_object_stub.cpp`, `message_parcel.cpp`
- Invoker: `binder_invoker.cpp`, `invoker_factory.cpp`

**关键 Deps**:
```gn
deps = [ "$SUBSYSTEM_DIR/ipc/native/src/core:ipc_common" ]
external_deps = [
  "c_utils:utils",
  "ffrt:libffrt",
  "hilog:libhilog",
  "selinux:libselinux",
]
```

### ipc_core（预编译包装）

| 属性 | 值 |
|------|-----|
| **目标** | `ipc_core` |
| **类型** | `ohos_prebuilt_shared_library` |
| **产物** | `libipc_core.z.so` |
| **路径** | `interfaces/innerkits/ipc_core:BUILD.gn` |

**Source**:
```gn
source = "${root_out_dir}/communication/ipc/libipc_single.z.so"
```

**Symlinks**:
```gn
symlink_ext = [ "lib/platformsdk/libipc_core.z.so" ]  # arm
symlink_ext = [ "lib64/platformsdk/libipc_core.z.so" ]  # arm64/x86_64
```

## 6.4 API 层 Targets

### C API (ipc_capi)

| 属性 | 值 |
|------|-----|
| **目标** | `ipc_capi` |
| **类型** | `ohos_shared_library` |
| **产物** | `libipc_capi.z.so` |
| **安装路径** | `./ndk/` |

**Sources**: `ipc_cparcel.cpp`, `ipc_cremote_object.cpp`, `ipc_cskeleton.cpp`, etc.

### NDK Kit

| 目标 | 类型 | 描述 |
|------|------|------|
| `ipc_capi_header` | `ohos_ndk_headers` | 安装 C 头文件 |
| `libipc_capi` | `ohos_ndk_library` | NDK 库元数据 |

**导出头文件**:
- `ipc_cparcel.h`
- `ipc_cremote_object.h`
- `ipc_cskeleton.h`
- `ipc_error_code.h`
- `ipc_kit.h`

### JS NAPI (rpc)

| 属性 | 值 |
|------|-----|
| **目标** | `rpc` |
| **类型** | `ohos_shared_library` |
| **产物** | `librpc.z.so` |
| **安装路径** | `module/` |

**Sources**:
```gn
sources = [
  "$SUBSYSTEM_DIR/ipc/native/src/napi/src/napi_calling_info.cpp",
  "$SUBSYSTEM_DIR/ipc/native/src/napi/src/napi_ipc_skeleton.cpp",
  "$SUBSYSTEM_DIR/ipc/native/src/napi/src/napi_message_option.cpp",
  "$SUBSYSTEM_DIR/ipc/native/src/napi/src/napi_remote_proxy.cpp",
  "$SUBSYSTEM_DIR/ipc/native/src/napi/src/napi_rpc_native_module.cpp",
]
```

**Deps**:
```gn
deps = [
  "$SUBSYSTEM_DIR/interfaces/innerkits/ipc_core:ipc_core",
  "$SUBSYSTEM_DIR/interfaces/innerkits/ipc_napi_common:ipc_napi",
]
external_deps = [
  "c_utils:utils",
  "hilog:libhilog",
  "napi:ace_napi",
]
```

### DBinder (libdbinder)

| 属性 | 值 |
|------|-----|
| **目标** | `libdbinder` |
| **类型** | `ohos_shared_library` |
| **产物** | `libdbinder.z.so` |

**Sources**: `dbinder_death_recipient.cpp`, `dbinder_service.cpp`, `dbinder_service_stub.cpp`

### Rust 绑定

| 目标 | 类型 | 产物 |
|------|------|------|
| `ipc_rust` | `ohos_rust_shared_library` | `libipc_rust.so` |
| `ipc_rust_cxx` | `ohos_static_library` | C++ Rust 包装 |

## 6.5 产物清单

### 标准系统产物

| 产物 | 类型 | 路径 |
|------|------|------|
| `libipc_common.z.so` | 共享库 | `out/.../communication/ipc/` |
| `libipc_single.z.so` | 共享库 | `out/.../communication/ipc/` |
| `libipc_core.z.so` | 共享库 | `system/lib64/platformsdk/` |
| `librpc.z.so` | 共享库 | `out/.../module/` |
| `libipc_capi.z.so` | 共享库 | `out/.../ndk/` |
| `libdbinder.z.so` | 共享库 | `out/.../` |
| `libipc_rust.so` | 共享库 | `out/.../` |

### SDK 符号链接

| 符号链接 | 目标 | 架构 |
|----------|------|------|
| `lib/platformsdk/libipc_core.z.so` | `libipc_core.z.so` | arm |
| `lib64/platformsdk/libipc_core.z.so` | `libipc_core.z.so` | arm64/x86_64 |

## 6.6 依赖关系图

```
ipc_components (group)
│
├── ipc_single (shared_lib)
│   └── ipc_common (shared_lib)
│       ├── c_utils:utils
│       ├── ffrt:libffrt
│       ├── hilog:libhilog
│       └── selinux:libselinux
│
├── ipc_core (prebuilt)
│   └── ipc_single (source)
│       └── c_utils:utils
│
├── ipc_capi (shared_lib)
│   ├── ipc_common
│   └── ipc_core
│       └── c_utils:utils
│
├── ipc_napi (shared_lib)
│   ├── ipc_core
│   ├── c_utils:utils
│   ├── napi:ace_napi
│   └── libuv:uv
│
├── rpc (shared_lib)
│   ├── ipc_core
│   └── ipc_napi
│
├── libdbinder (shared_lib)
│   ├── ipc_common
│   ├── ipc_core
│   └── ffrt:libffrt
│
└── rust_ipc_component (group)
    └── ipc_rust (rust_shared_lib)
        └── ipc_rust_cxx (static_lib)
            └── ipc_single
```

## 6.7 编译命令

### 完整编译

```bash
# 编译整个 IPC 组件
hb set -p <board>
hb build -p <product>
```

### 指定组件编译

```bash
# 单独编译 ipc 组件
hb build -p ipc
```

### GN 直接编译

```bash
# 使用 gn/ninja
gn gen out/<target>
ninja -C out/<target> ipc_components
```

---

*证据来源*:
- `BUILD.gn` - 根构建文件
- `config.gni` - 配置变量
- `interfaces/innerkits/ipc_single:BUILD.gn` - ipc_single 目标
- `interfaces/kits/js/napi:BUILD.gn` - JS NAPI 目标
- Phase 1 全局扫描结果
