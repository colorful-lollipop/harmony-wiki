# Audio Framework - N-API 接口

## 模块入口

**JS 导入名**: `multimedia.audio`

**入口文件**: `frameworks/js/napi/common/napi_audio_entry.cpp`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_register_func = Init,
    .nm_modname = "multimedia.audio",  // JS import name
};
extern "C" __attribute__((constructor)) void RegisterModule(void) {
    napi_module_register(&g_module);
}
```

**证据来源**: `napi_audio_entry.cpp:60-73`

## JS API 清单

### 1. AudioManager

**获取方式**: `const audioManager = audio.getAudioManager();`

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `setVolume(volumeType, volume)` | SetVolume | 设置音量 |
| `getVolume(volumeType)` | GetVolume | 获取音量 |
| `getMaxVolume(volumeType)` | GetMaxVolume | 获取最大音量 |
| `getMinVolume(volumeType)` | GetMinVolume | 获取最小音量 |
| `mute(volumeType, mute)` | SetStreamMute | 静音 |
| `isMute(volumeType)` | IsStreamMute | 查询静音状态 |
| `isActive(volumeType)` | IsStreamActive | 查询流状态 |
| `setRingerMode(mode)` | SetRingerMode | 设置响铃模式 |
| `getRingerMode()` | GetRingerMode | 获取响铃模式 |
| `setAudioScene(scene)` | SetAudioScene | 设置音频场景 |
| `getAudioScene()` | GetAudioScene | 获取音频场景 |
| `getDevices(deviceFlag)` | GetDevices | 获取设备列表 |
| `setDeviceActive(device, active)` | SetDeviceActive | 激活/禁用设备 |
| `isDeviceActive(device)` | IsDeviceActive | 查询设备状态 |
| `setMicrophoneMute(mute)` | SetMicrophoneMute | 麦克风静音 |
| `isMicrophoneMute()` | IsMicrophoneMute | 查询麦克风状态 |
| `on(eventName, callback)` | On | 事件订阅 |
| `off(eventName)` | Off | 取消订阅 |
| `getStreamManager()` | GetStreamManager | 获取流管理器 |
| `getVolumeManager()` | GetVolumeManager | 获取音量管理器 |
| `getRoutingManager()` | GetRoutingManager | 获取路由管理器 |
| `getEffectManager()` | GetEffectManager | 获取音效管理器 |
| `getInterruptManager()` | GetInterruptManager | 获取中断管理器 |
| `getSpatializationManager()` | GetSpatializationManager | 获取空间化管理器 |
| `getSessionManager()` | GetSessionManager | 获取会话管理器 |

**证据来源**: `frameworks/js/napi/audiomanager/napi_audio_manager.cpp:104-141`

---

### 2. AudioRenderer

**获取方式**:
```js
const audioRenderer = await audio.createAudioRenderer(options);
const audioRenderer = audio.createAudioRendererSync(options);
```

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `start()` | Start | 开始播放 |
| `write(buffer)` | Write | 写入音频数据 |
| `stop()` | Stop | 停止播放 |
| `pause()` | Pause | 暂停播放 |
| `drain()` | Drain | 排空缓冲区 |
| `flush()` | Flush | 刷新缓冲区 |
| `release()` | Release | 释放资源 |
| `setVolume(volume)` | SetVolume | 设置音量 (0.0-1.0) |
| `getVolume()` | GetVolume | 获取音量 |
| `setRenderRate(rate)` | SetRenderRate | 设置渲染速率 |
| `getRenderRate()` | GetRenderRate | 获取渲染速率 |
| `getBufferSize()` | GetBufferSize | 获取缓冲区大小 |
| `getAudioTime()` | GetAudioTime | 获取播放时间 |
| `getAudioStreamId()` | GetAudioStreamId | 获取流 ID |
| `getRendererInfo()` | GetRendererInfo | 获取渲染器信息 |
| `getStreamInfo()` | GetStreamInfo | 获取流信息 |
| `state` | GetState | 获取当前状态 |
| `on(event, callback)` | On | 事件订阅 |
| `off(event)` | Off | 取消订阅 |

**事件类型**: `stateChange`, `markReach`, `periodReach`, `interrupt`

**证据来源**: `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:80-134`

---

### 3. AudioCapturer

**获取方式**:
```js
const audioCapturer = await audio.createAudioCapturer(options);
const audioCapturer = audio.createAudioCapturerSync(options);
const audioCapturer = audio.createMicInAudioCapturer(options);
```

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `start()` | Start | 开始采集 |
| `read(buffer)` | Read | 读取音频数据 |
| `stop()` | Stop | 停止采集 |
| `release()` | Release | 释放资源 |
| `getBufferSize()` | GetBufferSize | 获取缓冲区大小 |
| `getAudioTime()` | GetAudioTime | 获取采集时间 |
| `getAudioStreamId()` | GetAudioStreamId | 获取流 ID |
| `getCapturerInfo()` | GetCapturerInfo | 获取采集器信息 |
| `getStreamInfo()` | GetStreamInfo | 获取流信息 |
| `getCurrentInputDevices()` | GetCurrentInputDevices | 获取输入设备 |
| `getCurrentMicrophones()` | GetCurrentMicrophones | 获取麦克风列表 |
| `state` | GetState | 获取当前状态 |
| `on(event, callback)` | On | 事件订阅 |
| `off(event)` | Off | 取消订阅 |

**事件类型**: `audioCapturerChange`, `readData`, `inputDeviceChange`

**证据来源**: `frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp:74-101`

---

### 4. TonePlayer

**获取方式**:
```js
const tonePlayer = await audio.createTonePlayer();
const tonePlayer = audio.createTonePlayerSync();
```

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `load(zoneId, options)` | Load | 加载音调配置 |
| `start(toneType)` | Start | 播放音调 |
| `stop()` | Stop | 停止播放 |
| `release()` | Release | 释放资源 |

**证据来源**: `frameworks/js/napi/audiorenderer/napi_toneplayer.cpp:74-84`

---

### 5. AudioVolumeManager

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `getVolumeGroupInfos()` | GetVolumeGroupInfos | 获取音量组信息 |
| `getSystemVolume(volumeType)` | GetSystemVolume | 获取系统音量 |
| `setSystemVolume(volumeType, volume)` | SetSystemVolumeByUid | 设置系统音量 |
| `getMaxSystemVolume(volumeType)` | GetMaxSystemVolume | 获取最大音量 |
| `getMinSystemVolume(volumeType)` | GetMinSystemVolume | 获取最小音量 |
| `isSystemMuted(volumeType)` | IsSystemMutedForStream | 查询系统静音 |
| `on(event, callback)` | On | 音量变化事件 |
| `off(event)` | Off | 取消订阅 |

**证据来源**: `frameworks/js/napi/audiomanager/napi_audio_volume_manager.cpp:152-186`

---

### 6. AudioRoutingManager

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `getDevices(flag)` | GetDevices | 获取设备列表 |
| `selectOutputDevice(device)` | SelectOutputDevice | 选择输出设备 |
| `selectInputDevice(device)` | SelectInputDevice | 选择输入设备 |
| `setCommunicationDevice(device, active)` | SetCommunicationDevice | 设置通信设备 |
| `getPreferredOutputDeviceForRendererInfo(info)` | GetPreferredOutputDeviceForRendererInfo | 获取首选输出设备 |
| `on(event, callback)` | On | 设备变化事件 |
| `off(event)` | Off | 取消订阅 |

**证据来源**: `frameworks/js/napi/audiomanager/napi_audio_routing_manager.cpp:90-119`

---

### 7. AudioStreamManager

| JS 方法 | C++ 函数 | 描述 |
|---------|----------|------|
| `getCurrentAudioRendererInfoArray()` | GetCurrentAudioRendererInfoArray | 获取渲染器列表 |
| `getCurrentAudioCapturerInfoArray()` | GetCurrentAudioCapturerInfoArray | 获取采集器列表 |
| `isActive(volumeType)` | IsActive | 查询流是否活跃 |
| `isAudioLoopbackSupported()` | IsAudioLoopbackSupported | 检查回环支持 |

**证据来源**: `frameworks/js/napi/audiomanager/napi_audio_stream_manager.cpp:89-113`

---

## 音频枚举值

### AudioVolumeType
- `STREAM_VOICE_CALL`, `STREAM_SYSTEM`, `STREAM_RING`
- `STREAM_MUSIC`, `STREAM_ALARM`, `STREAM_NOTIFICATION`
- `STREAM_BLUETOOTH_SCO`, `STREAM_DTMF`, `STREAM_TTS`

### DeviceType
- `DEVICE_TYPE_SPEAKER`, `DEVICE_TYPE_HEADSET`
- `DEVICE_TYPE_BLUETOOTH_SCO`, `DEVICE_TYPE_BLUETOOTH_A2DP`
- `DEVICE_TYPE_USB_HEADSET`, `DEVICE_TYPE_WIRED_HEADSET`

### AudioState
- `STATE_INVALID`, `STATE_NEW`, `STATE_PREPARED`
- `STATE_RUNNING`, `STATE_PAUSED`, `STATE_STOPPED`

**证据来源**: `frameworks/js/napi/common/napi_audio_enum.cpp`

## 调用链示例

### JS → AudioRenderer 调用链

```
JS: audio.createAudioRenderer(options)
    ↓
N-API: napi_audio_renderer.cpp (createAudioRenderer)
    ↓
Native: AudioRenderer::Create()
    ↓
IPC: IStandardAudioService::CreateAudioProcess()
    ↓
Service: AudioServer (SA 3001)
    ↓
HAL: hdiadapter_new/ (硬件交互)
```

## 相关文档

- [架构设计](02_Architecture.md)
- [构建系统](04_Build.md)
- [安全机制](05_Security.md)
