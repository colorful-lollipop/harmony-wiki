# 附录：关键配置与宏

## 概述

本文档汇总 ArkCompiler Runtime Core 中的关键编译配置、宏定义和特性开关。

## 编译配置 (GN Args)

### 根配置

**文件**: `ark_config.gni`, `static_core/ark_config.gni`

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `ark_standalone_build` | bool | false | 是否独立构建（不依赖 OHOS） |
| `enable_static_vm` | bool | true | 启用静态 VM |
| `enable_codegen` | bool | true | 启用代码生成（JIT/AOT） |
| `enable_irtoc` | bool | true | 启用 IR 到代码转换 |
| `is_llvmbackend` | bool | false | 使用 LLVM 后端 |
| `is_llvm_aot` | bool | false | 使用 LLVM AOT |
| `is_asan` | bool | false | 启用 Address Sanitizer |
| `is_debug` | bool | false | 调试构建 |

### 平台配置

| 配置项 | 说明 |
|--------|------|
| `is_linux` | Linux 平台 |
| `is_ohos` | OpenHarmony 平台 |
| `is_mac` | macOS 平台 |
| `is_mingw` | Windows (MinGW) 平台 |
| `is_mob` | 移动平台 |

### 架构配置

| 配置项 | 说明 |
|--------|------|
| `current_cpu = "arm"` | ARM32 |
| `current_cpu = "arm64"` | ARM64 |
| `current_cpu = "x86"` | x86 32位 |
| `current_cpu = "x64"` | x86 64位 |

## 宏定义

### 平台宏

**定义位置**: `BUILD.gn` 中的 `ark_public_config`

| 宏 | 说明 | 定义条件 |
|----|------|----------|
| `PANDA_TARGET_UNIX` | Unix 平台 | is_linux \|\| is_mac \|\| is_ohos |
| `PANDA_TARGET_LINUX` | Linux | is_linux |
| `PANDA_TARGET_OHOS` | OpenHarmony | is_ohos |
| `PANDA_TARGET_MACOS` | macOS | is_mac |
| `PANDA_TARGET_WINDOWS` | Windows | is_mingw |
| `PANDA_TARGET_MOBILE` | 移动设备 | is_mob |

### 架构宏

| 宏 | 说明 | 定义条件 |
|----|------|----------|
| `PANDA_TARGET_ARM32` | ARM32 | current_cpu == "arm" |
| `PANDA_TARGET_ARM64` | ARM64 | current_cpu == "arm64" |
| `PANDA_TARGET_X86` | x86 | current_cpu == "x86" |
| `PANDA_TARGET_AMD64` | x86_64 | current_cpu == "x64" |
| `PANDA_TARGET_32` | 32位 | arm \|\| x86 |
| `PANDA_TARGET_64` | 64位 | arm64 \|\| x64 |
| `PANDA_USE_32_BIT_POINTER` | 使用32位指针 | arm64 \|\| x64 |

### 特性宏

| 宏 | 说明 | 定义条件 |
|----|------|----------|
| `PANDA_WITH_BYTECODE_OPTIMIZER` | 字节码优化器 | is_linux \|\| is_mingw \|\| is_mac |
| `PANDA_WITH_COMPILER` | 编译器 | is_linux \|\| is_mingw \|\| is_mac \|\| is_ohos |
| `PANDA_WITH_CODEGEN` | 代码生成 | enable_codegen |
| `PANDA_WITH_IRTOC` | IRTOC | enable_irtoc |
| `PANDA_USE_FUTEX` | 使用 futex | !is_mac && !is_asan |
| `PANDA_LLVM_BACKEND` | LLVM 后端 | is_llvmbackend |
| `PANDA_LLVM_AOT` | LLVM AOT | is_llvm_aot |
| `PANDA_LLVM_IRTOC` | LLVM IRTOC | is_llvm_interpreter \|\| is_llvm_fastpath |

### 调试宏

| 宏 | 说明 | 定义条件 |
|----|------|----------|
| `NDEBUG` | 非调试模式 | !is_debug |
| `PANDA_ENABLE_GLOBAL_REGISTER_VARIABLES` | 全局寄存器变量 | arm64 && !use_hwasan |
| `PANDA_TARGET_MOBILE_WITH_MANAGED_LIBS` | 移动托管库 | 默认启用 |

