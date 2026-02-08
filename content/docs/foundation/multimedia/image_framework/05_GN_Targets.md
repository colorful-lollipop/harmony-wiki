# GN 构建目标

## 目的

本文档详细描述 Image Framework 的 GN 构建系统，包括所有构建目标、依赖关系和配置选项。

## 构建配置

### 配置文件

| 文件 | 作用 |
|------|------|
| `ide/image_decode_config.gni` | 主配置（平台定义、特性开关） |
| `plugins/cross/image_native_ios.gni` | iOS 专用配置 |
| `plugins/cross/image_native_android.gni` | Android 专用配置 |

### 关键配置变量

```gni
# ide/image_decode_config.gni

# 硬件解码开关
declare_args() {
  enable_jpeg_hw_decode = false
  enable_heif_hw_decode = false
  enable_heif_hw_encode = false
}

# 平台定义
image_decode_ios_defines = [
  "CROSS_PLATFORM",
  "IOS_PLATFORM",
]

image_decode_android_defines = [
  "CROSS_PLATFORM", 
  "ANDROID_PLATFORM",
]

# Skia 配置
skia_ohos_path = "//third_party/skia"
image_decode_ohos_defines = ["NEW_SKIA", "SK_BUILD_FOR_OHOS"]
```

## 顶层目标

### image_framework (Group)

**文件**: `BUILD.gn`

```gn
group("image_framework") {
  deps = [
    "frameworks/innerkitsimpl/utils:image_utils",
    "frameworks/kits/cj:cj_image_ffi",
    "interfaces/innerkits:image_native",
    "interfaces/kits/js/common:image",
    "interfaces/kits/js/common:image_napi",
    # ... NDK 等
  ]
}
```

### plugins (Group)

```gn
group("plugins") {
  deps = [
    "plugins/manager:pluginmanager",
    "plugins/common/libs:multimediaplugin",
  ]
}
```

## 核心库目标

### image_native (Shared Library)

**文件**: `interfaces/innerkits/BUILD.gn`

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` (OpenHarmony) / `ohos_source_set` (iOS/Android) |
| **输出** | `libimage_native.so` |
| **头文件** | `include/` |
| **源文件** | `frameworks/innerkitsimpl/common/src/`, `stream/src/`, `accessor/src/` |

**关键定义**:
```gn
defines = [
  "IMAGE_DEBUG_FLAG",
  "IMAGE_COLORSPACE_FLAG", 
  "NEW_SKIA",
  "DUAL_ADAPTER",
]

