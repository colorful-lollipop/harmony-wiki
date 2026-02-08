# 04_External_API - 对外 API

> 本文档详细说明 Cast+ Stream 模块的对外 IPC API，包括接口清单、调用方式和权限要求。

---

## 1. API 概述

### 1.1 接口类型

本模块为纯原生 C++ 模块，通过 **OpenHarmony IPC 机制** 对外提供服务。接口采用 **Stub/Proxy 模式** 实现：

- **Stub 端**（服务端）：`CastSessionImplStub`、`MirrorPlayerImplStub`、`StreamPlayerImplStub`
- **Proxy 端**（客户端）：在父级框架 `castengine_cast_framework` 中实现

### 1.2 接口层次

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           对外接口层次                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    Application (应用层)                              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                  Framework Layer (N-API 层)                          │    │
│  │              (castengine_cast_framework)                            │    │
│  │                                                                     │    │
│  │  本层提供 JS/ArkTS 接口，调用下方的 IPC 接口                            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    IPC Interface (本模块)                            │    │
│  │                                                                     │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │    │
│  │  │ ICastSession │  │ IMirrorPlayer│  │ IStreamPlayer│                 │    │
│  │  │   Impl      │  │   Impl      │  │   Impl      │                 │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                 │    │
│  │                                                                     │    │
│  │  注: 接口定义在父级框架中，本模块提供 Stub 实现                        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. ICastSessionImpl 接口

### 2.1 接口说明

| 属性 | 说明 |
|------|------|
| **接口定义** | `i_cast_session_impl.h`（父级框架） |
| **Stub 实现** | `CastSessionImplStub` |
| **Stub 文件** | `include/cast_session_impl_stub.h` |
| **通信方式** | 同步 IPC |

### 2.2 方法清单

| IPC Code | 方法名 | 说明 | 权限要求 |
|----------|--------|------|----------|
| `REGISTER_LISTENER` | `RegisterListener()` | 注册会话监听器 | 无 |
| `UNREGISTER_LISTENER` | `UnregisterListener()` | 注销会话监听器 | 无 |
| `ADD_DEVICE` | `AddDevice()` | 添加远程设备 | Mirror 或 Stream 权限 |
| `REMOVE_DEVICE` | `RemoveDevice()` | 移除远程设备 | Mirror 或 Stream 权限 |
| `START_AUTH` | `StartAuth()` | 启动认证 | Mirror 或 Stream 权限 |
| `GET_SESSION_ID` | `GetSessionId()` | 获取会话 ID | Mirror 或 Stream 权限 |
| `GET_DEVICE_STATE` | `GetDeviceState()` | 获取设备状态 | Mirror 或 Stream 权限 |
| `SET_SESSION_PROPERTY` | `SetSessionProperty()` | 设置会话属性 | Mirror 或 Stream 权限 |
| `CREAT_MIRROR_PLAYER` | `CreateMirrorPlayer()` | 创建镜像播放器 | Mirror 权限 |
| `CREAT_STREAM_PLAYER` | `CreateStreamPlayer()` | 创建流媒体播放器 | Stream 权限 |
| `NOTIFY_EVENT` | `NotifyEvent()` | 通知事件 | 无 |
| `SET_CAST_MODE` | `SetCastMode()` | 设置投屏模式 | Mirror 权限 |
| `RELEASE` | `Release()` | 释放会话 | Mirror 或 Stream 权限 |

### 2.3 详细接口定义

#### RegisterListener
```cpp
// 请求
struct Request {
    IRemoteObject listener;  // 监听器对象（代理）
};

// 响应
struct Response {
    int32_t result;  // 0=成功，其他=错误码
};

// 权限: 无
// 说明: 注册客户端监听器，用于接收会话事件回调
```

