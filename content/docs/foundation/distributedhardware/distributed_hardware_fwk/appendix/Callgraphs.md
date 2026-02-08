# 关键调用链

本文档描述分布式硬件管理框架的关键调用链。

> **适用范围**: 需要理解代码调用路径的开发者

---

## N-API 调用链

### pauseDistributedHardware 调用链

```
JavaScript
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ native_distributedhardwarefwk_js.cpp                   │
│ DistributedHardwareManager::PauseDistributedHardware() │
│     ├── 参数解析 (argv → networkId, type)             │
│     ├── Verify()                                       │
│     │   ├── IsSystemApp()                             │
│     │   └── HasAccessDHPermission()                  │
│     └── DistributedHardwareFwkKit::PauseDistributedHardware() │
└────────────────────────┬────────────────────────────────┘
                         │
                         │ IPC
                         ▼
┌─────────────────────────────────────────────────────────┐
│ distributed_hardware_proxy.cpp                           │
│ DistributedHardwareProxy::SendRequest()                 │
│     ├── MessageParcel::WriteInterfaceToken()           │
│     ├── MessageParcel::WriteInt32()                    │
│     └── IRemoteObject::SendRequest()                   │
└────────────────────────┬────────────────────────────────┘
                         │
                         │ IPC (Binder)
                         ▼
┌─────────────────────────────────────────────────────────┐
│ distributed_hardware_stub.cpp                           │
│ DistributedHardwareStub::OnRemoteRequest()             │
│     ├── ReadInterfaceToken()                           │
│     ├── HasAccessDHPermission()                        │
│     └── OnRemoteRequestEx()                            │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ distributed_hardware_service.cpp                         │
│ DistributedHardwareService::PauseDistributedHardware() │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ component_manager.cpp                                   │
│ ComponentManager::PauseDistributedHardware()           │
│     ├── TaskFactory::CreatePauseTask()                  │
│     └── TaskExecutor::Execute()                         │
└─────────────────────────────────────────────────────────┘
```

---

## 设备上线调用链

```
DeviceManager (设备上线事件)
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ access_manager.cpp                                       │
│ AccessManager::OnDeviceOnline()                        │
│     └── DistributedHardwareManager::OnDeviceOnline()   │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ task_factory.cpp                                         │
│ TaskFactory::CreateOnlineTask()                         │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ online_task.cpp                                          │
│ OnlineTask::Execute()                                   │
│     ├── ResourceManager::SyncRemoteCapabilities()       │
│     ├── VersionManager::CheckVersionCompatibility()    │
│     └── ComponentManager::EnableSink()                 │
│           └── ComponentLoader::LoadComponent()          │
└─────────────────────────────────────────────────────────┘
```

---

## 设备下线调用链

```
DeviceManager (设备下线事件)
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ access_manager.cpp                                       │
│ AccessManager::OnDeviceOffline()                       │
│     └── DistributedHardwareManager::OnDeviceOffline()  │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ task_factory.cpp                                         │
│ TaskFactory::CreateOfflineTask()                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ offline_task.cpp                                         │
│ OfflineTask::Execute()                                 │
│     └── ComponentManager::DisableSink()                │
│           └── ComponentLoader::UnloadComponent()        │
└─────────────────────────────────────────────────────────┘
```

---

## 跨设备 IPC 调用链

```
本地设备                                                   远程设备
    │                                                        │
    │  ┌─────────────────────────────────────────────────┐   │
    │  │ dh_comm_tool.cpp                                 │   │
    │  │ DHCommTool::SendMessage()                       │   │
    │  │     ├── CheckCallerAclRight()                  │   │
    │  │     ├── SoftBus Adapter                        │───┼───▶│
    │  │     └── Send to Remote Device                  │   │   │
    │  └─────────────────────────────────────────────────┘   │   │
    │                                                        │   │
    │                                                        ▼   │
    │  ┌─────────────────────────────────────────────────┐   │
    │  │ dh_transport.cpp                                 │   │
    │  │ DHTransport::OnMessageReceived()                │◀──┤
    │  │     ├── CheckCalleeAclRight()                  │   │
    │  │     └── Dispatch to Handler                     │   │
    │  └─────────────────────────────────────────────────┘   │
    │                                                        │
    ▼                                                        ▼
```

---

## 文件位置索引

| 调用链 | 关键文件 |
|--------|----------|
| N-API 调用 | `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp` |
| IPC 代理 | `interfaces/inner_kits/src/ipc/distributed_hardware_proxy.cpp` |
| IPC 桩 | `services/distributedhardwarefwkservice/src/distributed_hardware_stub.cpp` |
| 核心服务 | `services/distributedhardwarefwkservice/src/distributed_hardware_service.cpp` |
| 任务调度 | `services/distributedhardwarefwkservice/src/task/` |
| 部件管理 | `services/distributedhardwarefwkservice/src/componentmanager/` |
| 传输层 | `services/distributedhardwarefwkservice/src/transport/` |

---

## 返回文档

- [返回主文档](../README.md)
- [架构说明](../02_Architecture.md)
- [N-API 参考](../03_NAPI_Reference.md)
