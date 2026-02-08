# libpng 原始库简介

## 1.1 库基本信息

**libpng** 是 PNG（Portable Network Graphics）格式的官方参考实现库，由 Group PNG（PNG 开发者在 1995 年成立的技术工作组）维护。该库提供了完整的 PNG 图像读写功能，是理解和使用 PNG 格式的标准参考。

| 属性 | 值 |
|------|-----|
| **当前版本** | 1.6.44 |
| **发布年份** | 1995 年创建，至今已近 30 年 |
| **维护者** | Glenn Randers-Pehrson 等 |
| **许可证** | libpng License（BSD-2-Clause 变体） |
| **上游地址** | http://www.libpng.org/pub/png/libpng.html |

---

## 1.2 核心功能

### 图像读写

libpng 提供了完整的 PNG 图像处理能力：

**读取功能**:
- 从文件、内存、stdio 读取 PNG 图像
- 支持逐行读取和渐进式读取
- 自动解析图像元数据（尺寸、位深、颜色类型等）
- 支持图像变换（颜色空间转换、缩放、格式转换等）

**写入功能**:
- 将图像数据写入 PNG 文件
- 支持多种压缩级别
- 支持图像元数据写入
- 支持交织（interlacing）输出

### 图像格式支持

| 特性 | 支持情况 |
|------|----------|
| 1/2/4/8/16 位索引色 | ✅ 支持 |
| 灰度/灰度+Alpha | ✅ 支持 |
| 真彩色 (RGB/RGBA) | ✅ 支持 |
| 交织图像 (Adam7) | ✅ 支持 |
| 透明色通道 | ✅ 支持 |
| Gamma 校正 | ✅ 支持 |
| ICC 色彩管理 | ✅ 支持 |
| 文本注释 | ✅ 支持 |
| XMP 元数据 | ✅ 支持 |

### 压缩引擎

libpng 内部使用 zlib 进行图像数据压缩：

- **压缩级别**: 0-9 级压缩
- **滤波器**: SUB, UP, AVG, PAETH 及其变体
- **内存模式**: 逐行/批量解码

---

## 1.3 API 架构

libpng 采用分层 API 设计：

### 高级 API (png_image)

```c
// PNG 图像读写的高级接口
png_image_begin_read_from_file(&image, "test.png");
png_image_begin_read_from_memory(&image, buffer, size);
png_image_begin_read_from_stdio(&image, fp);
```

### 核心 API (png_struct / png_info)

```c
// 核心读写接口
png_ptr = png_create_read_struct(PNG_LIBPNG_VER_STRING, ...);
info_ptr = png_create_info_struct(png_ptr);
png_init_io(png_ptr, fp);
png_read_image(png_ptr, row_pointers);
```

### 底层 API

提供对 PNG 块（chunk）的直接访问，支持自定义块的处理。

---

## 1.4 性能优化特性

### ARM NEON 加速

libpng 支持 ARM NEON 指令集优化，显著提升滤波器运算性能：

- `PNG_ARM_NEON_OPT`: NEON 优化开关
- 支持 RGB (3通道) 和 RGBA (4通道) 的向量化计算
- 优化函数: `png_read_filter_row_*_neon`

### 多线程支持

libpng 支持在多线程环境中使用：

- 每个线程需要独立的 `png_struct`
- 图像数据缓冲区可在线程间共享

---

## 1.5 在 OpenHarmony 中的定位

### 系统角色

在 OpenHarmony 生态系统中，libpng 承担以下角色：

```
应用层
    ↓
ACE Framework / Image Framework
    ↓
libpng (PNG 图像解码)
    ↓
zlib (压缩支持)
```

### 核心使用场景

1. **UI 资源加载**: 解析应用界面中的 PNG 图片资源
2. **图像处理**: image_framework 的 PNG 编解码后端
3. **测试验证**: vk-gl-cts 图形测试中的参考图像处理
4. **字体渲染**: freetype 的嵌入式图像支持

### 版本策略

OpenHarmony 采用 **跟随上游 + 安全修复** 的策略：

- 基础版本: libpng 1.6.44（2024年5月）
- 安全补丁: 持续回移植上游安全修复
- 功能优化: OH 特有的 ARM NEON 扩展

---

## 1.6 与同类库的比较

| 特性 | libpng | stb_image | libjpeg-turbo |
|------|--------|-----------|---------------|
| 格式支持 | PNG | PNG/JPG/TGA | JPEG |
| 性能 | 高 | 中 | 最高（JPEG） |
| 代码体积 | 中等 | 最小 | 大 |
| 活跃维护 | 是 | 低频 | 是 |
| 许可证 | libpng | Public Domain | BSD/GPL |
| OH 采用 | ✅ 官方 PNG 库 | ❌ | ❌ |

---

## 1.7 扩展阅读

- [PNG 格式规范](http://www.w3.org/TR/PNG/)
- [libpng 官方文档](http://www.libpng.org/pub/png/libpng-manual.html)
- [OpenHarmony Image Framework](../multimedia/image_framework/README.md)

---

## 1.8 OH 适配总结

libpng 作为 PNG 格式的官方参考库，在 OpenHarmony 中：

- ✅ 提供完整的 PNG 编解码能力
- ✅ 已集成 ARM NEON 性能优化
- ✅ 包含完整的安全修复
- ✅ 被 image_framework 等核心模块依赖

下一章将详细介绍 OpenHarmony 对 libpng 的 Patch 定制。
