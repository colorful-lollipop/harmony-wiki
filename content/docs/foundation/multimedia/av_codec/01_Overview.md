# 01_项目概述

## 项目定位

av_codec 是 OpenHarmony 媒体子系统的**核心编解码组件**，为系统提供统一的音视频编解码、封装/解封装能力。

**证据**: `bundle.json:2-4`
```json
{
  "name": "@ohos/av_codec",
  "description": "Media standard provides atomic capabilities",
  "version": "3.1"
}
```

## 核心能力

| 能力 | 描述 | SysCap |
|------|------|--------|
| **音频编解码** | AAC/MP3/G711 等音频编码解码 | `SystemCapability.Multimedia.Media.AudioCodec` |
| **视频编解码** | H.264/H.265/VP8/VP9/AV1 等视频编码解码 | `SystemCapability.Multimedia.Media.VideoDecoder/Encoder` |
| **解封装** | MP4/MKV/TS 等容器格式解析 | `SystemCapability.Multimedia.Media.Splitter` |
| **封装** | MP4/TS 等容器格式生成 | `SystemCapability.Multimedia.Media.Muxer` |
| **能力查询** | 编解码器能力信息查询 | `SystemCapability.Multimedia.Media.CodecBase` |

**证据**: `bundle.json:15-24`
```json
"syscap": [
  "SystemCapability.Multimedia.Media.Muxer",
  "SystemCapability.Multimedia.Media.Spliter",
  "SystemCapability.Multimedia.Media.AudioCodec",
  "SystemCapability.Multimedia.Media.AudioDecoder",
  "SystemCapability.Multimedia.Media.AudioEncoder",
  "SystemCapability.Multimedia.Media.VideoDecoder",
  "SystemCapability.Multimedia.Media.VideoEncoder",
  "SystemCapability.Multimedia.Media.CodecBase"
]
```

## 子系统归属

- **子系统**: `multimedia`
- **部件名**: `av_codec`
- **系统类型**: `standard`（仅标准系统）

**证据**: `bundle.json:13-14`
```json
"name": "av_codec",
"subsystem": "multimedia",
"adapted_system_type": [ "standard" ]
```

## 运行环境

| 环境 | 支持情况 |
|------|---------|
| OpenHarmony Standard (标准系统) | ✅ 支持 |
| OpenHarmony Lite (轻量系统) | ❌ 不支持 |

## 关键概念

### Codec (编解码器)

编解码器分为两类：

1. **硬件编解码器 (HCodec)**: 使用专用硬件加速，效率高但受硬件限制
2. **软件编解码器 (SCodec)**: 使用 CPU 计算，兼容性好但性能较低

### Surface

视频编解码器使用 Surface 作为输入/输出缓冲区，避免数据拷贝：
- **解码器输入**: Surface 接收编码后的视频数据
- **解码器输出**: Surface 渲染解码后的帧
- **编码器输入**: Surface 采集原始视频帧
- **编码器输出**: Surface 输出编码后的数据

### Buffer vs Memory

- **Buffer 模式**: 使用 `OH_AVBuffer` 管理，支持零拷贝
- **Memory 模式**: 使用 `OH_AVMemory` 管理，简单但可能有拷贝

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 3.1 | 2026-02 | 当前版本 |

## 相关仓库

| 仓库 | 职责 |
|------|------|
| `av_codec` | Native C API 与服务实现（本文档对应仓库） |
| `media_library` | N-API JavaScript 绑定层 |
| `media_foundation` | 媒体基础框架 |
| `audio_framework` | 音频框架 |

---

**相关文档**: [架构设计](03_Architecture.md) | [C-API 文档](04_C_API.md)
