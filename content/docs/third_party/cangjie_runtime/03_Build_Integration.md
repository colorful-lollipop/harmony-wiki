# 03 - OH 构建适配

## 3.1 BUILD.gn 结构

### 根目录 BUILD.gn

**文件位置**: `//third_party/cangjie_runtime/BUILD.gn`

这是 OH 构建系统的入口文件，负责引入预编译的仓颉运行时和标准库。

```gn
# Copyright (c) 2025 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ...

import("//build/ohos.gni")
import("platform.gni")

# 定义预编译库模板
template("cangjie_prebuilts") {
  _deps = []
  forward_variables_from(invoker, [...])
  
  # 1. 标准预编译库（安装到 platformsdk/cjsdk）
  foreach(lib_name, prebuilt_libs) {
    ohos_prebuilt_shared_library("lib${lib_name}") {
      source = "${prebuilt_libs_path}/lib${lib_name}.${cj_config.dyn_extension}"
      enable_strip = true
      install_enable = true
      part_name = "cangjie_runtime"
      subsystem_name = "thirdparty"
      relative_install_dir = "platformsdk/cjsdk"
      install_images = [ "system" ]
    }
    _deps += [ ":lib${lib_name}" ]
  }
  
  # 2. 特殊库（安装到 chipset-sdk）
  foreach(lib_name, special_libs) { ... }
  
  # 3. 运行时库（安装到 platformsdk/cjsdk/runtime）
  foreach(lib_name, runtime_libs) { ... }
  
  # 4. 全名预编译库（PCRE2）
  foreach(full_lib_name, fullname_prebuilt_libs) { ... }
  
  # 5. 汇总组
  group(target_name) {
    deps = _deps
  }
}

# 实际调用
cangjie_prebuilts("cangjie_prebuilts_package") {
  prebuilt_libs_path = "${prebults_cangjie_sdk_linux_x86_64_path}/runtime/lib/${cj_config.platform_target}"
  prebuilt_libs = [ ... 31个标准库 ... ]
  special_libs = [ "cangjie-demangle" ]
  runtime_libs = [ "cangjie-runtime", "boundscheck" ]
  fullname_prebuilt_libs = [ "libpcre2-8.so", ... ]
}
```

### 预编译库清单

#### 1. 标准库模块（31个，platformsdk/cjsdk）

```
cangjie-std-argopt
cangjie-std-binary
cangjie-std-collection.concurrent
cangjie-std-collection
cangjie-std-console
cangjie-std-convert
cangjie-std-core
cangjie-std-crypto.cipher
cangjie-std-crypto.digest
cangjie-std-crypto
cangjie-std-database
cangjie-std-database.sql
cangjie-std-env
cangjie-std-fs
cangjie-std-interop
cangjie-std-io
cangjie-std-math.numeric
cangjie-std-math
cangjie-std-net
cangjie-std-objectpool
cangjie-std-overflow
cangjie-std-posix
cangjie-std-process
cangjie-std-random
cangjie-std-ref
cangjie-std-reflect
cangjie-std-regex
cangjie-std-runtime
cangjie-std-sort
cangjie-std-sync
cangjie-std-time
cangjie-std
cangjie-std-unicode
```

#### 2. 运行时库（2个，platformsdk/cjsdk/runtime）

```
cangjie-runtime      # 运行时核心
boundscheck          # 边界检查
```

#### 3. 特殊库（1个，chipset-sdk）

```
cangjie-demangle     # 符号反混淆，供底层使用
```

#### 4. PCRE2 库（3个，platformsdk/cjsdk）

```
libpcre2-8.so
libpcre2-8.so.0
libpcre2-8.so.0.14.0
```

### 库分类逻辑

| 类别 | 安装路径 | 用途 |
|-----|---------|------|
| prebuilt_libs | `platformsdk/cjsdk` | 标准库，供应用开发 |
| runtime_libs | `platformsdk/cjsdk/runtime` | 运行时核心 |
| special_libs | `chipset-sdk` | 底层工具库 |
| fullname_prebuilt_libs | `platformsdk/cjsdk` | 第三方依赖（PCRE2） |

---

## 3.2 平台配置（platform.gni）

**文件位置**: `//third_party/cangjie_runtime/platform.gni`

