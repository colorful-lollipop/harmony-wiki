# 其他 API

## AVMetadataExtractor

### 模块信息

- **命名空间**: ohos.multimedia
- **实现文件**: `frameworks/js/metadatahelper/avmetadataextractor_napi.cpp`
- **注册入口**: `AVMetadataExtractorNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| setSource | 设置数据源 | string: url | number: 错误码 |
| resolveMetadata | 解析元数据 | - | object: metadata |
| fetchAlbumCover | 获取封面 | - | PixelMap |
| release | 释放资源 | - | number: 错误码 |

## AVImageGenerator

### 模块信息

- **命名空间**: ohos.multimedia
- **实现文件**: `frameworks/js/metadatahelper/avimagegenerator_napi.cpp`
- **注册入口**: `AVImageGeneratorNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| initMediaSource | 初始化数据源 | MediaSource | number: 错误码 |
| fetchFrameByTime | 获取帧图片 | number: time, type | PixelMap |
| release | 释放资源 | - | number: 错误码 |

## AVScreenCapture

### 模块信息

- **命名空间**: ohos.multimedia.avscreen_capture
- **实现文件**: `frameworks/js/avscreen_capture/avscreen_capture_napi.cpp`
- **注册入口**: `AVScreenCaptureNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| init | 初始化 | - | number: 错误码 |
| setAudioSource | 设置音频源 | AudioSourceType | number: 错误码 |
| setVideoSource | 设置视频源 | VideoSourceType | number: 错误码 |
| start | 开始录制 | - | number: 错误码 |
| stop | 停止录制 | - | number: 错误码 |
| pause | 暂停录制 | - | number: 错误码 |
| resume | 恢复录制 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |

### 权限要求

```cpp
// 文件: frameworks/js/avscreen_capture/avscreen_capture_napi.cpp:303
ThrowCustomError(env, MSERR_EXT_API9_PERMISSION_DENIED, "permission denied");
```

## ScreenCaptureMonitor

### 模块信息

- **命名空间**: ohos.multimedia
- **实现文件**: `frameworks/js/screencapturemonitor/screen_capture_monitor_napi.cpp`
- **注册入口**: `ScreenCaptureMonitorNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| on | 订阅事件 | string: event | - |
| off | 取消订阅 | string: event | - |

## MediaSource

### 模块信息

- **命名空间**: ohos.multimedia.mediasource
- **实现文件**: `frameworks/js/mediasource/media_source_napi.cpp`
- **注册入口**: `MediaSourceNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| constructor | 创建 MediaSource | - | MediaSource |
| setSrc | 设置数据源 | string: url | - |
| setAuthInfo | 设置认证信息 | AuthInfo | - |

## 相关文档

- [播放器 API](06_AVPlayer.md)
- [录制器 API](07_AVRecorder.md)
