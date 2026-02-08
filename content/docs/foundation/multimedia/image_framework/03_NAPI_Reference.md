# N-API 接口文档

## 目的

本文档详细描述 Image Framework 的 JavaScript/TypeScript API（N-API 层），包括所有 JS 类、方法、参数和错误码。

## 模块注册

**模块名**: `multimedia.image`  
**注册文件**: `frameworks/kits/js/common/native_module_ohos_image.cpp`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_register_func = Export,
    .nm_modname = "multimedia.image",
    // ...
};
```

**注册的类**:
1. `ImageSource` - 图像解码
2. `ImagePacker` - 图像编码
3. `PixelMap` - 像素数据操作
4. `ImageReceiver` - 图像接收器
5. `ImageCreator` - 图像创建器
6. `Image` - Surface 图像
7. `Picture` - HDR 图像容器
8. `AuxiliaryPicture` - 辅助图片
9. `Metadata` - 元数据（EXIF/MakerNote/HEIFS）

## ImageSource

**N-API 实现**: `frameworks/kits/js/common/image_source_napi.cpp`  
**头文件**: `interfaces/kits/js/common/include/image_source_napi.h`

### 静态方法

| JS API | C++ 入口 | 说明 |
|--------|----------|------|
| `createImageSource(src: string \| Resource)` | `CreateImageSource()` | 创建图像源（文件路径或资源） |
| `createImageSource(fd: number)` | `CreateImageSource()` | 从文件描述符创建 |
| `createImageSource(data: ArrayBuffer)` | `CreateImageSource()` | 从内存数据创建 |
| `createIncrementalSource()` | `CreateIncrementalSource()` | 创建渐进式源 |
| `getSupportedFormats()` | `GetSupportedFormats()` | 获取支持的格式列表 |

### 实例方法

| JS API | C++ 入口 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `createPixelMap(options?)` | `CreatePixelMap()` | Async (Promise) | 创建 PixelMap |
| `createPixelMapSync(options?)` | `CreatePixelMapSync()` | Sync | 同步创建 PixelMap |
| `getImageInfo()` | `GetImageInfo()` | Async | 获取图像信息 |
| `getImageInfoSync()` | `GetImageInfoSync()` | Sync | 同步获取图像信息 |
| `createPicture(options?)` | `CreatePicture()` | Async | 创建 HDR Picture |
| `createThumbnail(options?)` | `CreateThumbnail()` | Async | 创建缩略图 |
| `getImageProperty(key: string)` | `GetImageProperty()` | Async | 获取图像属性（EXIF） |
| `getImagePropertySync(key: string)` | `GetImagePropertySync()` | Sync | 同步获取属性 |
| `modifyImageProperty(key, value)` | `ModifyImageProperty()` | Async | 修改图像属性 |
| `updateData(data, isComplete)` | `UpdateData()` | Async | 更新渐进式数据 |
| `release()` | `Release()` | Sync | 释放资源 |

### 参数结构

```typescript
// DecodeOptions
interface DecodeOptions {
    sampleSize?: number;           // 采样率
    rotate?: number;               // 旋转角度
    editable?: boolean;            // 是否可编辑
    desiredSize?: Size;            // 目标尺寸
    desiredRegion?: Region;        // 裁剪区域
    desiredPixelFormat?: PixelFormat;  // 目标像素格式
    index?: number;                // 帧索引（动图）
}

// ImageInfo
interface ImageInfo {
    size: Size;                    // 图像尺寸
    colorSpace: ColorSpace;        // 颜色空间
    pixelFormat: PixelFormat;      // 像素格式
    alphaType: AlphaType;          // Alpha 类型
}
```

### 调用链

```
JS: imageSource.createPixelMap(options)
    ↓
NAPI: ImageSourceNapi::CreatePixelMap()
    ↓
Native: ImageSource::CreatePixelMapEx()
    ↓
Plugin: AbsImageDecoder::Decode()
    ↓
