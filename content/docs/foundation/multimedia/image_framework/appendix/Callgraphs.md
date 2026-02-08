# 附录：关键调用链

## 目的

本文档记录 Image Framework 的关键调用链，帮助理解代码执行路径。

## 解码调用链

### 1. JS 创建 ImageSource

```
JS: image.createImageSource(path)
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::CreateImageSource()
        napi_get_value_string_utf8()    // 获取路径
        std::make_unique<ImageSourceAsyncContext>()
        IMG_CREATE_CREATE_ASYNC_WORK()  // 创建异步工作
        napi_queue_async_work()         // 入队
    ↓
ImageSourceNapi::CreateImageSourceComplete()
    ImageSource::CreateImageSource(path, opts, errorCode)
        ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
            ImageSource::DoImageSourceCreate()
                FileSourceStream::CreateSourceStream()  // 创建流
                new ImageSource()                       // 创建实例
```

### 2. JS 创建 PixelMap

```
JS: imageSource.createPixelMap(options)
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::CreatePixelMap()
        napi_create_promise()           // 创建 Promise
        napi_create_async_work()        // 创建异步工作
        napi_queue_async_work()
    ↓
ImageSourceNapi::CreatePixelMapExecute()
    ImageSource::CreatePixelMapEx()
        ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
            ImageSource::CreatePixelMapExtended()
                CreateDecoder()
                    ↓
                plugins/manager/src/plugin_server.cpp
                    PluginServer::CreateObject()
                        // 查找并创建解码器插件
                    ↓
                AbsImageDecoder::SetDecodeOptions()
                AbsImageDecoder::Decode()
                    ↓
                plugins/common/libs/image/libjpegplugin/src/jpeg_decoder.cpp
                    // 或其他格式插件
                ↓
                UpdatePixelMapInfo()    // 更新 PixelMap 信息
    ↓
ImageSourceNapi::CreatePixelMapComplete()
    PixelMapNapi::CreatePixelMap()      // 创建 JS PixelMap 对象
    napi_resolve_deferred()             // 解析 Promise
```

### 3. HDR Picture 解码

```
JS: imageSource.createPicture(options)
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::CreatePicture()
    ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
    ImageSource::CreatePicture()
        ParseHdrType()                  // 解析 HDR 类型
        CheckHdrType()
        ↓
        // 解码主图
        CreatePixelMapEx()
            ↓
        // 如果是 HDR，解码增益图
        DecodeJpegGainMap()
            AuxiliaryPicture::Create()
            picture->SetAuxiliaryPicture()
        ↓
        // 设置缩略图
        SetThumbnailForPicture()
    ↓
frameworks/kits/js/common/picture_napi.cpp
    PictureNapi::CreatePicture()
```

## 编码调用链

### 1. JS 编码 PixelMap 到文件

```
JS: imagePacker.packing(pixelMap, options)
    ↓
frameworks/kits/js/common/image_packer_napi.cpp
    ImagePackerNapi::Packing()
    ↓
frameworks/innerkitsimpl/common/src/image_packer.cpp
    ImagePacker::StartPacking()
        PackerStream::CreatePackerStream()
    ImagePacker::AddImage()
        GetEncoderPlugin()
            ↓
        plugins/manager/src/plugin_server.cpp
            PluginServer::CreateObject()
        ↓
        AbsImageEncoder::AddImage()
            ↓
        plugins/common/libs/image/libjpegplugin/src/jpeg_encoder.cpp
            // 或其他格式编码器
    ImagePacker::FinalizePacking()
        AbsImageEncoder::FinalizeEncode()
```

## PixelMap 变换调用链

### 1. 缩放操作

```
JS: pixelMap.scale(x, y, options)
    ↓
frameworks/kits/js/common/pixel_map_napi.cpp
    PixelMapNapi::Scale()
        napi_create_async_work()
    ↓
PixelMapNapi::ScaleExec()
    ↓
frameworks/innerkitsimpl/common/src/pixel_map.cpp
    PixelMap::scale()
        ↓
        // 使用 Skia 进行缩放
        Skia::Scale()
            ↓
        // 或使用软件实现
        DoTranslation()
```

### 2. 颜色空间转换

