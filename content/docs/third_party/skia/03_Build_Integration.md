# 03. OH 构建集成

> **文档版本**: 1.0
> **最后更新**: 2026-02-08
> **阅读时间**: 20 分钟

---

## 目录

- [1. BUILD.gn 结构说明](#1-buildgn-结构说明)
- [2. OH 特定宏定义](#2-oh-特定宏定义)
- [3. 性能优化配置](#3-性能优化配置)
- [4. 构建变体](#4-构建变体)
- [5. 关键编译选项](#5-关键编译选项)
- [6. 与上游构建系统的差异](#6-与上游构建系统的差异)

---

## 1. BUILD.gn 结构说明

### 1.1 主要构建目标

根目录 `BUILD.gn` 定义了三个主要目标：

| 目标 | 类型 | 路径 | 说明 |
|------|------|------|------|
| `skia_canvaskit` | ohos_shared_library | `//third_party/skia:skia_canvaskit` | 主共享库，完整 Skia 功能 |
| `skia_canvaskit_static` | ohos_static_library | `//third_party/skia:skia_canvaskit_static` | 静态库版本，ArkUI-X 使用 |
| `sksl_ext_static` | ohos_static_library | `//third_party/skia:sksl_ext_static` | SKSL 独立编译库 |

### 1.2 BUILD.gn 文件层次

```
third_party/skia/
├── BUILD.gn                          # OHOS 主构建入口
│   ├── skia_canvaskit                # 共享库目标
│   ├── skia_canvaskit_static        # 静态库目标
│   └── sksl_ext_static               # SKSL 库目标
│
├── build_overrides/
│   └── skia.gni                      # 版本控制（m133 vs 旧版本）
│
├── m133/
│   ├── BUILD.gn                      # Skia 原始构建配置
│   ├── gn/
│   │   ├── skia.gni                 # 主配置模板
│   │   ├── oh_skia.gni              # OHOS 特定配置
│   │   ├── core.gni                 # 核心源文件列表
│   │   └── shared_sources.gni        # 共享源配置
│   └── src/ports/skia_ohos/        # OHOS 实现
│       ├── SkFontMgr_ohos.cpp
│       ├── SkTypeface_ohos.cpp
│       └── config/
│           ├── fontconfig_ohos.json
│           └── fontconfig_ohos_wearable.json
```

### 1.3 构建依赖关系

```
skia_canvaskit (共享库)
├── fontmgr_ohos              # OHOS 字体管理器
├── gpu                      # GPU 后端
├── graphite                 # 新渲染引擎
├── jpeg_encode/decode       # JPEG 编解码
├── png_encode/decode        # PNG 编解码
├── webp_encode/decode      # WebP 编解码
├── skshaper                # 文本塑形
├── skunicode               # Unicode 支持
├── svg                     # SVG 渲染
├── skcms                   # 色彩管理
└── ... (其他模块)
```

---

## 2. OH 特定宏定义

### 2.1 平台标识宏

| 宏 | 定义位置 | 用途 |
|----|---------|------|
| `SK_OHOS_EXTENSION` | BUILD.gn | OHOS 扩展功能总开关 |
| `SKIA_OHOS` | oh_skia.gni | OHOS 平台标识 |
| `CROSS_PLATFORM` | BUILD.gn | 跨平台支持（ArkUI-X） |
| `ARKUI_X_ENABLE` | BUILD.gn | ArkUI-X 框架启用 |

### 2.2 功能特性宏

| 宏 | 定义位置 | 用途 |
|----|---------|------|
| `SK_ENABLE_SDF_BLUR_SWITCH` | BUILD.gn | SDF 模糊开关 |
| `SK_ENABLE_PATH_COMPLEXITY_DFX` | BUILD.gn | 路径复杂度 DFX 调试 |
| `SK_ENABLE_OHOS_CODEC` | BUILD.gn | OHOS 特定编解码器 |
| `SK_HAS_ANDROID_CODEC` | oh_skia.gni | Android 编解码器支持 |
| `SK_GL` | oh_skia.gni | OpenGL 支持 |
| `SK_ENABLE_SVG` | oh_skia.gni | SVG 支持 |

### 2.3 性能/调试宏

| 宏 | 定义位置 | 用途 |
|----|---------|------|
| `SKIA_OHOS_SHADER_REDUCE` | SkDebug_ohos.cpp | Shader 优化缩减 |
| `SKIA_DFX_FOR_OHOS` | SkDebug_ohos.cpp | DFX 调试功能 |
| `SKIA_DFX_FOR_RECORD_VKIMAGE` | SkDebug_ohos.cpp | Vulkan 图像记录调试 |
| `SKIA_OHOS_SINGLE_OWNER` | SkDebug_ohos.cpp | 单所有者模式（Render Service） |
| `SK_USE_HISPEED_PLUGIN` | BUILD.gn | 高速插件支持 |
| `SK_BUILD_FOR_DEBUGGER` | BUILD.gn | 调试器构建 |

### 2.4 宏使用示例

```cpp
// SkDebug_ohos.cpp
#ifdef SKIA_OHOS_SINGLE_OWNER
bool GetEnableSkiaSingleOwner()
{
    return IsRenderService() && IsBeta();
}
#endif

#ifdef SK_ENABLE_OHOS_CODEC
// OHOS 特定编解码器
class SkOHOSCodec : public SkCodec { ... };
#endif
```

---

## 3. 性能优化配置

### 3.1 PGO（Profile Guided Optimization）

#### 启用配置
```gn
# bundle.json features
"skia_feature_enable_pgo": true
"skia_feature_pgo_path": "/path/to/pgo/data"
```

#### 编译选项
```gn
if (skia_feature_enable_pgo && enable_enhanced_opt) {
  ldflags += [
    "-Wl,-mllvm,-vectorization-bonus-factor=4",
    "-Wl,-mllvm,-vectorizer-min-trip-count=4",
    "-Wl,--aarch64-inline-plt",
    "-Wl,-mllvm,-enable-partial-inlining",
    "-Wl,-mllvm,-tail-dup-profile-hot-percentile-override=999990",
  ]
}
```

#### Code Merge 优化
```gn
if (skia_feature_enable_codemerge && !is_asan) {
  ldflags += [
    "-Wl,-plugin-opt=-split-machine-functions",
    "-Wl,-mllvm,-fixup-unconditional-branch-unsafely",
    "-Wl,--no-create-thunks-introduced-by-mfs",
    "-Wl,-mllvm,-mfs-psi-cutoff=999500",
    "-Wl,-z,keep-text-section-prefix",
    "-Wl,--symbol-ordering-file=" + rebase_path(
      "${skia_feature_pgo_path}/libskia_canvaskit.txt",
      root_build_dir
    ),
  ]
}
```

### 3.2 LTO（Link Time Optimization）

#### 启用条件
```gn
if (is_ohos && is_clang && (target_cpu == "arm" || target_cpu == "arm64")) {
  ldflags = [
    "-Wl,--lto-O2",
    "-Wl,-mllvm,-wholeprogramdevirt-check=fallback",
    "-Wl,-Bsymbolic",
  ]
}
```

#### 优化效果
- **代码大小**: -10% ~ -20%
- **运行性能**: +5% ~ +15%
- **编译时间**: +2x ~ 3x

### 3.3 JPEG/PNG 优化

#### OH ISSUE 特定优化
```gn
# oh_skia.gni
if (target_platform == "pc" || target_platform == "phone" || target_platform == "tablet") {
  defines += [
    "TURBO_JPEG_HUFF_DECODE_OPT",      # JPEG Huffman 解码优化
    "TURBO_PNG_MULTY_LINE_OPT",       # PNG 多行优化
  ]
}
```

#### 性能提升
- JPEG 解码：+15% ~ +25%
- PNG 解码：+20% ~ +30%

### 3.4 SIMD 优化

#### 自动检测
```cpp
// adler32.c
static inline uint32_t adler32_z(...)
{
    if (arm_has_crc32()) {
        return adler32_armv8(...);  // ARM CRC32 指令
    }
    if (x86_has_sse42()) {
        return adler32_sse42(...);  // SSE4.2 指令
    }
    // 默认实现
}
```

#### 编译选项
```gn
cflags = [
  "-march=armv8-a",           # ARMv8 指令集
  "-mfpu=neon",              # NEON SIMD
  "-mthumb",                 # Thumb 模式
]
```

---

## 4. 构建变体

### 4.1 平台变体

#### ArkUI-X 跨平台
```gn
if (is_arkui_x) {
  defines += [
    "CROSS_PLATFORM",
    "ARKUI_X_ENABLE",
  ]
  public_deps = [ ":skia_canvaskit_static" ]
}
```

#### 手机/平板
```gn
if (target_platform == "phone" || target_platform == "tablet") {
  defines += [
    "TURBO_JPEG_HUFF_DECODE_OPT",
    "TURBO_PNG_MULTY_LINE_OPT",
  ]
}
```

#### 穿戴设备
```gn
if (target_platform == "wearable") {
  # 使用可穿戴字体配置
  font_config = "fontconfig_ohos_wearable.json"
}
```

### 4.2 版本变体

#### M133 版本（当前）
```gn
# build_overrides/skia.gni
if (skia_feature_upgrade) {
  skia_root_dir = "//third_party/skia/m133"
}
```

#### 旧版本（兼容性）
```gn
if (!skia_feature_upgrade) {
  skia_root_dir = "//third_party/skia"
}
```

### 4.3 调试/Release 变体

#### Release
```gn
if (!is_debug) {
  defines += [ "NDEBUG" ]
  cflags += [ "-O2", "-DNDEBUG" ]
}
```

#### Debug
```gn
if (is_debug || skia_build_for_debugger) {
  defines += [ "SK_BUILD_FOR_DEBUGGER" ]
  cflags += [ "-O0", "-g" ]
}
```

#### AddressSanitizer
```gn
if (is_asan) {
  defines += [ "SK_SANITIZE_ADDRESS" ]
  # Code Merge 优化与 ASan 不兼容
  skia_feature_enable_codemerge = false
}
```

---

## 5. 关键编译选项

### 5.1 Defines

#### 核心定义
```gn
defines = skia_common_defines
```

**skia_common_defines 内容**（oh_skia.gni）：
```gn
skia_common_defines = [
  "SK_HAS_ANDROID_CODEC",
  "SK_CODEC_DECODES_JPEG",
  "SK_CODEC_DECODES_PNG",
  "SK_CODEC_DECODES_WEBP",
  "SK_GL",
  "SK_HAS_HEIF_LIBRARY",
  "SK_ENABLE_SVG",
  "SK_SUPPORT_PDF",
  "SKSHAPER_IMPLEMENTATION=1",
  "SK_UNICODE_AVAILABLE",
  "USE_M133_SKIA",
  # ... 更多定义
]
```

### 5.2 Cflags

#### 公共 Cflags
```gn
cflags = [
  "-std=c17",                          # C 标准版本
  "-ffunction-sections",               # 函数段分离
  "-fdata-sections",                  # 数据段分离
  "-fvisibility=hidden",               # 符号隐藏
  "-fno-unwind-tables",               # 无展开表
  "-fno-asynchronous-unwind-tables",   # 无异步展开表
]
```

#### C++ Cflags
```gn
cflags_cc = [
  "-std=c++17",                       # C++17 标准
  "-fvisibility-inlines-hidden",       # 内联符号隐藏
  "-fno-rtti",                       # 禁用 RTTI
]
```

#### 警告抑制
```gn
cflags = [
  "-Wno-ignored-attributes",
  "-Wno-deprecated-declarations",
  "-Wno-pessimizing-move",
  "-Wno-return-type",
  "-Wno-sign-compare",
  "-Wno-unused-function",
  "-Wno-unused-variable",
]
```

### 5.3 Ldflags

#### 链接优化
```gn
if (is_ohos && is_clang && (target_cpu == "arm" || target_cpu == "arm64")) {
  ldflags = [
    "-Wl,--gc-sections",              # 垃圾回收未使用段
    "-Wl,--icf=safe",                # 相同函数合并
  ]
}
```

#### 符号绑定
```gn
ldflags = [
  "-Wl,-Bsymbolic",                   # 符号绑定
]
```

### 5.4 Include_dirs

#### 标准包含路径
```gn
include_dirs = [
  "${skia_root_dir}",
  "${skia_root_dir}/include/core",
  "${skia_root_dir}/src/core",
  "${skia_root_dir}/third_party/externals/harfbuzz/src",
  "${skia_root_dir}/third_party/externals/icu/source/common",
]
```

#### OHOS 特定路径
```gn
include_dirs = [
  "${skia_root_dir}/src/ports/skia_ohos",     # OHOS 端口实现
  "${skia_root_dir}/third_party/externals/piex",  # Piex 库
]
```

---

## 6. 与上游构建系统的差异

### 6.1 构建系统

| 方面 | 上游 | OH |
|------|------|-----|
| **构建工具** | GN/Goma | GN + OH 构建系统 |
| **后端** | 多后端（Bazel, CMake） | 仅 GN |
| **配置文件** | args.gn | bundle.json + .gni |
| **特性开关** | args.gn | bundle.json features |

### 6.2 平台适配

| 方面 | 上游 | OH |
|------|------|-----|
| **平台数量** | 10+ | 重点：Linux/Android + OHOS |
| **字体管理** | 多种实现 | 仅 fontmgr_ohos |
| **日志系统** | 平台默认 | HiLog 集成 |
| **性能追踪** | 可选 | hitrace 集成 |

### 6.3 编译选项

| 方面 | 上游 | OH |
|------|------|-----|
| **C++ 标准** | C++14/C++17 | C++17 |
| **LTO** | 可选 | 默认启用（ARM64） |
| **PGO** | 可选 | 特性开关控制 |
| **SIMD** | 可选 | 自动检测 + 强制优化 |

### 6.4 源码差异

#### OH 独有文件

| 文件 | 用途 |
|------|------|
| `src/ports/SkDebug_ohos.cpp` | HiLog 集成 |
| `src/ports/skia_ohos/` | OHOS 字体管理器 |
| `build_overrides/skia.gni` | 版本控制 |
| `m133/gn/oh_skia.gni` | OHOS 配置 |

#### 修改的上游文件

| 文件 | 修改内容 |
|------|---------|
| `BUILD.gn` | OH 特定构建目标 |
| `m133/gn/skia.gni` | OH 特定配置 |
| `m133/BUILD.gn` | 条件编译（is_ohos） |

---

## 附录

### A. 完整编译示例

```bash
# 编译 Skia 共享库（M133 版本）
hb build -f \
  --target-cpu arm64 \
  --ccache \
  --build-type release \
  --gn-args "skia_feature_upgrade=true" \
  --gn-args "skia_feature_enable_pgo=true" \
  --gn-args "skia_feature_pgo_path=/path/to/pgo"

# 编译 Skia 静态库（ArkUI-X）
hb build -f \
  --target-cpu x86_64 \
  --build-type release \
  --gn-args "is_arkui_x=true"

# 编译 Debug 版本
hb build -f \
  --build-type debug \
  --gn-args "skia_build_for_debugger=true"
```

### B. 构建配置文件

#### bundle.json
```json
{
  "component": {
    "name": "skia",
    "features": [
      "skia_feature_upgrade",
      "skia_feature_enable_pgo",
      "skia_feature_enable_codemerge",
      "skia_feature_hispeed_plugin",
      "skia_feature_use_vulkan"
    ]
  }
}
```

#### oh_skia.gni
```gn
# OHOS 特定配置
skia_common_defines = [
  "SK_OHOS_EXTENSION",
  "SKIA_OHOS",
  "SK_ENABLE_OHOS_CODEC",
  # ... 更多定义
]

# OH 特定编译选项
skia_common_cflags = [
  "-Wno-ignored-attributes",
  "-ffunction-sections",
  "-fdata-sections",
  # ... 更多选项
]
```

---

**下一节**: [04. 依赖关系与使用](04_Usage_in_OH.md)
