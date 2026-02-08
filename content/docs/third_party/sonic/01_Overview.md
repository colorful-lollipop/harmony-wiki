# Sonic 库概览

## 1. 原始库简介

### 1.1 基本信息

| 属性 | 内容 |
|------|------|
| **库名称** | Sonic |
| **上游仓库** | https://github.com/waywardgeek/sonic |
| **作者** | Bill Cox (waywardgeek@gmail.com) |
| **版本** | 0.2.0 |
| **许可证** | Apache 2.0 License |
| **编程语言** | C (ANSI C) / Java |

### 1.2 功能描述

Sonic 是一个**语音变速算法库**，专门用于加速或减速语音播放，同时保持语音的自然度和可理解性。

**核心特性**:
- **高倍速支持**: 优化支持 2 倍速以上的加速，最高可达 6 倍速或更高
- **高质量**: 在任何变速因子下都能生成高质量的语音输出
- **双向变速**: 支持加速（speed > 1.0）和减速（speed < 1.0）
- **低延迟**: 适用于实时流式语音应用
- **轻量级**: 纯 C 实现，无外部依赖

### 1.3 算法原理

Sonic 实现了 Bill Cox 发明的新算法，主要基于以下原理：

#### 高速变速（speed >= 2.0）
使用线性插值在两个音高周期之间生成新样本：
```
newSamples = period / (speed - 1.0)
out[t] = (samples[t] * (newSamples - t) + samples[t + period] * t) / newSamples
```

#### 中速变速（1.0 < speed < 2.0）
使用 **PICOLA (Pitch Synchronous Overlap and Add)** 算法：
1. 首先用上述算法将速度加倍
2. 然后直接将输入复制到输出以达到目标速度

#### 音高检测
使用 **AMDF (Average Magnitude Difference Function)** 进行音高检测：
- 支持多通道音频（自动混合为单声道进行音高检测）
- 支持降采样以提高速度（quality = 0 时）

### 1.4 原始 API 概述

Sonic 提供两种使用模式：

#### 流式接口（推荐）
```c
// 创建流
sonicStream stream = sonicCreateStream(sampleRate, numChannels);

// 设置参数
sonicSetSpeed(stream, 2.0f);   // 2倍速
sonicSetPitch(stream, 1.0f);   // 保持音高
sonicSetRate(stream, 1.0f);    // 保持采样率

// 写入音频数据
sonicWriteShortToStream(stream, samples, numSamples);

// 读取处理后的数据
int samplesRead = sonicReadShortFromStream(stream, outBuffer, maxSamples);

// 清理
sonicDestroyStream(stream);
```

#### 非流式接口（简单场景）
```c
// 一次性处理
int newNumSamples = sonicChangeShortSpeed(
    samples, numSamples, speed, pitch, rate, 
    volume, useChordPitch, sampleRate, numChannels
);
```

---

## 2. Sonic 在 OpenHarmony 中的作用和定位

### 2.1 功能定位

根据 `bundle.json` 中的描述：

> Sonic is a simple algorithm for speeding up or slowing down speech, **which is used by multimedia audio subsystems for change speed playback**.

**在 OH 中的主要用途**:
- **音频变速播放**: 多媒体音频子系统的变速播放功能
- **应用场景**: 
  - 音频播放器倍速播放（1.5x、2x 等）
  - 语音消息变速播放
  - 有声读物变速播放

### 2.2 架构位置

