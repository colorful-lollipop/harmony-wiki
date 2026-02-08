# Audio Framework - 项目概览

> OpenHarmony 音频框架项目概览、核心能力和快速上手指南。

---

## 1. 项目定位

### 1.1 一句话定义

**@ohos/audio_framework** 是 OpenHarmony 多媒体子系统的核心音频框架，提供完整的音频播放、录制、设备管理、音量控制等能力。

### 1.2 项目元数据

| 属性 | 值 |
|------|-----|
| 组件名 | `@ohos/audio_framework` |
| 版本 | 4.0 |
| 许可证 | Apache License 2.0 |
| 仓库路径 | `foundation/multimedia/audio_framework` |
| 子系统 | multimedia |

### 1.3 运行域

| 层级 | 运行域 | 说明 |
|------|--------|------|
| N-API | 应用框架 (Framework) | JS/ArkTS 应用调用 |
| Native API | 应用框架 (Framework) | C/C++ 应用调用 |
| AudioService | 系统服务 (System) | SAID: 3001 |
| AudioPolicy | 系统服务 (System) | SAID: 3009 |

---

## 2. 核心能力

### 2.1 System Capability

**证据来源**: `bundle.json:15-27`

| System Capability | 描述 |
|------------------|------|
| `SystemCapability.Multimedia.Audio.Core` | 音频核心能力 |
| `SystemCapability.Multimedia.Audio.Renderer` | 音频播放（渲染） |
| `SystemCapability.Multimedia.Audio.Capturer` | 音频采集 |
| `SystemCapability.Multimedia.Audio.Device` | 设备管理 |
| `SystemCapability.Multimedia.Audio.Volume` | 音量控制 |
| `SystemCapability.Multimedia.Audio.Communication` | 通信音频 |
| `SystemCapability.Multimedia.Audio.Tone` | 音调播放（DTMF） |
| `SystemCapability.Multimedia.Audio.Interrupt` | 音频中断管理 |
| `SystemCapability.Multimedia.Audio.PlaybackCapture` | 播放流采集 |
| `SystemCapability.Multimedia.Audio.Spatialization` | 空间音频 |
| `SystemCapability.Multimedia.Audio.SuiteEngine` | Audio Suite 引擎 |

### 2.2 能力边界

**支持的功能**:
- ✅ PCM 格式音频播放/录制
- ✅ 多音频流并发管理
- ✅ 音频焦点和抢占
- ✅ 音量控制（系统/流级别）
- ✅ 设备路由管理
- ✅ 音效处理（均衡器、混响等）
- ✅ 空间音频（3D 音频）
- ✅ 蓝牙音频（A2DP、SCO）
- ✅ USB 音频设备
- ✅ 低延迟播放模式

**不支持的功能**:
- ❌ 压缩格式直接解码（如 MP3、AAC）- 需配合 AVPlayer
- ❌ MIDI 播放
- ❌ 特定 DRM 保护音频

---

## 3. 关键概念

### 3.1 音频基础

**采样 (Sampling)**: 将模拟信号在特定时间间隔内离散化的过程

**采样率 (Sampling Rate)**: 每秒采样的次数，单位 Hz
- 常见值: 8kHz, 16kHz, 44.1kHz, 48kHz, 96kHz, 192kHz
- 人耳范围: 20Hz - 20kHz

**通道 (Channel)**: 独立音频信号的空间位置
- 单声道 (Mono)
- 立体声 (Stereo)
- 多声道 (5.1, 7.1)

**音频帧 (Audio Frame)**: 2.5-60ms 的 PCM 数据单元

**PCM**: 脉冲编码调制，将模拟信号数字化的方法

### 3.2 音频流类型

| 类型 | 描述 | 典型用途 |
|------|------|----------|
| `STREAM_MUSIC` | 音乐流 | 音乐播放 |
| `STREAM_RING` | 铃声流 | 来电铃声 |
| `STREAM_ALARM` | 闹钟流 | 闹钟提醒 |
| `STREAM_VOICE_CALL` | 语音通话 | 电话通话 |
| `STREAM_NOTIFICATION` | 通知流 | 消息通知 |

### 3.3 核心组件

| 组件 | 描述 |
|------|------|
| `AudioRenderer` | 音频渲染器，用于播放 PCM 音频 |
| `AudioCapturer` | 音频采集器，用于录制 PCM 音频 |
| `AudioSystemManager` | 音频系统管理器，全局控制 |
| `AudioPolicy` | 音频策略服务，管理音频路由和焦点 |
| `AudioStream` | 音频流抽象 |

---

## 4. 快速开始

### 4.1 音频播放示例

