# 内部 API 文档

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档提供 CastEngine 框架的内部 C++ API 接口定义、模块依赖关系和稳定性说明。

## API 概览

### 接口层级

```
内部 API 层 (libcast_engine_client.z.so)
    │
    ├── 公共接口 (interfaces/inner_api/include/)
    │   • ICastSession
    │   • IStreamPlayer
    │   • IMirrorPlayer
    │   • ICastSessionManager
    │   • 数据结构定义
    │
    └── 客户端实现 (client/src/)
    │   • Proxy 实现
    │   • 对象封装
    │   • IPC 调用
```

### 使用场景

**目标用户**:
- OpenHarmony 系统框架开发者
- 其他需要集成 CastEngine 能力的组件
- 测试和验证代码

**调用方式**:
1. 链接到 `libcast_engine_client.z.so`
2. 包含接口头文件
3. 调用接口方法
4. 实现监听器接口

---

## 接口定义

### 1. ICastSession

**位置**: `interfaces/inner_api/include/i_cast_session.h`
**职责**: 投屏会话接口

**主要方法**:

```cpp
class ICastSession {
public:
    // 设备管理
    virtual int32_t AddDevice(const CastDeviceInfo& device) = 0;
    virtual int32_t RemoveDevice(const std::string& deviceId) = 0;
    virtual std::vector<CastDeviceInfo> GetDevices() const = 0;

    // 会话信息
    virtual int32_t GetSessionId(std::string& sessionId) const = 0;
    virtual int32_t SetSessionProperty(const CastSessionProperty& property) = 0;
    virtual int32_t GetSessionProperty(CastSessionProperty& property) const = 0;

    // 播放器创建
    virtual std::shared_ptr<IMirrorPlayer> CreateMirrorPlayer() = 0;
    virtual std::shared_ptr<IStreamPlayer> CreateStreamPlayer() = 0;

    // 模式设置
    virtual int32_t SetCastMode(CastMode mode) = 0;

    // 会话生命周期
    virtual int32_t RegisterListener(const std::shared_ptr<ICastSessionListener>& listener) = 0;
    virtual int32_t UnregisterListener(const std::shared_ptr<ICastSessionListener>& listener) = 0;
    virtual int32_t Release() = 0;
};
```

**实现位置**: `service/src/session/src/cast_session_impl.cpp`
**客户端代理**: `client/src/cast_session_impl_proxy.cpp`

### 2. IStreamPlayer

**位置**: `interfaces/inner_api/include/i_stream_player.h`
**职责**: 流媒体播放器接口

**主要方法**:

```cpp
class IStreamPlayer {
public:
    // Surface 设置
    virtual int32_t SetSurface(const sptr<Surface>& surface) = 0;

    // 媒体加载
    virtual int32_t Load(const CastMediaInfoHolder& mediaInfo) = 0;
    virtual int32_t LoadNext() = 0;
    virtual int32_t LoadPrevious() = 0;

    // 播放控制
    virtual int32_t Start() = 0;
    virtual int32_t Play() = 0;
    virtual int32_t Pause() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Next() = 0;
    virtual int32_t Previous() = 0;
    virtual int32_t Seek(int position) = 0;
    virtual int32_t FastForward() = 0;
    virtual int32_t FastRewind() = 0;

    // 属性设置
    virtual int32_t SetVolume(int volume) = 0;
    virtual int32_t SetLoopMode(LoopMode mode) = 0;
    virtual int32_t SetSpeed(float speed) = 0;
    virtual int32_t SetMute(bool mute) = 0;

    // 状态查询
    virtual int32_t GetPlayerStatus() const = 0;
    virtual int32_t GetPosition() const = 0;
    virtual int32_t GetVolume() const = 0;
    virtual int32_t GetMute() const = 0;
    virtual int32_t GetLoopMode() const = 0;
    virtual int32_t GetPlaySpeed() const = 0;
    virtual int32_t GetMediaInfoHolder(CastMediaInfoHolder& info) const = 0;

    // 监听器
    virtual int32_t RegisterListener(const std::shared_ptr<IStreamPlayerListener>& listener) = 0;
    virtual int32_t UnregisterListener(const std::shared_ptr<IStreamPlayerListener>& listener) = 0;

    // 生命周期
    virtual int32_t Release() = 0;
};
```

**实现位置**: `service/src/session/src/stream/src/player/src/cast_stream_player.cpp`
**客户端代理**: `client/src/stream_player_impl_proxy.cpp`

### 3. IMirrorPlayer

**位置**: `interfaces/inner_api/include/i_mirror_player.h`
**职责**: 镜像投屏播放器接口

**主要方法**:

```cpp
class IMirrorPlayer {
public:
    // 播放控制
    virtual int32_t Play() = 0;
    virtual int32_t Pause() = 0;

    // 应用信息
    virtual int32_t SetAppInfo(const AppInfo& appInfo) = 0;

    // Surface 设置
    virtual int32_t SetSurface(const sptr<Surface>& surface) = 0;

    // 屏幕配置
    virtual int32_t ResizeVirtualScreen(int width, int height) = 0;
    virtual int32_t SetCastRoute(const CastRoute& route) = 0;

    // 截图
    virtual int32_t GetScreenshot(const PixelMap& image) const = 0;

    // 生命周期
    virtual int32_t Release() = 0;
};
```

