# GN 构建系统

## 目的

本文档描述 `telephony_core_service` 的 GN 构建配置，包括构建目标、依赖关系和编译产物。

---

## 构建文件清单

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 主构建文件，定义核心服务库 |
| `telephony_core_service.gni` | GN 导入文件，公共变量定义 |
| `interfaces/innerkits/BUILD.gn` | 内部 API 库构建 |
| `frameworks/js/sim/BUILD.gn` | SIM N-API 构建 |
| `frameworks/js/network_search/BUILD.gn` | Radio N-API 构建 |
| `frameworks/js/esim/BUILD.gn` | eSIM N-API 构建 |
| `frameworks/js/vcard/BUILD.gn` | VCard N-API 构建 |
| `utils/BUILD.gn` | 工具库构建 |

---

## Feature Flags

**文件**: `telephony_core_service.gni`

```gn
# eSIM 支持
declare_args() {
  core_service_support_esim = false
}

# 低功耗模式
declare_args() {
  core_service_low_power_class_2 = false
}

# 卫星通信支持
declare_args() {
  core_service_satellite = false
}
```

### 条件编译定义

```gn
telephony_extra_defines = []

if (core_service_support_esim) {
  telephony_extra_defines += [ "CORE_SERVICE_SUPPORT_ESIM" ]
}

if (core_service_satellite) {
  telephony_extra_defines += [ "CORE_SERVICE_SATELLITE" ]
}
```

---

## 构建目标详解

### 1. tel_core_service (核心服务)

**文件**: `BUILD.gn:29`

```gn
ohos_shared_library("tel_core_service") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  version_script = "libtel_core_service.versionscript"
  branch_protector_ret = "pac_ret"
  install_enable = true
  
  # 源文件列表 (services/ 目录下)
  sources = [
    "$TELEPHONY_SIM_ROOT/src/sim_manager.cpp",
    "$TELEPHONY_NETWORK_SEARCH_ROOT/src/network_search_manager.cpp",
    "$TELEPHONY_TEL_RIL_ROOT/src/tel_ril_manager.cpp",
    "services/core/src/core_service.cpp",
    # ... 更多源文件
  ]
  
  deps = [
    "interfaces/innerkits:tel_core_service_api",
    "utils:libtel_common",
  ]
  
  external_deps = [
    "ipc:ipc_single",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "hilog:libhilog",
    "drivers_interface_ril:libril_proxy_1.5",
    # ... 更多外部依赖
  ]
  
  part_name = "core_service"
  subsystem_name = "telephony"
}
```

**产物**:
- 输出: `libtel_core_service.z.so`
- 安装路径: `/system/lib64/` 或 `/system/lib/`
- SA 配置: `sa_profile/4010.json`

---

### 2. tel_core_service_api (内部 API)

**文件**: `interfaces/innerkits/BUILD.gn:64`

```gn
ohos_shared_library("tel_core_service_api") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  version_script = "libtel_core_service_api.versionscript"
  branch_protector_ret = "pac_ret"
  
  sources = [
    "frameworks/native/src/core_service_client.cpp",
    "frameworks/native/src/core_service_proxy.cpp",
    "frameworks/native/src/telephony_state_registry_client.cpp",
    # ... 更多源文件
  ]
  
  external_deps = [
    "ipc:ipc_single",
    "samgr:samgr_proxy",
    "eventhandler:libeventhandler",
    # ...
  ]
  
  innerapi_tags = [ "platformsdk" ]
  part_name = "core_service"
  subsystem_name = "telephony"
}
```

**产物**:
- 输出: `libtel_core_service_api.z.so`
- 安装路径: `/system/lib64/` 或 `/system/lib/`

---

### 3. SIM N-API (sim.z.so)

**文件**: `frameworks/js/sim/BUILD.gn:16`

```gn
ohos_shared_library("sim") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "frameworks/js/napi/napi_util.cpp",
    "src/napi_sim.cpp",
  ]
  
  deps = [
    "interfaces/innerkits:tel_core_service_api",
    "utils:libtel_common",
  ]
  
  external_deps = [
    "napi:ace_napi",
    "ipc:ipc_single",
    "hilog:libhilog",
  ]
  
  relative_install_dir = "module/telephony"
}
```

**产物**:
- 输出: `sim.z.so`
- 安装路径: `/system/lib64/module/telephony/` 或 `/system/lib/module/telephony/`
- JS 模块名: `telephony.sim`

---

### 4. Radio N-API (radio.z.so)

**文件**: `frameworks/js/network_search/BUILD.gn:17`