if (enable_jpeg_hw_decode) {
  defines += [ "JPEG_HW_DECODE_ENABLE" ]
}
if (enable_heif_hw_decode) {
  defines += [ "HEIF_HW_DECODE_ENABLE" ]
}
if (enable_heif_hw_encode) {
  defines += [ "HEIF_HW_ENCODE_ENABLE" ]
}
```

**外部依赖**:
```gn
external_deps = [
  "skia:skia_canvaskit",
  "graphic_2d:color_manager",
  "graphic_surface:surface",
  "libjpeg-turbo:turbojpeg",
  "libpng:libpng",
  "ffmpeg:libohosffmpeg",
  "hiviewdfx:hilog",
  "hiviewdfx:hisysevent",
]
```

### image_static (Static Library)

**类型**: `ohos_static_library`  
**平台**: Windows, Mac, iOS, Android

### image_utils (Shared Library)

**文件**: `frameworks/innerkitsimpl/utils/BUILD.gn`

| 属性 | 值 |
|------|-----|
| **输出** | `libimage_utils.so` |
| **源文件** | ImageTrace, ImageUtils, ColorUtils, PixelYuvUtils |

**条件定义**:
```gn
if (multimedia_video_processing_engine) {
  defines += [ "IMAGE_VPE_FLAG" ]
}
if (open_source_libyuv) {
  defines += [ "EXT_PIXEL" ]
}
```

## 插件目标

### Plugin Manager

**文件**: `plugins/manager/BUILD.gn`

| 目标 | 类型 | 输出 | 说明 |
|------|------|------|------|
| `pluginmanager` | `ohos_shared_library` | `libpluginmanager.so` | 插件管理框架 |
| `pluginmanager_static` | `ohos_static_library` | - | 静态库版本 |

**Sanitize**: CFI (Control Flow Integrity) 已启用（非 Android/iOS）

### 格式插件

每个插件产出两个目标：
- `{name}plugin` - 共享库 (`.so`)
- `{name}pluginmetadata` - 元数据文件 (`.pluginmeta`)

#### jpegplugin

**文件**: `plugins/common/libs/image/libjpegplugin/BUILD.gn`

| 属性 | 值 |
|------|-----|
| **源文件** | JpegDecoder, JpegEncoder, ExifInfo, IccProfileInfo |
| **外部依赖** | `libjpeg-turbo:turbojpeg` |

#### pngplugin

| 属性 | 值 |
|------|-----|
| **源文件** | PngDecoder, NinePatchListener |
| **外部依赖** | `zlib:shared_libz`, `libpng:libpng` |

#### extplugin (Extended)

**文件**: `plugins/common/libs/image/libextplugin/BUILD.gn`

| 目标 | 类型 | 说明 |
|------|------|------|
| `extplugin` | `ohos_shared_library` | 扩展编解码 |
| `heifparser` | `ohos_shared_library` / `ohos_source_set` | HEIF/CR3 解析 |
| `heifimpl` | `ohos_shared_library` | HEIF 实现 |
| `textureEncoderCL` | `ohos_shared_library` | OpenCL 纹理编码 |
| `exifhelper` | `ohos_static_library` | EXIF 助手 |

**条件定义**:
```gn
if (enable_jpeg_hw_decode) {
  defines += [ "JPEG_HW_DECODE_ENABLE", "SK_ENABLE_OHOS_CODEC" ]
}
if (enable_heif_hw_decode) {
  defines += [ "HEIF_HW_DECODE_ENABLE", "HEIF_HW_DECODE_LINE" ]
}
if (enable_heif_hw_encode) {
  defines += [ "HEIF_HW_ENCODE_ENABLE" ]
}
if (image_framework_feature_upgrade_skia) {
  defines += [ "USE_M133_SKIA" ]
}
```

#### tiffplugin

**条件**: `has_libtiff` 必须为 true

### Format Agent

**文件**: `plugins/common/libs/image/formatagentplugin/BUILD.gn`

包含所有格式检测代理：
- BmpFormatAgent
- Cr3FormatAgent
- GifFormatAgent
- HeifFormatAgent
- JpegFormatAgent
- PngFormatAgent
- RawFormatAgent
- SvgFormatAgent
- WbmpFormatAgent
- WebpFormatAgent
- TiffFormatAgent

## NDK 目标

### JS/Common NDK

**文件**: `frameworks/kits/js/common/ndk/BUILD.gn`

| 目标 | 输出 | 说明 |
|------|------|------|
| `image_ndk` | `libimage_ndk.so` | 基础 NDK |
| `image_receiver_ndk` | `libimage_receiver_ndk.so` | ImageReceiver NDK |
| `image_source_ndk` | `libimage_source_ndk.so` | ImageSource NDK |
| `image_packer_ndk` | `libimage_packer_ndk.so` | ImagePacker NDK |
| `image_source` | `libimage_source.so` | 原生 ImageSource |
| `image_packer` | `libimage_packer.so` | 原生 ImagePacker |
| `pixelmap` | `libpixelmap.so` | PixelMap NDK |
| `picture` | `libpicture.so` | Picture NDK |

### Native NDK

**文件**: `frameworks/kits/native/common/ndk/BUILD.gn`

| 目标 | 输出 |
|------|------|
| `ohimage` | `libohimage.so` |
| `image_receiver` | `libimage_receiver.so` |

## JS/NAPI 目标

### JS Interface

**文件**: `interfaces/kits/js/common/BUILD.gn`

| 目标 | 输出 | 说明 |
|------|------|------|
| `image` | `libimage_napi.so` | 主 JS 库 |
| `image_napi` | `libimage.so` | NAPI 模块 |
| `sendableimage` | `libsendableimage.so` | Sendable 变体 |
| `image_error_convert` | `libimage_error_convert.so` | 错误转换 |
| `multimedia_image` | - | Android 专用 |

## GPU/EGL 目标

**文件**: `frameworks/innerkitsimpl/egl_image/BUILD.gn`

| 目标 | 类型 | 输出 |
|------|------|------|
| `egl_image` | `ohos_shared_library` | `libegl_image.so` |
| `post_proc_gl` | `ohos_shared_library` | - |

**定义**: `SK_SUPPORT_GPU=1`, `SK_GL=1`, `GR_TEST_UTILS=1`

## 其他库

### Pixel Converter

**文件**: `frameworks/innerkitsimpl/pixelconverter/BUILD.gn`

| 目标 | 类型 |
|------|------|
| `pixelconvertadapter` | `ohos_shared_library` / `ohos_source_set` |
| `pixelconvertadapter_static` | `ohos_static_library` |

### Accessor

**文件**: `frameworks/innerkitsimpl/accessor/BUILD.gn`

| 目标 | 类型 | 输出 |
|------|------|------|
| `image_accessor` | `ohos_shared_library` | `libimage_accessor.so` |

## 依赖关系图

```
image_framework
├── image_utils
│   └── c_utils
├── image_native
│   ├── image_utils
│   ├── pluginmanager
│   ├── pixelconvertadapter
│   ├── heifparser (extplugin)
│   ├── image_accessor
│   ├── skia
│   ├── graphic_2d
│   └── libjpeg-turbo
├── image_napi
│   ├── image_native
│   └── napi
└── NDK 库
    └── image_native

