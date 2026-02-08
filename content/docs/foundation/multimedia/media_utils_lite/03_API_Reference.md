# API 参考

## 概述

本章节详细描述 `media_utils_lite` 提供的所有公共 API。

**重要说明**: 本组件**不提供 N-API (JS) 接口**，所有 API 均为 **C++ 原生接口**，主要供 OpenHarmony 其他子系统（如 camera_lite、audio_lite、media_lite）使用。

## API 分类

| 分类 | 说明 | 头文件 |
|------|------|--------|
| 错误码 | 多媒体操作错误码定义 | `media_errors.h` |
| 媒体源 | Source/StreamSource 类型与操作 | `source.h` |
| 格式化数据 | Format/FormatData 键值对容器 | `format.h` |
| 媒体枚举 | 音视频编解码格式枚举 | `media_info.h` |
| 数据流 | DataBuffer/DataStream 接口 | `data_stream.h` |
| 日志宏 | 基于 HiLog 的日志输出 | `media_log.h` |
| HAL 接口 | 硬件抽象层 API | `hal_*.h` |

---

## media_errors.h - 错误码定义

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/media_errors.h` |
| 版本 | 1.0 |
| 命名空间 | `OHOS::Media` |

### 错误码基值定义

```cpp
// 错误码生成宏
constexpr int MODULE_MEDIA = 1;
constexpr int SUBSYS_MEDIA = 30;
constexpr int SUBSYSTEM_BIT_NUM = 21;
constexpr int MODULE_BIT_NUM = 16;

// 错误码基值
constexpr ErrCode ErrCodeOffset(unsigned int subsystem, unsigned int module = 0)
{
    return (subsystem << SUBSYSTEM_BIT_NUM) | (module << MODULE_BIT_NUM);
}
constexpr int32_t BASE_MEDIA_ERR_OFFSET = ErrCodeOffset(SUBSYS_MEDIA, MODULE_MEDIA);
// 结果: 0x3C10000
```

**证据来源**: `interfaces/kits/media_errors.h:44-65`

### 错误码速查表

| 错误码 (十六进制) | 宏定义 | 说明 |
|------------------|--------|------|
| `0x0` | `SUCCESS` | 操作成功 |
| `0xffffffff` | `ERR_INVALID_READ` | 读数据失败 |
| `0x3c10000` | `ERROR` | 操作失败（通用） |
| `0x3c10001` | `ERR_ILLEGAL_STATE` | 状态错误 |
| `0x3c10002` | `ERR_INVALID_PARAM` | 参数无效 |
| `0x3c10003` | `ERR_EARLY_PREPARE` | 媒体启动提前 |
| `0x3c10004` | `ERR_SOURCE_NOT_SET` | 媒体源未设置 |
| `0x3c10005` | `ERR_INVALID_OPERATION` | 无效操作 |
| `0x3c10006` | `ERR_NOFREE_CHANNEL` | 通道无空闲 |
| `0x3c10007` | `ERR_READ_BUFFER` | 读缓冲区出错 |
| `0x3c10008` | `ERR_NOT_STARTED` | 设备未启动 |
| `0x3c100c8` | `ERR_UNKNOWN` | 未知错误 |

**证据来源**: `interfaces/kits/media_errors.h:67-101`

### 使用示例

```cpp
#include "media_errors.h"

int32_t result = DoSomeMediaOperation();
if (result != OHOS::Media::SUCCESS) {
    // 处理错误
    if (result == OHOS::Media::ERR_INVALID_PARAM) {
        // 参数错误处理
    } else if (result == OHOS::Media::ERR_NOT_STARTED) {
        // 设备未启动处理
    }
}
```

---

## source.h - 媒体源

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/source.h` |
| 版本 | 1.0 |
| 命名空间 | `OHOS::Media` |

### SourceType 枚举

```cpp
enum class SourceType : int32_t {
    SOURCE_TYPE_URI = 0,      // 本地文件路径或网络地址
    SOURCE_TYPE_FD,            // 本地文件描述符
    SOURCE_TYPE_STREAM,       // 流数据（如 AAC 流）
};
```

**证据来源**: `interfaces/kits/source.h:58-65`

### StreamCallback 结构体

