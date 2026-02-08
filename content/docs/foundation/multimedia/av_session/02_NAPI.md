# N-API 接口文档

## 模块注册

### 主模块入口

**文件**: `frameworks/js/napi/session/src/napi_module.cpp:55`

```cpp
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&module);
}
```

**模块名**: `multimedia.avsession`

### 导出模块

| 模块 | 注册函数 | 功能 |
|------|----------|------|
| `InitEnums` | `napi_avsession_enum.cpp` | 导出枚举常量 |
| `NapiAVSessionManager::Init` | `napi_avsession_manager.cpp` | 会话管理器 |
| `NapiAVSession::Init` | `napi_avsession.cpp` | 会话对象 |
| `NapiAVSessionController::Init` | `napi_avsession_controller.cpp` | 控制器 |
| `NapiAVCastPickerHelper::Init` | `napi_avcast_picker_helper.cpp` | 投屏选择器 |
| `NapiAVCastController::Init` | `napi_avcast_controller.cpp` | 投屏控制器 |

### 其他 N-API 模块

| 模块名 | 文件路径 | 功能 |
|--------|----------|------|
| `multimedia.avVolumePanel` | `avvolumepanel/avvolumepanel.cpp` | 音量面板 |
| `multimedia.avCastPicker` | `avpicker/avpicker.cpp` | 投屏选择器 |
| `multimedia.avCastPickerParam` | `avpicker/avpickerparam.cpp` | 投屏参数 |
| `multimedia.avInputCastPicker` | `avinputcastpicker/avinputcastpicker.cpp` | 输入投屏 |

---

## AVSessionManager API

### 静态方法列表

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `createAVSession` | tag: string, type: number, elementName: ElementName | Promise\<AVSession\> | 创建会话 |
| `getAVSession` | sessionId: string | Promise\<AVSession\> | 获取会话 |
| `getAllSessionDescriptors` | - | Promise\<Array\<AVSessionDescriptor\>\> | 获取所有会话 |
| `getSessionDescriptors` | category: SessionCategory | Promise\<Array\<AVSessionDescriptor\>\> | 按类别获取 |
| `getHistoricalSessionDescriptors` | maxSize: number | Promise\<Array\<AVSessionDescriptor\>\> | 获取历史会话 |
| `getHistoricalAVQueueInfos` | maxSize: number, maxAppSize: number | Promise\<Array\<AVQueueInfo\>\> | 获取历史队列 |
| `startAVPlayback` | bundleName: string, assetId: string | Promise\<number\> | 启动播放 |
| `createController` | sessionId: string | Promise\<AVSessionController\> | 创建控制器 |
| `getAVCastController` | sessionId: string | Promise\<AVCastController\> | 获取投屏控制器 |
| `castAudio` | token: SessionToken, descriptors: Array\<AudioDeviceDescriptor\> | Promise\<number\> | 音频投送 |
| `startCasting` | sessionToken: SessionToken, outputDevice: OutputDeviceInfo | Promise\<number\> | 开始投屏 |
| `stopCasting` | sessionToken: SessionToken | Promise\<number\> | 停止投屏 |
| `startCastDeviceDiscovery` | - | Promise\<number\> | 开始设备发现 |
| `stopCastDeviceDiscovery` | - | Promise\<number\> | 停止设备发现 |
| `sendSystemAVKeyEvent` | keyEvent: KeyEvent | Promise\<number\> | 发送系统按键 |
| `sendSystemControlCommand` | command: AVControlCommand | Promise\<number\> | 发送系统命令 |
| `isDesktopLyricSupported` | - | Promise\<boolean\> | 是否支持桌面歌词 |

### 事件监听

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `on('sessionCreate')` | (sessionDescriptor: AVSessionDescriptor) => void | 会话创建 |
| `on('sessionDestroy')` | (sessionId: string) => void | 会话销毁 |
| `on('topSessionChange')` | (sessionDescriptor: AVSessionDescriptor) => void | 顶层会话变化 |
| `on('activeSessionChange')` | (sessionDescriptors: Array\<AVSessionDescriptor\>) => void | 活动会话变化 |
| `on('deviceAvailable')` | (deviceInfo: OutputDeviceInfo) => void | 设备可用 |
| `on('deviceOffline')` | (deviceId: string) => void | 设备离线 |
| `on('deviceLogEvent')` | (event: DeviceLogEvent) => void | 设备日志事件 |

---

## AVSession API