```
JS: pixelMap.applyColorSpace(colorSpace)
    ↓
frameworks/kits/js/common/pixel_map_napi.cpp
    PixelMapNapi::ApplyColorSpace()
    ↓
frameworks/innerkitsimpl/common/src/pixel_map.cpp
    PixelMap::ApplyColorSpace()
        ↓
frameworks/innerkitsimpl/pixelconverter/src/pixel_convert_adapter.cpp
    PixelConvertAdapter::ConvertColorSpace()
        // 使用色彩管理引擎
```

## 插件加载调用链

```
应用启动 / 首次解码
    ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
    ImageSource::InitClass()
        PluginServer::Register()
            ↓
plugins/manager/src/plugin_server.cpp
            ScanPlugins()               // 扫描插件路径
            LoadPluginMetadata()        // 加载 .pluginmeta
            dlopen()                    // 加载 .so
            RegisterPlugin()            // 注册插件接口
    ↓
ImageSource::CreateDecoder()
    PluginServer::CreateObject()
        // 根据格式选择插件
        CapabilityMatch()
        CreateDecoderInstance()
```

## 渐进式解码调用链

```
JS: imageSource.updateData(data, isComplete)
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::UpdateData()
    ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
    ImageSource::UpdateData()
        sourceStreamPtr_->UpdateData()
        ↓
        IncrementalPixelMap::PromoteDecoding()
            ↓
            ImageSource::PromoteDecoding()
                DoIncrementalDecoding()
                    decoder_->Decode()
            ↓
        DecodeListener::OnDecodeComplete()  // 回调进度
```

## 元数据操作调用链

### EXIF 读取

```
JS: imageSource.getImageProperty("ImageWidth")
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::GetImageProperty()
    ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
    ImageSource::GetImagePropertyString()
        ↓
    plugins/common/libs/image/libjpegplugin/src/exif_info.cpp
        ExifInfo::GetExifValue()
            // 解析 EXIF 数据
```

### EXIF 写入

```
JS: imageSource.modifyImageProperty(key, value)
    ↓
frameworks/kits/js/common/image_source_napi.cpp
    ImageSourceNapi::ModifyImageProperty()
    ↓
frameworks/innerkitsimpl/common/src/image_source.cpp
    ImageSource::ModifyImageProperty()
        CreateMetadataAccessorForWrite()
        ModifyImageProperties()
        WriteExifMetadataToFile()
```

## IPC 调用链 (PixelMap 传输)

```
应用 A: pixelMap.marshalling()
    ↓
frameworks/kits/js/common/pixel_map_napi.cpp
    PixelMapNapi::Marshalling()
    ↓
frameworks/innerkitsimpl/common/src/pixel_map.cpp
    PixelMap::Marshalling()
        WriteImageInfoToParcel()
        WriteMemInfoToParcel()
        WriteAshmemDataToParcel()  // 或 WriteImageData()
    ↓
Parcel 通过 IPC 传输
    ↓
应用 B: PixelMap.unmarshalling()
    ↓
frameworks/innerkitsimpl/common/src/pixel_map.cpp
    PixelMap::Unmarshalling()
        ReadImageInfo()
        ReadMemInfoFromParcel()
        ReadAshmemDataFromParcel()
```

## 硬件解码调用链 (JPEG)

```
ImageSource::CreatePixelMap()
    ↓
CreateDecoder()
    ↓
#ifdef JPEG_HW_DECODE_ENABLE
    plugins/common/libs/image/libextplugin/src/hardware/jpeg_hw_decoder.cpp
        JpegHwDecoder::Decode()
            // 检查是否支持硬件解码
            IsHardwareDecodeSupported()
            ↓
            // 申请 SurfaceBuffer
            AllocSurfaceBuffer()
            ↓
            // 调用硬件解码器
            HwDecoder::Decode()
                // 通过 HDI 调用驱动
#else
    // 软件解码
    libjpegplugin/src/jpeg_decoder.cpp
#endif
```

## 相关文档

- [架构说明](../02_Architecture.md) - 系统设计
- [N-API 接口](../03_NAPI_Reference.md) - API 参考
- [内部 API](../04_Inner_API.md) - 接口详情
