# 02_Architecture - 架构说明

本文档描述 audio_lite 的整体架构设计，包括组件图、数据流、线程模型和关键调用链。

## 2.1 整体架构图

### 2.1.1 分层架构

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              应用层 (Applications)                              │
└─────────────────────────────────┬───────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           interfaces/kits (对外 API 层)                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                      AudioCapturer (C++ 接口)                           │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────┬───────────────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
┌─────────────────────────────────┐  ┌───────────────────────────────────────────────┐
│   frameworks/ (框架层)          │  │           services/ (服务层)                 │
│                                 │  │                                               │
│  ┌─────────────────────────┐    │  │  ┌─────────────────┐  ┌───────────────────┐  │
│  │ AudioCapturerClient    │    │  │  │     server/     │  │      impl/        │  │
│  │ ├─ Binder IPC 模式     │    │  │  │                 │  │                   │  │
│  │ └─ Passthrough 模式    │    │  │  │ AudioCapturer   │  │ AudioCapturerImpl│  │
│  └─────────────────────────┘    │  │  │ Server          │  │                   │  │
│                                 │  │  │ ├─ Samgr 注册    │  │ ├─ AudioSource    │  │
│                                 │  │  │ └─ IPC 分发      │  │ └─ AudioEncoder   │  │
└─────────────────────────────────┘  │  └─────────────────┘  └───────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             系统层 (System Layer)                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────────────────┐  │
│  │   Samgr      │  │  IPC/Surface │  │   PMS       │  │     HAL 层             │  │
│  │  (SA 管理)   │  │  (进程通信)   │  │  (权限管理)  │  │ ├─ Audio HW            │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │ └─ Codec              │  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.1.2 组件关系图

```
                    ┌─────────────────────────┐
                    │    AudioCapturer        │
                    │   (对外 C++ 接口)        │
                    └───────────┬─────────────┘
                                │
                                │ std::unique_ptr<AudioCapturerClient>
                                ▼
            ┌───────────────────┴───────────────────┐
            │                                       │
            ▼                                       ▼
┌─────────────────────────┐           ┌─────────────────────────┐
│  BinderAudioCapturer   │           │  PassthroughAudio      │
│  Client (IPC 客户端)    │           │  CapturerClient         │
│                         │           │  (本地直连)             │
│  - IClientProxy *proxy │           │                         │
│  - Surface            │           │  - AudioCapturerImpl*  │
│  - ProxyCallbackFunc   │           │  (直接调用)             │
└───────────┬─────────────┘           └───────────┬─────────────┘
            │                                   │
            │ Invoke()                          │ 直接调用
            │ IpcIo                             │
            ▼                                   ▼
┌─────────────────────────┐           ┌─────────────────────────┐
│  AudioCapturerServer   │           │  AudioCapturerImpl      │
│  (IPC 服务端)           │           │  (核心实现)              │
│                         │           │                         │
│  - clientPid_          │           │  - audioSource_         │
│  - AudioCapturerImpl*  │           │  - audioEncoder_        │
│  - dataThread_         │           │  - status_              │
└───────────┬─────────────┘           └───────────┬─────────────┘
            │                                   │
            │ 创建/管理                         │ 持有/协调
            ▼                                   ▼
┌─────────────────────────┐           ┌─────────────────────────┐
│  AudioCapturerImpl      │──────────►│  AudioSource            │
│  (共享实例)              │   调用     │  (HAL 采集)             │
└─────────────────────────┘           └───────────┬─────────────┘
                                                  │
                                                  │ 调用 HAL
                                                  ▼
                                        ┌─────────────────────────┐
                                        │  Audio HW / Codec       │
                                        │  (硬件抽象层)             │
                                        └─────────────────────────┘
```

## 2.2 数据流

### 2.2.1 音频数据流（Binder IPC 模式）

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              音频数据流向                                     │
└─────────────────────────────────────────────────────────────────────────────┘

  AudioSource         AudioCapturerImpl      AudioCapturerServer      AudioCapturerClient
     │                      │                        │                        │
     │  HAL 原始 PCM         │                        │                        │
     │─────────────────────►│  ReadFromHAL()          │                        │
     │                      │                        │                        │
     │                      │  编码 (AudioEncoder)   │                        │
     │                      │───────────────────────►│                        │
     │                      │                        │                        │
     │                      │                        │  写入 SurfaceBuffer     │
     │                      │                        │  (含时间戳)              │
     │                      │                        │────────────────────────►│
     │                      │                        │                        │
     │                      │                        │                        │  AcquireBuffer()
     │                      │                        │                        │◄────────────────
     │                      │                        │                        │
     │                      │                        │                        │  Read()
     │                      │                        │                        │◄────────────────
     │                      │                        │                        │
     ▼                      ▼                        ▼                        ▼

  [HAL 驱动]        [AudioEncoder]        [Surface 共享内存]        [应用缓冲区]
