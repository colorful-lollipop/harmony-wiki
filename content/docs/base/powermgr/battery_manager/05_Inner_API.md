# 内部 API 接口

> **目的**: 记录 battery_manager 项目的内部 C++ API 接口、依赖方向、稳定性和可替换点

**适用范围**: 模块间调用的内部接口

---

## BatterySrvClient 接口

### 类定义

**文件路径**: `interfaces/inner_api/native/include/battery_srv_client.h`

**类签名**:
```cpp
class BatterySrvClient final : public DelayedRefSingleton<BatterySrvClient> {
public:
    // 电池信息查询接口
    int32_t GetCapacity();
    BatteryChargeState GetChargingStatus();
    BatteryHealthState GetHealthStatus();
    BatteryPluggedType GetPluggedType();
    int32_t GetVoltage();
    bool GetPresent();
    std::string GetTechnology();
    int32_t GetBatteryTemperature();
    int32_t GetNowCurrent();
    int32_t GetRemainEnergy();
    int32_t GetTotalEnergy();
    BatteryCapacityLevel GetCapacityLevel();
    int64_t GetRemainingChargeTime();

    // 配置管理接口
    BatteryError SetBatteryConfig(const std::string& sceneName, const std::string& value);
    BatteryError GetBatteryConfig(const std::string& sceneName, std::string& result);
    BatteryError IsBatteryConfigSupported(const std::string& sceneName, bool& result);

    // 内部实现（不暴露给外部）
    bool Connect();
    void ResetProxy(const wptr<IRemoteObject>& remote);

private:
    sptr<IBatterySrv> Connect();
    void ResetProxy(const wptr<IRemoteObject>& remote);
    sptr<IBatterySrv> proxy_ {nullptr};
    sptr<IRemoteObject::DeathRecipient> deathRecipient_ {nullptr};
    std::mutex mutex_;
};
```

**证据**: `interfaces/inner_api/native/include/battery_srv_client.h:29-121`

### 接口稳定性

| 接口方法 | 稳定性说明 | 证据 |
|----------|----------|------|
| 所有 Get*() 方法 | **稳定** | 公开的 Inner API，长期稳定 | `interfaces/inner_api/native/include/battery_srv_client.h:38-101` |
| SetBatteryConfig() | **稳定** | 公开的 Inner API，长期稳定 | `interfaces/inner_api/native/include/battery_srv_client.h:93` |
| GetBatteryConfig() | **稳定** | 公开的 Inner API，长期稳定 | `interfaces/inner_api/native/include/battery_srv_client.h:97` |
| IsBatteryConfigSupported() | **稳定** | 公开的 Inner API，长期稳定 | `interfaces/inner_api/native/include/battery_srv_client.h:101` |
| Connect() | **不稳定** | 私有实现，可能随架构变更 | `interfaces/inner_api/native/include/battery_srv_client.h:116` |
| ResetProxy() | **不稳定** | 私有实现，可能随架构变更 | `interfaces/inner_api/native/include/battery_srv_client.h:117` |

### 依赖方向

```
BatterySrvClient
    ↓ 依赖
IBatterySrv (ZIDL 接口)
    ↓ 生成自
IBatterySrv.idl
    ↓ 实现于
BatteryService
```

**证据**:
- 依赖: `interfaces/inner_api/BUILD.gn:31`
- IDL: `services/zidl/IBatterySrv.idl:17-35`

### 连接管理

```cpp
sptr<IBatterySrv> BatterySrvClient::Connect() {
    std::lock_guard<std::mutex> lock(mutex_);

    if (proxy_ != nullptr) {
        return proxy_;
    }

    sptr<ISystemAbilityManager> sysMgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    sptr<IRemoteObject> remoteObject_ = sysMgr->CheckSystemAbility(POWER_MANAGER_BATT_SERVICE_ID);
    proxy_ = iface_cast<IBatterySrv>(remoteObject_);
    deathRecipient_ = new BatterySrvDeathRecipient(*this);
    remoteObject_->AddDeathRecipient(deathRecipient_);

    return proxy_;
}

void BatterySrvClient::ResetProxy(const wptr<IRemoteObject>& remote) {
    std::lock_guard<std::mutex> lock(mutex_);
    if (remote == proxy_) {
        proxy_ = nullptr;
    }
}
```

**证据**: `frameworks/native/src/battery_srv_client.cpp:35-65`

---

## BatteryService 接口

### 类定义

