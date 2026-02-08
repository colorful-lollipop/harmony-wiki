# 目录结构与模块职责

> 本文档详细描述 multimedia_cangjie_wrapper 的目录结构、文件组织及各模块职责

---

## 目的与适用范围

**目的**：帮助开发者快速定位代码，理解各目录和文件的职责

**适用范围**：新加入的开发者、需要查找特定代码的维护者

---

## 顶层目录结构

```
foundation/multimedia/multimedia_cangjie_wrapper/
├── figures/                          # 架构图和资源文件
├── kit/                              # Kit 对外接口层
│   ├── CameraKit/                    # 相机 Kit
│   ├── ImageKit/                     # 图像 Kit
│   ├── MediaKit/                     # 媒体 Kit
│   └── MediaLibraryKit/              # 媒体库 Kit
├── ohos/                             # FFI 实现层
│   ├── file/
│   │   └── photo_access_helper/      # 相册访问实现
│   └── multimedia/
│       ├── camera/                   # 相机实现
│       ├── image/                    # 图像实现
│       └── media/                    # 媒体实现
├── mock/                             # Mock 实现（Windows/Mac 编译用）
├── test/                             # 测试代码（本文档不涉及）
├── wiki/                             # 本文档目录
├── BUILD.gn                          # 根构建文件
├── bundle.json                       # 组件配置
├── LICENSE                           # 许可证
├── OAT.xml                           # 开源合规配置
├── README.md                         # 项目说明（英文）
└── README_zh.md                      # 项目说明（中文）
```

---

## Kit 层目录 (`kit/`)

### 职责
Kit 层是对外暴露的最简接口，仅负责导出 ohos 层的模块。

### 文件列表

| 文件路径 | 职责 | 代码内容 |
|----------|------|----------|
| `kit/CameraKit/index.cj` | 导出相机模块 | `public import ohos.multimedia.camera.*` |
| `kit/ImageKit/index.cj` | 导出图像模块 | `public import ohos.multimedia.image.*` |
| `kit/MediaKit/index.cj` | 导出媒体模块 | `public import ohos.multimedia.media.*` |
| `kit/MediaLibraryKit/index.cj` | 导出相册模块 | `public import ohos.file.photo_access_helper.*` |

### 构建配置

```gn
# kit/CameraKit/BUILD.gn
ohos_cangjie_shared_library("kit.CameraKit") {
  sources = ["index.cj"]
  cj_deps = ["../../ohos/multimedia/camera:ohos.multimedia.camera"]
}
```

---

## FFI 实现层目录 (`ohos/`)

### 相机模块 (`ohos/multimedia/camera/`)

#### 文件职责表

| 文件名 | 职责 | 关键内容 |
|--------|------|----------|
| `camera_ffi.cj` | FFI 函数声明与 C 结构体 | `foreign {}` 块，包含所有 C 函数声明；`@C struct` 定义 |
| `camera.cj` | 相机入口函数 | `getCameraManager()` 函数 |
| `camera_manager.cj` | CameraManager 类 | 相机设备管理、能力查询、输入/输出创建 |
| `camera_session.cj` | Session 相关类 | 会话配置、添加输入输出、启动停止 |
| `camera_input.cj` | CameraInput 类 | 相机硬件控制（打开/关闭/错误处理） |
| `camera_output.cj` | 输出基类 | CameraOutput 基类定义 |
| `photo_output.cj` | PhotoOutput 类 | 拍照控制、拍照事件回调 |
| `preview_output.cj` | PreviewOutput 类 | 预览控制、帧率设置 |
| `video_output.cj` | VideoOutput 类 | 录像控制、录像事件回调 |
| `camera_ability.cj` | 相机能力相关 | 相机能力查询实现 |
| `camera_common.cj` | 公共定义 | 错误码映射、枚举定义、工具函数 |

#### 类继承关系

```
RemoteDataLite (来自 cangjie_ark_interop)
    ↑
CameraManager
CameraInput
CameraOutput
    ↑
    ├── PreviewOutput
    ├── PhotoOutput
    └── VideoOutput

Session
    ├── PhotoSession
    └── VideoSession
```

### 图像模块 (`ohos/multimedia/image/`)

#### 文件职责表

| 文件名 | 职责 | 关键内容 |
|--------|------|----------|
| `image.cj` | Image 类 | 图像对象操作（clipRect、size、format、getComponent） |
| `image_source.cj` | ImageSource 类 | 图像源（解码、属性获取） |
| `image_packer.cj` | ImagePacker 类 | 图像打包（编码） |
| `image_receiver.cj` | ImageReceiver 类 | 图像接收器 |
| `pixel_map.cj` | PixelMap 类 | 像素图操作（读写像素、信息获取） |
| `cj_image_common.cj` | 公共类型 | ImageInfo、Size、Region、Component、DecodingOptions 等 |
| `cj_image_enum.cj` | 枚举定义 | PixelMapFormat、AlphaType、ScaleMode 等 |
| `cj_image_log.cj` | 日志封装 | IMAGE_LOG 定义 |
| `cj_image_utils.cj` | 工具函数 | 颜色空间转换等 |

#### 关键数据结构

```cangjie
// cj_image_common.cj
public class ImageInfo {
    public var size: Size
    public var density: Int32
    public var stride: Int32
    public var pixelFormat: PixelMapFormat
    public var alphaType: AlphaType
    public var mimeType: String
    public var isHdr: Bool
}

public class Size {
    public var height: Int32
    public var width: Int32
}

public class Region {
    public var size: Size
    public var x: Int32
    public var y: Int32
}
```