```

### 2.2.2 控制命令流（IPC 调用）

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              控制命令流向                                      │
└─────────────────────────────────────────────────────────────────────────────┘

  AudioCapturer                    AudioCapturerClient              AudioCapturerServer
     │                                    │                                │
     │  API 调用                          │                                │
     │───────────────────────────────────►│  IpcIo 序列化                   │
     │                                    │───────────────────────────────►│
     │                                    │                                │
     │                                    │        Invoke(funcId, req)     │
     │                                    │◄───────────────────────────────│
     │                                    │                                │
     │  返回结果                          │  ProxyCallbackFunc()           │
     │◄───────────────────────────────────│                                │
     │                                    │                                │
     ▼                                    ▼                                ▼

  [应用线程]                    [IPC 回调线程]                  [Samgr 服务线程]
```

**证据**：`frameworks/binder/audio_capturer_client.cpp:52-99`

```cpp
static int32_t ProxyCallbackFunc(void *owner, int code, IpcIo *reply)
{
    // IPC 调用完成后的回调处理
    // 根据 funcId 解析返回数据
    CallBackPara* para = static_cast<CallBackPara*>(owner);
    AudioCapturerFuncId funcId = (AudioCapturerFuncId)para->funcId;
    
    switch (funcId) {
        case AUD_CAP_FUNC_START:
            // 处理 Start 返回
            break;
        case AUD_CAP_FUNC_READ:
            // 处理 Read 返回
            break;
        // ... 其他 funcId
    }
}
```

## 2.3 线程模型

### 2.3.1 线程划分

| 线程 | 所属组件 | 职责 | 优先级 |
|------|----------|------|--------|
| **应用线程** | 调用方 | 创建 AudioCapturer、调用 API | 取决于应用 |
| **Samgr 服务线程** | AudioCapturerServer | IPC 请求分发、处理连接 | LEVEL_HIGH |
| **数据读取线程** | AudioCapturerServer | 从 HAL 读取音频数据、写入 Surface | PRI_BELOW_NORMAL |

### 2.3.2 服务线程配置

**证据**：`services/server/src/audio_capturer_samgr.cpp:69-74`

```cpp
static TaskConfig GetTaskConfig(Service *service)
{
    (void)service;
    TaskConfig config = {
        LEVEL_HIGH,        // 优先级级别：高
        PRI_BELOW_NORMAL,  // 调度优先级：低于正常
        0x800,             // 栈大小：2KB
        20,                // 消息队列大小：20
        SHARED_TASK        // 共享任务模式
    };
    return config;
}
```

### 2.3.3 数据读取线程

**证据**：`services/server/src/audio_capturer_server.cpp`

```cpp
// 创建数据读取线程
pthread_create(&dataThreadId_, nullptr, ReadAudioDataProcess, this);

// 线程函数
void *AudioCapturerServer::ReadAudioDataProcess(void *arg)
{
    AudioCapturerServer *server = reinterpret_cast<AudioCapturerServer*>(arg);
    
    while (!server->threadExit_) {
        // 从 AudioCapturerImpl 读取数据
        // 写入 SurfaceBuffer
        // 5ms 轮询间隔
        usleep(5000);
    }
    return nullptr;
}

// 停止线程
void AudioCapturerServer::StopThread()
{
    threadExit_ = true;
    pthread_join(dataThreadId_, nullptr);
}
```

### 2.3.4 线程同步机制

| 类 | 同步原语 | 保护对象 |
|----|----------|----------|
| `AudioCapturerImpl` | `std::mutex mutex_` | status_、info_、timestamp_ |
| `AudioCapturerServer` | `std::mutex lock_` | clientPid_、capturer_ |
| `AudioCapturerClient` | `std::mutex lock_` | surface_、curTimestamp_ |

**证据**：`services/impl/audio_capturer_impl.h:19,59`

```cpp
class AudioCapturerImpl {
private:
    std::mutex mutex_;           // 保护成员变量
    State status_ = INITIALIZED;
    AudioCapturerInfo info_;
    Timestamp timestamp_;
};
```

## 2.4 关键调用链

### 2.4.1 AudioCapturer::Read() 调用链（Binder 模式）

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AudioCapturer::Read() 调用链                         │
└─────────────────────────────────────────────────────────────────────────────┘

  应用调用
      │
      ▼
┌───────────────────┐
│ AudioCapturer::   │ interfaces/kits/audio_capturer.h:208
│ Read()            │
└─────────┬─────────┘
          │
          ▼ (委托给 impl_)
