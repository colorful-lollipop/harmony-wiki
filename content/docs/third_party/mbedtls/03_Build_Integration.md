# OpenHarmony 构建适配

## 构建系统概述

mbedtls 库在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统，通过 `BUILD.gn` 和 `mbedtls.gni` 文件实现完整的构建适配。

### 构建配置结构

```
third_party/mbedtls/
├── BUILD.gn              # 主构建配置
├── mbedtls.gni          # 构建变量定义
├── port/
│   └── BUILD.gn         # Port 层构建配置
└── library/             # mbedtls 源文件
```

## 构建变量定义 (mbedtls.gni)

### 核心变量

```gni
MBEDTLSDIR = "//third_party/mbedtls/"
KERNELDIR = "//kernel/liteos_m/"

# 可配置参数
declare_args() {
  mbedtls_porting_path = ""              # 自定义移植路径
  mbedtls_enable_ssl_srv = false          # 启用 SSL 服务器功能
}
```

### 源文件列表

mbedtls.gni 中定义了完整的源文件列表，包含以下模块：

| 模块类别 | 源文件数量 | 主要文件 |
|----------|-----------|----------|
| **加密原语** | 约 30 个 | aes.c, sha256.c, cipher.c |
| **公钥密码** | 约 15 个 | rsa.c, ecp.c, ecdsa.c |
| **TLS 协议** | 约 20 个 | ssl_tls.c, ssl_tls13_*.c |
| **X.509 证书** | 约 10 个 | x509_crt.c, x509_crl.c |
| **PSA 加密** | 约 15 个 | psa_crypto.c |
| **工具类** | 约 10 个 | platform.c, entropy.c |

### 包含目录

```gni
MBEDTLS_INLCUDE_DIRS = [
  "$MBEDTLSDIR/include",           # 标准头文件目录
  "$MBEDTLSDIR/library",           # 库内部头文件
  "$MBEDTLSDIR/include/mbedtls",   # mbedtls 头文件
  "$MBEDTLSDIR/tests/include",     # 测试头文件
  "$MBEDTLSDIR/port/include",      # OH Port 头文件
]
```

## 主构建配置 (BUILD.gn)

### 构建目标类型

#### 1. 标准系统构建 (ohos_lite = false)

```gn
ohos_shared_library("mbedtls_shared") {
  # 共享库配置
  output_name = "mbedtls"
  subsystem_name = "thirdparty"
  version_script = "libmbedtls.map"
  
  # SSL 服务器功能（可选）
  if (mbedtls_enable_ssl_srv == true) {
    defines += [
      "MBEDTLS_KEY_EXCHANGE_PSK_ENABLED",
      "MBEDTLS_SSL_SRV_C",
    ]
  }
  
  # 依赖配置
  external_deps = [ "bounds_checking_function:libsec_static" ]
  part_name = "mbedtls"
  
  # 安装配置
  install_images = [
    "system",
    "updater",
  ]
}

ohos_static_library("mbedtls_static") {
  sources = MBEDTLS_SOURCES
  public_configs = [ ":mbedtls_config" ]
  external_deps = [ "bounds_checking_function:libsec_static" ]
  part_name = "mbedtls"
  subsystem_name = "thirdparty"
}

group("mbedtls") {
  public_deps = [ ":mbedtls_shared" ]
}
```

#### 2. LiteOS 系统构建 (ohos_lite = true)

```gn
# LiteOS 共享库
lite_library("mbedtls_shared") {
  target_type = "shared_library"
  public_configs = [ ":mbedtls_config" ]
  output_name = "mbedtls"
  sources = MBEDTLS_SOURCES
}

# LiteOS 静态库
lite_library("mbedtls_static") {
  target_type = "static_library"
  if (board_toolchain_type == "clang") {
    cflags = [
      "-Wno-error=parentheses-equality",
      "-Wno-error=implicit-function-declaration",
    ]
  }
  public_configs = [ ":mbedtls_config" ]
  output_name = "mbedtls"
  sources = MBEDTLS_SOURCES
}

group("mbedtls") {
  if (ohos_kernel_type == "liteos_m") {
    if (mbedtls_porting_path != "") {
      public_deps = [ mbedtls_porting_path ]
    } else {
      public_deps = [ ":mbedtls_static" }
  } else {
    public_deps = [ ":mbedtls_shared" ]
  }
}
```

### 配置文件

```gn
config("mbedtls_config") {
  include_dirs = MBEDTLS_INLCUDE_DIRS
}

# LiteOS 特定配置
if (ohos_kernel_type == "liteos_m") {
  config("mbedtls_config") {
    include_dirs += [ "//third_party/bounds_checking_function/include" ]
    defines += [
      "__unix__",
      "MBEDTLS_CONFIG_FILE=<../port/config/config_liteos_m.h>",
    ]
  }
}

if (ohos_kernel_type == "liteos_a") {
  config("mbedtls_config") {
    include_dirs += [ "//third_party/bounds_checking_function/include" ]
    defines += [
      "__unix__",
      "MBEDTLS_CONFIG_FILE=<../port/config/config_liteos_a.h>",
    ]
  }
}
```