**文件路径**: `services/native/include/battery_service.h`

**类签名**:
```cpp
class BatteryService final : public SystemAbility,
    public BatterySrvStub {
public:
    // SystemAbility 生命周期
    virtual void OnStart() override;
    virtual void OnStop() override;
    virtual void OnAddSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;

    // 服务状态查询
    bool IsServiceReady() const;
    bool IsBootCompleted() const;

    // Dump 接口
    int32_t Dump(int fd, const std::vector<std::u16string> &args) override;

    // ZIDL 接口实现（公开给 IPC 调用）
    int32_t GetCapacity(int32_t& capacity) override;
    int32_t GetChargingStatus(uint32_t& chargeState) override;
    int32_t GetHealthStatus(uint32_t& healthState) override;
    int32_t GetPluggedType(uint32_t& pluggedType) override;
    int32_t GetVoltage(int32_t& voltage) override;
    int32_t GetPresent(bool& present) override;
    int32_t GetTechnology(std::string& technology) override;
    int32_t GetTotalEnergy(int32_t& totalEnergy) override;
    int32_t GetCurrentAverage(int32_t& curAverage) override;
    int32_t GetNowCurrent(int32_t& nowCurr) override;
    int32_t GetRemainEnergy(int32_t& remainEnergy) override;
    int32_t GetBatteryTemperature(int32_t& temperature) override;
    int32_t GetCapacityLevel(uint32_t& batteryCapacityLevel) override;
    int32_t GetRemainingChargeTime(int64_t& remainTime) override;
    int32_t SetBatteryConfig(const std::string& sceneName, const std::string& value, int32_t& batteryErr) override;
    int32_t GetBatteryConfig(const std::string& sceneName, std::string& result, int32_t& batteryErr) override;
    int32_t IsBatteryConfigSupported(const std::string& featureName, bool& result, int32_t& batteryErr) override;

    // 内部实现（不暴露给外部）
    void InitConfig();
    void HandleTemperature(int32_t temperature);
    bool RegisterHdiStatusListener();
    bool RegisterBatteryHdiCallback();
    int32_t HandleBatteryCallbackEvent(const HDI::Battery::V2_0::BatteryInfo& event);

private:
    bool Init();
    void AddBootCommonEvents();
    bool FillCommonEvent(std::string& ueventName, std::string& type);
    void WakeupDevice(BatteryChargeState chargeState);
    void RegisterBootCompletedCallback();
    void ConvertingEvent(const OHOS::HDI::Battery::V2_0::BatteryInfo &event);
    void InitBatteryInfo();
    void HandleBatteryInfo();
    void CalculateRemainingChargeTime(int32_t capacity, BatteryChargeState chargeState);
    void HandleCapacity(int32_t capacity, BatteryChargeState chargeState, bool isBatteryPresent);
    bool IsLastPlugged();
    bool IsNowPlugged(BatteryPluggedType pluggedType);
    bool IsPlugged(BatteryPluggedType pluggedType);
    bool IsUnplugged(BatteryPluggedType pluggedType);
    void WakeupDevice(BatteryPluggedType pluggedType);
    bool IsCharging(BatteryChargeState chargeState);
    bool IsInExtremePowerSaveMode();
    void CreateShutdownGuard();
    void LockShutdownGuard();
    void UnlockShutdownGuard();
    void SetLowCapacityThreshold();
    void ClearLowCapacityShutdownTask();
    void SubscribeHibernateCommonEvent();
    void UnsubscribeHibernateCommonEvent();

    // 成员变量
    bool ready_ { false };
    static std::atomic_bool isBootCompleted_;
    std::shared_mutex mutex_;
    std::unique_ptr<BatteryNotify> batteryNotify_ { nullptr };
    BatteryLight batteryLight_;
    sptr<HDI::Battery::V2_0::IBatteryInterface> iBatteryInterface_ { nullptr };
    sptr<OHOS::HDI::ServiceManager::V1_0::IServiceManager> hdiServiceMgr_ { nullptr };
    sptr<HdiServiceStatusListener::IServStatListener> hdiServStatListener_ { nullptr };
    bool isLowPower_ { false };
    int32_t lastCapacity_ { 0 };
    int64_t remainTime_ { 0 };
    BatteryInfo batteryInfo_;
    BatteryInfo lastBatteryInfo_;
    std::mutex shutdownGuardMutex_;
};
```

**证据**: `services/native/include/battery_service.h:51-210`

