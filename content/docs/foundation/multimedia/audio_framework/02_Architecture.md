# Audio Framework - 架构与数据流

> 本文档详细描述 OpenHarmony 音频框架的架构设计、数据流和 IPC 机制。

---

## 1. 整体架构

### 1.1 架构分层图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                 应用层                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ArkTS/JS App    │  C/C++ App    │  OpenSL ES App                    │   │
│  └──────────────────┴───────────────┴───────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                框架层 (Framework)                            │
│  ┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐  │
│  │   N-API         │  OHAudio (C)    │  OpenSL ES      │  CJ-FFI         │  │
│  │   (JS/ArkTS)    │                 │                 │                 │  │
│  └────────┬────────┴────────┬────────┴────────┬────────┴────────┬────────┘  │
│           │                 │                 │                 │           │
│  ┌────────▼─────────────────▼─────────────────▼─────────────────▼────────┐  │
│  │                      Native 框架层                                    │  │
│  │  AudioRenderer │ AudioCapturer │ AudioSystemManager │ ...           │  │
│  └────────┬──────────────────────────────────────────────────────────────┘  │
├───────────┼─────────────────────────────────────────────────────────────────┤
│           │                     IPC 层                                      │
│           │  ┌─────────────────────────────────────────────────────────┐   │
│           │  │  AudioPolicyServer (SAID: 3009) │ AudioServer (SAID: 3001) │   │
│           │  └─────────────────────────────────────────────────────────┘   │
├───────────┼─────────────────────────────────────────────────────────────────┤
│           │                    服务层 (Service)                              │
│           │  ┌─────────────────────────────────────────────────────────┐   │
│           │  │  AudioPolicy │ AudioService │ AudioEngine │ AudioSuite   │   │
│           │  └─────────────────────────────────────────────────────────┘   │
├───────────┼─────────────────────────────────────────────────────────────────┤
│           │                    驱动层 (Driver)                               │
│           │  ┌─────────────────────────────────────────────────────────┐   │
│           │  │  HDI Adapter │ PulseAudio │ Hardware Driver             │   │
│           │  └─────────────────────────────────────────────────────────┘   │
└───────────┴─────────────────────────────────────────────────────────────────┘
```

### 1.2 核心组件职责

| 组件 | 职责 | 运行域 |
|------|------|--------|
| N-API | JS/ArkTS 应用接口 | 应用进程 |
| OHAudio | C API 接口 | 应用进程 |
| Native 框架 | 业务逻辑封装 | 应用进程 |
| AudioPolicyServer | 策略决策、设备管理 | 系统服务 |
| AudioServer | 音频流管理、HDI 接口 | 系统服务 |
| AudioEngine | 高性能音频处理 | 系统服务 |
| HDI Adapter | 硬件驱动抽象 | 系统服务 |

## System Ability 详细说明

### AudioServer (SA 3001)

| 属性 | 值 |
|------|-----|
| SA ID | 3001 |
| 库文件 | `libaudio_service.z.so` |
| 进程 | audio_service |
| 基类 | SystemAbility, StandardAudioServiceStub |

**核心职责**:
- 音频设备路由和 HAL 交互
- 音效链管理
- 音频进程/流创建
- 音量/空间化控制

**证据来源**: `sa_profile/pulseaudio.json`

### AudioPolicyServer (SA 3009)

| 属性 | 值 |
|------|-----|
| SA ID | 3009 |
| 库文件 | `libaudio_policy_service.z.so` |
| 进程 | system_service |
| 基类 | SystemAbility, AudioPolicyStub |

**核心职责**:
- 音量控制/响铃模式
- 音频焦点/中断管理
- 设备选择/路由策略
- 流状态追踪

**证据来源**: `sa_profile/audio_policy.json`

## IPC 接口定义

### IStandardAudioService (SA 3001)

**文件**: `services/audio_service/idl/IStandardAudioService.idl`

| 方法类别 | 方法 |
|----------|------|
| 音频参数 | `GetAudioParameter()`, `SetAudioParameter()` |
| 设备路由 | `UpdateActiveDeviceRoute()`, `SetOutputDeviceSink()` |
| 音频处理 | `CreateAudioProcess()`, `LoadAudioEffectLibraries()` |
| 音量 | `SetVoiceVolume()`, `OffloadSetVolume()` |
| ASR | `SetAsrAecMode()`, `SetAsrNoiseSuppressionMode()` |

### IAudioPolicy (SA 3009)

**文件**: `services/audio_policy/idl/IAudioPolicy.idl`

| 方法类别 | 方法 |
|----------|------|
| 音量 | `GetMaxVolumeLevel()`, `SetSystemVolumeLevel()` |
| 设备 | `GetDevices()`, `SelectOutputDevice()`, `SelectInputDevice()` |
| 中断 | `ActivateInterrupt()`, `DeactivateInterrupt()` |
| 场景 | `SetAudioScene()`, `GetAudioScene()` |
| 追踪 | `RegisterTracker()`, `UpdateTracker()` |

## IPC 通信模式

### 1. 服务发现

```cpp
auto samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
sptr<IRemoteObject> object = samgr->GetSystemAbility(AUDIO_DISTRIBUTED_SERVICE_ID);
sptr<IStandardAudioService> proxy = iface_cast<IStandardAudioService>(object);
```

### 2. 请求分发

```cpp
int32_t OnRemoteRequest(uint32_t code, MessageParcel &data,
                       MessageParcel &reply, MessageOption &option)
```

### 3. 调用者身份识别

```cpp
IPCSkeleton::GetCallingUid();
IPCSkeleton::GetCallingPid();
IPCSkeleton::GetCallingTokenID();
```

### 4. 死亡监听

```cpp
class ProxyDeathRecipient : public IRemoteObject::DeathRecipient {
    void OnRemoteDied(const wptr<IRemoteObject> &remote) override;
};
```

## 依赖的外部 SA

| SA ID | 服务 | 用途 |
|-------|------|------|
| varies | Bluetooth Host | 蓝牙集成 |
| varies | Power Manager | 电源状态 |
| varies | Multimodal Input | 音量键事件 |
| varies | KV Store | 设置存储 |
| varies | Bundle Manager | 应用信息 |
| varies | USB Manager | USB 设备 |
| varies | Common Event | 系统事件 |

## 相关文档

- [目录结构](01_Directory_Structure.md)
- [N-API 接口](03_NAPI.md)
- [安全机制](05_Security.md)
