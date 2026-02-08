# Data Share 架构设计

## 目的

本文档描述 Data Share 的核心架构，包括组件图、数据流、线程模型和关键类关系。

## 适用范围

- 系统架构师
- 需要深入理解实现细节的开发者
- 性能优化工程师

## 架构概览

Data Share 采用**分层架构**设计，从上到下依次为：

1. **应用层 (Application)** - JS/ETS/C++ 应用代码
2. **框架层 (Framework)** - NAPI/ANI/Native 实现
3. **IPC 层** - Binder 跨进程通信
4. **系统服务层** - datamgr_service

## 组件图

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                Application Layer                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐  │
│  │  JS/ETS App │    │  JS/ETS App │    │ Cangjie App │    │   Native App    │  │
│  │  (Consumer) │    │  (Provider) │    │  (Consumer) │    │    (Consumer)   │  │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    └────────┬────────┘  │
└─────────┼──────────────────┼──────────────────┼────────────────────┼───────────┘
          │                  │                  │                    │
          ▼                  ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                Framework Layer                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                        NAPI / ANI / FFI Layer                            │   │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                │   │
│  │  │   NAPI (JS)   │  │  ANI (ArkTS)  │  │  FFI (Cangjie)│                │   │
│  │  │               │  │               │  │               │                │   │
│  │  │• DataShare    │  │• DataShare    │  │• Predicates   │                │   │
│  │  │  Helper       │  │  Helper       │  │               │                │   │
│  │  │• Predicates   │  │• ResultSet    │  │               │                │   │
│  │  │• Observer     │  │• Observer     │  │               │                │   │
│  │  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘                │   │
│  └──────────┼──────────────────┼──────────────────┼────────────────────────┘   │
│             │                  │                  │                            │
│             └──────────────────┼──────────────────┘                            │
│                                ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                        Native Consumer Layer                             │   │
│  │                                                                          │   │
│  │   DataShareHelper (Factory)                                              │   │
│  │        │                                                                 │   │
│  │        ├── Creator() ──► CreateServiceHelper() ──► Silent Mode           │   │
│  │        │                                               │                 │   │
│  │        │                                               ▼                 │   │
│  │        │                                  GeneralControllerServiceImpl   │   │
│  │        │                                          │                      │   │
│  │        │                                          ▼                      │   │
│  │        │                              DataShareManagerImpl ──► IPC      │   │
│  │        │                                                                  │   │
│  │        └── Creator() ──► CreateExtHelper() ──► Non-Silent Mode           │   │
│  │                                                    │                     │   │
│  │                                                    ▼                     │   │
│  │                                      GeneralControllerProviderImpl       │   │
│  │                                                    │                     │   │
│  │                                                    ▼                     │   │
│  │                                        DataShareConnection ──► IPC      │   │
│  │                                                                          │   │
│  │   DataShareHelperImpl (组合)                                             │   │
│  │   ├── generalCtl_: GeneralController                                     │   │
│  │   ├── extSpCtl_: ExtSpecialController                                   │   │
│  │   ├── persistentDataCtl_: PersistentDataController                      │   │
│  │   └── publishedDataCtl_: PublishedDataController                        │   │
│  │                                                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                        Native Provider Layer                             │   │
│  │                                                                          │   │
│  │   DataShareStub (IPC Stub)                                               │   │
│  │        │                                                                 │   │
│  │        ├── CmdInsert() / CmdUpdate() / CmdDelete() / CmdQuery()         │   │
│  │        ├── CmdRegisterObserver() / CmdUnregisterObserver()              │   │
│  │        └── ...                                                          │   │
│  │        │                                                                 │   │
│  │        ▼                                                                 │   │
│  │   DataShareStubImpl                                                      │   │
│  │        │                                                                 │   │
│  │        ├── CheckCallingPermission() ──► AccessTokenKit::Verify()        │   │
│  │        ├── VerifyProvider() ──► Provider 白名单检查                      │   │
│  │        │                                                                 │   │
│  │        └── JsSyncCall() / StsSyncCall() ──► UV Queue                     │   │
│  │                │                                                         │   │
│  │                ▼                                                         │   │
│  │        DataShareExtAbility                                               │   │
│  │                │                                                         │   │
│  │        ┌───────┴───────┐                                                 │   │
│  │        ▼               ▼                                                 │   │
│  │   JsDataShare      StsDataShare                                          │   │
│  │   ExtAbility       ExtAbility                                            │   │
│  │        │               │                                                 │   │
│  │        ▼               ▼                                                 │   │
│  │   [JS Extension]   [ArkTS Extension]                                     │   │
│  │                                                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                   IPC Layer                                      │
│                                                                                  │
│   ┌─────────────────────────┐         Binder IPC         ┌─────────────────────┐ │
│   │   DataShareProxy        │  ═══════════════════════►  │   DataShareStub     │ │
│   │   (IRemoteProxy)        │                            │   (IRemoteStub)     │ │
│   └─────────────────────────┘                            └─────────────────────┘ │
│                                                                                  │
│   • CMD_INSERT (0x01)           • CMD_QUERY (0x04)                               │
│   • CMD_UPDATE (0x02)           • CMD_BATCH_INSERT (0x07)                       │
│   • CMD_DELETE (0x03)           • CMD_REGISTER_OBSERVER (0x0a)                  │
│   • ...                         • ...                                           │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              System Service Layer                                │
│                                                                                  │
│   ┌─────────────────────────┐    ┌─────────────────────────┐                    │
│   │   datamgr_service       │    │   BMS / AccessToken     │                    │
│   │   (分布式数据管理服务)    │    │   (权限/包管理服务)      │                    │
│   │                         │    │                         │                    │
│   │ • Silent 访问处理        │    │ • VerifyAccessToken()   │                    │
│   │ • proxyData 管理        │    │ • GetBundleInfo()       │                    │
│   │ • 订阅管理               │    │ • GetApplicationInfo()  │                    │
│   └─────────────────────────┘    └─────────────────────────┘                    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 数据流

