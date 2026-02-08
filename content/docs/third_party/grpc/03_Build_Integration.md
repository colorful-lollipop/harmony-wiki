# 03 - OH 构建适配

## 概述

本文档详细说明 gRPC 在 OpenHarmony 中的构建系统适配，包括 BUILD.gn 结构、编译选项、依赖配置和与上游构建系统的差异。

## 1. BUILD.gn 结构概览

### 1.1 文件位置

```
third_party/grpc/
├── BUILD.gn          # OH 构建定义（本文档分析对象）
├── BUILD             # 上游 Bazel 构建文件
├── CMakeLists.txt    # 上游 CMake 构建文件
├── Makefile          # 上游 Make 构建文件
└── bazel/            # 上游 Bazel 配置
```

### 1.2 BUILD.gn 主要内容

```gn
# BUILD.gn 结构
├── 配置定义 (config)
│   ├── pulbic_grpc_config    # 公共配置（头文件路径、宏定义）
│   ├── private_grpc_config   # 私有配置（编译选项、警告抑制）
│   └── upb_config            # UPB 库配置
│
├── 目标定义
│   ├── address_sorting       # IP 地址排序库
│   ├── upb / upb_text_format # UPB Protocol Buffers 运行时
│   ├── gpr                   # gRPC 平台运行时 (共享库)
│   ├── grpc                  # gRPC C 核心库 (共享库)
│   ├── grpcpp                # gRPC C++ 接口 (source_set)
│   ├── grpcxx                # gRPC C++ 库 (共享库)
│   └── grpc_cpp_plugin       # protoc 插件 (可执行文件)
│
└── 工具链
    └── grpc_plugin_toolchain # 插件工具组合
```

### 1.3 构建目标关系图

