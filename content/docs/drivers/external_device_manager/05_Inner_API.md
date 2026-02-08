# 内部模块接口参考

本文档描述扩展外部设备管理模块的内部 C++ 接口，供系统组件集成和扩展开发使用。内部接口主要在 `interfaces/innerkits/` 和 `services/` 目录下定义，包括 IPC 接口、数据类型和核心服务类。

## 内部接口概览

内部接口主要分为以下几类：IPC 接口定义（IDL 文件）、客户端 API（DriverExtMgrClient）、数据类型定义（Parcelable 类）和核心服务类。这些接口仅供系统组件使用，不直接暴露给三方应用。

## IPC 接口定义

### IDriverExtMgr 接口

主服务接口，定义在 `IDriverExtMgr.idl` 文件中，描述了 Manager SA 对外提供的 RPC 方法。

```idl
interface OHOS.ExternalDeviceManager.IDriverExtMgr {
    void QueryDevice([out] int errorCode, [in] unsigned int busType, 
        [out] sharedptr<DeviceData>[] devices);
    
    void BindDevice([out] int errorCode, [in] unsigned long deviceId, 
        [in] IDriverExtMgrCallback connectCallback);
    
    void UnBindDevice([out] int errorCode, [in] unsigned long deviceId);
    
    void BindDriverWithDeviceId([out] int errorCode, [in] unsigned long deviceId, 
        [in] IDriverExtMgrCallback connectCallback);
    
    void UnBindDriverWithDeviceId([out] int errorCode, [in] unsigned long deviceId);
    
    void QueryDeviceInfo([out] int errorCode, 
        [out] sharedptr<DeviceInfoData>[] deviceInfos, 
        [in] boolean isByDeviceId, [in] unsigned long deviceId);
    
    void QueryDriverInfo([out] int errorCode, 
        [out] sharedptr<DriverInfoData>[] driverInfos, 
        [in] boolean isByDriverUid, [in] String driverUid);
    
    void NotifyUsbPeripheralFault([in] String domain, [in] String faultName);
}
```

### IDriverExtMgrCallback 接口

回调接口，定义在 `IDriverExtMgrCallback.idl` 文件中，描述了设备绑定结果回调方法。

```idl
[callback] interface OHOS.ExternalDeviceManager.IDriverExtMgrCallback {
    [oneway] void OnConnect([in] unsigned long deviceId, 
        [in] IRemoteObject drvExtObj, [in] ErrMsg errMsg);
    
    [oneway] void OnDisconnect([in] unsigned long deviceId, 
        [in] ErrMsg errMsg);
    
    [oneway] void OnUnBind([in] unsigned long deviceId, 
        [in] ErrMsg errMsg);
}
```

## 客户端 API

### DriverExtMgrClient 类

客户端封装类，定义在 `driver_ext_mgr_client.h` 文件中，封装了获取 SA 代理和调用 SA 方法的逻辑class DriverExtMgrClient final :。

```cpp
 public DelayedRefSingleton<DriverExtMgrClient> {
public:
    DISALLOW_COPY_AND_MOVE(DriverExtMgrClient);
    
    UsbErrCode QueryDevice(uint32_t busType, 
        std::vector<std::shared_ptr<DeviceData>> &devices);
    
    UsbErrCode BindDevice(uint64_t deviceId, 
        const sptr<IDriverExtMgrCallback> &connectCallback);
    
    UsbErrCode UnBindDevice(uint64_t deviceId);
    
    UsbErrCode BindDriverWithDeviceId(uint64_t deviceId, 
        const sptr<IDriverExtMgrCallback> &connectCallback);
    
    UsbErrCode UnbindDriverWithDeviceId(uint64_t deviceId);
    
    UsbErrCode QueryDeviceInfo(
        std::vector<std::shared_ptr<DeviceInfoData>> &deviceInfos);
    
    UsbErrCode QueryDeviceInfo(const uint64_t deviceId, 
        std::vector<std::shared_ptr<DeviceInfoData>> &deviceInfos);
    
    UsbErrCode QueryDriverInfo(
        std::vector<std::shared_ptr<DriverInfoData>> &driverInfos);
    
    UsbErrCode QueryDriverInfo(const std::string &driverUid, 
        std::vector<std::shared_ptr<DriverInfoData>> &driverInfos);
    
    UsbErrCode NotifyUsbPeripheralFault(const std::string &domain, 
        const std::string &faultName);

private:
    UsbErrCode Connect();
    void DisConnect(const wptr<IRemoteObject> &remote);
    
    class DriverExtMgrDeathRecipient : public IRemoteObject::DeathRecipient {
    public:
        void OnRemoteDied(const wptr<IRemoteObject> &remote);
    };
    
    std::mutex mutex_;
    sptr<IDriverExtMgr> proxy_ {nullptr};
    sptr<IRemoteObject::DeathRecipient> deathRecipient_ {nullptr};
};
```

