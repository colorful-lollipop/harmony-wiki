# Audio Framework - 攻击面分析

> 本文档分析 OpenHarmony 音频框架的攻击面，面向安全研究员。

---

## 1. 攻击面概览

### 1.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           应用层 (不可信)                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                       │
│  │   JS App     │  │  ArkTS App   │  │   C++ App    │                       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                       │
│         │                 │                 │                                │
│         ▼                 ▼                 ▼                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      N-API 层 (攻击面 #1)                            │   │
│  │  NapiAudioRenderer │ NapiAudioCapturer │ NapiAudioManager ...        │   │
│  └──────────────────────────┬──────────────────────────────────────────┘   │
└─────────────────────────────┼───────────────────────────────────────────────┘
                              │ 信任边界
┌─────────────────────────────┼───────────────────────────────────────────────┐
│                             ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                   Native 框架层 (攻击面 #2)                          │   │
│  │  AudioRenderer │ AudioCapturer │ AudioSystemManager ...              │   │
│  └──────────────────────────┬──────────────────────────────────────────┘   │
│                             │                                                │
│                             ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     IPC 层 (攻击面 #3)                               │   │
│  │  AudioServer (SAID:3001) │ AudioPolicyServer (SAID:3009)              │   │
│  └──────────────────────────┬──────────────────────────────────────────┘   │
└─────────────────────────────┼───────────────────────────────────────────────┘
                              │ 信任边界
┌─────────────────────────────┼───────────────────────────────────────────────┐
│                             ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                  系统服务层 (攻击面 #4)                              │   │
│  │  AudioService │ AudioPolicy │ AudioEngine │ HDI Adapter               │   │
│  └──────────────────────────┬──────────────────────────────────────────┘   │
│                             │                                                │
│                             ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                   配置文件 (攻击面 #5)                               │   │
│  │  XML Configs │ JSON Profiles │ Audio Files                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面统计

| 攻击面类别 | 入口数量 | 风险等级 | 说明 |
|-----------|---------|---------|------|
| N-API 接口 | 15+ 类，200+ 方法 | 中-高 | JS/ArkTS 应用直接访问 |
| IPC 接口 | 376+ 方法 | 高 | 跨进程通信 |
| 配置文件 | 15+ 文件 | 中 | XML/JSON 解析 |
| 共享内存 | 多处 | 高 | 音频数据传递 |
| HDI 驱动接口 | 多个 | 高 | 内核态交互 |

---

## 2. N-API 攻击面

### 2.1 N-API 模块清单

**注册入口**: `frameworks/js/napi/common/napi_audio_entry.cpp:36-58`

```cpp
static napi_value Init(napi_env env, napi_value exports)
{
    NapiAudioEnum::Init(env, exports);
    NapiAudioRenderer::Init(env, exports);           // 音频渲染器
    NapiAudioCapturer::Init(env, exports);           // 音频采集器
    NapiTonePlayer::Init(env, exports);              // Tone 播放器
    NapiAudioStreamMgr::Init(env, exports);          // 流管理器
    NapiAudioEffectMgr::Init(env, exports);          // 音效管理器
    NapiAudioRoutingManager::Init(env, exports);     // 路由管理器
    NapiAudioVolumeManager::Init(env, exports);      // 音量管理器
    NapiAudioInterruptManager::Init(env, exports);   // 中断管理器
    NapiAudioSpatializationManager::Init(env, exports); // 空间音频
    NapiAudioManager::Init(env, exports);            // 音频管理器
    NapiAsrProcessingController::Init(env, exports); // ASR 控制器
    NapiAudioSessionMgr::Init(env, exports);         // 会话管理器
    NapiAudioCollaborativeManager::Init(env, exports); // 协同管理器
    NapiAudioLoopback::Init(env, exports);           // 音频回环
    return exports;
}
```

### 2.2 NapiAudioRenderer 攻击面

**文件**: `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp`

#### 高危接口

| 方法名 | 参数 | 风险 | 说明 |
|--------|------|------|------|
| `write` | Buffer, length | 高 | 音频数据写入，需验证 buffer 有效性 |
| `start` | - | 中 | 启动播放，可能触发资源分配 |
| `setVolume` | float volume | 中 | 音量设置，需验证范围 [0.0, 1.0] |
| `create` | RendererOptions | 高 | 创建渲染器，需验证参数合法性 |

**关键代码**: 
```cpp
// frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:251
rendererNapi->audioRenderer_ = AudioRenderer::Create(cacheDir, rendererOptions);
// 需要验证: cacheDir 路径合法性, rendererOptions 参数范围
```

#### 输入验证分析

```cpp
// 需要检查参数验证代码...
// TODO(待分析): 详细的参数验证逻辑
```

### 2.3 NapiAudioCapturer 攻击面

**文件**: `frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp`

#### 高危接口

| 方法名 | 参数 | 风险 | 说明 |
|--------|------|------|------|
| `read` | Buffer, length | 高 | 读取音频数据，涉及麦克风访问 |
| `start` | - | 高 | 启动录音，需权限检查 |
| `create` | CapturerOptions | 高 | 创建采集器 |

**关键代码**:
```cpp
// frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp:152
napiCapturer->audioCapturer_ = AudioCapturer::Create(capturerOptions, cacheDir);
```

### 2.4 通用攻击向量

#### 参数类型混淆
```javascript
// 攻击示例：传递错误类型参数
audioRenderer.setVolume("not_a_number");  // 可能导致类型混淆
audioRenderer.write(null, -1);            // 可能导致空指针解引用
```

#### 资源耗尽
```javascript
// 攻击示例：创建大量实例
for (let i = 0; i < 10000; i++) {
    audio.createAudioRenderer({});  // 可能导致资源耗尽 DoS
}
```

---

## 3. IPC 攻击面

### 3.1 AudioServer (SAID: 3001)

**IPC 接口定义**: `services/audio_service/client/include/pulseaudio_ipc_interface_code.h`

**关键 IPC 方法**:

| 方法码 | 风险 | 说明 |
|--------|------|------|
| `CREATE_AUDIOPROCESS` | 高 | 创建音频进程，需验证权限 |
| `LOAD_AUDIO_EFFECT_LIBRARIES` | 高 | 加载音效库，路径验证关键 |
| `SET_MICROPHONE_MUTE` | 高 | 控制麦克风，需系统权限 |
| `FORCE_STOP_AUDIO_STREAM` | 中 | 强制停止音频流 |

### 3.2 AudioPolicyServer (SAID: 3009)

**IPC 接口定义**: `services/audio_policy/common/include/audio_policy_ipc_interface_code.h`

**关键 IPC 方法**:

| 方法码 | 风险 | 说明 |
|--------|------|------|
| `CREATE_RENDERER_CLIENT` | 高 | 创建渲染器客户端 |
| `CREATE_CAPTURER_CLIENT` | 高 | 创建采集器客户端，涉及麦克风 |
| `SET_SYSTEM_VOLUMELEVEL` | 中 | 设置系统音量 |
| `SET_AUDIO_SCENE` | 中 | 设置音频场景 |

### 3.3 IPC 权限检查点

**文件**: `services/audio_service/server/src/audio_server.cpp`

```cpp
// TODO(待分析): 定位 OnRemoteRequest 中的权限检查代码
// 需要验证每个 IPC 方法是否都有适当的权限校验
```

---

## 4. 配置文件攻击面

### 4.1 XML 配置文件

| 文件路径 | 风险 | 说明 |
|---------|------|------|
| `services/audio_policy/server/infra/config/file/audio_effect_config.xml` | 中 | 音效配置，XXE 风险 |
| `services/audio_policy/server/infra/config/file/audio_volume_config.xml` | 低 | 音量配置 |
| `services/audio_policy/server/infra/config/file/audio_strategy_router.xml` | 中 | 路由策略 |
| `services/audio_policy/server/infra/config/file/audio_interrupt_policy_config.xml` | 低 | 中断策略 |
| `services/audio_policy/server/infra/config/file/audio_device_privacy.xml` | 中 | 设备隐私配置 |

**XML 解析代码**:
```cpp
// TODO(待分析): 定位 XML 解析器实现
// 需要验证是否禁用外部实体 (XXE 防护)
```

### 4.2 SA 配置文件

**文件**: `sa_profile/audio_policy.json`
```json
{
    "services": [{
        "name": "audio_policy",
        "uid": "audio",
        "gid": ["audio", "system"],
        "secon": "u:r:audio_policy:s0"
    }]
}
```

**文件**: `sa_profile/pulseaudio.json`
```json
{
    "services": [{
        "name": "pulseaudio",
        "uid": "pulseaudio",
        "gid": ["pulseaudio", "shell"],
        "secon": "u:r:pulseaudio:s0"
    }]
}
```

---

## 5. 共享内存攻击面

### 5.1 OHAudioBuffer

**文件**: `services/audio_service/common/include/va_shared_buffer.h`

共享内存用于：
- 音频数据跨进程传输
- 减少数据拷贝开销

**风险**:
- 竞态条件
- 内存越界访问
- 未初始化内存使用

```cpp
// TODO(待分析): 验证共享内存的边界检查和同步机制
```

---

## 6. HDI 驱动接口攻击面

### 6.1 渲染器接口

**文件**: `frameworks/native/hdiadapter_new/include/sink/i_audio_render_sink.h`

### 6.2 采集器接口

**文件**: `frameworks/native/hdiadapter_new/include/source/i_audio_capture_source.h`

**风险**:
- 驱动接口调用可能触发内核漏洞
- 需验证所有参数传递给驱动前已校验

---

## 7. 外部输入清单

### 7.1 用户输入

| 输入点 | 类型 | 风险 |
|--------|------|------|
| N-API 参数 | JavaScript 任意值 | 高 |
| IPC 数据 | Parcel 序列化数据 | 高 |
| 配置文件 | XML/JSON | 中 |

### 7.2 系统输入

| 输入点 | 类型 | 风险 |
|--------|------|------|
| 音频文件 | WAV/PCM | 中 |
| 设备热插拔事件 | 系统事件 | 中 |
| 蓝牙连接事件 | 系统事件 | 中 |

---

## 8. 敏感操作清单

### 8.1 特权操作

| 操作 | 位置 | 所需权限 |
|------|------|---------|
| 麦克风访问 | AudioCapturer::Start | ohos.permission.MICROPHONE |
| 系统音量设置 | AudioPolicyServer | 系统权限 |
| 设备路由切换 | AudioRoutingManager | 系统权限 |
| 音频焦点抢占 | AudioInterruptManager | 应用权限 |

### 8.2 系统调用

```cpp
// TODO(待分析): 定位所有系统调用点
// 例如: ioctl, open, write 等调用硬件驱动的地方
```

---

## 9. 风险热力图

```
                    低          中          高
                 ┌──────────┬──────────┬──────────┐
    N-API 接口   │          │  setVol  │  write   │
                 │          │          │  create  │
                 ├──────────┼──────────┼──────────┤
    IPC 接口     │          │ setScene │ create   │
                 │          │          │ micMute  │
                 ├──────────┼──────────┼──────────┤
    配置文件     │ volConf  │ effect   │          │
                 │          │ config   │          │
                 ├──────────┼──────────┼──────────┤
    共享内存     │          │          │ OHBuffer │
                 │          │          │          │
                 ├──────────┼──────────┼──────────┤
    HDI 驱动     │          │          │ ioctl    │
                 │          │          │ render   │
                 └──────────┴──────────┴──────────┘
```

---

## 10. TODO 清单

### 高优先级
- [ ] 分析 N-API 参数验证的具体实现
- [ ] 定位 IPC 权限检查的完整代码
- [ ] 检查 XML 解析器的 XXE 防护
- [ ] 分析共享内存的边界检查

### 中优先级
- [ ] 检查音频文件解析的安全性
- [ ] 分析设备热插拔事件处理
- [ ] 检查蓝牙连接的安全性

### 低优先级
- [ ] 分析日志中的敏感信息泄露
- [ ] 检查调试接口的安全性

---

*最后更新: 2026-02-07*