```gn
ohos_shared_library("radio") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "frameworks/js/napi/napi_util.cpp",
    "src/napi_radio.cpp",
    "src/get_radio_state_callback.cpp",
    "src/set_radio_state_callback.cpp",
    # ... 更多回调文件
  ]
  
  deps = [
    "interfaces/innerkits:tel_core_service_api",
    "utils:libtel_common",
  ]
  
  external_deps = [
    "napi:ace_napi",
    "ipc:ipc_single",
    "hilog:libhilog",
  ]
  
  relative_install_dir = "module/telephony"
}
```

**产物**:
- 输出: `radio.z.so`
- 安装路径: `/system/lib64/module/telephony/` 或 `/system/lib/module/telephony/`
- JS 模块名: `telephony.radio`

---

### 5. 工具库

#### libtel_common

**文件**: `utils/BUILD.gn`

```gn
ohos_shared_library("libtel_common") {
  sources = [
    "common/src/telephony_permission.cpp",
    "common/src/telephony_config.cpp",
    "common/src/tel_event_handler.cpp",
    # ...
  ]
  
  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libprivacy_sdk",
    "hilog:libhilog",
    "ipc:ipc_single",
    # ...
  ]
}
```

**产物**:
- 输出: `libtel_common.z.so`

#### libtel_vcard (eSIM)

```gn
ohos_shared_library("libtel_vcard") {
  sources = [
    "vcard/src/vcard_manager.cpp",
    "vcard/src/vcard_decoder.cpp",
    # ...
  ]
}
```

**产物**:
- 输出: `libtel_vcard.z.so`

---

## 产物清单

| 产物名称 | 类型 | 安装路径 | 说明 |
|----------|------|----------|------|
| `libtel_core_service.z.so` | Shared Library | /system/lib*/ | 核心服务库 |
| `libtel_core_service_api.z.so` | Shared Library | /system/lib*/ | 内部 API 库 |
| `libtel_common.z.so` | Shared Library | /system/lib*/ | 通用工具库 |
| `libtel_vcard.z.so` | Shared Library | /system/lib*/ | vCard 工具库 |
| `sim.z.so` | N-API Module | /system/lib*/module/telephony/ | SIM JS API |
| `radio.z.so` | N-API Module | /system/lib*/module/telephony/ | Radio JS API |
| `esim.z.so` | N-API Module | /system/lib*/module/telephony/ | eSIM JS API |
| `vcard.z.so` | N-API Module | /system/lib*/module/telephony/ | VCard JS API |
| `telephonyres.hap` | HAP | /system/app/telephonyres/ | 资源包 |

---

## 运行时加载关系

```
Application (JS)
├── sim.z.so ──┬──→ libtel_core_service_api.z.so
│              └──→ libtel_common.z.so
├── radio.z.so ──┬──→ libtel_core_service_api.z.so
│                └──→ libtel_common.z.so
└── esim.z.so ──┬──→ libtel_core_service_api.z.so
                └──→ libtel_common.z.so

telephony process (SA:4010)
└── libtel_core_service.z.so ──┬──→ libtel_core_service_api.z.so
                               ├──→ libtel_common.z.so
                               ├──→ libril_proxy_*.z.so (HDI)
                               └──→ libtel_vcard.z.so (if eSIM enabled)
```

---

## 安全编译选项

### Sanitizer 配置

```gn
sanitize = {
  cfi = true              # 控制流完整性
  cfi_cross_dso = true    # 跨 DSO 的 CFI
  debug = false
}
branch_protector_ret = "pac_ret"  # 返回地址保护
```

### 编译定义

```gn
defines = [
  "LOG_TAG = \"CoreService\"",
  "LOG_DOMAIN = 0xD001F04",
  "OPENSSL_SUPPRESS_DEPRECATED",
]
```

---

## bundle.json 构建配置

**文件**: `bundle.json:81-156`

```json
{
  "build": {
    "group_type": {
      "base_group": [
        "//base/telephony/core_service/test/mock/ffrt:ffrt_mocked",
        "//base/telephony/core_service/interfaces/kits/c/telephony_radio:telephony_radio"
      ],
      "fwk_group": [
        "//base/telephony/core_service/interfaces/innerkits:tel_core_service_api",
        "//base/telephony/core_service/frameworks/js/network_search:radio",
        "//base/telephony/core_service/frameworks/js/sim:sim",
        "//base/telephony/core_service/frameworks/js/vcard:vcard"
      ],
      "service_group": [
        "//base/telephony/core_service:tel_core_service",
        "//base/telephony/core_service/sa_profile:core_service_sa_profile"
      ]
    },
    "inner_kits": [
      {
        "header": {
          "header_base": "//base/telephony/core_service/interfaces/innerkits",
          "header_files": []
        },
        "name": "//base/telephony/core_service/interfaces/innerkits:tel_core_service_api"
      }
    ]
  }
}
```

---

## 相关链接

- [目录结构](./01_Directory_Structure.md)
- [内部 API](./04_Inner_API.md)
- [安全风险](./06_Security.md)
