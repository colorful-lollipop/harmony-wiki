# HDF Core 关键调用链

## 1. 服务注册调用链

### 1.1 驱动注册服务

```
Driver::Dispatch (驱动实现)
    │
    ▼
DevSvcManagerAddService (hdf_device_desc.c)
    │
    ▼
DevSvcManagerProxyAddService (devsvc_manager_proxy.c)
    │
    ▼
HdfRemoteServiceCall (hdf_remote_service.c)
    │
    ▼
HdfRemoteAdapterDispatch (hdf_remote_adapter.cpp)
    │
    ▼
IPC SendRequest (Binder)
    │
    ▼
DevSvcManagerStubDispatch (devsvc_manager_stub.c)
    │
    ▼
DevSvcManagerStubAddService
    │
    ├── AddServicePermCheck (SELinux 检查)
    │   │
    │   ├── HdfRemoteGetCallingPid
    │   ├── HdfRemoteGetCallingSid
    │   └── HdfAddServiceCheck
    │
    ▼
DevSvcMgrStubGetPara (参数解析)
    │
    ▼
ObtainServiceObject (创建服务对象)
    │
    ▼
IDevSvcManager::AddService (服务注册到列表)
```

### 1.2 代码位置

| 函数 | 文件 | 行号 |
|------|------|------|
| DevSvcManagerAddService | hdf_device_desc.c | ~290 |
| DevSvcManagerProxyAddService | devsvc_manager_proxy.c | ~50 |
| DevSvcManagerStubDispatch | devsvc_manager_stub.c | ~538 |
| DevSvcManagerStubAddService | devsvc_manager_stub.c | ~254 |
| ObtainServiceObject | devsvc_manager_stub.c | ~162 |

## 2. 服务发现调用链

```
System Service (C++)
    │
    ▼
IServiceManager::GetService (iservmgr_client.cpp)
    │
    ▼
HDIServiceManager::GetService (servmgr_client.c)
    │
    ▼
HdfRemoteServiceCall
    │
    ▼
IPC SendRequest
    │
    ▼
DevSvcManagerStubDispatch
    │
    ▼
DevSvcManagerStubGetService
    │
    ├── GetServicePermCheck (SELinux)
    │
    ▼
IDevSvcManager::GetObject (查找服务)
    │
    ▼
CheckServiceObjectValidNoLock (验证服务)
    │
    ▼
HdfSbufWriteRemoteService (返回服务代理)
```

### 2.1 代码位置

| 函数 | 文件 | 行号 |
|------|------|------|
| IServiceManager::GetService | iservmgr_client.cpp | ~40 |
| HDIServiceManager::GetService | servmgr_client.c | ~30 |
| DevSvcManagerStubGetService | devsvc_manager_stub.c | ~326 |
| CheckServiceObjectValidNoLock | devsvc_manager_stub.c | ~97 |

## 3. 设备加载调用链

```
Init Process
    │
    ▼
Start hdf_devmgr
    │
    ▼
main (devmgr_main.c)
    │
    ▼
DevmgrServiceStart
    │
    ▼
HcsGetRoot (解析 HCS 配置)
    │
    ▼
DevMgrServiceLoadHosts (加载 Host 配置)
    │
    ▼
DriverInstallerFull (驱动安装器)
    │
    ├── StartDevHost (启动 DevHost 进程)
    │   │
    │   └── fork/exec devhost
    │
    ▼
DevHostServiceFull (DevHost 主循环)
    │
    ▼
AddDevice (添加设备)
    │
    ▼
DriverLoaderFull::LoadDriver (加载驱动 .so)
    │
    ▼
dlopen (加载驱动库)
    │
    ▼
HdfDriverEntry::Bind (绑定设备)
    │
    ▼
HdfDriverEntry::Init (初始化驱动)
    │
    ▼
AddService (注册服务)
```

### 3.1 代码位置

| 函数 | 文件 | 行号 |
|------|------|------|
| DevmgrServiceStart | devmgr_service.c | ~200 |
| DevMgrServiceLoadHosts | devmgr_service.c | ~150 |
| StartDevHost | driver_installer_full.c | ~80 |
| DevHostServiceFull | devhost_service_full.c | ~100 |
| LoadDriver | driver_loader_full.c | ~50 |

