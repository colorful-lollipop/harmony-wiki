# 对外 N-API (JavaScript API) 文档

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档提供 CastEngine 框架对外 N-API（JavaScript API）的完整参考，包括所有导出的接口、参数、错误码和使用示例。

## 模块注册

### 模块信息

- **模块名称**: `cast`
- **注册入口**: `interfaces/kits/js/src/native_module.cpp:33-59`
- **模块版本**: 1
- **注册函数**: `Init(env, exports)`

### 注册的类

1. `InitEnums(env, exports)` - 枚举类型导出
2. `NapiCastSessionManager::Init(env, exports)` - 会话管理器
3. `NapiCastSession::DefineCastSessionJSClass(env)` - 投屏会话类
4. `NapiStreamPlayer::DefineStreamPlayerJSClass(env)` - 流播放器类
5. `NapiMirrorPlayer::DefineMirrorPlayerJSClass(env)` - 镜像播放器类

**证据**: `interfaces/kits/js/src/native_module.cpp:36-40`

---

## API 清单

### 1. CastSessionManager API

**类名**: `CastSessionManager`
**导出位置**: `interfaces/kits/js/src/napi_cast_session_manager.cpp:51-70`

#### 方法列表

| 方法名 | 同步/异步 | 参数 | 返回值 | 错误码 | 文件位置 |
|-------|----------|------|--------|--------|----------|
| on(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - | native_module.cpp:54 |
| off(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - | napi_cast_session_manager.cpp:55 |
| startDiscovery(protocolType) | 异步 | protocolType: object | Promise\<void\> | 1003, 1006, 1002 | napi_cast_session_manager.cpp:56 |
| stopDiscovery() | 异步 | 无 | Promise\<void\> | 1003, 1002 | napi_cast_session_manager.cpp:57 |
| setDiscoverable(isEnable) | 异步 | isEnable: boolean | Promise\<void\> | 1003, 1002 | napi_cast_session_manager.cpp:58 |
| createCastSession(property) | 异步 | property: object | Promise\<CastSession\> | 1003, 1002 | napi_cast_session_manager.cpp:59 |
| release() | 异步 | 无 | Promise\<void\> | 1003, 1002 | napi_cast_session_manager.cpp:60 |

#### 事件类型

| 事件名 | 触发条件 | 说明 |
|-------|----------|------|
| serviceDie | 服务异常死亡 | CastEngine 服务进程崩溃或停止 |
| deviceFound | 发现到新设备 | 设备发现成功，触发回调 |
| sessionCreate | 会话创建完成 | createCastSession 成功完成 |
| deviceOffline | 设备离线 | 已连接设备断开连接 |

**证据**: `interfaces/kits/js/src/napi_cast_session_manager.cpp:42-49`

#### 参数说明

##### protocolType 对象

**位置**: `napi_cast_session_manager.cpp:86-96`

包含以下属性：
- `protocolType`: 协议类型（枚举值）

##### property 对象

**位置**: `napi_cast_session_manager.cpp:197-206`

包含会话配置属性。

#### 错误处理

所有异步方法都会检查以下错误码：

| 错误 | 值 | 说明 | 检查位置 |
|-----|------|------|----------|
| ERR_NO_PERMISSION | -1003 | 缺少必需权限 | napi_cast_session_manager.cpp:102 |
| ERR_INVALID_PARAM | -1002 | 参数无效 | napi_cast_session_manager.cpp:91, 105, 159, 169 |
| 其他 | -1 | 原生服务异常 | napi_cast_session_manager.cpp:107, 133, 175, 212, 251 |

**证据**: 所有方法在错误检查后设置 `NapiErrors::errcode_[ret]`

---

### 2. CastSession API

**类名**: `CastSession`
**导出位置**: `interfaces/kits/js/src/napi_cast_session.cpp:49-61`

#### 方法列表

| 方法名 | 同步/异步 | 参数 | 返回值 | 错误码 |
|-------|----------|------|--------|--------|
| on(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - |
| off(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - |
| addDevice(deviceInfo) | 异步 | deviceInfo: object | Promise\<void\> | 1003, 1006, 1002 |
| removeDevice(deviceId) | 异步 | deviceId: string | Promise\<void\> | 1003, 1006, 1002 |
| getSessionId() | 同步 | 无 | string | - |
| setSessionProperty(property) | 异步 | property: object | Promise\<void\> | 1003, 1002 |
| createMirrorPlayer() | 异步 | 无 | Promise\<MirrorPlayer\> | 1003, 1006 |
| createStreamPlayer() | 异步 | 无 | Promise\<StreamPlayer\> | 1003, 1006 |
| setCastMode(mode) | 异步 | mode: number | Promise\<void\> | 1003, 1002 |
| release() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| getRemoteDeviceInfo() | 同步 | 无 | object | - |

#### 事件类型

| 事件名 | 参数 | 说明 |
|-------|------|------|
| event | event: object | 通用事件 |
| deviceState | state: object | 设备状态变化 |

**证据**: `interfaces/kits/js/src/napi_cast_session.cpp:40-45`

#### 错误处理

| 错误 | 值 | 说明 |
|-----|------|------|
| ERR_NO_PERMISSION | -1003 | 缺少镜像或流权限 |
| ERR_SESSION_NOT_EXIST | -1004 | 会话不存在 |
| ERR_SESSION_STATE_NOT_MATCH | -1006 | 会话状态不匹配 |

---

### 3. StreamPlayer API

**类名**: `StreamPlayer`
**导出位置**: `interfaces/kits/js/src/napi_stream_player.cpp:57-83`

#### 方法列表

| 方法名 | 同步/异步 | 参数 | 返回值 | 错误码 |
|-------|----------|------|--------|--------|
| on(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - |
| off(eventType, callback) | 同步 | eventType: string, callback: function | undefined | - |
| setSurface(surface) | 同步 | surface: object | undefined | - |
| load(mediaInfo) | 异步 | mediaInfo: object | Promise\<void\> | 1003, 1006 |
| start() | 异步 | 无 | Promise\<void\> | 1003, 1002 |
| play() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| pause() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| stop() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| next() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| previous() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| seek(position) | 异步 | position: number | Promise\<void\> | 1003, 1006 |
| fastForward() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| fastRewind() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| setVolume(volume) | 异步 | volume: number | Promise\<void\> | 1003, 1006 |
| setLoopMode(mode) | 异步 | mode: number | Promise\<void\> | 1003, 1006 |
| setSpeed(speed) | 异步 | speed: number | Promise\<void\> | 1003, 1006 |
| setMute(mute) | 异步 | mute: boolean | Promise\<void\> | 1003, 1006 |
| getPlayerStatus() | 同步 | 无 | number | - |
| getPosition() | 同步 | 无 | number | - |
| getVolume() | 同步 | 无 | number | - |
| getMute() | 同步 | 无 | boolean | - |
| getLoopMode() | 同步 | 无 | number | - |
| getPlaySpeed() | 同步 | 无 | number | - |
| getMediaInfoHolder() | 同步 | 无 | object | - |
| release() | 异步 | 无 | Promise\<void\> | 1003, 1006 |

**证据**: `interfaces/kits/js/src/napi_stream_player.cpp:58-82`

#### 事件类型

| 事件名 | 回调参数 | 说明 |
|-------|----------|------|
| stateChanged | state: number | 播放状态变化 |
| positionChanged | position: number | 播放位置变化 |
| mediaItemChanged | item: object | 媒体项变化 |
| volumeChanged | volume: number | 音量变化 |
| videoSizeChanged | width: number, height: number | 视频尺寸变化 |
| loopModeChanged | mode: number | 循环模式变化 |
| playSpeedChanged | speed: number | 播放速度变化 |
| playerError | error: number | 播放器错误 |
| nextRequest | - | 下一首请求 |
| previousRequest | - | 上一首请求 |
| seekDone | position: number | 跳转完成 |
| endOfStream | - | 流结束 |
| imageChanged | image: object | 图像变化 |

**证据**: `interfaces/kits/js/src/napi_stream_player.cpp:37-53`

#### 错误码映射

StreamPlayer 使用统一的错误码系统：

| 错误类别 | 基础码 | 示例 |
|----------|--------|------|
| 播放器错误 | 1100-1199 | 1100: 播放器错误 |
| IO 错误 | 2000-2199 | 2001: 网络连接失败 |
| 解析错误 | 3000-3099 | 3001: 容器格式错误 |
| 解码错误 | 4000-4099 | 4001: 解码器初始化失败 |
| DRM 错误 | 6000-6199 | 6001: 不支持的 DRM 方案 |

**完整定义**: `interfaces/inner_api/include/cast_engine_errors.h:28-110`

---

### 4. MirrorPlayer API

**类名**: `MirrorPlayer`
**导出位置**: `interfaces/kits/js/src/napi_mirror_player.cpp:43-52`

#### 方法列表

| 方法名 | 同步/异步 | 参数 | 返回值 | 错误码 |
|-------|----------|------|--------|--------|
| play() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| pause() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| setAppInfo(appInfo) | 异步 | appInfo: object | Promise\<void\> | 1003, 1002 |
| setSurface(surface) | 同步 | surface: object | undefined | - |
| release() | 异步 | 无 | Promise\<void\> | 1003, 1006 |
| resizeVirtualScreen(width, height) | 异步 | width: number, height: number | Promise\<void\> | 1003, 1002 |
| setCastRoute(route) | 异步 | route: object | Promise\<void\> | 1003, 1002 |
| getScreenshot() | 异步 | 无 | Promise\<object\> | 1003, 1006 |

**证据**: `interfaces/kits/js/src/napi_mirror_player.cpp:44-51`

#### 错误处理

MirrorPlayer 使用统一的权限错误码：

| 错误 | 值 | 检查位置 |
|-----|------|----------|
| ERR_NO_PERMISSION | -1003 | 缺少镜像权限 | napi_mirror_player.cpp:155, 201, 247 |
| ERR_INVALID_PARAM | -1002 | 参数无效 | napi_mirror_player.cpp:298, 331, 379, 428 |
| ERR_SERVICE_STATE_NOT_MATCH | -1005 | 服务状态不匹配 | napi_mirror_player.cpp:1655 |

---

## 枚举和常量

### 设备类型

**枚举**: `DeviceType`
**位置**: `interfaces/inner_api/include/cast_engine_common.h:33-48`

**N-API 导出**: `interfaces/kits/js/src/napi_castengine_enum.cpp`

| 值 | 名称 | 说明 |
|-----|------|------|
| 0 | DEVICE_OTHERS | 其他设备 |
| 1 | DEVICE_SCREEN_PLAYER | 屏幕播放器 |
| 2 | DEVICE_HW_TV | 华为电视 |
| 3 | DEVICE_SOUND_BOX | 音箱 |
| 4 | DEVICE_HICAR | HiCar |
| 5 | DEVICE_MATEBOOK | MateBook |
| 6 | DEVICE_PAD | 平板 |
| 7 | DEVICE_CAST_PLUS | Cast+ |
| 13 | DEVICE_MIRACAST | Miracast |

### 设备状态

**枚举**: `DeviceState`
**位置**: `interfaces/inner_api/include/cast_engine_common.h:86-101`

| 值 | 名称 | 说明 |
|-----|------|------|
| 0 | CONNECTING | 连接中 |
| 1 | CONNECTED | 已连接 |
| 2 | PAUSED | 已暂停 |
| 3 | PLAYING | 播放中 |
| 4 | DISCONNECTING | 断开连接中 |
| 5 | DISCONNECTED | 已断开 |

### 错误码

完整的错误码列表见：[错误码定义](../_work/NOTES.md#错误码定义)

**基础码**:
- 成功: `CAST_ENGINE_SUCCESS = 0`
- 错误基础: `CAST_ENGINE_ERROR_BASE = 1000`

**类别**:
- 通用错误: 1000-1007
- 播放器错误: 1100-1199
- IO 错误: 2000-2199
- 解析错误: 3000-3099
- 解码错误: 4000-4099
- DRM 错误: 6000-6199

---

## 参数校验

### 通用校验规则

所有 N-API 方法都进行参数类型校验：

**类型检查**:
```cpp
napi_valuetype expectedTypes[expectedArgc] = { napi_object, napi_function, ... };
bool isParamsTypeValid = CheckJSParamsType(env, argv, expectedArgc, expectedTypes);
```

**位置**: `interfaces/kits/js/src/napi_castengine_utils.cpp`

**检查项**:
- 参数数量
- 参数类型（null、number、string、object、function）
- 字符串长度（有上限检查）
- 数值范围（有下限和上限）

### 权限校验

所有需要权限的操作都会在服务端检查权限：

**权限类型**:
- `ACCESS_CAST_ENGINE_MIRROR` - 镜像投屏权限
- `ACCESS_CAST_ENGINE_STREAM` - 流播放权限

**权限检查实现**:
```cpp
// location: service/src/session/src/utils/src/permission.cpp:75-83
bool Permission::CheckMirrorPermission() {
    return CheckPermission(MIRROR_PERMISSION);
}
```

**权限常量**:
```cpp
const std::string MIRROR_PERMISSION = "ohos.permission.ACCESS_CAST_ENGINE_MIRROR";
const std::string STREAM_PERMISSION = "ohos.permission.ACCESS_CAST_ENGINE_STREAM";
```

**位置**: `service/src/session/src/utils/src/permission.cpp:39-40`

---

## 异步处理

### 异步工作队列

所有耗时操作都使用异步工作队列：

**实现**: `interfaces/kits/js/src/napi_async_work.cpp`

**流程**:
1. JavaScript 主线程接收调用
2. 将任务推送到工作队列
3. 在工作线程执行 C++ 实现
4. 通过回调返回结果到主线程

**证据**: 所有异步方法都调用 `NapiAsyncWork::Enqueue(env, task, "MethodName", executor, complete)`

**位置示例**: `interfaces/kits/js/src/napi_cast_session_manager.cpp:115`

---

## 使用示例

### 示例 1: 设备发现和投屏

```javascript
import cast from '@ohos.cast';

// 1. 开始设备发现
const castSessionManager = new cast.CastSessionManager();
await castSessionManager.startDiscovery(cast.ProtocolType.WIFI_DISPLAY);

// 2. 监听设备发现事件
castSessionManager.on('deviceFound', (device) => {
    console.log('Found device:', device);
});

// 3. 创建投屏会话
const session = await castSessionManager.createCastSession({
    deviceId: device.deviceId
});

// 4. 创建镜像播放器
const mirrorPlayer = await session.createMirrorPlayer();
await mirrorPlayer.play();

// 5. 清理
await mirrorPlayer.release();
await session.release();
await castSessionManager.stopDiscovery();
```

### 示例 2: 流媒体播放

```javascript
import cast from '@ohos.cast';

// 1. 创建会话和流播放器
const castSessionManager = new cast.CastSessionManager();
const session = await castSessionManager.createCastSession({});
const streamPlayer = await session.createStreamPlayer();

// 2. 监听播放器事件
streamPlayer.on('stateChanged', (state) => {
    console.log('Player state:', state);
});

// 3. 加载并播放媒体
await streamPlayer.setSurface(surface);
await streamPlayer.load({ url: 'http://example.com/video.mp4' });
await streamPlayer.play();

// 4. 控制播放
await streamPlayer.pause();
await streamPlayer.play();
await streamPlayer.setVolume(0.5);

// 5. 清理
await streamPlayer.release();
await session.release();
```

---

## 权限要求

使用 CastEngine N-API 需要在应用清单中声明以下权限：

| 权限名称 | 用途 | 说明 |
|-----------|------|------|
| ohos.permission.ACCESS_CAST_ENGINE_MIRROR | 镜像投屏 | 创建和使用镜像播放器 |
| ohos.permission.ACCESS_CAST_ENGINE_STREAM | 流播放 | 创建和使用流播放器 |

**证据**: `service/src/session/src/utils/src/permission.cpp:39-40`

---

## 相关文档

- [内部 API](04_Internal_API.md) - N-API 调用的 C++ 接口
- [错误码定义](../_work/NOTES.md#错误码定义) - 完整的错误码列表
- [安全评审](07_Security_Review.md) - 权限和安全机制
- [常见问题](08_QA.md) - N-API 使用常见问题

---

**返回**: [SUMMARY.md](SUMMARY.md)