### 接口稳定性

| 接口方法 | 稳定性说明 | 证据 |
|----------|----------|------|
| 所有 Get*() 方法 | **稳定** | ZIDL 接口定义，IPC 接口 | `services/zidl/IBatterySrv.idl:18-35` |
| SetBatteryConfig() | **稳定** | ZIDL 接口定义 | `services/zidl/IBatterySrv.idl:32` |
| GetBatteryConfig() | **稳定** | ZIDL 接口定义 | `services/zidl/IBatterySrv.idl:33` |
| IsBatteryConfigSupported() | **稳定** | ZIDL 接口定义 | `services/zidl/IBatterySrv.idl:34` |
| Dump() | **稳定** | SystemAbility 标准接口 | `services/native/include/battery_service.h:71` |
| OnStart/OnStop/OnAddSystemAbility | **稳定** | SystemAbility 生命周期接口 | `services/native/include/battery_service.h:57-59` |

### 依赖方向

```
BatteryService
    ↓ 继承
SystemAbility + BatterySrvStub
    ↓ 生成自
IBatterySrv.idl + SystemAbility 框架
    ↓ 依赖
IBatteryInterface (HDI)
    ↓ 依赖
HDI::Battery::V2_0::IBatteryInterface (底层驱动)
    ↓ 依赖
BatteryNotify (通知管理)
    ↓ 依赖
HdiServiceStatusListener (HDI 服务状态监听)
```

**证据**: `services/native/include/battery_service.h:33-48`

---

## BatteryNotify 接口

### 类定义

**文件路径**: `services/native/include/battery_notify.h`

**类签名**:
```cpp
class BatteryNotify {
public:
    explicit BatteryNotify(sptr<OHOS::HDI::Battery::V2_0::IBatteryInterface> iBatteryInterface);
    ~BatteryNotify();

    void PublishChangedEventInner();
    void PublishCustomEvent(const std::string& eventName, const std::string& key, const std::string& value);
    void PublishShutdownEvent();
    void UpdateBatteryInfo();
    void SetSubscriberPermissions();
    void ClearSubscriberPermissions();

private:
    void PublishEvent(const std::string& eventName, const std::map<std::string, std::string>& data);
    void UpdateCommonEventData(std::map<std::string, std::string>& data);

    sptr<OHOS::HDI::Battery::V2_0::IBatteryInterface> iBatteryInterface_;
    BatteryInfo batteryInfo_;
};
```

**证据**: `services/native/include/battery_notify.h`

### 接口稳定性

| 接口方法 | 稳定性说明 |
|----------|----------|
| PublishChangedEventInner() | **稳定** | 内部通知接口，长期稳定 |
| PublishCustomEvent() | **稳定** | 内部通知接口，长期稳定 |
| PublishShutdownEvent() | **稳定** | 内部通知接口，长期稳定 |
| UpdateBatteryInfo() | **稳定** | 内部接口，长期稳定 |
| SetSubscriberPermissions() | **稳定** | 内部接口，长期稳定 |

### 依赖方向

```
BatteryNotify
    ↓ 依赖
OHOS::HDI::Battery::V2_0::IBatteryInterface (获取电池信息)
    ↓ 使用
CommonEventService (发布事件)
```

**证据**: `services/native/include/battery_notify.h`

---

## BatteryConfig 接口

### 类定义

**文件路径**: `services/native/include/battery_config.h`

**类签名**:
```cpp
class BatteryConfig {
public:
    explicit BatteryConfig(sptr<OHOS::HDI::Battery::V2_0::IBatteryInterface> iBatteryInterface);
    ~BatteryConfig();

    void InitConfig();
    bool SetConfigValue(const std::string& sceneName, const std::string& value);
    std::string GetConfigValue(const std::string& sceneName);
    bool IsBatteryConfigSupportedInner(const std::string& featureName);
    BatteryError GetBatteryConfigInner(const std::string& sceneName, std::string& result);

private:
    void UpdateTemperatureThreshold(int32_t temperature);
    void UpdateCapacityThreshold(int32_t capacity);
    void ParseConfigFile();

    std::mutex configMutex_;
    std::map<std::string, std::string> configMap_;
    BatteryInfo batteryInfo_;
    int32_t highTemperature_ { INT32_MAX };
    int32_t lowTemperature_ { INT32_MIN };
};
```

**证据**: `services/native/include/battery_config.h`

### 接口稳定性

