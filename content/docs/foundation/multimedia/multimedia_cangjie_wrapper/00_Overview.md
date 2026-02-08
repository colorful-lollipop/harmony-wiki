# 项目概览

> 本文档介绍 multimedia_cangjie_wrapper 的项目定位、核心能力、运行环境与关键概念

---

## 目的与适用范围

**目的**：为仓颉（Cangjie）编程语言提供统一的多媒体接口封装，使开发者能够使用仓颉语言访问 OpenHarmony 的相机、图像、媒体和相册功能。

**适用范围**：
- OpenHarmony Standard 标准设备
- 使用仓颉语言开发的应用
- 需要访问系统多媒体资源的场景

---

## 项目定位

### 在系统中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层                                   │
│         (Cangjie Application / ArkTS Application)             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             │
│  │ CameraKit   │ │  ImageKit   │ │  MediaKit   │             │
│  │(仓颉接口)   │ │(仓颉接口)   │ │(仓颉接口)   │             │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘             │
├─────────┼───────────────┼───────────────┼─────────────────────┤
│         │               │               │     MediaLibraryKit │
│  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐             │
│  │ohos.multim  │ │ohos.multim  │ │ohos.multim  │ ┌──────────┐ │
│  │edia.camera  │ │edia.image   │ │edia.media   │ │ohos.file │ │
│  │(FFI层)      │ │(FFI层)      │ │(FFI层)      │ │.photo_   │ │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ │access_   │ │
│         │               │               │        │helper    │ │
├─────────┼───────────────┼───────────────┼────────┴─────┬─────┤
│         │               │               │              │     │
│  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼────┐│
│  │ camera_     │ │ image_      │ │player_frame-│ │media_lib- ││
│  │ framework   │ │ framework   │ │work         │ │rary       ││
│  │(C++底层)    │ │(C++底层)    │ │(C++底层)    │ │(C++底层)  ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └──────────┘│
└─────────────────────────────────────────────────────────────┘
                              ↑
                   multimedia_cangjie_wrapper
