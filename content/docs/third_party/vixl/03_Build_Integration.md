# OH 构建适配

## 构建系统概述

VIXL 在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统，这与其上游使用的 SCons/CMake 不同。所有 OH 特定的构建适配都集中在 `BUILD.gn` 文件中，源代码保持与上游一致。

## BUILD.gn 结构

### 整体架构

```gn
# 导入 OpenHarmony 构建模板
import("//build/ohos.gni")

# 公共配置 (头文件路径、宏定义)
config("vixl_public_config") { ... }

# 静态库目标
ohos_static_library("libvixl") { ... }
```

### 配置层次

| 目标 | 类型 | 用途 |
|-----|------|------|
| `vixl_public_config` | config | 公共编译配置 (defines, include_dirs) |
| `libvixl` | ohos_static_library | 静态库目标 |

## 关键编译配置

### 公共配置 (`vixl_public_config`)

```gn
config("vixl_public_config") {
  defines = []

  # 调试模式添加 VIXL_DEBUG
  if (is_debug) {
    defines += [ "VIXL_DEBUG" ]
  }

  # OH 特定宏定义
  defines += [
    "PANDA_BUILD",           # Panda 构建标识
    "VIXL_CODE_BUFFER_MMAP"  # 使用 mmap 分配代码缓冲区
  ]

  # OHOS 工具链特定警告抑制
  if (defined(ark_standalone_build) && ark_standalone_build) {
    cflags_cc += [ "-Wno-bitwise-instead-of-logical" ]
  }

  # 头文件搜索路径
  include_dirs = [ "src" ]

  # 架构特定头文件路径
  if (target_cpu == "arm") {
    include_dirs += [ "src/aarch32" ]
  } else if (target_cpu == "arm64" || target_cpu == "amd64" ||
             target_cpu == "x64" || target_cpu == "x86_64") {
    include_dirs += [ "src/aarch64" ]
  }
}
```

### OH 特定宏定义

| 宏 | 作用 | 来源 |
|---|-----|------|
| `PANDA_BUILD` | 标识 Panda/OH 构建环境 | OH |
| `VIXL_CODE_BUFFER_MMAP` | 使用 mmap 而非 malloc 分配代码缓冲区 | OH |
| `VIXL_INCLUDE_TARGET_A32` | 包含 AArch32 (A32) 指令集支持 | 条件编译 |
| `VIXL_INCLUDE_TARGET_A64` | 包含 AArch64 指令集支持 | 条件编译 |
| `VIXL_INCLUDE_SIMULATOR_AARCH64` | 包含 AArch64 模拟器支持 | 条件编译 |

### 静态库目标 (`libvixl`)

#### 通用源文件

```gn
sources = [
  "src/code-buffer-vixl.cc",
  "src/compiler-intrinsics-vixl.cc",
  "src/cpu-features.cc",
  "src/utils-vixl.cc",
]
```

#### 通用编译选项

```gn
cflags_cc = [
  "-std=c++17",         # C++17 标准
  "-pedantic",          # 严格标准检查
  "-Wall",              # 启用所有警告
  "-Wextra",            # 额外警告
  "-Werror",            # 警告视为错误
  "-fno-rtti",          # 禁用 RTTI
  "-fno-exceptions",    # 禁用异常
  "-Wno-invalid-offsetof",
  "-Wno-gnu-statement-expression",
  "-Wno-unused-parameter",
  "-Wno-unused-result",
  "-Wno-deprecated-declarations",
]
```

#### 调试构建选项

```gn
if (is_debug) {
  cflags_cc += [
    "-Og",        # 优化调试
    "-ggdb3",     # GDB 调试信息
    "-gdwarf-4",  # DWARF 4 调试格式
  ]
}
```

#### Address Sanitizer 支持

```gn
if (is_asan) {
  cflags_cc += [ "-g" ]
  defines += [ "__SANITIZE_ADDRESS__" ]
}
```

### 架构特定配置

#### AArch32 (arm) 配置

