# 对外 API（IPC 接口）

## 概述

分布式音频通过 **System Ability（SA）** 机制对外提供服务。提供两个 SA：

| SA ID | 服务名 | 说明 |
|-------|--------|------|
| **4805** | DAudioSourceService | Source 端服务，管理音频发送 |
| **4806** | DAudioSinkService | Sink 端服务，管理音频接收 |

**调用方式**: 通过 OpenHarmony IPC 框架，使用 Proxy/Stub 模式调用。

## 重要说明

### 无 N-API 接口

**本项目不直接提供 N-API（JS/TS）接口**。应用开发者应使用以下方式间接访问：

1. **推荐**: 通过音频框架 (`@ohos.multimedia.audio`)
2. **系统级**: 通过分布式硬件框架

### 接口类型

本文档描述的接口是 **Inner API**（Native C++ 接口），供：
- 分布式硬件框架调用
- 系统服务间通信
- 需要原生能力的系统应用

## Source 端接口（SA 4805）

### 接口定义

```cpp
// interfaces/inner_kits/native_cpp/audio_source/include/idaudio_source.h
class IDAudioSource : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedaudiosource");

    virtual int32_t InitSource(const std::string &params, 
                               const sptr<IDAudioIpcCallback> &callback) = 0;
    virtual int32_t ReleaseSource() = 0;
    virtual int32_t RegisterDistributedHardware(const std::string &devId, 
                                                 const std::string &dhId,
                                                 const EnableParam &param, 
                                                 const std::string &reqId) = 0;
    virtual int32_t UnregisterDistributedHardware(const std::string &devId, 
                                                   const std::string &dhId,
                                                   const std::string &reqId) = 0;
    virtual int32_t ConfigDistributedHardware(const std::string &devId, 
                                               const std::string &dhId, 
                                               const std::string &key,
                                               const std::string &value) = 0;
    virtual void DAudioNotify(const std::string &devId, 
                              const std::string &dhId, 
                              const int32_t eventType,
                              const std::string &eventContent) = 0;
    virtual int32_t UpdateDistributedHardwareWorkMode(const std::string &devId, 
                                                       const std::string &dhId,
                                                       const WorkModeParam &param) = 0;
};
```

### IPC 命令码

```cpp
// common/include/daudio_ipc_interface_code.h
enum DAudioSourceInterfaceCode {
    INIT_SOURCE = 0,
    RELEASE_SOURCE = 1,
    REGISTER_DISTRIBUTED_HARDWARE = 2,
    UNREGISTER_DISTRIBUTED_HARDWARE = 3,
    CONFIG_DISTRIBUTED_HARDWARE = 4,
    DAUDIO_NOTIFY = 5,
    UPDATE_WORKMODE = 6,
};
```

### API 清单

| 方法 | 命令码 | 同步/异步 | 权限要求 | 说明 |
|------|--------|-----------|----------|------|
| `InitSource` | 0 | 同步 | DISTRIBUTED_DATASYNC | 初始化 Source 服务 |
| `ReleaseSource` | 1 | 同步 | DISTRIBUTED_DATASYNC | 释放 Source 服务 |
| `RegisterDistributedHardware` | 2 | 异步 | DISTRIBUTED_DATASYNC | 注册分布式硬件 |
| `UnregisterDistributedHardware` | 3 | 异步 | DISTRIBUTED_DATASYNC | 注销分布式硬件 |
| `ConfigDistributedHardware` | 4 | 同步 | DISTRIBUTED_DATASYNC | 配置硬件参数 |
| `DAudioNotify` | 5 | 异步 | DISTRIBUTED_DATASYNC | 事件通知 |
| `UpdateDistributedHardwareWorkMode` | 6 | 同步 | DISTRIBUTED_DATASYNC | 更新工作模式 |

### 调用示例

```cpp
#include "idaudio_source.h"
#include "daudio_source_proxy.h"

// 获取 SA 服务
sptr<IRemoteObject> remote = 
    OHOS::DelayedSingleton<SaMgrClient>::GetInstance()-&gt;GetSystemAbility(4805);
sptr<IDAudioSource> sourceService = iface_cast<IDAudioSource>(remote);

// 初始化
std::string params = "{}";
sptr<IDAudioIpcCallback> callback = new MyIpcCallback();
int32_t ret = sourceService-&gt;InitSource(params, callback);

// 注册硬件
EnableParam param;
param.audioParam = audioParams;
ret = sourceService-&gt;RegisterDistributedHardware(devId, dhId, param, reqId);
```