### NDK 库构建

```gn
ndk_lib("mbedtls_ndk") {
  if (ohos_kernel_type == "liteos_m") {
    lib_extension = ".a"  # 静态库
  } else {
    lib_extension = ".so"  # 共享库
  }
  deps = [ ":mbedtls" ]
  head_files = [ "include" ]
}
```

### 测试构建

```gn
if (ohos_build_type == "debug" && ohos_kernel_type != "liteos_m") {
  config("mbedtls_profile_test") {
    include_dirs = [
      "./include",
      "./configs",
    ]
    defines = [
      "MBEDTLS_CONFIG_FILE=<config_rsa_aes_cbc.h>",
      "__unix__",
    ]
    ldflags = [
      "-s",
      "-w",
    ]
  }
  
  static_library("mbedtls_gt") {
    sources = MBEDTLS_SOURCES
    output_name = "mbedtls_gt"
    public_configs = [ ":mbedtls_profile_test" ]
  }
}
```

## Port 层构建配置 (port/BUILD.gn)

```gn
if (ohos_build_type == "debug") {
  config("tls_client") {
    include_dirs = [
      "./include",
      "//third_party/mbedtls/include",
      "//commonlibrary/utils_lite/include",
      "//third_party/bounds_checking_function/include",
      "//third_party/mbedtls/port/include",
    ]
  }
  
  tls_client_src = [
    "src/tls_certificate.c",
    "src/tls_client.c",
  ]
  
  static_library("tls_client_static") {
    sources = tls_client_src
    deps = [
      "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
      "//third_party/bounds_checking_function:libsec_shared",
      "//third_party/mbedtls:mbedtls_gt",
    ]
    public_configs = [ ":tls_client" ]
  }
}
```

## 编译选项详解

### 预处理器定义

| 宏定义 | 用途 | 设置位置 |
|--------|------|----------|
| `__unix__` | Unix 环境标识 | config_liteos_*.h |
| `MBEDTLS_CONFIG_FILE` | 指定配置文件 | BUILD.gn |
| `MBEDTLS_KEY_EXCHANGE_PSK_ENABLED` | 启用 PSK 密钥交换 | BUILD.gn (可选) |
| `MBEDTLS_SSL_SRV_C` | 启用 SSL 服务器功能 | BUILD.gn (可选) |

### 包含目录优先级

1. 标准头文件: `include/`, `include/mbedtls/`
2. Port 头文件: `port/include/`
3. 兼容性配置: `port/config/compat_*/`
4. 边界检查: `third_party/bounds_checking_function/include`

### 条件编译逻辑

```gn
# 内核类型判断
if (ohos_kernel_type == "liteos_m") {
  # LiteOS-M 特定配置
}

if (ohos_kernel_type == "liteos_a") {
  # LiteOS-A 特定配置
}

# 产品类型判断
if (product_name == "xxx") {
  # 特定产品配置
}
```

## 构建产物

### 标准系统

| 构建目标 | 输出文件 | 类型 |
|----------|----------|------|
| `mbedtls_shared` | libmbedtls.so | 共享库 |
| `mbedtls_static` | libmbedtls.a | 静态库 |
| `mbedtls_ndk` | libmbedtls.so/.a | NDK 库 |

### LiteOS-M

| 构建目标 | 输出文件 | 类型 |
|----------|----------|------|
| `mbedtls_shared` | libmbedtls.so | 共享库 |
| `mbedtls_static` | libmbedtls.a | 静态库 |
| `mbedtls_ndk` | libmbedtls.a | 静态库 |

### 安装路径

- **系统库**: `system/lib/`
- **NDK 库**: `ndk/`
- **更新模块**: `updater/`

## 构建变体

### Debug 构建

```bash
# 启用调试信息
hb build -b debug
```

### Release 构建

```bash
# 优化构建
hb build -b release
```

### 自定义配置

```bash
# 启用 SSL 服务器
hb set # 选择 mbedtls_enable_ssl_srv
hb build
```

## 依赖关系

### 内部依赖

```gn
# mbedtls 内部模块依赖
external_deps = [ "bounds_checking_function:libsec_static" ]
```

### 外部依赖

| 依赖项 | 用途 | 必需 |
|--------|------|------|
| `bounds_checking_function` | 边界安全检查 | 是 |
| `hilog_lite` | 日志系统 | 可选 (调试时) |

## 常见构建问题

### 1. 头文件找不到

**症状**: `fatal error: 'xxx.h' file not found`

**解决方案**: 检查 `MBEDTLS_INLCUDE_DIRS` 配置

### 2. 符号未定义

**症状**: `undefined reference to 'xxx'`

**解决方案**: 确保链接了正确的 mbedtls 库变体

### 3. 配置冲突

**症状**: `multiple definition of 'xxx'`

**解决方案**: 检查是否同时链接了静态库和共享库
