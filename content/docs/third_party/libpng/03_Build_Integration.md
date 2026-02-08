# OH 构建适配

本文档详细说明 libpng 在 OpenHarmony 构建系统中的集成方式、编译配置和特殊处理。

---

## 3.1 构建架构概述

### 3.1.1 构建流程

```
libpng-1.6.44.tar.gz
        ↓
    [libpng_action]  (Python 脚本)
        ↓
    解压 + Patch 应用
        ↓
    生成源码文件 (png.c, pngread.c, 等)
        ↓
    [libpng / png_static / libpng_static]
        ↓
    静态库/动态库输出
```

### 3.1.2 构建入口

**位置**: `//third_party/libpng/BUILD.gn`

**构建目标**:
| 目标名称 | 类型 | 说明 |
|----------|------|------|
| `libpng` | shared_library | 动态链接库 |
| `png_static` | source_set | 源码集（直接引用） |
| `libpng_static` | static_library | 静态库 |

---

## 3.2 核心构建配置

### 3.2.1 libpng_action 配置

```gn
action("libpng_action") {
  script = "//third_party/libpng/install.py"
  
  outputs = [
    "${target_gen_dir}/libpng-1.6.44/png.c",
    "${target_gen_dir}/libpng-1.6.44/pngerror.c",
    "${target_gen_dir}/libpng-1.6.44/pngget.c",
    "${target_gen_dir}/libpng-1.6.44/pngmem.c",
    "${target_gen_dir}/libpng-1.6.44/pngpread.c",
    "${target_gen_dir}/libpng-1.6.44/pngread.c",
    "${target_gen_dir}/libpng-1.6.44/pngrio.c",
    "${target_gen_dir}/libpng-1.6.44/pngrtran.c",
    "${target_gen_dir}/libpng-1.6.44/pngrutil.c",
    "${target_gen_dir}/libpng-1.6.44/pngset.c",
    "${target_gen_dir}/libpng-1.6.44/pngtrans.c",
    "${target_gen_dir}/libpng-1.6.44/pngwio.c",
    "${target_gen_dir}/libpng-1.6.44/pngwrite.c",
    "${target_gen_dir}/libpng-1.6.44/pngwtran.c",
    "${target_gen_dir}/libpng-1.6.44/pngwutil.c",
    "${target_gen_dir}/libpng-1.6.44/arm/arm_init.c",
    "${target_gen_dir}/libpng-1.6.44/arm/filter_neon_intrinsics.c",
    "${target_gen_dir}/libpng-1.6.44/arm/palette_neon_intrinsics.c",
  ]

  inputs = [ "//third_party/libpng/libpng-1.6.44.tar.gz" ]

  inputs += [
    "backport-libpng-1.6.37-enable-valid.patch",
    "CVE-2018-14048.patch",
    "CVE-2019-6129.patch",
    "huawei_libpng_CMakeList.patch",
    "libpng-fix-arm-neon.patch",
    "libpng-multilib.patch",
    "libpng_optimize.patch",
  ]

  args = [
    "--gen-dir",
    "$libpng_path",
    "--source-dir",
    "$libpng_source_path",
  ]
}
```

### 3.2.2 编译配置

#### 通用配置

```gn
config("libpng_config") {
  include_dirs = [ "${target_gen_dir}/libpng-1.6.44/" ]
}
```

#### 警告抑制配置

```gn
config("libpng_wno_config") {
  cflags = [ "-Wno-implicit-fallthrough" ]
  if (target_platform == "pc") {
    if (is_ohos && is_clang &&
        (target_cpu == "arm" || target_cpu == "arm64")) {
      ldflags = [ "-Wl,-Bsymbolic" ]
      defines = [ "PNG_ARM_NEON" ]
    }
  }
}
```

#### 静态库符号隐藏配置

```gn
config("libpng_wno_config_static") {
  if (target_platform == "pc") {
    if (is_ohos && is_clang &&
        (target_cpu == "arm" || target_cpu == "arm64")) {
      cflags = [ "-fvisibility=hidden" ]
    }
  }
}
```

---

## 3.3 库目标定义

### 3.3.1 共享库 (libpng)

```gn
ohos_shared_library("libpng") {
  sources = get_target_outputs(":libpng_action")
  include_dirs = [ "${target_gen_dir}/libpng-1.6.44/" ]
  
  deps = [ ":libpng_action" ]
  if (is_arkui_x) {
    deps += [ "//third_party/zlib:libz" ]
  } else {
    external_deps = [ "zlib:libz" ]
  }

  public_configs = [ ":libpng_config" ]
  configs = [ ":libpng_wno_config" ]
  
  innerapi_tags = [
    "platformsdk",
    "chipsetsdk",
  ]
  
  subsystem_name = "thirdparty"
  part_name = "libpng"
  
  install_images = [
    "system",
    "updater",
  ]
  
  install_enable = true
  output_name = "libpng"
}
```

### 3.3.2 静态库 (libpng_static)

```gn
ohos_static_library("libpng_static") {
  visibility = [
    ":*",
    "//foundation/arkui/ui_ext_lite/tools/ide/*",
    "//foundation/arkui/ui_lite/ext/ide/*",
    "//developtools/global_resource_tool/*",
    "//third_party/freetype/*",
    "//third_party/skia/m133/*",
    "//out/*",
    "//vendor/hisi/confidential/contexthub/src/framework/hisi/ui/third_party/*",
  ]
  
  sources = get_target_outputs(":libpng_action")
  include_dirs = [ "${target_gen_dir}/libpng-1.6.44/" ]
  
  deps = [ ":libpng_action" ]
  if (is_arkui_x) {
    deps += [ "//third_party/zlib:libz" ]
  } else {
    external_deps = [ "zlib:libz" ]
  }
  
  public_configs = [ ":libpng_config" ]
  configs = [ ":libpng_wno_config" ]
  configs += [ ":libpng_wno_config_static" ]
  
  subsystem_name = "thirdparty"
  part_name = "libpng"
}
```