#### AddDevice
```cpp
// 请求
struct Request {
    CastRemoteDevice device;  // 远程设备信息
};

// 响应
struct Response {
    int32_t result;  // 0=成功，其他=错误码
};

// 权限: ohos.permission.ACCESS_CAST_ENGINE_MIRROR 或
//       ohos.permission.ACCESS_CAST_ENGINE_STREAM
// 说明: 添加远程设备到会话，触发连接流程
```

#### CreateMirrorPlayer
```cpp
// 请求
struct Request {
    // 无参数
};

// 响应
struct Response {
    int32_t result;           // 0=成功
    IRemoteObject player;     // 镜像播放器 Stub 对象
};

// 权限: ohos.permission.ACCESS_CAST_ENGINE_MIRROR
// 说明: 创建镜像播放器，返回播放器 IPC 对象
```

#### CreateStreamPlayer
```cpp
// 请求
struct Request {
    // 无参数
};

// 响应
struct Response {
    int32_t result;           // 0=成功
    IRemoteObject player;     // 流媒体播放器 Stub 对象
};

// 权限: ohos.permission.ACCESS_CAST_ENGINE_STREAM
// 说明: 创建流媒体播放器，返回播放器 IPC 对象
```

---

## 3. ICastSessionListenerImpl 回调接口

### 3.1 接口说明

| 属性 | 说明 |
|------|------|
| **接口定义** | `i_cast_session_listener_impl.h`（父级框架） |
| **Proxy 实现** | `CastSessionListenerImplProxy` |
| **Proxy 文件** | `include/cast_session_listener_impl_proxy.h` |
| **通信方式** | 异步 IPC |

### 3.2 回调方法清单

| IPC Code | 方法名 | 说明 | 触发时机 |
|----------|--------|------|----------|
| `ON_DEVICE_STATE` | `OnDeviceState()` | 设备状态变化 | 设备连接/断开/状态变更 |
| `ON_EVENT` | `OnEvent()` | 通用事件 | 会话事件（错误、警告等） |
| `ON_REMOTE_CTRL_EVENT` | `OnRemoteCtrlEvent()` | 远程控制事件 | Sink 端输入事件 |

### 3.3 详细回调定义

#### OnDeviceState
```cpp
// 通知
struct Notification {
    DeviceStateInfo stateInfo;  // 设备状态信息
    // - deviceId: 设备 ID
    // - state: 设备状态 (DISCONNECTED/CONNECTING/CONNECTED/PLAYING/PAUSED)
    // - eventCode: 事件码
};

// 通信方式: 异步 (TF_ASYNC)
// 说明: 当设备状态发生变化时通知客户端
```

#### OnEvent
```cpp
// 通知
struct Notification {
    EventId eventId;      // 事件 ID
    string jsonParam;     // JSON 格式参数
};

// 通信方式: 异步 (TF_ASYNC)
// 说明: 通用事件通知，如错误、警告、能力协商结果等
```

#### OnRemoteCtrlEvent
```cpp
// 通知
struct Notification {
    int32_t eventType;    // 事件类型
    uint8_t[] data;       // 事件数据
    uint32_t len;         // 数据长度
};

// 通信方式: 异步
// 说明: Sink 端的用户输入事件（触摸、按键等）回传到 Source
```

---

## 4. IMirrorPlayerImpl 接口

### 4.1 接口说明

| 属性 | 说明 |
|------|------|
| **接口定义** | `i_mirror_player_impl.h`（父级框架） |
| **Stub 实现** | `MirrorPlayerImplStub` |
| **Stub 文件** | `src/mirror/include/mirror_player_impl_stub.h` |
| **通信方式** | 同步 IPC |

### 4.2 方法清单

