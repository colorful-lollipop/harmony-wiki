# 录制器 API

## AudioRecorder

### 模块信息

- **命名空间**: ohos.media
- **实现文件**: `frameworks/js/recorder/audio_recorder_napi.cpp`
- **注册入口**: `AudioRecorderNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| prepare | 准备录制 | AudioRecorderConfig | number: 错误码 |
| start | 开始录制 | - | number: 错误码 |
| pause | 暂停录制 | - | number: 错误码 |
| resume | 恢复录制 | - | number: 错误码 |
| stop | 停止录制 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |
| getCurrentCount | 获取当前录音字节数 | - | number: 字节数 |

### 权限要求

| 权限 | 用途 | 强制级别 |
|------|------|---------|
| ohos.permission.MICROPHONE | 麦克风录音 | user_grant |

### 权限检查

```cpp
// 文件: frameworks/js/player/audio_player_napi.cpp:226
asyncContext->SignError(MSERR_EXT_API9_NO_PERMISSION, "CreateAudioRecorder no permission");
```

## VideoRecorder

### 模块信息

- **命名空间**: ohos.media
- **实现文件**: `frameworks/js/recorder/video_recorder_napi.cpp`
- **注册入口**: `VideoRecorderNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| prepare | 准备录制 | VideoRecorderConfig | number: 错误码 |
| start | 开始录制 | - | number: 错误码 |
| pause | 暂停录制 | - | number: 错误码 |
| resume | 恢复录制 | - | number: 错误码 |
| stop | 停止录制 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |

### 权限要求

| 权限 | 用途 |
|------|------|
| ohos.permission.MICROPHONE | 麦克风录音 |
| ohos.permission.CAMERA | 摄像头 |

## AVRecorder

### 模块信息

- **命名空间**: ohos.multimedia.avrecorder
- **实现文件**: `frameworks/js/avrecorder/avrecorder_napi.cpp`
- **注册入口**: `AVRecorderNapi::Init(env, exports)`
- **支持版本**: API9+

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| prepare | 准备录制 | AVRecorderConfig | Promise\<void\> |
| start | 开始录制 | - | Promise\<void\> |
| pause | 暂停录制 | - | Promise\<void\> |
| resume | 恢复录制 | - | Promise\<void\> |
| stop | 停止录制 | - | Promise\<void\> |
| reset | 重置 | - | Promise\<void\> |
| release | 释放资源 | - | Promise\<void\> |
| getState | 获取状态 | - | string: 状态 |

### 权限要求

```cpp
// 文件: services/services/recorder/ipc/recorder_service_stub.cpp:1103
Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
```

## 错误码

| 错误码 | 含义 |
|--------|------|
| MSERR_EXT_API9_NO_PERMISSION | 无权限 |
| MSERR_EXT_API9_INVALID_VAL | 无效参数 |
| MSERR_EXT_API9_OPERATION_FAILED | 操作失败 |

## 相关文档

- [播放器 API](06_AVPlayer.md)
- [N-API 接口总览](05_NAPI_Overview.md)
- [安全风险评审](15_Security_Review.md)