### 回调接口

```cpp
// interfaces/inner_kits/native_cpp/audio_source/include/idaudio_ipc_callback.h
class IDAudioIpcCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedaudioipccallback");
    
    virtual void OnNotifyRegResult(const std::string &devId, 
                                   const std::string &dhId,
                                   const std::string &reqId, 
                                   int32_t status, 
                                   int32_t result) = 0;
    virtual void OnNotifyUnregResult(const std::string &devId, 
                                     const std::string &dhId,
                                     const std::string &reqId, 
                                     int32_t result) = 0;
    virtual void OnHardwareStateChanged(const std::string &devId, 
                                        const std::string &dhId,
                                        int32_t state) = 0;
    virtual void OnDataSyncTrigger(const std::string &devId) = 0;
};
```

## Sink 端接口（SA 4806）

### 接口定义

```cpp
// interfaces/inner_kits/native_cpp/audio_sink/include/idaudio_sink.h
class IDAudioSink : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedaudiosink");

    virtual int32_t InitSink(const std::string &params, 
                             const sptr<IDAudioSinkIpcCallback> &sinkCallback) = 0;
    virtual int32_t ReleaseSink() = 0;
    virtual int32_t SubscribeLocalHardware(const std::string &dhId, 
                                           const std::string &param) = 0;
    virtual int32_t UnsubscribeLocalHardware(const std::string &dhId) = 0;
    virtual void DAudioNotify(const std::string &devId, 
                              const std::string &dhId, 
                              const int32_t eventType,
                              const std::string &eventContent) = 0;
    virtual int32_t PauseDistributedHardware(const std::string &networkId) = 0;
    virtual int32_t ResumeDistributedHardware(const std::string &networkId) = 0;
    virtual int32_t StopDistributedHardware(const std::string &networkId) = 0;
    virtual int32_t SetAccessListener(const sptr<IAccessListener> &listener, 
                                      int32_t timeOut,
                                      const std::string &pkgName) = 0;
    virtual int32_t RemoveAccessListener(const std::string &pkgName) = 0;
    virtual int32_t SetAuthorizationResult(const std::string &requestId, 
                                           bool granted) = 0;
};
```

### IPC 命令码

```cpp
// common/include/daudio_ipc_interface_code.h
enum DAudioSinkInterfaceCode {
    INIT_SINK = 0,
    RELEASE_SINK = 1,
    SUBSCRIBE_LOCAL_HARDWARE = 2,
    UNSUBSCRIBE_LOCAL_HARDWARE = 3,
    DAUDIO_NOTIFY = 4,
    PAUSE_DISTRIBUTED_HARDWARE = 5,
    RESUME_DISTRIBUTED_HARDWARE = 6,
    STOP_DISTRIBUTED_HARDWARE = 7,
    SET_ACCESS_LISTENER = 8,
    REMOVE_ACCESS_LISTENER = 9,
    SET_AUTHORIZATION_RESULT = 10,
};
```

### API 清单

| 方法 | 命令码 | 同步/异步 | 权限要求 | 说明 |
|------|--------|-----------|----------|------|
| `InitSink` | 0 | 同步 | DISTRIBUTED_DATASYNC | 初始化 Sink 服务 |
| `ReleaseSink` | 1 | 同步 | DISTRIBUTED_DATASYNC | 释放 Sink 服务 |
| `SubscribeLocalHardware` | 2 | 异步 | DISTRIBUTED_DATASYNC | 订阅本地硬件 |
| `UnsubscribeLocalHardware` | 3 | 异步 | DISTRIBUTED_DATASYNC | 取消订阅 |
| `DAudioNotify` | 4 | 异步 | DISTRIBUTED_DATASYNC | 事件通知 |
| `PauseDistributedHardware` | 5 | 同步 | DISTRIBUTED_DATASYNC | 暂停硬件 |
| `ResumeDistributedHardware` | 6 | 同步 | DISTRIBUTED_DATASYNC | 恢复硬件 |
| `StopDistributedHardware` | 7 | 同步 | DISTRIBUTED_DATASYNC | 停止硬件 |
| `SetAccessListener` | 8 | 同步 | DISTRIBUTED_DATASYNC | 设置访问监听 |
| `RemoveAccessListener` | 9 | 同步 | DISTRIBUTED_DATASYNC | 移除访问监听 |
| `SetAuthorizationResult` | 10 | 同步 | DISTRIBUTED_DATASYNC | 设置授权结果 |