### Insert 数据流

```
┌─────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   App   │────►│   NAPI      │────►│   Helper    │────►│   Proxy     │────►│    Stub     │
│   (JS)  │     │   Layer     │     │   Impl      │     │   (IPC)     │     │   (Provider)│
└─────────┘     └─────────────┘     └─────────────┘     └─────────────┘     └──────┬──────┘
                                                                                   │
                                                                                   ▼
┌─────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Result │◄────│   NAPI      │◄────│   Result    │◄────│   IPC       │◄────│  Extension  │
│  (JS)   │     │   Callback  │     │   Parcel    │     │   Response  │     │  (JS/ArkTS) │
└─────────┘     └─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘

Data Flow:
1. JS ValuesBucket ──► NAPI 解析 ──► DataShareValuesBucket (C++)
2. DataShareValuesBucket ──► IPC Parcel ──► Remote Side
3. Remote Side ──► UV Queue ──► JS Extension
4. JS Extension ──► Return Value ──► UV Queue ──► IPC Response
5. IPC Response ──► Result Parcel ──► NAPI Callback ──► JS Result
```

### Query 数据流 (共享内存优化)

```
┌─────────┐     ┌─────────────┐     ┌─────────────────────────────────────────────┐
│   App   │────►│   NAPI      │────►│   DataShareProxy::Query()                   │
│   (JS)  │     │   Layer     │     │        │                                     │
└─────────┘     └─────────────┘     │        ▼                                     │
                                    │   SendRequest(CMD_QUERY)                     │
                                    │        │                                     │
                                    │        ▼                                     │
                                    │   DataShareStub::CmdQuery()                  │
                                    │        │                                     │
                                    │        ▼                                     │
                                    │   Ashmem (共享内存) ◄───────┐                │
                                    │        │                  │                │
                                    │        ▼                  │                │
                                    │   ResultSetBridge         │                │
                                    │        │                  │                │
                                    │        ▼                  │                │
                                    │   WriteToAshmem() ────────┘                │
                                    └─────────────────────────────────────────────┘
                                                       │
                                                       ▼
┌─────────┐     ┌─────────────┐     ┌─────────────────────────────────────────────┐
│  Result │◄────│   NAPI      │◄────│   ISharedResultSet::ReadFromAshmem()        │
│  (JS)   │     │   Callback  │     │        │                                     │
└─────────┘     └─────────────┘     │        ▼                                     │
                                    │   DataShareResultSetProxy (NAPI 包装)       │
                                    └─────────────────────────────────────────────┘
```

**关键代码**:
- `frameworks/native/common/src/ishared_result_set.cpp` - 共享内存结果集
- `frameworks/native/provider/include/result_set_bridge.h:24` - ResultSet 桥接

## 线程模型

