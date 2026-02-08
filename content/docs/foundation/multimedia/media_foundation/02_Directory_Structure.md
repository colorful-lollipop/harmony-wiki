# 目录结构与模块职责

## 根目录概览

```
media_foundation/
├── engine/                    # 引擎代码（核心）
│   ├── foundation/            # 基础工具类
│   ├── include/               # 对外头文件
│   ├── pipeline/              # Pipeline 框架
│   ├── plugin/                # 插件系统
│   └── scene/                 # 场景封装
├── interface/                 # 对外接口
│   ├── inner_api/             # 内部 API
│   └── kits/                  # NDK/C API
├── services/                  # 服务
│   └── media_monitor/        # 媒体监控
├── src/                       # C API 实现
├── BUILD.gn                   # 根构建配置
├── config.gni                 # Feature flags 配置
├── bundle.json                # 组件配置
└── wiki/                      # 文档（本文档）
```

## engine/ 目录详解

### engine/foundation/

**职责**: 提供基础工具类和操作系统适配层。

```
engine/foundation/
├── osal/                      # 操作系统适配层
│   ├── task/                  # 任务/线程抽象
│   │   ├── pthread/          # POSIX 线程实现
│   │   └── ffrt/             # FFRT 实现（可选）
│   └── filesystem/            # 文件系统操作
└── utils/                     # 工具类
    ├── constants.cpp          # 常量定义
    └── ...
```

**关键模块**:

| 模块 | 文件 | 职责 |
|------|------|------|
| Task | `osal/task/task.h` | 任务抽象 |
| Thread | `osal/task/thread.h` | 线程抽象 |
| Mutex | `osal/task/mutex.h` | 互斥锁 |
| ConditionVariable | `osal/task/condition_variable.h` | 条件变量 |
| FileSystem | `osal/filesystem/file_system.h` | 文件操作 |

### engine/include/

**职责**: 为其他模块提供调用 HiStreamer 的必要头文件。

```
engine/include/
├── foundation/                # 基础工具库头文件
│   ├── logging/              # 日志打印
│   ├── buffer/               # Buffer 工具
│   ├── osal/                 # OS 适配
│   └── utils/                # 工具函数
├── pipeline/                 # Pipeline 框架头文件
│   ├── core/                 # Pipeline/Filter 接口
│   ├── factory/              # Filter 工厂
│   └── filters/              # Filter 接口
└── plugin/                   # 插件头文件
    ├── common/               # 插件基础类型
    ├── interface/             # 插件接口
    └── factory/               # 插件工厂
```

### engine/pipeline/

**职责**: Pipeline 框架实现，包括 Filter 基类、工厂、数据流管理。

```
engine/pipeline/
├── core/                     # 核心实现
│   ├── pipeline_core.h/cpp   # Pipeline 编排器
│   ├── filter_base.h/cpp     # Filter 基类
│   ├── filter.h              # Filter 接口
│   ├── port.h/cpp            # 端口连接
│   └── event.h               # 事件类型
├── factory/                   # Filter 工厂
│   ├── filter_factory.h/cpp  # Filter 创建
│   └── auto_register.h       # 自动注册模板
└── filters/                   # Filter 实现
    ├── codec/                 # 编解码 Filter
    │   ├── audio_decoder/    # 音频解码 Filter
    │   ├── video_decoder/    # 视频解码 Filter
    │   ├── audio_encoder/    # 音频编码 Filter
    │   └── video_encoder/    # 视频编码 Filter
    ├── demux/                 # 解封装 Filter
    │   └── demuxer_filter.*  # Demuxer Filter
    ├── muxer/                 # 封装 Filter
    │   └── muxer_filter.*    # Muxer Filter
    ├── sink/                  # 输出 Filter
    │   ├── audio_sink/        # 音频输出 Filter
    │   ├── video_sink/        # 视频输出 Filter
    │   └── output_sink/       # 目标输出 Filter
    └── source/                # 数据源 Filter
        ├── media_source/      # 媒体源 Filter
        ├── audio_capture/     # 音频采集 Filter
        └── video_capture/     # 视频采集 Filter
```

### engine/plugin/

**职责**: 插件框架和插件实现。

