# GN 构建配置

本文档描述分布式硬件管理框架的 GN 构建系统配置，包括 Targets 清单、依赖关系和编译开关。

> **适用范围**: 需要理解构建系统、添加新模块或修改编译配置的开发者

---

## 概述

**构建系统**: GN (Generate Ninja)
**配置文件**: `.gni` 和 `BUILD.gn`
**模块配置**: `bundle.json`

---

## 关键配置文件

### 根配置 (distributedhardwarefwk.gni)

**路径**: `distributedhardwarefwk.gni`

**关键定义**:

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `distributedhardwarefwk_path` | `//foundation/distributedhardware/distributed_hardware_fwk` | 框架根路径 |
| `common_path` | `${distributedhardwarefwk_path}/common` | 公共路径 |
| `utils_path` | `${distributedhardwarefwk_path}/utils` | 工具路径 |
| `services_path` | `${distributedhardwarefwk_path}/services` | 服务路径 |
| `innerkits_path` | `${distributedhardwarefwk_path}/interfaces/inner_kits` | Inner Kit 路径 |
| `av_trans_path` | `${distributedhardwarefwk_path}/av_transport` | AV 传输路径 |

**编译开关**:

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `distributed_hardware_fwk_low_latency` | `false` | 低延迟模式开关 |
| `dhfwk_os_account` | 根据 `global_parts_info` 确定 | OS 账户支持 |

**证据**: `distributedhardwarefwk.gni:14-49`
```gni
distributedhardwarefwk_path =
    "//foundation/distributedhardware/distributed_hardware_fwk"

common_path = "${distributedhardwarefwk_path}/common"
utils_path = "${distributedhardwarefwk_path}/utils"
services_path = "${distributedhardwarefwk_path}/services"
innerkits_path = "${distributedhardwarefwk_path}/interfaces/inner_kits"
av_trans_path = "${distributedhardwarefwk_path}/av_transport"

declare_args() {
  distributed_hardware_fwk_low_latency = false
}

if (!defined(global_parts_info) ||
      defined(global_parts_info.account_os_account)) {
    dhfwk_os_account = true
  } else {
    dhfwk_os_account = false
  }
```

---

## Targets 清单

### 核心服务 Targets

| Target 名称 | 类型 | 路径 | 依赖 | 输出 |
|-------------|------|------|------|------|
| `distributedhardwarefwksvr` | `ohos_shared_library` | `services/distributedhardwarefwkservice/` | `distributedhardwareutils`, `dhfwk_idl_hardware_source` | `libdistributedhardwarefwksvr.so` |
| `distributedhardwareutils` | `ohos_shared_library` | `utils/` | cJSON, device_manager | `libdistributedhardwareutils.so` |
| `libdhfwk_sdk` | `ohos_shared_library` | `interfaces/inner_kits/` | `distributedhardwarefwksvr` | `libdhfwk_sdk.so` |

**证据**: `bundle.json:78`
```json
"//foundation/.../services/distributedhardwarefwkservice:distributedhardwarefwksvr"
```

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:19`
```gn
ohos_shared_library("distributedhardwarefwksvr") {
  include_dirs = [
    "include",
    "${innerkits_path}/include",
    ...
  ]
  sources = [
    "${av_center_svc_path}/src/av_sync_manager.cpp",
    "src/accessmanager/access_manager.cpp",
    "src/componentmanager/component_manager.cpp",
    ...
  ]
  deps = [
    "${utils_path}:distributedhardwareutils",
    "${innerkits_path}:dhfwk_idl_hardware_source",
  ]
}
```

---

### AV 传输 Targets

| Target 名称 | 类型 | 路径 | 依赖 |
|-------------|------|------|------|
| `distributed_av_sender` | `ohos_shared_library` | `av_transport/av_trans_engine/av_sender/` | `libdhfwk_sdk`, `distributed_av_pipeline_fwk` |
| `distributed_av_receiver` | `ohos_shared_library` | `av_transport/av_trans_engine/av_receiver/` | `libdhfwk_sdk`, `distributed_av_pipeline_fwk` |
| `distributed_av_pipeline_fwk` | `ohos_shared_library` | `av_transport/framework/` | `libdhfwk_sdk`, `media_foundation` |
| `histreamer_ability_querier` | `ohos_shared_library` | `av_transport/av_trans_handler/histreamer_ability_querier/` | `media_foundation` |

---

### 接口层 Targets

| Target 名称 | 类型 | 路径 | 依赖 |
|-------------|------|------|------|
| `hardwaremanager` | `ohos_shared_library` | `interfaces/kits/napi/` | `libdhfwk_sdk`, `napi`, `access_token` |
| `hardware_taihe` | `taihe_shared_library` | `taihe/` | `libdhfwk_sdk` |
| `distributed_hardware_taihe` | `group` | `taihe/` | `hardware_taihe`, `hardware_taihe_abc` |

---

### 配置与配置文件 Targets

| Target 名称 | 类型 | 路径 | 输出 |
|-------------|------|------|------|
| `dhfwk_sa_profile` | `ohos_sa_profile` | `sa_profile/` | `4801.json` |
| `dhardware.cfg` | `ohos_prebuilt_etc` | `sa_profile/` | `init/dhardware.cfg` |

**证据**: `sa_profile/BUILD.gn`
```gn
ohos_sa_profile("dhfwk_sa_profile") {
  source = "4801.json"
}

