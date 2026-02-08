# Inner API 接口文档

## 概述

Inner API 提供 Native C++ 接口供系统级组件使用，主要包含 WfdSink 和 WfdSource 两个核心接口。

## WfdSink 接口

### 接口定义

**头文件**: `interfaces/innerkits/native/wfd/include/wfd_sink.h`

```cpp
class WfdSink {
public:
    virtual ~WfdSink() = default;

    virtual int32_t Stop() = 0;
    virtual int32_t Start() = 0;
    virtual int32_t Play(std::string deviceId) = 0;
    virtual int32_t Pause(std::string deviceId) = 0;
    virtual int32_t Close(std::string deviceId) = 0;

    virtual int32_t Mute(std::string deviceId) = 0;
    virtual int32_t UnMute(std::string deviceId) = 0;

    virtual int32_t GetSinkConfig(SinkConfig &sinkConfig) = 0;

    virtual int32_t AppendSurface(std::string deviceId, uint64_t surfaceId) = 0;
    virtual int32_t AppendSurface(std::string deviceId, sptr<IBufferProducer> producer) = 0;
    virtual int32_t RemoveSurface(std::string deviceId, uint64_t surfaceId) = 0;

    virtual int32_t SetListener(const std::shared_ptr<IWfdEventListener> &listener) = 0;

    virtual int32_t SetSceneType(std::string deviceId, uint64_t surfaceId, SceneType sceneType) = 0;
    virtual int32_t SetMediaFormat(std::string deviceId, CodecAttr videoAttr, CodecAttr audioAttr) = 0;
    virtual int32_t GetBoundDevicesList(std::vector<BoundDeviceInfo> &devices) = 0;
    virtual int32_t DeleteBoundDevice(std::string &deviceAddress) = 0;
};
```

### 方法说明

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Start()` | 无 | `int32_t` | 启动 Sink 服务 |
| `Stop()` | 无 | `int32_t` | 停止 Sink 服务 |
| `Play(deviceId)` | 设备 ID | `int32_t` | 播放指定设备的投屏 |
| `Pause(deviceId)` | 设备 ID | `int32_t` | 暂停指定设备的投屏 |
| `Close(deviceId)` | 设备 ID | `int32_t` | 关闭指定设备的投屏 |
| `Mute(deviceId)` | 设备 ID | `int32_t` | 静音指定设备 |
| `UnMute(deviceId)` | 设备 ID | `int32_t` | 取消静音指定设备 |
| `GetSinkConfig(sinkConfig)` | 接收配置 | `int32_t` | 获取 Sink 配置 |
| `AppendSurface(deviceId, surfaceId)` | 设备 ID, Surface ID | `int32_t` | 添加 Surface |
| `AppendSurface(deviceId, producer)` | 设备 ID, BufferProducer | `int32_t` | 添加 BufferProducer |
| `RemoveSurface(deviceId, surfaceId)` | 设备 ID, Surface ID | `int32_t` | 移除 Surface |
| `SetListener(listener)` | 事件监听器 | `int32_t` | 设置事件监听 |
| `SetSceneType(deviceId, surfaceId, sceneType)` | 设备 ID, Surface ID, 场景类型 | `int32_t` | 设置场景类型 |
| `SetMediaFormat(deviceId, videoAttr, audioAttr)` | 设备 ID, 视频属性, 音频属性 | `int32_t` | 设置媒体格式 |
| `GetBoundDevicesList(devices)` | 设备列表 | `int32_t` | 获取已绑定设备列表 |
| `DeleteBoundDevice(deviceAddress)` | 设备地址 | `int32_t` | 删除已绑定设备 |

### 工厂方法

**头文件**: `interfaces/innerkits/native/wfd/include/wfd_sink.h:52-60`

```cpp
class __attribute__((visibility("default"))) WfdSinkFactory {
public:
    static std::shared_ptr<WfdSink> CreateSink(int32_t type, const std::string key);

private:
    WfdSinkFactory() = default;
    ~WfdSinkFactory() = default;
    static std::shared_ptr<WfdSink> wfdSinkImpl_;
};
```

### 事件监听

**头文件**: `interfaces/innerkits/native/wfd/include/wfd.h:24-29`

```cpp
class IWfdEventListener {
public:
    virtual ~IWfdEventListener() = default;

    virtual void OnInfo(std::shared_ptr<BaseMsg> &info) = 0;
};
```

## WfdSource 接口

### 接口定义

**头文件**: `interfaces/innerkits/native/wfd/include/wfd_source.h`

```cpp
class WfdSource {
public:
    virtual ~WfdSource() = default;
    virtual int32_t StopDiscover() = 0;
    virtual int32_t StartDiscover() = 0;
    virtual int32_t RemoveDevice(std::string deviceId) = 0;
    virtual int32_t AddDevice(uint64_t screenId, WfdCastDeviceInfo &deviceInfo) = 0;
    virtual int32_t SetListener(const std::shared_ptr<IWfdEventListener> &listener) = 0;
};
```

### 方法说明

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `StartDiscover()` | 无 | `int32_t` | 开始发现设备 |
| `StopDiscover()` | 无 | `int32_t` | 停止发现设备 |
| `RemoveDevice(deviceId)` | 设备 ID | `int32_t` | 移除设备 |
| `AddDevice(screenId, deviceInfo)` | Screen ID, 设备信息 | `int32_t` | 添加设备 |
| `SetListener(listener)` | 事件监听器 | `int32_t` | 设置事件监听 |

### 工厂方法

**头文件**: `interfaces/innerkits/native/wfd/include/wfd_source.h:35-43`

```cpp
class __attribute__((visibility("default"))) WfdSourceFactory {
public:
    static std::shared_ptr<WfdSource> CreateSource(int32_t type, const std::string key);

private:
    WfdSourceFactory() = default;
    ~WfdSourceFactory() = default;
    static std::shared_ptr<WfdSource> wfdSourceImpl_;
};
```

## 稳定性说明

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `IWfdEventListener` | 稳定 | 纯虚接口，版本兼容 |
| `WfdSink` | 稳定 | 主要对外接口 |
| `WfdSource` | 稳定 | 主要对外接口 |
| `WfdSinkFactory` | 稳定 | 提供创建方法 |
| `WfdSourceFactory` | 稳定 | 提供创建方法 |

**稳定性依据**:
- 接口使用 `virtual` 纯虚函数定义
- 析构函数为 `default`
- 头文件位于 `interfaces/innerkits/` 目录

## 依赖方向

```
                    ┌─────────────────────┐
                    │  WfdSink/WfdSource  │
                    └──────────┬──────────┘
                               │
                               ├──► IWfdEventListener
                               │           │
                               │           ▼
                               │      BaseMsg
                               │
                               ├──► SinkConfig
                               │           │
                               │           ▼
                               │      CodecAttr
                               │
                               ├──► IBufferProducer
                               │           │
                               │           ▼
                               │      Surface
                               │
                               └──► WfdCastDeviceInfo
```

## 相关文档

- [N-API 接口](03_N-API_Interface.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [架构设计](02_Architecture.md)
