# N-API 接口文档

> 本文档详细描述 multimedia_cangjie_wrapper 对外暴露的 N-API（仓颉）接口

---

## 目的与适用范围

**目的**：为使用仓颉语言开发多媒体功能的开发者提供完整的 API 参考

**适用范围**：应用开发者、SDK 开发者

---

## 接口总览

### Kit 与命名空间对应关系

| Kit 名称 | 命名空间 | 入口函数/类 |
|----------|----------|-------------|
| CameraKit | `ohos.multimedia.camera` | `getCameraManager()` |
| ImageKit | `ohos.multimedia.image` | `createImageSource()` |
| MediaKit | `ohos.multimedia.media` | `createAVImageGenerator()` |
| MediaLibraryKit | `ohos.file.photo_access_helper` | `getPhotoAccessHelper()` |

### 权限要求汇总

| 接口 | 权限 | 授权方式 |
|------|------|----------|
| CameraManager.createCameraInput() | `ohos.permission.CAMERA` | user_grant |
| PhotoAccessHelper.getAssets() | `ohos.permission.READ_IMAGEVIDEO` | user_grant |
| PhotoAccessHelper.getAlbums() | `ohos.permission.READ_IMAGEVIDEO` | user_grant |
| PhotoAccessHelper.applyChanges() | `ohos.permission.WRITE_IMAGEVIDEO` | user_grant |

---

## CameraKit API

### 入口函数

```cangjie
// camera.cj
public func getCameraManager(): CameraManager
```

### CameraManager 类