### 客户端线程模型

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                           Client Thread Model                                  │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │                     Silent Mode (GeneralControllerServiceImpl)         │   │
│  │                                                                        │   │
│  │   ┌───────────────────────────────────────────────────────────────┐   │   │
│  │   │                    ExecutorPool (ThreadPool)                   │   │   │
│  │   │  • MAX_THREADS = 2  (general_controller_service_impl.cpp:33)   │   │   │
│  │   │  • MIN_THREADS = 0                                              │   │   │
│  │   │                                                                 │   │   │
│  │   │  Thread 1 ──► Query() ──► IPC ──► Response                      │   │   │
│  │   │  Thread 2 ──► Query() ──► IPC ──► Response (concurrent)         │   │   │
│  │   │                                                                 │   │   │
│  │   │  Supports TimedQuery:                                           │   │   │
│  │   │  • timeout 参数指定等待时间                                      │   │   │
│  │   │  • 超时返回 TIMEOUT_ERROR                                       │   │   │
│  │   └───────────────────────────────────────────────────────────────┘   │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                                │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │                   Non-Silent Mode (GeneralControllerProviderImpl)      │   │
│  │                                                                        │   │
│  │   ┌───────────────────────────────────────────────────────────────┐   │   │
│  │   │                   Single Connection (Blocking)                 │   │   │
│  │   │                                                                 │   │   │
│  │   │  Main Thread ──► Insert() ──► IPC SendRequest()                │   │   │
│  │   │       │                         │                              │   │   │
│  │   │       │                         ▼                              │   │   │
│  │   │       │                    Blocking Wait                       │   │   │
│  │   │       │                         │                              │   │   │
│  │   │       ▼                         ▼                              │   │   │
│  │   │  Return Result ◄─── IPC Response Received                     │   │   │
│  │   │                                                                 │   │   │
│  │   │  特点:                                                           │   │   │
│  │   │  • 单连接，阻塞式调用                                             │   │   │
│  │   │  • 每次操作需等待返回                                             │   │   │
│  │   │  • 简单但并发性能受限                                             │   │   │
│  │   └───────────────────────────────────────────────────────────────┘   │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                                │
└───────────────────────────────────────────────────────────────────────────────┘
```

### 服务端线程模型

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                           Server Thread Model                                  │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │                     IPC Thread (Binder Thread)                         │   │
│  │                                                                        │   │
│  │   OnRemoteRequest() ──► CmdXXX() ──► DataShareStubImpl::XXX()        │   │
│  │       │                                                                │   │   │
│  │       ▼                                                                │   │   │
│  │   CheckCallingPermission()  (权限检查)                                  │   │   │
│  │       │                                                                │   │   │
│  │       ▼                                                                │   │   │
│  │   uvQueue_->JsSyncCall() / StsSyncCall()                                │   │   │
│  │       │                                                                │   │   │
│  │       ▼                                                                │   │   │
│  │   [Submit Task to UV Queue]                                            │   │   │
│  │       │                                                                │   │   │
│  │       ▼                                                                │   │   │
│  │   Wait for Result (std::condition_variable)                            │   │   │
│  │                                                                        │   │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │                     JS / ArkTS Thread (Extension Thread)               │   │
│  │                                                                        │   │
│  │   ┌───────────────────────────────────────────────────────────────┐   │   │
│  │   │                    DataShareUvQueue                            │   │   │
│  │   │                                                                 │   │   │
│  │   │  std::mutex mutex_  (datashare_stub_impl.h:100)               │   │   │
│  │   │                                                                 │   │   │
│  │   │  JsSyncCall(task, getRetFunc):                                │   │   │
│  │   │    1. std::lock_guard<std::mutex> lock(mutex_)                │   │   │
│  │   │    2. Submit task to UV loop                                  │   │   │
│  │   │    3. Wait for result via condition_variable                  │   │   │
│  │   │    4. Return result                                           │   │   │
│  │   │                                                                 │   │   │
│  │   │  特点:                                                         │   │   │
│  │   │  • 线程安全：使用 mutex 保护队列操作                             │   │   │
│  │   │  • 同步等待：JS 代码执行完才返回 IPC 响应                        │   │   │
│  │   │  • 双模式：支持 JS 和 ArkTS 扩展                                │   │   │
│  │   └───────────────────────────────────────────────────────────────┘   │   │
│  │                                                                        │   │
│  │   JsDataShareExtAbility::Insert() / Update() / ...                    │   │
│  │        │                                                               │   │
│  │        ▼                                                               │   │
│  │   [执行 JS Extension 代码]                                             │   │
│  │        │                                                               │   │
│  │        ▼                                                               │   │
│  │   NotifyResult() ──► Resume IPC Thread                                 │   │
│  │                                                                        │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                                │
└───────────────────────────────────────────────────────────────────────────────┘
```