ohos_prebuilt_etc("dhardware.cfg") {
  source = "dhardware.cfg"
  dep_subset =("", "low_latency") {
    source = "close_source/dhardware.cfg"
  }
  relative_install_dir = "init"
}
```

---

### 应用 Targets

| Target 名称 | 类型 | 路径 | 说明 |
|-------------|------|------|------|
| `DHardware_UI` | `ohos_app` | `application/` | 系统应用 HAP |
| `DHardware_UI_js_assets` | `ohos_js_assets` | `application/` | JS 资产 |
| `DHardware_UI_resources` | `ohos_resources` | `application/` | 资源文件 |

---

## 依赖配置

### 外部依赖 (external_deps)

| 依赖组件 | 用途 |
|----------|------|
| `ability_runtime` | Ability 管理 |
| `access_token` | 权限管理 |
| `bundle_framework` | Bundle 框架 |
| `dsoftbus` | 软总线通信 |
| `device_manager` | 设备管理 |
| `kv_store` | 分布式数据 |
| `hilog` | 日志 |
| `ipc` | IPC 通信 |
| `safwk` | System Ability 框架 |
| `samgr` | 服务管理 |
| `media_foundation` | 媒体框架 |
| `ffmpeg` | 编解码 |
| `hisysevent` | 事件追踪 |

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:158-181`
```gn
external_deps = [
  "ability_runtime:ability_manager",
  "access_token:libaccesstoken_sdk",
  "dsoftbus:softbus_client",
  "device_manager:devicemanagersdk",
  "hilog:libhilog",
  "ipc:ipc_core",
  "kv_store:distributeddata_inner",
  "media_foundation:media_foundation",
  ...
]
```

---

### 条件依赖

| 条件 | 依赖 | 说明 |
|------|------|------|
| `dhfwk_os_account` | `os_account:libaccountkits` | OS 账户支持 |
| `distributed_hardware_fwk_low_latency` | `libevdev` | 低延迟模式 |

---

## 安全加固配置

### 编译器标志 (cflags)

所有共享库均启用以下安全标志：

| 标志 | 说明 |
|------|------|
| `-fstack-protector-strong` | 堆栈保护 |
| `-D_FORTIFY_SOURCE=2` | 运行时边界检查 |
| `-O2` | 优化级别 |
| `-fpie` | 位置无关可执行文件 |

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:132-138`
```gn
cflags = [
  "-fstack-protector-strong",
  "-D_FORTIFY_SOURCE=2",
  "-O2",
]
```

### 链接标志 (ldflags)

| 标志 | 说明 |
|------|------|
| `-fpie` | 位置无关代码 |
| `-Wl,-z,relro` | 只读重定位 |
| `-Wl,-z,now` | 立即绑定符号 |

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:140-144`
```gn
ldflags = [
  "-fpie",
  "-Wl,-z,relro",
  "-Wl,-z,now",
]
```

---

###  sanitizer 配置

| sanitizer | 说明 |
|-----------|------|
| `boundary_sanitize` | 边界检查 |
| `integer_overflow` | 整数溢出检测 |
| `ubsan` | 未定义行为检测 |
| `cfi` | 控制流完整性 |
| `cfi_cross_dso` | 跨 DSO 控制流完整性 |

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:20-27`
```gn
sanitize = {
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true
  cfi = true
  cfi_cross_dso = true
  debug = false
}
branch_protector_ret = "pac_ret"
```

---

## 条件编译

### 低延迟模式

```gn
if (distributed_hardware_fwk_low_latency) {
  defines += [ "DHARDWARE_LOW_LATENCY" ]
} else {
  defines += [ "DHARDWARE_OPEN_SOURCE" ]
}

if (distributed_hardware_fwk_low_latency) {
  defines += [ "DHARDWARE_CHECK_RESOURCE" ]
}
```

**证据**: `services/distributedhardwarefwkservice/BUILD.gn:146-156`
```gn
if (distributed_hardware_fwk_low_latency) {
  defines += [ "DHARDWARE_LOW_LATENCY" ]
}

if (!distributed_hardware_fwk_low_latency) {
  defines += [ "DHARDWARE_OPEN_SOURCE" ]
}

if (distributed_hardware_fwk_low_latency) {
  defines += [ "DHARDWARE_CHECK_RESOURCE" ]
}
```

---

## 后续文档

- 编译产物说明 → [06_Build_Artifacts.md](06_Build_Artifacts.md)
- 安全评审 → [07_Security_Review.md](07_Security_Review.md)
- 附录配置开关 → [appendix/Config_Flags.md](appendix/Config_Flags.md)