#### 能力查询

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getSupportedCameras()` | - | `Array<CameraDevice>` | 同步 | 7400201 |
| `getSupportedSceneModes(camera: CameraDevice)` | CameraDevice | `Array<SceneMode>` | 同步 | 7400201 |
| `getSupportedOutputCapability(camera: CameraDevice, mode: SceneMode)` | CameraDevice, SceneMode | `CameraOutputCapability` | 同步 | 7400201 |
| `isCameraMuted()` | - | `Bool` | 同步 | - |
| `isTorchSupported()` | - | `Bool` | 同步 | - |
| `isTorchModeSupported(mode: TorchMode)` | TorchMode | `Bool` | 同步 | - |
| `getTorchMode()` | - | `TorchMode` | 同步 | - |

#### 创建输入输出

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 错误码 |
|------|------|--------|-----------|------|--------|
| `createCameraInput(camera: CameraDevice)` | CameraDevice | `CameraInput` | 同步 | CAMERA | 7400101, 7400102, 7400201 |
| `createCameraInput(position: CameraPosition, cameraType: CameraType)` | CameraPosition, CameraType | `CameraInput` | 同步 | CAMERA | 7400101, 7400102, 7400201 |
| `createPreviewOutput(profile: Profile, surfaceId: String)` | Profile, String | `PreviewOutput` | 同步 | - | 7400101, 7400201 |
| `createPreviewOutput(surfaceId: String)` | String | `PreviewOutput` | 同步 | - | 7400101, 7400201 |
| `createPhotoOutput(profile!: ?Profile)` | ?Profile | `PhotoOutput` | 同步 | - | 7400101, 7400201 |
| `createVideoOutput(profile: VideoProfile, surfaceId: String)` | VideoProfile, String | `VideoOutput` | 同步 | - | 7400101, 7400201 |
| `createVideoOutput(surfaceId: String)` | String | `VideoOutput` | 同步 | - | 7400101, 7400201 |
| `createSession(mode: SceneMode)` | SceneMode | `Session` | 同步 | - | 7400101, 7400201 |

#### 控制接口

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `setTorchMode(mode: TorchMode)` | TorchMode | `Unit` | 同步 | 7400102, 7400201 |

#### 事件注册

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `on(eventType: CameraEvents.CameraStatus, callback: Callback1Argument<CameraStatusInfo>)` | CameraEvents, Callback | `Unit` | 异步 | 7400201 |
| `on(eventType: CameraEvents.FoldStatusChange, callback: Callback1Argument<FoldStatusInfo>)` | CameraEvents, Callback | `Unit` | 异步 | 7400201 |
| `on(eventType: CameraEvents.TorchStatusChange, callback: Callback1Argument<TorchStatusInfo>)` | CameraEvents, Callback | `Unit` | 异步 | 7400201 |
| `off(eventType: CameraEvents, callback: Callback1Argument<T>)` | CameraEvents, Callback | `Unit` | 异步 | 7400201 |
| `off(eventType: CameraEvents)` | CameraEvents | `Unit` | 异步 | 7400201 |

### CameraInput 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `open()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `close()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `on(eventType: CameraEvents.CameraError, callback: Callback1Argument<Error>)` | CameraEvents, Callback | `Unit` | 异步 | - |
| `off(eventType: CameraEvents.CameraError, callback!: ?Callback1Argument<Error>)` | CameraEvents, ?Callback | `Unit` | 异步 | - |

### Session 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `beginConfig()` | - | `Unit` | 同步 | 7400103, 7400201 |
| `commitConfig()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `canAddInput(cameraInput: CameraInput)` | CameraInput | `Bool` | 同步 | 7400201 |
| `addInput(cameraInput: CameraInput)` | CameraInput | `Unit` | 同步 | 7400101, 7400102, 7400107, 7400110, 7400201 |
| `removeInput(cameraInput: CameraInput)` | CameraInput | `Unit` | 同步 | 7400101, 7400103, 7400201 |
| `canAddOutput(cameraOutput: CameraOutput, outputType: CameraOutputType)` | CameraOutput, CameraOutputType | `Bool` | 同步 | 7400201 |
| `addOutput(cameraOutput: CameraOutput, outputType: CameraOutputType)` | CameraOutput, CameraOutputType | `Unit` | 同步 | 7400101, 7400102, 7400107, 7400110, 7400201 |
| `removeOutput(cameraOutput: CameraOutput, outputType: CameraOutputType)` | CameraOutput, CameraOutputType | `Unit` | 同步 | 7400101, 7400103, 7400201 |
| `start()` | - | `Unit` | 同步 | 7400102, 7400104, 7400201 |
| `stop()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `release()` | - | `Unit` | 同步 | 7400201 |

### PhotoOutput 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `capture()` | - | `Unit` | 异步 | 7400102, 7400104, 7400201 |
| `capture(setting: PhotoCaptureSetting)` | PhotoCaptureSetting | `Unit` | 异步 | 7400102, 7400104, 7400201 |
| `isMovingPhotoSupported()` | - | `Bool` | 同步 | - |
| `enableMovingPhoto(enabled: Bool)` | Bool | `Unit` | 同步 | 7400102, 7400201 |
| `isMirrorSupported()` | - | `Bool` | 同步 | - |
| `enableMirror(isMirror: Bool)` | Bool | `Unit` | 同步 | 7400102, 7400201 |
| `getActiveProfile()` | - | `Profile` | 同步 | 7400201 |
| `getPhotoRotation(deviceDegree: Int32)` | Int32 | `Int32` | 同步 | 7400201 |
| `release()` | - | `Unit` | 同步 | - |

#### 事件

| 事件类型 | 回调参数类型 | 说明 |
|----------|-------------|------|
| `CaptureStartWithInfo` | `CaptureStartInfo` | 拍照开始 |
| `FrameShutter` | `FrameShutterInfo` | 快门事件 |
| `CaptureEnd` | `CaptureEndInfo` | 拍照结束 |
| `FrameShutterEnd` | `FrameShutterEndInfo` | 快门结束 |
| `CaptureReady` | - | 可拍照 |
| `EstimatedCaptureDuration` | `Int64` | 预估时长 |
| `Error` | `Error` | 错误事件 |

### PreviewOutput 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getSupportedFrameRates()` | - | `Array<FrameRateRange>` | 同步 | 7400201 |
| `setFrameRate(min: Int32, max: Int32)` | Int32, Int32 | `Unit` | 同步 | 7400102, 7400201 |
| `getActiveFrameRate()` | - | `FrameRateRange` | 同步 | 7400201 |
| `getActiveProfile()` | - | `Profile` | 同步 | 7400201 |
| `getPreviewRotation(value: Int32)` | Int32 | `Int32` | 同步 | 7400201 |
| `setPreviewRotation(imageRotation: Int32, isDisplayLocked: Bool)` | Int32, Bool | `Unit` | 同步 | 7400102, 7400201 |
| `release()` | - | `Unit` | 同步 | - |

