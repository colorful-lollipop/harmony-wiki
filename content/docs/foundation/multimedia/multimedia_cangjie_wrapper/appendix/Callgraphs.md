# 附录：关键调用链

> 本文档整理 multimedia_cangjie_wrapper 的关键调用链（入口 → 核心逻辑）

---

## 调用链索引

| 调用链 | 场景 | 起始点 | 终点 |
|--------|------|--------|------|
| [相机初始化](#相机初始化) | 获取 CameraManager | `getCameraManager()` | `camera_framework` |
| [相机拍照](#相机拍照) | 拍照流程 | `PhotoOutput.capture()` | `camera_framework` |
| [图像解码](#图像解码) | 从文件解码图像 | `createImageSource()` | `image_framework` |
| [图像编码](#图像编码) | 编码为文件 | `ImagePacker.packing()` | `image_framework` |
| [视频缩略图](#视频缩略图) | 获取视频缩略图 | `AVImageGenerator.fetchFrameByTime()` | `player_framework` |
| [相册查询](#相册查询) | 查询媒体资源 | `PhotoAccessHelper.getAssets()` | `media_library` |
| [相册变更](#相册变更) | 修改相册内容 | `PhotoAccessHelper.applyChanges()` | `media_library` |

---

## 相机初始化

### 调用链

```
[应用层]
    ↓ 调用
getCameraManager(): CameraManager
    ↓ 调用 constructor
CameraManager.init()
    ↓ 调用 FFI
FfiCameraManagerConstructor(): Int64
    ↓ IPC/FFI
[camera_framework]
    ↓ 创建 C++ CameraManager
CameraManager* CameraManager::GetInstance()
    ↓ 返回
Int64 (RemoteData ID)
```

### 关键代码

**入口** (`ohos/multimedia/camera/camera.cj`):
```cangjie
public func getCameraManager(): CameraManager {
    CameraManager()
}
```

**FFI** (`ohos/multimedia/camera/camera_ffi.cj:25`):
```cangjie
foreign {
    func FfiCameraManagerConstructor(): Int64
}
```

**构造函数** (`ohos/multimedia/camera/camera_manager.cj:42-44`):
```cangjie
init() {
    super(unsafe { FfiCameraManagerConstructor() })
}
```

---

## 相机拍照

### 调用链

```
[应用层]
    ↓ 调用
PhotoOutput.capture()
    ↓ 调用 FFI
FfiCameraPhotoOutputCapture(id: Int64): Int32
    ↓ IPC/FFI
[camera_framework]
    ↓ 异步处理
    ↓ 回调
PhotoOutput.onCaptureEnd
    ↓ 调用 Cangjie 回调
Callback1Argument.invoke()
    ↓
[应用层回调]
```

### 关键代码

**拍照入口** (`ohos/multimedia/camera/photo_output.cj`):
```cangjie
public func capture(): Unit {
    unsafe {
        let errCode = FfiCameraPhotoOutputCapture(getID())
        successOrThrow(errCode)
    }
}
```

**FFI** (`ohos/multimedia/camera/camera_ffi.cj:104`):
```cangjie
foreign {
    func FfiCameraPhotoOutputCapture(id: Int64): Int32
}
```

**事件注册** (`ohos/multimedia/camera/photo_output.cj`):
```cangjie
public func on(eventType: CameraEvents, callback: Callback1Argument<CaptureEndInfo>): Unit {
    // 注册回调...
    let wrapper = {value: CCaptureEndInfo => callback.invoke(None, value.toCaptureEndInfo())}
    let lambdaData = Callback1Param<CCaptureEndInfo, Unit>(wrapper)
    FfiCameraPhotoOutputOnCaptureEnd(getID(), lambdaData.getID())
}
```

---

## 图像解码

### 调用链

```
[应用层]
    ↓ 调用
createImageSource(uri: String): ImageSource
    ↓ 调用 FFI
FfiOHOSImageSourceCreateFromUri(uri: CString): Int64
    ↓ IPC/FFI
[image_framework]
    ↓ 创建 ImageSource
ImageSource* ImageSource::CreateImageSource(const string& uri)
    ↓ 返回
Int64 (RemoteData ID)
    ↓
ImageSource.createPixelMap(opts): PixelMap
    ↓ 调用 FFI
FfiOHOSImageSourceCreatePixelMap(id: Int64, opts: CDecodingOptionsV2): Int64
    ↓ IPC/FFI
[image_framework]
    ↓ 解码
PixelMap* ImageSource::CreatePixelMap(const DecodeOptions& opts)
    ↓ 返回
Int64 (PixelMap ID)
```

### 关键代码

**创建 ImageSource** (`ohos/multimedia/image/image_source.cj`):
```cangjie
public func createImageSource(uri: String): ImageSource {
    unsafe {
        try (cUri = LibC.mallocCString(uri).asResource()) {
            let errCode = 0
            let id = FfiOHOSImageSourceCreateFromUri(cUri.value, inout errCode)
            checkAndThrow(errCode)
            return ImageSource(id)
        }
    }
}
```

**创建 PixelMap** (`ohos/multimedia/image/image_source.cj`):
```cangjie
public func createPixelMap(opts!: ?DecodingOptions): PixelMap {
    // ...
    let cOpts = opts.getValue().getValue()
    let errCode = FfiOHOSImageSourceCreatePixelMap(getID(), cOpts, inout pixelMapId)
    checkAndThrow(errCode)
    return PixelMap(pixelMapId)
}
```

---

## 图像编码

### 调用链

```
[应用层]
    ↓ 调用
createImagePacker(): ImagePacker
    ↓ 调用 FFI
FfiOHOSImagePackerCreate(): Int64
    ↓
ImagePacker.packing(pixelMap, option)
    ↓ 调用 FFI
FfiOHOSImagePackerPackingToData(packerId, pixelMapId, option): CArrUI8
    ↓ IPC/FFI
[image_framework]
    ↓ 编码
ImagePacker::Packing(PixelMap* pixelMap, const PackOption& option, uint8_t** data, uint32_t& size)
    ↓ 返回
Array<UInt8>
```

---

## 视频缩略图

### 调用链

```
[应用层]
    ↓ 调用
AVImageGenerator.fetchFrameByTime(timeUs, option, params)
    ↓ 调用 FFI
FfiOHOSAVImageGeneratorFetchFrameByTime(id, timeUs, option, params, pixelMapIdPtr): Int32
    ↓ IPC/FFI
[player_framework]
    ↓ 提取帧
AVImageGenerator::FetchFrameByTime(int64_t timeUs, ...)
    ↓ 返回
PixelMap
```

### 关键代码

**入口** (`ohos/multimedia/media/avimage_generator.cj`):
```cangjie
public func fetchFrameByTime(timeUs: Int64, option: QueryOption, param: PixelMapParams): PixelMap {
    unsafe {
        // ...
        let ret = FfiOHOSAVImageGeneratorFetchFrameByTime(getID(), timeUs, option.getValue(), 
            cPixelMapParams, inout pixelMapId, inout errCode)
        checkAndThrow(errCode)
        return PixelMap(pixelMapId)
    }
}
```

---

## 相册查询

### 调用链

```
[应用层]
    ↓ 调用
getPhotoAccessHelper(context): PhotoAccessHelper
    ↓ 调用 FFI
FfiPhotoAccessHelperGetPhotoAccessHelper(contextId): Int64
    ↓
PhotoAccessHelper.getAssets(options)
    ↓ 调用 FFI
FfiPhotoAccessHelperGetAssets(id, cOptions, errCodePtr): Int64
    ↓ IPC/FFI
[media_library]
    ↓ 查询数据库
PhotoAccessHelper::GetAssets(const FetchOptions& options)
    ↓ 返回
Int64 (FetchResult ID)
    ↓
FetchResult.getFirstObject(): PhotoAsset
    ↓ 调用 FFI
FfiFetchResultGetFirstObject(id, errCodePtr): FetchResultObject
    ↓ 返回
PhotoAsset
```

### 关键代码

**获取 Helper** (`ohos/file/photo_access_helper/photo_accesshelper.cj:44-52`):
```cangjie
public func getPhotoAccessHelper(context: UIAbilityContext): PhotoAccessHelper {
    unsafe {
        let ret = FfiPhotoAccessHelperGetPhotoAccessHelper(context.getID())
        if (ret == -1) {
            checkRet(INVALID_ARGUMENT_ERROR, "getPhotoAccessHelper")
        }
        PhotoAccessHelper(ret, context)
    }
}
```

**查询资源** (`ohos/file/photo_access_helper/photo_accesshelper.cj:90-99`):
```cangjie
public func getAssets(options: FetchOptions): PhotoAssetResult {
    var errCode = 0i32
    unsafe {
        let cOptions = options.toCFetchOptions()
        let ret = FfiPhotoAccessHelperGetAssets(getID(), cOptions, inout errCode)
        cOptions.free()
        checkRet(errCode, "getAssets")
        PhotoAssetResult(ret)
    }
}
```

---

## 相册变更

### 调用链

```
[应用层]
    ↓ 调用
MediaAssetChangeRequest.createAssetRequest(...)
    ↓
PhotoAccessHelper.applyChanges(changeRequest)
    ↓ 根据类型分发
MediaAssetChangeRequest.applyChanges()
    ↓ 调用 FFI
FfiMediaAssetChangeRequestImplApplyChanges(id): Int32
    ↓ IPC/FFI
[media_library]
    ↓ 执行变更
MediaAssetChangeRequest::ApplyChanges()
    ↓ 返回
Int32 (错误码)
```

### 关键代码

**应用变更** (`ohos/file/photo_access_helper/photo_accesshelper.cj:349-355`):
```cangjie
public func applyChanges(mediaChangeRequest: MediaChangeRequest): Unit {
    match (mediaChangeRequest) {
        case v: MediaAlbumChangeRequest => v.applyChanges()
        case v: MediaAssetChangeRequest => v.applyChanges()
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**变更请求 FFI** (`ohos/file/photo_access_helper/photo_accesshelper_ffi.cj:132`):
```cangjie
foreign {
    func FfiMediaAssetChangeRequestImplApplyChanges(id: Int64): Int32
}
```

---

## 回调注册流程

### 通用回调注册模式

```
[应用层]
    ↓ 调用
CameraManager.on(CameraEvents.CameraStatus, callback)
    ↓ 包装回调
let wrapper = {value: CCameraStatusInfo => 
    callback.invoke(None, value.toCameraStatusInfo())
}
    ↓ 创建 FFI 回调对象
Callback1Param<CCameraStatusInfo, Unit>(wrapper)
    ↓ 注册到底层
FfiCameraManagerOnCameraStatusChanged(managerId, callbackId)
    ↓ IPC/FFI
[camera_framework]
    ↓ 保存回调
CameraManager::OnCameraStatusChanged(callback)
    ↓ 异步触发
    ↓ 事件发生时
callback(CCameraStatusInfo)
    ↓ FFI 回调
Callback1Param.invoke()
    ↓ 调用 wrapper
wrapper(CCameraStatusInfo)
    ↓ 类型转换
CCameraStatusInfo.toCameraStatusInfo()
    ↓ 调用用户回调
callback.invoke(None, CameraStatusInfo)
    ↓
[应用层回调函数]
```

### 关键代码

**注册** (`ohos/multimedia/camera/camera_manager.cj:381-399`):
```cangjie
synchronized(cameraStatusMutex) {
    // 检查是否已注册
    if (findCallbackObject(callbackList, callback)) {
        CAMERA_LOG.error("CameraManager on failed: callback already registered")
        return
    }
    unsafe {
        // 包装回调
        let wrapper = {value: CCameraStatusInfo => 
            callback.invoke(None, value.toCameraStatusInfo())
        }
        let lambdaData = Callback1Param<CCameraStatusInfo, Unit>(wrapper)
        let errCode = FfiCameraManagerOnCameraStatusChanged(getID(), lambdaData.getID())
        successOrThrow(errCode)
        callbackList.add((callback, lambdaData.getID()))
    }
}
```

---

## 相关跳转

- [架构说明](01_Architecture.md)
- [N-API 接口](03_NAPI_Interfaces.md)
- [内部 API](04_Inner_API.md)
