# Image Framework 项目概览

## 目的

本文档提供 OpenHarmony Image Framework（图像框架）的整体介绍，帮助开发者快速理解项目定位、核心能力和运行环境。

## 适用范围

- 应用开发者（使用 JS/TS API）
- 系统开发者（使用 Native API）
- 安全审计人员
- 构建维护人员

## 项目定位

Image Framework 是 OpenHarmony 多媒体子系统的核心组件，提供**统一的图像编解码能力**，支持多种图像格式的解析、解码、编码和处理。

### 核心能力

| 能力 | 说明 | 证据 |
|------|------|------|
| **图像解码** | 支持 JPEG、PNG、BMP、GIF、TIFF、RAW、HEIF、WebP、SVG、ASTC 等格式 | `interfaces/innerkits/include/image_source.h:169-181` |
| **图像编码** | 支持 JPEG、PNG、WebP、GIF、HEIF 等格式输出 | `interfaces/innerkits/include/image_packer.h:98-109` |
| **PixelMap 操作** | 像素数据缩放、旋转、裁剪、颜色空间转换 | `interfaces/innerkits/include/pixel_map.h:147-623` |
| **HDR 图像** | Ultra HDR、Gainmap、Picture 容器支持 | `interfaces/innerkits/include/picture.h` |
| **渐进式解码** | 支持网络图片渐进加载 | `interfaces/innerkits/include/image_source.h:180` |
| **硬件加速** | 可选 JPEG/HEIF 硬件编解码 | `plugins/common/libs/image/libextplugin/BUILD.gn:14-18` |
| **元数据操作** | EXIF、ICC Profile、HEIF 元数据读写 | `interfaces/innerkits/include/metadata.h` |

### 运行环境

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| OpenHarmony Standard | ✅ 完整支持 | 默认目标平台 |
| iOS | ✅ 交叉编译 | `use_clang_ios` 标志 |
| Android | ✅ 交叉编译 | `use_clang_android` 标志 |
| Windows (MinGW) | ⚠️ 有限支持 | 仅基础功能 |
| macOS | ⚠️ 有限支持 | 开发调试使用 |

## 系统能力声明

组件声明的系统能力（SystemCapability）：

```json
// bundle.json:15-21
"syscap": [
  "SystemCapability.Multimedia.Image.Core",
  "SystemCapability.Multimedia.Image.ImageSource",
  "SystemCapability.Multimedia.Image.ImagePacker",
  "SystemCapability.Multimedia.Image.ImageReceiver",
  "SystemCapability.Multimedia.Image.ImageCreator"
]
```

## 架构总览

```
┌─────────────────────────────────────────────────────────────┐
│  Application Layer (JS/TS/C/C++)                            │
├─────────────────────────────────────────────────────────────┤
│  API Layer                                                  │
│  ├── N-API (multimedia.image)                               │
│  ├── NDK (libimage_*.so)                                    │
│  ├── Cangjie FFI                                            │
│  └── ANI (ArkUI Native Interface)                           │
├─────────────────────────────────────────────────────────────┤
│  Framework Layer                                            │
│  ├── ImageSource (解码器管理)                                │
│  ├── ImagePacker (编码器管理)                                │
│  ├── PixelMap (像素数据容器)                                 │
│  └── Picture (HDR/辅助图容器)                                │
├─────────────────────────────────────────────────────────────┤
│  Plugin System                                              │
│  ├── PluginManager (插件管理)                                │
│  └── Format Plugins (JPEG/PNG/GIF/HEIF...)                  │
└─────────────────────────────────────────────────────────────┘
```

## 关键概念

### PixelMap

内存中的位图表示，支持多种像素格式和内存分配器：

```cpp
// interfaces/innerkits/include/image_type.h
enum class PixelFormat : int32_t {
    UNKNOWN = 0,
    RGBA_8888 = 1,
    BGRA_8888 = 2,
    RGB_565 = 3,
    // ... 更多格式
};

enum class AllocatorType : int32_t {
    DEFAULT = 0,
    HEAP_ALLOC = 1,       // 堆内存
    SHARE_MEM_ALLOC = 2,  // 共享内存
    DMA_ALLOC = 4,        // DMA 零拷贝
};
```

### ImageSource

图像数据源，支持多种输入方式：

```cpp
// interfaces/innerkits/include/image_source.h:169-181
static std::unique_ptr<ImageSource> CreateImageSource(const std::string &pathName, ...);
static std::unique_ptr<ImageSource> CreateImageSource(const int fd, ...);
static std::unique_ptr<ImageSource> CreateImageSource(const uint8_t *data, uint32_t size, ...);
static std::unique_ptr<ImageSource> CreateImageSource(std::unique_ptr<std::istream> is, ...);
static std::unique_ptr<ImageSource> CreateIncrementalImageSource(...);
```

### Picture

HDR 图像容器，包含主图、增益图和辅助图片：

```cpp
// interfaces/innerkits/include/picture.h
class Picture : public Parcelable {
    std::shared_ptr<PixelMap> mainPixelMap_;
    std::map<AuxiliaryPictureType, std::shared_ptr<AuxiliaryPicture>> auxiliaryPictures_;
    std::map<MetadataType, std::shared_ptr<ImageMetadata>> metadata_;
};
```

## 内存限制

| 限制项 | 值 | 位置 |
|--------|-----|------|
| 最大源文件大小 | 300 MB | `frameworks/innerkitsimpl/codec/src/image_source.cpp:160` |
| 最大像素数据大小 | 128 MB (600MB RAM) | `interfaces/innerkits/include/pixel_map.h:89` |
| 最大图像维度 | ~536M 像素 | `frameworks/innerkitsimpl/utils/src/image_utils.cpp:116` |

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织详情
- [架构说明](02_Architecture.md) - 组件关系与数据流
- [N-API 接口](03_NAPI_Reference.md) - JS API 参考
- [安全风险](07_Security_Risks.md) - 安全分析

## 外部依赖

关键第三方库（来自 `bundle.json:29-71`）：

- `skia` - 2D 图形库
- `libjpeg-turbo` - JPEG 编解码
- `libpng` - PNG 编解码
- `ffmpeg` - 多媒体框架
- `astc-encoder` - ASTC 纹理压缩
- `libexif` - EXIF 元数据
- `opencl-headers` - GPU 计算
