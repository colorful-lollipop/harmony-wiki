# 03_架构设计

## 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OpenHarmony 应用层                                   │
│                    (JavaScript/ArkTS - 位于独立仓库)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                           N-API 绑定层                                        │
│                    (multimedia/media_library - 独立仓库)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                          C-API 对外接口层                                     │
│ ──────┐   │
│  ┌──────────────────────────────────────────────────────────────── │                    interfaces/kits/c/*.h                              │   │
│  │    OH_VideoDecoder_*, OH_VideoEncoder_*, OH_AVDemuxer_*, ...         │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────────┤
│                          框架层 (frameworks/native)                            │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│  │  C-API 实现     │  │  C-API 实现     │  │  C-API 实现     │               │
│  │  native_*.cpp  │  │  native_*.cpp  │  │  native_*.cpp  │               │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘               │
├──────────┼───────────────────┼───────────────────┼─────────────────────────┤
│          │                   │                   │                          │
│          └───────────────────┴───────────────────┘                          │
│                              │                                              │
│                    ┌─────────┴─────────┐                                    │
│                    │   IPC 客户端层      │                                    │
│                    │  (IRemoteProxy)   │                                    │
│                    └─────────┬─────────┘                                    │
├──────────────────────────────┼───────────────────────────────────────────────┤
│                               ▼                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     av_codec_service (SA 3011)                         │  │
│  │  ┌─────────────────────────┐  ┌─────────────────────────────────────┐  │  │
│  │  │    AVCodecServer        │  │    System Ability Registry         │  │  │
│  │  │    (OnRemoteRequest)    │  │    REGISTER_SYSTEM_ABILITY_BY_ID   │  │  │
│  │  └───────────┬─────────────┘  └──────────────────┬──────────────────┘  │  │
│  │              │                                  │                       │  │
│  │              ▼                                  │                       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                     IPC 分发层                                   │  │  │
│  │  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │  │  │
│  │  │  │ CodecService   │ │ CodecListService│ │  其他 Service   │      │  │  │
│  │  │  │  (30+ 方法)    │ │  (4 方法)        │ │                 │      │  │  │
│  │  │  └────────┬────────┘ └────────┬────────┘ └─────────────────┘      │  │  │
│  │  └───────────┼──────────────────┼──────────────────────────────────────┘  │  │
│  │              │                  │                                          │
│  └──────────────┼──────────────────┼──────────────────────────────────────────┘  │
│                 │                  │                                             │
│  ┌──────────────┼──────────────────┼──────────────────────────────────────────┐  │
│  │              ▼                  ▼                                            │  │
│  │  ┌─────────────────────────────────────────────────────────────────────┐ │  │
│  │  │                    引擎层 (services/engine)                          │ │  │
│  │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │ │  │
│  │  │  │ HCodec      │ │ SCodec      │ │  Demuxer    │ │  Muxer      │   │ │  │
│  │  │  │ (硬件加速)   │ │ (软件)      │ │  (解封装)   │ │  (封装)     │   │ │  │
│  │  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────┬──────┘   │ │  │
│  │  │         │                │                │                │          │ │  │
│  │  └─────────┼────────────────┼────────────────┼────────────────┼──────────┘ │  │
│  │            │                │                │                │             │  │
│  │  ┌─────────┴────────────────┴────────────────┴────────────────┴─────────┐ │  │
│  │  │                    插件层 (services/media_engine/plugins)                │ │  │
│  │  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐           │ │  │
│  │  │  │ FFmpeg Adapter  │ │ HTTP Source     │ │  Native Plugin  │           │ │  │
│  │  │  │ (音频编解码)     │ │ (网络资源)       │ │                 │           │ │  │
│  │  │  └─────────────────┘ └─────────────────┘ └─────────────────┘           │ │  │
│  │  └─────────────────────────────────────────────────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────────┐  │
│  │                    外部依赖 (IPC 调用)                                       │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │  │
│  │  │ SAMGR       │ │ DRM         │ │ Audio FW    │ │ Surface     │           │  │
│  │  │ (SA 注册)   │ │ (解密)       │ │ (音频)       │ │ (图形)       │           │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘           │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘──┘
```

## IPC 架构

### System Ability 注册

av_codec 服务以 **SA 形式运行，拥有独立的进程**。

| 属性 | 值 | 证据 |
|------|-----|------|
| SA ID | `3011` | `test/unittest/video_test/sa_avcodec_test/mock/include/system_ability_definition.h:20` |
| 注册宏 | `REGISTER_SYSTEM_ABILITY_BY_ID` | `services/services/sa_avcodec/server/avcodec_server.cpp:36` |
| 服务类 | `AVCodecServer` | `services/services/sa_avcodec/server/avcodec_server.cpp:26` |

### SA 生命周期

```cpp
// 文件: services/services/sa_avcodec/server/avcodec_server.cpp:52-87
void AVCodecServer::OnStart() {
    // 1. 发布服务
    Publish(this);
    // 2. 配置 IPC 线程数
    IPCSkeleton::SetMaxWorkThreadNum(64);
    // 3. 注册 SA 监听器
    AddSystemAbilityListener(MEMORY_MANAGER_SA_ID);
}

void AVCodecServer::OnStop() {
    // 服务停止时的清理
}
```

### IPC 接口编码

#### SA 级接口 (AVCodecServiceInterfaceCode)

| Code | 值 | 方法 | 用途 |
|------|-----|------|------|
| GET_SUBSYSTEM | 0 | `GetSubSystemAbility()` | 获取子 SA |
| FREEZE | 1 | `SuspendFreeze()` | 挂起服务 |
| ACTIVE | 2 | `SuspendActive()` | 恢复服务 |
| ACTIVEALL | 3 | `SuspendActiveAll()` | 恢复所有 |
| GET_ACTIVE_SECURE_DECODER_PIDS | 4 | `GetActiveSecureDecoderPids()` | 获取安全解码器 PID |

**证据**: `services/services/sa_avcodec/ipc/av_codec_service_ipc_interface_code.h:75-81`

#### Codec 服务级接口 (CodecServiceInterfaceCode)

| Code | 值 | 方法 | 描述 |
|------|-----|------|------|
| SET_LISTENER_OBJ | 0 | `SetListenerObj()` | 设置监听器 |
| INIT | 1 | `Init()` | 初始化 |
| CONFIGURE | 2 | `Configure()` | 配置 |
| PREPARE | 3 | `Prepare()` | 准备 |
| START | 4 | `Start()` | 启动 |
| STOP | 5 | `Stop()` | 停止 |
| FLUSH | 6 | `Flush()` | 刷新 |
| RESET | 7 | `Reset()` | 重置 |
| RELEASE | 8 | `Release()` | 释放 |
| ... | ... | ... | 30+ 方法 |

**证据**: `services/services/sa_avcodec/ipc/av_codec_service_ipc_interface_code.h:31-66`

### 线程模型

```
┌─────────────────────────────────────────────────────────┐
│                   IPC 线程池 (64 线程)                    │
│  职责: 处理跨进程请求，调用引擎层方法                        │
├─────────────────────────────────────────────────────────┤
│  注意: 编解码处理不在此线程执行，委托给引擎层                  │
└─────────────────────────────────────────────────────────┘
```

**证据**: `services/services/sa_avcodec/server/avcodec_server.cpp:57`
```cpp
IPCSkeleton::SetMaxWorkThreadNum(64);
```

## 数据流

### 解码流程

```
应用层
  │
  ▼ (N-API 调用)
┌────────────────────────┐
│  C-API (native_*.cpp)  │
│  OH_VideoDecoder_*()   │
└───────────┬────────────┘
            │ (IPC 调用)
            ▼
┌─────────────────────────────────────┐
│  CodecServiceStub::OnRemoteRequest() │
│  (IPC 分发)                         │
└───────────┬─────────────────────────┘
            │
            ▼
┌─────────────────────────────────────┐
│          引擎层处理                   │
│  framewoks/native/capi/avcodec/     │
└───────────┬─────────────────────────┘
            │
            ▼
┌─────────────────────────────────────┐
│         硬件/软件解码器               │
│  services/engine/codec/video/       │
│  - Hcodec (硬件)                    │
│  - Fcodec (软件)                    │
└─────────────────────────────────────┘
```

## 关键组件

### 1. AVCodecServer

**职责**: SA 主服务，处理 SA 级别的 IPC 请求

**关键方法**:
- `OnStart()`: 服务启动
- `OnStop()`: 服务停止
- `OnAddSystemAbility()`: SA 依赖就绪通知

**证据**: `services/services/sa_avcodec/server/include/avcodec_server.h:26`

### 2. CodecServiceStub

**职责**: 编解码服务的 Stub 端，处理 30+ IPC 方法调用

**关键方法**:
- `OnRemoteRequest()`: IPC 请求分发
- `Init()`, `Configure()`, `Start()`, `Stop()` 等

**证据**: `services/services/codec/ipc/codec_service_stub.cpp:179-203`

### 3. 硬件编解码器 (HCodec)

**职责**: 使用硬件加速进行编解码

**证据**: `config.gni:21`
```gni
av_codec_support_hcodec = true
```

**相关代码**:
- `services/engine/codec/video/hcodec/` - 硬件编码器
- `services/engine/codec/video/hevcdecoder/` - HEVC 解码器
- `services/engine/codec/video/avcencoder/` - AVC 编码器

### 4. 软件编解码器 (Fcodec)

**职责**: 使用 CPU 进行编解码

**证据**: `config.gni:37`
```gni
av_codec_support_fcodec = true
```

**相关代码**: `services/engine/codec/video/fcodec/`

---

**相关文档**: [对外 C-API](04_C_API.md) | [Inner API](05_Inner_API.md) | [安全风险评审](07_Security_Review.md)