```cpp
struct StreamCallback {
    // 缓冲区标志
    enum BufferFlags : uint32_t {
        STREAM_FLAG_SYNCFRAME = 1,        // 同步帧
        STREAM_FLAG_CODECCONFIG = 2,     // 编解码配置信息
        STREAM_FLAG_EOS = 4,             // 流结束 (EOS)
        STREAM_FLAG_PARTIAL_FRAME = 8,   // 帧的一部分
        STREAM_FLAG_ENDOFFRAME = 16,     // 帧结束（与 PARTIAL_FRAME 配对使用）
        STREAM_FLAG_MUXER_DATA = 32,     // 容器文件数据（如 MP4）
    };

    // 获取缓冲区虚拟地址
    virtual uint8_t* GetBuffer(size_t index) = 0;

    // 将填充好的缓冲区写入播放器
    virtual void QueueBuffer(size_t index, size_t offset, size_t size,
                            int64_t timestampUs, uint32_t flags) = 0;

    // 设置流附加信息
    virtual void SetParameters(const Format &params) = 0;
};
```

**证据来源**: `interfaces/kits/source.h:74-130`

### StreamSource 类

```cpp
class StreamSource {
public:
    StreamSource(void);
    virtual ~StreamSource(void);

#ifndef SURFACE_DISABLED
    void SetSurface(Surface* surface);
    Surface* GetSurface(void);
#endif

    uint8_t* GetSharedBuffer(size_t& size);
    int QueueSharedBuffer(void* buffer, size_t size);

    // 通知应用可填充数据的缓冲区
    virtual void OnBufferAvailable(size_t index, size_t offset, size_t size) {}

    // 设置流回调函数
    virtual void SetStreamCallback(const std::shared_ptr<StreamCallback> &callback) {}

private:
#ifndef SURFACE_DISABLED
    Surface* surface_;
    SurfaceBuffer* curBuffer_;
#endif
};
```

**证据来源**: `interfaces/kits/source.h:146-187`

### Source 类

```cpp
class Source {
public:
    // 构造函数（URI 方式）
    explicit Source(const std::string& uri);
    Source(const std::string &uri, const std::map<std::string, std::string> &header);

    // 构造函数（Stream 方式）
    Source(const std::shared_ptr<StreamSource> &stream, const Format &formats);

    // 构造函数（DataConsumer 方式）
    explicit Source(const std::shared_ptr<DataConsumer> &dataConsumer);

    ~Source() = default;

    // 获取源类型
    SourceType GetSourceType() const;

    // 获取 URI（仅 SOURCE_TYPE_URI 有效）
    const std::string &GetSourceUri() const;

    // 获取 HTTP 头（仅 SOURCE_TYPE_URI 有效）
    const std::map<std::string, std::string> &GetSourceHeader() const;

    // 获取流信息（仅 SOURCE_TYPE_STREAM 有效）
    const std::shared_ptr<StreamSource> &GetSourceStream() const;

    // 获取流格式
    const Format &GetSourceStreamFormat() const;

    // 获取数据流消费者（仅 SOURCE_TYPE_STREAM 有效）
    const std::shared_ptr<DataConsumer> &GetDataConsumer() const;

private:
    std::string uri_;
    SourceType sourceType_;
    std::map<std::string, std::string> header_;
    std::shared_ptr<StreamSource> stream_;
    Format format_;
    std::shared_ptr<DataConsumer> dataConsumer_;
};
```

**证据来源**: `interfaces/kits/source.h:195-313`

---

## format.h - 格式化数据

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/format.h` |
| 版本 | 1.0 |
| 命名空间 | `OHOS::Media` |

### 预定义常量

```cpp
// 编解码 MIME 类型键
extern const char *CODEC_MIME;
extern const char *MIME_AUDIO_AAC;
extern const char *MIME_AUDIO_RAW;
extern const char *PAUSE_AFTER_PLAY;
```

**证据来源**: `interfaces/kits/format.h:47-55`

### FormatDataType 枚举

```cpp
enum FormatDataType : uint32_t {
    FORMAT_TYPE_NONE = 0,
    FORMAT_TYPE_INT32,
    FORMAT_TYPE_INT64,
    FORMAT_TYPE_FLOAT,
    FORMAT_TYPE_DOUBLE,
    FORMAT_TYPE_STRING
};
```

**证据来源**: `interfaces/kits/format.h:63-76`

### FormatData 类

```cpp
class FormatData {
public:
    explicit FormatData(FormatDataType type);
    FormatData();
    ~FormatData();

