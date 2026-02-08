# FaultLoggerd GN 构建系统

## 目的

本文档介绍 faultloggerd 的 GN 构建系统、编译目标和依赖关系。

## 适用范围

- 目标读者：系统开发者、构建工程师
- 内容范围：生产代码 BUILD.gn 文件（排除测试目录）

## 构建系统概览

faultloggerd 使用 **GN (Generate Ninja)** 构建系统：
- 根 BUILD.gn：`BUILD.gn` - 定义目标组和子模块
- 配置文件：`faultloggerd.gni` - 全局编译选项
- 平台差异：`is_ohos_lite` - 轻量级 vs 标准版

## 关键编译选项（faultloggerd.gni:14-34）

| 选项 | 默认值 | 说明 |
|------|---------|------|
| `libunwinder_debug` | `false` | 启用 unwinder 调试日志 |
| `has_libunwindstack` | `false` | 使用 libunwindstack 库 |
| `processdump_minidebuginfo_enable` | `true` | 启用 mini 调试信息 |
| `faultloggerd_hisysevent_enable` | `false`（自动检测） | 启用 HiSysEvent 集成 |
| `faultloggerd_liteperf_enable` | `true` | 启用轻量性能分析 |
| `processdump_parse_lock_owner_enable` | `false` | 启用锁持有者解析 |
| `faultloggerd_enable_build_targets` | `true` | 主开关，控制所有目标构建 |

## 目标分类

### 1. 可执行文件（Executables）

| Target | 路径 | 类型 | 输出 | 安装路径 | 关键依赖 |
|--------|------|------|--------|------------|----------|
| `faultloggerd` | `services/BUILD.gn` | `ohos_executable` | `/system/bin/faultloggerd` | system, updater | libfaultloggerd, dfx_util, dfx_trace, libdfx_procinfo, dfx_local_handler_src |
| `processdump` | `tools/process_dump/BUILD.gn` | `ohos_executable` | `/system/bin/processdump` | system, updater | libunwinder, libdfx_dumpcatcher, libfaultloggerd, libasync_stack, libbacktrace_local, dfx_local_handler_src |
| `dumpcatcher` | `tools/dump_catcher/BUILD.gn` | `ohos_executable` | `/system/bin/dumpcatcher` | system, updater | libdfx_dumpcatcher, libfaultloggerd, libbacktrace_local, libjson_stack_formatter |
| `crashvalidator` | `tools/crash_validator/BUILD.gn` | `ohos_executable` | `/system/bin/crashvalidator` | c_utils, hisysevent | libfaultloggerd, libdfx_dumpcatcher |
| `crasher_c` | `tools/crasher_c/BUILD.gn` | `ohos_executable` | `crasher_c` | - | hilog |
| `crasher_cpp` | `tools/crasher_cpp/BUILD.gn` | `ohos_executable` | `crasher_cpp` | - | dfx_signalhandler, libunwinder, libasync_stack |
| `rustpanic_maker` | `tools/panic_maker/BUILD.gn` | `ohos_rust_executable` | `rustpanic_maker` | - | panic_handler, stacktrace_rust |
| `dumpcatcherdemo` | `example/BUILD.gn` | `ohos_executable` | `dumpcatcherdemo` | - | libdfx_dumpcatcher, libjson_stack_formatter |

### 2. 共享库（Shared Libraries .so）

| Target | 路径 | 类型 | 输出 | 安装路径 | 关键特性 |
|--------|------|------|--------|------------|----------|
| `libdfx_dumpcatcher` | `interfaces/innerkits/dump_catcher/BUILD.gn` | `ohos_shared_library` | `libdfx_dumpcatcher.z.so` | system, updater | 主动抓栈 SDK，支持混合栈 |
| `libunwinder` | `interfaces/innerkits/unwinder/BUILD.gn` | `ohos_shared_library` | `libunwinder.z.so` | system, updater | 多架构符号解析（ARM64/ARM/x86_64/RISC-V/LoongArch） |
| `libfaultloggerd` | `interfaces/innerkits/faultloggerd_client/BUILD.gn` | `ohos_shared_library` | `libfaultloggerd.z.so` | system, updater | Socket 客户端库 |
| `libbacktrace_local` | `interfaces/innerkits/backtrace/BUILD.gn` | `ohos_shared_library` | `libbacktrace_local.z.so` | system, updater | 本地回栈功能 |
| `libdfx_procinfo` | `interfaces/innerkits/procinfo/BUILD.gn` | `ohos_shared_library` | `libdfx_procinfo.z.so` | system, updater | 进程信息查询 |
| `libasync_stack` | `interfaces/innerkits/async_stack/BUILD.gn` | `ohos_shared_library` | `libasync_stack.z.so` | system, updater | 异步栈跟踪（仅 ARM64） |
| `libstack_printer` | `interfaces/innerkits/stack_printer/BUILD.gn` | `ohos_shared_library` | `libstack_printer.z.so` | - | 栈打印工具 |
| `libjson_stack_formatter` | `interfaces/innerkits/formatter/BUILD.gn` | `ohos_shared_library` | `libjson_stack_formatter.z.so` | system, updater | JSON 输出格式化 |
| `crash_exception` | `interfaces/innerkits/crash_exception/BUILD.gn` | `ohos_shared_library` | `libcrash_exception.z.so` | system, updater | 崩溃异常处理 |
| `dfx_signalhandler` | `interfaces/innerkits/signal_handler/BUILD.gn` | `ohos_shared_library` | `libdfx_signalhandler.z.so` | system, updater | 信号处理和崩溃捕获 |
| `kernel_snapshot` | `services/snapshot/BUILD.gn` | `ohos_shared_library` | `libkernel_snapshot.z.so` | system | 内核快照处理 |

