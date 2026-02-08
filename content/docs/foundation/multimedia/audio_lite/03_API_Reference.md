# 03_API_Reference - API 参考

本文档描述 audio_lite 的对外 C++ API。注意：**本项目不包含 N-API/JS 接口**，是纯 C++ framework。

## 3.1 命名空间

所有公开 API 位于 `OHOS::Audio` 命名空间。

**证据**：`interfaces/kits/audio_capturer.h:48-49`

```cpp
namespace OHOS {
namespace Audio {
```

## 3.2 AudioCapturer 类

### 3.2.1 类概述

| 属性 | 值 |
|------|-----|
| 头文件 | `interfaces/kits/audio_capturer.h` |
| 命名空间 | `OHOS::Audio` |
| 线程安全 | 否（调用方负责同步） |
| 生命周期 | 用户创建，单实例 |

### 3.2.2 公开方法清单

| 方法 | 功能 | 同步/异步 | 返回类型 |
|------|------|----------|----------|
| `AudioCapturer()` | 构造函数 | 同步 | - |
| `~AudioCapturer()` | 析构函数 | 同步 | - |
| `GetMinFrameCount()` | 获取最小帧数 | 静态 | `bool` |
| `SetCapturerInfo()` | 设置采集参数 | 同步 | `int32_t` |
| `GetCapturerInfo()` | 获取采集参数 | 同步 | `int32_t` |
| `Start()` | 开始采集 | 同步 | `bool` |
| `Read()` | 读取音频数据 | 同步/阻塞可选 | `int32_t` |
| `GetStatus()` | 获取状态 | 同步 | `State` |
| `GetAudioTime()` | 获取时间戳 | 同步 | `bool` |
| `Stop()` | 停止采集 | 同步 | `bool` |
| `Release()` | 释放资源 | 同步 | `bool` |

### 3.2.3 方法详细说明

#### AudioCapturer()

**声明**：`interfaces/kits/audio_capturer.h:130`

```cpp
AudioCapturer();
```

**功能**：创建 AudioCapturer 实例。

**实现细节**：
- 创建内部 `AudioCapturerClient` 实例
- 连接到 AudioCapturerServer（Binder 模式）或直接持有 AudioCapturerImpl（Passthrough 模式）

**证据**：`frameworks/audio_capturer.cpp:30-33`

```cpp
AudioCapturer::AudioCapturer()
    : impl_(new(std::nothrow) AudioCapturerClient())
{
}
```

**异常**：
- 若连接失败，构造函数抛出 `std::runtime_error`

---

#### ~AudioCapturer()

**声明**：`interfaces/kits/audio_capturer.h:131`

```cpp
virtual ~AudioCapturer();
```

**功能**：析构对象。建议先调用 `Release()` 释放资源。

---

#### GetMinFrameCount()

**声明**：`interfaces/kits/audio_capturer.h:145-146`

```cpp
static bool GetMinFrameCount(int32_t sampleRate, int32_t channelCount, 
                            AudioCodecFormat audioFormat, size_t &frameCount);
```

**功能**：获取指定采样率、声道数和音频格式下的最小帧数。

**参数**：
| 参数 | 类型 | 说明 | 范围 |
|------|------|------|------|
| `sampleRate` | `int32_t` | 采样率 (Hz) | 正整数 |
| `channelCount` | `int32_t` | 声道数 | 1 或 2 |
| `audioFormat` | `AudioCodecFormat` | 音频格式 | 见枚举 |
| `frameCount` | `size_t&` | 输出：最小帧数 | 输出参数 |

**返回值**：
| 值 | 说明 |
|------|------|
| `true` | 成功 |
| `false` | 失败（参数无效） |

**调用链**：
```
AudioCapturer::GetMinFrameCount()
  └─► AudioCapturerClient::GetMinFrameCount()
        └─► IPC 调用 / 直接调用
```

---

#### SetCapturerInfo()

**声明**：`interfaces/kits/audio_capturer.h:168`

```cpp
int32_t SetCapturerInfo(const AudioCapturerInfo info);
```

