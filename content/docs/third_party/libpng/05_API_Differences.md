# API 差异分析

本文档记录 libpng 在 OpenHarmony 中的 API 变更、新增接口和禁用功能。

---

## 5.1 API 变更概览

### 1.1 新增 API

OpenHarmony 版本的 libpng 基于上游 1.6.44，**未公开新增 API**。所有优化均为内部实现，不暴露新接口。

### 1.2 修改的 API

| API | 修改类型 | 修改内容 |
|-----|----------|----------|
| 无 | 无 | 上游 API 完全保持兼容 |

### 1.3 禁用的功能

| 功能 | 禁用原因 | 影响 |
|------|----------|------|
| ARM NEON 自动检测 | 改用显式编译时定义 | 需要手动定义 `PNG_ARM_NEON` |

---

## 5.2 配置宏差异

### 5.2.1 OH 特有关联配置宏

| 宏名称 | 定义位置 | 默认值 | 说明 |
|--------|----------|--------|------|
| `PNG_ARM_NEON` | BUILD.gn | 未定义 | ARM NEON 优化开关（OH 显式定义） |
| `PNG_MULTY_LINE_ENABLE` | 内部 | 未定义 | 多行解码优化开关 |
| `PNG_ALIGNED_MEMORY_SUPPORTED` | 系统 | 依赖平台 | 内存对齐支持 |

### 5.2.2 ARM NEON 条件编译

**上游原版**:
```c
// pngpriv.h
#if (defined(__ARM_NEON__) || defined(__ARM_NEON)) && \
    defined(PNG_ALIGNED_MEMORY_SUPPORTED)
#  define PNG_ARM_NEON_OPT 2
#endif
```

**OH 修改版**:
```c
// pngpriv.h (libpng-fix-arm-neon.patch)
#if defined(PNG_ARM_NEON) && (defined(__ARM_NEON__) || defined(__ARM_NEON)) && \
    defined(PNG_ALIGNED_MEMORY_SUPPORTED)
#  define PNG_ARM_NEON_OPT 2
#endif
```

**差异说明**:
- 上游：自动检测 CPU 能力
- OH：需要显式定义 `PNG_ARM_NEON` 宏才能启用优化

**优点**:
- 精确控制优化启用/禁用
- 避免运行时检测开销
- 便于在不同平台上统一配置

**缺点**:
- 需要在 BUILD.gn 中正确配置
- 非 ARM 平台可能意外启用

---

## 5.3 内部函数扩展

### 5.3.1 新增 NEON 优化函数

libpng_optimize.patch 在内部添加了以下 NEON 优化函数（不对外公开）：

```c
// arm/filter_neon_intrinsics.c

// UP 滤波器扩展
void png_read_filter_row_up_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// UP 滤波器双行版本
void png_read_filter_row_up_x2_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// AVG 滤波器扩展（RGB）
void png_read_filter_row_avg3_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// AVG 滤波器双行版本（RGB）
void png_read_filter_row_avg3_x2_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// AVG 滤波器扩展（RGBA）
void png_read_filter_row_avg4_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// AVG 滤波器双行版本（RGBA）
void png_read_filter_row_avg4_x2_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// PAETH 滤波器扩展（RGB）
void png_read_filter_row_paeth3_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// PAETH 滤波器双行版本（RGB）
void png_read_filter_row_paeth3_x2_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// PAETH 滤波器扩展（RGBA）
void png_read_filter_row_paeth4_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);

// PAETH 滤波器双行版本（RGBA）
void png_read_filter_row_paeth4_x2_neon(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row);
```

### 5.3.2 新增批量行处理函数

```c
// pngpread.c

// 渐进式读取模式下的双行处理
static void png_push_process_row_x2(png_structrp png_ptr,
   png_row_info row_info_in);

// 批量行处理初始化（PNG_MULTY_LINE_ENABLE）
static void png_init_push_multiline(png_structrp png_ptr);
```

