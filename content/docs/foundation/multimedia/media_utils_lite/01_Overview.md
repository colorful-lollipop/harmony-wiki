# 项目概述

## 组件定位

`media_utils_lite` 是 OpenHarmony 多媒体子系统的**公共基础组件**，负责定义音频/视频录制和播放所需的**公共数据类型、错误码及 HAL 抽象接口**。

### 核心职责

1. **数据类型定义**: 提供 SourceType、AudioSourceType、CodecFormat 等枚举和结构体
2. **错误码体系**: 定义多媒体操作的统一错误码
3. **HAL 抽象层**: 定义相机、显示、视频处理的硬件抽象接口
4. **数据流抽象**: 提供 DataBuffer/DataStream 数据流接口

### 定位图示

```
┌─────────────────────────────────────────────────────────────┐
│                    Multimedia Subsystem                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│  │ camera_lite  │  │ audio_lite   │  │    media_lite        │ │
│  │ (相机功能)   │  │ (音频功能)   │  │    (媒体播放/录制)   │ │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬─────────┘ │
│         │                 │                     │           │
│         └────────────┬────┴─────────────┬────────┘           │
│                      │                 │                     │
│              ┌───────▼───────────────▼───────┐              │
│              │      media_utils_lite          │ ◀── 本组件 │
│              │   (公共类型/错误码/HAL接口)    │              │
│              └───────────────────────────────┘              │
│                          │                                  │
│                          ▼                                  │
│              ┌───────────────────────────────┐              │
│              │   third_party/bounds_check    │              │
│              │   hiviewdfx/hilog_lite       │              │
│              │   graphic/surface_lite        │              │
│              └───────────────────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

| 能力类别 | 具体内容 |
|----------|----------|
| **数据类型** | SourceType, AudioSourceType, AudioCodecFormat, VideoCodecFormat, AudioBitWidth, AudioStreamType |
| **错误码** | 12 种错误码定义，含通用错误和多媒体专用错误 |
| **HAL 接口** | 视频处理器、相机、显示输出的硬件抽象 |
| **数据流** | DataBuffer, DataStream, DataProducer, DataConsumer |
| **日志** | 基于 HiLog 的媒体日志宏 |

## 运行环境

| 属性 | 值 |
|------|-----|
| **目标系统** | OpenHarmony mini/small 系统 |
| **内核支持** | LiteOS-M, Linux |
| **最低 C++ 标准** | C++11 |
| **ROM 占用** | ~1024 kB |
| **RAM 占用** | ~500 kB |

## 目录结构

```
media_utils_lite/
├── interfaces/kits/          # 公共头文件 (对外 API)
│   ├── media_errors.h        # 错误码定义
│   ├── source.h              # 媒体源类型与类
│   ├── format.h              # 格式化数据结构
│   ├── media_info.h          # 媒体信息枚举
│   ├── data_stream.h         # 数据流接口
│   └── media_log.h           # 日志宏定义
│
├── hals/                     # HAL 适配层接口
│   ├── hal_media.h           # 视频处理器 HAL
│   ├── hal_camera.h          # 相机 HAL
│   └── hal_display.h         # 显示输出 HAL
│
├── src/                      # 实现代码
│   ├── format.cpp            # Format/FormatData 实现
│   └── source.cpp            # Source/StreamSource 实现
│
├── figures/                  # 架构图资源
├── BUILD.gn                  # GN 构建配置
├── bundle.json               # 组件描述文件
├── config.gni                # 构建参数配置
└── README.md                 # 项目说明
```

### 目录职责说明

| 目录 | 职责 | 稳定性 |
|------|------|--------|
| `interfaces/kits/` | 公共头文件，供其他模块引用 | 稳定 |
| `hals/` | HAL 接口定义，需与硬件适配层对接 | 较稳定 |
| `src/` | Format/Source 等核心数据结构实现 | 稳定 |

## 与其他组件的关系

### 被依赖 (Inner Kit)

以下组件通过 Inner Kit 依赖本组件：

| 组件 | 依赖内容 |
|------|----------|
| `camera_lite` | hal_camera.h, 相机能力枚举 |
| `audio_lite` | 音频格式枚举, media_errors.h |
| `media_lite` | Source, Format, DataStream 等 |

### 依赖第三方

| 依赖 | 用途 | 来源 |
|------|------|------|
| `bounds_checking_function` | 安全字符串函数 | third_party |
| `hilog_lite` | 日志框架 | hiviewdfx |
| `surface_lite` | 图形表面 | graphic |

## 版本信息

| 属性 | 值 |
|------|-----|
| **组件版本** | 3.2 |
| **Bundle 名称** | @ohos/media_utils_lite |
| **发布类型** | code-segment |
| **目标路径** | foundation/multimedia/media_utils_lite |