Return: PixelMap NAPI object
```

## ImagePacker

**N-API 实现**: `frameworks/kits/js/common/image_packer_napi.cpp`  
**头文件**: `interfaces/kits/js/common/include/image_packer_napi.h`

### 静态方法

| JS API | C++ 入口 | 说明 |
|--------|----------|------|
| `createImagePacker()` | `CreateImagePacker()` | 创建打包器 |
| `getSupportedFormats()` | `GetSupportedFormats()` | 获取支持的编码格式 |

### 实例方法

| JS API | C++ 入口 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `packing(source, options)` | `Packing()` | Async | 打包 PixelMap/ImageSource |
| `packToFile(source, fd, options)` | `PackToFile()` | Async | 打包到文件描述符 |
| `release()` | `Release()` | Sync | 释放资源 |

### 参数结构

```typescript
// PackOption
interface PackOption {
    format: string;                // "image/jpeg", "image/png", etc.
    quality?: number;              // 压缩质量 0-100
    numberHint?: number;           // 帧数提示（GIF）
    loop?: number;                 // 循环次数（GIF）
    delayTimes?: number[];         // 帧延迟（GIF）
}
```

## PixelMap

**N-API 实现**: `frameworks/kits/js/common/pixel_map_napi.cpp`  
**头文件**: `interfaces/kits/js/common/include/pixel_map_napi.h`

### 静态方法

| JS API | C++ 入口 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `create(options, buffer?)` | `CreatePixelMap()` | Async | 创建 PixelMap |
| `createSync(options, buffer?)` | `CreatePixelMapSync()` | Sync | 同步创建 |
| `createUnpremultipliedPixelMap(pm)` | `CreateUnpremultipliedPixelMap()` | Async | 创建非预乘 PixelMap |
| `createPremultipliedPixelMap(pm)` | `CreatePremultipliedPixelMap()` | Async | 创建预乘 PixelMap |
| `unmarshalling(sequence)` | `Unmarshalling()` | Async | 反序列化 |
| `createPixelMapFromParcel(sequence)` | `CreatePixelMapFromParcel()` | Async | 从 Parcel 创建 |

### 实例属性

| JS 属性 | C++ 实现 | 说明 |
|---------|----------|------|
| `isEditable` | `GetIsEditable()` | 是否可编辑 |
| `isStrideAlignment` | `GetIsStrideAlignment()` | 是否步长对齐 |

### 实例方法

| JS API | C++ 入口 | 同步/异步 | 说明 |
|--------|----------|-----------|------|
| `readPixelsToBuffer(dst, options?)` | `ReadPixelsToBuffer()` | Async | 读取像素到缓冲区 |
| `readPixelsToBufferSync(dst, options?)` | `ReadPixelsToBufferSync()` | Sync | 同步读取 |
| `readPixels(src, dst)` | `ReadPixels()` | Async | 读取像素区域 |
| `readPixelsSync(src, dst)` | `ReadPixelsSync()` | Sync | 同步读取区域 |
| `writePixels(src, options?)` | `WritePixels()` | Async | 写入像素 |
| `writePixelsSync(src, options?)` | `WritePixelsSync()` | Sync | 同步写入 |
| `writeBufferToPixels(src)` | `WriteBufferToPixels()` | Async | 缓冲区写入像素 |
| `writeBufferToPixelsSync(src)` | `WriteBufferToPixelsSync()` | Sync | 同步写入 |
| `getImageInfo()` | `GetImageInfo()` | Sync | 获取图像信息 |
| `getBytesNumberPerRow()` | `GetBytesNumberPerRow()` | Sync | 获取每行字节数 |
| `getPixelBytesNumber()` | `GetPixelBytesNumber()` | Sync | 获取每像素字节数 |
| `scale(x, y, options?)` | `Scale()` | Async | 缩放 |
| `scaleSync(x, y, options?)` | `ScaleSync()` | Sync | 同步缩放 |
| `rotate(angle)` | `Rotate()` | Async | 旋转 |
| `rotateSync(angle)` | `RotateSync()` | Sync | 同步旋转 |
| `translate(x, y)` | `Translate()` | Async | 平移 |
| `translateSync(x, y)` | `TranslateSync()` | Sync | 同步平移 |
| `flip(horizontal, vertical)` | `Flip()` | Async | 翻转 |
| `flipSync(horizontal, vertical)` | `FlipSync()` | Sync | 同步翻转 |
| `crop(region)` | `Crop()` | Async | 裁剪 |
| `cropSync(region)` | `CropSync()` | Sync | 同步裁剪 |
| `getDensity()` | `GetDensity()` | Sync | 获取 DPI |
| `setDensity(density)` | `SetDensity()` | Sync | 设置 DPI |
| `setAlpha(alpha)` | `SetAlpha()` | Async | 设置透明度 |
| `setAlphaSync(alpha)` | `SetAlphaSync()` | Sync | 同步设置透明度 |
| `getColorSpace()` | `GetColorSpace()` | Sync | 获取颜色空间 |
| `setColorSpace(cs)` | `SetColorSpace()` | Sync | 设置颜色空间 |
| `applyColorSpace(cs)` | `ApplyColorSpace()` | Async | 应用颜色空间 |
| `marshalling()` | `Marshalling()` | Async | 序列化 |
| `createAlphaPixelmap()` | `CreateAlphaPixelmap()` | Async | 创建 Alpha 通道图 |
| `toSdr()` | `ToSdr()` | Async | 转换为 SDR |
| `release()` | `Release()` | Sync | 释放资源 |
| `getMetadata(type)` | `GetMetadata()` | Sync | 获取元数据 |
| `setMetadata(type, metadata)` | `SetMetadata()` | Sync | 设置元数据 |
| `setMetadataSync(type, metadata)` | `SetMetadataSync()` | Sync | 同步设置元数据 |
| `clone()` | `Clone()` | Async | 克隆 |
| `cloneSync()` | `CloneSync()` | Sync | 同步克隆 |

### 变换参数

```typescript
// AntiAliasingOption
enum AntiAliasingOption {
    NONE = 0,
    LOW = 1,
    MEDIUM = 2,
    HIGH = 3
}