```gn
if (target_cpu == "arm") {
  sources += [
    "src/aarch32/assembler-aarch32.cc",
    "src/aarch32/constants-aarch32.cc",
    "src/aarch32/disasm-aarch32.cc",
    "src/aarch32/instructions-aarch32.cc",
    "src/aarch32/location-aarch32.cc",
    "src/aarch32/macro-assembler-aarch32.cc",
    "src/aarch32/operands-aarch32.cc",
  ]
  defines += [ "VIXL_INCLUDE_TARGET_A32" ]

  cflags_cc += [
    "-march=armv7-a",
    "-mfloat-abi=softfp",
    "-marm",
    "-mfpu=vfp",
  ]
}
```

#### AArch64 (arm64) 配置

```gn
if (target_cpu == "arm64" || target_cpu == "amd64" ||
    target_cpu == "x64" || target_cpu == "x86_64") {
  sources += [
    "src/aarch64/assembler-aarch64.cc",
    "src/aarch64/assembler-sve-aarch64.cc",
    "src/aarch64/cpu-aarch64.cc",
    "src/aarch64/cpu-features-auditor-aarch64.cc",
    "src/aarch64/debugger-aarch64.cc",
    "src/aarch64/decoder-aarch64.cc",
    "src/aarch64/disasm-aarch64.cc",
    "src/aarch64/instructions-aarch64.cc",
    "src/aarch64/logic-aarch64.cc",
    "src/aarch64/macro-assembler-aarch64.cc",
    "src/aarch64/macro-assembler-sve-aarch64.cc",
    "src/aarch64/operands-aarch64.cc",
    "src/aarch64/pointer-auth-aarch64.cc",
    "src/aarch64/simulator-aarch64.cc",
  ]
  defines += [
    "VIXL_INCLUDE_TARGET_A64",
    "VIXL_INCLUDE_SIMULATOR_AARCH64",
  ]
}
```

#### macOS 特殊配置

```gn
if (is_mac) {
  defines += [ "PANDA_TARGET_MACOS" ]
}
```

## 模块导出配置

### 组件定义 (`bundle.json`)

```json
{
  "component": {
    "name": "vixl",
    "subsystem": "thirdparty",
    "build": {
      "sub_component": [
        "//third_party/vixl:libvixl"
      ],
      "inner_kits": [
        {
          "name": "//third_party/vixl:libvixl",
          "header": {
            "header_files": [],
            "header_base": "//third_party/vixl/src"
          }
        }
      ]
    }
  }
}
```

### 导出配置

```gn
public_configs = [ ":vixl_public_config" ]

subsystem_name = "thirdparty"
part_name = "vixl"
```

## 与上游构建系统的差异

| 方面 | 上游 (SCons/CMake) | OH (GN) |
|-----|-------------------|---------|
| **构建文件** | `SConstruct`, `CMakeLists.txt` | `BUILD.gn` |
| **C++ 标准** | C++11 (默认) | C++17 (显式指定) |
| **RTTI** | 启用 | 禁用 (`-fno-rtti`) |
| **异常** | 启用 | 禁用 (`-fno-exceptions`) |
| **默认分配器** | malloc | mmap (OH 特定) |
| **编译器标识** | 无 | `PANDA_BUILD` |

## 编译命令

### 全量编译

```bash
./build.sh --product-name rk3568 --build-target libvixl_frontend_static
```

### 单独编译

```bash
# 使用 hdc 编译
hb build -p rk3568 -T //third_party/vixl:libvixl
```

### 构建产物

| 产物路径 | 说明 |
|--------|------|
| `out/rk3568/obj/third_party/vixl/libvixl_frontend_static.a` | 前端静态库 |
| `out/rk3568/obj/third_party/vixl/libvixl.a` | 主静态库 |

## 依赖关系

### 无外部依赖

VIXL 库本身**不依赖任何第三方库**，这是其设计特点之一。

### 被依赖情况

```gn
# arkcompiler 中的依赖声明
external_deps += [ "vixl:libvixl" ]
```

## 故障排查

### 常见编译错误

#### 1. RTTI 相关错误

**错误**: `error: cannot use typeid with -fno-rtti`

**解决**: 确保所有使用 VIXL 的代码也使用 `-fno-rtti` 编译。

#### 2. 异常相关错误

**错误**: `error: exception handling disabled`

**解决**: 确保禁用了异常，或使用 `try/catch` 的代码被正确隔离。

#### 3. 头文件路径错误

**错误**: `fatal error: 'xxx.h' file not found`

**解决**: 确保 `vixl_public_config` 被正确添加到 `public_configs`。