### 回调接口

```cpp
// interfaces/inner_kits/native_cpp/audio_sink/include/idaudio_sink_ipc_callback.h
class IDAudioSinkIpcCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedaudiosinkipccallback");
    
    virtual int32_t OnNotifyResourceInfo(const ResourceEventType &type, 
                                         const std::string &subType,
                                         const std::string &networkId,
                                         bool &isSensitive,
                                         bool &isSameAccount) = 0;
    virtual int32_t OnInitHardwareResource(const std::string &networkId,
                                           const std::string &deviceName) = 0;
};
```

## 权限要求

### 必需权限

| 权限 | 权限名 | 说明 |
|------|--------|------|
| **DISTRIBUTED_DATASYNC** | ohos.permission.DISTRIBUTED_DATASYNC | 分布式数据同步权限（必须）|

### SA 配置权限

```ini
# sa_profile/daudio.cfg
[service]
privileges = [
    "ohos.permission.MICROPHONE",
    "ohos.permission.DISTRIBUTED_DATASYNC",
    "ohos.permission.ACCESS_SERVICE_DM",
    "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
    "ohos.permission.CAPTURE_VOICE_DOWNLINK_AUDIO"
]
```

### 权限检查实现

```cpp
// services/audiomanager/servicesource/src/daudio_source_stub.cpp:84-92
bool DAudioSourceStub::VerifyPermission() {
    AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t result = AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, OHOS_PERMISSION_DISTRIBUTED_DATASYNC);
    return result == AccessToken::PERMISSION_GRANTED;
}
```

## 错误码

```cpp
// common/include/daudio_errorcode.h
enum DAudioErrorCode {
    DH_SUCCESS = 0,
    ERR_DH_AUDIO_FAILED = -1,
    ERR_DH_AUDIO_SA_INVALID_INTERFACE_TOKEN = -2,
    ERR_DH_AUDIO_SA_PERMISSION_CHECK_FAIL = -3,
    ERR_DH_AUDIO_SA_INIT_FAILED = -4,
    ERR_DH_AUDIO_SA_DEVICE_NOT_EXIST = -5,
    ERR_DH_AUDIO_SA_PARAM_INVALID = -6,
    // ... 更多错误码
};
```

## 调用链示例

### 注册分布式硬件

```
分布式硬件框架
    │
    ▼
DAudioSourceProxy::RegisterDistributedHardware()  [interfaces/inner_kits/native_cpp/audio_source/src/daudio_source_proxy.cpp]
    │
    ▼ (IPC)
DAudioSourceStub::OnRemoteRequest()  [services/audiomanager/servicesource/src/daudio_source_stub.cpp:50]
    │
    ▼ (权限检查)
DAudioSourceStub::VerifyPermission()  [services/audiomanager/servicesource/src/daudio_source_stub.cpp:84]
    │
    ▼
DAudioSourceService::RegisterDistributedHardware()  [services/audiomanager/servicesource/src/daudio_source_service.cpp]
    │
    ▼
DAudioSourceManager::EnableDAudio()  [services/audiomanager/managersource/src/daudio_source_manager.cpp]
    │
    ▼
DAudioSourceDev::AwakeAudioDev()  [services/audiomanager/managersource/src/daudio_source_dev.cpp]
```

## 接口使用场景

### 场景 1: 设备 A 使用设备 B 的扬声器

```
设备 A (Source)              设备 B (Sink)
    │                            │
    │  1. RegisterDistributedHardware()  │
    │ ───────────────────────────────► │
    │                            │
    │                            │  2. 订阅本地扬声器
    │                            │  3. 初始化传输引擎
    │                            │
    │  4. OnNotifyRegResult(成功)      │
    │ ◄─────────────────────────────── │
    │                            │
    │  5. 应用使用虚拟扬声器           │
    │  6. 音频数据 ──────────────► 7. 播放
```

### 场景 2: 设备 A 使用设备 B 的麦克风

```
设备 A (Source)              设备 B (Sink)
    │                            │
    │  1. RegisterDistributedHardware()  │
    │ ───────────────────────────────► │
    │                            │
    │                            │  2. 订阅本地麦克风
    │                            │  3. 开始采集
    │                            │
    │  4. 接收音频数据 ◄─────────────── │
    │                            │
    │  5. 应用读取虚拟麦克风数据       │
```

---

*文档生成时间: 2025-02-06*
