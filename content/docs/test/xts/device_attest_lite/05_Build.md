# GN 构建配置

## 5.1 构建入口

### 根 BUILD.gn

```gn
# 文件: BUILD.gn:16-27

lite_component("device_atTest_lite") {
  features = []
  if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
    features += [
      "framework:devattest_service",
      "interfaces/kit/js:kit_device_attest",
      "test/startup:devattest_client",
    ]
  } else if (ohos_kernel_type == "liteos_m") {
    features += [ "framework:devattest_sdk" ]
  }
}
```

**证据**: `BUILD.gn:16-27`

---

## 5.2 Targets 清单

### Framework Layer Targets

| Target | 类型 | 平台 | Sources | 产物 |
|--------|------|------|---------|------|
| `devattest_server` | shared_library | small | `service/attest_framework_feature.c`, `service/attest_framework_server.c` | `libdevattest_server.so` |
| `devattest_client` | shared_library | small | `client/attest_framework_client_proxy.c` | `libdevattest_client.so` |
| `devattest_service` | executable | small | `service/attest_framework_service.c` | `devattest_service` |
| `devattest_client` (mini) | group | mini | 空 | 空 |
| `devattest_sdk` | static_library | mini | `mini/src/attest_framework_client_mini.c` | `libdevattest_sdk.a` |

**证据**: `framework/BUILD.gn:17-103`

### JS Kit Target

| Target | 类型 | Sources | 产物 |
|--------|------|---------|------|
| `kit_device_attest` | shared_library | `src/native_device_attest.cpp` | `libkit_device_attest.so` |

**证据**: `interfaces/kit/js/BUILD.gn:17-46`

### Core Layer Targets

| Target | 类型 | 平台 | Sources | 产物 |
|--------|------|------|---------|------|
| `devattest_core` | static_library | mini | `sources_common` + mini 适配 | `libdevattest_core.a` |
| `devattest_core` | shared_library | small | `sources_common` + small 适配 | `libdevattest_core.so` |

**证据**: `services/core/BUILD.gn:75-136`

---

## 5.3 关键 Targets 详细配置

### kit_device_attest

```gn
# 文件: interfaces/kit/js/BUILD.gn:17-46

shared_library("kit_device_attest") {
  sources = [ "src/native_device_attest.cpp" ]
  cflags = [
    "-ftrapv",
    "-Werror",
    "-Wextra",
    "-Wshadow",
    "-fstack-protector-all",
    "-Wformat=2",
    "-Wfloat-equal",
    "-Wdate-time",
    "-fPIC",
    "-pthread",
  ]
  include_dirs = [
    "include",
    "${devattest_path}/common",
    "${devattest_path}/common/log",
    "${devattest_path}/interfaces/innerkits",
    "${devattest_path}/framework/small/include",
    "//third_party/bounds_checking_function/include",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/base",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/jsi",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/async",
  ]
  deps = [
    "${devattest_path}/framework:devattest_client",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
  ]
}
```

### devattest_server (small)

```gn
# 文件: framework/BUILD.gn:42-57

shared_library("devattest_server") {
  sources = [
    "small/src/service/attest_framework_feature.c",
    "small/src/service/attest_framework_server.c",
  ]
  cflags = CFLAGS_COMMON
  cflags += [ "-fPIC" ]
  ldflags = [ "-pthread" ]
  include_dirs = INCLUDE_COMMON
  include_dirs += [ "${devattest_path}/services/core" ]
  deps = [
    "${devattest_path}/services/core:devattest_core",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]
}
```

### devattest_core (small)

```gn
# 文件: services/core/BUILD.gn:101-136

shared_library("devattest_core") {
  sources = sources_common
  sources += sources_mock
  sources += [
    "small/adapter/attest_adapter_network_config.c",
    "small/attest/attest_service_pcid.c",
    "small/utils/attest_utils_file_detail.c",
  ]

  public_configs = [
    ":devattest_core_config",
    ":devattest_core_mini_config",
  ]
  cflags = [
    "-ftrapv",
    "-Wextra",
    "-Wshadow",
    "-Wformat=2",
    "-Wfloat-equal",
    "-Wdate-time",
    "-fPIE",
  ]

  deps = [
    "$ohos_product_adapter_dir/utils/token:haltoken_shared",
    "//base/startup/init/interfaces/innerkits:parameter",
  ]
  deps += [
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//build/lite/config/component/cJSON:cjson_shared",
    "//third_party/mbedtls:mbedtls",
  ]
  deps += [ "//developtools/syscap_codec:syscap_interface_shared" ]
}
```