## 数据类型定义

### Parcelable 数据类型

模块定义了一组 Parcelable 数据类型，用于 IPC 序列化和跨进程传输。

**ErrMsg 结构**

错误消息结构，包含错误码和错误信息。

```cpp
struct ErrMsg : public Parcelable {
    ErrMsg(UsbErrCode code = UsbErrCode::EDM_NOK, 
        const std::string &message = "") 
        : errCode(code), msg(message) {}
    
    bool IsOk() const { return errCode == UsbErrCode::EDM_OK; }
    
    bool Marshalling(Parcel &parcel) const;
    static ErrMsg* Unmarshalling(Parcel &data);
    
    UsbErrCode errCode;
    std::string msg;
};
```

**DeviceData 类**

设备数据基类，包含设备的基本信息。

```cpp
class DeviceData : public Parcelable {
public:
    virtual ~DeviceData() = default;
    
    virtual bool Marshalling(Parcel &parcel) const;
    static DeviceData* Unmarshalling(Parcel &data);
    virtual std::string Dump();
    
    BusType busType;
    uint64_t deviceId;
    std::string descripton;
};
```

**USBDevice 类**

USB 设备数据，继承自 DeviceData，包含 USB 特有属性。

```cpp
class USBDevice : public DeviceData {
public:
    virtual ~USBDevice() = default;
    
    virtual bool Marshalling(Parcel &parcel) const override;
    static USBDevice* Unmarshalling(Parcel &data);
    std::string Dump() override;
    
    uint16_t productId;
    uint16_t vendorId;
};
```

**DeviceInfoData 类**

设备信息类，包含设备的详细信息和匹配状态。

```cpp
class DeviceInfoData : public Parcelable {
public:
    virtual ~DeviceInfoData() = default;
    virtual bool Marshalling(Parcel &parcel) const;
    static DeviceInfoData* Unmarshalling(Parcel &data);
    static BusType GetBusTypeByDeviceId(uint64_t deviceId);
    
    uint64_t deviceId;
    bool isDriverMatched = false;
    std::string driverUid = "";
};
```

**USBDeviceInfoData 类**

USB 设备信息类，继承自 DeviceInfoData，包含 USB 特有信息和接口描述。

```cpp
class USBDeviceInfoData : public DeviceInfoData {
public:
    virtual ~USBDeviceInfoData() = default;
    
    virtual bool Marshalling(Parcel &parcel) const override;
    static USBDeviceInfoData* Unmarshalling(Parcel &data);
    
    uint16_t productId;
    uint16_t vendorId;
    std::vector<std::shared_ptr<USBInterfaceDesc>> interfaceDescList;
};
```

**USBInterfaceDesc 类**

USB 接口描述类，包含接口的描述符信息。

```cpp
class USBInterfaceDesc {
public:
    virtual ~USBInterfaceDesc() = default;
    
    bool Marshalling(Parcel &parcel) const;
    static std::shared_ptr<USBInterfaceDesc> Unmarshalling(Parcel &data);
    
    uint8_t bInterfaceNumber;
    uint8_t bClass;
    uint8_t bSubClass;
    uint8_t bProtocol;
};
```

**DriverInfoData 类**

驱动信息类，包含驱动的元数据信息。

```cpp
class DriverInfoData : public Parcelable {
public:
    virtual ~DriverInfoData() = default;
    virtual bool Marshalling(Parcel &parcel) const;
    static DriverInfoData* Unmarshalling(Parcel &data);
    
    BusType busType;
    std::string driverUid;
    std::string driverName;
    std::string bundleSize;
    std::string version;
    std::string description;
};
```

**USBDriverInfoData 类**

USB 驱动信息类，继承自 DriverInfoData，包含 USB 驱动支持的 VID/PID 列表。