**功能**：设置音频采集参数。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `info` | `const AudioCapturerInfo` | 采集参数结构体 |

**返回值**：
| 值 | 说明 |
|------|------|
| `SUCCESS` | 成功 |
| 错误码 | 见 `media_errors.h` |

**前置条件**：
- 状态为 `INITIALIZED`

**后置条件**：
- 状态变为 `PREPARED`（若成功）

**证据**：`services/impl/audio_capturer_impl.cpp`（实现位置）

---

#### GetCapturerInfo()

**声明**：`interfaces/kits/audio_capturer.h:181`

```cpp
int32_t GetCapturerInfo(AudioCapturerInfo &info);
```

**功能**：获取当前采集参数。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `info` | `AudioCapturerInfo&` | 输出：采集参数 |

**返回值**：
| 值 | 说明 |
|------|------|
| `SUCCESS` | 成功 |
| 错误码 | 失败 |

**前置条件**：
- 已调用 `SetCapturerInfo()` 成功

---

#### Start()

**声明**：`interfaces/kits/audio_capturer.h:190`

```cpp
bool Start();
```

**功能**：开始音频采集。

**返回值**：
| 值 | 说明 |
|------|------|
| `true` | 成功开始采集 |
| `false` | 失败 |

**前置条件**：
- 状态为 `PREPARED`

**后置条件**：
- 状态变为 `RECORDING`

**调用链**：
```
Start()
  └─► StartEncoder()
        └─► StartSource()
              └─► CreateDataThread()
```

---

#### Read()

**声明**：`interfaces/kits/audio_capturer.h:208`

```cpp
int32_t Read(uint8_t *buffer, size_t userSize, bool isBlockingRead);
```

**功能**：读取音频数据。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `buffer` | `uint8_t*` | 目标缓冲区指针 |
| `userSize` | `size_t` | 缓冲区大小（字节） |
| `isBlockingRead` | `bool` | 是否阻塞读取 |

**返回值**：
| 值 | 说明 |
|------|------|
| 正整数 | 读取到的数据大小（字节） |
| `0` | 无数据（非阻塞模式） |
| `ERR_INVALID_READ` | 参数错误 |
| `ERR_ILLEGAL_STATE` | 状态错误 |
| `ERR_SOURCE_NOT_SET` | 硬件异常 |

**前置条件**：
- 状态为 `RECORDING`

**缓冲区大小要求**：
```
userSize >= frameCount * channelCount * BytesPerSample
```

**证据**：`frameworks/binder/audio_capturer_client.cpp:379-413`

```cpp
int32_t AudioCapturer::AudioCapturerClient::Read(uint8_t *buffer, size_t userSize, bool isBlockingRead)
{
    // 从 Surface 获取数据
    SurfaceBuffer *surfaceBuf = surface_->AcquireBuffer();
    
    // 复制数据到用户 buffer
    (void)memcpy_s(buffer, userSize, buf + sizeof(Timestamp), dataSize - sizeof(Timestamp));
    (void)memcpy_s(&curTimestamp_, sizeof(Timestamp), buf, sizeof(Timestamp));
    
    surface_->ReleaseBuffer(surfaceBuf);
    return readLen;
}
```

---

#### GetStatus()

**声明**：`interfaces/kits/audio_capturer.h:217`

```cpp
State GetStatus();
```

**功能**：获取当前采集状态。

**返回值**：`State` 枚举值

---

#### GetAudioTime()

**声明**：`interfaces/kits/audio_capturer.h:229`

```cpp
bool GetAudioTime(Timestamp &timestamp, Timestamp::Timebase base);
```

**功能**：获取最近读取帧的时间戳。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `timestamp` | `Timestamp&` | 输出：时间戳 |
| `base` | `Timestamp::Timebase` | 时间基准（MONOTONIC/BOOTTIME） |

**返回值**：
| 值 | 说明 |
|------|------|
| `true` | 成功 |
| `false` | 失败（无可用时间戳） |

---

