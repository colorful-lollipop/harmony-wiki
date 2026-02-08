# 关键调用链

## 调用链概述

本文档描述 UDMF 各层接口的关键调用链，从应用入口到核心服务，涵盖数据操作、类型管理和 IPC 通信等主要流程。调用链按功能分类，每条调用链包含完整路径和关键节点说明。

## 数据操作调用链

### SetData 调用链

**功能描述**：应用向 UDMF 存储统一数据

**调用链路径**：
```
应用层
  │
  ├─→ N-API 入口
  │   └─→ interfaces/jskits/module/unified_data_channel_napi_module.cpp:97-101
  │       └─→ UnifiedDataChannelNapi::InsertData()
  │           └─→ framework/jskitsimpl/data/unified_data_channel_napi.cpp
  │
  ├─→ NDK 入口
  │   └─→ interfaces/ndk/data/udmf.h:997
  │       └─→ OH_Udmf_SetUnifiedData()
  │           └─→ framework/ndkimpl/data/udmf.cpp
  │
  └─→ InnerKit 入口
      └─→ interfaces/innerkits/client/udmf_client.h:37
          └─→ UdmfClient::SetData()
              └─→ framework/innerkitsimpl/client/udmf_client.cpp
                  │
                  ├─→ 参数校验
                  │   └─→ ValidateOption()
                  │
                  ├─→ 数据序列化
                  │   └─→ UnifiedDataHelper::Pack()
                  │       └─→ TLVUtil::Writing()
                  │
                  └─→ IPC 调用
                      └─→ UdmfServiceProxy::SetData()
                          ├─→ MessageParcel::WriteInterfaceToken()
                          ├─→ MessageParcel::Marshalling()
                          │   └─→ UnifiedData::Marshalling()
                          │       └─→ TLVUtil::Writing()
                          │
                          └─→ SendRequest()
                              └─→ IPC 框架传输
                                  │
                                  └─→ 服务端
                                      └─→ UdmfService::SetData()
                                          ├─→ AccessToken 检查
                                          ├─→ 权限验证
                                          └─→ 持久化
                                              └─→ KvStore::Put()
```

### GetData 调用链

**功能描述**：应用从 UDMF 查询统一数据

**调用链路径**：
```
应用层
  │
  ├─→ N-API 入口
  │   └─→ UnifiedDataChannelNapi::QueryData()
  │       └─→ framework/jskitsimpl/data/unified_data_channel_napi.cpp
  │
  ├─→ NDK 入口
  │   └─→ OH_Udmf_GetUnifiedData()
  │       └─→ framework/ndkimpl/data/udmf.cpp
  │
  └─→ InnerKit 入口
      └─→ UdmfClient::GetData()
          │
          ├─→ 本地缓存检查
          │   └─→ dataCache_.Find(key)
          │       └─→ ConcurrentMap::Find()
          │
          └─→ IPC 调用（缓存未命中）
              └─→ UdmfServiceProxy::GetData()
                  │
                  ├─→ MessageParcel::WriteInterfaceToken()
                  │
                  └─→ SendRequest()
                      └─→ IPC 框架传输
                          │
                          └─→ 服务端
                              └─→ UdmfService::GetData()
                                  ├─→ AccessToken 检查
                                  ├─→ 权限验证
                                  │   └─→ CheckPermission()
                                  │
                                  └─→ 数据检索
                                      └─→ KvStore::Get()
                                          │
                                          └─→ 反序列化
                                              └─→ UnifiedData::Unmarshalling()
                                                  └─→ TLVUtil::Reading()
                                                      │
                                                      └─→ 返回结果
                                                          ├─→ MessageParcel::ReadString()
                                                          └─→ UnifiedData::Unmarshalling()
```

### GetBatchData 调用链

**功能描述**：应用批量查询统一数据