#### 事件

| 事件类型 | 回调参数类型 | 说明 |
|----------|-------------|------|
| `FrameStart` | - | 帧开始 |
| `FrameEnd` | - | 帧结束 |
| `Error` | `Error` | 错误事件 |

### VideoOutput 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `start()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `stop()` | - | `Unit` | 同步 | 7400102, 7400201 |
| `getSupportedFrameRates()` | - | `Array<FrameRateRange>` | 同步 | 7400201 |
| `setFrameRate(minFps: Int32, maxFps: Int32)` | Int32, Int32 | `Unit` | 同步 | 7400102, 7400201 |
| `getActiveFrameRate()` | - | `FrameRateRange` | 同步 | 7400201 |
| `getActiveProfile()` | - | `VideoProfile` | 同步 | 7400201 |
| `getVideoRotation(imageRotation: Int32)` | Int32 | `Int32` | 同步 | 7400201 |
| `release()` | - | `Unit` | 同步 | - |

### CameraKit 错误码

| 错误码 | 含义 | 说明 |
|--------|------|------|
| 7400101 | 参数错误 | 参数缺失或类型不正确 |
| 7400102 | 操作不允许 | 当前状态不允许此操作 |
| 7400103 | 会话未配置 | 会话尚未配置 |
| 7400104 | 会话未运行 | 会话尚未启动 |
| 7400105 | 会话配置已锁定 | - |
| 7400106 | 设备设置已锁定 | - |
| 7400107 | 相机冲突 | 无法使用相机，发生冲突 |
| 7400108 | 相机被禁用 | 安全原因导致相机被禁用 |
| 7400109 | 相机被抢占 | - |
| 7400110 | 配置冲突未解决 | 与当前配置存在未解决的冲突 |
| 7400201 | 相机服务致命错误 | - |

---

## ImageKit API

### 入口函数

```cangjie
// image_source.cj
public func createImageSource(uri: String): ImageSource
public func createImageSource(fd: FileDescriptor): ImageSource
public func createIncrementalSource(opts!: SourceOptions): ImageSource
```

### ImageSource 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `createPixelMap(opts!: ?DecodingOptions)` | ?DecodingOptions | `PixelMap` | 同步 | 62980001-62980121 |
| `createPixelMapList(opts!: ?DecodingOptions)` | ?DecodingOptions | `Array<PixelMap>` | 同步 | - |
| `getImageInfo()` | - | `ImageInfo` | 同步 | - |
| `getImageProperty(key: String, opts!: ?ImagePropertyOptions)` | String, ?ImagePropertyOptions | `String` | 同步 | - |
| `updateData(buf: Array<UInt8>, isCompleted: Bool)` | Array<UInt8>, Bool | `Unit` | 同步 | - |
| `release()` | - | `Unit` | 同步 | - |

### ImagePacker 类

```cangjie
public func createImagePacker(): ImagePacker
```

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `packing(source: ImageSource, option: PackingOption)` | ImageSource, PackingOption | `Array<UInt8>` | 同步 | - |
| `packing(pixelMap: PixelMap, option: PackingOption)` | PixelMap, PackingOption | `Array<UInt8>` | 同步 | - |
| `release()` | - | `Unit` | 同步 | - |