## 4. IPC 调用链

```
Client
    │
    ▼
HdfRemoteServiceCall (C 接口)
    │
    ▼
HdfRemoteServiceDispatch (hdf_remote_service.c)
    │
    ▼
HdfRemoteAdapterDispatch (hdf_remote_adapter.cpp)
    │
    ▼
HdfRemoteServiceHolder::SendRequest
    │
    ▼
IRemoteObject::SendRequest (Binder C++)
    │
    ▼
Binder Driver (Kernel)
    │
    ▼
IPCObjectStub::OnRemoteRequest
    │
    ▼
HdfRemoteServiceStub::OnRemoteRequest (hdf_remote_adapter.cpp)
    │
    ▼
HdfRemoteAdapterOnRemoteRequest
    │
    ▼
dispatcher->Dispatch (C 回调)
    │
    ▼
Server Handler
```

### 4.1 代码位置

| 函数 | 文件 | 行号 |
|------|------|------|
| HdfRemoteServiceCall | hdf_remote_service.c | ~80 |
| HdfRemoteAdapterDispatch | hdf_remote_adapter.cpp | ~120 |
| HdfRemoteServiceStub::OnRemoteRequest | hdf_remote_adapter.cpp | ~60 |
| HdfRemoteAdapterOnRemoteRequest | hdf_remote_adapter.cpp | ~150 |

## 5. 安全权限检查调用链

```
IPC Call Entry
    │
    ▼
DevSvcManagerStubDispatch
    │
    ▼
DevSvcManagerStubAddService / GetService / RemoveService
    │
    ▼
AddServicePermCheck / GetServicePermCheck / ListServicePermCheck
    │
    ├── HdfRemoteGetCallingPid (获取调用者 PID)
    ├── HdfRemoteGetCallingUid (获取调用者 UID)
    ├── HdfRemoteGetCallingSid (获取调用者 SID)
    │
    ▼
HdfAddServiceCheck / HdfGetServiceCheck / HdfListServiceCheck
    (selinux_adapter: libservice_checker)
    │
    ▼
SELinux Policy Check
```

### 5.1 代码位置

| 函数 | 文件 | 行号 |
|------|------|------|
| AddServicePermCheck | devsvc_manager_stub.c | ~36 |
| GetServicePermCheck | devsvc_manager_stub.c | ~56 |
| ListServicePermCheck | devsvc_manager_stub.c | ~77 |
| HdfRemoteGetCallingPid | hdf_remote_adapter.cpp | ~200 |
| HdfRemoteGetCallingSid | hdf_remote_adapter.cpp | ~210 |

## 6. 平台驱动调用链（以 GPIO 为例）

```
Application
    │
    ▼
GpioRead / GpioWrite (gpio_if.h)
    │
    ▼
GpioCntlrRead / GpioCntlrWrite (gpio_if.c)
    │
    ▼
GpioMethod::read / write (驱动实现)
    │
    ▼
Hardware Register Access
```

### 6.1 代码位置

| 函数 | 文件 | 路径 |
|------|------|------|
| GpioRead | gpio_if.h | framework/include/platform/ |
| GpioCntlrRead | gpio_if.c | framework/support/platform/src/ |

## 7. 配置解析调用链

```
HCS File (hdf.hcs)
    │
    ▼
hc-gen (编译时)
    │
    ▼
HCB File (hdf.hcb)
    │
    ▼
HcsBlobParse (运行时)
    │
    ▼
HcsGetRoot
    │
    ▼
HcsGetNodeByRef (获取节点)
    │
    ▼
HcsGetString / HcsGetUint32 / ... (读取属性)
```

### 7.1 代码位置

| 函数 | 文件 | 路径 |
|------|------|------|
| HcsBlobParse | hcs_blob_load.c | adapter/uhdf2/utils/src/hcs_parser/ |
| HcsGetRoot | hcs_tree_if.h | interfaces/inner_api/utils/ |
| HcsGetString | hcs_tree_if.c | framework/utils/src/hcs_parser/ |

## 8. 相关文档

- [架构说明](./03_Architecture.md)
- [HDI 接口](./04_HDI_API.md)
- [内部 API](./05_Inner_API.md)
