# 配置开关与宏定义

本文档汇总方舟工具链中所有的编译配置开关、宏定义和 Feature Flags。

## GN 配置变量

### 全局配置 (toolchain_config.gni)

| 变量名 | 默认值 | 类型 | 说明 |
|--------|--------|------|------|
| `ark_standalone_build` | false | bool | 独立编译模式（不依赖 OHOS 构建系统） |
| `toolchain_enable_cmc_gc` | false | bool | 启用 CMC GC（Concurrent Marking GC） |
| `enable_leak_check` | false | bool | 启用内存泄漏检查 |
| `enable_cow_array` | true | bool | 启用写时复制数组（Copy-On-Write） |
| `enable_coverage` | false | bool | 启用代码覆盖率检测 |
| `enable_asm_assert` | false | bool | 启用汇编断言 |
| `ark_compile_mode` | "debug" | string | 编译模式（debug/release） |

### 平台检测变量

| 变量名 | 说明 | 适用平台 |
|--------|------|----------|
| `is_ohos` | OHOS 平台标识 | OHOS 标准设备 |
| `is_linux` | Linux 平台标识 | Linux 桌面/服务器 |
| `is_mingw` | MinGW 平台标识 | Windows (MinGW) |
| `is_mac` | macOS 平台标识 | macOS |
| `target_os == "android"` | Android 平台 | Android |
| `target_os == "ios"` | iOS 平台 | iOS |
| `is_arkui_x` | ArkUI-X 跨平台 | 跨平台 |
| `is_standard_system` | 标准系统标识 | OHOS 标准系统 |
| `is_wearable_product` | 可穿戴设备标识 | 智能手表等 |

### HiViewDFX 功能开关

| 变量名 | 默认条件 | 说明 |
|--------|----------|------|
| `enable_hilog` | `!ark_standalone_build && !is_arkui_x && (is_ohos \|\| is_mingw \|\| is_mac)` | 启用 HiLog 日志输出 |
| `enable_dump_in_faultlog` | `!ark_standalone_build && !is_arkui_x && is_ohos && is_standard_system` | 启用故障日志转储 |
| `enable_bytrace` | `!ark_standalone_build && !is_arkui_x && is_ohos && is_standard_system` | 启用 ByTrace 跟踪 |
| `enable_hitrace` | `!ark_standalone_build && !is_arkui_x && is_ohos && is_standard_system` | 启用 HiTrace 追踪 |

## 编译宏定义

### 平台宏 (ark_platform_config)

| 宏定义 | 平台 | 说明 |
|--------|------|------|
| `OHOS_PLATFORM` | OHOS | OHOS 平台 |
| `UNIX_PLATFORM` | Unix-like | Unix 通用平台 |
| `LINUX_PLATFORM` | Linux | Linux 平台 |
| `WINDOWS_PLATFORM` | Windows | Windows 平台 |
| `ANDROID_PLATFORM` | Android | Android 平台 |
| `MAC_PLATFORM` | macOS | macOS 平台 |
| `IOS_PLATFORM` | iOS | iOS 平台 |

### 架构宏 (ark_toolchain_common_config)

| 宏定义 | 架构 | 说明 |
|--------|------|------|
| `PANDA_TARGET_ARM32` | ARM 32位 | ARM 32位架构 |
| `PANDA_TARGET_ARM32_ABI_SOFT=1` | ARM 32位 | ARM 软 ABI |
| `PANDA_TARGET_ARM64` | ARM 64位 | ARM 64位架构 |
| `PANDA_TARGET_X86` | x86 | x86 架构 |
| `PANDA_TARGET_AMD64` | amd64 | AMD64/x86_64 架构 |
| `PANDA_TARGET_32` | 32位 | 32位通用 |
| `PANDA_TARGET_64` | 64位 | 64位通用 |

### 指针/寄存器宏

| 宏定义 | 架构 | 说明 |
|--------|------|------|
| `PANDA_USE_32_BIT_POINTER` | arm64/x64 | 使用 32位指针（压缩指针） |
| `PANDA_ENABLE_GLOBAL_REGISTER_VARIABLES` | arm64 | 启用全局寄存器变量 |

### 编译选项宏

