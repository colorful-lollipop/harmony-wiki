# libtiff 原始库简介

## 库基本信息

| 字段 | 值 |
|-----|-----|
| **原始库名称** | LibTIFF - TIFF Library and Utilities |
| **当前版本** | 4.7.0 |
| **上游最新版本** | 4.7.1（截至文档更新时） |
| **许可证** | LibTIFF License (类 BSD 协议) |
| **上游地址** | https://gitlab.com/libtiff/libtiff |
| **官方文档** | https://libtiff.gitlab.io/libtiff/ |

---

## 什么是 libtiff？

LibTIFF 是一个用于支持 **Tag Image File Format (TIFF)** 的软件库，用于读写和处理 TIFF 图像文件。

TIFF 是一种灵活的图像格式，支持多种压缩算法、色彩空间和位深度，广泛应用于：
- **印刷出版**: 高质量图像存储
- **医学影像**: DICOM 转换和图像处理
- **地理信息系统 (GIS)**: 卫星图像和地图数据
- **文档扫描**: 档案和文档数字化
- **科学数据可视化**: 大图像数据集存储

---

## 核心功能

### 支持的压缩算法

| 压缩类型 | 说明 |
|---------|------|
| **无压缩** | Uncompressed |
| **CCITT Group 3/4** | 传真压缩 |
| **LZW** | Lempel-Ziv & Welch |
| **JPEG** | JPEG 压缩 |
| **PackBits** | 行程编码 |
| **Deflate/Adobe Deflate** | ZIP 风格压缩 |
| **LERC** | Limited Error Raster Compression |
| **PixarLog** | Pixar 压缩格式 |
| **LogLuv** | 对数压缩格式 |

### API 层级

LibTIFF 提供多层抽象接口：

1. **高级接口** - TIFFRGBAImage 支持，直接读取为 RGBA 像素栅格
2. **扫描行接口** - TIFFReadScanline()/TIFFWriteScanline()
3. **条带接口** - TIFFReadEncodedStrip()/TIFFWriteEncodedStrip()
4. **瓦片接口** - TIFFReadTile()/TIFFWriteTile()
5. **原始数据接口** - 直接访问未压缩的条带/瓦片数据

### 支持特性

- ✅ **多页/多图像 TIFF** - 单文件存储多幅图像
- ✅ **BigTIFF** - 支持超过 4GB 的大文件
- ✅ **自定义目录** - 支持 EXIF、GPS 等自定义 TIFF 目录
- ✅ **字节序无关** - 支持大端和小端字节序
- ✅ **跨平台** - 支持 UNIX (Linux, BSD, MacOS X) 和 Windows

---

## 在 OpenHarmony 中的作用和定位

### 集成位置

- **子系统**: thirdparty
- **组件**: libtiff
- **包名**: @ohos/libtiff
- **使用子系统**: multimedia（图像框架）

### 核心功能

在 OpenHarmony 中，libtiff 作为**媒体子系统的基础组件**，提供：

1. **TIFF 图像格式解码能力**
   - 读取和显示 TIFF 图片
   - 支持多种压缩算法（LZW, JPEG, Deflate 等）
   - 支持多种色彩空间和位深度

2. **Image Framework 插件**
   - 通过 tiffplugin 插件集成到图像框架
   - 为上层应用提供统一的图像解码接口
   - 支持跨平台（Android/iOS/HarmonyOS）

### 功能限制

根据 README_zh.md 和 README_en.md 说明：

| 功能 | OH 支持 | 说明 |
|-----|---------|------|
| **图像解码** | ✅ 支持 | 读取和显示 TIFF 图片 |
| **图像编码** | ❌ 不支持 | 无法写入 TIFF 图片 |
| **元数据编辑** | ❌ 不支持 | 无法修改或添加 TIFF 标签 |

**推断**: OpenHarmony 仅需要 libtiff 的解码功能，编码和元数据编辑功能未集成。这可能出于以下原因：
- 精简系统体积
- 需求不足（应用通常不需要创建 TIFF）
- 安全性考虑

---

## 依赖关系

### 外部依赖

libtiff 在 OpenHarmony 中依赖以下组件：

| 依赖组件 | 用途 | 条件 |
|---------|------|------|
| **c_utils** | 基础工具库 | 必需 |
| **zlib** | Deflate/PixarLog 压缩 | enable_zip / enable_pixarlog |
| **libjpeg-turbo** | JPEG/Old JPEG 压缩 | enable_jpeg / enable_ojpeg |
| **lzma** | LZMA 压缩 | enable_lzma（当前禁用） |

### 被依赖关系

libtiff 主要被以下模块依赖：

1. **tiffplugin** - TIFF 图像解码器插件（核心依赖者）
2. **multimediaplugin** - 图像格式插件聚合器
3. **tiffplugintest** - TIFF 解码器单元测试
4. **ImageTiffPluginFuzzTest** - TIFF 解码器模糊测试

详见 [04_Usage_in_OH.md](04_Usage_in_OH.md)。

---

## 代码结构