**实现位置**: `service/src/session/src/mirror/src/mirror_player_impl.cpp`
**客户端代理**: `client/src/mirror_player_impl_proxy.cpp`

### 4. ICastSessionManager

**位置**: `interfaces/inner_api/include/cast_session_manager.h`
**职责**: 会话管理器接口

**主要方法**:

```cpp
class ICastSessionManager {
public:
    // 设备发现
    virtual int32_t StartDiscovery(int32_t protocolType) = 0;
    virtual int32_t StopDiscovery() = 0;
    virtual int32_t SetDiscoverable(bool isEnable) = 0;

    // 会话管理
    virtual int32_t CreateCastSession(const CastSessionProperty& property,
                                std::shared_ptr<ICastSession>& session) = 0;
    virtual int32_t Release() = 0;

    // 监听器
    virtual int32_t RegisterListener(const std::shared_ptr<ICastSessionManagerListener>& listener) = 0;
    virtual int32_t UnregisterListener(const std::shared_ptr<ICastSessionManagerListener>& listener) = 0;

    // 服务状态
    virtual int32_t GetServiceState() const = 0;
};
```

**实现位置**: `service/src/cast_session_manager_service.cpp`
**客户端代理**: `client/src/cast_session_manager_service_proxy.cpp`

### 5. 监听器接口

#### ICastSessionListener

**位置**: `interfaces/inner_api/include/i_cast_session.h`（在头文件末尾定义）

**主要方法**:
```cpp
class ICastSessionListener {
public:
    virtual void OnDeviceState(const DeviceState& state) = 0;
    virtual void OnEvent(const CastEngineEvent& event) = 0;
};
```

#### ICastSessionManagerListener

**位置**: `interfaces/inner_api/include/i_cast_session_manager_listener.h`

**主要方法**:
```cpp
class ICastSessionManagerListener {
public:
    virtual void OnServiceDied() = 0;
    virtual void OnDeviceFound(const CastRemoteDevice& device) = 0;
    virtual void OnSessionCreated(const std::shared_ptr<ICastSession>& session) = 0;
    virtual void OnDeviceOffline(const std::string& deviceId) = 0;
};
```

#### IStreamPlayerListener

**位置**: `interfaces/inner_api/include/`（在 StreamPlayer 实现中定义）

**主要方法**:
```cpp
class IStreamPlayerListener {
public:
    virtual void OnStateChanged(const PlayerState& state) = 0;
    virtual void OnPositionChanged(int position) = 0;
    virtual void OnMediaItemChanged(const MediaInfo& item) = 0;
    virtual void OnVolumeChanged(int volume) = 0;
    virtual void OnVideoSizeChanged(int width, int height) = 0;
    virtual void OnLoopModeChanged(LoopMode mode) = 0;
    virtual void OnPlaySpeedChanged(float speed) = 0;
    virtual void OnPlayerError(int errorCode) = 0;
    virtual void OnNextRequest() = 0;
    virtual void OnPreviousRequest() = 0;
    virtual void OnSeekDone(int position) = 0;
    virtual void OnEndOfStream() = 0;
    virtual void OnImageChanged(const PixelMap& image) = 0;
};
```

---

## 数据结构

### CastDeviceInfo

**位置**: `interfaces/inner_api/include/cast_engine_common.h`

**用途**: 设备信息结构

**主要字段**:
```cpp
struct CastDeviceInfo {
    std::string deviceId;      // 设备 ID
    std::string deviceName;    // 设备名称
    DeviceType deviceType;      // 设备类型
    std::string ipAddress;     // IP 地址
    int32_t port;            // 端口号
    // ... 其他字段
};
```

### CastSessionProperty

**位置**: `interfaces/inner_api/include/cast_engine_common.h`

**用途**: 会话属性配置

**主要字段**:
```cpp
struct CastSessionProperty {
    std::string deviceId;      // 目标设备 ID
    CastMode castMode;       // 投屏模式
    // ... 其他配置字段
};
```

### CastMediaInfoHolder

**位置**: `interfaces/inner_api/include/cast_engine_common.h`

**用途**: 媒体信息结构

**主要字段**:
```cpp
struct CastMediaInfoHolder {
    std::string url;         // 媒体 URL
    std::string title;       // 标题
    int32_t duration;        // 时长（毫秒）
    // ... 其他字段
};
```

---

## 模块依赖

### 依赖方向

```
外部依赖者（系统框架、其他组件）
    │
    ↓ 依赖
┌──────────────────────────────────┐
│   CastEngine 内部 API 层    │
│   (libcast_engine_client.z.so)  │
└──────────┬──────────────────────┘
           │ 依赖
           ↓
    ┌────────────────────────────────────────────┐
    │          CastEngine 公共代码层        │
    │     (libcast_engine_common_sources.a)    │
    │    • 日志 (cast_engine_log)             │
    │    • DFX (cast_engine_dfx)               │
    │    • 工具 (cast_engine_common_helper)    │
    └──────────────────────────────────────────────┘
```

