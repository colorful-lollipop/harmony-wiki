# 04_对外 C-API

## 概述

av_codec 对外暴露 **Native C API**，位于 `interfaces/kits/c/` 目录。这些 C API 是整个部件对外能力的基础入口。

**重要说明**: 本仓库**不包含 N-API JavaScript 绑定层**。JavaScript/ArkTS 接口位于独立仓库（如 `media_library`），通过调用这些 C API 实现功能。

## API 清单

### 1. 视频解码器 (Video Decoder)

**头文件**: `native_avcodec_videodecoder.h`

#### 创建与销毁

| JS API | C API | 描述 | 绑定位置 |
|--------|-------|------|----------|
| `createVideoDecoder()` | `OH_VideoDecoder_CreateByMime()` | 通过 MIME 类型创建 | `native_avcodec_videodecoder.h:60` |
| `createVideoDecoderByName()` | `OH_VideoDecoder_CreateByName()` | 通过名称创建 | `native_avcodec_videodecoder.h:73` |
| - | `OH_VideoDecoder_Destroy()` | 销毁解码器 | `native_avcodec_videodecoder.h:89` |

#### 生命周期

| C API | 参数 | 返回值 | 描述 |
|-------|------|--------|------|
| `OH_VideoDecoder_RegisterCallback()` | `OH_AVCodec *codec, OH_AVCodecCallback callback, void *userData` | `OH_AVErrCode` | 注册回调 |
| `OH_VideoDecoder_SetSurface()` | `OH_AVCodec *codec, OHNativeWindow *window` | `OH_AVErrCode` | 设置输出 Surface |
| `OH_VideoDecoder_Configure()` | `OH_AVCodec *codec, const OH_AVFormat *format` | `OH_AVErrCode` | 配置解码器 |
| `OH_VideoDecoder_Prepare()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 准备资源 |
| `OH_VideoDecoder_Start()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 启动解码 |
| `OH_VideoDecoder_Stop()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 停止解码 |
| `OH_VideoDecoder_Flush()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 刷新缓冲区 |
| `OH_VideoDecoder_Reset()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 重置解码器 |

#### 数据处理

| C API | 参数 | 返回值 | 描述 |
|-------|------|--------|------|
| `OH_VideoDecoder_GetInputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVMemory*` | 获取输入缓冲区 |
| `OH_VideoDecoder_PushInputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVErrCode` | 提交输入数据 |
| `OH_VideoDecoder_RenderOutputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVErrCode` | 渲染输出帧 |
| `OH_VideoDecoder_FreeOutputBuffer()` | `OH_VideoDecoder *codec, uint32_t index` | `OH_AVErrCode` | 释放输出缓冲区 |

---

### 2. 视频编码器 (Video Encoder)

**头文件**: `native_avcodec_videoencoder.h`

#### 创建与销毁

| JS API | C API | 描述 | 绑定位置 |
|--------|-------|------|----------|
| `createVideoEncoder()` | `OH_VideoEncoder_CreateByMime()` | 通过 MIME 创建 | `native_avcodec_videoencoder.h:73` |
| `createVideoEncoderByName()` | `OH_VideoEncoder_CreateByName()` | 通过名称创建 | `native_avcodec_videoencoder.h:86` |
| - | `OH_VideoEncoder_Destroy()` | 销毁编码器 | `native_avcodec_videoencoder.h:102` |

#### 生命周期

| C API | 参数 | 返回值 | 描述 |
|-------|------|--------|------|
| `OH_VideoEncoder_RegisterCallback()` | `OH_AVCodec *codec, OH_AVCodecCallback callback, void *userData` | `OH_AVErrCode` | 注册回调 |
| `OH_VideoEncoder_Configure()` | `OH_AVCodec *codec, const OH_AVFormat *format` | `OH_AVErrCode` | 配置编码器 |
| `OH_VideoEncoder_Prepare()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 准备资源 |
| `OH_VideoEncoder_Start()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 启动编码 |
| `OH_VideoEncoder_Stop()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 停止编码 |
| `OH_VideoEncoder_Flush()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 刷新缓冲区 |
| `OH_VideoEncoder_Reset()` | `OH_AVCodec *codec` | `OH_AVErrCode` | 重置编码器 |
| `OH_VideoEncoder_GetSurface()` | `OH_AVCodec *codec, OHNativeWindow **window` | `OH_AVErrCode` | 获取输入 Surface |

#### 数据处理

| C API | 参数 | 返回值 | 描述 |
|-------|------|--------|------|
| `OH_VideoEncoder_GetInputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVMemory*` | 获取输入缓冲区 |
| `OH_VideoEncoder_PushInputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVErrCode` | 提交输入数据 |
| `OH_VideoEncoder_FreeOutputBuffer()` | `OH_AVCodec *codec, uint32_t index` | `OH_AVErrCode` | 释放输出缓冲区 |

---

### 3. 解封装器 (Demuxer)

**头文件**: `native_avdemuxer.h`

| JS API | C API | 描述 | 绑定位置 |
|--------|-------|------|----------|
| `createMediaDemuxer()` | `OH_AVDemuxer_CreateWithSource()` | 通过 Source 创建 | `native_avdemuxer.h:79` |
| - | `OH_AVDemuxer_Destroy()` | 销毁解封装器 | `native_avdemuxer.h:93` |
| `selectTrackByID()` | `OH_AVDemuxer_SelectTrackByID()` | 选择轨道 | `native_avdemuxer.h:113` |
| `unselectTrackByID()` | `OH_AVDemuxer_UnselectTrackByID()` | 取消选择 | `native_avdemuxer.h:130` |
| `readSampleBuffer()` | `OH_AVDemuxer_ReadSampleBuffer()` | 读取采样数据 | `native_avdemuxer.h:182` |
| `seekToTime()` | `OH_AVDemuxer_SeekToTime()` | 跳转到时间戳 | `native_avdemuxer.h:206` |

---

### 4. 封装器 (Muxer)

**头文件**: `native_avmuxer.h`

| JS API | C API | 描述 | 绑定位置 |
|--------|-------|------|----------|
| `createMediaMuxer()` | `OH_AVMuxer_Create()` | 创建封装器 | `native_avmuxer.h:58` |
| - | `OH_AVMuxer_SetRotation()` | 设置旋转角度 | `native_avmuxer.h:72` |
| `addTrack()` | `OH_AVMuxer_AddTrack()` | 添加轨道 | `native_avmuxer.h:105` |
| `start()` | `OH_AVMuxer_Start()` | 开始封装 | `native_avmuxer.h:119` |
| `writeSampleBuffer()` | `OH_AVMuxer_WriteSampleBuffer()` | 写入采样数据 | `native_avmuxer.h:161` |
| `stop()` | `OH_AVMuxer_Stop()` | 停止封装 | `native_avmuxer.h:175` |
| - | `OH_AVMuxer_Destroy()` | 销毁封装器 | `native_avmuxer.h:186` |

---

### 5. 能力查询 (Capability)

**头文件**: `native_avcapability.h`

| JS API | C API | 描述 | 绑定位置 |
|--------|-------|------|----------|
| `getCodecCapability()` | `OH_AVCodec_GetCapability()` | 按 MIME 查询能力 | `native_avcapability.h:105` |
| `getCodecCapabilityByCategory()` | `OH_AVCodec_GetCapabilityByCategory()` | 按类型查询 | `native_avcapability.h:118` |
| `isHardware()` | `OH_AVCapability_IsHardware()` | 是否硬件加速 | `native_avcapability.h:128` |
| `getName()` | `OH_AVCapability_GetName()` | 获取编解码器名称 | `native_avcapability.h:137` |
| `isFeatureSupported()` | `OH_AVCapability_IsFeatureSupported()` | 查询特性支持 | `native_avcapability.h:480` |

---

## 共享库产物

| 库文件 | 导出能力 | 证据 |
|--------|---------|------|
| `libnative_media_vdec.so` | 视频解码 | `interfaces/kits/c/BUILD.gn:320-352` |
| `libnative_media_venc.so` | 视频编码 | `interfaces/kits/c/BUILD.gn:354-379` |
| `libnative_media_adec.so` | 音频解码 | `interfaces/kits/c/BUILD.gn:266-291` |
| `libnative_media_aenc.so` | 音频编码 | `interfaces/kits/c/BUILD.gn:293-318` |
| `libnative_media_avdemuxer.so` | 解封装 | `interfaces/kits/c/BUILD.gn:133-165` |
| `libnative_media_avmuxer.so` | 封装 | `interfaces/kits/c/BUILD.gn:105-131` |
| `libnative_media_avsource.so` | 媒体源 | `interfaces/kits/c/BUILD.gn:167-196` |
| `libnative_media_codecbase.so` | 基础能力 | `interfaces/kits/c/BUILD.gn:198-226` |
| `libnative_media_avcencinfo.so` | CENC 加密信息 | `interfaces/kits/c/BUILD.gn:381-409` |

---

## 错误码

所有 API 返回 `OH_AVErrCode` 类型错误码：

| 错误码 | 值 | 描述 |
|--------|-----|------|
| `AV_ERR_OK` | 0 | 成功 |
| `AV_ERR_OPERATE_NOT_PERMIT` | 1 | 操作不允许 |
| `AV_ERR_OPERATE_NOT_SUPPORT` | 2 | 操作不支持 |
| `AV_ERR_NO_MEMORY` | 3 | 内存不足 |
| `AV_ERR_NO_DATA` | 4 | 无数据 |
| `AV_ERR_DATA_INVALID` | 5 | 数据无效 |
| `AV_ERR_IO` | 6 | IO 错误 |
| `AV_ERR_TIMEOUT` | 7 | 超时 |
| `AV_ERR_SERVICE_DIED` | 8 | 服务已终止 |
| `AV_ERR_INVALID_VAL` | 9 | 参数无效 |
| `AV_ERR_UNKNOWN` | 10 | 未知错误 |

**证据**: `interfaces/kits/c/native_avcodec_base.h` 错误码定义

---

## 使用示例

### 创建视频解码器

```cpp
#include "native_avcodec_videodecoder.h"

// 1. 创建解码器
OH_AVCodec *codec = OH_VideoDecoder_CreateByMime("video/avc");
if (codec == nullptr) {
    // 错误处理
}

// 2. 配置解码器
OH_AVFormat *format = OH_AVFormat_Create();
OH_AVFormat_SetIntValue(format, OH_MD_KEY_WIDTH, 1920);
OH_AVFormat_SetIntValue(format, OH_MD_KEY_HEIGHT, 1080);
OH_AVFormat_SetIntValue(format, OH_MD_KEY_FRAME_RATE, 30);
OH_VideoDecoder_Configure(codec, format);

// 3. 注册回调
OH_VideoDecoder_RegisterCallback(codec, callback, nullptr);

// 4. 准备
OH_VideoDecoder_Prepare(codec);

// 5. 启动
OH_VideoDecoder_Start(codec);
```

---

**相关文档**: [架构设计](03_Architecture.md) | [Inner API](05_Inner_API.md) | [故障排查指南](08_Troubleshooting.md)
