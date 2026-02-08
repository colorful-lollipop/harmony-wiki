# GN 构建目标

## 目的

本文档详细说明 Pasteboard 的 GN 构建配置、目标定义、依赖关系和编译产物。

## 适用范围

- 进行构建配置或修改的开发者
- 需要理解编译产物结构的工程师
- 进行系统集成的工程师

## 构建目标

### 1. 共享库 (Shared Libraries)

| Target | 输出文件名 | 类型 | 安装路径 | 代码位置 |
|--------|-----------|------|----------|----------|
| `pasteboard_service` | `libpasteboard_service.z.so` | Service | `/system/lib/` | `services/BUILD.gn:65` |
| `pasteboard_client` | `libpasteboard_client.z.so` | Client | `/system/lib/` | `framework/innerkits/BUILD.gn` |
| `pasteboard_data` | `libpasteboard_data.z.so` | Data | `/system/lib/` | `framework/innerkits/BUILD.gn` |
| `pasteboard_framework` | `libpasteboard_framework.z.so` | Framework | `/system/lib/` | `framework/framework/BUILD.gn` |
| `pasteboard_adapter` | `libpasteboard_adapter.z.so` | Adapter | `/system/lib/` | `adapter/BUILD.gn` |
| `pasteboard_napi` | `libpasteboard_napi.z.so` | N-API | `/system/lib/module/` | `interfaces/kits/BUILD.gn:17` |
| `libpasteboard` | `libpasteboard.so` | NDK | `/system/lib/ndk/` | `interfaces/ndk/BUILD.gn` |
| `cj_pasteboard_ffi` | `libcj_pasteboard_ffi.so` | CJ FFI | `/system/lib/` | `interfaces/cj/BUILD.gn` |
| `pasteboard_ani` | `libpasteboard_ani.so` | ANI | `/system/lib/` | `interfaces/ani/BUILD.gn` |

### 2. HAP 包

| Target | 输出文件名 | 类型 | 安装路径 | 代码位置 |
|--------|-----------|------|----------|----------|
| `pasteboard_dialog_hap` | `pasteboard_dialog.hap` | UI | `/system/app/com.ohos.pasteboarddialog/` | `services/dialog/BUILD.gn` |

### 3. 配置文件

| Target | 输出文件名 | 类型 | 安装路径 | 代码位置 |
|--------|-----------|------|----------|----------|
| `pasteboardservice.cfg` | `pasteboardservice.cfg` | Config | `/system/etc/init/` | `etc/init/BUILD.gn` |
| `distributeddatamgr_pasteboard_sa_profiles` | `3701.json` | SA Profile | `/system/profile/` | `profile/BUILD.gn` |

### 4. Source Sets

| Target | 用途 | 代码位置 |
|--------|------|----------|
| `pasteboard_stub_proxy` | Stub/Proxy 代码 | `services/BUILD.gn:205` |
| `pasteboard_client_idl` | Client IDL | `services/BUILD.gn:229` |
| `pasteboard_service_idl` | Service IDL | `services/BUILD.gn:260` |

## 构建组

### Root Group

```gn
# BUILD.gn:18
group("pasteboard_packages") {
  if (is_standard_system) {
    deps = [
      "etc/init:pasteboardservice.cfg",
      "framework/framework:build_module",
      "framework/innerkits:pasteboard_client",
      "framework/innerkits:pasteboard_data",
      "interfaces/cj:cj_pasteboard_ffi",
      "interfaces/kits:pasteboard_napi",
      "interfaces/ndk:libpasteboard",
      "profile:distributeddatamgr_pasteboard_sa_profiles",
      "services:pasteboard_service",
    ]
  }
}
```

### Build Groups (from bundle.json)

**fwk_group**:
- `//foundation/distributeddatamgr/pasteboard/adapter:pasteboard_adapter`
- `//foundation/distributeddatamgr/pasteboard/framework/framework:pasteboard_framework`
- `//foundation/distributeddatamgr/pasteboard/framework/innerkits:pasteboard_client`
- `//foundation/distributeddatamgr/pasteboard/framework/innerkits:pasteboard_data`
- `//foundation/distributeddatamgr/pasteboard/interfaces/cj:cj_pasteboard_ffi`
- `//foundation/distributeddatamgr/pasteboard/interfaces/kits:pasteboard_napi`
- `//foundation/distributeddatamgr/pasteboard/services/dialog:pasteboard_dialog_hap`
- `//foundation/distributeddatamgr/pasteboard/interfaces/taihe:pasteboard_taihe`

**service_group**:
- `//foundation/distributeddatamgr/pasteboard/etc/init:pasteboardservice.cfg`
- `//foundation/distributeddatamgr/pasteboard/profile:distributeddatamgr_pasteboard_sa_profiles`
- `//foundation/distributeddatamgr/pasteboard/services:pasteboard_service`

## 依赖关系

### pasteboard_service 依赖