### 媒体模块 (`ohos/multimedia/media/`)

#### 文件职责表

| 文件名 | 职责 | 关键内容 |
|--------|------|----------|
| `avimage_generator.cj` | AVImageGenerator 类 | 视频缩略图生成 |
| `media_ffi.cj` | FFI 类型定义 | CAVFileDescriptor、CPixelMapParams 等 C 结构体 |
| `media_common.cj` | 公共定义 | 媒体相关常量 |

#### 关键类

```cangjie
// avimage_generator.cj
@!APILevel[single: "22", syscap: "SystemCapability.Multimedia.Media.AVImageGenerator"]
public class AVImageGenerator <: RemoteDataLite {
    public func fetchFrameByTime(timeUs: Int64, option: QueryOption, param: PixelMapParams): PixelMap
    public func fetchFrameByIndex(index: Int32, option: QueryOption, param: PixelMapParams): PixelMap
}
```

### 相册模块 (`ohos/file/photo_access_helper/`)

#### 文件职责表

| 文件名 | 职责 | 关键内容 |
|--------|------|----------|
| `photo_accesshelper.cj` | PhotoAccessHelper 类 | 入口类（getAssets、getAlbums、registerChange） |
| `photo_accesshelper_ffi.cj` | FFI 函数与结构体 | `foreign {}` 块，`@C struct` 定义 |
| `photo_accesshelper_utils.cj` | 工具函数 | 错误检查、日志 |
| `photo_asset.cj` | PhotoAsset 类 | 媒体资源操作（获取缩略图、URI、文件名等） |
| `album.cj` | Album 类 | 相册操作（获取名称、数量、封面等） |
| `fetch_result.cj` | FetchResult 类 | 查询结果迭代器 |
| `media_asset_change_request.cj` | MediaAssetChangeRequest | 资源变更请求（创建、删除） |
| `media_album_change_request.cj` | MediaAlbumChangeRequest | 相册变更请求（重命名、添加/移除资源） |

#### 关键权限

```cangjie
// photo_accesshelper.cj:85
@!APILevel[
    since: "22",
    permission: "ohos.permission.READ_IMAGEVIDEO",  // 读权限
    syscap: "SystemCapability.FileManagement.PhotoAccessHelper.Core",
    throwexception: true,
    workerthread: true
]
public func getAssets(options: FetchOptions): PhotoAssetResult

// photo_accesshelper.cj:343
@!APILevel[
    since: "22",
    permission: "ohos.permission.WRITE_IMAGEVIDEO",  // 写权限
    syscap: "SystemCapability.FileManagement.PhotoAccessHelper.Core",
    throwexception: true,
    workerthread: true
]
public func applyChanges(mediaChangeRequest: MediaChangeRequest): Unit
```

---

## Mock 目录 (`mock/`)

### 职责
为非 Linux 平台（Windows、Mac）提供空实现，使代码能够编译通过。

### 文件列表

| 文件 | 对应模块 | 内容 |
|------|----------|------|
| `ohos.multimedia.cj` | ohos.multimedia | 空实现 |
| `ohos.multimedia.camera.cj` | ohos.multimedia.camera | 空实现 |
| `ohos.multimedia.image.cj` | ohos.multimedia.image | 空实现 |
| `ohos.multimedia.media.cj` | ohos.multimedia.media | 空实现 |
| `ohos.file.photo_access_helper.cj` | ohos.file.photo_access_helper | 空实现 |

### 构建条件

```gn
# ohos/multimedia/camera/BUILD.gn:20-36
if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.camera.cj"]
} else {
    sources = [
        "camera.cj",
        "camera_ability.cj",
        // ... 完整实现
    ]
}
```

---

## 代码统计

### 源文件数量

| 模块 | .cj 文件数（不含测试） | 主要代码行数估算 |
|------|----------------------|-----------------|
| kit/ | 4 | ~80 |
| ohos/multimedia/camera/ | 10 | ~2500 |
| ohos/multimedia/image/ | 9 | ~2000 |
| ohos/multimedia/media/ | 3 | ~300 |
| ohos/file/photo_access_helper/ | 8 | ~1800 |
| mock/ | 5 | ~100 |
| **总计** | **39** | **~4780** |

---

## 文件命名规范

| 模式 | 用途 | 示例 |
|------|------|------|
| `*_ffi.cj` | FFI 声明文件 | `camera_ffi.cj`, `photo_accesshelper_ffi.cj` |
| `cj_*_common.cj` | 公共定义文件 | `cj_image_common.cj`, `cj_image_enum.cj` |
| `*_common.cj` | 模块公共文件 | `camera_common.cj`, `media_common.cj` |
| `index.cj` | Kit 入口文件 | `kit/CameraKit/index.cj` |
| `ohos.*.cj` | Mock 文件 | `ohos.multimedia.camera.cj` |

---

## 关键结论

1. **分层清晰**：kit/（对外）→ ohos/（FFI实现）→ mock/（平台适配）
2. **职责单一**：每个 .cj 文件职责明确，如 `camera_ffi.cj` 仅声明 FFI 函数
3. **命名规范**：通过文件名即可判断文件职责
4. **平台适配**：通过 mock/ 目录支持非 Linux 平台编译

---

## 相关跳转

- [架构说明](01_Architecture.md)
- [N-API 接口](03_NAPI_Interfaces.md)
- [GN Targets](05_GN_Targets.md)