```
                    ┌──────────────────┐
                    │  grpc_plugin_    │
                    │  toolchain       │
                    └────────┬─────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ↓                   ↓                   ↓
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ grpc_cpp_plugin │  │ grpc_plugin_    │  │ grpc_cpp_plugin │
│   (目标平台)     │  │ support         │  │  (主机工具链)    │
└─────────────────┘  └─────────────────┘  └─────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                        运行库依赖                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────────┐                                      │
│   │    grpcxx       │  ← C++ 应用入口                      │
│   │  (共享库)        │                                      │
│   └────────┬────────┘                                      │
│            │ deps                                           │
│            ↓                                                │
│   ┌─────────────────┐  ┌─────────────────┐                 │
│   │    grpcpp       │  │      gpr        │                 │
│   │  (source_set)   │  │   (共享库)       │                 │
│   └────────┬────────┘  └─────────────────┘                 │
│            │ deps                                           │
│            ↓                                                │
│   ┌─────────────────┐  ┌─────────────────┐                 │
│   │     grpc        │  │  address_sorting│                 │
│   │   (共享库)       │  │   (共享库)       │                 │
│   └────────┬────────┘  └─────────────────┘                 │
│            │ deps                                           │
│            ↓                                                │
│   ┌─────────────────┐  ┌─────────────────┐                 │
│   │  upb_* 系列     │  │  系统库 (SSL等)  │                 │
│   │   (静态库)       │  │                 │                 │
│   └─────────────────┘  └─────────────────┘                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 2. 关键配置详解

### 2.1 公共配置 (pulbic_grpc_config)

```gn
config("pulbic_grpc_config") {
  include_dirs = [
    "${GRPC_DIR}/",
    "${GRPC_DIR}/include/",
  ]
  defines = [
    "GRPC_USE_PROTO_LITE",          # ★ 使用 protobuf lite
    "GPR_SUPPORT_CHANNELS_FROM_FD", # ★ 支持从 fd 创建通道
    "GRPC_ARES=0",                  # ★ 禁用 c-ares
    "GPR_PTHREAD_TLS",              # ★ 使用 pthread TLS
  ]
}
```

#### 配置项分析

| 宏定义 | 值 | 说明 | OH 目的 |
|--------|----|----|---------|
| `GRPC_USE_PROTO_LITE` | 定义 | 使用 protobuf lite | 减少 ROM 占用 |
| `GPR_SUPPORT_CHANNELS_FROM_FD` | 定义 | 支持从文件描述符创建通道 | 支持特殊场景 |
| `GRPC_ARES` | 0 | 禁用 c-ares DNS | 减少依赖，使用系统 DNS |
| `GPR_PTHREAD_TLS` | 定义 | 使用 pthread 线程本地存储 | 平台兼容 |

**详细说明**:

1. **GRPC_USE_PROTO_LITE**:
   - protobuf lite 是 protobuf 的精简版本
   - 移除反射等高级特性，减小二进制体积
   - 适用于资源受限的嵌入式/移动设备

2. **GRPC_ARES=0**:
   - c-ares 是一个异步 DNS 解析库
   - 禁用后使用系统原生的 DNS 解析
   - 减少依赖，简化部署

3. **GPR_SUPPORT_CHANNELS_FROM_FD**:
   - 允许从已存在的文件描述符创建 gRPC 通道
   - 适用于与现有系统集成的场景

### 2.2 私有配置 (private_grpc_config)

```gn
config("private_grpc_config") {
  visibility = [ ":*" ]  # 仅内部可见
  
  cflags = [
    "-Wno-implicit-fallthrough",
    "-Wno-unused-variable",
    "-Wno-ignored-qualifiers",
    "-Wno-atomic-implicit-seq-cst",
    "-Wno-undef",
    "-Wno-missing-prototypes",
    "-Wno-missing-variable-declarations",
    "-Wno-sign-compare",
    "-Wno-shadow-uncaptured-local",
    "-Wno-deprecated-builtins",
    "-Wno-error=unused-variable",
  ]

  ldflags = [ "-Wl,--lto-O0" ]  # ★ 禁用 LTO
  defines = [ "OPENSSL_SUPPRESS_DEPRECATED" ]
}
```

#### 编译选项分析

**Warning 抑制**:

| 选项 | 说明 | 原因 |
|------|------|------|
| `-Wno-implicit-fallthrough` | 忽略 switch 的隐式 fallthrough | 第三方代码风格 |
| `-Wno-unused-variable` | 忽略未使用变量 | 减少编译警告噪音 |
| `-Wno-ignored-qualifiers` | 忽略类型修饰符被忽略 | 兼容性 |
| `-Wno-deprecated-builtins` | 忽略废弃内置函数 | 使用旧版编译器特性 |

**链接选项**:

- `-Wl,--lto-O0`: 禁用链接时优化 (LTO)
  - 加快编译速度
  - 减少内存使用
  - 可能略微增大二进制体积

**宏定义**:

- `OPENSSL_SUPPRESS_DEPRECATED`: 抑制 OpenSSL 废弃 API 警告
  - gRPC 可能使用了一些较旧的 OpenSSL API
  - 避免升级 OpenSSL 时产生大量警告

### 2.3 gpr 库 (平台运行时)

```gn
ohos_shared_library("gpr") {
  sources = [
    # 平台抽象层代码
    "${GRPC_DIR}/src/core/util/alloc.cc",
    "${GRPC_DIR}/src/core/util/crash.cc",
    "${GRPC_DIR}/src/core/util/time.cc",
    # ... 更多源文件
  ]
  
  external_deps = [
    "abseil-cpp:absl_base",
    "abseil-cpp:absl_log",
    "abseil-cpp:absl_strings",
    # ... 多个 abseil 组件
  ]

  if (current_toolchain != host_toolchain) {
    external_deps += [ "hilog:libhilog" ]  # ★ OH 特有
  }
  
  defines = [ "GPR_ANDROID" ]  # ★ 使用 Android 兼容模式
}
```

**关键特性**:

1. **hilog 集成**:
   - 仅在目标平台（非主机）编译时引入
   - 使用 OH 的统一日志系统

2. **Android 兼容**:
   - 定义 `GPR_ANDROID`
   - 可能与 Android 共享代码路径
   - 利用 Android 生态的成熟优化

### 2.4 grpc 库 (核心库)

```gn
ohos_shared_library("grpc") {
  sources = [
    # 核心功能代码
    "${GRPC_DIR}/src/core/call/*.cc",
    "${GRPC_DIR}/src/core/channelz/*.cc",
    "${GRPC_DIR}/src/core/client_channel/*.cc",
    "${GRPC_DIR}/src/core/credentials/**/*.cc",
    "${GRPC_DIR}/src/core/ext/**/*.cc",
    # ... 大量源文件
  ]
  
  deps = [
    ":address_sorting",
    ":gpr",
    ":upb_message_lib",
    ":upb_text_format",
    ":upb_wire_lib",
  ]
  
  external_deps = [
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
    "re2:re2",
    "zlib:libz",
    "abseil-cpp:absl_*",
  ]
  
  if (!is_asan && !is_debug) {
    version_script = "libgrpc.map"  # ★ 符号版本控制
  }
}
```

**关键特性**:

1. **符号版本控制**:
   - 使用 `libgrpc.map` 控制导出的符号
   - 仅在非调试构建时启用
   - 有助于二进制兼容性

2. **依赖链**:
   - 依赖 gpr（平台抽象）
   - 依赖 upb 系列（Protocol Buffers）
   - 依赖系统安全库（OpenSSL）

### 2.5 grpcxx 库 (C++ 接口)

```gn
ohos_shared_library("grpcxx") {
  deps = [ ":grpcpp" ]
  # ...
}