```gn
# services/BUILD.gn:123-161
deps = [
  ":pasteboard_service_idl",
  "${pasteboard_framework_path}:pasteboard_framework",
  "${pasteboard_innerkits_path}:pasteboard_data",
]

external_deps = [
  "ability_base:base",
  "ability_base:want",
  "ability_base:zuri",
  "ability_runtime:uri_permission_mgr",
  "access_token:libaccesstoken_sdk",
  "access_token:libprivacy_sdk",
  "bundle_framework:appexecfwk_base",
  "bundle_framework:appexecfwk_core",
  "c_utils:utils",
  "common_event_service:cesfwk_innerkits",
  "device_manager:devicemanagersdk",          # optional
  "dlp_permission_service:libdlp_permission_sdk",  # optional
  "ipc:ipc_single",
  "samgr:samgr_proxy",
  "screenlock_mgr:screenlock_client",         # optional
  "udmf:udmf_client",
  ...
]
```

### pasteboard_napi 依赖

```gn
# interfaces/kits/BUILD.gn:60-81
deps = [
  "${pasteboard_framework_path}:pasteboard_framework",
  "${pasteboard_innerkits_path}:pasteboard_client",
  "${pasteboard_innerkits_path}:pasteboard_data",
]

external_deps = [
  "ability_base:want",
  "ability_base:zuri",
  "ability_runtime:napi_common",
  "c_utils:utils",
  "ffrt:libffrt",
  "hilog:libhilog",
  "image_framework:image",
  "ipc:ipc_single",
  "napi:ace_napi",
  "udmf:udmf_client",
  "udmf:udmf_data_napi",
]
```

## Feature Flags

### 定义位置

```gn
# pasteboard.gni:32-38
declare_args() {
  pasteboard_dlp_part_enabled = true
  pasteboard_device_info_manager_part_enabled = true
  pasteboard_device_manager_part_enabled = true
  pasteboard_screenlock_mgr_part_enabled = true
  pasteboard_vixl_part_enabled = true
  pasteboard_dataclassification_enabled = true
}
```

### 条件编译

```gn
# services/BUILD.gn:173-199
if (pasteboard_dlp_part_enabled) {
  external_deps += [ "dlp_permission_service:libdlp_permission_sdk" ]
  defines += [ "WITH_DLP" ]
}

if (pasteboard_device_manager_part_enabled) {
  external_deps += [ "device_manager:devicemanagersdk" ]
  defines += [ "PB_DEVICE_MANAGER_ENABLE" ]
}

if (pasteboard_screenlock_mgr_part_enabled) {
  external_deps += [ "screenlock_mgr:screenlock_client" ]
  defines += [ "PB_SCREENLOCK_MGR_ENABLE" ]
}

if (pasteboard_dataclassification_enabled) {
  external_deps += [ "dataclassification:data_transit_mgr" ]
  defines += [ "PB_DATACLASSIFICATION_ENABLE" ]
}
```

## 编译产物

### 输出映射

```
out/target/product/{product}/
├── system/
│   ├── lib/
│   │   ├── libpasteboard_service.z.so
│   │   ├── libpasteboard_client.z.so
│   │   ├── libpasteboard_data.z.so
│   │   ├── libpasteboard_framework.z.so
│   │   ├── libpasteboard_adapter.z.so
│   │   ├── libcj_pasteboard_ffi.so
│   │   └── libpasteboard_ani.so
│   ├── lib/module/
│   │   └── libpasteboard_napi.z.so
│   ├── lib/ndk/
│   │   └── libpasteboard.so
│   ├── app/com.ohos.pasteboarddialog/
│   │   └── pasteboard_dialog.hap
│   ├── etc/init/
│   │   └── pasteboardservice.cfg
│   └── profile/
│       └── 3701.json
```

### 运行时加载关系

```
Application
    ↓ loads
libpasteboard_napi.z.so
    ↓ depends on
libpasteboard_client.z.so → libpasteboard_framework.z.so → libpasteboard_data.z.so
    ↓ IPC (binder)
system_server (pasteboard_service)
    ↓ loads
libpasteboard_service.z.so → libpasteboard_framework.z.so → libpasteboard_data.z.so
```

## 安全配置

### 安全加固

```gn
# services/BUILD.gn:66-74
ohos_shared_library("pasteboard_service") {
  branch_protector_ret = "pac_ret"  # PAC-RET 保护
  sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
}
```

### 编译标志

```gn
# services/BUILD.gn:112-121
cflags_cc = [
  "-fdata-sections",
  "-ffunction-sections",
  "-fno-asynchronous-unwind-tables",
  "-fno-unwind-tables",
  "-fomit-frame-pointer",
  "-fvisibility=hidden",
  "-D_FORTIFY_SOURCE=2",
  "-O2",
]
```

## 关键结论

1. **模块化构建**: 清晰的服务、框架、接口分层，每个模块独立构建目标。

2. **Feature 裁剪**: 通过 6 个 feature flag 支持功能裁剪，适配不同设备。

3. **安全加固**: 所有共享库启用 PAC-RET、CFI、边界检查等安全机制。

4. **多语言支持**: N-API、NDK、CJ、ANI、Taihe 五种接口，分别构建为独立库。

5. **运行时依赖**: 服务进程独立运行，客户端库通过 IPC 与服务通信。

## 相关链接

- [目录结构 → 02_Directory_Structure.md](02_Directory_Structure.md)
- [内部 API → 04_Inner_API.md](04_Inner_API.md)
- [安全评审 → 06_Security.md](06_Security.md)