### PixelMap 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getImageInfo()` | - | `ImageInfo` | 同步 | - |
| `readPixels(area: PositionArea)` | PositionArea | `Unit` | 同步 | - |
| `readPixels()` | - | `Array<UInt8>` | 同步 | - |
| `writePixels(area: PositionArea)` | PositionArea | `Unit` | 同步 | - |
| `setAlphaType(alphaType: AlphaType)` | AlphaType | `Unit` | 同步 | - |
| `release()` | - | `Unit` | 同步 | - |

### ImageKit 错误码

| 错误码 | 含义 |
|--------|------|
| 62980001 | 未知错误 |
| 62980002 | 参数错误 |
| 62980003 | 初始化失败 |
| 62980004 | 内存不足 |
| 62980005 | 不支持的操作 |
| 62980006 | 图像解码失败 |
| 62980007 | 图像编码失败 |
| 62980008 | 图像源不存在 |
| 62980009 | 图像文件损坏 |
| 62980010 | 图像太大 |
| 62980011 | 图像格式不支持 |
| 62980012 | 图像属性不存在 |
| 62980013 | 图像区域错误 |
| 62980014 | 图像组件错误 |
| 62980015 | 无效的旋转角度 |
| 62980016 | 无效的裁剪区域 |
| 62980104 | 内部对象初始化失败 |
| 62980115 | 无效的旋转/格式参数 |
| 62980121 | 图像源数据不完整 |

---

## MediaKit API

### 入口函数

```cangjie
// avimage_generator.cj
public func createAVImageGenerator(): AVImageGenerator
```

### AVImageGenerator 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `setAVFileDescriptor(fd: AVFileDescriptor)` | AVFileDescriptor | `Unit` | 同步 | - |
| `fetchFrameByTime(timeUs: Int64, option: QueryOption, param: PixelMapParams)` | Int64, QueryOption, PixelMapParams | `PixelMap` | 同步 | - |
| `fetchFrameByIndex(index: Int32, option: QueryOption, param: PixelMapParams)` | Int32, QueryOption, PixelMapParams | `PixelMap` | 同步 | - |
| `release()` | - | `Unit` | 同步 | - |

### 数据结构

```cangjie
public struct AVFileDescriptor {
    public let fd: Int32
    public let offset: Int64
    public let length: Int64
}

public struct PixelMapParams {
    public let width: Int32
    public let height: Int32
}

public enum QueryOption {
    | ClosestSync
    | NextSync
    | PreviousSync
}
```

---

## MediaLibraryKit API

### 入口函数

```cangjie
// photo_accesshelper.cj
public func getPhotoAccessHelper(context: UIAbilityContext): PhotoAccessHelper
```

### PhotoAccessHelper 类

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 错误码 |
|------|------|--------|-----------|------|--------|
| `getAssets(options: FetchOptions)` | FetchOptions | `PhotoAssetResult` | Worker 线程 | READ_IMAGEVIDEO | 201, 13900020, 14000011 |
| `getBurstAssets(burstKey: String, options: FetchOptions)` | String, FetchOptions | `PhotoAssetResult` | Worker 线程 | READ_IMAGEVIDEO | 201, 14000011 |
| `getAlbums(albumType: AlbumType, subtype: AlbumSubtype, options!: FetchOptions)` | AlbumType, AlbumSubtype, ?FetchOptions | `AlbumResult` | Worker 线程 | READ_IMAGEVIDEO | 201, 13900020, 14000011 |
| `registerChange(uri: String, forChildUris: Bool, callback: Callback1Argument<ChangeData>)` | String, Bool, Callback | `Unit` | 异步 | - | 13900012, 13900020 |
| `unregisterChange(uri: String, callback!: ?Callback1Argument<ChangeData>)` | String, ?Callback | `Unit` | 异步 | - | 13900012, 13900020 |
| `showAssetsCreationDialog(srcFileUris: Array<String>, photoCreationConfigs: Array<PhotoCreationConfig>, callback: Callback1Argument<Array<String>>)` | Array<String>, Array<PhotoCreationConfig>, Callback | `Unit` | 异步 | - | 14000011 |
| `applyChanges(mediaChangeRequest: MediaChangeRequest)` | MediaChangeRequest | `Unit` | Worker 线程 | WRITE_IMAGEVIDEO | 201, 14000011 |
| `release()` | - | `Unit` | Worker 线程 | - | 13900020, 14000011 |