**调用链路径**：
```
应用层
  │
  └─→ UdmfClient::GetBatchData()
      │
      ├─→ 构建批量查询请求
      │   └─→ QueryOption::BatchQuery()
      │
      └─→ IPC 调用
          └─→ UdmfServiceProxy::GetBatchData()
              │
              └─→ 服务端处理
                  └─→ UdmfService::GetBatchData()
                      │
                      ├─→ 批量检索
                      │   └─→ KvStore::GetBatch()
                      │
                      └─→ 批量反序列化
                          └─→ TLVUtil::Reading()
                              │
                              └─→ 返回数据集合
                                  └─→ std::vector<UnifiedData>
```

## 类型管理调用链

### GetTypeDescriptor 调用链

**功能描述**：查询 UTD 类型描述符

**调用链路径**：
```
应用层
  │
  ├─→ N-API 入口
  │   └─→ UniformTypeDescriptorNapi::GetTypeDescriptor()
  │       └─→ framework/jskitsimpl/data/uniform_type_descriptor_napi.cpp
  │
  ├─→ NDK 入口
  │   └─→ OH_Utd_Create()
  │       └─→ framework/ndkimpl/data/utd.cpp
  │
  └─→ InnerKit 入口
      └─→ UtdClient::GetTypeDescriptor()
          │
          ├─→ 检查本地缓存
          │   └─→ descriptorCfgs_.Find(typeId)
          │
          └─→ 服务查询（缓存未命中）
              └─→ UtdServiceProxy::GetTypeDescriptor()
                  │
                  └─→ 服务端
                      └─→ UtdService::GetTypeDescriptor()
                          │
                          └─→ 类型查找
                              ├─→ PresetTypeDescriptors 查找
                              │   └─→ GetInstance()
                              │       └─→ GetTypeDescriptor(typeId)
                              │
                              └─→ CustomUtdStore 查找
                                  └─→ CustomUtdStore::Get()
```

### 类型识别调用链

**功能描述**：通过文件扩展名或 MIME 类型识别 UTD

**调用链路径**：
```
应用层
  │
  ├─→ UtdClient::GetUniformDataTypeByFilenameExtension()
  │   │
  │   ├─→ 构建查询键
  │   │   └─→ extension + "|" + belongsTo
  │   │
  │   └─→ 内部查找
  │       └─→ UtdGraph::FindPath()
  │           │
  │           └─→ 层次图遍历
  │               ├─→ BFS/DFS 遍历
  │               └─→ 类型匹配
  │
  └─→ UtdClient::GetUniformDataTypeByMIMEType()
      │
      └─→ 类似流程
          └─→ MIME 类型到 UTD 映射
```

## 权限管理调用链

### AddPrivilege 调用链

**功能描述**：为数据添加访问权限

**调用链路径**：
```
应用层
  │
  └─→ UdmfClient::AddPrivilege()
      │
      ├─→ 调用者权限检查
      │   └─→ CheckCallingPermission()
      │       └─→ AccessTokenKit::VerifyAccessToken()
      │
      ├─→ 数据所有权检查
      │   └─→ GetDataOwner(key)
      │       └─→ KvStore::GetMetadata()
      │
      └─→ IPC 调用
          └─→ UdmfServiceProxy::AddPrivilege()
              │
              └─→ 服务端
                  └─→ UdmfService::AddPrivilege()
                      │
                      └─→ 权限持久化
                          └─→ PrivilegeStore::Add()
```

### ShareOption 设置调用链

**功能描述**：设置应用的共享选项

**调用链路径**：
```
应用层
  │
  └─→ UdmfClient::SetAppShareOption()
      │
      ├─→ 系统权限检查
      │   └─→ CheckSystemPermission()
      │       └─→ AccessTokenKit::GetTokenType()
      │
      └─→ IPC 调用
          └─→ UdmfServiceProxy::SetAppShareOption()
              │
              └─→ 服务端
                  └─→ UdmfService::SetAppShareOption()
                      │
                      └─→ 配置存储
                          └─→ AppConfigStore::Update()
```

## 异步操作调用链

### 进度监听调用链

**功能描述**：异步数据操作的进度回调