| 接口方法 | 稳定性说明 |
|----------|----------|
| SetConfigValue() | **稳定** | 内部配置接口，长期稳定 |
| GetConfigValue() | **稳定** | 内部配置接口，长期稳定 |
| IsBatteryConfigSupportedInner() | **稳定** | 内部配置接口，长期稳定 |
| GetBatteryConfigInner() | **稳定** | 内部配置接口，长期稳定 |

---

## 依赖关系图

### 模块依赖方向

```
┌─────────────────────────────────────────────┐
│         应用层                         │
│  - N-API (batteryInfo, battery)         │
│  - C API (ohbattery_info)              │
│  - CJ FFI (battery_info_ffi)          │
└──────────────────┬──────────────────────────┘
                 │ 依赖
┌──────────────────┴──────────────────────────┐
│    BatterySrvClient (客户端)            │
│  ┌──────────────────────────────────┐  │
│  │ 依赖 IBatterySrv (ZIDL)    │  │
│  └──────────────────────────────────┘  │
└──────────────────┬──────────────────────────┘
                 │ Binder IPC
┌──────────────────┴──────────────────────────┐
│     BatteryService (服务端 SA 3302)    │
│  ┌──────────────────────────────────┐  │
│  │ 继承 SystemAbility +        │  │
│  │    BatterySrvStub             │  │
│  └──────────────────────────────────┘  │
│                                         │
├───────────────────────────────────────────┤
│ 依赖模块                              │
│  - BatteryNotify (通知管理）            │
│  - BatteryLight (LED 控制）            │
│  - BatteryConfig (配置管理）            │
│  - BatteryCallback (HDI 回调）          │
│  - HdiServiceStatusListener             │
└───────────────────────────────────────────┘
│                                         │
└──────────────────┬───────────────────────────┘
                 │ HDI 调用
┌──────────────────┴──────────────────────────┐
│     HDI Battery Interface (驱动层）      │
└─────────────────────────────────────────────┘
```

**证据**: 综合各模块头文件

### 接口可替换性

| 接口层 | 可替换性 | 说明 |
|----------|----------|------|
| N-API 层 | **不可替换** | JS API 绑定，模块化设计 |
| BatterySrvClient | **可替换** | IPC 客户端可独立替换 |
| BatteryService | **不可替换** | SA 主类，系统核心 |
| BatteryNotify | **可替换** | 通知管理模块可独立替换 |
| BatteryConfig | **可替换** | 配置管理模块可独立替换 |
| BatteryLight | **可替换** | LED 控制模块可独立替换 |

---

## 线程安全

### 互斥锁保护

```cpp
// BatterySrvClient 连接保护
class BatterySrvClient {
    std::mutex mutex_;  // 保护 proxy_ 和 deathRecipient_
};

// BatteryService 状态保护
class BatteryService {
    std::shared_mutex mutex_;  // 保护电池信息更新
};

// BatteryConfig 配置保护
class BatteryConfig {
    std::mutex configMutex_;  // 保护配置读写
};
```

**证据**: 各类定义中的 mutex_ 成员

### 线程模型

| 组件 | 线程模型 | 说明 |
|------|----------|------|
| BatteryService | 主线程 + Binder 线程池 | 作为 SA 在主线程运行，IPC 在 Binder 线程池处理 |
| BatterySrvClient | 多线程安全 | 使用 mutex 保护连接状态 |
| BatteryNotify | 多线程安全 | 使用 shared_mutex 保护状态 |
| BatteryConfig | 多线程安全 | 使用 mutex 保护配置 |

---

## 资源管理

### 单例模式

**BatterySrvClient** 使用 `DelayedRefSingleton` 确保全局唯一实例。

**证据**: `interfaces/inner_api/native/include/battery_srv_client.h:29`

### 生命周期管理

| 组件 | 创建 | 销毁 | 说明 |
|------|------|------|------|
| BatterySrvClient | 延迟初始化 | 应用退出时清理 | 单例模式，延迟初始化 |
| BatteryService | SA OnStart() | SA OnStop() | SA 生命周期管理 |
| BatteryNotify | 构造函数 | 析构函数 | 依赖 HDI 接口 |
| BatteryConfig | 构造函数 | 析构函数 | 依赖 HDI 接口 |

---

## 相关跳转

- [系统架构](03_Architecture.md)
- [N-API 文档](04_NAPI_API.md)

---

**返回**: [导航](SUMMARY.md)
