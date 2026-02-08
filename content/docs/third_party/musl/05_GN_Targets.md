# GN 构建目标

> OpenHarmony musl GN 构建系统详解

---

## 目的与适用范围

**目的**: 详细说明 musl 的 GN 构建配置、targets 和编译选项。

**适用范围**: 系统构建工程师、移植人员。

---

## 构建文件概览

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 主构建配置 |
| `musl_src.gni` | 源文件列表 |
| `musl_template.gni` | 构建模板定义 |
| `musl_config.gni` | 编译配置参数 |
| `bundle.json` | OHOS 组件配置 |

---

## 主要 Targets

### 1. 库 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `soft_libc_musl_static` | static_library | `libc.a` | 静态库 |
| `soft_libc_musl_shared` | shared_library | `libc.so` | 动态库 |
| `soft_libc_musl_shared_sp` | shared_library | `libc.so` (sp) | 强保护版本 |
| `soft_libm` | static_library | `libm.a` | 数学库 |
| `soft_libpthread` | static_library | `libpthread.a` | 线程库 |
| `soft_libdl` | static_library | `libdl.a` | 动态链接库 |
| `soft_librt` | static_library | `librt.a` | 实时库 |
| `soft_libcrypt` | static_library | `libcrypt.a` | 加密库 |
| `soft_libresolv` | static_library | `libresolv.a` | DNS 解析库 |
| `soft_libutil` | static_library | `libutil.a` | 工具库 |
| `soft_libxnet` | static_library | `libxnet.a` | X/Open 网络库 |

### 2. 头文件 Targets

| Target | 类型 | 说明 |
|--------|------|------|
| `musl_headers` | ohos_shared_headers | 头文件集合 |
| `create_alltypes_h` | action | 生成 alltypes.h |
| `create_version_h` | action | 生成 version.h |
| `create_syscall_h` | action | 生成 syscall.h |

### 3. 动态链接器 Targets

| Target | 类型 | 说明 |
|--------|------|------|
| `soft_musl_ldso_static` | source_set | 静态链接器代码 |
| `soft_musl_ldso_shared` | source_set | 动态链接器代码 |
| `soft_create_linker` | copy | 创建 ld-musl-*.so.1 |

### 4. 配置 Targets

| Target | 类型 | 说明 |
|--------|------|------|
| `musl_ns_config` | group | Namespace 配置文件 |
| `musl_sysparam` | ohos_prebuilt_etc | 系统参数 |
| `copy_uapi` | group | UAPI 头文件 |

---

## 构建模板

### static_and_shared_libs_template

定义在 `musl_template.gni`，用于生成静态和共享库：

```gn
template("static_and_shared_libs_template") {
  # 参数:
  # - musl_use_gwp_asan: 是否启用 GWP-ASan
  # - musl_use_flto: 是否启用 LTO
  
  # 生成的 targets:
  # - soft_musl_hook_${target_name}
  # - soft_musl_src_${target_name}
  # - soft_musl_ldso_${target_name}
}
```

### musl_libs

定义在 `musl_template.gni:551`，用于组织所有库：

```gn
template("musl_libs") {
  # 生成:
  # - soft_libc_musl_static
  # - soft_libc_musl_shared
  # - soft_libc_musl_shared_sp (aarch64)
  # - 各种静态库 (libm, libpthread, 等)
  # - soft_musl_crt_libs (C 运行时)
}
```

---

## 编译配置

### 1. 架构配置 (musl_config.gni)

```gn
# 支持的架构
if (current_cpu == "arm") {
  musl_arch = "arm"
} else if (current_cpu == "arm64") {
  musl_arch = "aarch64"
} else if (current_cpu == "x86_64") {
  musl_arch = "x86_64"
} else if (current_cpu == "mipsel") {
  musl_arch = "mips"
} else if (current_cpu == "riscv64") {
  musl_arch = "riscv64"
} else if (current_cpu == "loongarch64") {
  musl_arch = "loongarch64"
}
```

### 2. 特性开关

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `musl_use_gwp_asan` | false | 启用 GWP-ASan |
| `musl_use_jemalloc` | false | 使用 jemalloc |
| `musl_use_mutex_wait_opt` | false | 互斥锁等待优化 |
| `musl_secure_level` | 1 | 内存安全级别 (0-3) |
| `musl_ld128_flag` | x86_64: true | 长双精度支持 |
| `musl_use_flto` | false | 启用 LTO |
| `musl_enable_musl_log` | true | 启用日志 |