```cpp
class USBDriverInfoData : public DriverInfoData {
public:
    virtual ~USBDriverInfoData() = default;
    
    virtual bool Marshalling(Parcel &parcel) const override;
    static USBDriverInfoData* Unmarshalling(Parcel &data);
    
    std::vector<uint16_t> pids;
    std::vector<uint16_t> vids;
};
```

## 核心服务类

### DriverExtMgr 类

主服务类，继承自 SystemAbility 和 DriverExtMgrStub，以延迟单例模式运行。

```cpp
class DriverExtMgr : public SystemAbility, public DriverExtMgrStub {
    DECLARE_SYSTEM_ABILITY(DriverExtMgr)
    DECLARE_DELAYED_SINGLETON(DriverExtMgr);

public:
    void OnStart() override;
    void OnStop() override;
    int Dump(int fd, const std::vector<std::u16string> &args) override;
    void OnAddSystemAbility(int32_t systemAbilityId, 
        const std::string &deviceId) override;
    
    // IPC 接口实现
    virtual ErrCode QueryDevice(int32_t &errorCode, uint32_t busType,
        std::vector<std::shared_ptr<DeviceData>> &devices) override;
    
    virtual ErrCode BindDevice(int32_t &errorCode, uint64_t deviceId,
        const sptr<IDriverExtMgrCallback> &connectCallback) override;
    
    virtual ErrCode UnBindDevice(int32_t &errorCode, uint64_t deviceId) override;
    
    virtual ErrCode BindDriverWithDeviceId(int32_t &errorCode, uint64_t deviceId,
        const sptr<IDriverExtMgrCallback> &connectCallback) override;
    
    virtual ErrCode UnBindDriverWithDeviceId(int32_t &errorCode, 
        uint64_t deviceId) override;
    
    virtual ErrCode QueryDeviceInfo(int32_t &errorCode, 
        std::vector<std::shared_ptr<DeviceInfoData>> &deviceInfos,
        bool isByDeviceId = false, const uint64_t deviceId = 0) override;
    
    virtual ErrCode QueryDriverInfo(int32_t &errorCode, 
        std::vector<std::shared_ptr<DriverInfoData>> &driverInfos,
        bool isByDriverUid = false, const std::string &driverUid = "") override;
    
    virtual ErrCode NotifyUsbPeripheralFault(const std::string &domain, 
        const std::string &faultName) override;

private:
    std::mutex connectCallbackMutex;
    std::map<uint64_t, std::vector<sptr<IDriverExtMgrCallback>>> connectCallbackMap;
};
```

### ExtDeviceManager 类

设备管理器单例类，负责设备的注册、查询和连接管理。

```cpp
class ExtDeviceManager final {
    DECLARE_SINGLE_INSTANCE_BASE(ExtDeviceManager);

public:
    ~ExtDeviceManager();
    int32_t Init();
    
    int32_t RegisterDevice(shared_ptr<DeviceInfo> devInfo);
    int32_t UnRegisterDevice(const shared_ptr<DeviceInfo> devInfo);
    
    vector<shared_ptr<DeviceInfo>> QueryDevice(const BusType busType);
    vector<shared_ptr<Device>> QueryAllDevices();
    vector<shared_ptr<Device>> QueryDevicesById(const uint64_t deviceId);
    
    int32_t ConnectDevice(uint64_t deviceId, uint32_t callingTokenId,
        const sptr<IDriverExtMgrCallback> &connectCallback);
    int32_t DisConnectDevice(uint64_t deviceId, uint32_t callingTokenId);
    
    int32_t ConnectDriverWithDeviceId(uint64_t deviceId, uint32_t callingTokenId,
        const unordered_set<std::string> &accessibleBundles, 
        const sptr<IDriverExtMgrCallback> &connectCallback);
    
    int32_t DisConnectDriverWithDeviceId(uint64_t deviceId, uint32_t callingTokenId);
    
    void RemoveDeviceOfDeviceMap(shared_ptr<Device> device);
    std::unordered_set<uint64_t> DeleteBundlesOfBundleInfoMap(
        const std::string &bundleName = "");
    
    void MatchDriverInfos(std::unordered_set<uint64_t> deviceIds);
    void ClearMatchedDrivers(const int32_t userId);
    void SetDriverChangeCallback(
        shared_ptr<IDriverChangeCallback> &driverChangeCallback);

private:
    std::shared_ptr<Device> QueryDeviceByDeviceID(uint64_t deviceId);
    int32_t CheckAccessPermission(const std::shared_ptr<DriverInfo> &driverInfo,
        const unordered_set<std::string> &accessibleBundles) const;
    void UnLoadSelf(void);
    
    unordered_map<BusType, unordered_map<uint64_t, shared_ptr<Device>>> deviceMap_;
    unordered_map<string, unordered_set<uint64_t>> bundleMatchMap_;
    mutex deviceMapMutex_;
    mutex bundleMatchMapMutex_;
};
```

