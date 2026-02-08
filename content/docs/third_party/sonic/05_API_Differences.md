# Sonic API 与接口差异

## 1. API 概述

### 1.1 原始库 API 完整性

Sonic 在 OpenHarmony 中**保持了完整的原始 API**，没有添加或删除任何接口函数。

**API 文件**: `third_party/sonic/sonic.h`

| API 类别 | 函数数量 | 状态 |
|----------|----------|------|
| 流生命周期管理 | 2 | 完整保留 |
| 数据写入 | 3 | 完整保留 |
| 数据读取 | 3 | 完整保留 |
| 参数设置 | 14 | 完整保留 |
| 批量处理 | 2 | 完整保留 |
| 频谱图功能 | 7 | 完整保留（条件编译）|

### 1.2 API 列表

#### 流生命周期管理
```c
sonicStream sonicCreateStream(int sampleRate, int numChannels);
void sonicDestroyStream(sonicStream stream);
```

#### 数据写入
```c
int sonicWriteFloatToStream(sonicStream stream, float* samples, int numSamples);
int sonicWriteShortToStream(sonicStream stream, short* samples, int numSamples);
int sonicWriteUnsignedCharToStream(sonicStream stream, unsigned char* samples, int numSamples);
```

#### 数据读取
```c
int sonicReadFloatFromStream(sonicStream stream, float* samples, int maxSamples);
int sonicReadShortFromStream(sonicStream stream, short* samples, int maxSamples);
int sonicReadUnsignedCharFromStream(sonicStream stream, unsigned char* samples, int maxSamples);
```

#### 参数设置（Getter/Setter）
```c
// 速度
float sonicGetSpeed(sonicStream stream);
void sonicSetSpeed(sonicStream stream, float speed);

// 音高
float sonicGetPitch(sonicStream stream);
void sonicSetPitch(sonicStream stream, float pitch);

// 速率
float sonicGetRate(sonicStream stream);
void sonicSetRate(sonicStream stream, float rate);

// 音量
float sonicGetVolume(sonicStream stream);
void sonicSetVolume(sonicStream stream, float volume);

// 和弦音高
int sonicGetChordPitch(sonicStream stream);
void sonicSetChordPitch(sonicStream stream, int useChordPitch);

// 质量
int sonicGetQuality(sonicStream stream);
void sonicSetQuality(sonicStream stream, int quality);

// 采样率
int sonicGetSampleRate(sonicStream stream);
void sonicSetSampleRate(sonicStream stream, int sampleRate);

// 声道数
int sonicGetNumChannels(sonicStream stream);
void sonicSetNumChannels(sonicStream stream, int numChannels);
```

#### 批量处理（简单模式）
```c
int sonicChangeFloatSpeed(float* samples, int numSamples, float speed,
                          float pitch, float rate, float volume,
                          int useChordPitch, int sampleRate, int numChannels);

int sonicChangeShortSpeed(short* samples, int numSamples, float speed,
                          float pitch, float rate, float volume,
                          int useChordPitch, int sampleRate, int numChannels);
```

#### 其他控制
```c
int sonicFlushStream(sonicStream stream);
int sonicSamplesAvailable(sonicStream stream);
```

---

## 2. OH 特有修改

### 2.1 API 修改汇总

| 类型 | 数量 | 说明 |
|------|------|------|
| **新增 API** | 0 | 无 OH 特有 API 增加 |
| **删除 API** | 0 | 无 API 删除 |
| **修改 API** | 0 | 无 API 签名修改 |
| **行为变更** | 1 | 双声道处理 Bug 修复 |

### 2.2 行为变更详情

#### 双声道处理修复

**影响函数**: `sonicWriteShortToStream`, `sonicWriteFloatToStream` 等所有写入函数

**变更内容**: 
- **修复前**: 处理双声道（2ch）音频时，输出错误地变为单声道，并产生杂音
- **修复后**: 正确处理双声道音频，保持声道数不变，无杂音

**技术细节**: 
- 涉及 `sonic.c` 中的缓冲区大小计算和交错采样索引处理
- 具体修改涉及 `numChannels` 参数在所有处理路径中的正确传递

**上游版本**: 当前修复可能未合入上游（ba331411f17702e01f6c2d7016eefebaa695871f 之后的修改）

---

## 3. API 使用限制

### 3.1 编译时限制

**通过 `sonic.h` 定义的常量**:

| 常量 | 值 | 说明 |
|------|-----|------|
| `SONIC_MIN_PITCH` | 65 | 最小音高频率（Hz）|
| `SONIC_MAX_PITCH` | 400 | 最大音高频率（Hz）|
| `SONIC_AMDF_FREQ` | 4000 | AMDF 降采样频率 |

### 3.2 运行时限制

**参数有效范围**:

| 参数 | 最小值 | 最大值 | 默认值 |
|------|--------|--------|--------|
| `speed` | > 0.0 | 无硬性限制 | 1.0 |
| `pitch` | > 0.0 | 无硬性限制 | 1.0 |
| `rate` | > 0.0 | 无硬性限制 | 1.0 |
| `volume` | 0.0 | 无硬性限制 | 1.0 |
| `sampleRate` | 1000 | 500000 | - |
| `numChannels` | 1 | 32 | - |
| `quality` | 0 | 1 | 0 |

**建议范围**（根据算法设计）:
- `speed`: 0.5 ~ 6.0（最优化范围）
- `pitch`: 0.9 ~ 1.1（保持自然语音）
- `quality`: 0（性能优先）或 1（质量优先）

---

## 4. 与上游版本的差异

### 4.1 版本对比

