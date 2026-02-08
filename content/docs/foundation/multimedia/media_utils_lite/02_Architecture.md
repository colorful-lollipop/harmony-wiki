# 架构设计

## 整体架构

`media_utils_lite` 采用分层架构设计，从上到下分为四层：

```
┌──────────────────────────────────────────────────────────────────┐
│                      应用层 (Applications)                         │
│         camera_lite, audio_lite, media_lite 等子系统              │
├──────────────────────────────────────────────────────────────────┤
│                     公共类型层 (Common Types)                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │  Source  │ │ Format   │ │   Error  │ │  Media   │            │
│  │          │ │          │ │   Codes  │ │   Info   │            │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘            │
│  interfaces/kits/  (对外 API)                                     │
├──────────────────────────────────────────────────────────────────┤
│                     数据流层 (Data Stream)                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────────┐        │
│  │DataBuffer│ │DataStream│ │    StreamSource          │        │
│  └──────────┘ └──────────┘ └──────────────────────────┘        │
├──────────────────────────────────────────────────────────────────┤
│                     HAL 抽象层 (Hardware Abstraction)              │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────────┐        │
│  │hal_media │ │hal_camera│ │     hal_display          │        │
│  │(视频处理) │ │(相机)    │ │     (显示输出)            │        │
│  └──────────┘ └──────────┘ └──────────────────────────┘        │
├──────────────────────────────────────────────────────────────────┤
│                    硬件/驱动层 (Hardware/Drivers)                  │
│              需由芯片厂商/板级适配层实现 HAL 接口                    │
└──────────────────────────────────────────────────────────────────┘
```

## HAL 抽象层详解

### 1. 视频处理器 HAL (hal_media.h)

负责视频编解码处理器的抽象。

```c
// 核心类型
typedef int32_t HalProcessorHdl;     // 处理器句柄
#define HAL_INVALID_PROCESSOR (-1)   // 无效句柄

// 视频处理器属性
typedef struct {
    uint32_t width;    // 宽度
    uint32_t height;   // 高度
    uint32_t fps;      // 帧率
} HalVideoProcessorAttr;

// 主要函数
HalProcessorHdl HalCreateVideoProcessor(HalVideoProcessorAttr *attr);
void HalDestroyVideoProcessor();
uint32_t HalGetProcessorDeviceId(HalProcessorHdl hdl);

// 初始化函数
int32_t HalMediaInitialize();
int32_t HalCameraInitialize();
void HalCameraUnInitialize();
```

**证据来源**: `hals/hal_media.h:30-48`

### 2. 相机 HAL (hal_camera.h)

负责相机设备能力的抽象。

```c
// 核心枚举
typedef enum {
    FORMAT_YVU420, FORMAT_JPEG, FORMAT_AVC, FORMAT_HEVC,
    FORMAT_RGB_BAYER_12BPP, FORMAT_PRIVATE
} ImageFormat;

typedef enum {
    STREAM_PREVIEW, STREAM_VIDEO, STREAM_CAPTURE, STREAM_CALLBACK
} StreamType;

// 缓冲区定义
typedef struct {
    ImageFormat format;
    int32_t width, height;
    uint16_t fps;
    RectInfo crop;
    uint8_t invertMode;
} StreamAttr;

// 相机元数据结果
typedef struct {
    CameraAEMode aeMode;
    CameraAFMode afMode;
    CameraAWBMode awbMode;
    uint32_t privateData[PRIVATE_META_MAX_LEN];
} CameraMetaResult;
```

**证据来源**: `hals/hal_camera.h:54-209`

### 3. 显示输出 HAL (hal_display.h)

负责视频输出到显示设备的抽象。

```c
// 视频输出句柄
typedef int32_t HalVideoOutputHdl;

// 视频输出属性
typedef struct {
    int32_t regionPositionX, regionPositionY;
    int32_t regionWidth, regionHeight;
    uint32_t priority;
} HalVideoOutputAttr;

// 核心操作
int32_t HalCreateVideoOutput(HalVideoOutputHdl *handle, HalVideoOutputAttr attr);
int32_t HalDestroyVideoOutput(HalVideoOutputHdl handle);
int32_t HalStartVideoOutput(HalVideoOutputHdl handle);
int32_t HalStopVideoOutput(HalVideoOutputHdl handle);
int32_t HalWriteVo(HalVideoOutputHdl handle, const void *buffer);
```

**证据来源**: `hals/hal_display.h:27-57`

## 数据流设计

### 数据流架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        DataProducer                               │
│              (数据生产者，如解码器、文件读取器)                     │
│     bool GetEmptyBuffer(std::shared_ptr<DataBuffer>& buffer)     │
│     bool QueueDataBuffer(const std::shared_ptr<DataBuffer>&)     │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DataStream                               │
│               (数据流，同时实现生产和消费接口)                      │
│    同时继承 DataProducer 和 DataConsumer                          │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DataConsumer                               │
│              (数据消费者，如渲染器、编码器)                        │
│     bool GetDataBuffer(std::shared_ptr<DataBuffer>& buffer)      │
│     bool QueueEmptyBuffer(const std::shared_ptr<DataBuffer>&)    │
└─────────────────────────────────────────────────────────────────┘
```

### 数据缓冲区 (DataBuffer)

```c
// 内存类型
enum class MemoryType {
    VIRTUAL_ADDR = 0,   // 虚拟地址
    SURFACE_BUFFER,     // Surface 缓冲区
    SHARE_MEMORY,       // 共享内存 fd
};