┌───────────────────┐
│ AudioCapturer::   │ frameworks/audio_capturer.cpp:93-97
│ AudioCapturer     │
│ Client::Read()    │
└─────────┬─────────┘
          │
          ▼ (IPC 调用)
┌───────────────────┐
│ IClientProxy::    │ IPC 框架
│ Invoke()          │
└─────────┬─────────┘
          │
          ▼ (IPC 传输)
┌───────────────────┐
│ AudioCapturer     │ services/server/src/audio_capturer_server.cpp
│ Server::Dispatch()│
└─────────┬─────────┘
          │
          ▼ (分发)
┌───────────────────┐
│ AudioCapturerImpl │ services/impl/audio_capturer_impl.cpp
│ ::Read()          │
└─────────┬─────────┘
          │
          ▼ (共享内存)
┌───────────────────┐
│ Surface::         │ surface_lite
│ AcquireBuffer()   │
└─────────┬─────────┘
          │
          ▼ (返回数据)
  应用接收 buffer
```

### 2.4.2 Start() 时序图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Start() 时序图                                    │
└─────────────────────────────────────────────────────────────────────────────┘

participant 应用 as App
participant AudioCapturer
participant AudioCapturerClient
participant AudioCapturerServer
participant AudioCapturerImpl

App->>AudioCapturer: Start()
AudioCapturer->>AudioCapturerClient: Start()
AudioCapturerClient->>AudioCapturerClient: InitSurface()
AudioCapturerClient->>AudioCapturerClient: SetQueueSize(5)
AudioCapturerClient->>AudioCapturerClient: SetSize(8192)

AudioCapturerClient->>AudioCapturerServer: Invoke(AUD_CAP_FUNC_START)
AudioCapturerServer->>AudioCapturerServer: Dispatch(AUD_CAP_FUNC_START, pid)
AudioCapturerServer->>AudioCapturerImpl: Start()

AudioCapturerImpl->>AudioCapturerImpl: StartEncoder()
AudioCapturerImpl->>AudioCapturerImpl: StartSource()
AudioCapturerImpl->>AudioCapturerImpl: CreateDataThread()

AudioCapturerImpl-->>AudioCapturerServer: SUCCESS
AudioCapturerServer-->>AudioCapturerClient: SUCCESS
AudioCapturerClient-->>AudioCapturer: true
AudioCapturer-->>App: true
```

### 2.4.3 服务注册时序

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         服务注册时序图                                      │
└─────────────────────────────────────────────────────────────────────────────┘

  系统启动
      │
      ▼
┌───────────────────┐
│ SAMGR_Bootstrap() │ frameworks/binder/audio_capturer_client.cpp:28
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ AudioCapturer     │ 构造函数
│ Client::ctor()    │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ SAMGR_GetInstance │ 获取 Samgr 单例
│ ->GetDefault      │
│ FeatureApi()      │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ AudioCapturer     │ services/server/src/audio_capturer_samgr.cpp:94-116
│ ServiceReg()      │
│                   │
│ RegisterService() │
│ RegisterDefault   │
│ FeatureApi()      │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ AudioCapturer     │ 就绪，可接收请求
│ Server            │
```

## 2.5 信任边界

### 2.5.1 边界定义

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界图                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           客户端进程边界                                      │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  AudioCapturer::AudioCapturerClient                                 │   │
│  │    - IClientProxy *proxy_ (可信 IPC 代理)                           │   │
│  │    - Surface (共享内存)                                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ══════════════════════════════════════════════════════════════════════    │
│                              IPC 边界                                        │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  AudioCapturerService (Samgr_lite 服务)                            │   │
│  │    - Invoke() 分发器                                                 │   │
│  │    - GetCallingPid() (从 IPC 框架获取可信 PID)                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              │ SAMgr IPC
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           服务端进程边界                                      │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  AudioCapturerServer                                                │   │
│  │    - clientPid_ (验证连接客户端)                                      │   │
│  │    - AcceptServer() / DropServer()                                   │   │
│  │    - AudioCapturerImpl* capturer_                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ══════════════════════════════════════════════════════════════════════    │
│                            实现边界                                           │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  AudioCapturerImpl                                                  │   │
│  │    - AudioSource (HAL 访问)                                          │   │
│  │    - AudioEncoder                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.5.2 安全检查点

| 边界 | 检查点 | 实现位置 |
|------|--------|----------|
| 连接建立 | `clientPid_ == -1`（单客户端限制） | `audio_capturer_server.cpp:40-47` |
| 请求分发 | `pid == clientPid_`（验证调用者身份） | `audio_capturer_server.cpp:99-103` |
| PID 获取 | `GetCallingPid()`（IPC 框架提供） | `audio_capturer_samgr.cpp:83` |

---

**上一章**：[01_Overview](01_Overview.md) | **下一章**：[03_API_Reference](03_API_Reference.md)