plugins
├── pluginmanager
│   ├── c_utils
│   └── json
└── multimediaplugin (group)
    ├── formatagentplugin
    ├── jpegplugin
    ├── pngplugin
    ├── gifplugin
    ├── extplugin
    │   ├── heifparser
    │   ├── heifimpl
    │   └── textureEncoderCL
    └── tiffplugin (conditional)
```

## 条件编译

### 平台检测

```gn
if (use_mingw_win) {
  # Windows 构建
} else if (use_clang_mac) {
  # macOS 构建
} else if (use_clang_ios) {
  # iOS 构建
} else if (use_clang_android) {
  # Android 构建
} else {
  # OpenHarmony 默认构建
}
```

### 特性开关

| 特性 | 检测条件 | 定义 |
|------|----------|------|
| JPEG 硬件解码 | `enable_jpeg_hw_decode` | `JPEG_HW_DECODE_ENABLE` |
| HEIF 硬件解码 | `enable_heif_hw_decode` | `HEIF_HW_DECODE_ENABLE` |
| HEIF 硬件编码 | `enable_heif_hw_encode` | `HEIF_HW_ENCODE_ENABLE` |
| Picture API | `enable_picture` | `IMAGE_PICTURE_ENABLE` |
| Purgeable | `memory_utils_purgeable_ashmem_enable` | `IMAGE_PURGEABLE_PIXELMAP` |
| VPE 支持 | `multimedia_video_processing_engine` | `IMAGE_VPE_FLAG` |
| Skia M133 | `image_use_new_skia` | `USE_M133_SKIA` |
| TIFF 支持 | `has_libtiff` | `SUPPORT_TIFF_DECODER` |
| QoS | `resourceschedule_qos_manager` | `IMAGE_QOS_ENABLE` |

## 测试目标

**文件**: `frameworks/innerkitsimpl/test/BUILD.gn`

主要测试目标：
- `pixelmaptest`
- `picturetest`
- `auxiliarypicturetest`
- `imagesourcetest`
- `jpegdecoderextest`
- `transformtest`
- `formatagentplugintest`
- `convertertest`
- `ndktest`

## 构建命令示例

```bash
# 构建完整框架
gn gen out --args='target_os="ohos"'
ninja -C out image_framework

# 构建插件
ninja -C out plugins

# 构建测试
ninja -C out image_test_list

# 启用硬件解码
gn gen out --args='target_os="ohos" enable_jpeg_hw_decode=true'
```

## 相关文档

- [编译产物](06_Build_Artifacts.md) - 输出文件说明
- [附录/配置标志](appendix/Config_Flags.md) - 完整配置参考
