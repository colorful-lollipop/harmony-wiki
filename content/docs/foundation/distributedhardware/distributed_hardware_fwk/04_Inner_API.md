# 内部 API 参考

本文档描述分布式硬件管理框架的内部 Inner API，供其他 OpenHarmony 子系统调用。

> **适用范围**: 需要集成分布式硬件能力的子系统开发者（分布式相机、分布式屏幕等）

---

## 概述

**SDK 名称**: `libdhfwk_sdk`
**头文件目录**: `interfaces/inner_kits/include/`
**实现目录**: `interfaces/inner_kits/src/`

**证据**: `bundle.json:83-92`
```json
"inner_kits": [
  {
    "type": "so",
    "name": "//foundation/.../interfaces/inner_kits:libdhfwk_sdk",
    "header": {
      "header_files": ["distributed_hardware_fwk_kit.h", "distributed_hardware_fwk_kit_paras.h"],
      "header_base": "//foundation/.../interfaces/inner_kits/include"
    }
  }
]
```

---

## 核心类

### DistributedHardwareFwkKit

分布式硬件框架的对外接口类。

**头文件**: `distributed_hardware_fwk_kit.h`

**关键方法**:

| 方法 | 说明 |
|------|------|
| `PauseDistributedHardware()` | 暂停分布式硬件 |
| `ResumeDistributedHardware()` | 恢复分布式硬件 |
| `StopDistributedHardware()` | 停止分布式硬件 |

**证据**: `interfaces/inner_kits/include/distributed_hardware_fwk_kit.h`
```cpp
class DistributedHardwareFwkKit {
public:
    /**
     * @brief Pause distributed hardware
     * @param dhType Distributed hardware type
     * @param networkId Device network ID
     * @return 0: success, other: error code
     */
    int32_t PauseDistributedHardware(DHType dhType, const std::string &networkId);

    /**
     * @brief Resume distributed hardware
     * @param dhType Distributed hardware type
     * @param networkId Device network ID
     * @return 0: success, other: error code
     */
    int32_t ResumeDistributedHardware(DHType dhType, const std::string &networkId);

    /**
     * @brief Stop distributed hardware
     * @param dhType Distributed hardware type
     * @param networkId Device network ID
     * @return 0: success, other: error code
     */
    int32_t StopDistributedHardware(DHType dhType, const std::string &networkId);
};
```

---

## 枚举与类型

### DHType (硬件类型)

| 枚举值 | 说明 |
|--------|------|
| `UNKNOWN` | 未知类型 |
| `CAMERA` | 相机 |
| `AUDIO` | 音频 |
| `SCREEN` | 屏幕 |
| `ALL` | 所有类型 |

### DHSubtype (硬件子类型)

| 枚举值 | 说明 |
|--------|------|
| `CAMERA` | 相机 |
| `AUDIO_MIC` | 音频麦克风 |
| `AUDIO_SPEAKER` | 音频扬声器 |

**证据**: `interfaces/inner_kits/include/distributed_hardware_fwk_kit_paras.h`

---

## 接口定义

### IDistributedHardwareManager

管理分布式硬件的源端（Source）和宿端（Sink）。

**头文件**: `idistributed_hardware_manager.h`

**关键接口**:

```cpp
class IDistributedHardwareManager {
public:
    // 获取管理器实例
    static std::shared_ptr<IDistributedHardwareManager> GetInstance();

    // 初始化
    virtual int32_t Initialize() = 0;

    // 发布硬件能力
    virtual int32_t PublishHardwareAbility(const std::vector<DHAbility> &abilityList) = 0;

    // 订阅硬件能力
    virtual int32_t SubscribeHardwareAbility(const std::string &networkId,
        const std::vector<DHAbility> &abilityList,
        const std::shared_ptr<IHardwareHandler> &handler) = 0;

    // 取消订阅
    virtual int32_t UnSubscribeHardwareAbility(const std::string &networkId) = 0;
};
```

---

### IDistributedHardwareSource

分布式硬件源端接口（提供硬件能力）。

**头文件**: `idistributed_hardware_source.h`

```cpp
class IDistributedHardwareSource {
public:
    // 使能硬件
    virtual int32_t EnableDistributedHardware(const std::string &dhId, const std::string &networkId) = 0;

    // 去使能硬件
    virtual int32_t DisableDistributedHardware(const std::string &dhId, const std::string &networkId) = 0;

    // 订阅设备状态
    virtual int32_t SubscribeDeviceState(const std::string &networkId,
        const std::shared_ptr<ISourceHandler> &handler) = 0;
};
```

---

### IDistributedHardwareSink

分布式硬件宿端接口（使用远程硬件）。

**头文件**: `idistributed_hardware_sink.h`

```cpp
class IDistributedHardwareSink {
public:
    // 使能分布式硬件
    virtual int32_t EnableDistributedHardware(const std::string &dhId, const std::string &networkId) = 0;

    // 去使能分布式硬件
    virtual int32_t DisableDistributedHardware(const std::string &dhId, const std::string &networkId) = 0;

    // 订阅设备状态
    virtual int32_t SubscribeDeviceState(const std::string &networkId,
        const std::shared_ptr<ISinkHandler> &handler) = 0;
};
```

