# 附录：配置开关

本文档描述 ArkCompiler ETS Runtime 的关键宏定义和 Feature Flags，用于控制编译选项和运行时行为。

## 编译时配置

### 1. 构建模式配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ark_standalone_build` | false | 独立构建模式 |
| `ark_hybrid` | false | 混合模式（ETS/JS） |
| `ark_js_hybrid` | true | JS/ETS 混合执行 |
| `ark_compile_mode` | debug | 编译模式（debug/release） |

### 2. GC 配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_cms_gc` | false | 启用并发标记清除 GC |
| `ets_runtime_enable_cmc_gc` | false | 启用 CMC 混合 GC |
| `enable_cow_array` | true | 启用 COW 数组优化 |

### 3. 国际化配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_ark_intl` | true | 使用 ICU 实现国际化 |
| `ARK_SUPPORT_INTL` | - | 国际化支持宏（条件编译） |

### 4. 调试与诊断配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_dump_in_faultlog` | false | 启用故障日志 |
| `enable_bytrace` | false | 启用 bytrace 追踪 |
| `enable_hitrace` | false | 启用 hitrace |
| `enable_hilog` | true | 启用日志 |
| `enable_hisysevent` | false | 启用系统事件 |
| `enable_unwinder` | false | 启用栈展开 |
| `enable_backtrace_local` | false | 启用本地回溯 |
| `enable_async_stack` | false | 启用异步栈追踪 |

### 5. 性能分析配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ark_profiler_features` | - | 性能分析器特性列表 |
| `ECMASCRIPT_SUPPORT_CPUPROFILER` | - | CPU 分析器 |
| `ECMASCRIPT_SUPPORT_HEAPPROFILER` | - | 堆分析器 |
| `ECMASCRIPT_SUPPORT_HEAPSAMPLING` | - | 堆采样 |
| `ECMASCRIPT_SUPPORT_SNAPSHOT` | - | 快照支持 |
| `ECMASCRIPT_SUPPORT_TRACING` | - | 追踪支持 |
| `ECMASCRIPT_SUPPORT_DEBUGGER` | - | 调试器支持 |

## 运行时 Feature Flags

### 1. PGO 优化配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ets_runtime_feature_enable_pgo` | false | 启用 PGO 优化 |
| `ets_runtime_feature_pgo_path` | "" | PGO 数据路径 |

### 2. 代码优化配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_next_optimization` | true | 下一代优化 |
| `enable_latest_optimization` | true | 最新优化 |
| `enable_asm_assert` | false | 汇编断言 |
| `ets_runtime_feature_enable_codemerge` | false | 代码合并 |
| `ets_runtime_feature_enable_inst_prefetch` | false | 指令预取 |

### 3. JIT 配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ets_runtime_support_jit_code_sign` | false | JIT 代码签名 |
| `enable_jit_code_sign` | false | 启用 JIT 签名 |
| `disable_fort_switch` | false | 禁用 Fort 切换 |

### 4. AOT 配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ets_runtime_feature_enable_list` | false | 启用 AOT 名单 |
| `enable_target_compilation` | false | 目标编译 |

## 平台特定配置

### 1. Linux 平台

```gn
if (is_linux) {
  defines += [
    "PANDA_TARGET_PREVIEW",
    "PANDA_TARGET_UNIX",
    "PANDA_TARGET_LINUX",
    "PANDA_USE_FUTEX",
  ]
}
```

### 2. Windows 平台

```gn
if (is_mingw) {
  defines += [
    "PANDA_TARGET_PREVIEW",
    "PANDA_TARGET_WINDOWS",
    "_CRTBLD",
    "__LIBMSVCRT__",
  ]
}
```

### 3. macOS 平台

```gn
if (is_mac) {
  defines += [
    "PANDA_TARGET_PREVIEW",
    "PANDA_TARGET_UNIX",
    "PANDA_TARGET_MACOS",
  ]
}
```

### 4. OpenHarmony 平台

```gn
if (is_ohos) {
  defines += [
    "PANDA_TARGET_OHOS",
    "ENABLE_COLD_STARTUP_GC_POLICY",
  ]
}
```

## 架构特定配置

### 1. ARM64 架构

```gn
if (current_cpu == "arm64") {
  defines += [
    "PANDA_TARGET_ARM64",
    "PANDA_TARGET_64",
    "PANDA_ENABLE_GLOBAL_REGISTER_VARIABLES",
    "PANDA_USE_32_BIT_POINTER",
    "ENABLE_POSTFORK_FORCEEXPAND",
    "ENABLE_HISPEED_PLUGIN",
  ]
}
```

### 2. ARM32 架构

```gn
if (current_cpu == "arm") {
  defines += [
    "PANDA_TARGET_ARM32_ABI_SOFT=1",
    "PANDA_TARGET_ARM32",
    "PANDA_TARGET_32",
  ]
}
```

### 3. x86_64 架构

```gn
if (current_cpu == "amd64" || current_cpu == "x64" ||
    current_cpu == "x86_64") {
  defines += [
    "PANDA_TARGET_64",
    "PANDA_TARGET_AMD64",
    "PANDA_USE_32_BIT_POINTER",
  ]
}
```

## 安全相关配置

### 1. 代码加密

```gn
# 代码加密启用（条件编译）
if (code_encryption_enable) {
  defines += [ "CODE_ENCRYPTION_ENABLE" ]
}
```

### 2. ASAN 运行时

```gn
# ASAN 检测
if (is_asan) {
  defines += [
    "ECMASCRIPT_ENABLE_ASAN_DFX_CONFIG",
    "ECMASCRIPT_ENABLE_ASAN_THREAD_CHECK",
  ]
}

# 使用 ASAN 构建
if (use_hwasan) {
  defines += [ "USE_HWASAN" ]
} else {
  defines += [ "USE_ASAN" ]
}
```

### 3. 代码签名

```gn
# JIT 代码签名
if (enable_jit_code_sign) {
  defines += [ "JIT_ENABLE_CODE_SIGN" ]
  if (disable_fort_switch) {
    defines += [ "JIT_FORT_DISABLE" ]
  }
}
```

## 实验性功能配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_litecg_emit` | true | 启用轻量代码生成 |
| `enable_asm_interp` | - | 启用汇编解释器 |
| `enable_fastverify` | false | 启用快速验证 |
| `enable_lto_O0` | false | 禁用 LTO 优化 |
| `enable_gc_dfx_options` | false | 启用 GC 诊断选项 |

## 相关文档

- [构建系统](../06_Build_System.md)
- [架构说明](../03_Architecture.md)
- [内存管理](../03_Architecture.md#内存布局)
