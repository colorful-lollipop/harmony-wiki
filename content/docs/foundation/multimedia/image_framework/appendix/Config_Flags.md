# 附录：配置标志

## 目的

本文档汇总 Image Framework 的所有编译配置标志（defines）。

## 平台定义

| 标志 | 说明 | 触发条件 |
|------|------|----------|
| `CROSS_PLATFORM` | 跨平台构建 | iOS/Android |
| `IOS_PLATFORM` | iOS 平台 | `use_clang_ios` |
| `ANDROID_PLATFORM` | Android 平台 | `use_clang_android` |
| `_WIN32` | Windows 平台 | `use_mingw_win` |
| `_APPLE` | macOS 平台 | `use_clang_mac` |

## 调试与日志

| 标志 | 说明 | 默认值 |
|------|------|--------|
| `IMAGE_DEBUG_FLAG` | 启用调试日志 | 开启 |
| `HICHECKER_ENABLE` | 启用 HiChecker | 条件 |

## 功能特性

| 标志 | 说明 | 依赖 |
|------|------|------|
| `IMAGE_COLORSPACE_FLAG` | 颜色空间支持 | 图形子系统 |
| `NEW_SKIA` | 使用 Skia | `skia:skia_canvaskit` |
| `USE_M133_SKIA` | 使用 Skia M133 | `image_use_new_skia` |
| `DUAL_ADAPTER` | 双适配器模式 | 非 Windows/Mac |
| `IMAGE_PICTURE_ENABLE` | Picture API | `enable_picture` |

## 硬件加速

| 标志 | 说明 | 依赖 |
|------|------|------|
| `JPEG_HW_DECODE_ENABLE` | JPEG 硬件解码 | `enable_jpeg_hw_decode` |
| `HEIF_HW_DECODE_ENABLE` | HEIF 硬件解码 | `enable_heif_hw_decode` |
| `HEIF_HW_ENCODE_ENABLE` | HEIF 硬件编码 | `enable_heif_hw_encode` |
| `HEIF_HW_DECODE_LINE` | HEIF 逐行解码 | HEIF 硬件解码 |
| `SK_ENABLE_OHOS_CODEC` | 启用 OHOS 编解码 | 硬件解码 |
| `ENABLE_ASTC_ENCODE_BASED_GPU` | GPU ASTC 编码 | OpenCL |
| `SUT_ENCODE_ENABLE` | SUT 编码 | `graphic_graphic_2d_ext` |

## 内存管理

| 标志 | 说明 | 依赖 |
|------|------|------|
| `IMAGE_PURGEABLE_PIXELMAP` | 可清除 PixelMap | `memory_utils_purgeable_ashmem_enable` |
| `IMAGE_QOS_ENABLE` | QoS 支持 | `resourceschedule_qos_manager` |

## 扩展功能

| 标志 | 说明 | 依赖 |
|------|------|------|
| `IMAGE_VPE_FLAG` | VPE 支持 | `multimedia_video_processing_engine` |
| `EXT_PIXEL` | 扩展像素格式 | `open_source_libyuv` |
| `SUPPORT_TIFF_DECODER` | TIFF 支持 | `has_libtiff` |
| `USE_NEON` | ARM NEON 优化 | `arm64` 或 `arm` |
| `SUT_PATH_X64` | X64 优化 | `arm64` 或模拟器 |

## 安全与边界检查

| 标志 | 说明 | 来源 |
|------|------|------|
| `BOUNDS_CHECK` | 边界检查 | `bounds_checking_function` |

## 条件编译示例

```cpp
#if !defined(IOS_PLATFORM) && !defined(ANDROID_PLATFORM)
    // OpenHarmony 特有功能
    std::unique_ptr<Picture> CreatePicture(...);
#endif

#ifdef JPEG_HW_DECODE_ENABLE
    // 使用硬件解码
    JpegHwDecoder::Decode(...);
#else
    // 使用软件解码
    JpegSoftwareDecoder::Decode(...);
#endif

#ifdef IMAGE_COLORSPACE_FLAG
    // 颜色空间转换
    ApplyColorSpace(...);
#endif
```

## GN 参数汇总

```gni
# ide/image_decode_config.gni
declare_args() {
  # 硬件解码开关
  enable_jpeg_hw_decode = false
  enable_heif_hw_decode = false
  enable_heif_hw_encode = false
  
  # Picture API
  enable_picture = true
  enable_picture_ndk = true
  
  # Skia 版本
  image_use_new_skia = false  # 设为 true 使用 M133
}

# 系统特性检测
if (memory_utils_purgeable_ashmem_enable) {
  defines += [ "IMAGE_PURGEABLE_PIXELMAP" ]
}

if (multimedia_video_processing_engine) {
  defines += [ "IMAGE_VPE_FLAG" ]
}

if (has_libtiff) {
  defines += [ "SUPPORT_TIFF_DECODER" ]
}
```

## 相关文档

- [GN 构建目标](../05_GN_Targets.md) - 构建系统详情
- [编译产物](../06_Build_Artifacts.md) - 输出文件说明
