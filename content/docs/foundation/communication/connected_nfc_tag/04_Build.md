# Connected NFC Tag - 构建系统文档

## 1. GN 构建概述

### 1.1 构建系统信息

| 属性 | 值 |
|------|------|
| **构建系统** | GN (Generate Ninja) |
| **构建入口** | `BUILD.gn` (根目录) |
| **配置文件** | `connected_nfc_tag.gni` |
| **组件类型** | subsystem → part |

### 1.2 构建产物类型

| 产物类型 | 说明 |
|----------|------|
| `.so` | 共享库 (N-API、服务) |
| `.a` | 静态库 (HDI 适配层) |
| SA Profile | 系统能力注册配置 |

---

## 2. GN Targets 清单

### 2.1 共享库 (Shared Libraries)

| Target | 产物 | 路径 | 用途 |
|--------|------|------|------|
| `connectedtag` | `libconnectedtag.z.so` | `frameworks/js/napi/BUILD.gn` | JS/NAPI 接口 |
| `nfc_tag_service` | `libnfc_tag_service.z.so` | `services/BUILD.gn` | NFC Tag 服务 |
| `nfc_tag_inner_kits` | `libnfc_tag_inner_kits.z.so` | `interfaces/inner_api/BUILD.gn` | 系统内部 API |

### 2.2 静态库 (Static Libraries)

| Target | 产物 | 路径 | 用途 |
|--------|------|------|------|
| `nfc_tag_hdi_adapter` | `libnfc_tag_hdi_adapter.a` | `services/src/hdi/BUILD.gn` | HDI 适配层 |
| `nfc_tag_sa_listener` | `libnfc_tag_sa_listener.a` | `utils/sa_listener/BUILD.gn` | SA 监听器 |

### 2.3 配置产物

| Target | 产物 | 路径 | 用途 |
|--------|------|------|------|
| `nfc_tag_profile` | `1148.json` | `sa_profile/BUILD.gn` | SA 注册配置 |
| `etc` | 配置目录 | `services/etc/init/BUILD.gn` | 服务初始化配置 |

---

## 3. Target 详细说明

### 3.1 connectedtag (N-API 接口)

**文件**: `frameworks/js/napi/BUILD.gn`

```gn
ohos_shared_library("connectedtag") {
  install_enable = true
  include_dirs = [
    "$NFC_TAG_DIR/interfaces/inner_api/include",
  ]
  defines = [ "NFC_TAG_HILOG" ]
  branch_protector_ret = "pac_ret"

  sources = [
    "nfc_napi_adapter.cpp",
    "nfc_napi_entry.cpp",
    "nfc_napi_event.cpp",
    "nfc_napi_utils.cpp",
  ]
  deps = [ "$NFC_TAG_DIR/interfaces/inner_api:nfc_tag_inner_kits" ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "napi:ace_napi",
  ]

  relative_install_dir = "module"
  part_name = "connected_nfc_tag"
  subsystem_name = "communication"
}
```

**产物**: `libconnectedtag.z.so`
**安装路径**: `$module_install_dir/lib/`
**依赖**:
- `nfc_tag_inner_kits` (内部 API)
- `ace_napi` (N-API 框架)
- `libhilog` (日志)
- `ipc_core` (IPC)

---

### 3.2 nfc_tag_service (服务)

**文件**: `services/BUILD.gn`

```gn
ohos_shared_library("nfc_tag_service") {
  cflags = common_cflags
  cflags_cc = common_cflags
  sanitize = global_sanitize
  defines = global_defines
  branch_protector_ret = "pac_ret"

  version_script = "libnfc_tag_service_version_script.txt"

  include_dirs = nfc_tag_service_include_dirs

  sources = nfc_tag_service_source

  deps = [
    "$NFC_TAG_DIR/services/etc/init:etc",
    "$NFC_TAG_DIR/services/src/hdi:nfc_tag_hdi_adapter",
  ]

  external_deps = nfc_tag_service_external_deps

  part_name = "connected_nfc_tag"
  subsystem_name = "communication"
}
```

