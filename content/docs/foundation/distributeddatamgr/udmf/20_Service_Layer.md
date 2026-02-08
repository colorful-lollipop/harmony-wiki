# 服务层架构

## 架构概述

UDMF 服务层是基于 OpenHarmony IPC 框架实现的分布式数据管理服务，负责处理跨进程数据操作请求、权限校验和数据持久化。服务层采用代理-存根（Proxy-Stub）模式，通过 System Ability Manager（SAMGR）注册系统服务，供其他进程通过 IPC 调用访问。

根据源码证据（`framework/innerkitsimpl/service/`），服务层包含两个主要服务：`UdmfService`（统一数据服务）和 `UtdService`（统一类型描述符服务）。两个服务分别提供数据操作和类型管理的核心能力，通过统一的 IPC 接口暴露给客户端。

服务层架构遵循以下设计原则：首先，服务采用单例模式运行在系统进程中，保证数据一致性和访问控制；其次，所有 IPC 调用都经过参数序列化和权限校验，确保安全性；最后，服务支持进程死亡检测和自动恢复，保证服务可用性。

## 服务注册与发现

### System Ability 注册

UDMF 服务通过 SAMGR 注册为系统能力（`framework/innerkitsimpl/service/udmf_service_client.cpp`）：

```cpp
// 获取系统能力管理器
auto samgr = SystemAbilityManagerClient::GetSystemAbilityManager();

// 获取 UDMF 服务能力
auto remoteObj = samgr->GetSystemAbility(DISTRIBUTED_KV_DATA_SERVICE_ABILITY_ID);

// 转换为 UDMF 服务接口
auto udmfService = iface_cast<IUdmfService>(remoteObj);
```

### 服务标识

```cpp
// 服务能力 ID
static constexpr char DISTRIBUTED_KV_DATA_SERVICE_ABILITY_ID[] = 
    "DistributedKvDataServiceAbility";

// 服务接口码（IPC 调用使用）
enum class UdmfServiceInterfaceCode {
    SET_DATA = 0,
    GET_DATA,
    GET_BATCH_DATA,
    UPDATE_DATA,
    DELETE_DATA,
    GET_SUMMARY,
    ADD_PRIVILEGE,
    SYNC,
    IS_REMOTE_DATA,
    SET_APP_SHARE_OPTION,
    GET_APP_SHARE_OPTION,
    REMOVE_APP_SHARE_OPTION,
    OBTAIN_ASYN_PROCESS,
    CLEAR_ASYN_PROCESS_BY_KEY,
    SET_DELAY_INFO,
    PUSH_DELAY_DATA,
    GET_DATA_IF_AVAILABLE,
};
```

### 服务接口定义

**IUdmfService 接口**（`framework/innerkitsimpl/service/udmf_service_proxy.h:29-61`）：

```cpp
class IUdmfService : public UdmfService, public IRemoteBroker {
public:
    // 数据操作
    int32_t SetData(CustomOption &option, UnifiedData &unifiedData, 
                    std::string &key) override;
    int32_t GetData(const QueryOption &query, 
                    UnifiedData &unifiedData) override;
    int32_t GetBatchData(const QueryOption &query, 
                         std::vector<UnifiedData> &unifiedDataSet) override;
    int32_t UpdateData(const QueryOption &query, 
                        UnifiedData &unifiedData) override;
    int32_t DeleteData(const QueryOption &query, 
                        std::vector<UnifiedData> &unifiedDataSet) override;
    
    // 元数据操作
    int32_t GetSummary(const QueryOption &query, 
                       Summary &summary) override;
    int32_t AddPrivilege(const QueryOption &query, 
                          Privilege &privilege) override;
    
    // 同步与状态
    int32_t Sync(const QueryOption &query, 
                 const std::vector<std::string> &devices) override;
    int32_t IsRemoteData(const QueryOption &query, 
                          bool &result) override;
    
    // 共享选项
    int32_t SetAppShareOption(const std::string &intention, 
                               int32_t shareOption) override;
    int32_t GetAppShareOption(const std::string &intention, 
                               int32_t &shareOption) override;
    int32_t RemoveAppShareOption(const std::string &intention) override;
    
    // 异步处理
    int32_t ObtainAsynProcess(AsyncProcessInfo &processInfo) override;
    int32_t ClearAsynProcessByKey(const std::string &businessUdKey) override;
    
    // 延迟数据
    int32_t SetDelayInfo(const DataLoadInfo &dataLoadInfo, 
                          sptr<IRemoteObject> iUdmfNotifier,
                          std::string &key) override;
    int32_t PushDelayData(const std::string &key, 
                          UnifiedData &unifiedData) override;
    int32_t GetDataIfAvailable(const std::string &key, 
                                 const DataLoadInfo &dataLoadInfo,
                                 sptr<IRemoteObject> iUdmfNotifier,
                                 std::shared_ptr<UnifiedData> unifiedData) override;
};
```