### 目录组织

```
libtiff/
├── libtiff/          # 核心库源代码（约 40 个 .c 文件）
│   ├── tif_aux.c      # 辅助函数
│   ├── tif_dir.c      # TIFF 目录读写
│   ├── tif_getimage.c # 图像数据读取
│   ├── tif_jpeg.c     # JPEG 压缩支持
│   ├── tif_zip.c      # Deflate 压缩支持
│   └── ...           # 其他源文件
├── tools/             # 命令行工具
│   ├── tiffcp         # TIFF 文件复制/转换
│   ├── tiffdump       # TIFF 信息转储
│   └── ...           # 其他工具
├── port/              # 跨平台适配
│   ├── getopt.c       # getopt 实现
│   └── libport.h      # 平台接口头文件
├── config/            # Autotools 配置文件
├── cmake/             # CMake 构建配置
├── doc/               # 文档
└── test/              # 测试用例
```

### 关键源文件

| 文件 | 大小 | 说明 |
|-----|------|------|
| `libtiff/tif_dir.c` | ~2,500 行 | TIFF 目录读写（最复杂的文件之一） |
| `libtiff/tif_getimage.c` | ~1,500 行 | 图像数据读取 |
| `libtiff/tif_dirread.c` | ~1,200 行 | 目录项读取 |
| `libtiff/tif_write.c` | ~900 行 | 图像数据写入 |
| `libtiff/tif_jpeg.c` | ~1,500 行 | JPEG 压缩支持 |
| `libtiff/tif_zip.c` | ~600 行 | Deflate 压缩支持 |

---

## 使用场景

### OpenHarmony 典型使用场景

1. **图像查看应用** - 显示 TIFF 格式的图片
2. **图像处理应用** - 读取 TIFF 格式的原始图像数据
3. **多媒体框架** - 支持多种图像格式的统一解码接口
4. **文档查看器** - 显示扫描的 TIFF 文档

### API 使用示例

```c
// 1. 打开 TIFF 文件
TIFF *tif = TIFFOpen("image.tif", "r");

// 2. 读取图像元数据
uint32_t width, height;
TIFFGetField(tif, TIFFTAG_IMAGEWIDTH, &width);
TIFFGetField(tif, TIFFTAG_IMAGELENGTH, &height);

// 3. 解码图像数据
uint32_t *raster = (uint32_t *)_TIFFmalloc(width * height * sizeof(uint32_t));
if (TIFFReadRGBAImage(tif, width, height, raster, 0)) {
    // 处理解码后的像素数据
}

// 4. 释放资源
_TIFFfree(raster);
TIFFClose(tif);
```

详见 [README_zh.md](../README_zh.md) 和 [04_Usage_in_OH.md](04_Usage_in_OH.md)。

---

## 版本历史

### 当前集成版本

OpenHarmony 集成 **libtiff 4.7.0**（2024年9月发布）。

### 近期版本

| 版本 | 发布日期 | 主要变更 |
|-----|---------|---------|
| **4.7.1** (最新) | 2024年 | 安全修复、性能优化、新增 API |
| **4.7.0** (OH 当前) | 2024年9月 | 恢复 v4.6.0 移出的工具 |
| **4.6.0** | 2023年 | 大多数工具移至 archive/ |
| **4.5.1** | 2023年 | 安全修复和稳定性改进 |

### 版本差异

4.7.1 相比 4.7.0 主要改进：
- 新增 API: `TIFFOpenOptionsSetWarnAboutUnknownTags()`
- 内存泄漏修复
- 缓冲区溢出修复
- LZW 解压性能优化
- 大量安全性修复（CVE 相关）

详见 [06_Security.md](06_Security.md)。

---

## 参考链接

### 官方资源

- **源代码仓库**: https://gitlab.com/libtiff/libtiff
- **官方主页**: https://libtiff.gitlab.io/libtiff/
- **历史主页**: http://www.simplesystems.org/libtiff/
- **下载站点**: https://download.osgeo.org/libtiff/

### 文档资源

- **函数文档**: https://libtiff.gitlab.io/libtiff/functions.html
- **构建说明**: https://libtiff.gitlab.io/libtiff/build.html
- **发行历史**: https://libtiff.gitlab.io/libtiff/releases/

### OpenHarmony 资源

- **中文说明**: [README_zh.md](../README_zh.md)
- **英文说明**: [README_en.md](../README_en.md)
- **构建配置**: [BUILD.gn](../BUILD.gn)
- **组件配置**: [bundle.json](../bundle.json)

---

## 总结

libtiff 是一个成熟、稳定的 TIFF 图像处理库，在 OpenHarmony 中作为图像解码基础设施被集成。

**关键特点**:
- 零 Patch 集成，升级相对容易
- 丰富的压缩算法支持
- 跨平台兼容性良好
- 功能受限（仅支持解码）

**适用场景**:
- 需要读取 TIFF 图像的应用
- 多媒体图像框架
- 图像处理和显示应用

---

**文档版本**: 1.0
**最后更新**: 2026年2月8日