**产物**: `libnfc_tag_service.z.so`
**安装路径**: `/system/lib/`
**条件编译**:
```gn
if (connected_nfc_tag_only_system_app_access_api) {
  nfc_tag_service_source += ["src/nfc_tag_sys_perm.cpp"]
}
```

**依赖**:
- `nfc_tag_hdi_adapter` (HDI 适配)
- `access_token:libaccesstoken_sdk` (权限)
- `ipc_core` (IPC)
- `safwk:system_ability_fwk` (SA 框架)
- `samgr:samgr_proxy` (服务管理)

---

### 3.3 nfc_tag_inner_kits (内部 API)

**文件**: `interfaces/inner_api/BUILD.gn`

```gn
ohos_shared_library("nfc_tag_inner_kits") {
  cflags = common_cflags
  cflags_cc = common_cflags
  sanitize = global_sanitize
  defines = global_defines
  branch_protector_ret = "pac_ret"
  configs = [ ":nfc_tag_config" ]
  public_configs = [ ":nfc_tag_public_config" ]

  sources = [
    "src/nfc_tag_callback_stub.cpp",
    "src/nfc_tag_client.cpp",
    "src/nfc_tag_proxy.cpp",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]

  subsystem_name = "communication"
  part_name = "connected_nfc_tag"
}
```

**产物**: `libnfc_tag_inner_kits.z.so`
**安装路径**: `/system/lib/`

---

### 3.4 nfc_tag_hdi_adapter (HDI 适配)

**文件**: `services/src/hdi/BUILD.gn`

```gn
ohos_static_library("nfc_tag_hdi_adapter") {
  cflags = common_cflags
  cflags_cc = common_cflags
  sanitize = global_sanitize
  defines = global_defines
  branch_protector_ret = "pac_ret"

  include_dirs = [
    "include",
    "$NFC_TAG_DIR/interfaces/inner_api/include",
    "$NFC_TAG_DIR/utils/sa_listener",
    "//third_party/bounds_checking_function/include",
  ]

  sources = [
    "src/nfc_tag_hdi_adapter.cpp",
    "src/nfc_tag_hdi_impl.cpp",
  ]

  deps = [
    "$NFC_TAG_DIR/utils/sa_listener:nfc_tag_sa_listener",
  ]

  external_deps = [
    "c_utils:utils",
    "drivers_interface_connected_nfc_tag:libconnected_nfc_tag_proxy_1.1",
    "hdf_core:libhdi",
    "hilog:libhilog",
    "ipc:ipc_core",
  ]

  subsystem_name = "communication"
  part_name = "connected_nfc_tag"
}
```

**产物**: `libnfc_tag_hdi_adapter.a` (静态库)

---

## 4. 编译配置

1 全局配置### 4. (connected_nfc_tag.gni)

```gn
NFC_TAG_DIR = "//foundation/communication/connected_nfc_tag"

declare_args() {
  connected_nfc_tag_only_system_app_access_api = false
  if (defined(global_parts_info.connected_nfc_tag_only_system_app_access_api)) {
    connected_nfc_tag_only_system_app_access_api = true
  }
}

common_cflags = [
  "-D_FORTIFY_SOURCE=2",
  "-fdata-sections",
  "-ffunction-sections",
  "-Os",
  "-O2",
]

global_defines = ["NFC_TAG_HILOG"]

global_sanitize = {
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
  integer_overflow = true
  ubsan = true
  debug = false
}
```

### 4.2 安全编译选项