### 3.3.3 源码集 (png_static)

```gn
ohos_source_set("png_static") {
  sources = get_target_outputs(":libpng_action")
  include_dirs = [ "${target_gen_dir}/libpng-1.6.44/" ]
  
  external_deps = [ "zlib:libz" ]
  deps = [ ":libpng_action" ]
  
  public_configs = [ ":libpng_config" ]
  
  part_name = "libpng"
  subsystem_name = "thirdparty"
}
```

---

## 3.4 LiteOS 适配

### 3.4.1 Lite 构建分支

```gn
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")

  config("libpng_config") {
    include_dirs = [ "${target_gen_dir}/libpng-1.6.44/" ]
  }

  libpng_source = get_target_outputs(":libpng_action")

  lite_library("libpng") {
    if (ohos_kernel_type == "liteos_m") {
      target_type = "static_library"
      deps = [ "//build/lite/config/component/zlib:zlib_static" ]
    } else {
      target_type = "shared_library"
      deps = [ "//build/lite/config/component/zlib:zlib_shared" }
    }
    
    deps += [ ":libpng_action" ]
    
    # IAR 编译器特殊处理
    if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
      cflags = [
        "--diag_suppress",
        "Pa082,Pa084",
      ]
      cflags_cc = cflags
    }
    
    sources = libpng_source
    public_configs = [ ":libpng_config" ]
  }
}
```

### 3.4.2 LiteOS 编译参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--diag_suppress Pa082` | IAR 编译器警告 | 禁止隐式函数声明警告 |
| `--diag_suppress Pa084` | IAR 编译器警告 | 禁止隐式转换警告 |

---

## 3.5 特殊编译选项

### 3.5.1 ARM NEON 支持

```gn
defines = [ "PNG_ARM_NEON" ]
ldflags = [ "-Wl,-Bsymbolic" ]
cflags = [ "-fvisibility=hidden" ]
```

**说明**:
- `PNG_ARM_NEON`: 启用 ARM NEON 优化代码路径
- `-Wl,-Bsymbolic`: 减少动态链接时的符号解析开销
- `-fvisibility=hidden`: 隐藏静态库中的符号，减少冲突风险

### 3.5.2 警告抑制

```gn
cflags = [ "-Wno-implicit-fallthrough" ]
```

**说明**: 抑制 `switch` 语句缺少 `break` 导致的警告（某些故意 fallthrough 的代码）

---

## 3.6 安装配置

### 3.6.1 安装目标

```gn
install_images = [
  "system",
  "updater",
]
```

| 镜像 | 用途 |
|------|------|
| system | 系统分区，包含核心系统库 |
| updater | 升级分区，用于 OTA 升级 |

### 3.6.2 API 标签

```gn
innerapi_tags = [
  "platformsdk",
  "chipsetsdk",
]
```

| 标签 | 说明 |
|------|------|
| platformsdk | 平台 SDK 内部 API |
| chipsetsdk | 芯片 SDK 内部 API |

---

## 3.7 依赖关系

### 3.7.1 外部依赖

| 依赖 | 类型 | 配置方式 |
|------|------|----------|
| zlib | 必需 | `external_deps = [ "zlib:libz" ]` |

### 3.7.2 内部依赖

| 依赖 | 类型 |
|------|------|
| libpng_action | 构建时依赖（生成源码） |

---

## 3.8 与上游构建系统的差异

| 特性 | 上游 CMake | OH GN |
|------|------------|-------|
| 源码生成 | configure/makefile | install.py |
| ARM NEON | 自动检测 | 显式 `PNG_ARM_NEON` 定义 |
| 多库支持 | 独立配置 | libpng-multilib.patch |
| 安装路径 | cmake install | install_images |
| ABI 兼容 | pkg-config | innerapi_tags |

---

## 3.9 构建问题排查

### 常见问题

#### 问题 1: ARM NEON 符号未定义

**症状**: 链接时出现 `undefined reference to 'png_read_filter_row_up_neon'`

**解决方案**: 确保在 ARM/ARM64 平台上添加 `PNG_ARM_NEON` 定义。

```gn
if (is_ohos && is_clang && (target_cpu == "arm" || target_cpu == "arm64")) {
  defines += [ "PNG_ARM_NEON" ]
}
```

#### 问题 2: zlib 链接失败

**症状**: 链接时出现 `undefined reference to 'zlibVersion'`

**解决方案**: 确保正确配置 zlib 依赖。

```gn
external_deps = [ "zlib:libz" ]
```

#### 问题 3: IAR 编译器警告

**症状**: 编译时出现 `Pa082` 或 `Pa084` 警告

**解决方案**: 添加编译器抑制参数。

```gn
if (board_toolchain_type == "iccarm") {
  cflags = [ "--diag_suppress", "Pa082,Pa084" ]
}
```

---

## 3.10 构建性能优化

### 3.10.1 建议优化配置

| 配置 | 建议值 | 说明 |
|------|--------|------|
| 并行编译 | 启用 | libpng 源文件可并行编译 |
| 预编译头 | 不适用 | libpng 为 C 项目 |
| 增量编译 | 启用 | Patch 更新时才需重编译 |

### 3.10.2 缓存建议

libpng 源码在 `${target_gen_dir}/libpng-1.6.44/` 生成后会被缓存，Patch 更新时需要重新应用。
