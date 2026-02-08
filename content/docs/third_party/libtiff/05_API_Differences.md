# API/接口差异

## 概述

libtiff 在 OpenHarmony 中的 API 与上游版本基本一致，但有功能限制。

---

## OH 新增的 API

**无**。OpenHarmony 未新增任何 libtiff API。

---

## 行为变更的 API

**无**。所有 API 行为与上游版本一致。

---

## 废弃或禁用的功能

### 1. 图像编码功能

**状态**: ❌ **禁用**

**说明**: OpenHarmony 不支持 TIFF 图像编码（写入）功能。

**相关 API**:
- `TIFFWriteScanline()` - 写入扫描行
- `TIFFWriteEncodedStrip()` - 写入编码条带
- `TIFFWriteTile()` - 写入瓦片
- `TIFFWriteDirectory()` - 写入目录
- `TIFFClose()` - 关闭文件（编码时）

**影响**:
- 无法创建新的 TIFF 文件
- 无法修改现有的 TIFF 文件
- 无法保存图像为 TIFF 格式

**原因** (推断):
- OH 主要需要解码功能（读取和显示 TIFF）
- 编码功能需求较少
- 精简系统体积

**替代方案**:
- 使用其他图像格式（如 JPEG、PNG）保存
- 或集成完整的 libtiff（如需要编码功能）

---

### 2. 元数据编辑功能

**状态**: ❌ **禁用**

**说明**: OpenHarmony 不支持 TIFF 元数据编辑（修改或添加标签）功能。

**相关 API**:
- `TIFFSetField()` - 设置 TIFF 标签
- `TIFFUnsetField()` - 清除 TIFF 标签
- `TIFFWriteDirectory()` - 写入目录

**影响**:
- 无法修改图像元数据（如分辨率、作者信息等）
- 无法添加自定义 TIFF 标签
- 无法修改色彩配置等

**原因** (推断):
- OH 主要需要读取元数据，不需要修改
- 精简系统体积

**替代方案**:
- 使用其他图像处理库（如 ImageMagick）进行元数据编辑
- 或集成完整的 libtiff（如需要元数据编辑功能）

---

### 3. 工具程序

**状态**: ❌ **未集成**

**说明**: OpenHarmony 未集成 libtiff 的命令行工具。

**未集成的工具**:

| 工具 | 功能 | 上游版本 |
|-----|------|---------|
| **tiffcp** | TIFF 文件复制/转换 | 已集成 |
| **tiffinfo** | 显示 TIFF 信息 | 已集成 |
| **tiffdump** | 详细目录转储 | 已集成 |
| **tiffset** | 修改 TIFF 标签 | 未集成 |
| **tiffsplit** | 分割多页 TIFF | 未集成 |
| **tiffcrop** | 裁剪/旋转/镜像 | 未集成 |
| **tiff2pdf** | TIFF 转 PDF | 未集成 |
| **tiff2ps** | TIFF 转 PostScript | 未集成 |
| **tiff2rgba** | 转换为 RGBA | 未集成 |
| **tiffdither** | 图像二值化 | 未集成 |
| **pal2rgb** | 调色板转 RGB | 未集成 |
| **raw2tiff** | 原始数据转 TIFF | 未集成 |
| **fax2tiff** | 传真数据转 TIFF | 未集成 |
| **fax2ps** | 传真数据转 PS | 未集成 |

**影响**:
- 无法使用命令行工具处理 TIFF 文件
- 必须通过编程方式使用 libtiff API

**原因** (推断):
- OH 是移动操作系统，不需要命令行工具
- 精简系统体积

**替代方案**:
- 使用 libtiff API 编程实现相同功能
- 或在其他平台（如 Linux）使用这些工具

---

## 功能限制总结

| 功能 | OH 支持 | 说明 |
|-----|---------|------|
| **图像解码** | ✅ 支持 | 读取和显示 TIFF 图片 |
| **图像编码** | ❌ 不支持 | 无法写入 TIFF 图片 |
| **元数据读取** | ✅ 支持 | 读取 TIFF 标签 |
| **元数据编辑** | ❌ 不支持 | 无法修改或添加 TIFF 标签 |
| **命令行工具** | ⚠️ 部分支持 | tiffcp, tiffinfo, tiffdump 已集成；其他未集成 |
| **多页 TIFF** | ✅ 支持 | 单文件存储多幅图像 |
| **BigTIFF** | ✅ 支持 | 超过 4GB 的大文件 |
| **自定义目录** | ✅ 支持 | EXIF、GPS 等自定义 TIFF 目录 |