source_set("grpcpp") {
  sources = [
    "${GRPC_DIR}/src/cpp/client/*.cc",
    "${GRPC_DIR}/src/cpp/server/*.cc",
    "${GRPC_DIR}/src/cpp/common/*.cc",
    # ...
  ]
  external_deps = [
    "protobuf:protobuf_lite",  # ★ 使用 lite
  ]
}
```

**设计**:

- `grpcpp` 是 source_set，直接包含源文件
- `grpcxx` 是共享库，包装 grpcpp
- 应用链接 `grpcxx` 即可获得完整功能

## 3. 与上游构建系统的差异

### 3.1 构建系统对比

| 方面 | 上游 (Bazel) | OH (GN) | 差异说明 |
|------|-------------|---------|---------|
| **构建工具** | Bazel | GN + Ninja | OH 使用 GN 元构建 |
| **目标类型** | 灵活 | 受限制 | GN 目标类型较少 |
| **条件编译** | select() | if() | 语法不同 |
| **外部依赖** | 子模块 | OH 组件 | 使用 bundle.json |

### 3.2 编译选项对比

| 选项 | 上游 | OH | 原因 |
|------|------|----|----|
| **LTO** | 默认启用 | 禁用 (`-Wl,--lto-O0`) | 加快编译 |
| **Warning 级别** | 严格 | 大量抑制 | 第三方代码兼容 |
| **Debug 符号** | 可选 | 标准配置 | 调试需求 |
| **优化级别** | -O2/-O3 | 标准 | 性能与体积平衡 |

### 3.3 依赖管理对比

**上游**:
```bazel
# Bazel WORKSPACE
git_repository(
    name = "com_google_protobuf",
    remote = "...",
    tag = "...",
)
```

**OH**:
```gn
# BUILD.gn
external_deps = [
    "protobuf:protobuf_lite",
    "abseil-cpp:absl_base",
    # ...
]
```

**差异**:
- 上游：通过 Bazel 下载和管理依赖
- OH：依赖 OH 组件系统，通过 bundle.json 声明

### 3.4 输出格式对比

| 输出 | 上游 | OH |
|------|------|-----|
| libgrpc | 静态/动态可选 | 动态库 (`.so`) |
| libgpr | 静态/动态可选 | 动态库 (`.so`) |
| libgrpc++ | 静态/动态可选 | 动态库 (`.so`) |
| 插件 | 可执行文件 | 可执行文件 |

## 4. 关键差异详解

### 4.1 DNS 解析器选择

**上游默认**: c-ares (异步 DNS)
```bazel
# 上游默认启用 c-ares
defines = ["GRPC_ARES"]  # 或省略
```

**OH 配置**: 禁用 c-ares
```gn
defines = ["GRPC_ARES=0"]
```

**原因**:
1. 减少依赖项
2. 使用系统原生 DNS
3. 简化部署

**影响**:
- DNS 解析变为同步（阻塞）
- 在大多数场景下影响很小
- 高并发场景可能有影响

### 4.2 Protobuf 版本

**上游**: 使用 full protobuf

**OH**: 使用 protobuf lite
```gn
defines = ["GRPC_USE_PROTO_LITE"]
external_deps = ["protobuf:protobuf_lite"]
```

**差异**:

| 特性 | Full | Lite |
|------|------|------|
| 反射 | 支持 | 不支持 |
| 二进制体积 | 较大 | 较小 |
| 性能 | 略低 | 略高 |
| 功能 | 完整 | 核心功能 |

**原因**: 减少 ROM 占用，适用于资源受限设备

### 4.3 TLS/SSL 实现

**上游**: BoringSSL (Google 分支)

**OH**: OpenSSL (系统版本)
```gn
external_deps = [
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
]
```

**原因**:
1. 使用系统统一的安全库
2. 便于安全更新管理
3. 与 OH 安全框架集成

### 4.4 日志系统

**上游**: gRPC 原生日志

**OH**: hilog 集成
```gn
if (current_toolchain != host_toolchain) {
    external_deps += [ "hilog:libhilog" ]
}
```

**原因**: 统一使用 OH 的日志系统，便于调试和日志收集

## 5. 构建使用指南

### 5.1 依赖 gRPC 的模块

在您的 BUILD.gn 中：

```gn
ohos_executable("my_app") {
    sources = [ "main.cpp" ]
    external_deps = [
        "grpc:grpcxx",      # C++ API
        # 或 "grpc:grpc",    # C API
    ]
}
```

### 5.2 使用 protoc 插件

```gn
action("generate_grpc_code") {
    script = "//third_party/grpc/grpc_cpp_plugin"
    inputs = [ "my_service.proto" ]
    outputs = [
        "${target_gen_dir}/my_service.grpc.pb.h",
        "${target_gen_dir}/my_service.grpc.pb.cc",
    ]
    args = [
        "--grpc_out=${target_gen_dir}",
        "--plugin=protoc-gen-grpc=${script}",
        "${inputs}",
    ]
}
```

### 5.3 编译选项覆盖

如需覆盖默认配置：

```gn
config("my_grpc_config") {
    defines = ["MY_CUSTOM_DEFINE"]
    cflags = ["-O3"]
}

ohos_executable("my_app") {
    configs += [":my_grpc_config"]
}
```

## 6. 常见问题

### Q1: 为什么禁用 c-ares？

A: 减少依赖，使用系统 DNS。如果确实需要异步 DNS，可以：
1. 移除 `GRPC_ARES=0`
2. 在 deps 中添加 c-ares 组件

### Q2: 如何启用调试日志？

A: gRPC 使用环境变量控制日志：
```bash
export GRPC_VERBOSITY=DEBUG
export GRPC_TRACE=all
```

### Q3: 能否使用静态库？

A: 当前配置输出动态库。如需静态库，需要修改 BUILD.gn，将 `ohos_shared_library` 改为 `ohos_static_library`。

### Q4: protobuf full 和 lite 能混用吗？

A: 不建议。整个系统应该统一使用一种版本。如果其他组件使用 full，可能需要调整 gRPC 配置。

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - gRPC 简介
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景