### 3. 静态库（Static Libraries .a）

| Target | 路径 | 类型 | 用途 |
|--------|------|------|------|
| `libunwinder_static` | `interfaces/innerkits/unwinder/BUILD.gn` | `ohos_static_library` | 静态链接的 unwinder |
| `libunwinder_base` | `interfaces/innerkits/unwinder/BUILD.gn` | `ohos_static_library` | unwinder 基础库（最小依赖） |
| `backtrace_local` | `interfaces/innerkits/backtrace/BUILD.gn` | `ohos_static_library` | 静态链接的 backtrace |
| `dfx_procinfo_static` | `interfaces/innerkits/procinfo/BUILD.gn` | `ohos_static_library` | 静态链接的 procinfo |
| `dfx_util` | `common/dfxutil/BUILD.gn` | `ohos_static_library` | DFX 工具函数 |
| `dfx_util_static` | `common/dfxutil/BUILD.gn` | `ohos_static_library` | 静态 DFX 工具 |
| `dfx_cutil` | `common/cutil/BUILD.gn` | `ohos_static_library` | C 工具函数 |
| `dfx_hilog` | `common/dfxlog/BUILD.gn` | `ohos_static_library` | 日志封装 |
| `dfx_sigdump_handler` | `interfaces/innerkits/sigdump_handler/BUILD.gn` | `ohos_static_library` | SIGDUMP 处理 |

### 4. Rust 库

| Target | 路径 | 类型 | 输出 | Crate 类型 |
|--------|------|------|--------|------------|
| `stacktrace_rust` | `interfaces/rust/stacktrace/BUILD.gn` | `ohos_rust_shared_library` | `libstacktrace_rust.so` | dylib |
| `panic_handler` | `interfaces/rust/panic_handler/BUILD.gn` | `ohos_rust_shared_library` | `libpanic_handler.so` | dylib |
| `rustc_demangle` | `interfaces/rust/rustc_demangle/BUILD.gn` | `ohos_rust_shared_ffi` | `librustc_demangle.so` | cdylib |

### 5. 源集合（Source Sets）

| Target | 路径 | 用途 |
|--------|------|------|
| `libunwinder_src` | `interfaces/innerkits/unwinder/BUILD.gn` | Unwinder 源代码集合（用于测试） |
| `process_info_src` | `tools/process_dump/BUILD.gn` | ProcessDump 源代码可复用集合 |
| `dfx_local_handler_src` | `interfaces/innerkits/signal_handler/BUILD.gn` | 本地信号处理源代码 |
| `dfx_limited_src` | `frameworks/limited/BUILD.gn` | 轻量级故障日志客户端源代码 |
| `dfx_allocator_src` | `frameworks/allocator/BUILD.gn` | 分配器源代码 |
| `dfx_trace` | `common/trace/BUILD.gn` | 追踪实现 |

## 平台差异

### ohos_lite vs Full Build

| 组件 | ohos_lite | Full Build |
|------|-----------|------------|
| **libunwinder** | shared only | shared + static + base + src |
| **libbacktrace_local** | shared only | shared + static |
| **libdfx_procinfo** | shared only | shared + static |
| **libasync_stack** | stub | 完整实现 |
| **libstack_printer** | stub | 完整实现 |
| **libjson_stack_formatter** | stub | 完整实现 |
| **dfx_signalhandler** | lite 版本 | 完整版本 |
| **Rust components** | 排除 | 包含 |
| **kernel_snapshot** | 排除 | 包含 |

### 架构特定特性