**调用链路径**：
```
应用层
  │
  └─→ GetDataParamsNapi::SetProgressListener()
      │
      ├─→ 创建 Threadsafe Function
      │   └─→ napi_create_threadsafe_function()
      │       ├─→ 接收器：CallProgressListener()
      │       ├──→ native thread callback
      │       │
      │       └─→ 调用时机：napi_tsfn_blocking
      │
      └─→ IPC 注册
          └─→ UdmfServiceProxy::ObtainAsynProcess()
              │
              └─→ 服务端
                  └─→ UdmfService::ObtainAsynProcess()
                      │
                      └─→ 进度回调注册
                          └─→ ProgressSignal::Register()
                              │
                              └─→ 异步执行
                                  ├─→ 状态更新
                                  ├─→ napi_call_threadsafe_function()
                                  │
                                  └─→ JS 回调触发
```

### 延迟数据加载调用链

**功能描述**：懒加载大型数据

**调用链路径**：
```
应用层
  │
  ├─→ UdmfClient::SetDelayInfo()
  │   │
  │   ├─→ 生成延迟键
  │   │   └─→ GenerateDelayKey()
  │   │
  │   └─→ IPC 注册
  │       └─→ UdmfServiceProxy::SetDelayInfo()
  │           │
  │           └─→ 服务端
  │               └─→ UdmfService::SetDelayInfo()
  │                   │
  │                   └─→ 注册延迟回调
  │                       └─→ DelayDataManager::Register()
  │
  └─→ UdmfClient::GetDataIfAvailable()
      │
      ├─→ 检查延迟数据是否可用
      │   └─→ DelayDataManager::IsAvailable(key)
      │
      └─→ 触发加载（不可用时）
          └─→ DelayDataCallback::OnDelayDataLoad()
              │
              ├─→ GetterSystem::GetGetter()
              │   └─→ EntryGetter::Get()
              │       └─→ 实际数据加载
              │
              └─→ 通知回调
                  └─→ IRemoteObject::SendRequest()
```

## 服务发现调用链

### 单例初始化调用链

**功能描述**：UDMF 客户端单例初始化

**调用链路径**：
```
UdmfClient::GetInstance()
  │
  ├─→ std::call_once
  │   └─→ InitializeOnce()
  │       │
  │       ├─→ 获取 SAMGR
  │       │   └─→ SystemAbilityManagerClient::GetSystemAbilityManager()
  │       │
  │       ├─→ 获取 KV 数据服务
  │       │   └─→ GetSystemAbility(DISTRIBUTED_KV_DATA_SERVICE_ABILITY_ID)
  │       │
  │       ├─→ 获取 UDMF 功能接口
  │       │   └─→ GetFeatureInterface("udmf")
  │       │       └─→ iface_cast<IUdmfService>()
  │       │
  │       └─→ 注册死亡通知
  │           └─→ AddDeathRecipient()
  │               └─→ ServiceDeathRecipient::OnRemoteDied()
```

## 数据转换调用链

### 记录转条目调用链

**功能描述**：将 UnifiedRecord 转换为条目格式

**调用链路径**：
```
应用层
  │
  └─→ UnifiedData::ConvertRecordsToEntries()
      │
      ├─→ 遍历所有记录
      │   └─→ for (auto& record : records_)
      │
      └─→ 每个记录转换
          └─→ UnifiedRecord::ConvertToEntry()
              │
              ├─→ 获取值
              │   └─→ GetValue()
              │
              ├─→ 序列化为 JSON
              │   └─→ JsonUtil::Serialize()
              │
              └─→ 添加到条目映射
                  └─→ entries_[typeId] = jsonValue
```

## 相关文档

- [00_Overview.md](../00_Overview.md)：项目概述
- [10_NAPI_Reference.md](../10_NAPI_Reference.md)：N-API 接口
- [11_NDK_Reference.md](../11_NDK_Reference.md)：NDK 接口
- [12_InnerKit_Reference.md](../12_InnerKit_Reference.md)：InnerKit 接口
- [20_Service_Layer.md](../20_Service_Layer.md)：服务层架构
- [21_Client_Layer.md](../21_Client_Layer.md)：客户端层架构
- [22_Core_Data_Structures.md](../22_Core_Data_Structures.md)：核心数据结构