| IPC Code | 方法名 | 说明 | 权限要求 |
|----------|--------|------|----------|
| `PLAY` | `Play()` | 开始镜像播放 | Mirror 权限 |
| `PAUSE` | `Pause()` | 暂停镜像播放 | Mirror 权限 |
| `SET_SURFACE` | `SetSurface()` | 设置渲染 Surface | Mirror 权限 |
| `DELIVER_INPUT_EVENT` | `DeliverInputEvent()` | 传递输入事件 | Mirror 权限 |
| `INJECT_EVENT` | `InjectEvent()` | 注入事件 | Mirror 权限 |
| `SET_APP_INFO` | `SetAppInfo()` | 设置应用信息 | Mirror 权限 |
| `GET_DISPLAYID` | `GetDisplayId()` | 获取显示 ID | Mirror 权限 |
| `RESIZE_VIRTUAL_SCREEN` | `ResizeVirtualScreen()` | 调整虚拟屏幕 | Mirror 权限 |
| `RELEASE` | `Release()` | 释放播放器 | Mirror 权限 |

---

## 5. IStreamPlayerIpc 接口

### 5.1 接口说明

| 属性 | 说明 |
|------|------|
| **接口定义** | `i_stream_player_ipc.h`（父级框架） |
| **Stub 实现** | `StreamPlayerImplStub` |
| **Stub 文件** | `src/stream/src/player/include/stream_player_impl_stub.h` |
| **通信方式** | 同步 IPC |

### 5.2 方法清单

| IPC Code | 方法名 | 说明 | 权限要求 |
|----------|--------|------|----------|
| `REGISTER_LISTENER` | `RegisterListener()` | 注册播放器监听器 | Stream 权限 |
| `UNREGISTER_LISTENER` | `UnregisterListener()` | 注销监听器 | Stream 权限 |
| `SET_SURFACE` | `SetSurface()` | 设置渲染 Surface | Stream 权限 |
| `LOAD` | `Load()` | 加载媒体 | Stream 权限 |
| `START` | `Start()` | 开始播放 | Stream 权限 |
| `PLAY` | `Play()` | 播放 | Stream 权限 |
| `PAUSE` | `Pause()` | 暂停 | Stream 权限 |
| `STOP` | `Stop()` | 停止 | Stream 权限 |
| `NEXT` | `Next()` | 下一曲 | Stream 权限 |
| `PREVIOUS` | `Previous()` | 上一曲 | Stream 权限 |
| `SEEK` | `Seek()` |  seek | Stream 权限 |
| `FAST_FORWARD` | `FastForward()` | 快进 | Stream 权限 |
| `FAST_REWIND` | `FastRewind()` | 快退 | Stream 权限 |
| `SET_VOLUME` | `SetVolume()` | 设置音量 | Stream 权限 |
| `SET_MUTE` | `SetMute()` | 设置静音 | Stream 权限 |
| `SET_LOOP_MODE` | `SetLoopMode()` | 设置循环模式 | Stream 权限 |
| `SET_SPEED` | `SetSpeed()` | 设置播放速度 | Stream 权限 |
| `GET_PLAYER_STATUS` | `GetPlayerStatus()` | 获取播放状态 | Stream 权限 |
| `GET_POSITION` | `GetPosition()` | 获取播放位置 | Stream 权限 |
| `GET_DURATION` | `GetDuration()` | 获取总时长 | Stream 权限 |
| `GET_VOLUME` | `GetVolume()` | 获取音量 | Stream 权限 |
| `RELEASE` | `Release()` | 释放播放器 | Stream 权限 |

---

## 6. 权限说明

### 6.1 系统权限

| 权限 | 权限名 | 说明 |
|------|--------|------|
| **Mirror 权限** | `ohos.permission.ACCESS_CAST_ENGINE_MIRROR` | 镜像投屏权限 |
| **Stream 权限** | `ohos.permission.ACCESS_CAST_ENGINE_STREAM` | 流媒体投屏权限 |

### 6.2 权限检查实现