| 版本 | 来源 | 主要差异 |
|------|------|----------|
| **上游 master** | GitHub | 最新版本，可能包含新特性 |
| **OH 当前版本** | ba331411f17702e01f6c2d7016eefebaa695871f + 双声道修复 | 基于上游特定提交，添加了双声道修复 |

### 4.2 特性对比

| 特性 | 上游最新 | OH 当前版本 | 差异说明 |
|------|----------|-------------|----------|
| 基础变速功能 | 支持 | 支持 | 无差异 |
| 双声道处理 | 可能已修复 | 已修复 | OH 版本确定修复 |
| 频谱图功能 | 支持（可选） | 支持（可选） | 需定义 `SONIC_SPECTROGRAM` |
| 单元测试 | 最新添加 | 无 | OH 版本较旧 |
| Java 版本 | 支持 | 支持 | 无差异 |

---

## 5. 调用示例

### 5.1 C 语言示例

```c
#include "sonic.h"
#include <stdio.h>
#include <stdlib.h>

// 变速播放示例
int process_audio_speed(const char* input_file, const char* output_file, float speed) {
    // 参数设置
    int sampleRate = 44100;
    int numChannels = 2;
    
    // 创建 sonic 流
    sonicStream stream = sonicCreateStream(sampleRate, numChannels);
    if (!stream) {
        fprintf(stderr, "Failed to create sonic stream\n");
        return -1;
    }
    
    // 设置变速参数
    sonicSetSpeed(stream, speed);        // 变速（如 2.0 = 2倍速）
    sonicSetPitch(stream, 1.0f);         // 保持音高
    sonicSetRate(stream, 1.0f);          // 保持采样率
    sonicSetVolume(stream, 1.0f);        // 保持音量
    sonicSetQuality(stream, 0);          // 快速模式（0）或高质量（1）
    
    // 模拟：读取输入音频并写入 sonic
    // ... 读取音频数据到 buffer ...
    // sonicWriteShortToStream(stream, buffer, numSamples);
    
    // 刷新处理
    sonicFlushStream(stream);
    
    // 读取处理后的数据
    // ... 从 sonic 读取：sonicReadShortFromStream(stream, outBuffer, maxSamples) ...
    
    // 清理
    sonicDestroyStream(stream);
    
    return 0;
}
```

### 5.2 Java 语言示例

```java
import java.io.*;

public class SonicExample {
    public static void main(String[] args) throws IOException {
        // 参数设置
        int sampleRate = 44100;
        int numChannels = 2;
        float speed = 2.0f;  // 2倍速
        
        // 创建 Sonic 实例
        Sonic sonic = new Sonic(sampleRate, numChannels);
        
        // 设置参数
        sonic.setSpeed(speed);
        sonic.setPitch(1.0f);
        sonic.setRate(1.0f);
        sonic.setVolume(1.0f);
        
        // 读取和写入音频数据
        byte[] inBuffer = new byte[4096];
        byte[] outBuffer = new byte[4096];
        
        FileInputStream fis = new FileInputStream("input.wav");
        FileOutputStream fos = new FileOutputStream("output.wav");
        
        int numRead;
        while ((numRead = fis.read(inBuffer)) != -1) {
            sonic.writeBytesToStream(inBuffer, numRead);
            int numWritten;
            do {
                numWritten = sonic.readBytesFromStream(outBuffer, outBuffer.length);
                if (numWritten > 0) {
                    fos.write(outBuffer, 0, numWritten);
                }
            } while (numWritten > 0);
        }
        
        fis.close();
        fos.close();
    }
}
```

---

## 6. 最佳实践

### 6.1 性能优化建议

| 建议 | 说明 |
|------|------|
| **使用 quality=0** | 默认质量模式已足够好，且速度更快 |
| **批量处理** | 每次写入足够多的样本（如 4096 样本）以减少处理开销 |
| **避免频繁设置参数** | 参数设置会重置内部状态，尽量在流创建时设置 |
| **合理缓冲区大小** | 输出缓冲区应足够大以容纳变速后的数据 |

### 6.2 质量优化建议

| 建议 | 说明 |
|------|------|
| **使用 quality=1** | 高质量模式适合对音质要求高的场景 |
| **保持 pitch=1.0** | 变速时不改变音高可获得最佳语音质量 |
| **适当速度范围** | 速度在 0.5~3.0 范围内质量最佳 |

### 6.3 常见陷阱

| 陷阱 | 说明 | 解决方案 |
|------|------|----------|
| **输出缓冲区不足** | 变速后数据量可能大于输入 | 输出缓冲区应 ≥ 输入缓冲区 × max(speed, 1/pitch) |
| **声道数不匹配** | 输入声道数与 stream 配置不符 | 确保所有数据与创建时指定的 numChannels 一致 |
| **采样率不匹配** | 输入采样率与配置不符 | 确保所有数据与创建时指定的 sampleRate 一致 |
| **忘记 flush** | 流结束时未 flush 导致数据丢失 | 结束写入后调用 sonicFlushStream |

---

## 7. 总结

### 7.1 API 兼容性

- **完全兼容上游**: OH 版本保留了所有上游 API
- **无扩展 API**: 没有添加 OH 特有的 API 函数
- **行为一致**: 除双声道 Bug 修复外，其他行为与上游一致

### 7.2 升级建议

当升级上游版本时：
1. **保留 API 兼容性**: 新版本的 sonic 应保持 API 兼容
2. **验证双声道处理**: 确认新版本的双声道处理是否正常
3. **测试回归**: 运行变速播放功能测试确保无回归