**关键代码**:
```cpp
// frameworks/native/provider/src/datashare_stub_impl.cpp:143-165
std::function<void()> syncTaskFunc = [extension, info, uri, mimeTypeFilter, result]() {
    extension->SetCallingInfo(info);
    extension->InitResult(result);
    extension->GetFileTypes(uri, mimeTypeFilter);
};
std::function<bool()> getRetFunc = [result, &ret]() -> bool {
    return result->GetRecvReply();
};
uvQueue_->JsSyncCall(syncTaskFunc, getRetFunc);
```

## 双模式架构

| 特性 | Silent 模式 | Non-Silent 模式 |
|------|-------------|-----------------|
| **URI 格式** | `datashareproxy://...?Proxy=true` | `datashare:///...` |
| **创建方式** | `CreateServiceHelper()` | `CreateExtHelper()` |
| **控制器** | `GeneralControllerServiceImpl` | `GeneralControllerProviderImpl` |
| **连接方式** | 连接系统服务 (datamgr_service) | 连接 ExtensionAbility |
| **Provider 状态** | 不需要启动 Provider | 需要 Provider 保持运行 |
| **适用场景** | 高频访问，无需复杂逻辑 | 需要实时交互或复杂处理 |
| **线程模型** | 线程池并发 | 单连接阻塞 |
| **超时支持** | 支持 | 不支持 |

**证据**:
- `frameworks/native/consumer/src/datashare_helper.cpp:97-128` - Create 方法实现
- `frameworks/native/consumer/controller/service/src/general_controller_service_impl.cpp:33` - MAX_THREADS
- `frameworks/native/consumer/controller/provider/src/general_controller_provider_impl.cpp` - Provider 控制器

## 关键类关系

```
IDataShare (Interface)
    ▲
    │
DataShareStub ─────────────────────────────────────┐
(IRemoteStub<IDataShare>)                          │
    ▲                                              │
    │                                              │
DataShareStubImpl ────────────────────────────────┤
(业务实现)                                         │
    │                                              │
    ├── DataShareExtAbility (extension_)           │
    │       ▲                                      │
    │       │                                      │
    │   JsDataShareExtAbility                     │
    │   StsDataShareExtAbility                    │
    │                                              │
    └── DataShareUvQueue (uvQueue_) ──────────────┤
                                                 │
DataShareProxy ──────────────────────────────────┘
(IRemoteProxy<IDataShare>)
    ▲
    │
GeneralController ───────────────────────────────────────────┐
    ▲                                                        │
    │         GeneralControllerServiceImpl ──────────────────┤
    ├── GeneralControllerProviderImpl                        │
    │       └── DataShareConnection ──► DataShareProxy       │
    │                                                        │
    └── (abstract)                                           │
            ▲                                                │
            │                                                │
DataShareHelperImpl ─────────────────────────────────────────┤
    ├── generalCtl_: GeneralController                       │
    ├── extSpCtl_: ExtSpecialController                     │
    ├── persistentDataCtl_: PersistentDataController        │
    └── publishedDataCtl_: PublishedDataController          │
                                                            │
DataShareHelper (Factory)                                   │
    └── Creator() ──► DataShareHelperImpl ──────────────────┘
```

## 关键结论

1. **分层清晰** - 从 NAPI/ANI 到 Native 再到 IPC，每层职责单一
2. **双模式支持** - Silent 模式适合高频访问，Non-Silent 模式需要 Provider 保持运行
3. **线程安全** - 服务端使用 mutex 和 condition_variable 保证线程安全
4. **共享内存优化** - Query 操作使用 Ashmem 优化大数据量传输
5. **UV Queue 调度** - Provider 端使用 UV Queue 将 IPC 调用调度到 JS/ArkTS 线程
6. **权限检查前置** - 每个 IPC 命令都先进行权限检查再执行业务逻辑

## 相关文档

- [目录结构](02_Directory_Structure.md) - 代码组织
- [N-API 参考](04_NAPI_Reference.md) - JS API 详情
- [关键调用链](appendix/Callgraphs.md) - 详细调用流程