    FormatDataType GetType() const;

    // 设置值
    bool SetValue(int32_t val);
    bool SetValue(int64_t val);
    bool SetValue(float val);
    bool SetValue(double val);
    bool SetValue(const std::string &val);

    // 获取值
    bool GetInt32Value(int32_t &val) const;
    bool GetInt64Value(int64_t &val) const;
    bool GetFloatValue(float &val) const;
    bool GetDoubleValue(double &val) const;
    bool GetStringValue(std::string &val) const;

private:
    FormatDataType type_;
    union {
        int32_t int32Val;
        int64_t int64Val;
        float floatVal;
        double doubleVal;
        std::string *stringVal;
    } val_;
};
```

**证据来源**: `interfaces/kits/format.h:84-213`

### Format 类

```cpp
class Format {
public:
    Format();
    ~Format();

    // 设置元数据
    bool PutIntValue(const std::string &key, int32_t value);
    bool PutLongValue(const std::string &key, int64_t value);
    bool PutFloatValue(const std::string &key, float value);
    bool PutDoubleValue(const std::string &key, double value);
    bool PutStringValue(const std::string &key, const std::string &value);

    // 获取元数据
    bool GetIntValue(const std::string &key, int32_t &value) const;
    bool GetLongValue(const std::string &key, int64_t &value) const;
    bool GetFloatValue(const std::string &key, float &value) const;
    bool GetDoubleValue(const std::string &key, double &value) const;
    bool GetStringValue(const std::string &key, std::string &value) const;

    // 复制格式
    bool CopyFrom(const Format &format);

    // 获取格式映射
    const std::map<std::string, FormatData *> &GetFormatMap() const;

private:
    template<typename T>
    bool SetFormatCommon(const std::string &key, const T &value, FormatDataType type);
    std::map<std::string, FormatData *> formatMap_;
};
```

**证据来源**: `interfaces/kits/format.h:221-365`

---

## media_info.h - 媒体信息枚举

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/media_info.h` |
| 版本 | 1.0 |
| (无命名空间) |

### 码率模式

```cpp
const int BITRATE_MODE_CQ  = 0;   // 恒定质量模式
const int BITRATE_MODE_VBR = 1;   // 可变比特率模式
const int BITRATE_MODE_CBR = 2;   // 恒定比特率模式
```

### 颜色格式

```cpp
const int32_t COLOR_FORMAT_ARGB8888_32BIT = 16;
const int32_t COLOR_FORMAT_YUV420SP = 21;
```

### AudioSourceType 枚举

```cpp
typedef enum {
    AUDIO_SOURCE_INVALID = -1,
    AUDIO_SOURCE_DEFAULT = 0,
    AUDIO_MIC = 1,                    // 麦克风
    AUDIO_VOICE_UPLINK = 2,           // 上行语音
    AUDIO_VOICE_DOWNLINK = 3,         // 下行语音
    AUDIO_VOICE_CALL = 4,             // 语音通话
    AUDIO_CAMCORDER = 5,              // 摄像机
    AUDIO_VOICE_RECOGNITION = 6,      // 语音识别
    AUDIO_VOICE_COMMUNICATION = 7,    // 语音通信
    AUDIO_REMOTE_SUBMIX = 8,          // 远程混合
    AUDIO_UNPROCESSED = 9,            // 未处理音频
    AUDIO_VOICE_PERFORMANCE = 10,     // 语音性能
    AUDIO_ECHO_REFERENCE = 1997,      // 回声参考
    AUDIO_RADIO_TUNER = 1998,         // 广播调谐器
    AUDIO_HOTWORD = 1999,             // 热词
    AUDIO_REMOTE_SUBMIX_EXTEND = 10007, // 远程混合扩展
} AudioSourceType;
```

### AudioStreamType 枚举

