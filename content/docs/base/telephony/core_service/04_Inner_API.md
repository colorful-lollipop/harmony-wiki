# 内部 API (Inner Kits)

## 目的

本文档描述 `telephony_core_service` 提供的内部 C++ API，供系统应用和 Native 服务调用。

---

## 接口架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Native Application                       │
├─────────────────────────────────────────────────────────────┤
│              CoreServiceClient (Singleton)                  │
│         DelayedRefSingleton<CoreServiceClient>              │
├─────────────────────────────────────────────────────────────┤
│  ICoreService (IPC) │  ITelRilManager │  ISimManager ...   │
├─────────────────────────────────────────────────────────────┤
│  CoreServiceProxy │  TelRilManager │  SimManager ...       │
└─────────────────────────────────────────────────────────────┘
```

---

## CoreServiceClient

### 定位

**文件**: `interfaces/innerkits/include/core_service_client.h`  
**实现**: `frameworks/native/src/core_service_client.cpp`

### 设计模式

使用单例模式通过 `DelayedRefSingleton` 获取实例:

```cpp
DelayedRefSingleton<CoreServiceClient>::GetInstance().GetSimState(slotId, callback);
```

### 主要接口

#### SIM 管理

| 方法 | 签名 | 说明 |
|------|------|------|
| `GetSimState` | `int32_t GetSimState(int32_t slotId, const sptr<IRawParcelCallback> &callback)` | 获取 SIM 状态 |
| `HasSimCard` | `int32_t HasSimCard(int32_t slotId, const sptr<IRawParcelCallback> &callback)` | 是否有卡 |
| `GetSimIccId` | `int32_t GetSimIccId(int32_t slotId, std::u16string &iccId)` | 获取 ICCID |
| `GetIMSI` | `int32_t GetIMSI(int32_t slotId, std::u16string &imsi)` | 获取 IMSI |
| `GetSimOperatorNumeric` | `std::u16string GetOperatorNumeric(int32_t slotId)` | 获取 PLMN |
| `GetISOCountryCodeForSim` | `int32_t GetISOCountryCodeForSim(int32_t slotId, std::u16string &countryCode)` | 获取国家码 |
| `IsSimActive` | `bool IsSimActive(int32_t slotId, const sptr<IRawParcelCallback> &callback)` | 是否激活 |

#### 网络搜索

| 方法 | 签名 | 说明 |
|------|------|------|
| `GetPsRadioTech` | `int32_t GetPsRadioTech(int32_t slotId, int32_t &psRadioTech)` | 获取 PS 制式 |
| `GetCsRadioTech` | `int32_t GetCsRadioTech(int32_t slotId, int32_t &csRadioTech)` | 获取 CS 制式 |
| `GetSignalInfoList` | `int32_t GetSignalInfoList(int32_t slotId, std::vector<sptr<SignalInformation>> &signals)` | 获取信号信息 |
| `GetNetworkState` | `int32_t GetNetworkState(int32_t slotId, sptr<NetworkState> &networkState)` | 获取网络状态 |
| `SetRadioState` | `int32_t SetRadioState(int32_t slotId, bool isOn, const sptr<INetworkSearchCallback> &callback)` | 设置射频开关 |
| `GetRadioState` | `int32_t GetRadioState(int32_t slotId, const sptr<INetworkSearchCallback> &callback)` | 获取射频状态 |

#### 设备信息

| 方法 | 签名 | 权限 |
|------|------|------|
| `GetImei` | `int32_t GetImei(int32_t slotId, const sptr<IRawParcelCallback> &callback)` | GET_TELEPHONY_STATE |
| `GetMeid` | `int32_t GetMeid(int32_t slotId, std::u16string &meid)` | GET_TELEPHONY_STATE |
| `GetUniqueDeviceId` | `int32_t GetUniqueDeviceId(int32_t slotId, std::u16string &deviceId)` | GET_TELEPHONY_STATE |
| `GetBasebandVersion` | `int32_t GetBasebandVersion(int32_t slotId, std::string &version)` | - |

### 连接管理

```cpp
// 连接核心服务
bool ConnectService();

// 注册服务死亡监听
void RegisterCoreServiceDeathRecipient();

