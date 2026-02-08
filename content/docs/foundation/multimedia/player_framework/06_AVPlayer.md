# 播放器 API

## AudioPlayer

### 模块信息

- **命名空间**: ohos.media
- **实现文件**: `frameworks/js/player/audio_player_napi.cpp`
- **注册入口**: `AudioPlayerNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| constructor | 创建实例 | - | AudioPlayer |
| setSource | 设置数据源 | string: fd/url | number: 错误码 |
| play | 开始播放 | - | number: 错误码 |
| pause | 暂停播放 | - | number: 错误码 |
| stop | 停止播放 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |
| getCurrentTime | 获取当前播放位置 | - | number: 当前位置 |
| getDuration | 获取媒体时长 | - | number: 时长(ms) |
| setVolume | 设置音量 | number: 0.0-1.0 | number: 错误码 |

### 权限要求

| 方法 | 权限 |
|------|-----|
| play | 无 |
| setSource (网络) | ohos.permission.INTERNET |

### 代码示例

```javascript
import AudioPlayer from '@ohos.multimedia.audioPlayer';

let player = new AudioPlayer();
player.setSource('/data/audio/test.mp3');
player.play();
```

## VideoPlayer

### 模块信息

- **命名空间**: ohos.media
- **实现文件**: `frameworks/js/player/video_player_napi.cpp`
- **注册入口**: `VideoPlayerNapi::Init(env, exports)`

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| constructor | 创建实例 | - | VideoPlayer |
| setSource | 设置数据源 | string: fd/url | number: 错误码 |
| setDisplaySurface | 设置显示surface | surfaceId | number: 错误码 |
| play | 开始播放 | - | number: 错误码 |
| pause | 暂停播放 | - | number: 错误码 |
| stop | 停止播放 | - | number: 错误码 |
| release | 释放资源 | - | number: 错误码 |

### 权限要求

| 方法 | 权限 |
|------|-----|
| setSource (网络) | ohos.permission.INTERNET |

## AVPlayer

### 模块信息

- **命名空间**: ohos.multimedia.avplayer
- **实现文件**: `frameworks/js/avplayer/avplayer_napi.cpp`
- **注册入口**: `AVPlayerNapi::Init(env, exports)`
- **支持版本**: API9+

### API 列表

| 方法 | 描述 | 参数 | 返回值 |
|------|------|-----|--------|
| prepare | 准备播放 | - | Promise\<void\> |
| play | 开始播放 | - | Promise\<void\> |
| pause | 暂停播放 | - | Promise\<void\> |
| stop | 停止播放 | - | Promise\<void\> |
| reset | 重置播放器 | - | Promise\<void\> |
| release | 释放资源 | - | Promise\<void\> |
| getCurrentTime | 获取当前时间 | - | Promise\<number\> |
| seek | 跳转到指定位置 | number: ms | Promise\<void\> |
| setVolume | 设置音量 | number: 0.0-1.0 | Promise\<void\> |

### 错误码

| 错误码 | 含义 |
|--------|------|
| 0 | 成功 |
| 401 | 无效参数 |
| 5400102 | 操作失败 |
| 5400103 | 无权限 |

### 权限检查点

```cpp
// 文件: frameworks/js/avplayer/avplayer_napi.cpp:231
uint64_t tokenId = IPCSkeleton::GetSelfTokenID();
```

## 相关文档

- [N-API 接口总览](05_NAPI_Overview.md)
- [录制器 API](07_AVRecorder.md)
