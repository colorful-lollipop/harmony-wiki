# 内部 API 与模块设计

> 本文档描述 multimedia_cangjie_wrapper 的内部模块设计、接口稳定性与依赖方向

---

## 目的与适用范围

**目的**：帮助框架开发者理解模块内部设计，进行维护和扩展

**适用范围**：框架开发者、维护者、架构师

---

## 模块划分

### 模块依赖图

```
                    ┌─────────────────┐
                    │   kit.* (对外)   │
                    └────────┬────────┘
                             │ 依赖
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ohos.multimedia│    │ohos.multimedia│    │   ohos.file   │
│   .camera     │    │   .image      │    │.photo_access_ │
│               │    │               │    │    helper     │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
        │ ┌──────────────────┘                    │
        │ │                                     │
        │ ▼                                     │
        │ohos.multimedia.image ◄────────────────┘
        │ (MediaKit 依赖 ImageKit)
        ▼
┌─────────────────────────────────────┐
│     外部 Wrapper 依赖（cj_external_deps）│
│  cangjie_ark_interop                │
│  ability_cangjie_wrapper            │
│  hiviewdfx_cangjie_wrapper          │
│  ...                                │
└─────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────┐
│       底层 C++ 框架（external_deps）   │
│  camera_framework:cj_camera_ffi     │
│  image_framework:cj_image_ffi       │
│  player_framework:cj_avplayer_ffi   │
│  media_library:cj_photoaccesshelper_│
└─────────────────────────────────────┘
```

---

## 模块职责详解

### 1. ohos.multimedia.camera

**文件**: `ohos/multimedia/camera/*.cj`

**职责**：
- 相机设备管理（发现、查询能力）
- 相机输入控制（打开/关闭）
- 预览/拍照/录像输出管理
- 会话配置与生命周期管理
- 事件回调注册与管理

**核心类设计**：

```cangjie
// CameraManager - 单例模式，相机设备管理
public class CameraManager <: RemoteDataLite {
    // 设备管理
    public func getSupportedCameras(): Array<CameraDevice>
    public func getSupportedSceneModes(camera: CameraDevice): Array<SceneMode>
    
    // 输入输出创建
    public func createCameraInput(camera: CameraDevice): CameraInput
    public func createPhotoOutput(profile!: ?Profile): PhotoOutput
    public func createPreviewOutput(profile: Profile, surfaceId: String): PreviewOutput
    public func createVideoOutput(profile: VideoProfile, surfaceId: String): VideoOutput
    
    // 会话创建
    public func createSession(mode: SceneMode): Session
}

// Session - 会话管理
public class Session <: RemoteDataLite {
    public func beginConfig(): Unit
    public func commitConfig(): Unit
    public func addInput(cameraInput: CameraInput): Unit
    public func addOutput(cameraOutput: CameraOutput, outputType: CameraOutputType): Unit
    public func start(): Unit
    public func stop(): Unit
}
```

**线程安全**：
- 使用 `Mutex` 保护回调列表：`cameraStatusMutex`, `foldStatusMutex`, `torchStatusMutex`
- 证据：`camera_manager.cj:36-40`

### 2. ohos.multimedia.image

**文件**: `ohos/multimedia/image/*.cj`

**职责**：
- 图像源管理（从 URI/FD 创建）
- 图像解码（创建 PixelMap）
- 图像编码（Packing）
- PixelMap 像素操作

**核心类设计**：

```cangjie
// ImageSource - 图像源
public class ImageSource <: RemoteDataLite {
    public func createPixelMap(opts!: ?DecodingOptions): PixelMap
    public func getImageInfo(): ImageInfo
    public func updateData(buf: Array<UInt8>, isCompleted: Bool): Unit
}

// ImagePacker - 图像编码器
public class ImagePacker <: RemoteDataLite {
    public func packing(source: ImageSource, option: PackingOption): Array<UInt8>
    public func packing(pixelMap: PixelMap, option: PackingOption): Array<UInt8>
}

// PixelMap - 像素图
public class PixelMap <: RemoteDataLite {
    public func readPixels(area: PositionArea): Unit
    public func writePixels(area: PositionArea): Unit
    public func getImageInfo(): ImageInfo
}
```

**参数校验**：
```cangjie
// cj_image_common.cj:430-438
func parseDecodeOptions() {
    if (rotate < 0 || rotate > 360) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid rotate ${rotate}")
    }
    if (desiredPixelFormat.getValue() > 9) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid desiredPixelFormat")
    }
}
```

### 3. ohos.multimedia.media

**文件**: `ohos/multimedia/media/*.cj`

**职责**：
- 视频缩略图生成

**核心类设计**：

```cangjie
// AVImageGenerator - 视频缩略图生成器
public class AVImageGenerator <: RemoteDataLite {
    public func setAVFileDescriptor(fd: AVFileDescriptor): Unit
    public func fetchFrameByTime(timeUs: Int64, option: QueryOption, param: PixelMapParams): PixelMap
    public func fetchFrameByIndex(index: Int32, option: QueryOption, param: PixelMapParams): PixelMap
}
```