## 运行时选项

### 内存/GC 选项

**文件**: `static_core/runtime/options.yaml`

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--gc-heap-size` | uint32 | 256MB | 堆大小 |
| `--gc-target-pause-time` | uint32 | 20ms | GC 目标暂停时间 |
| `--gc-enable-concurrent` | bool | true | 启用并发 GC |
| `--gc-type` | string | "g1-gc" | GC 类型 |

### 编译器选项

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--compiler-enable-jit` | bool | true | 启用 JIT |
| `--compiler-hotness-threshold` | uint32 | 1000 | JIT 编译阈值 |
| `--compiler-opt-level` | uint32 | 2 | 优化级别 |
| `--compiler-inline-depth` | uint32 | 5 | 内联深度 |

### 运行时选项

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--load-runtimes` | string | "ets" | 加载的运行时 |
| `--boot-panda-files` | string | "" | 启动 ABC 文件 |
| `--interpreter-type` | string | "irtoc" | 解释器类型 |
| `--enable-an` | bool | false | 启用 AOT 文件加载 |

## 特性开关

### bundle.json 中的特性

**文件**: `bundle.json`

```json
{
  "features": [
    "runtime_core_enable_codegen",
    "runtime_core_enable_ffrt"
  ]
}
```

| 特性 | 说明 |
|------|------|
| `runtime_core_enable_codegen` | 启用代码生成 |
| `runtime_core_enable_ffrt` | 启用 FFRT 运行时 |

### 系统能力

```json
{
  "syscap": [
    "SystemCapability.ArkCompiler.ANI"
  ]
}
```

| 能力 | 说明 |
|------|------|
| `SystemCapability.ArkCompiler.ANI` | ANI 接口支持 |

## 代码中的条件编译

### 平台相关代码示例

```cpp
// 平台适配
#ifdef PANDA_TARGET_OHOS
    // OpenHarmony 特定实现
#elif PANDA_TARGET_LINUX
    // Linux 特定实现
#elif PANDA_TARGET_MACOS
    // macOS 特定实现
#endif

// 架构适配
#ifdef PANDA_TARGET_ARM64
    // ARM64 特定优化
    asm volatile("...");
#elif PANDA_TARGET_AMD64
    // x86_64 特定优化
#endif

// 特性开关
#ifdef PANDA_WITH_COMPILER
    // 编译器相关代码
#endif

#ifdef PANDA_USE_FUTEX
    // 使用 futex 实现同步
#else
    // 使用其他同步机制
#endif
```

## 构建变体

### Debug 构建

```gn
is_debug = true
```

特性:
- 启用断言
- 包含调试符号
- 禁用优化 (-Og)
- 启用边界检查

### Release 构建

```gn
is_debug = false
```

特性:
- 禁用断言
- 优化级别 -O3
- 最小调试信息

### FastVerify 构建

```gn
is_fastverify = true
```

特性:
- 启用断言
- 优化级别 -O2
- 详细调试信息

### ASan 构建

```gn
is_asan = true
```

特性:
- 启用 Address Sanitizer
- 检测内存错误
- 性能开销大

## 配置示例

### 完整的 GN args 示例

```gn
# 目标平台
is_ohos = true
current_cpu = "arm64"

# 构建类型
is_debug = false

# 特性开关
enable_static_vm = true
enable_codegen = true
enable_irtoc = true
is_llvmbackend = false

# 安全
is_asan = false
use_hwasan = false

# 其他
ark_standalone_build = false
```

### 运行时参数示例

```bash
# 开发调试
arkts_bin \
  --log-level=debug \
  --log-gc \
  --gc-heap-size=512m \
  --compiler-enable-jit \
  app.abc Entry

# 生产环境
arkts_bin \
  --gc-heap-size=256m \
  --enable-an \
  --aot-files=app.an \
  app.abc Entry
```

## 参考

- [GN 构建目标](../06_GN_Targets.md)
- [编译产物](../07_Build_Artifacts.md)
- `ark_config.gni`
- `static_core/runtime/options.yaml`