### 依赖关系表

| 模块 | 依赖的模块 | 用途 |
|-----|-----------|------|
| libcast_engine_client.z.so | cast_engine_common_sources.a | 公共工具和数据结构 |
| interfaces/kits/js (libcast.z.so) | libcast_engine_client.z.so | 内部 API |
| cast_engine_service (SA) | cast_client_inner | 客户端代理 |
| cast_engine_service (SA) | cast_discovery | 设备发现 |
| cast_engine_service (SA) | cast_session | 会话实现 |
| cast_engine_service (SA) | 多个外部依赖 | 系统 API |

---

## 接口稳定性

### 稳定性分类

**公共接口** (interfaces/inner_api/include/):
- ✅ **稳定** - 可以被外部组件依赖
- ✅ 有版本控制
- ✅ 有文档说明

**私有接口** (common/include/private/):
- ⚠️ **不稳定** - 仅限内部使用
- ⚠️ 可能变更
- ⚠️ 无对外兼容性保证

**实现类** (service/src/, client/src/):
- ⚠️ **内部** - 实现细节
- ⚠️ 不作为公共 API

### 接口演化

| 版本 | 变化 | 说明 |
|-----|------|------|
| 当前 | - | 所有接口都是 v1 |
| 未来 | - | 可能添加新方法或新接口 |

### 替换点

**可替换的组件**:
1. **协议适配器**: DLNA/WiFi Display/Cast+Stream
   - 通过统一接口集成
   - 可独立更新

2. **设备发现模块**: DiscoveryManager
   - 支持新的发现机制

3. **播放器后端**: StreamPlayer/MirrorPlayer
   - 可替换不同的实现

---

## 使用示例

### 示例 1: 在 C++ 中使用内部 API

```cpp
#include "i_cast_session_manager.h"
#include "i_cast_session.h"
#include "i_stream_player.h"

using namespace OHOS::CastEngine;

class MyApplication {
public:
    void StartCasting() {
        // 1. 获取会话管理器
        auto sessionManager = CastSessionManager::GetInstance();

        // 2. 开始设备发现
        int32_t ret = sessionManager->StartDiscovery(PROTOCOL_WIFI_DISPLAY);
        if (ret != CAST_ENGINE_SUCCESS) {
            LOGE("Start discovery failed: %{public}d", ret);
            return;
        }

        // 3. 创建会话（需要等待 deviceFound 事件）
        // 在监听器中处理：OnDeviceFound()
    }
};
```

### 示例 2: 实现监听器

```cpp
class MySessionListener : public ICastSessionManagerListener {
public:
    void OnDeviceFound(const CastRemoteDevice& device) override {
        LOGI("Device found: %{public}s", device.deviceName.c_str());
        // 处理发现的设备
    }

    void OnSessionCreated(const std::shared_ptr<ICastSession>& session) override {
        LOGI("Session created");
        // 保存会话引用
        mySession_ = session;
    }

    void OnDeviceOffline(const std::string& deviceId) override {
        LOGI("Device offline: %{public}s", deviceId.c_str());
        // 处理设备离线
    }

    void OnServiceDied() override {
        LOGE("Service died!");
        // 处理服务死亡
    }

private:
    std::shared_ptr<ICastSession> mySession_;
};
```

---

## 错误码

所有接口方法返回标准的 CastEngine 错误码：

| 错误码 | 值 | 说明 |
|-------|------|------|
| CAST_ENGINE_SUCCESS | 0 | 成功 |
| CAST_ENGINE_ERROR | -1 | 通用错误 |
| ERR_NO_MEMORY | -1001 | 内存不足 |
| ERR_INVALID_PARAM | -1002 | 无效参数 |
| ERR_NO_PERMISSION | -1003 | 无权限 |
| ERR_SESSION_NOT_EXIST | -1004 | 会话不存在 |
| ERR_SERVICE_STATE_NOT_MATCH | -1005 | 服务状态不匹配 |
| ERR_SESSION_STATE_NOT_MATCH | -1006 | 会话状态不匹配 |
| ERR_SERVICE_IS_UNLOADING | -1007 | 服务正在卸载 |

**完整定义**: `interfaces/inner_api/include/cast_engine_errors.h:28-37`

---

## 相关文档

- [目录结构](01_Directory_Structure.md) - 接口文件的组织方式
- [N-API 文档](03_N-API.md) - JavaScript 调用的内部接口
- [架构文档](02_Architecture.md) - 接口在整体架构中的位置
- [GN Targets](05_GN_Targets.md) - 如何链接和使用这些接口
- [错误码定义](../_work/NOTES.md#错误码定义) - 完整的错误码列表

---

**返回**: [SUMMARY.md](SUMMARY.md)