| 特性 | ARM64 | 其他架构 |
|------|-------|----------|
| **Coredump 支持** | 是 | 否 |
| **ENABLE_MIXSTACK** | 定义 | 未定义 |
| **ENABLE_PARAMETER** | 定义 | 未定义 |
| **getcontext_x86_64.S** | 排除 | 包含 |

## 依赖关系

### 核心服务依赖图

```
faultloggerd (executable)
├── libfaultloggerd (shared)
│   ├── dfx_cutil
│   ├── dfx_hilog
│   ├── dfx_util
│   ├── dfx_trace
│   └── dfx_local_handler_src
├── libdfx_procinfo (shared)
│   └── dfx_util
├── libunwinder (shared)
│   ├── dfx_hilog
│   ├── dfx_util
│   └── dfx_trace_dlsym
├── libbacktrace_local (shared)
│   ├── dfx_hilog
│   ├── dfx_util
│   ├── dfx_trace_dlsym
│   └── libdfx_procinfo
├── libdfx_dumpcatcher (shared)
│   ├── dfx_hilog
│   ├── dfx_util
│   ├── dfx_trace_dlsym
│   ├── libfaultloggerd
│   ├── libbacktrace_local
│   └── libunwinder
├── libasync_stack (shared)
│   ├── dfx_hilog
│   ├── dfx_util
│   ├── libbacktrace_local
│   └── libunwinder
└── dfx_local_handler_src (source_set)
    ├── dfx_allocator_src
    ├── dfx_cutil
    ├── dfx_util
    └── libfaultloggerd
```

### DumpCatcher 依赖图

```
dumpcatcher (executable)
├── libdfx_dumpcatcher (shared)
│   ├── dfx_cutil
│   ├── dfx_hilog
│   ├── dfx_util
│   ├── dfx_trace_dlsym
│   ├── libfaultloggerd
│   ├── libbacktrace_local
│   └── libunwinder
├── libjson_stack_formatter (shared)
│   ├── dfx_trace_dlsym
│   ├── libbacktrace_local
│   └── libunwinder
└── dfx_local_handler_src (source_set)
    ├── dfx_cutil
    └── dfx_util
```

## 构建配置

### 全局 Defines

- `is_ohos_lite`：轻量级系统标志
- `HISYSEVENT_DISABLE`：禁用 HiSysEvent（条件编译）

### 公共 Includes

| 路径变量 | 路径 | 用途 |
|-----------|------|------|
| `$faultloggerd_interfaces_path/common` | 公共接口定义 |
| `$faultloggerd_common_path/dfxlog` | 日志宏定义 |
| `$faultloggerd_common_path/dfxutil` | 工具函数 |
| `$faultloggerd_common_path/cutil` | C 工具函数 |

### 目标条件

```gn
if (!defined(ohos_lite)) {
  # 完整构建目标
} else {
  # 轻量构建目标
}

if (faultloggerd_hisysevent_enable) {
  # HiSysEvent 相关目标
}
```

## 配置文件安装

| 配置文件 | 源路径 | 安装路径 | 用途 |
|---------|--------|------------|------|
| `faultloggerd.cfg` | `services/config/faultloggerd.cfg` | `/system/etc/faultloggerd.cfg` | 服务启动配置 |
| `faultloggerd_config.json` | `services/config/faultloggerd_config.json` | `/system/etc/faultloggerd_config.json` | 运行时配置 |
| `faultloggerd.para` | `services/config/faultloggerd.para` | `/system/etc/param/faultloggerd.para` | 参数配置 |
| `faultloggerd.para.dac` | `services/config/faultloggerd.para.dac` | `/system/etc/param/faultloggerd.para.dac` | DAC 权限 |
| `faultlogger.conf` | `services/config/faultlogger.conf` | `/system/etc/faultlogger.conf` | 日志显示配置 |
| `fault_coredump.json` | `services/config/fault_coredump.json` | `/system/etc/fault_coredump.json` | Coredump UID 白名单 |

## 编译示例

### 构建 faultloggerd 服务

```bash
# 标准 OpenHarmony 构建
./build.sh --product-name rk3568 --ccache --build-target faultloggerd
```

### 构建 processdump 工具

```bash
./build.sh --product-name rk3568 --ccache --build-target processdump
```

### 清理构建

```bash
gn clean out/rk3568
```

## TODO

- [ ] 📋 补充每个目标的详细 configs 定义
- [ ] 📋 补充条件编译选项文档
- [ ] 📋 分析构建产物大小

## 相关跳转

- [编译产物](06_Build_Artifacts.md) - 最终产物和安装路径
- [目录结构与模块职责](01_Directory_Structure.md) - 代码组织