#### Stop()

**声明**：`interfaces/kits/audio_capturer.h:238`

```cpp
bool Stop();
```

**功能**：停止音频采集。

**返回值**：
| 值 | 说明 |
|------|------|
| `true` | 成功 |
| `false` | 失败 |

**前置条件**：
- 状态为 `RECORDING`

**后置条件**：
- 状态变为 `STOPPED`

---

#### Release()

**声明**：`interfaces/kits/audio_capturer.h:247`

```cpp
bool Release();
```

**功能**：释放资源，销毁内部对象。

**返回值**：
| 值 | 说明 |
|------|------|
| `true` | 成功 |
| `false` | 失败 |

**前置条件**：
- 无（可从任何状态调用）

**后置条件**：
- 状态变为 `RELEASED`
- 资源被释放，不可再使用

**资源释放顺序**：
```
Release()
  └─► Stop()（若正在录制）
        └─► ReleaseEncoder()
              └─► ReleaseSource()
                    └─► UnloadAdapter()
```

## 3.3 数据结构

### 3.3.1 AudioCapturerInfo

**声明**：`interfaces/kits/audio_capturer.h:57-72`

```cpp
struct AudioCapturerInfo {
    AudioSourceType inputSource;    // 音频源类型，默认 AUDIO_MIC
    AudioCodecFormat audioFormat;   // 音频编码格式，默认 AUDIO_DEFAULT
    int32_t sampleRate;             // 采样率
    int32_t channelCount;           // 声道数
    int32_t bitRate;                // 码率
    AudioStreamType streamType;     // 音频流类型
    AudioBitWidth bitWidth;         // 位宽，默认 BIT_WIDTH_16
};
```

### 3.3.2 Timestamp

**声明**：`interfaces/kits/audio_capturer.h:80-100`

```cpp
class Timestamp {
public:
    uint32_t framePosition;         // 帧位置
    struct timespec time;           // 时间戳

    enum class Timebase : int32_t {
        MONOTONIC = 0,              // 单调时间（不含休眠）
        BOOTTIME = 1                // 启动时间（含休眠）
    };
};
```

### 3.3.3 State 枚举

**声明**：`interfaces/kits/audio_capturer.h:109-120`

```cpp
enum State : uint32_t {
    INITIALIZED = 0,   // 已初始化
    PREPARED = 1,       // 已准备
    RECORDING = 2,     // 采集中
    STOPPED = 3,        // 已停止
    RELEASED = 4       // 已释放
};
```

## 3.4 内部 API（仅供参考）

### 3.4.1 AudioCapturerClient

| 类 | 头文件 | 说明 |
|----|--------|------|
| `AudioCapturer::AudioCapturerClient` | `frameworks/binder/audio_capturer_client.h` | Binder 模式客户端 |
| `AudioCapturer::AudioCapturerClient` | `frameworks/passthrough/audio_capturer_client.h` | Passthrough 模式客户端 |

### 3.4.2 AudioCapturerServer

| 类 | 头文件 | 说明 |
|----|--------|------|
| `AudioCapturerServer` | `services/server/include/audio_capturer_server.h` | 服务端单例 |

### 3.4.3 AudioCapturerImpl

| 类 | 头文件 | 说明 |
|----|--------|------|
| `AudioCapturerImpl` | `services/impl/audio_capturer_impl.h` | 核心实现类 |

## 3.5 错误码

| 错误码 | 定义位置 | 说明 |
|--------|----------|------|
| `SUCCESS` | `media_errors.h` | 成功 |
| `ERR_INVALID_PARAM` | `media_errors.h` | 参数无效 |
| `ERR_ILLEGAL_STATE` | `media_errors.h` | 状态错误 |
| `ERR_SOURCE_NOT_SET` | `media_errors.h` | 音频源异常 |
| `ERR_INVALID_READ` | `audio_capturer_client.h` | 读取参数错误 |

---

**上一章**：[02_Architecture](02_Architecture.md) | **下一章**：[04_Build_System](04_Build_System.md)