### 3. 编译标志

```gn
# 基础定义
defines = [
  "__MUSL__",
  "_LIBCPP_HAS_MUSL_LIBC",
  "__BUILD_LINUX_WITH_CLANG",
]

# Hook 相关
if (!is_asan && musl_arch != "mips") {
  defines += [
    "HOOK_ENABLE",
    "OHOS_SOCKET_HOOK_ENABLE",
  ]
}

# 安全级别
if (musl_secure_level > 0) {
  defines += ["MALLOC_FREELIST_HARDENED"]
}
if (musl_secure_level > 1) {
  defines += ["MALLOC_FREELIST_QUARANTINE"]
}
if (musl_secure_level > 2) {
  defines += ["MALLOC_RED_ZONE"]
}
if (is_debug || musl_secure_level >= 3) {
  defines += ["MALLOC_SECURE_ALL"]
}
```

### 4. 编译选项

```gn
cflags = [
  "-O3",
  "-fPIC",
  "-fstack-protector-strong",
]

# LTO
if (__use_flto) {
  cflags += ["-flto"]
}

# 架构特定
if (musl_arch == "aarch64") {
  cflags += ["-mbranch-protection=pac-ret+b-key"]
}
```

---

## 源文件组织

### musl_src.gni

定义所有源文件：

```gn
# 架构特定源文件
musl_src_arch_file = [
  "src/fenv/${musl_arch}/fenv.s",
  "src/ldso/${musl_arch}/dlsym.s",
  # ... 更多
]

# 通用源文件
musl_src_file = [
  "src/internal/pthread_impl.h",
  "src/aio/aio.c",
  "src/complex/cabs.c",
  # ... 更多 (约 1600+ 文件)
]

# 动态链接器源文件
musl_src_ldso = [
  "ldso/dlstart.c",
  "ldso/linux/dynlink.c",
  # ... 更多
]
```

---

## 依赖关系

### 库依赖图

```
soft_libc_musl_shared
    ├── soft_musl_src_shared
    ├── soft_musl_ldso_shared
    ├── soft_musl_hook_shared
    ├── soft_musl_src_nossp
    ├── soft_musl_src_optimize
    ├── soft_libdl
    └── soft_libpthread

soft_libc_musl_static
    ├── soft_musl_src_static
    ├── soft_musl_hook_static
    ├── soft_musl_src_nossp
    ├── soft_musl_src_optimize
    └── soft_musl_src_strncpy
```

### 外部依赖

```gn
# 来自 bundle.json
deps: {
  components: [
    "init",
    "bounds_checking_function",
    "FreeBSD",
    "cJSON",
    "optimized_routines",
    "jemalloc"
  ]
}
```

---

## 构建命令示例

### 完整构建

```bash
# 构建所有 musl 目标
gn gen out --args="musl_secure_level=2"
ninja -C out third_party/musl:musl_all
```

### 仅构建静态库

```bash
ninja -C out third_party/musl:soft_libc_musl_static
```

### 仅构建动态库

```bash
ninja -C out third_party/musl:soft_libc_musl_shared
```

### 启用安全特性

```bash
gn gen out --args="
  musl_secure_level=3
  musl_use_gwp_asan=true
"
ninja -C out third_party/musl:musl_libs
```

---

## 关键宏定义

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `__MUSL__` | musl_template.gni:39 | musl 标识 |
| `_LIBCPP_HAS_MUSL_LIBC` | musl_template.gni:40 | libc++ 适配 |
| `HOOK_ENABLE` | musl_template.gni:46 | 启用 Hook |
| `OHOS_SOCKET_HOOK_ENABLE` | musl_template.gni:47 | Socket Hook |
| `USE_GWP_ASAN` | musl_template.gni:14 | GWP-ASan |
| `MALLOC_SECURE_ALL` | musl_template.gni:380 | 最高安全级别 |
| `FEATURE_ICU_LOCALE` | musl_template.gni:351 | ICU 全球化 |

---

## 相关跳转

- [编译产物](06_Build_Artifacts.md) - 输出文件和安装路径
- [目录结构](01_Directory_Structure.md) - 源码组织
- [附录/Config_Flags](appendix/Config_Flags.md) - 完整配置宏列表