### 实例方法列表

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setAVMetadata` | metadata: AVMetaData | Promise\<void\> | 设置元数据 |
| `setCallMetadata` | metadata: AVCallMetaData | Promise\<void\> | 设置通话元数据 |
| `setAVPlaybackState` | state: AVPlaybackState | Promise\<void\> | 设置播放状态 |
| `setAVCallState` | state: AVCallState | Promise\<void\> | 设置通话状态 |
| `setLaunchAbility` | ability: AbilityConstant.LaunchParam | Promise\<void\> | 设置启动能力 |
| `setExtras` | extras: {[key: string]: Object} | Promise\<void\> | 设置扩展信息 |
| `activate` | - | Promise\<void\> | 激活会话 |
| `deactivate` | - | Promise\<void\> | 停用会话 |
| `destroy` | - | Promise\<void\> | 销毁会话 |
| `getOutputDevice` | - | Promise\<OutputDeviceInfo\> | 获取输出设备 |
| `getController` | - | Promise\<AVSessionController\> | 获取控制器 |
| `dispatchSessionEvent` | event: string, extras: {[key: string]: Object} | Promise\<void\> | 分发会话事件 |
| `setAVQueueItems` | items: Array\<AVQueueItem\> | Promise\<void\> | 设置播放队列 |
| `setAVQueueTitle` | title: string | Promise\<void\> | 设置队列标题 |
| `sendCustomData` | code: number, data: {[key: string]: Object} | Promise\<void\> | 发送自定义数据 |
| `enableDesktopLyric` | enabled: boolean | Promise\<void\> | 启用桌面歌词 |
| `setDesktopLyricVisible` | visible: boolean | Promise\<void\> | 设置歌词可见性 |
| `getAllCastDisplays` | - | Promise\<Array\<CastDisplayInfo\>\> | 获取投屏显示 |

### 支持的事件类型

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `on('play')` | - | 播放 |
| `on('pause')` | - | 暂停 |
| `on('stop')` | - | 停止 |
| `on('playNext')` | - | 下一首 |
| `on('playPrevious')` | - | 上一首 |
| `on('fastForward')` | time?: number | 快进 |
| `on('rewind')` | time?: number | 快退 |
| `on('seek')` | time: number | 跳转 |
| `on('setSpeed')` | speed: number | 设置速度 |
| `on('setLoopMode')` | mode: LoopMode | 设置循环模式 |
| `on('toggleFavorite')` | assetId: string | 收藏 |
| `on('handleKeyEvent')` | keyEvent: KeyEvent | 处理按键 |
| `on('outputDeviceChange')` | state: number, device: OutputDeviceInfo | 设备变化 |
| `on('commonCommand')` | command: string, args: {[key: string]: Object} | 通用命令 |
| `on('skipToQueueItem')` | itemId: number | 跳转到队列项 |
| `on('answer')` | - | 接听 |
| `on('hangUp')` | - | 挂断 |
| `on('toggleCallMute')` | - | 切换静音 |

---

## AVSessionController API

### 实例方法列表

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getAVPlaybackState` | - | Promise\<AVPlaybackState\> | 获取播放状态 |
| `getAVMetadata` | - | Promise\<AVMetaData\> | 获取元数据 |
| `getAVCallState` | - | Promise\<AVCallState\> | 获取通话状态 |
| `getCallMetadata` | - | Promise\<AVCallMetaData\> | 获取通话元数据 |
| `getOutputDevice` | - | Promise\<OutputDeviceInfo\> | 获取输出设备 |
| `sendAVKeyEvent` | keyEvent: KeyEvent | Promise\<void\> | 发送按键事件 |
| `sendControlCommand` | command: AVControlCommand | Promise\<void\> | 发送控制命令 |
| `sendCommonCommand` | command: string, args: {[key: string]: Object} | Promise\<void\> | 发送通用命令 |
| `isActive` | - | Promise\<boolean\> | 是否激活 |
| `getValidCommands` | - | Promise\<Array\<AVControlCommandType\>\> | 获取有效命令 |
| `getLaunchAbility` | - | Promise\<AbilityConstant.LaunchParam\> | 获取启动能力 |
| `getRealPlaybackPosition` | - | Promise\<number\> | 获取播放位置 |
| `getAVQueueItems` | - | Promise\<Array\<AVQueueItem\>\> | 获取队列 |
| `skipToQueueItem` | itemId: number | Promise\<void\> | 跳转到项 |
| `getExtras` | - | Promise\<{[key: string]: Object}\> | 获取扩展信息 |
| `destroy` | - | Promise\<void\> | 销毁控制器 |

### 支持的事件类型

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `on('metadataChange')` | data: AVMetaData | 元数据变化 |
| `on('playbackStateChange')` | state: AVPlaybackState | 播放状态变化 |
| `on('activeStateChange')` | isActive: boolean | 激活状态变化 |
| `on('validCommandChange')` | commands: Array\<AVControlCommandType\> | 有效命令变化 |
| `on('queueItemsChange')` | items: Array\<AVQueueItem\> | 队列变化 |
| `on('queueTitleChange')` | title: string | 队列标题变化 |
| `on('sessionDestroy')` | - | 会话销毁 |
| `on('outputDeviceChange')` | state: number, device: OutputDeviceInfo | 设备变化 |
| `on('extrasChange')` | extras: {[key: string]: Object} | 扩展信息变化 |

---

## AVCastController API