---

## API 使用限制

### 1. 解码 API 完全支持

**支持的解码 API**:

| API | 功能 | OH 状态 |
|-----|------|---------|
| `TIFFOpen()` | 打开 TIFF 文件 | ✅ 支持 |
| `TIFFClose()` | 关闭 TIFF 文件 | ✅ 支持 |
| `TIFFGetField()` | 读取 TIFF 标签 | ✅ 支持 |
| `TIFFReadRGBAImage()` | 解码为 RGBA 像素 | ✅ 支持 |
| `TIFFReadScanline()` | 读取扫描行 | ✅ 支持 |
| `TIFFReadEncodedStrip()` | 读取编码条带 | ✅ 支持 |
| `TIFFReadTile()` | 读取瓦片 | ✅ 支持 |
| `TIFFSetDirectory()` | 设置当前目录 | ✅ 支持 |

---

### 2. 编码 API 不支持

**不支持的编码 API**:

| API | 功能 | OH 状态 |
|-----|------|---------|
| `TIFFWriteScanline()` | 写入扫描行 | ❌ 不支持 |
| `TIFFWriteEncodedStrip()` | 写入编码条带 | ❌ 不支持 |
| `TIFFWriteTile()` | 写入瓦片 | ❌ 不支持 |
| `TIFFWriteDirectory()` | 写入目录 | ❌ 不支持 |
| `TIFFSetField()` | 设置 TIFF 标签 | ❌ 不支持 |
| `TIFFUnsetField()` | 清除 TIFF 标签 | ❌ 不支持 |

**注意**: 即使编译这些 API 调用，也无法实际写入 TIFF 文件（OH 未集成编码功能）。

---

### 3. 元数据 API 限制

**限制**:
- `TIFFGetField()` ✅ 支持 - 读取元数据
- `TIFFSetField()` ❌ 不支持 - 设置元数据
- `TIFFUnsetField()` ❌ 不支持 - 清除元数据

**影响**:
- 只能读取 TIFF 标签，无法修改
- 无法添加自定义 TIFF 标签

---

## API 使用示例

### 解码 TIFF 图片（推荐）

```c
#include <tiff.h>
#include <tiffio.h>

void decode_tiff(const char* filename) {
    // 1. 打开 TIFF 文件（只读模式）
    TIFF* tif = TIFFOpen(filename, "r");
    if (!tif) {
        fprintf(stderr, "Failed to open TIFF file\n");
        return;
    }

    // 2. 读取图像元数据
    uint32_t width, height;
    TIFFGetField(tif, TIFFTAG_IMAGEWIDTH, &width);
    TIFFGetField(tif, TIFFTAG_IMAGELENGTH, &height);

    printf("Image size: %u x %u\n", width, height);

    // 3. 解码图像数据
    uint32_t* raster = (uint32_t*)_TIFFmalloc(width * height * sizeof(uint32_t));
    if (raster && TIFFReadRGBAImage(tif, width, height, raster, 0)) {
        // 处理解码后的像素数据
        for (uint32_t y = 0; y < height; y++) {
            for (uint32_t x = 0; x < width; x++) {
                uint32_t pixel = raster[y * width + x];
                // 处理像素...
            }
        }
    }

    // 4. 释放资源
    _TIFFfree(raster);
    TIFFClose(tif);
}
```

---

### 读取多页 TIFF

```c
#include <tiff.h>
#include <tiffio.h>

void read_multipage_tiff(const char* filename) {
    // 1. 打开 TIFF 文件
    TIFF* tif = TIFFOpen(filename, "r");
    if (!tif) {
        fprintf(stderr, "Failed to open TIFF file\n");
        return;
    }

    // 2. 遍历所有页面
    int dircount = 0;
    do {
        dircount++;

        // 读取当前页面的图像数据
        uint32_t width, height;
        TIFFGetField(tif, TIFFTAG_IMAGEWIDTH, &width);
        TIFFGetField(tif, TIFFTAG_IMAGELENGTH, &height);

        printf("Page %d: %u x %u\n", dircount, width, height);

        // 解码当前页面
        uint32_t* raster = (uint32_t*)_TIFFmalloc(width * height * sizeof(uint32_t));
        if (raster && TIFFReadRGBAImage(tif, width, height, raster, 0)) {
            // 处理像素数据...
        }
        _TIFFfree(raster);

    } while (TIFFReadDirectory(tif));

    printf("Total pages: %d\n", dircount);
    TIFFClose(tif);
}
```