---

## 5.4 编译特性开关

### devattestconfig.gni

```gn
# 文件: build/devattestconfig.gni:21-67

declare_args() {
  attest_build_target = attest_release    # 构建类型

  # 调试开关
  enable_attest_test_mock_network = false   # 模拟网络认证数据
  enable_attest_test_mock_device = false    # 模拟设备数据
  enable_attest_debug_memory_leak = false   # 内存泄漏检测
  enable_attest_sample = false              # 编译示例
  enable_attest_debug_dfx = false          # DFX 调试
  integrate_attest_mini_module = true       # 集成轻量设备模块
  disable_attest_active_site = false        # 关闭域名增强
  enable_attest_preset_token = false       # Token 预置方案
  enable_attest_log_debug = false          # 调试日志
}
```

### Conditional Defines

| 开关 | Define | 用途 |
|------|--------|------|
| enable_attest_log_debug | `__ATTEST_HILOG_LEVEL_DEBUG__` | 调试日志 |
| enable_attest_test_mock_network | `__ATTEST_MOCK_NETWORK_STUB__` | 模拟网络 |
| enable_attest_test_mock_device | `__ATTEST_MOCK_DEVICE_STUB__` | 模拟设备 |
| enable_attest_debug_memory_leak | `__ATTEST_DEBUG_MEMORY_LEAK__` | 内存调试 |
| enable_attest_debug_dfx | `__ATTEST_DEBUG_DFX__` | DFX 调试 |
| disable_attest_active_site | `__ATTEST_DISABLE_SITE__` | 禁用站点 |

**证据**: `services/core/BUILD.gn:26-54`

---

## 5.5 编译产物

### Small System 产物

| 产物 | 类型 | 路径 |
|------|------|------|
| `libkit_device_attest.so` | shared_library | `out/.../interfaces/kit/js/` |
| `libdevattest_server.so` | shared_library | `out/.../framework/` |
| `libdevattest_client.so` | shared_library | `out/.../framework/` |
| `libdevattest_core.so` | shared_library | `out/.../services/core/` |
| `devattest_service` | executable | `out/.../framework/`

### Mini System 产物

| 产物 | 类型 | 路径 |
|------|------|------|
| `libdevattest_sdk.a` | static_library | `out/.../framework/` |
| `libdevattest_core.a` | static_library | `out/.../services/core/` |

---

## 5.6 运行时加载关系

### Small System

```
应用进程加载:
    libace_engine_lite.so
         ↓
    libkit_device_attest.so (JS 绑定)
         ↓
    libdevattest_client.so (IPC 客户端)
         ↓ IPC (SAMgr)
    libdevattest_server.so (SA Feature)
         ↓
    libdevattest_core.so (核心业务)
         ↓
    libmbedtls.so, libcjson.so, libhal_token.so
```

### Mini System

```
应用进程加载:
    libace_engine_lite.so
         ↓
    libdevattest_sdk.a (静态链接)
         ↓
    libdevattest_core.a (静态链接)
         ↓
    libmbedtls.a, libcjson.a, libhal_token.a
```

---

## 5.7 构建命令

### 编译 XTS 模块

```bash
hb set
hb build --gn-args build_xts=true
```

### 启用调试日志

```bash
hb build --gn-args build_xts=true enable_attest_log_debug=true
```

### 启用模拟模式

```bash
hb build --gn-args build_xts=true enable_attest_test_mock_network=true
```

---

## 相关跳转

- [02_Architecture](02_Architecture.md) - 架构说明
- [04_InnerAPI](04_InnerAPI.md) - Inner API 说明