```gn
import("//build/ohos.gni")

# 预编译 SDK 路径配置
if (is_asan && use_hwasan) {
  prebults_cangjie_sdk_linux_x86_64_path = 
      "//prebuilts/cangjie_sdk/linux-x64-hwasan/cangjie"
} else {
  prebults_cangjie_sdk_linux_x86_64_path =
      "//prebuilts/cangjie_sdk/linux-x64/cangjie"
}

# 仓颉配置结构
cj_config = {
  platform_target = ""
  dyn_extension = "so"
}

# OH 平台配置
if (current_os == "ohos") {
  cj_config.dyn_extension = "so"
  if (target_cpu == "arm64") {
    cj_config.platform_target = "linux_ohos_aarch64_cjnative"
  } else if (target_cpu == "x86_64") {
    cj_config.platform_target = "linux_ohos_x86_64_cjnative"
  } else if (target_cpu == "arm") {
    cj_config.platform_target = "linux_ohos_arm_cjnative"
  } else {
    assert(false, "unsupport cpu type ${target_cpu}")
  }
}
```

### 配置说明

| 变量 | 说明 |
|-----|------|
| `prebults_cangjie_sdk_linux_x86_64_path` | 预编译 SDK 根目录 |
| `cj_config.platform_target` | 平台目标标识，用于定位子目录 |
| `cj_config.dyn_extension` | 动态库扩展名（OH 为 "so"） |

### 平台目标映射

| OH 架构 | target_cpu | platform_target |
|--------|-----------|-----------------|
| ARM64 | arm64 | linux_ohos_aarch64_cjnative |
| x86_64 | x86_64 | linux_ohos_x86_64_cjnative |
| ARM32 | arm | linux_ohos_arm_cjnative |

---

## 3.3 运行时构建配置（runtime/runtime_config.gni）

**文件位置**: `//third_party/cangjie_runtime/runtime/runtime_config.gni`

### 参数声明

```gn
declare_args() {
  CANGJIE_RUNTIME_PATH = "//third_party/cangjie_runtime/runtime"
  BUILD_TYPE = "Release"
  OHOS_PRODUCT_NAME = "rk3568"
  ASAN_SUPPORT_FLAG = 0
  HWASAN_SUPPORT_FLAG = 0
  SANITIZER_SUPPORT_FLAG = 0
  GWPASAN_SUPPORT_FLAG = 1
  CJ_SDK_VERSION = ""
  DISABLE_VERSION_CHECK = 1
  COPYGC_FLAG = "1"
  RUNTIME_FORWARD_PTRAUTH_CFI = 1
  RUNTIME_BACKWARD_PTRAUTH_CFI = 1
  OHOS_INCLUDE_DIR = "//prebuilts/ohos-sdk/linux/20/native/sysroot/usr"
  OHOS_INCLUDE_COMPONENT = ["hilog", "hitrace"]
}
```

### OH 平台条件配置

```gn
if(current_os == "ohos") {
  if (target_cpu == "arm64") {
    OHOS_FLAG = 1
    TARGET_ARCH = "aarch64"
  } else if (target_cpu == "x86_64") {
    OHOS_FLAG = 2
    TARGET_ARCH = "x86_64"
  }
}
```

### 关键配置项说明

| 配置项 | 默认值 | 说明 |
|-------|--------|------|
| `OHOS_PRODUCT_NAME` | rk3568 | 默认产品名称 |
| `OHOS_INCLUDE_DIR` | .../native/sysroot/usr | OHOS SDK 头文件路径 |
| `OHOS_INCLUDE_COMPONENT` | [hilog, hitrace] | 依赖的 OH 组件 |
| `OHOS_FLAG` | 0/1/2 | 平台标识：0=非OH，1=aarch64，2=x86_64，3=arm |
| `ASAN_SUPPORT_FLAG` | 0 | AddressSanitizer 支持 |
| `HWASAN_SUPPORT_FLAG` | 0 | Hardware ASAN 支持（仅 OH） |
| `GWPASAN_SUPPORT_FLAG` | 1 | GWP-ASAN 支持 |
| `RUNTIME_FORWARD_PTRAUTH_CFI` | 1 | 前向指针认证 CFI |
| `RUNTIME_BACKWARD_PTRAUTH_CFI` | 1 | 后向指针认证 CFI |

### 头文件搜索路径

```gn
include_dirs = [
  "${CANGJIE_RUNTIME_PATH}/src",
  "${CANGJIE_RUNTIME_PATH}/build/include/",
  "//prebuilts/clang/ohos/linux-x86_64/llvm/include/libcxx-ohos/include/c++/v1",
  "//out/${OHOS_PRODUCT_NAME}/obj/third_party/musl/usr/include/${TARGET_ARCH}-linux-ohos",
]
```

### 编译选项

#### Release 模式

```gn
cflags_cc = [
  "-D_FORTIFY_SOURCE=2",
  "-O2",
]
defines = ["_FORTIFY_SOURCE=2"]
```