### PhotoAsset 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getUri()` | - | `String` | 同步 | - |
| `getDisplayName()` | - | `String` | 同步 | - |
| `getMediaType()` | - | `MediaType` | 同步 | - |
| `get(member: PhotoKeys)` | PhotoKeys | `MemberType` | 同步 | - |
| `set(member: PhotoKeys, value: String)` | PhotoKeys, String | `Unit` | 同步 | - |
| `getThumbnail(size: Size)` | Size | `PixelMap` | 同步 | - |
| `commitModify()` | - | `Unit` | 同步 | - |

### Album 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getAlbumType()` | - | `AlbumType` | 同步 | - |
| `getAlbumSubtype()` | - | `AlbumSubtype` | 同步 | - |
| `getAlbumName()` | - | `String` | 同步 | - |
| `getAlbumUri()` | - | `String` | 同步 | - |
| `getCount()` | - | `Int32` | 同步 | - |
| `getCoverUri()` | - | `String` | 同步 | - |
| `getImageCount()` | - | `Int32` | 同步 | - |
| `getVideoCount()` | - | `Int32` | 同步 | - |
| `getAssets(options: FetchOptions)` | FetchOptions | `PhotoAssetResult` | 同步 | - |
| `commitModify()` | - | `Unit` | 同步 | - |

### FetchResult 类

| 方法 | 参数 | 返回值 | 同步/异步 | 错误码 |
|------|------|--------|-----------|--------|
| `getCount()` | - | `Int32` | 同步 | - |
| `isAfterLast()` | - | `Bool` | 同步 | - |
| `getFirstObject()` | - | `T` | 同步 | 14000011 |
| `getNextObject()` | - | `T` | 同步 | 14000011 |
| `getLastObject()` | - | `T` | 同步 | 14000011 |
| `getObjectAtPosition(position: Int32)` | Int32 | `T` | 同步 | 14000011 |
| `getAllObjects()` | - | `Array<T>` | 同步 | 14000011 |
| `close()` | - | `Unit` | 同步 | - |

### MediaLibraryKit 错误码

| 错误码 | 含义 |
|--------|------|
| 201 | 权限被拒绝 |
| 13900011 | 内存不足 |
| 13900012 | 权限被拒绝 |
| 13900020 | 参数无效 |
| 14000011 | 系统内部错误 |

---

## 调用链示例

### 相机拍照调用链

```
应用代码
    ↓ 调用
getCameraManager(): CameraManager
    ↓ 调用 FFI
FfiCameraManagerConstructor(): Int64
    ↓ 进入底层
camera_framework (C++)
```

### 图像解码调用链

```
应用代码
    ↓ 调用
createImageSource(uri: String): ImageSource
    ↓ 调用 FFI
FfiOHOSImageSourceCreateFromUri(): Int64
    ↓ 进入底层
image_framework (C++)
```

### 相册资源获取调用链

```
应用代码
    ↓ 调用
getPhotoAccessHelper(context): PhotoAccessHelper
    ↓ 调用 FFI
FfiPhotoAccessHelperGetPhotoAccessHelper(): Int64
    ↓ 调用
getAssets(options): PhotoAssetResult
    ↓ 调用 FFI
FfiPhotoAccessHelperGetAssets(): Int64
    ↓ 进入底层
media_library (C++)
```

---

## 相关跳转

- [目录结构](02_Directory_Structure.md)
- [内部 API](04_Inner_API.md)
- [安全风险分析](07_Security_Analysis.md)