```
engine/plugin/
├── core/                     # 插件框架核心
│   ├── plugin_manager.h/cpp  # 插件管理器
│   ├── plugin_register.h/cpp # 插件注册表
│   ├── plugin_loader.*        # 动态库加载
│   ├── codec.h/cpp           # Codec 封装
│   ├── demuxer.h/cpp         # Demuxer 封装
│   ├── source.h/cpp          # Source 封装
│   └── sink.h/cpp            # Sink 封装
├── common/                    # 公共类型
│   ├── plugin_types.h        # 插件类型枚举
│   ├── plugin_caps.h         # 能力描述
│   ├── plugin_tags.h         # 标签定义
│   ├── plugin_buffer.h       # 插件 Buffer
│   └── plugin_event.h        # 插件事件
├── interface/                 # 插件接口
│   ├── plugin_base.h         # 插件基类
│   ├── plugin_definition.h  # 插件定义宏
│   ├── source_plugin.h       # Source 接口
│   ├── demuxer_plugin.h      # Demuxer 接口
│   ├── codec_plugin.h        # Codec 接口
│   ├── muxer_plugin.h        # Muxer 接口
│   ├── audio_sink_plugin.h   # Audio Sink 接口
│   └── video_sink_plugin.h   # Video Sink 接口
├── factory/                   # 插件工厂
│   └── plugin_factory.h      # 插件创建工厂
├── convert/                   # 转换工具
│   └── ffmpeg_convert.h      # FFmpeg 转换
└── plugins/                   # 插件实现
    ├── codec_adapter/         # HDI Codec 适配器
    ├── ffmpeg_adapter/       # FFmpeg 适配器
    ├── hdi_adapter/          # HDI 适配器
    ├── minimp3_adapter/      # Minimp3 适配器
    ├── demuxer/              # 原生 Demuxer
    ├── sink/                  # Sink 插件
    └── source/                # Source 插件
```

### engine/scene/

**职责**: 播放、录制等场景的高层封装。

```
engine/scene/
├── common/                    # 公共类型
│   └── ...
├── lite/                     # 轻量设备接口
├── player/                   # 播放场景
│   ├── standard/             # 标准设备实现
│   │   ├── hiplayer_impl.h/cpp    # HiPlayer 主实现
│   │   └── play_executor.h        # 播放执行器接口
│   └── ...
├── recorder/                 # 录制场景
│   ├── standard/             # 标准设备实现
│   │   ├── hirecorder_impl.h/cpp  # HiRecorder 主实现
│   │   └── recorder_executor.h    # 录制执行器接口
│   └── ...
└── ...
```

## interface/ 目录详解

### interface/inner_api/

**职责**: HiStreamer 内部模块间的 API。

```
interface/inner_api/
├── buffer/                   # Buffer 缓冲区
├── meta/                     # 元数据
├── filter/                   # Filter 接口
├── pipeline/                  # Pipeline 接口
├── plugin/                   # 插件接口
├── common/                   # 公共类型
├── cpp_ext/                  # C++ 扩展
├── osal/                     # OS 适配
└── network/                  # 网络相关
```

### interface/kits/

**职责**: 外部调用接口（C API、NDK API）。

```
interface/kits/
├── c/                         # C API 头文件
│   ├── native_averrors.h      # 错误码
│   ├── native_avmemory.h      # 内存管理 (deprecated)
│   ├── native_avbuffer.h      # 缓冲区
│   ├── native_avformat.h      # 格式参数
│   ├── native_avbuffer_info.h # 缓冲区属性
│   └── native_audio_channel_layout.h
└── ndk/                       # NDK API
    └── core/                  # NDK 核心接口
```

## services/ 目录详解

### services/media_monitor/

**职责**: 媒体监控服务，收集和上报媒体相关事件。

```
services/media_monitor/
├── server/                    # 服务端实现
├── client/                    # 客户端 API
├── common/                    # 公共定义
├── buffer/                    # Buffer 监控
├── sa_profile/               # SA 配置文件
└── test/                      # 测试代码
```

## src/ 目录详解

**职责**: C API 实现代码。

```
src/
├── capi/                      # C API 实现
│   ├── native_avbuffer.cpp
│   ├── native_avformat.cpp
│   ├── native_avmemory.cpp
│   └── common/
├── buffer/                    # Buffer 实现
├── common/                    # 公共实现
├── filter/                    # Filter 实现
├── meta/                      # 元数据实现
├── osal/                      # OS 适配实现
├── pipeline/                  # Pipeline 实现
└── plugin/                    # 插件框架实现
```
