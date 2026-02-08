# Audio Framework - 接口文档

> 本文档详细描述 OpenHarmony 音频框架的 N-API 接口和 IPC 接口。

---

## 1. N-API 接口总览

### 1.1 模块注册信息

**入口文件**: `frameworks/js/napi/common/napi_audio_entry.cpp`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "multimedia.audio",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

**模块名**: `multimedia.audio`

**初始化函数**: `Init()` @ line 36-58

### 1.2 N-API 类清单

| 类名 | 文件路径 | 功能描述 |
|------|----------|----------|
| NapiAudioRenderer | `audiorenderer/napi_audio_renderer.cpp` | 音频渲染器 |
| NapiAudioCapturer | `audiocapturer/napi_audio_capturer.cpp` | 音频采集器 |
| NapiTonePlayer | `audiorenderer/napi_toneplayer.cpp` | Tone 播放器 |
| NapiAudioManager | `audiomanager/napi_audio_manager.cpp` | 音频管理器 |
| NapiAudioStreamManager | `audiomanager/napi_audio_stream_manager.cpp` | 流管理器 |
| NapiAudioEffectManager | `audiomanager/napi_audio_effect_manager.cpp` | 音效管理器 |
| NapiAudioRoutingManager | `audiomanager/napi_audio_routing_manager.cpp` | 路由管理器 |
| NapiAudioVolumeManager | `audiomanager/napi_audio_volume_manager.cpp` | 音量管理器 |
| NapiAudioVolumeGroupManager | `audiomanager/napi_audio_volume_group_manager.cpp` | 音量组管理器 |
| NapiAudioInterruptManager | `audiomanager/napi_audio_interrupt_manager.cpp` | 中断管理器 |
| NapiAudioSpatializationManager | `audiomanager/napi_audio_spatialization_manager.cpp` | 空间音频管理器 |
| NapiAudioSessionManager | `audiomanager/napi_audio_session_manager.cpp` | 会话管理器 |
| NapiAudioCollaborativeManager | `audiomanager/napi_audio_collaborative_manager.cpp` | 协同管理器 |
| NapiAudioLoopback | `audioloopback/napi_audio_loopback.cpp` | 音频回环 |
| NapiAsrProcessingController | `asrcontroller/napi_asr_processing_controller.cpp` | ASR 控制器 |
| NapiAudioEnum | `common/napi_audio_enum.cpp` | 枚举定义 |

---

## 2. AudioRenderer N-API 接口

### 2.1 接口清单

**文件**: `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp`

**属性定义** @ line 80-130:

| 方法名 | 类型 | 参数 | 返回 | 说明 |
|--------|------|------|------|------|
| create | 静态 | RendererOptions | AudioRenderer | 创建渲染器 |
| setRenderRate | 实例 | AudioRenderRate | void | 设置渲染速率 |
| getRenderRate | 实例 | - | AudioRenderRate | 获取渲染速率 |
| getRenderRateSync | 实例 | - | AudioRenderRate | 同步获取 |
| setRendererSamplingRate | 实例 | number | void | 设置采样率 |
| getRendererSamplingRate | 实例 | - | number | 获取采样率 |
| start | 实例 | - | void | 开始播放 |
| write | 实例 | ArrayBuffer, number | number | 写入音频数据 |
| getAudioTime | 实例 | - | object | 获取音频时间 |
| getAudioTimeSync | 实例 | - | object | 同步获取时间 |
| drain | 实例 | - | void | 排空缓冲区 |
| flush | 实例 | - | void | 刷新缓冲区 |
| pause | 实例 | - | void | 暂停播放 |
| stop | 实例 | - | void | 停止播放 |
| release | 实例 | - | void | 释放资源 |
| getBufferSize | 实例 | - | number | 获取缓冲区大小 |
| getBufferSizeSync | 实例 | - | number | 同步获取 |
| getAudioStreamId | 实例 | - | number | 获取流 ID |
| getAudioStreamIdSync | 实例 | - | number | 同步获取 |
| setVolume | 实例 | number | void | 设置音量 |
| getVolume | 实例 | - | number | 获取音量 |

### 2.2 关键接口详情

#### create

**位置**: `napi_audio_renderer.cpp:251`

**C++ 代码**:
```cpp
rendererNapi->audioRenderer_ = AudioRenderer::Create(cacheDir, rendererOptions);
```

**参数验证**: TODO(需分析)

#### write

**位置**: `napi_audio_renderer.cpp:87`

**风险等级**: 高