```
┌─────────────────────────────────────────────────────────┐
│                     应用层                               │
│  (音频播放器、语音消息、有声读物等)                        │
├─────────────────────────────────────────────────────────┤
│                   多媒体框架                             │
│         (media_library, player_framework)               │
├─────────────────────────────────────────────────────────┤
│                   音频子系统                             │
│    (audio_framework, audio_service, codec)              │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   音频编解码  │  │  音频效果处理  │  │   **sonic**      │  │
│  │   (codec)   │  │   (effect)   │  │  (变速处理)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 2.3 集成特点

| 特点 | 说明 |
|------|------|
| **无侵入性** | 作为独立算法库，不修改上层框架代码 |
| **纯算法** | 只提供音频处理功能，不涉及系统服务 |
| **低耦合** | 不依赖 OH 其他组件，仅有标准 C 库依赖 |
| **平台 SDK** | 标记为 `platformsdk` 内部 API，供系统组件使用 |

### 2.4 与 OH 版本的关联

| OH 版本 | sonic 版本 | 主要变更 |
|---------|-----------|----------|
| 当前版本 | 0.2.0 | 基础版本 + 双声道 Bug 修复 |

### 2.5 与其他音频库的关系

| 库/组件 | 关系 | 说明 |
|---------|------|------|
| audio_framework | 使用者 | 音频框架可能调用 sonic 实现变速 |
| media_foundation | 使用者 | 媒体基础库可能集成变速功能 |
| libogg/libvorbis | 无关 | 音频编解码，与 sonic 功能正交 |
| ffmpeg | 无关 | 多媒体处理，sonic 专注于语音变速 |

---

## 3. 关键技术指标

### 3.1 性能数据

根据原始 README 的性能测试（HP Pavilion dm4, Ubuntu 11.04）：

**测试文件**: 751,958,176 字节（9 小时 28 分钟，单声道 16-bit 11KHz）

| 实现 | 实际时间 | 用户时间 | 系统时间 |
|------|---------|---------|---------|
| C 版本 | 50.8s | 47.4s | 0.6s |
| Java 版本 | 52.0s | 51.2s | 0.3s |

**分析**: 处理速度约为 **11,200 倍实时**，性能优异。

### 3.2 支持的音频格式

| 格式 | 支持状态 | 接口函数 |
|------|---------|---------|
| 16-bit PCM | 原生支持 | `sonicWriteShortToStream` / `sonicReadShortFromStream` |
| Float (-1.0 ~ 1.0) | 原生支持 | `sonicWriteFloatToStream` / `sonicReadFloatFromStream` |
| 8-bit 无符号 | 原生支持 | `sonicWriteUnsignedCharToStream` / `sonicReadUnsignedCharFromStream` |

### 3.3 功能参数范围

| 参数 | 范围 | 默认值 | 说明 |
|------|------|--------|------|
| speed | 0.1 ~ 10.0+ | 1.0 | 播放速度 |
| pitch | 0.1 ~ 10.0+ | 1.0 | 音高调整 |
| rate | 0.1 ~ 10.0+ | 1.0 | 采样率调整（同时影响速度和音高）|
| volume | 0.0 ~ 10.0+ | 1.0 | 音量缩放 |
| sampleRate | 任意 | - | 输入音频采样率 |
| numChannels | 1, 2 | - | 声道数（支持单声道/立体声）|
| quality | 0, 1 | 0 | 质量模式（0=快速，1=高质量）|
| useChordPitch | 0, 1 | 0 | 和弦音高模式 |

---

## 4. 文档与资源

### 4.1 上游文档

| 资源 | 位置 |
|------|------|
| 上游仓库 | https://github.com/waywardgeek/sonic |
| 算法文档 | sonic.h 头部注释（详细算法描述）|
| 示例代码 | main.c（命令行工具示例）|

### 4.2 OH 相关文档

本文档目录结构：
```
wiki/
├── 01_Overview.md          # 本文件 - 库概览
├── 02_Patches.md           # Patch 详细分析
├── 03_Build_Integration.md # OH 构建适配
├── 04_Usage_in_OH.md       # 依赖关系与使用
├── 05_API_Differences.md   # API/接口差异
└── 06_Security.md          # 安全风险分析
```

### 4.3 源代码结构

```
third_party/sonic/
├── sonic.h          # 头文件，API 定义
├── sonic.c          # 核心实现，约 1200 行
├── main.c           # 命令行工具示例
├── wave.c/wave.h    # WAV 文件处理辅助
├── Sonic.java       # Java 版本实现
├── Main.java        # Java 示例
├── samples/         # 测试音频样本
└── doc/             # 文档
```
