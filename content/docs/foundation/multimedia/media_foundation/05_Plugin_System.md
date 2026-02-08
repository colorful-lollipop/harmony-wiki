# 插件系统架构

## 概述

HiStreamer 采用**插件化架构**，将媒体处理功能（解封装、编解码、输出等）抽象为插件，便于扩展和替换。

## 插件类型

| 类型 | 枚举值 | 用途 |
|------|--------|------|
| SOURCE | `PluginType::SOURCE` | 数据源（文件、网络） |
| DEMUXER | `PluginType::DEMUXER` | 解封装 |
| AUDIO_DECODER | `PluginType::AUDIO_DECODER` | 音频解码 |
| VIDEO_DECODER | `PluginType::VIDEO_DECODER` | 视频解码 |
| AUDIO_ENCODER | `PluginType::AUDIO_ENCODER` | 音频编码 |
| VIDEO_ENCODER | `PluginType::VIDEO_ENCODER` | 视频编码 |
| AUDIO_SINK | `PluginType::AUDIO_SINK` | 音频输出 |
| VIDEO_SINK | `PluginType::VIDEO_SINK` | 视频输出 |
| MUXER | `PluginType::MUXER` | 封装 |
| OUTPUT_SINK | `PluginType::OUTPUT_SINK` | 输出目标 |

## 核心接口

### PluginBase

**头文件**: `engine/include/plugin/interface/plugin_base.h:66`

所有插件的基类：

```cpp
struct PluginBase {
    virtual ~PluginBase() = default;

    // 生命周期
    virtual Status Init() = 0;
    virtual Status Deinit() = 0;
    virtual Status Prepare() = 0;
    virtual Status Reset() = 0;
    virtual Status Start() = 0;
    virtual Status Stop() = 0;

    // 参数管理
    virtual Status GetParameter(Tag tag, ValueType &value) = 0;
    virtual Status SetParameter(Tag tag, const ValueType &value) = 0;

    // 内存分配器
    virtual std::shared_ptr<Allocator> GetAllocator() = 0;

    // 事件回调
    virtual Status SetCallback(Callback* cb) = 0;
};
```

### SourcePlugin

**头文件**: `engine/include/plugin/interface/source_plugin.h`

数据源插件接口：

```cpp
struct SourcePlugin : public PluginBase {
    // 数据读取
    virtual Status Read(std::shared_ptr<Buffer>& buffer, size_t expectedLen) = 0;

    // 定位
    virtual Status SeekTo(int64_t offset) = 0;

    // 获取媒体信息
    virtual std::shared_ptr<Meta> GetSourceMeta() = 0;
};
```

### DemuxerPlugin

**头文件**: `engine/include/plugin/interface/demuxer_plugin.h`

解封装插件接口：

```cpp
struct DemuxerPlugin : public PluginBase {
    // 获取解封装后的数据
    virtual Status GetNextChunk(TrackType type, std::shared_ptr<Buffer>& buffer) = 0;

    // 定位
    virtual Status SeekToTime(int64_t trackIdx, int64_t time) = 0;

    // 获取轨道信息
    virtual std::shared_ptr<Meta> GetStreamMeta(TrackType type) = 0;
};
```

### CodecPlugin

**头文件**: `engine/include/plugin/interface/codec_plugin.h`

编解码插件接口：

```cpp
struct CodecPlugin : public PluginBase {
    // 获取输入缓冲区信息
    virtual Status GetInputInfo(VideoCodecBufferAttr& attr) = 0;

    // 处理数据
    virtual Status Process(const std::shared_ptr<Buffer>& inBuffer,
                          std::shared_ptr<Buffer>& outBuffer) = 0;
};
```

### AudioSinkPlugin

**头文件**: `engine/include/plugin/interface/audio_sink_plugin.h`

音频输出插件接口：

```cpp
struct AudioSinkPlugin : public PluginBase {
    // 写入数据
    virtual Status Write(const std::shared_ptr<Buffer>& buffer) = 0;

    // 获取延迟
    virtual int64_t GetLatency() = 0;
};
```

## 插件注册机制