---

## IPC 接口

### 分布式硬件代理 (DistributedHardwareProxy)

客户端 IPC 代理，用于与 SA 服务通信。

**头文件**: `ipc/distributed_hardware_proxy.h`

```cpp
class DistributedHardwareProxy : public IRemoteObject {
public:
    explicit DistributedHardwareProxy(const sptr<IRemoteObject> impl);

    // 获取远程对象
    sptr<IRemoteObject> Remote();

    // 发送 IPC 请求
    int32_t SendRequest(uint32_t code, MessageParcel &data, MessageParcel &reply, MessageOption &option);
};
```

**证据**: `interfaces/inner_kits/src/ipc/distributed_hardware_proxy.cpp:47-68`
```cpp
sptr<IRemoteObject> remote = Remote();
if (remote == nullptr) {
    DHLOGE("Get Remote IRemoteObject failed!");
    return ERR_DH_FWK_SERVICE_REMOTE_IS_NULL;
}
MessageParcel data;
MessageParcel reply;
MessageOption option;
if (!data.WriteInterfaceToken(GetDescriptor())) {
    DHLOGE("WriteInterfaceToken failed!");
    return ERR_DH_FWK_SERVICE_WRITE_TOKEN_FAILED;
}
```

---

### 分布式硬件桩 (DistributedHardwareStub)

服务端 IPC 桩，处理客户端请求。

**头文件**: `ipc/distributed_hardware_stub.h`

```cpp
class DistributedHardwareStub : public IRemoteObjectStub {
public:
    // 远程请求处理
    int32_t OnRemoteRequest(uint32_t code, MessageParcel &data, MessageParcel &reply, MessageOption &option) override;
};
```

**证据**: `services/.../src/distributed_hardware_stub.cpp:39-86`
```cpp
int32_t DistributedHardwareStub::OnRemoteRequest(uint32_t code, MessageParcel &data, 
    MessageParcel &reply, MessageOption &option)
{
    DHLOGI("OnRemoteRequest, code = %{public}u.", code);
    std::u16string descriptor = GetDescriptor();
    std::u16string remoteDescriptor = data.ReadInterfaceToken();
    if (descriptor != remoteDescriptor) {
        DHLOGE("check descriptor failed");
        return ERR_INVALID_DATA;
    }
    // 分发到具体处理函数
}
```

---

## 回调接口

### IHardwareHandler

硬件状态回调处理接口。

**头文件**: `ihardware_handler.h`

```cpp
class IHardwareHandler {
public:
    // 硬件使能回调
    virtual void OnEnabled(const std::string &networkId, const std::string &dhId) = 0;

    // 硬件去使能回调
    virtual void OnDisabled(const std::string &networkId, const std::string &dhId) = 0;

    // 硬件错误回调
    virtual void OnError(const std::string &networkId, const std::string &dhId, int32_t errorCode) = 0;
};
```

---

### IPublisherListener

发布监听接口。

**头文件**: `ipublisher_listener.h`

```cpp
class IPublisherListener {
public:
    // 能力发布回调
    virtual void OnPublisherNotify(const std::string &networkId, const DHAbility &ability,
        const std::string &pubId, int32_t reason) = 0;
};
```

---

## 模块依赖关系

```
                    ┌─────────────────────────────────────┐
                    │         分布式相机/屏幕等子系统       │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │      IDistributedHardwareSource     │
                    │    IDistributedHardwareSink          │
                    │      IHardwareHandler                │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │      DistributedHardwareFwkKit      │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │      DistributedHardwareProxy       │
                    │         (IPC 代理)                   │
                    └─────────────────┬───────────────────┘
                                      │ IPC
                                      ▼
                    ┌─────────────────────────────────────┐
                    │   DistributedHardwareService (SA)  │
                    │         (SA ID: 4801)               │
                    └─────────────────────────────────────┘
```

---

## 稳定性标注

### 稳定接口 (Stable)

以下接口为公共 API，推荐使用：

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `DistributedHardwareFwkKit` | 稳定 | 官方提供的 C++ SDK 接口 |
| `IDistributedHardwareManager` | 稳定 | 设备管理主接口 |
| `IDistributedHardwareSource` | 稳定 | 源端接口 |
| `IDistributedHardwareSink` | 稳定 | 宿端接口 |

### 内部接口 (Internal)

以下接口仅供框架内部使用：

| 接口 | 路径 | 说明 |
|------|------|------|
| `DistributedHardwareStub` | `ipc/` | IPC 桩实现 |
| `DistributedHardwareProxy` | `ipc/` | IPC 代理实现 |
| `PublisherListenerStub` | `ipc/` | 发布监听桩 |
| `PublisherListenerProxy` | `ipc/` | 发布监听代理 |

---

## 后续文档

- GN 构建配置 → [05_GN_Build.md](05_GN_Build.md)
- 编译产物说明 → [06_Build_Artifacts.md](06_Build_Artifacts.md)
- 安全评审 → [07_Security_Review.md](07_Security_Review.md)