| 选项 | 值 | 说明 |
|------|------|------|
| `-D_FORTIFY_SOURCE=2` | 启用 | 运行时缓冲区溢出检测 |
| `-fdata-sections` | 启用 | 数据段优化 |
| `-ffunction-sections` | 启用 | 函数段优化 |
| `branch_protector_ret` | `pac_ret` | ARM PAC 返回地址保护 |
| `boundary_sanitize` | `true` | 边界检查 |
| `cfi` | `true` | 控制流完整性 |
| `ubsan` | `true` | 未定义行为检测 |

---

## 5. 特性开关

### 5.1 connected_nfc_tag_only_system_app_access_api

**作用**: 控制 NFC Tag API 是否仅限系统应用访问。

**启用方式**:
```gn
# 方式 1: gn 参数
gn gen out/release --args="global_parts_info={ connected_nfc_tag_only_system_app_access_api = true }"

# 方式 2: build.gni
connected_nfc_tag_only_system_app_access_api = true
```

**效果**:
- 启用: `nfc_tag_sys_perm.cpp` 加入编译
- 禁用: 系统应用和普通应用均可访问

**注册位置** (`bundle.json`):
```json
"features": ["connected_nfc_tag_only_system_app_access_api"]
```

---

## 6. 编译产物映射

```
编译产物                          源文件/Target
───────────────────────────────────────────────────────────
libconnectedtag.z.so            → frameworks/js/napi:connectedtag
libnfc_tag_service.z.so         → services:nfc_tag_service
libnfc_tag_inner_kits.z.so      → interfaces/inner_api:nfc_tag_inner_kits
libnfc_tag_hdi_adapter.a        → services/src/hdi:nfc_tag_hdi_adapter
libnfc_tag_sa_listener.a       → utils/sa_listener:nfc_tag_sa_listener
libnfc_tag_service.z.so         → services:nfc_tag_service
```

---

## 7. 组件配置

### 7.1 bundle.json

```json
{
  "name": "@ohos/connected_nfc_tag",
  "version": "3.1",
  "component": {
    "name": "connected_nfc_tag",
    "subsystem": "communication",
    "syscap": [
      "SystemCapability.Communication.ConnectedTag"
    ],
    "features": ["connected_nfc_tag_only_system_app_access_api"],
    "deps": {
      "components": [
        "ipc",
        "c_utils",
        "hilog",
        "napi",
        "access_token",
        "hisysevent",
        "safwk",
        "samgr",
        "hdf_core",
        "drivers_interface_connected_nfc_tag"
      ]
    },
    "build": {
      "group_type": {
        "fwk_group": [
          "//foundation/communication/connected_nfc_tag/interfaces/inner_api:nfc_tag_inner_kits",
          "//foundation/communication/connected_nfc_tag/frameworks/js/napi:connectedtag"
        ],
        "service_group": [
          "//foundation/communication/connected_nfc_tag/sa_profile:nfc_tag_profile",
          "//foundation/communication/connected_nfc_tag/services:nfc_tag_service"
        ]
      }
    }
  }
}
```

### 7.2 SA 配置 (1148.json)

```json
{
  "process": "nfc_tag_service",
  "systemability": [
    {
      "name": 1148,
      "libpath": "libnfc_tag_service.z.so",
      "run-on-create": true,
      "distributed": false,
      "dump_level": 1,
      "min_hdi_proxy_version": ["libconnected_nfc_tag_proxy_1.1.z.so"]
    }
  ]
}
```

---

## 8. 构建命令

### 8.1 完整构建

```bash
# 初始化构建
hb set -p <product>
hb build -f

# 单独构建 connected_nfc_tag
hb build connected_nfc_tag
```

### 8.2 GN 命令

```bash
# 生成 ninja 文件
gn gen out/release --args="global_parts_info={ connected_nfc_tag_only_system_app_access_api = true }"

# 编译
ninja -C out/release //foundation/communication/connected_nfc_tag/...
```

---

## 9. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_NAPI.md](./02_NAPI.md) |
| 内部 API | [03_InnerAPI.md](./03_InnerAPI.md) |
| 安全评估 | [05_Security.md](./05_Security.md) |