### PLUGIN_DEFINITION 宏

**头文件**: `engine/include/plugin/interface/plugin_definition.h:191`

```cpp
#define PLUGIN_DEFINITION(name, license, registerFunc, unregisterFunc) \
    PLUGIN_EXPORT Status register_##name(                               \
        const std::shared_ptr<PackageRegister>& pkgReg)                 \
    {                                                                   \
        pkgReg->AddPackage({PLUGIN_INTERFACE_VERSION,                   \
                           PLUGIN_STRINGIFY(name), license});           \
        std::shared_ptr<Register> pluginReg = pkgReg;                   \
        return registerFunc(pluginReg);                                  \
    }                                                                   \
    PLUGIN_EXPORT void unregister_##name()                              \
    {                                                                   \
        unregisterFunc();                                                \
    }
```

### 注册函数签名

```cpp
// 注册函数类型
using RegisterFunc = Status (*)(std::shared_ptr<Register> reg);

// 注销函数类型
using UnregisterFunc = void (*)();
```

### 完整注册示例

```cpp
// minimp3_decoder_plugin.cpp
Status RegisterDecoderPlugin(const std::shared_ptr<Register>& reg) {
    CodecPluginDef definition;
    definition.name = "Minimp3DecoderPlugin";
    definition.pluginType = PluginType::AUDIO_DECODER;
    definition.rank = MAX_RANK;  // 优先级
    definition.creator = Minimp3DecoderCreator;  // 创建函数

    // 设置能力
    UpdatePluginDefinition(definition);

    return reg->AddPlugin(definition);
}

PLUGIN_DEFINITION(Minimp3Decoder, LicenseType::CC0,
                  RegisterDecoderPlugin, [] {});
```

## 能力系统 (Capability)

### Capability 结构

**头文件**: `engine/include/plugin/common/plugin_caps.h:44`

```cpp
struct Capability {
    std::string mime;              // MIME 类型
    KeyMap keys;                   // 能力键值对

    // 设置 MIME
    Capability& SetMime(std::string val);

    // 添加固定键
    template<typename T>
    Capability& AppendFixedKey(Key key, const T& val);

    // 添加区间键
    template<typename T>
    Capability& AppendIntervalKey(Key key, T rangeStart, T rangeEnd);

    // 添加离散键
    template<typename T>
    Capability& AppendDiscreteKeys(Key key, DiscreteCapability<T> vals);
};
```

### 能力键 (Tag)

**头文件**: `engine/include/plugin/common/plugin_tags.h`

| 分类 | Tag | 说明 |
|------|-----|------|
| 音频 | AUDIO_SAMPLE_RATE | 采样率 |
| 音频 | AUDIO_CHANNELS | 通道数 |
| 音频 | AUDIO_SAMPLE_FORMAT | 采样格式 |
| 视频 | VIDEO_WIDTH | 宽度 |
| 视频 | VIDEO_HEIGHT | 高度 |
| 视频 | VIDEO_PIXEL_FORMAT | 像素格式 |
| 通用 | MEDIA_BITRATE | 码率 |

### 能力定义示例

```cpp
CapabilityBuilder builder;
builder.SetMime(OHOS::Media::MEDIA_MIME_AUDIO_MPEG)
    .SetAudioSampleRate(44100)
    .SetAudioChannels(2)
    .SetAudioSampleFormat(AudioSampleFormat::SAMPLE_S16LE);
```

## 插件管理器

###头文件**: `engine/include PluginManager

**/plugin/core/plugin_manager.h`

单例模式，管理所有已注册插件：

```cpp
class PluginManager {
public:
    static PluginManager& Instance();

    // 创建插件
    std::shared_ptr<DemuxerPlugin> CreateDemuxerPlugin(const std::string& name);
    std::shared_ptr<CodecPlugin> CreateCodecPlugin(const std::string& name, PluginType type);
    std::shared_ptr<SourcePlugin> CreateSourcePlugin(const std::string& name);
    std::shared_ptr<AudioSinkPlugin> CreateAudioSinkPlugin(const std::string& name);