```typescript
import audio from '@ohos.multimedia.audio';

async function playAudio() {
  // 1. 创建 AudioRenderer
  const audioRenderer = await audio.createAudioRenderer({
    streamInfo: {
      samplingRate: audio.AudioSamplingRate.SAMPLE_RATE_44100,
      channels: audio.AudioChannel.CHANNEL_2,
      sampleFormat: audio.AudioSampleFormat.SAMPLE_FORMAT_S16LE,
      encodingType: audio.AudioEncodingType.ENCODING_TYPE_RAW
    },
    rendererInfo: {
      content: audio.ContentType.CONTENT_TYPE_MUSIC,
      usage: audio.StreamUsage.STREAM_USAGE_MEDIA
    }
  });

  // 2. 启动播放
  await audioRenderer.start();

  // 3. 写入音频数据
  const buffer = new ArrayBuffer(1024);
  // ... 填充音频数据 ...
  await audioRenderer.write(buffer);

  // 4. 停止和释放
  await audioRenderer.stop();
  await audioRenderer.release();
}
```

### 4.2 音频录制示例

```typescript
import audio from '@ohos.multimedia.audio';

async function recordAudio() {
  // 注意: 需要 ohos.permission.MICROPHONE 权限
  
  // 1. 创建 AudioCapturer
  const audioCapturer = await audio.createAudioCapturer({
    streamInfo: {
      samplingRate: audio.AudioSamplingRate.SAMPLE_RATE_44100,
      channels: audio.AudioChannel.CHANNEL_2,
      sampleFormat: audio.AudioSampleFormat.SAMPLE_FORMAT_S16LE,
      encodingType: audio.AudioEncodingType.ENCODING_TYPE_RAW
    },
    capturerInfo: {
      source: audio.SourceType.SOURCE_TYPE_MIC
    }
  });

  // 2. 启动录制
  await audioCapturer.start();

  // 3. 读取音频数据
  const buffer = await audioCapturer.read(1024, true);
  // ... 处理音频数据 ...

  // 4. 停止和释放
  await audioCapturer.stop();
  await audioCapturer.release();
}
```

### 4.3 权限声明

```json
// module.json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.MICROPHONE",
        "reason": "$string:mic_permission_reason"
      }
    ]
  }
}
```

---

## 5. 架构概览

### 5.1 分层架构

```
┌─────────────────────────────────────────────────────────┐
│  应用层 (Application)                                     │
│  - ArkTS/JavaScript 应用                                  │
│  - OpenSL ES 应用                                         │
├─────────────────────────────────────────────────────────┤
│  框架层 (Framework)                                       │
│  - N-API (JS/ArkTS 接口)                                  │
│  - OHAudio (C API)                                        │
│  - OpenSL ES 兼容层                                       │
│  - CJ-FFI (Cangjie 接口)                                  │
├─────────────────────────────────────────────────────────┤
│  服务层 (Service)                                         │
│  - AudioPolicyServer (策略服务, SAID: 3009)              │
│  - AudioServer (音频服务, SAID: 3001)                    │
│  - AudioEngine (音频引擎)                                │
├─────────────────────────────────────────────────────────┤
│  驱动层 (Driver)                                          │
│  - HDI 适配器                                             │
│  - PulseAudio                                             │
│  - 硬件驱动                                               │
└─────────────────────────────────────────────────────────┘
```

### 5.2 核心服务

**AudioServer (SAID: 3001)**:
- 音频流管理
- 效果处理
- HDI 接口
- 资源管理

**AudioPolicyServer (SAID: 3009)**:
- 音频策略决策
- 设备管理
- 音量管理
- 中断管理

---

## 6. Feature 开关

**证据来源**: `config.gni:14-45`

| Feature | 默认值 | 说明 |
|---------|--------|------|
| `audio_framework_feature_wired_audio` | true | 有线音频 |
| `audio_framework_feature_usb_audio` | false | USB 音频 |
| `audio_framework_feature_dtmf_tone` | true | DTMF tone |
| `audio_framework_feature_opensl_es` | true | OpenSL ES |
| `audio_framework_feature_distributed_audio` | true | 分布式音频 |
| `audio_framework_feature_low_latency` | true | 低延迟 |
| `audio_framework_feature_inner_capturer` | true | 内部采集器 |
| `audio_framework_feature_offline_effect` | true | 离线音效 |

---

## 7. 支持设备

1. **USB Type-C 耳机** - 自带 DAC 的数字耳机
2. **WIRED 耳机** - 模拟耳机（3.5mm 或 USB-C 无 DAC）
3. **蓝牙耳机** - A2DP 无线音频传输
4. **内置扬声器/麦克风** - 默认播放/录音设备

---

## 8. 相关文档

- [目录结构](03_CodeMap.md) - 详细的目录和代码导航
- [架构设计](02_Architecture.md) - 详细的架构和数据流
- [N-API 接口](04_Interface.md) - 完整的 API 清单
- [构建系统](07_Build.md) - 构建配置和产物
- [攻击面分析](05_AttackSurface.md) - 安全分析

---

*最后更新: 2026-02-07*
