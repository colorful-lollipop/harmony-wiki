# 编译产物说明

## 概述

本文档描述 HiStreamer 编译产物的类型、安装路径和运行时加载关系。

## 产物类型

### 动态库 (.so)

| 库名 | 路径 | 用途 |
|------|------|------|
| `libmedia_foundation.so` | `system/lib/` | 核心库 |
| `libnative_media_core.so` | `system/lib/` | C API 库 |
| `libhistreamer_base.so` | `system/lib/` | Pipeline 基础 |
| `libhistreamer_codec_filters.so` | `system/lib/` | 编解码 Filter |
| `libhistreamer_plugin_base.so` | `system/lib/` | 插件基础 |
| `libhistreamer_ffmpeg_convert.so` | `system/lib/` | FFmpeg 转换 |

### 插件库 (.so)

| 库名 | 安装路径 | 用途 |
|------|----------|------|
| `libFFmpegAudioDecoders.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 音频解码器 |
| `libFFmpegVideoDecoders.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 视频解码器 |
| `libFFmpegDemuxer.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 解封装器 |
| `libFFmpegAudioEncoders.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 音频编码器 |
| `libFFmpegVideoEncoders.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 视频编码器 |
| `libFFmpegMuxers.so` | `system/lib/media/histreamer_plugins/` | FFmpeg 封装器 |
| `libCodecAdapter.so` | `system/lib/media/histreamer_plugins/` | HDI Codec 适配器 |

### 服务库 (.so)

| 库名 | 用途 |
|------|------|
| `libmedia_monitor.so` | 媒体监控服务 |
| `libmedia_monitor_client.so` | 监控客户端 |
| `libmedia_monitor_wrapper.so` | 监控包装 |
| `libmedia_monitor_buffer.so` | Buffer 监控 |
| `libmedia_monitor_common.so` | 监控公共库 |

## 产物清单

### C API 产物

| 头文件 | 库 | 产物路径 |
|--------|------|----------|
| `native_averrors.h` | libnative_media_core.so | system/lib/ |
| `native_avmemory.h` | libnative_media_core.so | system/lib/ |
| `native_avbuffer.h` | libnative_media_core.so | system/lib/ |
| `native_avformat.h` | libnative_media_core.so | system/lib/ |
| `native_avbuffer_info.h` | libnative_media_core.so | system/lib/ |

### Inner API 产物

| 头文件目录 | 库 | 产物路径 |
|-----------|------|----------|
| `interface/inner_api/` | libmedia_foundation.so | system/lib/ |

### NDK 产物

| 头文件 | 库 | 产物路径 |
|--------|------|----------|
| `interface/kits/ndk/core/` | libnative_media_core.so | system/lib/ndk/ |

## 安装路径

### 标准系统 (Standard)

```
/system/lib/
├── libmedia_foundation.so
├── libnative_media_core.so
├── libhistreamer_base.so
├── libhistreamer_codec_filters.so
├── libhistreamer_plugin_base.so
├── libhistreamer_ffmpeg_convert.so
└── media/histreamer_plugins/
    ├── libFFmpegDemuxer.so
    ├── libFFmpegAudioDecoders.so
    ├── libFFmpegVideoDecoders.so
    ├── libFFmpegAudioEncoders.so
    ├── libFFmpegVideoEncoders.so
    ├── libFFmpegMuxers.so
    └── libCodecAdapter.so
```

### 轻量系统 (Small)

```
/system/lib/
├── libmedia_foundation.so
└── media/histreamer_plugins/
    ├── libFFmpegDemuxer.so
    ├── libaudio_sink_lib.so
    └── ...
```

### Mini 系统 (L0)

```
# 静态链接，无独立 .so
# 相关代码编译入系统库
```

## 运行时加载关系

### 依赖图

```
┌─────────────────────────────────────────────────────────┐
│                    应用层                               │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              player_framework (N-API)                   │
│              libmedia_ext.so                           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              libnative_media_core.so (C API)           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              libmedia_foundation.so                     │
│    ┌──────────────────────────────────────────────┐    │
│    │ Pipeline Core                               │    │
│    │  - FilterBase                              │    │
│    │  - FilterFactory                           │    │
│    │  - PipelineCore                            │    │
│    └──────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              libhistreamer_base.so                      │
└─────────────────────────────────────────────────────────┘
                          ↓
        ┌─────────────────┬─────────────────┐
        ↓                 ↓                 ↓
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│Plugin Libraries│ │Filter Libraries│ │ Service Libs │
│ (按需加载)     │ │               │ │              │
└───────────────┘ └───────────────┘ └───────────────┘
```

### 动态加载

插件库使用 `dlopen()` 动态加载：

```cpp
// plugin_loader.cpp
void* handle = dlopen(pluginPath.c_str(), RTLD_LAZY);
void* symbol = dlsym(handle, "register_PluginName");
```

## 符号导出

### 插件导出符号

每个插件库导出以下符号：

```cpp
// 注册函数
extern "C" Status register_PluginName(std::shared_ptr<PackageRegister> reg);

// 注销函数
extern "C" void unregister_PluginName();
```

### 可见性控制

```cpp
#if defined(WIN32) || defined(_WIN32)
#define PLUGIN_EXPORT extern "C" __declspec(dllexport)
#else
#if defined(__GNUC__)
#define PLUGIN_EXPORT extern "C" __attribute__((visibility("default")))
#else
#define PLUGIN_EXPORT
#endif
#endif
```

## 配置产物

### 配置文件

| 文件 | 用途 | 路径 |
|------|------|------|
| `media_monitor.para` | 监控参数 | /etc/param/ |
| `media_monitor_init` | 初始化配置 | /init/ |
| `sa_profile` | SA 配置 | /sa_profile/ |

## 版本信息

### 产物版本

| 产物 | 版本 | 说明 |
|------|------|------|
| libmedia_foundation.so | 3.1 | 与 bundle.json 一致 |
| libnative_media_core.so | 3.1 | C API 库 |

### Syscap 声明

```
SystemCapability.Multimedia.Media.Core
SystemCapability.Multimedia.VideoProcessingEngine
```
