# N-API 接口总览

## 模块清单

### 播放相关

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| AudioPlayer | ohos.media | AudioPlayerNapi | `frameworks/js/player/audio_player_napi.cpp` |
| VideoPlayer | ohos.media | VideoPlayerNapi | `frameworks/js/player/video_player_napi.cpp` |
| AVPlayer | ohos.multimedia.avplayer | AVPlayerNapi | `frameworks/js/avplayer/avplayer_napi.cpp` |

### 录制相关

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| AudioRecorder | ohos.media | AudioRecorderNapi | `frameworks/js/recorder/audio_recorder_napi.cpp` |
| VideoRecorder | ohos.media | VideoRecorderNapi | `frameworks/js/recorder/video_recorder_napi.cpp` |
| AVRecorder | ohos.multimedia.avrecorder | AVRecorderNapi | `frameworks/js/avrecorder/avrecorder_napi.cpp` |

### 媒体辅助

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| AVMetadataExtractor | ohos.multimedia | AVMetadataExtractorNapi | `frameworks/js/metadatahelper/avmetadataextractor_napi.cpp` |
| AVImageGenerator | ohos.multimedia | AVImageGeneratorNapi | `frameworks/js/metadatahelper/avimagegenerator_napi.cpp` |

### 屏幕相关

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| AVScreenCapture | ohos.multimedia.avscreen_capture | AVScreenCaptureNapi | `frameworks/js/avscreen_capture/avscreen_capture_napi.cpp` |
| ScreenCaptureMonitor | ohos.multimedia | ScreenCaptureMonitorNapi | `frameworks/js/screencapturemonitor/screen_capture_monitor_napi.cpp` |

### 音频特效

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| SoundPool | ohos.multimedia.soundpool | SoundPoolNapi | `frameworks/js/soundpool/soundpool_napi.cpp` |
| AudioHaptic | ohos.multimedia.audiohaptic | AudioHapticManagerNapi | `frameworks/js/audio_haptic/audio_haptic_manager_napi.cpp` |
| SystemSoundManager | ohos.systemSoundManager | SystemSoundManagerNapi | `frameworks/js/system_sound_manager/system_sound_manager_napi.cpp` |

### 其他

| 模块 | JS 命名空间 | 主要类 | 注册位置 |
|------|------------|-------|---------|
| MediaSource | ohos.multimedia.mediasource | MediaSourceNapi | `frameworks/js/mediasource/media_source_napi.cpp` |

## N-API 注册机制

### 主入口

```cpp
// 文件: frameworks/js/media/native_module_ohos_media.cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_register_func = Export,
    .nm_modname = "multimedia.media",
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&g_module);
}
```

### 条件编译

根据编译开关导出不同模块：

```cpp
#ifdef SUPPORT_PLAYER
    AudioPlayerNapi::Init(env, exports);
    VideoPlayerNapi::Init(env, exports);
#endif
#ifdef SUPPORT_RECORDER
    AudioRecorderNapi::Init(env, exports);
    VideoRecorderNapi::Init(env, exports);
#endif
#ifdef SUPPORT_METADATA
    AVMetadataExtractorNapi::Init(env, exports);
#endif
```

## 导出方法模式

### 静态属性

```cpp
static napi_property_descriptor staticProperty[] = {
    {"VideoPlayer", nullptr, nullptr, nullptr, nullptr, static_cast<napi_property_attributes>(napi_writable | napi_configurable), nullptr},
    // ...
};
```

### 实例方法

```cpp
static napi_method DeclareJSMethods(napi_env env, napi_value exports)
{
    napi_define_properties(env, exports,
        sizeof(staticProperty) / sizeof(staticProperty[0]), staticProperty);
}
```

## 权限检查

### 前置权限检查

```cpp
// 文件: frameworks/js/player/audio_player_napi.cpp
uint64_t tokenId = IPCSkeleton::GetSelfTokenID();
// 权限验证逻辑
```

### 服务端权限校验

```cpp
// 文件: services/utils/media_permission.cpp
int32_t MediaPermission::CheckMicPermission()
{
    auto callerUid = IPCSkeleton::GetCallingUid();
    Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
    return Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenCaller, "ohos.permission.MICROPHONE");
}
```

## 错误码

### 通用错误码

| 错误码 | 含义 |
|--------|------|
| MSERR_EXT_API9_NO_PERMISSION | 无权限 |
| MSERR_EXT_API9_PERMISSION_DENIED | 权限被拒 |
| MSERR_EXT_API9_INVALID_VAL | 无效参数 |

## 相关文档

- [播放器 API](06_AVPlayer.md)
- [录制器 API](07_AVRecorder.md)
- [安全风险评审](15_Security_Review.md)
