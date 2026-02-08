# 系统声音管理

## SystemSoundManager

### 模块信息

- **命名空间**: ohos.systemSoundManager
- **实现文件**: `frameworks/js/system_sound_manager/system_sound_manager_napi.cpp`
- **注册入口**: `SystemSoundManagerNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| getSystemSoundManager | 获取实例 | - | SystemSoundManager |
| getRingtone | 获取铃声 | RingtoneType | Ringtone |
| getTone | 获取提示音 | ToneType | TonePlayer |

### 权限要求

| 方法 | 权限 |
|------|-----|
| getSystemSoundManager | 无 |
| getRingtone | 无 |
| play | 无 |

### 权限检查

```cpp
// 文件: frameworks/js/system_sound_manager/system_sound_manager_napi.cpp:504
if (Security::AccessToken::AccessTokenKit::VerifyAccessToken(selfTokenID, "ohos.permission.WRITE_RINGTONE")) {
    // 权限验证
}
```

### 错误处理

```cpp
// 文件: frameworks/js/system_sound_manager/system_sound_manager_napi.cpp
ThrowCustomError(env, NAPI_ERR_PERMISSION_DENIED, "No system permission");
```

## Ringtone

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| start | 开始播放 | - | number: 错误码 |
| stop | 停止播放 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |

## TonePlayer

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| prepare | 准备播放 | TonePlayerConfig | number: 错误码 |
| start | 开始播放 | number: segmentId | number: 错误码 |
| stop | 停止播放 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |

### 权限检查

```cpp
// 文件: frameworks/js/system_sound_manager/src/system_tone_player/system_tone_player_napi.cpp
ThrowCustomError(env, NAPI_ERR_PERMISSION_DENIED, "No system permission");
```

## 相关文档

- [音频相关 API](08_Audio_APIs.md)