```cpp
typedef enum {
    TYPE_DEFAULT = -1,
    TYPE_MEDIA = 0,                     // 媒体
    TYPE_VOICE_COMMUNICATION = 1,      // 语音通话
    TYPE_SYSTEM = 2,                    // 系统声音
    TYPE_RING = 3,                     // 铃声
    TYPE_MUSIC = 4,                    // 音乐
    TYPE_ALARM = 5,                    // 闹钟
    TYPE_NOTIFICATION = 6,              // 通知
    TYPE_BLUETOOTH_SCO = 7,            // 蓝牙 SCO
    TYPE_ENFORCED_AUDIBLE = 8,         // 强制音频
    TYPE_DTMF = 9,                     // 双音多频
    TYPE_TTS = 10,                     // 语音合成
    TYPE_ACCESSIBILITY = 11,           // 无障碍
} AudioStreamType;
```

### AudioCodecFormat 枚举

```cpp
typedef enum {
    AUDIO_DEFAULT = 0,
    PCM = 1,                           // PCM
    AAC_LC = 2,                        // AAC Low Complexity
    AAC_HE_V1 = 3,                     // AAC High Efficiency v1
    AAC_HE_V2 = 4,                     // AAC High Efficiency v2
    AAC_LD = 5,                        // AAC Low Delay
    AAC_ELD = 6,                       // AAC Enhanced Low Delay
    G711A = 7,                          // G.711 A-law
    G711U = 8,                          // G.711 u-law
    G726 = 9,                           // G.726
    FORMAT_BUTT,                       // 无效值
} AudioCodecFormat;
```

### VideoCodecFormat 枚举

```cpp
typedef enum {
    VIDEO_DEFAULT = 0,
    H264 = 2,                           // H.264/AVC
    HEVC = 5,                           // H.265/HEVC
} VideoCodecFormat;
```

### AudioBitWidth 枚举

```cpp
typedef enum {
    BIT_WIDTH_8 = 8,
    BIT_WIDTH_16 = 16,
    BIT_WIDTH_24 = 24,
    BIT_WIDTH_32 = 32,
    BIT_WIDTH_BUTT,                     // 无效值
} AudioBitWidth;
```

### AudioDeviceDesc 结构体

```cpp
typedef struct {
    std::string deviceName;              // 设备名称
    AudioSourceType inputSourceType;     // 音频输入源类型
    uint32_t deviceId;                   // 设备 ID
} AudioDeviceDesc;
```

**证据来源**: `interfaces/kits/media_info.h:40-218`

---

## data_stream.h - 数据流接口

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/data_stream.h` |
| 版本 | 1.0 |
| 命名空间 | `OHOS::Media` |

### MemoryType 枚举

```cpp
enum class MemoryType {
    VIRTUAL_ADDR = 0,    // 虚拟地址
    SURFACE_BUFFER,      // Surface 缓冲区
    SHARE_MEMORY,        // 共享内存 fd
};
```

### DataBuffer 类

```cpp
class DataBuffer {
public:
    virtual ~DataBuffer() = default;

    // 获取 EOS 状态
    virtual bool IsEos() = 0;

    // 设置 EOS 状态
    virtual void SetEos(bool isEos) = 0;

    // 获取缓冲区地址
    virtual uint8_t* GetAddress() = 0;

    // 获取缓冲区容量
    virtual size_t GetCapacity() = 0;

    // 获取有效数据大小
    virtual size_t GetSize() = 0;

    // 设置有效数据大小
    virtual void SetSize(size_t size) = 0;
};
```

### DataProducer 接口

```cpp
class DataProducer {
public:
    virtual ~DataProducer() = default;

    // 获取空缓冲区
    // timeout: 超时时间（毫秒），-1 表示无限等待
    virtual bool GetEmptyBuffer(std::shared_ptr<DataBuffer>& buffer, int timeout = -1) = 0;

    // 提交数据缓冲区
    virtual bool QueueDataBuffer(const std::shared_ptr<DataBuffer>& buffer) = 0;
};
```

### DataConsumer 接口

```cpp
class DataConsumer {
public:
    virtual ~DataConsumer() = default;

    // 获取数据缓冲区
    virtual bool GetDataBuffer(std::shared_ptr<DataBuffer>& buffer, int timeout = -1) = 0;

    // 提交空缓冲区（使用 shared_ptr）
    virtual bool QueueEmptyBuffer(const std::shared_ptr<DataBuffer>& buffer) = 0;