**依赖关系**：
- 依赖 `ohos.multimedia.image` 的 `PixelMap`
- 证据：`media/BUILD.gn:31`
```gn
cj_deps = ["../../multimedia/image:ohos.multimedia.image"]
```

### 4. ohos.file.photo_access_helper

**文件**: `ohos/file/photo_access_helper/*.cj`

**职责**：
- 相册访问助手（获取资源、相册）
- 媒体资源操作（缩略图、属性）
- 相册操作（重命名、添加/移除资源）
- 变更监听（注册/注销）
- 资源变更请求（创建、删除）

**核心类设计**：

```cangjie
// PhotoAccessHelper - 入口类
public class PhotoAccessHelper <: RemoteDataLite {
    public func getAssets(options: FetchOptions): PhotoAssetResult
    public func getAlbums(albumType: AlbumType, subtype: AlbumSubtype, options!: FetchOptions): AlbumResult
    public func registerChange(uri: String, forChildUris: Bool, callback: Callback1Argument<ChangeData>): Unit
    public func unregisterChange(uri: String, callback!: ?Callback1Argument<ChangeData>): Unit
    public func applyChanges(mediaChangeRequest: MediaChangeRequest): Unit
}

// PhotoAsset - 媒体资源
public class PhotoAsset <: RemoteDataLite {
    public func getUri(): String
    public func getThumbnail(size: Size): PixelMap
    public func get(member: PhotoKeys): MemberType
    public func set(member: PhotoKeys, value: String): Unit
}

// Album - 相册
public class Album <: RemoteDataLite {
    public func getAlbumName(): String
    public func getCount(): Int32
    public func getAssets(options: FetchOptions): PhotoAssetResult
}
```

**包信息获取**：
```cangjie
// photo_accesshelper.cj:261-275
func getSelfBundleInfo(): FfiBundleInfo {
    let bundleFlags = BundleFlag.GET_BUNDLE_INFO_WITH_ABILITY |
        BundleFlag.GET_BUNDLE_INFO_WITH_HAP_MODULE |
        BundleFlag.GET_BUNDLE_INFO_WITH_SIGNATURE_INFO |
        BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    let bundleInfo = BundleManager.getBundleInfoForSelf(bundleFlags)
    // ... 获取 bundleName, appId, appName
    FfiBundleInfo(bundleName, appName, appId)
}
```

---

## 接口稳定性标注

### 稳定性分级

| 级别 | 含义 | 标识 | 示例 |
|------|------|------|------|
| **稳定** | 对外公开 API，向后兼容 | `@!APILevel` | 所有 Kit 层接口 |
| **内部** | 内部使用，可能变更 | `@!Hide[isChecked: true]` | 隐藏枚举值、内部类 |
| **不稳定** | 实验性功能 | TODO 标记 | Beta 功能 |

### 稳定接口（Public API）

