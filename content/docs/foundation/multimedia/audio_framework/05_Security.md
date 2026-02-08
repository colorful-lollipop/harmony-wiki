# Audio Framework - 权限与安全机制

## 权限清单

### 应用级权限

| 权限 | 描述 | 用途 |
|------|------|------|
| `ohos.permission.MICROPHONE` | 麦克风访问 | 音频采集 |
| `ohos.permission.MODIFY_AUDIO_SETTINGS` | 修改音频设置 | 系统设置 |
| `ohos.permission.ACCESS_NOTIFICATION_POLICY` | 访问通知策略 | 通知相关 |
| `ohos.permission.CAPTURE_VOICE_DOWNLINK_AUDIO` | 采集下行语音 | 通话录音 |
| `ohos.permission.RECORD_VOICE_CALL` | 录音通话 | 通话录音 |
| `ohos.permission.INJECT_PLAYBACK_TO_AUDIO_CAPTURE` | 注入播放流 | 通话内录 |

### 服务级权限

| 权限 | 描述 |
|------|------|
| `ohos.permission.DUMP_AUDIO` | 音频转储 |
| `ohos.permission.MANAGE_INTELLIGENT_VOICE` | 管理智能语音 |
| `ohos.permission.CAST_AUDIO_OUTPUT` | 投射音频输出 |
| `ohos.permission.CAPTURE_PLAYBACK` | 采集播放流 |

**证据来源**: `interfaces/inner_api/native/audiocommon/include/audio_info.h:92-97`
**证据来源**: `services/audio_service/server/src/audio_server.cpp`

## 权限验证机制

### PermissionUtil 类

**位置**: `frameworks/native/audioutils/src/audio_utils.cpp`

| 方法 | 描述 |
|------|------|
| `VerifyIsAudio()` | 验证调用者是否为 audio 服务 |
| `VerifyIsShell()` | 验证是否为 shell 令牌 |
| `VerifyIsSystemApp()` | 验证是否为系统应用 |
| `VerifySelfPermission(permName)` | 验证自生是否有系统权限 |
| `VerifySystemPermission(permName)` | 验证调用者是否有系统权限 |
| `VerifyPermission(permName, tokenId)` | 验证特定权限 |
| `CheckCallingUidPermission(allowedUids)` | 验证调用 UID 是否在白名单 |

### AccessTokenKit 使用

```cpp
#include <security/access_token.h>
using namespace OHOS::Security::AccessToken;

// 权限验证
int ret = AccessTokenKit::VerifyAccessToken(tokenId, permissionName, false);
if (ret != PermissionState::PERMISSION_GRANTED) {
    return ERR_PERMISSION_DENIED;
}
```

## 特殊 UID 白名单

| UID | 服务/组件 | 用途 |
|-----|-----------|------|
| UID_AUDIO | Audio Service | 音频服务 |
| UID_MEDIA | Media Service | 媒体服务 |
| UID_BOOTUP_MUSIC | 开机音乐 | 开机音乐播放 |
| ROOT_UID | Root | 调试模式 |

## 权限检查点

### AudioServer (SA 3001)

**文件**: `services/audio_service/server/src/audio_server.cpp`

| 检查点 | 验证方法 | 权限要求 |
|--------|----------|----------|
| `VerifyClientPermission()` | AccessTokenKit | 系统级权限 |
| `CheckPlaybackPermission()` | VerifyClientPermission | 播放权限 |
| `CheckRecorderPermission()` | VerifyClientPermission | 录制权限 |
| `VerifyBackgroundCapture()` | AccessTokenKit | 后台采集权限 |

### AudioPolicyServer (SA 3009)

**文件**: `services/audio_policy/server/service/service_main/src/audio_policy_server.cpp`

| 检查点 | 验证方法 | 权限要求 |
|--------|----------|----------|
| `VerifyPermission()` | AccessTokenKit | 系统级权限 |
| `VerifyBluetoothPermission()` | VerifyClientPermission | 蓝牙权限 |
| `VerifyVoiceCallPermission()` | VerifyClientPermission | 通话权限 |

## 安全风险分析

### ✅ 已实现的安全措施

1. **权限验证**: 所有敏感 API 均有权限检查
2. **Token 验证**: 使用 AccessTokenKit 验证权限令牌
3. **UID 白名单**: 关键服务仅接受特定 UID
4. **后台采集保护**: BackgroundCapture 需要额外验证
5. **系统应用区分**: System app 与普通应用权限分离

### ⚠️ 潜在风险点

#### 风险 1: 路径遍历风险

**位置**: `services/audio_service/common/src/audio_config_parser.cpp`

**描述**: XML 配置文件路径如果未正确验证，可能导致路径遍历攻击

**建议**: 所有配置文件路径应限制在 `/system/etc/audio/` 目录内

#### 风险 2: 音量设置范围验证

**位置**: `services/audio_policy/server/domain/volume/src/audio_volume_server.cpp`

**描述**: `SetVolume()` 需要验证音量值在 0-maxVolume 范围内

**证据**: 需验证 `audio_policy_server.cpp` 中的音量参数校验逻辑

#### 风险 3: 设备选择权限

**位置**: `napi_audio_routing_manager.cpp`

**描述**: `selectOutputDevice()`/`selectInputDevice()` 需要权限验证

**建议**: 确保普通应用不能强制切换系统音频设备

#### 风险 4: 麦克风状态泄露

**位置**: `napi_audio_manager.cpp`

**描述**: `isMicrophoneMute()` 可能泄露用户隐私状态

**建议**: 非系统应用可能需要模糊化返回结果

#### 风险 5: IPC 回调验证

**位置**: `services/audio_service/client/include/audio_manager_listener_stub_impl.cpp`

**描述**: 回调函数需要验证调用者身份，防止权限提升

**建议**: 确保回调仅处理来自有效客户端的请求

## 服务配置文件权限

**文件**: `services/audio_service/etc/audio_server.cfg`

**声明的权限**:
- `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE`
- `ohos.permission.REPORT_RESOURCE_SCHEDULE_EVENT`
- `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED`
- `ohos.permission.GET_SENSITIVE_PERMISSIONS`
- `ohos.permission.PERMISSION_USED_STATS`
- `ohos.permission.ACCESS_SERVICE_DM`
- `ohos.permission.MANAGE_BLUETOOTH`
- `ohos.permission.MANAGE_MEDIA_RESOURCES`
- `ohos.permission.PUBLISH_SYSTEM_COMMON_EVENT`

**证据来源**: `services/audio_service/etc/audio_server.cfg:65-101`

## 错误码

| 错误码 | 描述 |
|--------|------|
| `ERR_PERMISSION_DENIED` | 权限被拒绝 |
| `ERR_SYSTEM_PERMISSION_DENIED` | 需要系统级权限 |
| `PERMISSION_GRANTED` | 权限已授予 |
| `PERMISSION_DENIED` | 权限被拒绝 |
| `PERMISSION_UNKNOWN` | 权限状态未知 |

## 相关文档

- [架构设计](02_Architecture.md)
- [N-API 接口](03_NAPI.md)
- [构建系统](04_Build.md)
