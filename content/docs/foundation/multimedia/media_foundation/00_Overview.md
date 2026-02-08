# HiStreamer 媒体引擎概览

## 项目定位

HiStreamer 是 OpenHarmony 多媒体子系统的**核心媒体引擎组件**，提供播放、录制等场景的媒体数据流水线处理能力。

### 核心能力

| 能力 | 说明 |
|------|------|
| **媒体播放** | 支持音频/视频播放，构建 Source→Demuxer→Decoder→Sink 流水线 |
| **媒体录制** | 支持音视频录制，构建 Capture→Encoder→Muxer→Sink 流水线 |
| **格式支持** | 通过插件系统支持多种封装/解封装格式 |
| **编解码** | 支持软编解码（FFmpeg、Minimp3）和硬编解码（HDI） |
| **跨设备** | 支持 Mini/Small/Standard 三种系统配置 |

### 系统定位

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (ArkTS/JS)                         │
├─────────────────────────────────────────────────────────────┤
│               player_framework (N-API 绑定)                   │
├─────────────────────────────────────────────────────────────┤
│                   HiStreamer (本仓库)                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐  │
│  │ HiPlayer    │ │ HiRecorder  │ │ Pipeline Framework  │  │
│  │ 播放场景    │ │ 录制场景    │ │ Filter/Plugin 框架  │  │
│  └─────────────┘ └─────────────┘ └─────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│              audio_framework / graphic / ...                 │
└─────────────────────────────────────────────────────────────┘
```

## 逻辑架构

HiStreamer 内部分为**三层**：

### 1. 应用场景封装层

负责将复杂的 Pipeline 封装为易用的播放/录制 API。

| 模块 | 路径 | 职责 |
|------|------|------|
| HiPlayer | `engine/scene/player/` | 播放场景封装 |
| HiRecorder | `engine/scene/recorder/` | 录制场景封装 |
| PlayExecutor | `engine/scene/player/` | 播放执行器接口 |
| RecorderExecutor | `engine/scene/recorder/` | 录制执行器接口 |

### 2. Pipeline 框架层

负责媒体处理流水线的编排与调度。

| 组件 | 路径 | 职责 |
|------|------|------|
| PipelineCore | `engine/pipeline/core/` | Pipeline 编排器 |
| Filter | `engine/pipeline/core/` | Filter 基类 |
| Port | `engine/pipeline/core/` | 端口连接 |
| FilterFactory | `engine/pipeline/factory/` | Filter 工厂 |

### 3. 插件层

负责具体的媒体数据处理（解封装、解码、输出等）。

| 插件类型 | 示例 | 路径 |
|----------|------|------|
| Source | FileSource, HttpSource | `engine/plugin/plugins/source/` |
| Demuxer | FFmpegDemuxer, AacDemuxer | `engine/plugin/plugins/demuxer/` |
| Decoder | FFmpegDecoder, Minimp3Decoder | `engine/plugin/plugins/ffmpeg_adapter/` |
| Encoder | FFmpegEncoder, HdiCodecAdapter | `engine/plugin/plugins/codec_adapter/` |
| Sink | AudioSink, VideoSink | `engine/plugin/plugins/sink/` |

## 运行环境

### 支持的系统类型

| 系统类型 | 标识 | 特点 |
|----------|------|------|
| Mini | L0 | 静态链接，限制功能 |
| Small | L2 | 轻量系统，动态库 |
| Standard | L1/L2 | 全功能，动态库 |

### 配置文件

- `config.gni`: Feature flags 定义
- `bundle.json`: 组件配置、Syscap 声明

### 系统能力

```
SystemCapability.Multimedia.Media.Core
SystemCapability.Multimedia.VideoProcessingEngine
```

## 关键概念

### 数据流模式

| 模式 | 说明 |
|------|------|
| **Push** | 上游 Filter 主动推送数据到下游 |
| **Pull** | 下游 Filter 主动从上游拉取数据 |

### Filter 状态机

```
CREATED → INITIALIZED → PREPARING → READY → RUNNING → PAUSED
```

### 插件能力 (Capability)

用于描述插件支持的媒体格式、分辨率、码率等能力，用于插件匹配。

## 相关仓库

| 仓库 | 职责 |
|------|------|
| [multimedia_subsystem](...) | 多媒体子系统总览 |
| [player_framework](...) | N-API 绑定层（JS/ArkTS 接口） |
| [media_lite](...) | 轻量设备媒体库 |