```cpp
// 文件: src/utils/src/permission.cpp

// Mirror 权限检查
bool Permission::CheckMirrorPermission() {
    return CheckPermission("ohos.permission.ACCESS_CAST_ENGINE_MIRROR");
}

// Stream 权限检查
bool Permission::CheckStreamPermission() {
    return CheckPermission("ohos.permission.ACCESS_CAST_ENGINE_STREAM");
}

// PID 白名单检查
bool Permission::CheckPidPermission() {
    pid_t pid = IPCSkeleton::GetCallingPid();
    // 检查 PID 是否在白名单中
    // ...
}
```

### 6.3 权限检查位置

| 文件 | 方法 | 检查权限 |
|------|------|----------|
| `cast_session_impl_stub.cpp` | `DoAddDeviceTask()` | Mirror 或 Stream |
| `cast_session_impl_stub.cpp` | `DoRemoveDeviceTask()` | Mirror 或 Stream |
| `cast_session_impl_stub.cpp` | `DoStartAuthTask()` | Mirror 或 Stream |
| `cast_session_impl_stub.cpp` | `DoCreateMirrorPlayer()` | Mirror |
| `cast_session_impl_stub.cpp` | `DoCreateStreamPlayer()` | Stream |
| `stream_player_impl_stub.cpp` | `OnRemoteRequest()` | Stream + PID |

---

## 7. 错误码

### 7.1 通用错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERR_NONE` | 0 | 成功 |
| `ERR_NULL_OBJECT` | -1 | 空对象 |
| `ERR_INVALID_DATA` | -2 | 无效数据 |
| `ERR_UNKNOWN_TRANSACTION` | -3 | 未知事务（权限不足） |
| `IPC_STUB_WRITE_PARCEL_ERR` | -4 | IPC 写入错误 |
| `IPC_STUB_ERR` | -5 | IPC 错误 |

### 7.2 会话错误码

| 错误码 | 说明 |
|--------|------|
| `ERR_SESSION_NOT_FOUND` | 会话不存在 |
| `ERR_DEVICE_NOT_FOUND` | 设备不存在 |
| `ERR_CONNECTION_FAILED` | 连接失败 |
| `ERR_AUTHENTICATION_FAILED` | 认证失败 |

---

## 8. 调用示例

### 8.1 创建会话并添加设备

```cpp
// 1. 获取 Session 服务
sptr<IRemoteObject> service = GetCastSessionService();
sptr<ICastSessionImpl> session = iface_cast<ICastSessionImpl>(service);

// 2. 注册监听器
sptr<ICastSessionListenerImpl> listener = new MySessionListener();
session->RegisterListener(listener);

// 3. 添加设备
CastRemoteDevice device;
device.deviceId = "device_001";
device.deviceName = "Living Room TV";
device.endType = EndType::SINK;
int32_t ret = session->AddDevice(device);
if (ret != ERR_NONE) {
    // 处理错误
}

// 4. 创建镜像播放器
sptr<IMirrorPlayerImpl> mirrorPlayer;
ret = session->CreateMirrorPlayer(mirrorPlayer);
if (ret == ERR_NONE && mirrorPlayer != nullptr) {
    mirrorPlayer->Play(device.deviceId);
}
```

### 8.2 播放器控制

```cpp
// 创建流媒体播放器
sptr<IStreamPlayerIpc> streamPlayer;
session->CreateStreamPlayer(streamPlayer);

// 注册播放器监听器
sptr<IStreamPlayerListenerImpl> playerListener = new MyPlayerListener();
streamPlayer->RegisterListener(playerListener);

// 加载媒体
streamPlayer->Load("http://example.com/video.mp4", "video/mp4");

// 播放控制
streamPlayer->Play();
streamPlayer->Pause();
streamPlayer->Seek(60000);  // Seek 到 60 秒
streamPlayer->SetVolume(50);  // 设置音量 50%
```

---

## 9. 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [05_Internal_API.md](./05_Internal_API.md) - 内部 API
- [08_Security_Review.md](./08_Security_Review.md) - 安全评审