### 5.3.3 内部工具函数

```c
// paeth 预测器 NEON 实现
static uint8x8_t paeth(uint8x8_t a, uint8x8_t b, uint8x8_t c);
```

---

## 5.4 数据结构变更

### 5.4.1 无公开结构体变更

libpng 的公开数据结构（`png_struct`, `png_info`, `png_image` 等）在 OH 版本中**保持完全兼容**。

### 5.4.2 内部结构体扩展

```c
// pngstruct.h 或 pngpriv.h 中的扩展

// 滤波器函数指针表扩展（arm_init.c 中）
typedef void (*png_read_filter_ptr)(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row, int bpp);

// 新增双行滤波器函数指针
extern void png_read_filter_row_up_x2_neon(png_row_infop row_info,
   png_bytep row, png_const_bytep prev_row);
// ... 其他双行函数
```

---

## 5.5 构建配置差异

### 5.5.1 编译选项差异

| 选项 | 上游 CMake | OH GN |
|------|------------|-------|
| ARM NEON | 自动检测 | `PNG_ARM_NEON` 定义 |
| 符号隐藏 | 可选 | `-fvisibility=hidden`（静态库） |
| 警告抑制 | 可配置 | `-Wno-implicit-fallthrough` |

### 5.5.2 输出目标差异

| 目标 | 上游 | OH |
|------|------|-----|
| 共享库 | `libpng.so` / `libpng.framework` | `libpng.so` |
| 静态库 | `libpng.a` | `libpng_static.a` |
| 源码集 | 不支持 | `png_static` |

---

## 5.6 兼容性保证

### 5.6.1 ABI 兼容性

libpng OH 版本保证：
- ✅ 公开 API 100% 兼容上游 1.6.44
- ✅ 头文件接口完全一致
- ✅ 二进制接口（符号表）兼容

### 5.6.2 迁移建议

从其他 libpng 版本迁移到 OH 版本：

**API 兼容层**:
```c
// 原有代码无需修改
#include "png.h"

// 所有 API 调用保持不变
png_image image;
png_image_begin_read_from_file(&image, "test.png");
```

**需要注意的配置**:
```c
// 如果需要 ARM NEON 优化，在编译时添加
#ifdef __arm__
// 或在 BUILD.gn 中定义
defines = [ "PNG_ARM_NEON" ]
#endif
```

---

## 5.7 行为差异

### 5.7.1 NEON 优化行为

| 行为 | 上游 | OH |
|------|------|-----|
| NEON 检测 | 运行时自动检测 | 编译时定义 |
| 不支持 NEON 的芯片 | 自动回退 | 需确保未定义宏 |
| 性能 | 取决于运行时 | 取决于编译配置 |

### 5.7.2 内存对齐

| 平台 | 上游 | OH |
|------|------|-----|
| ARM | 自动处理 | 自动处理 |
| x86 | 不适用 | 不适用 |

---

## 5.8 文档差异

### 5.8.1 API 文档

libpng OH 版本使用上游文档：
- [官方 API 手册](http://www.libpng.org/pub/png/libpng-manual.html)
- [PNG 规范](http://www.w3.org/TR/PNG/)

### 5.8.2 OH 特有文档

本文档（wiki）提供 OH 特有的配置说明：
- BUILD.gn 集成指南
- ARM NEON 优化启用方法
- 性能调优建议

---

## 5.9 总结

libpng OH 版本与上游版本的 API 差异极小，主要差异体现在：

| 差异点 | 影响 | 建议 |
|--------|------|------|
| ARM NEON 编译定义 | 性能优化 | 按需启用 |
| 构建系统集成 | 构建配置 | 使用标准 BUILD.gn |
| 内部优化函数 | 性能提升 | 无需关注 |

**结论**: 开发者可以完全按照上游 libpng 文档使用 OH 版本，仅在需要 ARM NEON 优化时需要关注构建配置。