**说明**: 写入音频数据，需要验证 buffer 长度和有效性

---

## 3. AudioCapturer N-API 接口

### 3.1 接口清单

**文件**: `frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp`

| 方法名 | 类型 | 参数 | 返回 | 说明 |
|--------|------|------|------|------|
| create | 静态 | CapturerOptions, string | AudioCapturer | 创建采集器 |
| start | 实例 | - | void | 开始录制 |
| read | 实例 | number, boolean | ArrayBuffer | 读取音频数据 |
| stop | 实例 | - | void | 停止录制 |
| release | 实例 | - | void | 释放资源 |
| getBufferSize | 实例 | - | number | 获取缓冲区大小 |
| getAudioStreamId | 实例 | - | number | 获取流 ID |
| getState | 实例 | - | AudioState | 获取状态 |

### 3.2 关键接口详情

#### create

**位置**: `napi_audio_capturer.cpp:152`

**C++ 代码**:
```cpp
napiCapturer->audioCapturer_ = AudioCapturer::Create(capturerOptions, cacheDir);
```

**权限要求**: ohos.permission.MICROPHONE

#### read

**位置**: TODO

**风险等级**: 高

**说明**: 读取麦克风数据，涉及隐私

---

## 4. AudioManager N-API 接口

### 4.1 接口清单

**文件**: `frameworks/js/napi/audiomanager/napi_audio_manager.cpp`

| 方法名 | 功能 |
|--------|------|
| getAudioManager | 获取管理器实例 |
| setVolume | 设置音量 |
| getVolume | 获取音量 |
| setMute | 设置静音 |
| isMute | 是否静音 |
| setRingerMode | 设置铃声模式 |
| getRingerMode | 获取铃声模式 |
| setMicrophoneMute | 设置麦克风静音 |
| isMicrophoneMute | 麦克风是否静音 |
| setDeviceActive | 激活设备 |
| isDeviceActive | 设备是否激活 |
| getDevices | 获取设备列表 |
| setAudioScene | 设置音频场景 |
| getAudioScene | 获取音频场景 |
| on | 注册事件监听 |
| off | 注销事件监听 |

---

## 5. IPC 接口总览

### 5.1 AudioServer IPC (SAID: 3001)

**配置文件**: `sa_profile/pulseaudio.json`

**IPC 接口定义**: `services/audio_service/client/include/pulseaudio_ipc_interface_code.h`

**接口数量**: 128+ 个方法码

#### 关键 IPC 方法

| 方法码 | 值 | 说明 |
|--------|-----|------|
| GET_AUDIO_PARAMETER | 0 | 获取音频参数 |
| SET_AUDIO_PARAMETER | 1 | 设置音频参数 |
| SET_MICROPHONE_MUTE | 2 | 设置麦克风静音 |
| SET_AUDIO_SCENE | 3 | 设置音频场景 |
| UPDATE_ROUTE_REQ | 4 | 更新路由请求 |
| CREATE_AUDIOPROCESS | 10 | 创建音频进程 |
| LOAD_AUDIO_EFFECT_LIBRARIES | 20 | 加载音效库 |
| OFFLOAD_SET_VOLUME | 30 | 设置 Offload 音量 |
| SET_SPATIALIZATION_SCENE_TYPE | 40 | 设置空间音频场景 |
| CREATE_AUDIOWORKGROUP | 50 | 创建音频工作组 |
| FORCE_STOP_AUDIO_STREAM | 60 | 强制停止音频流 |
| SET_KARAOKE_PARAMETERS | 70 | 设置卡拉 OK 参数 |

### 5.2 AudioPolicyServer IPC (SAID: 3009)

**配置文件**: `sa_profile/audio_policy.json`

**IPC 接口定义**: `services/audio_policy/common/include/audio_policy_ipc_interface_code.h`

**接口数量**: 248+ 个方法码

#### 关键 IPC 方法