#### OH 特定编译选项

```gn
if (OHOS_FLAG == 1) {  # aarch64
  ldflags += [
    "--target=aarch64-linux-ohos",
    "-fuse-ld=lld",
  ]
  defines = ["__OHOS__"]
} else if (OHOS_FLAG == 2) {  # x86_64
  ldflags += [
    "--target=x86_64-linux-ohos",
    "-fuse-ld=lld",
  ]
  defines = ["__OHOS__"]
}
```

---

## 3.4 标准库构建配置（stdlib/libs/ohos_cangjie_std.gni）

**文件位置**: `//third_party/cangjie_runtime/stdlib/libs/ohos_cangjie_std.gni`

### C FFI 源文件集模板

```gn
template("ohos_cangjie_std_cffi_source_set") {
  ohos_source_set(target_name) {
    forward_variables_from(invoker, "*", ["cflags", "cflags_c", "cflags_cc"])
    
    # 默认禁用 LTO（链接时优化）
    cflags_c = [ "-fno-lto" ]
    cflags_cc = [ "-fno-lto" ]
    
    # 合并传入的 flags
    if (defined(invoker.cflags_c)) { cflags_c += invoker.cflags_c }
    if (defined(invoker.cflags_cc)) { cflags_cc += invoker.cflags_cc }
    if (defined(invoker.cflags)) { cflags += invoker.cflags }
    
    # 生成 Cangjie 元数据
    _cangjie_config_info = {
      outputs = _outputs
      out_dir = rebase_path(target_out_dir) + "/${target_name}"
      type = "source_set"
    }
    
    metadata = {
      cangjie_deps = [ _cangjie_config_info ]
    }
  }
}
```

### 平台配置

```gn
if (is_ohos) {
  if (current_cpu == "arm64") {
    target_arch = "linux_ohos_aarch64_cjnative"
  } else if (current_cpu == "x86_64") {
    target_arch = "linux_ohos_x86_64_cjnative"
  } else {
    print("unsupported ohos platform ${current_cpu}")
    assert(false)
  }
}
```

### 配置说明

| 配置 | 说明 |
|-----|------|
| `-fno-lto` | 禁用链接时优化，避免与 Cangjie 编译器优化冲突 |
| `cangjie_deps` | 元数据，供 Cangjie 编译器识别依赖关系 |

---

## 3.5 CMake 工具链适配

### OH 专用工具链文件

#### aarch64 工具链

**文件**: `stdlib/cmake/ohos_aarch64_clang_toolchain.cmake`

```cmake
set(TRIPLE aarch64-linux-ohos)
set(OHOS ON)
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

# 定义 __ohos__ 宏
add_compile_definitions(__ohos__)
set(TARGET_TRIPLE_DIRECTORY_PREFIX "linux_ohos_aarch64")

# 使用 OHOS LLVM 工具链
set(CMAKE_C_COMPILER "$ENV{OHOS_ROOT}/prebuilts/clang/ohos/.../clang")
set(CMAKE_CXX_COMPILER "$ENV{OHOS_ROOT}/prebuilts/clang/ohos/.../clang++")
set(CMAKE_AR "$ENV{OHOS_ROOT}/prebuilts/clang/ohos/.../llvm-ar")
set(CMAKE_RANLIB "$ENV{OHOS_ROOT}/prebuilts/clang/ohos/.../llvm-ranlib")

# 编译选项
set(CMAKE_C_FLAGS "... --target=aarch64-linux-ohos ...")
set(CMAKE_CXX_FLAGS "... --target=aarch64-linux-ohos ...")

# 系统根目录
set(CMAKE_SYSROOT "$ENV{OHOS_ROOT}/out/sdk/obj/third_party/musl/sysroot")

# OpenSSL 特殊配置
set(OHOS_INCLUDE "-I$ENV{OHOS_ROOT}/third_party/openssl/include")
```

#### arm 工具链

**文件**: `stdlib/cmake/ohos_arm_clang_toolchain.cmake`

与 aarch64 类似，主要区别：
```cmake
set(TRIPLE arm-linux-ohos)
set(CMAKE_SYSTEM_PROCESSOR arm)
set(TARGET_TRIPLE_DIRECTORY_PREFIX "linux_ohos_arm")

# ARM32 特殊编译选项
set(CMAKE_C_FLAGS "... --target=arm-linux-ohos -mfloat-abi=softfp -march=armv7-a ...")
```

#### x86_64 工具链

**文件**: `stdlib/cmake/ohos_x86_64_clang_toolchain.cmake`