    // 提交空缓冲区（使用地址）
    virtual bool QueueEmptyBuffer(uint8_t* address) = 0;
};
```

### DataStream 类

```cpp
class DataStream : public DataConsumer, public DataProducer {
};
```

### 工厂函数

```cpp
// 创建 DataStream
// size: 每个缓冲区的大小
// count: 缓冲区数量
// type: 内存类型，默认为 VIRTUAL_ADDR
std::shared_ptr<DataStream> CreateDataStream(size_t size, size_t count, 
                                              MemoryType type = MemoryType::VIRTUAL_ADDR);
```

**证据来源**: `interfaces/kits/data_stream.h:29-183`

---

## media_log.h - 日志宏

### 文件信息

| 属性 | 值 |
|------|-----|
| 路径 | `interfaces/kits/media_log.h` |
| 命名空间 | `OHOS::Media` |

### 日志域和标签

```cpp
#define LOG_DOMAIN 0xD002B00
#define LOG_TAG "MultiMedia"
```

### 日志宏定义

```cpp
// 调试日志
#define MEDIA_DEBUG_LOG(fmt, ...) DECORATOR_HILOG(HILOG_DEBUG, fmt, ##__VA_ARGS__)

// 错误日志
#define MEDIA_ERR_LOG(fmt, ...) DECORATOR_HILOG(HILOG_ERROR, fmt, ##__VA_ARGS__)

// 警告日志
#define MEDIA_WARNING_LOG(fmt, ...) DECORATOR_HILOG(HILOG_WARN, fmt, ##__VA_ARGS__)

// 信息日志
#define MEDIA_INFO_LOG(fmt, ...) DECORATOR_HILOG(HILOG_INFO, fmt, ##__VA_ARGS__)