| 方法码 | 说明 |
|--------|------|
| GET_MAX_VOLUMELEVEL | 获取最大音量 |
| GET_MIN_VOLUMELEVEL | 获取最小音量 |
| SET_SYSTEM_VOLUMELEVEL | 设置系统音量 |
| GET_SYSTEM_VOLUMELEVEL | 获取系统音量 |
| SET_STREAM_MUTE | 设置流静音 |
| GET_STREAM_MUTE | 获取流静音状态 |
| SET_DEVICE_ACTIVE | 激活设备 |
| IS_DEVICE_ACTIVE | 设备是否激活 |
| GET_ACTIVE_OUTPUT_DEVICE | 获取活动输出设备 |
| GET_ACTIVE_INPUT_DEVICE | 获取活动输入设备 |
| SET_AUDIO_SCENE | 设置音频场景 |
| GET_AUDIO_SCENE | 获取音频场景 |
| SET_MICROPHONE_MUTE | 设置麦克风静音 |
| IS_MICROPHONE_MUTE | 麦克风是否静音 |
| CREATE_RENDERER_CLIENT | 创建渲染器客户端 |
| CREATE_CAPTURER_CLIENT | 创建采集器客户端 |
| REGISTER_TRACKER | 注册追踪器 |
| UPDATE_TRACKER | 更新追踪器 |
| GET_RENDERER_CHANGE_INFOS | 获取渲染器变更信息 |
| GET_CAPTURER_CHANGE_INFOS | 获取采集器变更信息 |
| IS_SPATIALIZATION_ENABLED | 空间音频是否启用 |
| SET_SPATIALIZATION_ENABLED | 启用/禁用空间音频 |
| ACTIVATE_AUDIO_SESSION | 激活音频会话 |
| DEACTIVATE_AUDIO_SESSION | 停用音频会话 |

---

## 6. 配置文件接口

### 6.1 XML 配置文件

| 配置文件 | 路径 | 用途 |
|---------|------|------|
| audio_effect_config.xml | `services/audio_policy/server/infra/config/file/` | 音效配置 |
| audio_volume_config.xml | `services/audio_policy/server/infra/config/file/` | 音量配置 |
| audio_strategy_router.xml | `services/audio_policy/server/infra/config/file/` | 路由策略 |
| audio_interrupt_policy_config.xml | `services/audio_policy/server/infra/config/file/` | 中断策略 |
| audio_device_privacy.xml | `services/audio_policy/server/infra/config/file/` | 设备隐私 |

### 6.2 SA 配置

**audio_policy.json**:
```json
{
    "services": [{
        "name": "audio_policy",
        "path": ["/system/bin/sa_main", "/system/profile/audio_policy.json"],
        "uid": "audio",
        "gid": ["audio", "system"],
        "secon": "u:r:audio_policy:s0"
    }]
}
```

**pulseaudio.json**:
```json
{
    "services": [{
        "name": "pulseaudio",
        "path": ["/system/bin/sa_main", "/system/profile/pulseaudio.json"],
        "uid": "pulseaudio",
        "gid": ["pulseaudio", "shell"],
        "secon": "u:r:pulseaudio:s0"
    }]
}
```

---

## 7. 权限要求

### 7.1 N-API 接口权限

| 接口 | 权限 | 类型 |
|------|------|------|
| AudioCapturer.create | ohos.permission.MICROPHONE | user_grant |
| AudioCapturer.start | ohos.permission.MICROPHONE | user_grant |
| AudioManager.setMicrophoneMute | 系统权限 | system_grant |
| AudioManager.setRingerMode | 系统权限 | system_grant |
| AudioManager.setAudioScene | 系统权限 | system_grant |

### 7.2 IPC 接口权限

**AudioServer**:
- 大多数接口需要系统权限
- SET_MICROPHONE_MUTE: 系统权限

**AudioPolicyServer**:
- CREATE_CAPTURER_CLIENT: 需要 MICROPHONE 权限
- SET_SYSTEM_VOLUMELEVEL: 系统权限
- ACTIVATE_AUDIO_SESSION: 应用权限

---

## 8. 接口调用链

### 8.1 音频播放调用链

```
JS: audioRenderer.write(buffer)
  → NapiAudioRenderer::Write (napi_audio_renderer.cpp)
    → AudioRenderer::Write (native/audiorenderer/audio_renderer.cpp)
      → IAudioStream::Write (native/audiostream/)
        → IPC: AudioServer::Write
          → AudioService::ProcessAudio (services/audio_service/)
            → HDI: RenderSink::Render (hdiadapter_new/sink/)
              → 硬件驱动
```

### 8.2 音频录制调用链

```
JS: audioCapturer.read(size, isBlocking)
  → NapiAudioCapturer::Read (napi_audio_capturer.cpp)
    → AudioCapturer::Read (native/audiocapturer/audio_capturer.cpp)
      → IPC: AudioPolicy::CreateCapturerClient (权限检查点)
        → IAudioStream::Read
          → HDI: CaptureSource::Capture
            → 麦克风硬件
```

---

*最后更新: 2026-02-07*