标记方式：
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Multimedia.Camera.Core"
]
public class CameraManager <: RemoteDataLite
```

### 内部接口（Internal）

标记方式：
```cangjie
@!Hide[isChecked: true]
internal enum ResolutionQuality {
    @!Hide[isChecked: true]
    Low | Medium | High
}
```

### 隐藏功能列表

| 模块 | 隐藏功能 | 位置 |
|------|----------|------|
| Camera | DepthProfile | `camera_common.cj:78` |
| Camera | HostDeviceType | `camera_common.cj:81` |
| Camera | MetadataObjectType.HumanBody | `camera_common.cj:511` |
| Camera | MetadataObjectType.CatFace | `camera_common.cj:513` |
| Camera | MetadataObjectType.CatBody | `camera_common.cj:515` |
| Camera | MetadataObjectType.DogFace | `camera_common.cj:517` |
| Camera | MetadataObjectType.DogBody | `camera_common.cj:519` |
| Camera | MetadataObjectType.SalientDetection | `camera_common.cj:521` |
| Camera | MetadataObjectType.BarCodeDetection | `camera_common.cj:523` |
| Camera | CameraFormat.CameraFormatDng | `camera_common.cj:764` |
| Camera | CameraFormat.CameraFormatDngXdraw | `camera_common.cj:767` |
| Camera | CameraFormat.CameraFormatDepth16 | `camera_common.cj:815` |
| Camera | CameraFormat.CameraFormatDepth32 | `camera_common.cj:818` |
| Camera | CameraPosition.CameraPositionFoldInner | `camera_common.cj:159` |
| Camera | SceneMode.PortraitPhoto | `camera_common.cj:293` |
| Camera | SceneMode.NightPhoto | `camera_common.cj:295` |
| Camera | SceneMode.ProfessionalPhoto | `camera_common.cj:297` |
| Camera | SceneMode.ProfessionalVideo | `camera_common.cj:299` |
| Camera | SceneMode.SlowMotionVideo | `camera_common.cj:301` |
| Camera | SceneMode.MacroPhoto | `camera_common.cj:303` |
| Camera | SceneMode.MacroVideo | `camera_common.cj:305` |
| Camera | SceneMode.LightPaintingPhoto | `camera_common.cj:307` |
| Camera | SceneMode.HighResolutionPhoto | `camera_common.cj:309` |
| Camera | SceneMode.QuickShotPhoto | `camera_common.cj:329` |
| Camera | SceneMode.ApertureVideo | `camera_common.cj:331` |
| Camera | SceneMode.PanoramaPhoto | `camera_common.cj:333` |
| Camera | SceneMode.TimeLapsePhoto | `camera_common.cj:335` |
| Camera | SceneMode.FluorescencePhoto | `camera_common.cj:337` |
| Image | ResolutionQuality | `cj_image_common.cj:25` |
| Image | CropAndScaleStrategy | `cj_image_common.cj:38` |

---

## 依赖方向

### 依赖规则

1. **上层依赖下层**：kit/ → ohos/ → 底层框架
2. **同级可依赖**：media → image
3. **禁止循环依赖**：无循环依赖

### 依赖矩阵

| 模块 | 依赖的 Wrapper | 依赖的底层框架 |
|------|---------------|---------------|
| camera | cangjie_ark_interop, ability_cangjie_wrapper, hiviewdfx_cangjie_wrapper, graphic_cangjie_wrapper | camera_framework |
| image | cangjie_ark_interop, global_cangjie_wrapper, hiviewdfx_cangjie_wrapper, graphic_cangjie_wrapper | image_framework |
| media | cangjie_ark_interop, hiviewdfx_cangjie_wrapper, ohos.multimedia.image | player_framework |
| photo_access_helper | cangjie_ark_interop, ability_cangjie_wrapper, bundlemanager_cangjie_wrapper, distributeddatamgr_cangjie_wrapper, global_cangjie_wrapper, hiviewdfx_cangjie_wrapper, ohos.multimedia.image | media_library |

---

## 错误处理机制

### 统一错误处理

所有模块使用统一的错误处理模式：

```cangjie
// 错误码定义（camera_common.cj:36-50）
let ERROR_CODE_MAP = HashMap<Int32, String>(
    [
        (7400101, "Parameter missing or parameter type incorrect."),
        (7400102, "Operation not allowed."),
        (7400201, "Camera service fatal error."),
        // ...
    ]
)

// 错误抛出（camera_common.cj:62-66）
func successOrThrow(errCode: Int32): Unit {
    if (errCode != SUCCESS_CODE) {
        throw BusinessException(errCode, getErrorMsg(errCode))
    }
}

// 使用（camera_manager.cj:68）
let res = carr.toArray()
carr.free()
successOrThrow(errCode)
return res
```

### 错误传播链

```
底层 C++ 框架
    ↓ 返回 Int32 错误码
FFI 层 (Cangjie)
    ↓ 调用 successOrThrow()
仓颉业务代码
    ↓ 抛出 BusinessException
应用层
    ↓ try-catch 处理
用户界面
```

---

## 资源管理

### RemoteDataLite 模式

```
┌─────────────────────┐
│  Cangjie 对象        │
│  - myDataId: Int64   │ ← 引用底层对象 ID
│  - 其他业务属性      │
└─────────┬───────────┘
          │
          │ FFI 调用
          │
┌─────────▼───────────┐
│  底层 C++ 对象       │
│  (camera_framework)  │
└─────────────────────┘
```

### 资源释放

```cangjie
// 析构时释放
~init() {
    releaseFFIData(myDataId)
}

// 或显式释放
public func release(): Unit {
    unsafe { FfiOHOSImageRelease(getID()) }
    releaseFFIData(getID())
}
```

---

## 回调机制

### 回调注册流程

```cangjie
// 1. 包装用户回调
let wrapper = {value: CCameraStatusInfo => 
    callback.invoke(None, value.toCameraStatusInfo())
}

// 2. 创建 FFI 回调对象
let lambdaData = Callback1Param<CCameraStatusInfo, Unit>(wrapper)

// 3. 注册到底层
let errCode = FfiCameraManagerOnCameraStatusChanged(getID(), lambdaData.getID())

// 4. 保存引用（用于注销）
callbackList.add((callback, lambdaData.getID()))
```

### 回调注销流程

```cangjie
// 1. 从底层注销
let errCode = offFunc(idClassInfo.id, item[1])

// 2. 从列表移除
callbackList.removeIf({item => refEq(item[0], callback)})
```

---

## 关键结论

1. **分层清晰**：Kit → FFI → 底层框架，职责单一
2. **接口稳定**：对外 API 使用 `@!APILevel` 标记，内部使用 `@!Hide`
3. **依赖有序**：上层依赖下层，无循环依赖
4. **错误统一**：使用 BusinessException 统一错误处理
5. **资源安全**：RemoteDataLite 模式确保资源正确释放
6. **回调规范**：Callback1Param 包装用户回调，支持注册/注销

---

## 相关跳转

- [架构说明](01_Architecture.md)
- [N-API 接口](03_NAPI_Interfaces.md)
- [GN Targets](05_GN_Targets.md)