| 宏定义 | 构建类型 | 说明 |
|--------|----------|------|
| `NDEBUG` | Release | 发布模式（禁用断言） |
| `PANDA_ENABLE_LTO` | 所有 | 启用链接时优化（LTO） |
| `USE_CMC_GC` | 条件编译 | 启用 CMC GC |
| `ENABLE_FFRT_INTERFACES` | OHOS 标准系统 | 启用 FFRT 异步接口 |

### 功能支持宏 (ark_toolchain_public_config)

| 宏定义 | 平台 | 说明 |
|--------|------|------|
| `ECMASCRIPT_SUPPORT_CPUPROFILER` | 非 MinGW/macOS/iOS/Android | CPU 分析器支持 |
| `ECMASCRIPT_SUPPORT_HEAPPROFILER` | 非 MinGW/macOS/iOS | 堆内存分析器支持 |
| `ECMASCRIPT_SUPPORT_HEAPSAMPLING` | 非 MinGW/macOS/iOS | 堆采样支持 |
| `ECMASCRIPT_SUPPORT_SNAPSHOT` | 非 MinGW/macOS/iOS | 快照支持 |
| `ECMASCRIPT_SUPPORT_DEBUGGER` | 大部分平台 | 调试器支持 |
| `ECMASCRIPT_SUPPORT_TRACING` | 大部分平台 | 调用链追踪支持 |

**Android 平台例外**（仅启用部分功能）：
- `ECMASCRIPT_SUPPORT_CPUPROFILER`
- `ECMASCRIPT_SUPPORT_DEBUGGER`
- `ECMASCRIPT_SUPPORT_TRACING`

## 运行时配置 (RuntimeOption)

### 日志级别

| 级别 | 值 | 说明 |
|------|-----|------|
| `LOG_LEVEL::ERROR` | 0 | 仅错误 |
| `LOG_LEVEL::WARNING` | 1 | 警告及以上 |
| `LOG_LEVEL::INFO` | 2 | 信息及以上 |
| `LOG_LEVEL::DEBUG` | 3 | 调试及以上 |

### GC 类型

| 类型 | 说明 |
|------|------|
| `GC_TYPE::STW_GC` | Stop-The-World GC |
| `GC_TYPE::COMPRESS_GC` | 压缩 GC |
| `GC_TYPE::PARTIAL_GC` | 部分 GC |
| `GC_TYPE::LOCAL_GC` | 本地 GC |

## Feature Flags

### 调试功能开关

| 功能 | 默认值 | 配置位置 | 说明 |
|------|--------|----------|------|
| 断点 | 启用 | 运行时 | 调试会话中启用 |
| 单步执行 | 启用 | 运行时 | 调试会话中启用 |
| CallFrame 求值 | 启用 | 运行时 | 调试会话中启用 |
| 条件断点 | 启用 | 运行时 | 需应用支持 |

### Profiler 功能开关

| 功能 | 默认值 | 平台限制 | 说明 |
|------|--------|----------|------|
| CPU 采样 | 启用 | 非 MinGW/macOS/iOS | 基于 DFXJSNApi |
| 堆快照 | 启用 | 非 MinGW/macOS/iOS | 内存密集型操作 |
| 内存追踪 | 启用 | 非 MinGW/macOS/iOS | 周期性采样 |
| 采样分析 | 启用 | 非 MinGW/macOS/iOS | 基于 Allocator |

## 环境变量

### 调试相关

| 环境变量 | 说明 |
|----------|------|
| `ENABLE_DEBUG_LOG` | 启用调试日志 |
| `ARKDEBUGGER_SOCKET_PATH` | 指定 Unix Socket 路径 |
| `ARKDEBUGGER_PORT` | 指定 TCP 端口 |

### 性能相关

| 环境变量 | 说明 |
|----------|------|
| `CPUPROFILER_INTERVAL` | CPU 采样间隔（微秒） |
| `HEAPTRACKING_INTERVAL` | 堆追踪间隔（毫秒） |
| `MAX_HEAP_SNAPSHOT_SIZE` | 最大堆快照大小（MB） |

---

*相关文档：[05_GN_Build.md](./05_GN_Build.md) | [08_Troubleshooting.md](./08_Troubleshooting.md)*
