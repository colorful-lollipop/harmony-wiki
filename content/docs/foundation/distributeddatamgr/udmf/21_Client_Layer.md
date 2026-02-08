# 客户端层架构

## 架构概述

UDMF 客户端层是应用进程访问数据管理服务的中间件，提供高层 API 抽象和本地缓存能力。客户端层封装了 IPC 调用细节，实现数据本地缓存、类型管理、懒加载等核心功能，是应用使用 UDMF 的主要入口。

根据源码证据（`framework/innerkitsimpl/client/`），客户端层包含三个核心组件：`UdmfClient`（数据操作客户端）、`UtdClient`（类型管理客户端）和 `GetterSystem`（懒加载工厂）。三个组件均采用单例模式，保证进程内唯一实例。

客户端层的设计遵循以下原则：首先，通过单例模式提供全局访问点，简化 API 使用；其次，实现本地数据缓存，减少 IPC 调用开销；最后，支持延迟数据加载，优化大文件处理性能。

## UdmfClient 单例

### 单例实现

`UdmfClient` 是数据操作的高层 API 门面（`framework/innerkitsimpl/client/udmf_client.cpp`）：

```cpp
class UdmfClient {
public:
    // 获取单例实例
    static UdmfClient API_EXPORT &GetInstance();

private:
    // 构造函数私有化（单例模式）
    UdmfClient() = default;
    ~UdmfClient() = default;
    
    // 禁止拷贝
    UdmfClient(const UdmfClient &obj) = delete;
    UdmfClient &operator=(const UdmfClient &obj) = delete;
};
```

### 数据缓存机制

```cpp
class UdmfClient {
private:
    // 数据缓存：key -> UnifiedData
    ConcurrentMap<std::string, UnifiedData> dataCache_;
};
```

### 拖拽场景特殊处理

```cpp
// 处理应用内拖拽场景
void UdmfClient::ProcessDragIfInApp(UnifiedData &unifiedData,
                                     std::string &intentionDrag,
                                     std::string &key)
{
    // 应用内拖拽时直接返回数据，无需 IPC
    if (intentionDrag == "drag") {
        // 直接返回数据，不进行持久化
        return;
    }
}
```

### 异步客户端

`UdmfAsyncClient` 提供异步数据操作能力（`framework/innerkitsimpl/client/udmf_async_client.h`）：

```cpp
class UdmfAsyncClient {
public:
    // 异步设置数据
    std::shared_ptr<AsyncTask> SetDataAsync(CustomOption &option,
                                             UnifiedData &unifiedData,
                                             AsyncContext &context);
    
    // 异步获取数据
    std::shared_ptr<AsyncTask> GetDataAsync(const QueryOption &query,
                                              AsyncContext &context,
                                              std::shared_ptr<UnifiedData> result);
    
    // 异步批量获取
    std::shared_ptr<AsyncTask> GetBatchDataAsync(
        const QueryOption &query,
        AsyncContext &context,
        std::vector<std::shared_ptr<UnifiedData>> results);
};
```

## UtdClient 单例

### 单例实现

`UtdClient` 是 UTD 类型管理的单例（`framework/innerkitsimpl/client/utd_client.cpp`）：

```cpp
class UtdClient {
public:
    static UtdClient API_EXPORT &GetInstance();

private:
    // 一次性初始化
    static void InitializeOnce();
    
    // 初始化 UTD 类型
    void InitializeUtdTypes();
    
    // 获取自定义 UTD
    void GetCustomUtd();
    
    // 获取动态 UTD
    void GetDynamicUtd();
};
```

### 初始化流程

```cpp
UtdClient &UtdClient::GetInstance()
{
    static UtdClient instance;
    return instance;
}

void UtdClient::InitializeOnce()
{
    // 初始化 UTD 类型
    InitializeUtdTypes();
    
    // 注册服务通知器
    RegServiceNotifier();
    
    // 重试等待（最多 5 次）
    int32_t retryCount = 0;
    while (retryCount < MAX_RETRY_COUNT) {
        if (descriptorCfgs_.empty()) {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            retryCount++;
        } else {
            break;
        }
    }
}

void UtdClient::InitializeUtdTypes()
{
    // 加载预设类型
    GetCustomUtd();
    
    // 加载动态类型
    GetDynamicUtd();
}
```

### 类型配置管理

```cpp
class UtdClient {
private:
    // UTD 类型配置
    std::vector<TypeDescriptorCfg> descriptorCfgs_;
    
    // 互斥锁保护配置
    std::mutex utdMutex_;
};
```

## GetterSystem 懒加载

### 工厂模式

`GetterSystem` 实现 Entry Getter 工厂模式（`framework/innerkitsimpl/client/getter_system.cpp`）：

```cpp
class GetterSystem {
public:
    // 获取指定类型的 Getter
    std::shared_ptr<EntryGetter> GetGetter(const std::vector<std::string> &utdIds);
    
    // 注册 Getter 工厂
    void RegisterGetter(const std::string &utdId,
                        std::shared_ptr<EntryGetter> getter);
    
private:
    // Getter 工厂缓存
    ConcurrentMap<std::string, std::shared_ptr<EntryGetter>> getterFactories_;
};
```

### Entry Getter 接口

```cpp
class EntryGetter {
public:
    // 获取 Entry 值（懒加载实现）
    virtual ValueType Get(const std::string &typeId) = 0;
    
    // 获取所有类型
    virtual std::vector<std::string> GetTypes() = 0;
    
    // 初始化状态检查
    virtual bool IsInitialized() = 0;
};
```

