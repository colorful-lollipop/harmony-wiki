# 构建系统

本文档描述 ArkCompiler ETS Runtime 的 GN 构建系统，包括关键 Targets、依赖关系和编译产物。

## 构建系统概述

ArkCompiler ETS Runtime 使用 GN（Generate Ninja）作为构建系统，与 OpenHarmony 整体构建系统集成。

### 构建入口

- **根构建文件**：`arkcompiler/ets_runtime/BUILD.gn`
- **配置模板**：`arkcompiler/ets_runtime/js_runtime_config.gni`

### 构建命令

```bash
# Linux 工具链构建
./build.sh --product-name hispark_taurus_standard --build-target ark_js_host_linux_tools_packages

# 设备构建
./build.sh --product-name <product_name> --build-target ets_runtime

# 独立构建
gn gen out/<target> --args="ark_standalone_build=true"
ninja -C out/<target>
```

## 主要 Targets

### 运行时库 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libark_jsruntime` | ohos_shared_library | `.so` | 核心运行时库 |
| `libark_jsruntime_static` | ohos_static_library | `.a` | 静态库（可选） |
| `ark_js_unittest` | group | - | 单元测试组 |

### 工具 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `ark_js_vm` | ohos_executable | 可执行文件 | 命令行工具 |
| `ark_aot_compiler` | ohos_executable | 可执行文件 | AOT 编译器 |
| `quick_fix` | ohos_executable | 可执行文件 | 快速修复工具 |
| `profdump` | ohos_executable | 可执行文件 | 性能分析转储工具 |

### 编译器 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `ark_aot_compiler` | executable | 可执行文件 | AOT 编译器 |
| `ark_stub_compiler` | executable | 可执行文件 | Stub 编译器 |
| `libark_jsoptimizer` | shared_library | `.so` | JS 优化器库 |

## Target 依赖关系

### libark_jsruntime 依赖

```gn
ohos_shared_library("libark_jsruntime") {
  sources = [...]

  public_configs = [
    "$js_root:ark_jsruntime_public_config",
    "$ark_root/common_interfaces:common_interfaces_public_config",
  ]

  configs = [
    "$js_root:ark_jsruntime_common_config",
    "$js_root:asm_interp_enable_config",
  ]

  external_deps = [
    "libuv:uv",
    "zlib:shared_libz",
  ]

  deps = [
    "$ark_root/libpandafile:arkfile_header_deps",
  ]
}
```

### 依赖关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                    Target 依赖关系图                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   libark_jsruntime ──────────────────────────────────────┐     │
│       │                                                    │     │
│       ├──► libuv                                         │     │
│       ├──► zlib                                          │     │
│       ├──► arkfile_header_deps                           │     │
│       └──► common_interfaces                             │     │
│                                                            │     │
│   ark_js_vm ──────────────────────────────────────────────┤     │
│       │                                                    │     │
│       ├──► libark_jsruntime                               │     │
│       ├──► icu                                            │     │
│       └──► libuv                                          │     │
│                                                            │     │
│   ark_aot_compiler ───────────────────────────────────────┤     │
│       │                                                    │     │
│       ├──► libark_jsruntime                               │     │
│       └──► LLVM                                           │     │
│                                                            │     │
└─────────────────────────────────────────────────────────────────┘
```

## 编译产物

### 运行时库产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libark_jsruntime.so` | `out/.../arkcompiler/ets_runtime/` | 核心运行时动态库 |
| `libark_jsruntime.z.so` | `out/.../arkcompiler/ets_runtime/` | 代码加密版本（可选） |

### 工具产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `ark_js_vm` | `out/.../arkcompiler/ets_runtime/` | 命令行执行工具 |
| `ark_aot_compiler` | `out/.../arkcompiler/ets_runtime/` | AOT 编译器 |
| `quick_fix` | `out/.../arkcompiler/ets_runtime/` | 热修复工具 |
| `profdump` | `out/.../arkcompiler/ets_runtime/` | 性能分析工具 |

### 安装路径

在 OpenHarmony 标准系统中：

| 产物 | 安装路径 |
|------|----------|
| `libark_jsruntime.so` | `/system/lib64/module/arkcompiler/` |
| `ark_aot_compiler` | `/system/bin/` |

## 配置开关

### 运行时特性开关

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `ark_hybrid` | false | 启用混合模式 |
| `ark_js_hybrid` | true | 启用 JS/ETS 混合 |
| `enable_ark_intl` | true | 启用国际化支持 |
| `enable_cms_gc` | false | 启用 CMS GC |
| `ets_runtime_enable_cmc_gc` | false | 启用 CMC GC |

### 编译优化开关

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_next_optimization` | true | 启用下一代优化 |
| `enable_latest_optimization` | true | 启用最新优化 |
| `enable_asm_assert` | false | 启用汇编断言 |
| `enable_cow_array` | true | 启用 COW 数组 |

### 调试与诊断开关

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `enable_dump_in_faultlog` | false | 启用故障日志 |
| `enable_bytrace` | false | 启用 bytrace |
| `enable_hitrace` | false | 启用 hitrace |
| `enable_hilog` | true | 启用日志 |
| `enable_gc_dfx_options` | false | 启用 GC 诊断 |

## 产物与运行时加载关系

### 动态库加载

运行时依赖以下动态库的加载顺序：

```bash
# 设置库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:\
  out/.../arkcompiler/ets_runtime:\
  out/.../thirdparty/icu:\
  prebuilts/clang/ohos/linux-x86_64/llvm/lib

# 运行
./ark_js_vm helloworld.abc
```

### 加载顺序

1. `libark_jsruntime.so` - 核心运行时
2. `libuv.so` - 异步 I/O
3. `libicuuc.so` - 国际化（如果启用）
4. `libz.so` - 压缩库

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
- [内部 API](05_Inner_API.md)