// 严重日志
#define MEDIA_FATAL_LOG(fmt, ...) DECORATOR_HILOG(HILOG_FATAL, fmt, ##__VA_ARGS__)
```

### 调试模式

```cpp
#ifndef OHOS_DEBUG
// 发布模式：仅输出日志内容
#define DECORATOR_HILOG(op, fmt, args...) \
    do { op(LOG_CORE, fmt, ##args); } while (0)
#else
// 调试模式：输出函数名、文件名、行号
#define DECORATOR_HILOG(op, fmt, args...) \
    do { op(LOG_CORE, "{%s()-%s:%d} " fmt, __FUNCTION__, __FILENAME__, __LINE__, ##args); } while (0)
#endif
```

### 返回值宏

```cpp
#define MEDIA_OK 0
#define MEDIA_INVALID_PARAM (-1)
#define MEDIA_INIT_FAIL (-2)
#define MEDIA_ERR (-3)
#define MEDIA_PERMISSION_DENIED (-4)
#define MEDIA_IPC_FAILED (-5)
```

**证据来源**: `interfaces/kits/media_log.h:22-52`

---

## HAL 接口参考

### hal_media.h

```c
// 处理器句柄
typedef int32_t HalProcessorHdl;
#define HAL_INVALID_PROCESSOR (-1)
#define HAL_MAX_VPSS_NUM 10

// 处理器属性
typedef struct {
    uint32_t width;
    uint32_t height;
    uint32_t fps;
} HalVideoProcessorAttr;

// 创建/销毁视频处理器
HalProcessorHdl HalCreateVideoProcessor(HalVideoProcessorAttr *attr);
void HalDestroyVideoProcessor();

// 获取处理器信息
void HalCameraGetProcessorAttr(HalProcessorHdl hdls[HAL_MAX_VPSS_NUM], 
                               HalVideoProcessorAttr attrs[HAL_MAX_VPSS_NUM],
                               int32_t *size);
uint32_t HalGetProcessorDeviceId(HalProcessorHdl hdl);

// 初始化
int32_t HalMediaInitialize();
int32_t HalCameraInitialize();
void HalCameraUnInitialize();

#define HAL_MEDIA_OK 0
#define HAL_MEDIA_ERR 1
```

**证据来源**: `hals/hal_media.h:27-54`

### hal_camera.h

```c
// 常量定义
#define CAMERA_FPS_MAX_NUM 16
#define CAMERA_DESC_MAX_LEN 32
#define INFO_MAX_LEN 1024
#define DESC_MAX_LEN 64
#define AUTO_MODE_MAX_NUM 16
#define PRIVATE_META_MAX_LEN 32

// 图像格式
typedef enum {
    FORMAT_YVU420, FORMAT_JPEG, FORMAT_AVC, FORMAT_HEVC,
    FORMAT_RGB_BAYER_12BPP, FORMAT_PRIVATE
} ImageFormat;

// 流类型
typedef enum {
    STREAM_PREVIEW, STREAM_VIDEO, STREAM_CAPTURE, STREAM_CALLBACK
} StreamType;

// 3A 模式
typedef enum {
    AE_MODE_ON, AE_MODE_OFF,
    AF_MODE_AUTO, AF_MODE_OFF,
    AWB_MODE_AUTO, AWB_MODE_OFF
} CameraAEMode, CameraAFMode, CameraAWBMode;

// 缓冲区定义
typedef struct {
    ImageFormat format;
    int32_t width, height;
    uint16_t fps;
    RectInfo crop;
    uint8_t invertMode;
} StreamAttr;

// 相机元数据
typedef struct {
    CameraAEMode aeMode;
    CameraAFMode afMode;
    CameraAWBMode awbMode;
    uint32_t privateData[PRIVATE_META_MAX_LEN];
} CameraMetaResult;

// 回调函数类型
typedef void (*BufferAvailable)(uint32_t streamId, HalBuffer *halBuffer, uint32_t bufferNum);
typedef void (*CameraDetectCb)(uint32_t cameraId, CameraStatus status);
typedef void (*CameraResultCb)(uint32_t cameraId, CameraMetaResult result);

// 核心接口
int32_t HalCameraInit(void);
int32_t HalCameraDeinit(void);
int32_t HalCameraGetDeviceNum(uint8_t *num);
int32_t HalCameraGetDeviceList(uint32_t *cameraList, uint8_t listNum);
int32_t HalCameraDeviceOpen(uint32_t cameraId);
int32_t HalCameraDeviceClose(uint32_t cameraId);
int32_t HalCameraStreamCreate(uint32_t cameraId, const StreamAttr *stream, uint32_t *streamId);
int32_t HalCameraStreamDestroy(uint32_t cameraId, uint32_t streamId);
int32_t HalCameraStreamOn(uint32_t cameraId, uint32_t streamId);
int32_t HalCameraStreamOff(uint32_t cameraId, uint32_t streamId);
int32_t HalCameraDequeueBuf(uint32_t cameraId, uint32_t streamId, HalBuffer *buffer);
int32_t HalCameraQueueBuf(uint32_t cameraId, uint32_t streamId, const HalBuffer *buffer);
```

**证据来源**: `hals/hal_camera.h:27-237`

### hal_display.h

```c
// 视频输出句柄
typedef int32_t HalVideoOutputHdl;

// 视频输出属性
typedef struct {
    int32_t regionPositionX;
    int32_t regionPositionY;
    int32_t regionWidth;
    int32_t regionHeight;
    uint32_t priority;
} HalVideoOutputAttr;

// 创建/销毁
int32_t HalCreateVideoOutput(HalVideoOutputHdl *handle, HalVideoOutputAttr attr);
int32_t HalDestroyVideoOutput(HalVideoOutputHdl handle);

// 配置
int32_t HalConfigVideoOutput(HalVideoOutputHdl handle, HalVideoOutputAttr attr);
int32_t HalGetVideoOutputConfig(HalVideoOutputHdl handle, HalVideoOutputAttr *attr);

// 启动/停止
int32_t HalStartVideoOutput(HalVideoOutputHdl handle);
int32_t HalStopVideoOutput(HalVideoOutputHdl handle);

// 数据写入
int32_t HalWriteVo(HalVideoOutputHdl handle, const void *buffer);

// 相机视频输出
int32_t HalCreateCameraVideoOutput(uint32_t deviceId, HalVideoOutputAttr *attr);
int32_t HalDestroyCameraVideoOutput();

// 图层管理
int32_t VoLayerInit(uint32_t devId);
void VoLayerDeInit(uint32_t devId);
```

**证据来源**: `hals/hal_display.h:27-65`