### 实例方法列表

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `start` | item: AVQueueItem | Promise\<void\> | 开始播放 |
| `prepare` | item: AVQueueItem | Promise\<void\> | 准备播放 |
| `sendControlCommand` | command: AVCastControlCommand | Promise\<void\> | 发送控制命令 |
| `sendCustomData` | code: number, data: {[key: string]: Object} | Promise\<void\> | 发送自定义数据 |
| `getDuration` | - | Promise\<number\> | 获取时长 |
| `getAVPlaybackState` | - | Promise\<AVPlaybackState\> | 获取播放状态 |
| `getCurrentItem` | - | Promise\<AVQueueItem\> | 获取当前项 |
| `getValidCommands` | - | Promise\<Array\<AVCastControlCommandType\>\> | 获取有效命令 |
| `release` | - | Promise\<void\> | 释放控制器 |
| `setDisplaySurface` | surface: image.PixelMap | Promise\<void\> | 设置显示表面 |
| `processMediaKeyResponse` | assetId: string, response: {[key: string]: Object} | Promise\<void\> | 处理密钥响应 |

### 支持的事件类型

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `on('playbackStateChange')` | state: AVPlaybackState | 播放状态变化 |
| `on('mediaItemChange')` | item: AVQueueItem | 媒体项变化 |
| `on('playNext')` | - | 下一首 |
| `on('playPrevious')` | - | 上一首 |
| `on('seekDone')` | time: number | 跳转完成 |
| `on('videoSizeChange')` | width: number, height: number | 视频尺寸变化 |
| `on('error')` | code: number, data: Object | 错误 |
| `on('endOfStream')` | isPlayed: boolean | 播放结束 |

---

## AVCastPickerHelper API

### 实例方法列表

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `select` | - | Promise\<AVCastPickerResult\> | 显示选择器 |
| `resetCommunicationDevice` | deviceId: string, type: number | Promise\<void\> | 重置设备 |
| `on('pickerStateChange')` | state: number | 状态变化监听 |

---

## 枚举常量

### 会话类型 (AVSessionType)

```javascript
AVSessionManager.AVSESSION_TYPE_AUDIO = 0;       // 音频
AVSessionManager.AVSESSION_TYPE_VIDEO = 1;       // 视频
AVSessionManager.AVSESSION_TYPE_VOICE_CALL = 2;  // 语音通话
AVSessionManager.AVSESSION_TYPE_VIDEO_CALL = 3; // 视频通话
AVSessionManager.AVSESSION_TYPE_PHOTO = 4;      // 相册
```

### 播放状态 (PlaybackState)

```javascript
AVSessionManager.PLAYBACK_STATE_INITIAL = 0;     // 初始
AVSessionManager.PLAYBACK_STATE_PREPARE = 1;    // 准备
AVSessionManager.PLAYBACK_STATE_PLAY = 2;       // 播放
AVSessionManager.PLAYBACK_STATE_PAUSE = 3;      // 暂停
AVSessionManager.PLAYBACK_STATE_FAST_FORWARD = 4; // 快进
AVSessionManager.PLAYBACK_STATE_REWIND = 5;     // 快退
AVSessionManager.PLAYBACK_STATE_STOP = 6;        // 停止
AVSessionManager.PLAYBACK_STATE_COMPLETED = 7;   // 完成
AVSessionManager.PLAYBACK_STATE_RELEASED = 8;    // 释放
AVSessionManager.PLAYBACK_STATE_ERROR = 9;       // 错误
AVSessionManager.PLAYBACK_STATE_IDLE = 10;       // 空闲
AVSessionManager.PLAYBACK_STATE_BUFFERING = 11;  // 缓冲
```

### 循环模式 (LoopMode)

```javascript
AVSessionManager.LOOP_MODE_SEQUENCE = 1;   // 顺序播放
AVSessionManager.LOOP_MODE_SINGLE = 2;     // 单曲循环
AVSessionManager.LOOP_MODE_LIST = 3;       // 列表循环
AVSessionManager.LOOP_MODE_SHUFFLE = 4;    // 随机播放
AVSessionManager.LOOP_MODE_CUSTOM = 5;     // 自定义
```

### 投屏类别 (AVCastCategory)

```javascript
AVSessionManager.CATEGORY_LOCAL = 0;        // 本地
AVSessionManager.CATEGORY_REMOTE = 1;      // 远端
```

### 连接状态 (ConnectionState)

```javascript
AVSessionManager.STATE_CONNECTING = 0;     // 连接中
AVSessionManager.STATE_CONNECTED = 1;      // 已连接
AVSessionManager.STATE_DISCONNECTED = 2;   // 已断开
```

### 设备类型 (DeviceType)

```javascript
AVSessionManager.DEVICE_TYPE_LOCAL = 0;       // 本地
AVSessionManager.DEVICE_TYPE_TV = 1;          // 电视
AVSessionManager.DEVICE_TYPE_SMART_SPEAKER = 2; // 智能音箱
AVSessionManager.DEVICE_TYPE_BLUETOOTH = 3;    // 蓝牙设备
```

### 错误码 (AVSessionErrorCode)

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 6600101 | 会话不存在 |
| 6600102 | 会话已激活 |
| 6600103 | 会话未激活 |
| 6600104 | 会话已销毁 |
| 6600105 | 控制器不存在 |
| 6600106 | 控制器已销毁 |