// DataBuffer 接口
class DataBuffer {
    virtual bool IsEos();           // 是否结束
    virtual void SetEos(bool);      // 设置结束标记
    virtual uint8_t* GetAddress();  // 获取地址
    virtual size_t GetCapacity();   // 获取容量
    virtual size_t GetSize();       // 获取有效数据大小
    virtual void SetSize(size_t);   // 设置有效数据大小
};
```

**证据来源**: `interfaces/kits/data_stream.h:29-94`

## Source/StreamSource 设计

### Source 类型

```cpp
// 媒体源类型
enum class SourceType : int32_t {
    SOURCE_TYPE_URI = 0,      // URI (本地文件或网络地址)
    SOURCE_TYPE_FD,           // 文件描述符
    SOURCE_TYPE_STREAM,       // 流数据
};

// Source 类
class Source {
    explicit Source(const std::string& uri);
    Source(const std::string &uri, const std::map<std::string, std::string> &header);
    Source(const std::shared_ptr<StreamSource> &stream, const Format &formats);
    Source(const std::shared_ptr<DataConsumer> &dataConsumer);
    
    SourceType GetSourceType() const;
    const std::string &GetSourceUri() const;
    const std::shared_ptr<StreamSource> &GetSourceStream() const;
    const Format &GetSourceStreamFormat() const;
};
```

### StreamSource 回调

```cpp
// 流数据回调
struct StreamCallback {
    enum BufferFlags {
        STREAM_FLAG_SYNCFRAME = 1,       // 同步帧
        STREAM_FLAG_CODECCONFIG = 2,     // 编解码配置
        STREAM_FLAG_EOS = 4,             // 流结束
        STREAM_FLAG_PARTIAL_FRAME = 8,   // 部分帧
        STREAM_FLAG_ENDOFFRAME = 16,     // 帧结束
    };
    
    virtual uint8_t* GetBuffer(size_t index) = 0;
    virtual void QueueBuffer(size_t index, size_t offset, size_t size, 
                            int64_t timestampUs, uint32_t flags) = 0;
    virtual void SetParameters(const Format &params) = 0;
};

class StreamSource {
    virtual void OnBufferAvailable(size_t index, size_t offset, size_t size);
    virtual void SetStreamCallback(const std::shared_ptr<StreamCallback> &callback);
};
```

**证据来源**: `interfaces/kits/source.h:46-180`

## Format 数据结构

### FormatData 类型

```cpp
// 格式数据类型
enum FormatDataType : uint32_t {
    FORMAT_TYPE_NONE = 0,
    FORMAT_TYPE_INT32,
    FORMAT_TYPE_INT64,
    FORMAT_TYPE_FLOAT,
    FORMAT_TYPE_DOUBLE,
    FORMAT_TYPE_STRING
};

// FormatData 使用 union 存储不同类型
class FormatData {
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

### Format 类

```cpp
// Format 是键值对形式的元数据容器
class Format {
    bool PutIntValue(const std::string &key, int32_t value);
    bool PutLongValue(const std::string &key, int64_t value);
    bool PutFloatValue(const std::string &key, float value);
    bool PutDoubleValue(const std::string &key, double value);
    bool PutStringValue(const std::string &key, const std::string &value);
    
    bool GetIntValue(const std::string &key, int32_t &value) const;
    // ... 其他 Get 方法
    
    bool CopyFrom(const Format &format);
    const std::map<std::string, FormatData *> &GetFormatMap() const;
};
```

**证据来源**: `interfaces/kits/format.h:63-365`

## 线程模型

### 线程安全说明

根据代码分析，本组件的数据结构设计遵循以下原则：

1. **非线程安全**: Format/Source 等数据结构未提供显式线程同步
2. **调用方负责同步**: 使用者需确保在多线程环境下正确加锁
3. **HAL 接口**: HAL 函数未规定线程模型，由具体实现决定

### 建议使用模式

```
单线程模式:
┌──────────────────────────────────────────────┐
│  创建 Source → 设置参数 → 提交到媒体框架        │
│  (同一线程内完成所有操作)                       │
└──────────────────────────────────────────────┘

多线程模式:
┌──────────────────────────────────────────────┐
│  Thread 1: 创建 Source                          │
│  Thread 1: 加锁 → 修改 → 解锁                   │
│  Thread 2: 加锁 → 读取 → 解锁                   │
└──────────────────────────────────────────────┘
```

## 条件编译

### SURFACE_DISABLED 宏

用于在无 Surface 环境下编译：

```cpp
#ifndef SURFACE_DISABLED
#include "surface.h"
#endif

class StreamSource {
#ifndef SURFACE_DISABLED
    void SetSurface(Surface* surface);
    Surface* GetSurface();
    Surface* surface_;
    SurfaceBuffer* curBuffer_;
#endif
};
```

**证据来源**: `interfaces/kits/source.h:44-46`, `hals/hal_camera.h` 多处使用

### 平台适配

```gn
# BUILD.gn 中的平台适配
if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"
    public_deps += [ "...:hilog_static" ]
} else {
    target_type = "shared_library"
    public_deps += [ "...:surface_lite" ]
}
```

**证据来源**: `BUILD.gn:17-48`