## 数据操作流程

### SetData 流程

```cpp
Status UdmfClient::SetData(CustomOption &option, 
                           UnifiedData &unifiedData,
                           std::string &key)
{
    // 1. 参数校验
    if (!ValidateOption(option)) {
        return E_INVALID_PARAMETERS;
    }
    
    // 2. 类型检查
    if (!CheckFileUtdType(unifiedData.GetSummary(), 
                         option.allowTypes)) {
        return E_INVALID_TYPE_ID;
    }
    
    // 3. 数据序列化
    std::vector<uint8_t> serialized;
    if (!SerializeData(unifiedData, serialized)) {
        return E_JSON_CONVERT_FAILED;
    }
    
    // 4. IPC 调用服务
    auto service = GetServiceProxy();
    Status ret = service->SetData(option, unifiedData, key);
    
    // 5. 更新本地缓存（应用内场景）
    if (option.intention == "drag") {
        ProcessDragIfInApp(unifiedData, option.intention, key);
        dataCache_.Compute(key, 
            [&](const std::string &, UnifiedData &data) {
                return unifiedData;
            });
    }
    
    return ret;
}
```

### GetData 流程

```cpp
Status UdmfClient::GetData(const QueryOption &query,
                            UnifiedData &unifiedData)
{
    // 1. 检查本地缓存
    auto cached = dataCache_.Find(query.key);
    if (cached.has_value()) {
        unifiedData = cached.value();
        return E_OK;
    }
    
    // 2. IPC 调用服务
    auto service = GetServiceProxy();
    Status ret = service->GetData(query, unifiedData);
    
    if (ret == E_OK) {
        // 3. 存入本地缓存
        dataCache_.Insert(query.key, unifiedData);
    }
    
    return ret;
}
```

### GetBatchData 流程

```cpp
Status UdmfClient::GetBatchData(const QueryOption &query,
                                 std::vector<UnifiedData> &unifiedDataSet)
{
    // 1. IPC 调用服务
    auto service = GetServiceProxy();
    Status ret = service->GetBatchData(query, unifiedDataSet);
    
    if (ret == E_OK) {
        // 2. 批量更新缓存
        for (const auto &data : unifiedDataSet) {
            std::string key = data.GetRuntime()->GetKey();
            dataCache_.Insert(key, data);
        }
    }
    
    return ret;
}
```

## 权限管理

### AddPrivilege 流程

```cpp
Status UdmfClient::AddPrivilege(const QueryOption &query,
                                 Privilege &privilege)
{
    // 1. 校验调用者权限
    if (!CheckCallingPermission()) {
        return E_NO_PERMISSION;
    }
    
    // 2. 校验数据所有者
    std::string owner = GetDataOwner(query.key);
    if (owner != query.bundleName) {
        return E_NO_PERMISSION;
    }
    
    // 3. 添加权限
    auto service = GetServiceProxy();
    return service->AddPrivilege(query, privilege);
}
```

### ShareOption 管理

```cpp
Status UdmfClient::SetAppShareOption(const std::string &intention,
                                      ShareOptions shareOption)
{
    // 1. 校验系统权限
    if (!CheckSystemPermission()) {
        return E_NO_SYSTEM_PERMISSION;
    }
    
    // 2. 更新共享选项
    auto service = GetServiceProxy();
    return service->SetAppShareOption(intention, shareOption);
}
```

## 同步机制

### Sync 流程

```cpp
Status UdmfClient::Sync(const QueryOption &query,
                         const std::vector<std::string> &devices)
{
    // 1. 校验本地数据
    if (!IsLocalData(query.key)) {
        return E_INVALID_PARAMETERS;
    }
    
    // 2. 触发设备间同步
    auto service = GetServiceProxy();
    return service->Sync(query, devices);
}
```

### IsRemoteData 检查

```cpp
Status UdmfClient::IsRemoteData(const QueryOption &query,
                                 bool &result)
{
    // 1. 检查本地缓存
    auto cached = dataCache_.Find(query.key);
    if (cached.has_value()) {
        result = false;
        return E_OK;
    }
    
    // 2. 询问服务
    auto service = GetServiceProxy();
    return service->IsRemoteData(query, result);
}
```

## 延迟数据加载

### SetDelayInfo 流程

```cpp
Status UdmfClient::SetDelayInfo(const DataLoadParams &dataLoadParams,
                                 std::string &key)
{
    // 1. 生成延迟加载 key
    key = GenerateDelayKey();
    
    // 2. 注册延迟回调
    auto service = GetServiceProxy();
    return service->SetDelayInfo(dataLoadParams, 
                                  GetNotifierProxy(), key);
}
```

### GetDataIfAvailable 流程

```cpp
Status UdmfClient::GetDataIfAvailable(const std::string &key,
                                        const DataLoadInfo &dataLoadInfo,
                                        sptr<IRemoteObject> iUdmfNotifier,
                                        std::shared_ptr<UnifiedData> unifiedData)
{
    // 1. 检查本地缓存
    auto cached = dataCache_.Find(key);
    if (cached.has_value()) {
        unifiedData = cached.value();
        return E_OK;
    }
    
    // 2. 询问服务数据是否可用
    auto service = GetServiceProxy();
    return service->GetDataIfAvailable(key, dataLoadInfo,
                                         iUdmfNotifier, unifiedData);
}
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口
- [20_Service_Layer.md](./20_Service_Layer.md)：服务层架构
- [22_Core_Data_Structures.md](./22_Core_Data_Structures.md)：核心数据结构