// 变换选项
interface AntiAliasingOptions {
    antiAliasing?: AntiAliasingOption;  // 抗锯齿级别
}

// RWPixelsOptions
interface RWPixelsOptions {
    pixels: ArrayBuffer;
    offset: number;
    stride: number;
    region: Region;
}
```

## ImageReceiver

**N-API 实现**: `frameworks/kits/js/common/image_receiver_napi.cpp`

### 静态方法

| JS API | 说明 |
|--------|------|
| `createImageReceiver(width, height, format, capacity)` | 创建接收器 |

### 实例方法

| JS API | 同步/异步 | 说明 |
|--------|-----------|------|
| `getSize()` | Sync | 获取尺寸 |
| `getCapacity()` | Sync | 获取容量 |
| `getFormat()` | Sync | 获取格式 |
| `getReceivingSurfaceId()` | Sync | 获取 Surface ID |
| `readLatestImage()` | Async | 读取最新图像 |
| `readNextImage()` | Async | 读取下一图像 |
| `on(type, callback)` | Sync | 注册事件监听 |
| `off(type, callback?)` | Sync | 注销事件监听 |
| `release()` | Sync | 释放资源 |

## ImageCreator

**N-API 实现**: `frameworks/kits/js/common/image_creator_napi.cpp`

### 静态方法

| JS API | 说明 |
|--------|------|
| `createImageCreator(width, height, format, capacity)` | 创建创建器 |

### 实例方法

| JS API | 同步/异步 | 说明 |
|--------|-----------|------|
| `dequeueImage()` | Async | 出队图像 |
| `queueImage(image)` | Async | 入队图像 |
| `on(type, callback)` | Sync | 注册事件监听 |
| `off(type, callback?)` | Sync | 注销事件监听 |
| `release()` | Sync | 释放资源 |

## Picture (HDR)

**N-API 实现**: `frameworks/kits/js/common/picture_napi.cpp`

### 静态方法

| JS API | 说明 |
|--------|------|
| `createPicture(pixelMap)` | 从 PixelMap 创建 |
| `createPictureFromParcel(sequence)` | 从 Parcel 创建 |
| `createPictureByHdrAndSdrPixelMap(hdr, sdr)` | 从 HDR/SDR 创建 |

### 实例方法

| JS API | 同步/异步 | 说明 |
|--------|-----------|------|
| `getMainPixelmap()` | Sync | 获取主图 |
| `getHdrComposedPixelMap()` | Async | 获取 HDR 合成图 |
| `getHdrComposedPixelMapWithOptions(options)` | Async | 带选项获取 HDR |
| `getGainmapPixelmap()` | Sync | 获取增益图 |
| `getThumbnailPixelmap()` | Sync | 获取缩略图 |
| `setThumbnailPixelmap(pm)` | Sync | 设置缩略图 |
| `getAuxiliaryPicture(type)` | Sync | 获取辅助图 |
| `setAuxiliaryPicture(aux)` | Sync | 设置辅助图 |
| `dropAuxiliaryPicture(type)` | Sync | 删除辅助图 |
| `getMetadata(type)` | Sync | 获取元数据 |
| `setMetadata(type, metadata)` | Sync | 设置元数据 |
| `marshalling()` | Async | 序列化 |
| `release()` | Sync | 释放资源 |

## Metadata

**N-API 实现**: `frameworks/kits/js/common/metadata_napi.cpp`

### 类型

- `Metadata` - 基础元数据
- `ExifMetadata` - EXIF 元数据
- `MakerNoteMetadata` - MakerNote 元数据
- `HeifsMetadata` - HEIFS 元数据

### 方法

| JS API | 说明 |
|--------|------|
| `getProperties(key)` | 获取属性值 |
| `setProperties(key, value)` | 设置属性值 |
| `getAllProperties()` | 获取所有属性 |
| `clone()` | 克隆元数据 |
| `getBlob()` | 获取二进制数据 |
| `setBlob(data)` | 设置二进制数据 |

## 错误码

N-API 层使用 `BusinessError` 抛出异常：

```typescript
// 常见错误码
const enum ImageErrorCode {
    SUCCESS = 0,
    PARAMETER_ERROR = 401,           // 参数错误
    UNSUPPORTED_OPERATION = 7600101, // 不支持的操作
    OUT_OF_MEMORY = 7600102,         // 内存不足
    RESOURCE_UNAVAILABLE = 7600103,  // 资源不可用
    INVALID_PARAMETER = 7600104,     // 无效参数
    DECODING_FAILED = 7600105,       // 解码失败
    ENCODING_FAILED = 7600106,       // 编码失败
    CROP_FAILED = 7600107,           // 裁剪失败
}
```

## 参数验证

N-API 层使用宏进行参数验证：

```cpp
// frameworks/kits/js/common/include/image_napi_utils.h
#define IMG_NAPI_CHECK_RET_D(x, res, msg) \
    do { if (!(x)) { msg; return (res); } } while (0)

// 使用示例
IMG_NAPI_CHECK_RET_D(argCount == NUM_2 || argCount == NUM_1, 
    nullptr, IMAGE_LOGE("Invalid arg count"));
```

## 权限检查

部分 API 需要系统应用权限：

```cpp
// frameworks/kits/js/common/image_napi_utils.cpp:330
bool ImageNapiUtils::IsSystemApp() {
    static bool isSys = Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(
        IPCSkeleton::GetSelfTokenID());
    return isSys;
}
```

## 类型安全

使用 `napi_type_tag` 确保类型安全：

```cpp
// PixelMapNapi 示例
static constexpr napi_type_tag NAPI_TYPE_TAG = {
    .lower = 0x76e8cea642b74c67,
    .upper = 0x9cbb8e9d09251cc9
};

// 安全解包
status = napi_unwrap_s(env, thisVar, &NAPI_TYPE_TAG, 
    reinterpret_cast<void**>(&pixelMapNapi));
```

## 相关文档

- [架构说明](02_Architecture.md) - 架构设计
- [内部 API](04_Inner_API.md) - Native 层接口
- [安全风险](07_Security_Risks.md) - 安全分析
