# 音频相关 API

## SoundPool

### 模块信息

- **命名空间**: ohos.multimedia.soundpool
- **实现文件**: `frameworks/js/soundpool/soundpool_napi.cpp`
- **注册入口**: `SoundPoolNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| constructor | 创建 SoundPool | SoundPoolConfig | SoundPool |
| load | 加载音效 | string/buffer | number: soundId |
| unload | 卸载音效 | number: soundId | boolean |
| play | 播放音效 | number: soundId | number: streamId |
| stop | 停止播放 | number: streamId | boolean |
| setVolume | 设置音量 | number: left, right | boolean |
| setLoop | 设置循环 | number: streamId, loop | boolean |
| setRate | 设置播放速率 | number: streamId, rate | boolean |
| release | 释放资源 | - | boolean |

### 权限要求

```cpp
// 文件: frameworks/js/soundpool/soundpool_napi.cpp:241
asyncCtx->SignError(MSERR_EXT_API9_PERMISSION_DENIED, "failed to get without permission");
```

## AudioHaptic

### 模块信息

- **命名空间**: ohos.multimedia.audiohaptic
- **实现文件**: `frameworks/js/audio_haptic/audio_haptic_manager_napi.cpp`
- **注册入口**: `AudioHapticManagerNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| setAudioAndHaptic | 设置关联 | AudioHaptic | boolean |
| start | 开始播放 | number: id | boolean |
| stop | 停止播放 | number: id | boolean |
| release | 释放资源 | - | boolean |

### 代码证据

```cpp
// 文件: frameworks/js/audio_haptic/src/audio_haptic_manager_napi.cpp:802
napi_module_register(&g_module);
```

## 相关文档

- [播放器 API](06_AVPlayer.md)
- [系统声音管理](09_SystemSoundManager.md)