// 重新连接回调
void RegisterReconnectCallback(const std::function<void()> &callback);
```

---

## 接口类层次

### ICoreService

**文件**: `interfaces/innerkits/include/i_core_service.h`

```cpp
class ICoreService : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.telephony.ICoreService");
    
    virtual int32_t GetPsRadioTech(int32_t slotId, int32_t &psRadioTech) = 0;
    virtual int32_t GetSimState(int32_t slotId, const sptr<IRawParcelCallback> &callback) = 0;
    // ... 更多纯虚函数
};
```

### ISimManager

**文件**: `interfaces/innerkits/include/i_sim_manager.h`

```cpp
class ISimManager {
public:
    virtual ~ISimManager() = default;
    virtual int32_t OnInit(int32_t slotCount) = 0;
    virtual int32_t GetSimState(int32_t slotId) = 0;
    virtual int32_t GetSimIccId(int32_t slotId, std::u16string &iccId) = 0;
    // ...
};
```

### INetworkSearch

**文件**: `interfaces/innerkits/include/i_network_search.h`

```cpp
class INetworkSearch {
public:
    virtual ~INetworkSearch() = default;
    virtual bool OnInit() = 0;
    virtual int32_t GetPsRadioTech(int32_t slotId) = 0;
    virtual int32_t GetCsRadioTech(int32_t slotId) = 0;
    virtual int32_t GetSignalInfoList(int32_t slotId, std::vector<sptr<SignalInformation>> &signals) = 0;
    // ...
};
```

### ITelRilManager

**文件**: `interfaces/innerkits/include/i_tel_ril_manager.h`

```cpp
class ITelRilManager {
public:
    virtual ~ITelRilManager() = default;
    virtual bool OnInit() = 0;
    virtual bool DeInit() = 0;
    virtual int32_t RegisterCoreNotify(int32_t slotId, 
        const std::shared_ptr<AppExecFwk::EventHandler> &handler, int32_t what, int32_t *obj) = 0;
    virtual int32_t GetSimStatus(int32_t slotId, const AppExecFwk::InnerEvent::Pointer &response) = 0;
    // ...
};
```

---

## 回调接口

### INetworkSearchCallback

**文件**: `interfaces/innerkits/include/i_network_search_callback.h`

```cpp
class INetworkSearchCallback : public IRemoteBroker {
public:
    virtual int32_t OnNetworkSearchResult(const NetworkSearchResult &networkSearchResult) = 0;
    virtual int32_t OnGetNetworkSelectionModeResult(const NetworkSelectionMode &selectionMode) = 0;
    virtual int32_t OnSetNetworkSelectionModeResult(int32_t errorCode) = 0;
    // ...
};
```

### IRawParcelCallback

**文件**: `interfaces/innerkits/include/i_raw_parcel_callback.h`

```cpp
class IRawParcelCallback : public IRemoteBroker {
public:
    virtual int32_t OnCallback(int32_t errorCode, const MessageParcel &data) = 0;
};
```

### ImsRegInfoCallback

**文件**: `interfaces/innerkits/include/ims_reg_info_callback.h`

```cpp
class ImsRegInfoCallback : public IRemoteBroker {
public:
    virtual int32_t OnImsRegInfoChanged(int32_t slotId, ImsServiceType imsSrvType, const ImsRegInfo &info) = 0;
};
```

---

## 回调机制实现

### 注册回调流程

```cpp
// 1. 应用实现回调接口
class MyCallback : public INetworkSearchCallback {
public:
    int32_t OnNetworkSearchResult(const NetworkSearchResult &result) override {
        // 处理结果
        return 0;
    }
    // ...
};

// 2. 调用 API 并传入回调
sptr<MyCallback> callback = new MyCallback();
CoreServiceClient::GetInstance().GetNetworkSearchInformation(slotId, callback);
```

### 回调内部实现

**代码证据** (`frameworks/native/src/i_network_search_callback_stub.cpp`):

```cpp
int32_t INetworkSearchCallbackStub::OnRemoteRequest(
    uint32_t code, MessageParcel &data, MessageParcel &reply, MessageOption &option)
{
    switch (code) {
        case GET_NETWORK_SEARCH_RESULT:
            return OnNetworkSearchResultInner(data, reply);
        // ...
    }
}
```

---

## 数据结构

### NetworkState

**文件**: `interfaces/innerkits/include/network_state.h`

```cpp
class NetworkState : public Parcelable {
public:
    RegServiceState GetRegStatus() const;
    int32_t GetPsRadioTech() const;
    int32_t GetCsRadioTech() const;
    std::string GetOperatorNumeric() const;
    std::string GetOperatorName() const;
    // ...
private:
    RegServiceState regStatus_;
    int32_t psRadioTech_;
    int32_t csRadioTech_;
    // ...
};
```

### SignalInformation

**文件**: `interfaces/innerkits/include/signal_information.h`

```cpp
class SignalInformation : public Parcelable {
public:
    enum NetworkType { GSM, CDMA, LTE, WCDMA, TDSCDMA, NR };
    
    virtual int32_t GetSignalLevel() const = 0;
    virtual int32_t GetRssi() const = 0;
    // ...
};
```

### IccAccountInfo

**文件**: `interfaces/innerkits/include/sim_state_type.h`

```cpp
struct IccAccountInfo : public Parcelable {
    int32_t slotIndex;
    std::u16string iccId;
    std::u16string showName;
    std::u16string showNumber;
    int32_t simId;
    // ...
};
```

---

## 稳定性说明

| 接口层级 | 稳定性 | 说明 |
|----------|--------|------|
| CoreServiceClient | 稳定 | 公开接口，向后兼容 |
| ICoreService | 稳定 | IPC 接口，变更需同步 |
| ISimManager/INetworkSearch | 内部 | 仅内部使用，可能变更 |
| TelRilManager | 内部 | 仅内部使用 |

---

## 相关链接

- [N-API 接口](./03_NAPI_API.md)
- [架构设计](./02_Architecture.md)
- [GN 构建](./05_GN_Build.md)
