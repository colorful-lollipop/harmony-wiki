# T2Stack 构建系统

## 目录

- [1. 构建系统概述](#1-构建系统概述)
- [2. Feature Flags](#2-feature-flags)
- [3. 关键 Targets 列表](#3-关键-targets-列表)
- [4. 依赖关系](#4-依赖关系)
- [5. 构建配置](#5-构建配置)
- [6. Sanitizer 与安全编译](#6-sanitizer-与安全编译)

---

## 1. 构建系统概述

T2Stack 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。

### 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口，定义主 group |
| `t2stack.gni` | 全局配置参数（Feature Flags） |
| `bundle.json` | Bundle 元数据配置 |

### 构建目标

```
t2stack/
├── BUILD.gn                    # 根入口 → group("nstackx")
├── fillp/BUILD.gn             # → FillpSo.open
├── nstackx_congestion/BUILD.gn # → nstackx_congestion.open
├── nstackx_core/dfile/BUILD.gn # → nstackx_dfile.open
├── nstackx_ctrl/BUILD.gn       # → nstackx_ctrl
└── nstackx_util/BUILD.gn       # → nstackx_util.open
```

---

## 2. Feature Flags

### 全局配置 (t2stack.gni)

```gn
declare_args() {
  t2stack_feature_vtp = true           # VTP/Fillp 协议支持
  t2stack_feature_dfile = true          # DFile 文件传输
  t2stack_feature_coap = true           # CoAP 协议
  t2stack_feature_inner_coap = true     # 内部 CoAP 实现
  t2stack_feature_deps_wifi = true      # WiFi 依赖
}
```

### Feature 条件逻辑

```gn
# t2stack.gni:27-30
if (!t2stack_feature_coap || !t2stack_feature_deps_wifi) {
  t2stack_feature_inner_coap = false
} else {
  t2stack_feature_inner_coap = true
}
```

### Feature 与 Target 对应关系

| Feature | 相关 Target | 默认值 |
|---------|-------------|--------|
| `t2stack_feature_vtp` | `FillpSo.open` | true |
| `t2stack_feature_dfile` | `nstackx_dfile.open` | true |
| `t2stack_feature_inner_coap` | `nstackx_ctrl` | true |

---

## 3. 关键 Targets 列表

### 3.1 Fillp 模块

**BUILD.gn**: `fillp/BUILD.gn`

```gn
ohos_shared_library("FillpSo.open") {
  # 条件编译
  if (t2stack_feature_vtp) {
    sources += [
      "src/app_lib/src/api.c",
      "src/fillp_lib/src/fillp/fillp.c",
      # ... 更多源文件
    ]
  }

  # 编译标志
  cflags = [
    "-DPDT_MIRACAST",
    "-DFILLP_SERVER_SUPPORT",
    "-DFILLP_LINUX",
    "-DFILLP_POWER_SAVE",
  ]

  # 包含目录
  include_dirs = [
    "include",
    "src/app_lib/include",
    "src/fillp_lib/include",
    "src/public/include",
  ]

  # 依赖
  deps = [
    "$T2STACK_ROOT_PATH/nstackx_util:nstackx_util.open",
  ]
  external_deps = [
    "bounds_checking_function:libsec_shared",
  ]

  # 安全配置
  sanitize = {
    ubsan = true
    integer_overflow = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
  }
  branch_protector_ret = "pac_ret"
}
```

### 3.2 DFile 模块

**BUILD.gn**: `nstackx_core/dfile/BUILD.gn`

```gn
ohos_shared_library("nstackx_dfile.open") {
  cflags = [
    "-DNSTACKX_WITH_LINUX",
    "-DDFILE_ENABLE_HIDUMP",
    "-DENABLE_USER_LOG",
    "-DSSL_AND_CRYPTO_INCLUDED",
  ]

  if (t2stack_feature_dfile) {
    sources += [
      "core/nstackx_dfile.c",
      "core/nstackx_dfile_session.c",
      "core/nstackx_dfile_send.c",
      # ... 更多源文件
    ]
  }

  deps = [
    "$NSTACKX_ROOT/nstackx_congestion:nstackx_congestion.open",
    "$NSTACKX_ROOT/nstackx_util:nstackx_util.open",
  ]

  external_deps = [
    "bounds_checking_function:libsec_shared",
    "openssl:libcrypto_shared",  # Standard 系统
  ]
}
```

### 3.3 NStackX Ctrl 模块

**BUILD.gn**: `nstackx_ctrl/BUILD.gn`

```gn
ohos_shared_library("nstackx_ctrl") {
  if (t2stack_feature_inner_coap) {
    sources = base_src
    sources += standard_small_diff_src

    deps = [
      "../nstackx_util:nstackx_util.open",
    ]

    external_deps = [
      "bounds_checking_function:libsec_shared",
      "cJSON:cjson",
      "libcoap:libcoap",
    ]
  }

  # 安全链接标志
  ldflags = [
    "-Wl,-z,relro,-z,now",
    "-s",
    "-fPIC",
  ]
}
```

### 3.4 NStackX Util 模块

**BUILD.gn**: `nstackx_util/BUILD.gn`

```gn
ohos_shared_library("nstackx_util.open") {
  sources = [
    "core/nstackx_dev.c",
    "core/nstackx_event.c",
    "core/nstackx_log.c",
    "core/nstackx_socket.c",
    "core/nstackx_timer.c",
    "core/nstackx_util.c",
    "platform/unix/sys_dev.c",
    "platform/unix/sys_epoll.c",
    # ... 更多文件
  ]

  external_deps = [
    "bounds_checking_function:libsec_shared",
    "hilog:libhilog",
  ]

  if (is_standard_system) {
    sources += [ "core/nstackx_openssl.c" ]
    external_deps += [
      "c_utils:utils",
      "openssl:libcrypto_shared",
    ]
  }
}
```

---

## 4. 依赖关系

### 内部依赖

```
nstackx_util.open
    ↑
    ├── FillpSo.open
    ├── nstackx_ctrl
    └── nstackx_dfile.open
            ↑
            └── nstackx_congestion.open
```

### 外部依赖

| Target | 外部依赖 | 用途 |
|--------|----------|------|
| **FillpSo.open** | `bounds_checking_function:libsec_shared` | 安全函数 |
| **nstackx_dfile.open** | `openssl:libcrypto_shared` / `mbedtls:mbedtls_shared` | 加密 |
| **nstackx_ctrl** | `libcoap:libcoap`、`cJSON:cjson` | CoAP 协议 |
| **nstackx_util.open** | `hilog:libhilog`、`c_utils:utils` | 日志、工具 |

---

## 5. 构建配置

### 5.1 平台条件编译

```gn
# LiteOS 分支
if (defined(ohos_lite) && ohos_kernel_type == "liteos_a") {
  cflags += [ "-DNSTACKX_WITH_LITEOS" ]
} else if (ohos_kernel_type == "linux") {
  cflags += [ "-DNSTACKX_WITH_LINUX" ]
}
```

### 5.2 系统类型配置

```gn
# Standard 系统特有配置
if (is_standard_system) {
  external_deps += [ "c_utils:utils" ]
  sources += [ "core/nstackx_openssl.c" ]
}
```

---

## 6. Sanitizer 与安全编译

### 启用的安全特性

| 特性 | 配置 | 说明 |
|------|------|------|
| **UBSan** | `ubsan = true` | 未定义行为检测 |
| **Integer Overflow** | `integer_overflow = true` | 整数溢出检测 |
| **Boundary Sanitize** | `boundary_sanitize = true` | 边界检查 |
| **CFI** | `cfi = true`、`cfi_cross_dso = true` | 控制流完整性 |
| **PAC/Ret** | `branch_protector_ret = "pac_ret"` | 指针认证与返回地址保护 |
| **RELRO** | `-Wl,-z,relro,-z,now` | 只读重定位 |

### 安全编译标志

```gn
# 编译器标志
cflags = [
  "-Wall",
  "-fPIC",
  "-fno-unwind-tables",
  "-Os",
]

# 链接器标志
ldflags = [
  "-Wl,-z,relro,-z,now",  # Full RELRO
  "-s",                    # Strip symbols
]
```

---

## 相关文档

- [模块结构](./02_Module_Structure.md) - 模块职责
- [运行时产物](./06_Runtime_Artifacts.md) - 编译产物说明
- [安全评审](./07_Security_Review.md) - 安全风险分析

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
*代码证据来源：BUILD.gn 文件分析*