---

### 错误处理

```c
#include <tiff.h>
#include <tiffio.h>

void decode_with_error_handling(const char* filename) {
    // 1. 打开 TIFF 文件
    TIFF* tif = TIFFOpen(filename, "r");
    if (!tif) {
        fprintf(stderr, "Error: Failed to open TIFF file '%s'\n", filename);
        return;
    }

    // 2. 读取图像元数据
    uint32_t width, height;
    if (!TIFFGetField(tif, TIFFTAG_IMAGEWIDTH, &width) ||
        !TIFFGetField(tif, TIFFTAG_IMAGELENGTH, &height)) {
        fprintf(stderr, "Error: Failed to read image size\n");
        TIFFClose(tif);
        return;
    }

    // 3. 解码图像数据（stopOnError = 0，遇到错误尝试继续）
    uint32_t* raster = (uint32_t*)_TIFFmalloc(width * height * sizeof(uint32_t));
    if (raster) {
        if (TIFFReadRGBAImage(tif, width, height, raster, 0)) {
            printf("Decode successful\n");
        } else {
            fprintf(stderr, "Error: Failed to decode image\n");
        }
        _TIFFfree(raster);
    } else {
        fprintf(stderr, "Error: Failed to allocate memory\n");
    }

    // 4. 释放资源
    TIFFClose(tif);
}
```

---

## 性能优化建议

### 1. 使用逐行解码减少内存占用

**TIFFReadRGBAImage**:

```c
// 一次性解码整张图像（内存占用高）
uint32_t* raster = (uint32_t*)_TIFFmalloc(width * height * sizeof(uint32_t));
TIFFReadRGBAImage(tif, width, height, raster, 0);
```

**TIFFReadScanline**:

```c
// 逐行解码（内存占用低）
for (uint32_t row = 0; row < height; row++) {
    TIFFReadScanline(tif, buf, row);
    // 处理一行像素...
}
```

**建议**:
- 小图像（< 10 MP）: 使用 `TIFFReadRGBAImage`
- 大图像（> 10 MP）: 使用 `TIFFReadScanline`

---

### 2. 使用条件编译优化

**启用需要的压缩算法**:

```gn
# BUILD.gn
declare_args() {
    enable_jpeg = true   # 启用 JPEG（如需要）
    enable_zip = true    # 启用 Deflate（如需要）
    enable_lzw = true    # 启用 LZW（如需要）
    # ... 其他压缩算法
}
```

**说明**: 禁用不需要的压缩算法可以减少库体积。

---

## 兼容性注意事项

### 1. 字节序

libtiff 自动处理大端和小端字节序：

```c
TIFFOpen(filename, "r");  // 自动检测字节序
```

**无需手动处理字节序。**

---

### 2. BigTIFF 支持

libtiff 支持 BigTIFF（超过 4GB 的文件）：

```c
TIFF* tif = TIFFOpen("large_image.tif", "r");
if (tif) {
    // 自动检测 BigTIFF 格式
}
```

**无需特殊处理。**

---

### 3. 多页 TIFF

libtiff 支持多页 TIFF：

```c
do {
    // 处理当前页面...
} while (TIFFReadDirectory(tif));
```

**使用 `TIFFReadDirectory()` 遍历所有页面。**

---

## 总结

### API 差异总结

| 维度 | 结果 | 说明 |
|-----|------|------|
| **新增 API** | 无 | OH 未新增任何 API |
| **行为变更** | 无 | 所有 API 行为与上游一致 |
| **功能限制** | 部分 | 不支持编码、元数据编辑、大部分工具 |

### 功能限制清单

| 功能 | OH 支持 | 限制原因 |
|-----|---------|---------|
| **图像解码** | ✅ 支持 | - |
| **图像编码** | ❌ 不支持 | OH 主要需要解码功能 |
| **元数据读取** | ✅ 支持 | - |
| **元数据编辑** | ❌ 不支持 | OH 主要需要读取元数据 |
| **命令行工具** | ⚠️ 部分支持 | OH 是移动操作系统，不需要大部分工具 |

### 使用建议

1. **应用开发者**: 使用 Image Native API，无需直接调用 libtiff
2. **底层开发者**: 如需直接使用 libtiff，仅使用解码相关 API
3. **高级功能**: 如需编码或元数据编辑，考虑集成完整的 libtiff

---

**文档版本**: 1.0
**最后更新**: 2026年2月8日