## 代理模式实现

### UdmfServiceProxy

`UdmfServiceProxy` 封装了 IPC 调用细节（`framework/innerkitsimpl/service/udmf_service_proxy.h:34-61`）：

```cpp
class UdmfServiceProxy : public IRemoteProxy<IUdmfService> {
public:
    explicit UdmfServiceProxy(const sptr<IRemoteObject> &object);

private:
    // 发送 IPC 请求
    int32_t SendRequest(UdmfServiceInterfaceCode code, 
                        MessageParcel &data,
                        MessageParcel &reply, 
                        MessageOption &option);

    static inline BrokerDelegator<UdmfServiceProxy> delegator_;
};
```

**IPC 请求发送示例**：

```cpp
int32_t UdmfServiceProxy::SetData(CustomOption &option, 
                                   UnifiedData &unifiedData,
                                   std::string &key)
{
    MessageParcel data;
    MessageParcel reply;
    MessageOption option(MessageOption::TF_SYNC);
    
    // 序列化参数
    if (!data.WriteInterfaceToken(GetDescriptor())) {
        return E_WRITE_PARCEL_ERROR;
    }
    if (!option.Marshalling(data) || 
        !unifiedData.Marshalling(data)) {
        return E_WRITE_PARCEL_ERROR;
    }
    
    // 发送请求
    int32_t ret = SendRequest(
        UdmfServiceInterfaceCode::SET_DATA, 
        data, reply, option);
    
    // 反序列化结果
    if (ret == E_OK) {
        key = reply.ReadString();
    }
    return ret;
}
```

### 进程死亡处理

`UdmfServiceClient` 实现进程死亡监听（`framework/innerkitsimpl/service/udmf_service_client.cpp`）：

```cpp
class ServiceDeathRecipient : public IRemoteObject::DeathRecipient {
public:
    void OnRemoteDied(const wptr<IRemoteObject> &remote) override {
        // 服务死亡，重置单例实例
        std::lock_guard<std::mutex> lock(mutex_);
        if (instance_ != nullptr && 
            instance_->AsObject().GetRefPtr() == remote.GetRefPtr()) {
            instance_.reset();
            kvDataServiceProxy_ = nullptr;
        }
    }
};

// 注册死亡通知
remoteObj->AddDeathRecipient(new ServiceDeathRecipient());
```

## UdmfServiceClient 单例

### 单例实现

`UdmfServiceClient` 是客户端访问服务的入口（`framework/innerkitsimpl/service/udmf_service_client.h`）：

```cpp
class UdmfServiceClient {
public:
    // 获取单例
    static std::shared_ptr<UdmfServiceClient> GetInstance();

private:
    // 构造函数私有化
    UdmfServiceClient() = default;
    
    // 初始化服务连接
    int32_t Init();
    
    // 获取分布式 KV 数据服务
    sptr<IRemoteObject> GetDistributedKvDataService();
};
```

### 初始化流程

```cpp
std::shared_ptr<UdmfServiceClient> UdmfServiceClient::GetInstance()
{
    static std::shared_ptr<UdmfServiceClient> instance = nullptr;
    static std::once_flag flag;
    
    std::call_once(flag, [&]() {
        instance.reset(new UdmfServiceClient());
        int32_t ret = instance->Init();
        if (ret != E_OK) {
            instance.reset();
        }
    });
    
    return instance;
}

int32_t UdmfServiceClient::Init()
{
    // 获取 SAMGR
    auto samgr = SystemAbilityManagerClient::GetSystemAbilityManager();
    if (samgr == nullptr) {
        return E_IPC;
    }
    
    // 获取 KV 数据服务
    auto kvRemote = samgr->GetSystemAbility(
        DISTRIBUTED_KV_DATA_SERVICE_ABILITY_ID);
    if (kvRemote == nullptr) {
        return E_IPC;
    }
    
    // 获取 UDMF 功能接口
    auto kvDataService = iface_cast<IKvStoreDataService>(kvRemote);
    auto udmfFeature = kvDataService->GetFeatureInterface("udmf");
    serviceProxy_ = iface_cast<IUdmfService>(udmfFeature);
    
    if (serviceProxy_ == nullptr) {
        return E_IPC;
    }
    
    // 注册死亡通知
    auto deathRecipient = new ServiceDeathRecipient();
    udmfFeature->AsObject()->AddDeathRecipient(deathRecipient);
    
    return E_OK;
}
```