### Device 类

设备抽象类，表示物理设备及其连接状态。

```cpp
class Device : public std::enable_shared_from_this<Device> {
public:
    Device(uint64_t deviceId, BusType busType);
    virtual ~Device();
    
    uint64_t GetDeviceId() const;
    BusType GetBusType() const;
    
    virtual int32_t Connect(const sptr<IRemoteObject> &remote);
    virtual int32_t Disconnect();
    
    bool HasDriver() const;
    shared_ptr<DriverInfo> GetDriver() const;
    int32_t AddBundleInfo(const string &bundleInfo);
    
    virtual std::string Dump();
    
protected:
    uint64_t deviceId_;
    BusType busType_;
    bool isConnected_;
    shared_ptr<DriverInfo> matchedDriver_;
    unordered_set<string> boundBundles_;
    mutex deviceMutex_;
};
```

### DriverExtensionController 类

驱动扩展控制器，负责驱动的生命周期管理。

```cpp
class DriverExtensionController {
    DECLARE_SINGLE_INSTANCE_BASE(DriverExtensionController);

public:
    int32_t StartDriverExtension(const string& bundleName, 
        const string& abilityName);
    
    int32_t StopDriverExtension(const string& bundleName, 
        const string& abilityName, int32_t userId);
    
    int32_t ConnectDriverExtension(const string& bundleName, 
        const string& abilityName, 
        shared_ptr<IDriverExtensionConnectCallback> callback, 
        uint32_t deviceId);
    
    int32_t DisconnectDriverExtension(const string& bundleName, 
        const string& abilityName, 
        shared_ptr<IDriverExtensionConnectCallback> callback, 
        uint32_t deviceId);

private:
    class AbilityConnectionStub : public OHOS::AAFwk::AbilityConnectionStub {
    public:
        void OnAbilityConnectDone(const OHOS::AppExecFwk::ElementName &element, 
            const sptr<IRemoteObject> &remoteObject, 
            int resultCode) override;
        
        sptr<IRemoteObject> remoteObject_;
        shared_ptr<IDriverExtensionConnectCallback> callback_;
        uint32_t deviceId_;
    };
};
```

### DriverPkgManager 类

驱动包管理器，负责驱动元数据的存储和查询。

```cpp
class DriverPkgManager {
    DECLARE_SINGLE_INSTANCE_BASE(DriverPkgManager);

public:
    int32_t Init();
    
    shared_ptr<DriverInfo> QueryMatchDriver(shared_ptr<DeviceInfo> devInfo, 
        const std::string &type);
    
    int32_t QueryDriverInfo(vector<shared_ptr<DriverInfo>> &driverInfos,
        bool isByDriverUid = false, const string &driverUid = "");
    
    int32_t RegisterBundleCallback(
        shared_ptr<IBundleUpdateCallback> callback);
    
    bool SubscribeOsAccountSwitch();

private:
    shared_ptr<PkgDatabase> pkgDb_;
    shared_ptr<DrvBundleStateCallback> bundleStateCallback_;
};
```

## 接口代码枚举

IPC 接口调用码定义在 `hdf_ext_devmgr_interface_code.h` 文件中：

```cpp
enum class DriverExtMgrInterfaceCode : uint32_t {
    QUERY_DEVICE = 1,
    BIND_DEVICE,
    UNBIND_DEVICE,
    BIND_DRIVER_WITH_DEVICE_ID,
    UNBIND_DRIVER_WITH_DEVICE_ID,
    QUERY_DEVICE_INFO,
    QUERY_DRIVER_INFO,
    NOTIFY_USB_PERIPHERAL_FAULT,
    INVALID_CODE
};
```

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [06_Build_System.md](./06_Build_System.md) | GN 构建系统说明 |
