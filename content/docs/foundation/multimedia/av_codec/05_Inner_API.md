# 05_Inner API

## 概述

Inner API 是 av_codec 部件**内部模块之间的接口**，位于 `interfaces/inner_api/native/` 目录。这些接口不对外暴露，仅用于框架层与服务层之间的通信。

## Inner API 清单

### 核心 Filter 接口

| 接口文件 | 职责 | 关键类 |
|----------|------|--------|
| `audio_decoder_filter.h` | 音频解码 Filter | `AudioDecoderFilter` |
| `audio_encoder_filter.h` | 音频编码 Filter | `AudioEncoderFilter` |
| `video_decoder_adapter.h` | 视频解码适配器 | `VideoDecoderAdapter` |
| `audio_sink_filter.h` | 音频 Sink Filter | `AudioSinkFilter` |
| `audio_capture_filter.h` | 音频采集 Filter | `AudioCaptureFilter` |
| `surface_decoder_filter.h` | Surface 解码 Filter | `SurfaceDecoderFilter` |
| `muxer_filter.h` | 封装 Filter | `MuxerFilter` |
| `demuxer_filter.h` | 解封装 Filter | `DemuxerFilter` |
| `video_sink_filter.h` | 视频 Sink Filter | `VideoSinkFilter` |

**证据**: `bundle.json:262-277`
```json
{
  "type": "so",
  "name": "//foundation/multimedia/av_codec/services/media_engine/filters:av_codec_media_engine_filters",
  "header": {
    "header_files": [
      "audio_decoder_filter.h",
      "audio_sink_filter.h",
      "audio_capture_filter.h",
      "audio_encoder_filter.h",
      "video_capture_filter.h",
      "surface_encoder_filter.h",
      "muxer_filter.h",
      "codec_capability_adapter.h"
    ],
    "header_base": "//foundation/multimedia/av_codec/interfaces/inner_api/native"
  }
}
```

### 编解码核心接口

| 接口文件 | 职责 |
|----------|------|
| `avcodec_audio_codec.h` | 统一音频编解码 |
| `avcodec_audio_decoder.h` | 音频解码 |
| `avcodec_audio_encoder.h` | 音频编码 |
| `avcodec_video_decoder.h` | 视频解码 |
| `avcodec_video_encoder.h` | 视频编码 |

### 公共类型

| 接口文件 | 职责 |
|----------|------|
| `av_common.h` | 公共类型定义 |
| `avcodec_errors.h` | 错误码定义 |
| `avcodec_common.h` | 公共常量和结构 |
| `avcodec_monitor.h` | 监控接口 |
| `media_description.h` | 媒体描述 |

### 扩展接口

| 接口文件 | 职责 |
|----------|------|
| `audio_base_codec_ext.h` | 音频编解码扩展 |
| `avcodec_codec_name.h` | 编解码器名称 |
| `avcodec_info.h` | 编解码信息 |
| `avcodec_list.h` | 编解码列表 |
| `avcodec_mime_type.h` | MIME 类型定义 |

## 模块依赖方向

```
┌─────────────────────────────────────────────────────────────────┐
│                         框架层                                    │
│                  frameworks/native/capi/                         │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Inner API 层                          │    │
│  │         interfaces/inner_api/native/*.h                 │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│  ┌────────────────────────▼────────────────────────────────┐    │
│  │                    服务层                                 │    │
│  │              services/engine/*/                          │    │
│  │              services/media_engine/*/                    │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## 稳定性标注

### 稳定接口

以下接口为**内部稳定接口**，可被框架层可靠调用：

| 接口 | 稳定性依据 |
|------|-----------|
| Filter 基类接口 | `services/media_engine/filters/` 目录下统一定义 |
| 基础错误码 | `avcodec_errors.h` 全局定义 |
| 公共类型 | `av_common.h` 核心类型 |

### 内部模块

以下模块为**内部实现**，外部不应直接依赖：

| 模块 | 证据 | 说明 |
|------|------|------|
| `services/engine/codec/` | `services/engine/codec/BUILD.gn` | 编解码核心引擎 |
| `services/media_engine/modules/` | `services/media_engine/modules/BUILD.gn` | 媒体处理模块 |
| `services/dfx/` | `services/dfx/BUILD.gn` | 调试和监控 |

## 关键数据结构

### Filter 生命周期

```cpp
// 1. 创建
std::shared_ptr<AudioDecoderFilter> filter = AudioDecoderFilter::Create();

// 2. 配置
filter->Configure(format);

// 3. 准备
filter->Prepare();

// 4. 启动
filter->Start();

// 5. 停止
filter->Stop();

// 6. 释放
filter->Release();
```

### Buffer 传递

Inner API 层使用 `AVBuffer` 进行零拷贝数据传递：

| 类型 | 用途 |
|------|------|
| `AVMemory` | 内存模式（简单但可能有拷贝） |
| `AVBuffer` | Buffer 模式（支持零拷贝） |

**证据**: `interfaces/kits/c/native_avcodec_base.h:51-52`
```cpp
typedef struct OH_AVCodec OH_AVCodec;
typedef struct NativeWindow OHNativeWindow;
```

---

**相关文档**: [对外 C-API](04_C_API.md) | [GN 构建系统](06_Build_System.md)