## UtdService 服务

### 服务接口

`IUtdService` 提供类型管理能力（`framework/innerkitsimpl/service/utd_service_proxy.h`）：

```cpp
class IUtdService : public UtdService, public IRemoteBroker {
public:
    // 类型描述符查询
    int32_t GetTypeDescriptor(const std::string &typeId,
                               std::shared_ptr<TypeDescriptor> &descriptor);
    
    // 类型识别
    int32_t GetUniformDataTypeByFilenameExtension(
        const std::string &fileExtension, std::string &typeId);
    int32_t GetUniformDataTypeByMIMEType(
        const std::string &mimeType, std::string &typeId);
    
    // 类型注册
    int32_t RegisterTypeDescriptors(
        const std::vector<TypeDescriptorCfg> &descriptors);
    int32_t UnregisterTypeDescriptors(
        const std::vector<std::string> &typeIds);
    
    // 通知管理
    int32_t RegisterChangeNotifier(sptr<IRemoteObject> notifier);
};
```

### 服务客户端

`UtdServiceClient` 提供客户端访问接口（`framework/innerkitsimpl/service/utd_service_client.h/cpp`）：

```cpp
class UtdServiceClient {
public:
    static std::shared_ptr<UtdServiceClient> GetInstance();
    
private:
    std::shared_ptr<IUtdService> GetServiceProxy();
    
    sptr<IRemoteObject> utdServiceProxy_;
};
```

## 通知机制

### 数据变更通知

`IUdmfNotifier` 定义数据变更通知接口（`framework/innerkitsimpl/service/iudmf_notifier.h`）：

```cpp
class IUdmfNotifier : public IRemoteBroker {
public:
    // 数据变更通知
    virtual int32_t OnDataChanged(const std::string &businessUdKey) = 0;
    
    // 异步处理进度
    virtual int32_t OnProcessDied(const std::string &businessUdKey) = 0;
};

// 延迟数据回调
class IDelayDataCallback : public IRemoteBroker {
public:
    virtual int32_t OnDelayDataLoad(const std::string &businessUdKey,
                                     UnifiedData &unifiedData) = 0;
};
```

### UTD 变更通知

`IUtdNotifier` 定义 UTD 变更通知接口（`framework/innerkitsimpl/service/utd_notifier.h`）：

```cpp
class IUtdNotifier : public IRemoteBroker {
public:
    // UTD 变更通知
    virtual int32_t OnUtdChanged(
        const std::vector<std::string> &typeIds) = 0;
    
    // 自定义 UTD 变更
    virtual int32_t OnCustomUtdChanged(const std::string &bundleName,
                                       int32_t userId) = 0;
};
```

## 进度回调

`IProgressSignal` 定义异步操作进度回调（`framework/innerkitsimpl/service/progress_callback.h`）：

```cpp
class IProgressSignal : public IRemoteBroker {
public:
    // 进度更新
    virtual int32_t OnProgress(int32_t status, 
                               int32_t progress,
                               std::shared_ptr<UnifiedData> data) = 0;
};
```

## IPC 接口码

`distributeddata_udmf_ipc_interface_code.h` 定义所有 IPC 接口码：

```cpp
enum class UdmfServiceInterfaceCode : uint32_t {
    SET_DATA = 0,
    GET_DATA = 1,
    GET_BATCH_DATA = 2,
    UPDATE_DATA = 3,
    DELETE_DATA = 4,
    GET_SUMMARY = 5,
    ADD_PRIVILEGE = 6,
    SYNC = 7,
    IS_REMOTE_DATA = 8,
    SET_APP_SHARE_OPTION = 9,
    GET_APP_SHARE_OPTION = 10,
    REMOVE_APP_SHARE_OPTION = 11,
    OBTAIN_ASYN_PROCESS = 12,
    CLEAR_ASYN_PROCESS_BY_KEY = 13,
    SET_DELAY_INFO = 14,
    PUSH_DELAY_DATA = 15,
    GET_DATA_IF_AVAILABLE = 16,
};

enum class UtdServiceInterfaceCode : uint32_t {
    GET_TYPE_DESCRIPTOR = 0,
    GET_TYPE_BY_EXTENSION = 1,
    GET_TYPE_BY_MIME = 2,
    REGISTER_TYPES = 3,
    UNREGISTER_TYPES = 4,
    REGISTER_NOTIFIER = 5,
};
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口
- [21_Client_Layer.md](./21_Client_Layer.md)：客户端层架构
- [30_GN_Build_Targets.md](./30_GN_Build_Targets.md)：构建目标