    // 列出插件
    std::vector<std::string> ListPlugins(PluginType type);
};
```

## 插件适配器

### FFmpeg Adapter

**路径**: `engine/plugin/plugins/ffmpeg_adapter/`

封装 FFmpeg 为 HiStreamer 插件：

| 子模块 | 说明 |
|--------|------|
| audio_decoder | FFmpeg 音频解码器 |
| video_decoder | FFmpeg 视频解码器 |
| audio_encoder | FFmpeg 音频编码器 |
| video_encoder | FFmpeg 视频编码器 |
| demuxer | FFmpeg 解封装器 |
| muxer | FFmpeg 封装器 |

### HDI Codec Adapter

**路径**: `engine/plugin/plugins/codec_adapter/`

封装硬件编解码器 HDI 接口：

```cpp
// 硬件编解码器适配
class HdiCodecAdapter : public CodecPlugin {
    // 实现 HDI 接口与 Plugin 接口的转换
};
```

### HDI Audio Adapter

**路径**: `engine/plugin/plugins/hdi_adapter/`

封装音频 HDI 输出：

```cpp
// 音频 HDI 输出适配
class HosAudioSink : public AudioSinkPlugin {
    // 使用 Audio HDI 进行音频播放
};
```

### Minimp3 Adapter

**路径**: `engine/plugin/plugins/minimp3_adapter/`

轻量级 MP3 解码器：

- 解封装 + 解码一体化
- 无外部依赖（CC0 许可）

## 插件目录结构

```
engine/plugin/plugins/
├── codec_adapter/           # HDI Codec 适配器
│   ├── hdi_codec_adapter.h
│   ├── codec_manager.h
│   └── ...
├── ffmpeg_adapter/          # FFmpeg 适配器
│   ├── audio_decoder/
│   ├── video_decoder/
│   ├── audio_encoder/
│   ├── video_encoder/
│   ├── demuxer/
│   └── muxer/
├── hdi_adapter/             # HDI 适配器
│   └── sink/
├── minimp3_adapter/         # Minimp3 适配器
├── demuxer/                 # 原生 Demuxer
│   ├── aac_demuxer/
│   ├── wav_demuxer/
│   └── minimp4_demuxer/
├── sink/                    # Sink 插件
│   ├── audio_server_sink/
│   ├── file_sink/
│   └── sdl/
└── source/                  # Source 插件
    ├── file_source/
    ├── http_source/
    └── stream_source/
```

## 开发新插件

### 步骤 1：继承插件接口

```cpp
class MyDemuxerPlugin : public DemuxerPlugin {
public:
    explicit MyDemuxerPlugin(std::string name) : PluginBase(name) {}

    // 实现接口方法...
    Status Init() override { ... }
    Status GetNextChunk(TrackType type,
                        std::shared_ptr<Buffer>& buffer) override { ... }
};
```

### 步骤 2：实现创建函数

```cpp
std::shared_ptr<MyDemuxerPlugin> MyDemuxerCreator(const std::string& name) {
    return std::make_shared<MyDemuxerPlugin>(name);
}
```

### 步骤 3：注册插件

```cpp
Status RegisterMyDemuxer(const std::shared_ptr<Register>& reg) {
    DemuxerPluginDef def;
    def.name = "MyDemuxerPlugin";
    def.pluginType = PluginType::DEMUXER;
    def.rank = 50;
    def.creator = MyDemuxerCreator;

    // 设置能力
    CapabilityBuilder builder;
    builder.SetMime("video/mp4");
    def.outCaps.push_back(builder.Build());

    return reg->AddPlugin(def);
}

PLUGIN_DEFINITION(MyDemuxer, LicenseType::APACHE_V2,
                  RegisterMyDemuxer, [] {});
```

### 步骤 4：添加到 BUILD.gn

```gn
ohos_shared_library("my_demuxer_plugin") {
    sources = [
        "my_demuxer_plugin.cpp",
    ]
    deps = [
        "//foundation/multimedia/media_foundation/engine/plugin:plugin_base",
    ]
    output_name = "libmy_demuxer_plugin"
}
```