```

### 核心功能矩阵

| 功能模块 | CameraKit | ImageKit | MediaKit | MediaLibraryKit |
|---------|-----------|----------|----------|-----------------|
| **预览** | ✅ | - | - | - |
| **拍照** | ✅ | - | - | - |
| **录像** | ✅ | - | - | - |
| **图像解码** | - | ✅ | - | - |
| **图像编码** | - | ✅ | - | - |
| **视频缩略图** | - | - | ✅ | - |
| **相册读取** | - | - | - | ✅ |
| **相册修改** | - | - | - | ✅ |

---

## 核心能力与限制

### 支持功能（相比 ArkTS）

| 模块 | 支持功能 | 证据位置 |
|------|----------|----------|
| CameraKit | 预览、拍照、录像、手电筒控制 | `ohos/multimedia/camera/camera_ffi.cj` |
| ImageKit | 图像编解码、PixelMap 操作 | `ohos/multimedia/image/` |
| MediaKit | 视频缩略图获取（AVImageGenerator） | `ohos/multimedia/media/avimage_generator.cj` |
| MediaLibraryKit | 相册创建、访问、修改 | `ohos/file/photo_access_helper/` |

### 不支持功能（相比 ArkTS）

来源：`README.md:94-108`

| 模块 | 不支持功能 | 备注 |
|------|-----------|------|
| Camera | 闪光灯模式 | - |
| Camera | 图片质量设置 | - |
| Camera | 色彩空间参数 | - |
| Image | 多图对象 | - |
| Image | 图像元数据 | - |
| Media | 音视频播放 | 本仓库仅支持缩略图 |
| Media | 音视频录制 | - |
| Media | 视频转码 | - |
| Media | 音视频元数据获取 | - |
| Media | 屏幕录制 | - |
| 其他 Kit | Audio Kit | 暂不支持 |
| 其他 Kit | AVCodec Kit | 暂不支持 |
| 其他 Kit | AVSession Kit | 暂不支持 |
| 其他 Kit | DRM Kit | 暂不支持 |
| 其他 Kit | Ringtone Kit | 暂不支持 |
| 其他 Kit | Scan Kit | 暂不支持 |

---

## 运行环境

### 系统要求

| 属性 | 要求 | 证据 |
|------|------|------|
| 系统类型 | Standard（标准设备） | `bundle.json:18-19` |
| API 级别 | 22+ | `camera_common.cj` 等多处 `@!APILevel[single: "22"]` |
| 语言 | Cangjie（仓颉） | 项目定位 |

### 资源占用

来源：`bundle.json:20-21`

| 资源类型 | 占用量 |
|----------|--------|
| ROM | 1300 KB |
| RAM | 1212 KB |

### 依赖组件

来源：`bundle.json:24-36`

#### Wrapper 层依赖
- `ability_cangjie_wrapper` - Ability 上下文接口
- `bundlemanager_cangjie_wrapper` - 包管理接口
- `cangjie_ark_interop` - 仓颉-ArkTS 互操作（BusinessException、FFI）
- `distributeddatamgr_cangjie_wrapper` - 分布式数据管理
- `global_cangjie_wrapper` - 资源管理接口
- `graphic_cangjie_wrapper` - 色彩管理类型定义
- `hiviewdfx_cangjie_wrapper` - 日志接口（HiLog）

#### 底层多媒体框架
- `media_library` - 相册基础功能
- `camera_framework` - 相机基础功能
- `image_framework` - 图像基础功能
- `player_framework` - 播放器基础功能

---

## 关键概念

### 1. FFI（Foreign Function Interface）层

本仓库的核心技术模式，通过 FFI 调用底层 C/C++ 多媒体框架：

```cangjie
// 示例：camera_ffi.cj 中的 FFI 声明
foreign {
    func FfiCameraManagerConstructor(): Int64
    func FfiCameraManagerGetSupportedCameras(id: Int64, errCode: CPointer<Int32>): CArrCCameraDevice
    // ...
}
```

### 2. Kit 层 vs ohos 层

| 层级 | 作用 | 位置 | 示例 |
|------|------|------|------|
| Kit 层 | 对外暴露的简洁接口 | `kit/*/index.cj` | `kit.CameraKit` |
| ohos 层 | FFI 封装与业务逻辑 | `ohos/**/*.cj` | `ohos.multimedia.camera` |

### 3. RemoteDataLite 模式

仓颉对象通过 ID 引用底层 C++ 对象：

```
Cangjie 对象 (RemoteDataLite)
    ↓ 持有 Int64 ID
底层 C++ 对象 (通过 FFI 创建/操作)
```

证据：`ohos/multimedia/camera/camera_manager.cj:34`
```cangjie
public class CameraManager <: RemoteDataLite {
    init() {
        super(unsafe { FfiCameraManagerConstructor() })  // 获取底层对象 ID
    }
}
```

### 4. 回调机制

使用 `Callback1Param` 实现异步回调：

证据：`ohos/multimedia/camera/camera_manager.cj:392-398`
```cangjie
let wrapper = {value: CCameraStatusInfo => callback.invoke(None, value.toCameraStatusInfo())}
let lambdaData = Callback1Param<CCameraStatusInfo, Unit>(wrapper)
let errCode = FfiCameraManagerOnCameraStatusChanged(getID(), lambdaData.getID())
```

### 5. 错误处理

统一使用 `BusinessException`：

证据：`ohos/multimedia/camera/camera_common.cj:62-66`
```cangjie
func successOrThrow(errCode: Int32): Unit {
    if (errCode != SUCCESS_CODE) {
        throw BusinessException(errCode, getErrorMsg(errCode))
    }
}
```

---

## 版本信息

| 属性 | 值 |
|------|-----|
| 组件名称 | multimedia_cangjie_wrapper |
| 版本 | 6.1 |
| 许可证 | Apache License 2.0 |
| 状态 | Beta Feature |
| 子系统 | multimedia |

---

## 相关链接

- [仓库 README](../README.md)
- [架构说明](01_Architecture.md)
- [N-API 接口](03_NAPI_Interfaces.md)