```cmake
set(TRIPLE x86_64-linux-ohos)
set(CMAKE_SYSTEM_PROCESSOR x86_64)
set(TARGET_TRIPLE_DIRECTORY_PREFIX "linux_ohos_x86_64")

set(CMAKE_C_FLAGS "... --target=x86_64-linux-ohos ...")
```

### 工具链对比

| 配置项 | aarch64 | arm | x86_64 |
|-------|---------|-----|--------|
| TRIPLE | aarch64-linux-ohos | arm-linux-ohos | x86_64-linux-ohos |
| target | aarch64-linux-ohos | arm-linux-ohos | x86_64-linux-ohos |
| 特殊选项 | PAC-RET | -mfloat-abi=softfp | - |
| 前缀 | linux_ohos_aarch64 | linux_ohos_arm | linux_ohos_x86_64 |

---

## 3.6 安全编译选项

### CFI（Control Flow Integrity）

```gn
# aarch64
"-flto -fsanitize=cfi -fno-sanitize=cfi-nvcall,cfi-icall -mbranch-protection=pac-ret"

# x86_64
"-flto -fsanitize=cfi -fno-sanitize=cfi-nvcall,cfi-icall"
```

### 指针认证（PAC-RET）

仅在 ARM64 启用：
```gn
"-mbranch-protection=pac-ret"
```

### 堆栈保护

```gn
cflags = [
  "-fstack-protector-strong",
  "-Wstack-protector",
]
```

###  fortify_source

```gn
cflags = ["-D_FORTIFY_SOURCE=2"]
```

---

## 3.7 与上游构建系统的差异

| 方面 | 上游构建 | OH 构建 |
|-----|---------|---------|
| **目标** | 通用 Linux/macOS/Windows | OpenHarmony |
| **工具链** | 系统默认 GCC/Clang | OHOS LLVM 工具链 |
| **libc** | glibc / musl | OHOS Musl |
| **安全选项** | 可选 | 强制（CFI、PAC-RET） |
| **日志** | stdout/stderr | Hilog |
| **集成方式** | 源码编译 | 预编译二进制 |

---

## 3.8 构建流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    OH 构建流程                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 解析 BUILD.gn                                           │
│     └── 读取 platform.gni 获取平台配置                       │
│                                                             │
│  2. 确定平台目标                                            │
│     └── linux_ohos_aarch64_cjnative（示例）                  │
│                                                             │
│  3. 定位预编译库                                            │
│     └── //prebuilts/cangjie_sdk/linux-x64/cangjie/         │
│         └── runtime/lib/linux_ohos_aarch64_cjnative/       │
│             ├── libcangjie-std-*.so (31个)                 │
│             ├── libcangjie-runtime.so                      │
│             └── ...                                        │
│                                                             │
│  4. 创建预编译库目标                                        │
│     └── ohos_prebuilt_shared_library(...)                  │
│                                                             │
│  5. 组包                                                    │
│     └── group("cangjie_prebuilts_package")                 │
│                                                             │
│  6. 安装到系统镜像                                          │
│     └── out/.../system/lib/platformsdk/cjsdk/              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.9 关键构建目标

### bundle.json 中的构建目标

```json
{
  "component": {
    "build": {
      "sub_component": [
        "//third_party/cangjie_runtime:cangjie_prebuilts_package"
      ]
    }
  }
}
```

### 运行时独立构建目标

**文件**: `runtime/BUILD.gn`

```gn
ohos_group("runtime") {
  deps = [
    "//third_party/cangjie_runtime/runtime/src:libcangjie-runtime",
    "//third_party/cangjie_runtime/runtime/src:libcangjie-runtime-static",
  ]
}
```

### CJThread 构建目标

**文件**: `runtime/src/CJThread/BUILD.gn`

```gn
ohos_static_library("libcangjie-thread") { ... }
ohos_static_library("libcangjie-aio") { ... }
```

---

## 3.10 调试构建配置

### ASAN（Address Sanitizer）

```gn
if (is_asan) {
  ASAN_SUPPORT_FLAG = 1
  SANITIZER_SUPPORT_FLAG = 1
  GWPASAN_SUPPORT_FLAG = 0
}
```

### HWASAN（Hardware ASAN）

仅支持 OHOS：
```gn
if (is_asan && use_hwasan) {
  HWASAN_SUPPORT_FLAG = 1
  # 使用 linux-x64-hwasan 预编译 SDK
}
```

### GWP-ASAN

```gn
GWPASAN_SUPPORT_FLAG = 1  # 默认启用
```

---

*本文档基于 cangjie_runtime 1.1.0-alpha.69 版本编写*
