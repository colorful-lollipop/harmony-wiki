# HDF Core HDI 对外接口

## 1. 概述

**HDI**（Hardware Driver Interface）是 HDF 向系统服务提供的标准化硬件访问接口。本文档描述 HDI 接口的定义、使用方式和调用规范。

**重要说明**: HDF Core 是**纯 C/C++ 驱动框架**，不提供 N-API（Node-API）JavaScript 绑定。上层 JS 应用需通过系统服务间接访问硬件。

## 2. HDI 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    System Services (C++)                    │
│         Camera Service / Audio Service / etc.               │
├─────────────────────────────────────────────────────────────┤
│                    HDI C++ Interface                        │
│    IServiceManager / IDeviceManager / IDriverInterface      │
├─────────────────────────────────────────────────────────────┤
│                    IPC (Binder)                             │
├─────────────────────────────────────────────────────────────┤
│                    HDI C Interface                          │
│    HDIServiceManager / HDIDeviceManager                     │
├─────────────────────────────────────────────────────────────┤
│                    UHDF (User Space)                        │
│              DevMgr / DevHost / Drivers                     │
└─────────────────────────────────────────────────────────────┘
```

## 3. C++ HDI 接口

### 3.1 IServiceManager

**文件**: `interfaces/inner_api/hdi/iservmgr_hdi.h`

**功能**: 服务管理接口，用于发现和管理驱动服务

```cpp
namespace OHOS {
namespace HDI {

class IServiceManager : public HdiBase {
public:
    virtual ~IServiceManager() = default;
    
    // 获取服务
    virtual int32_t GetService(
        const std::u16string& serviceName, 
        sptr<IRemoteObject>& service) = 0;
    
    // 列出所有服务
    virtual int32_t ListAllService(
        std::vector<ServiceInfo>& serviceList) = 0;
    
    // 注册服务状态监听器
    virtual int32_t RegisterServiceStatusListener(
        sptr<IServStatListener>& listener, 
        uint16_t deviceClass) = 0;
    
    // 注销服务状态监听器
    virtual int32_t UnregisterServiceStatusListener(
        sptr<IServStatListener>& listener) = 0;
    
    // 根据接口描述列出服务
    virtual int32_t ListServiceByInterfaceDesc(
        std::vector<ServiceInfo>& serviceList,
        const std::u16string& interfaceDesc) = 0;
};

} // namespace HDI
} // namespace OHOS
```

**使用示例**:
```cpp
#include <iservmgr_hdi.h>

// 获取 ServiceManager
sptr<IServiceManager> servMgr = IServiceManager::Get();
if (servMgr == nullptr) {
    return -1;
}

// 获取相机服务
sptr<IRemoteObject> cameraService;
int32_t ret = servMgr->GetService(u"camera_service", cameraService);
if (ret != 0 || cameraService == nullptr) {
    return -1;
}
```

### 3.2 IDeviceManager

**文件**: `interfaces/inner_api/hdi/idevmgr_hdi.h`

**功能**: 设备管理接口，用于动态加载/卸载设备

```cpp
namespace OHOS {
namespace HDI {

class IDeviceManager : public HdiBase {
public:
    virtual ~IDeviceManager() = default;
    
    // 加载设备
    virtual int32_t LoadDevice(const std::string& serviceName) = 0;
    
    // 卸载设备
    virtual int32_t UnloadDevice(const std::string& serviceName) = 0;
    
    // 列出所有设备
    virtual int32_t ListAllDevice(std::vector<DeviceInfo>& deviceList) = 0;
};

} // namespace HDI
} // namespace OHOS
```

### 3.3 接口描述符

每个 HDI 服务通过接口描述符标识：

```cpp
// 服务管理器接口描述符
#define HDI_SERVICE_MANAGER_INTERFACE_DESC "HDI.IServiceManager.V1_0"

// 设备管理器接口描述符
#define HDI_DEVICE_MANAGER_INTERFACE_DESC "HDI.IDeviceManager.V1_0"
```

## 4. C HDI 接口

### 4.1 HDIServiceManager

**文件**: `interfaces/inner_api/hdi/servmgr_hdi.h`

```c
struct HDIServiceManager {
    // 获取服务
    int32_t (*GetService)(struct HDIServiceManager *self, 
                          const char *serviceName, 
                          struct HdfRemoteService **service);
    
    // 列出所有服务
    int32_t (*ListAllService)(struct HDIServiceManager *self, 
                              struct HdfSBuf *reply);
    
    // 注册服务状态监听器
    int32_t (*RegisterServiceStatusListener)(struct HDIServiceManager *self,
                                             struct ServiceStatusListener *listener);
    
    // 注销服务状态监听器
    int32_t (*UnregisterServiceStatusListener)(struct HDIServiceManager *self,
                                               struct ServiceStatusListener *listener);
    
    // 根据接口描述列出服务
    int32_t (*ListServiceByInterfaceDesc)(struct HDIServiceManager *self,
                                          const char *interfaceDesc,
                                          struct HdfSBuf *reply);
};

// 获取 ServiceManager 实例
struct HDIServiceManager *HDIServiceManagerGet(void);
```

**使用示例**:
```c
#include "servmgr_hdi.h"

// 获取 ServiceManager
struct HDIServiceManager *servMgr = HDIServiceManagerGet();
if (servMgr == NULL) {
    return HDF_FAILURE;
}

// 获取服务
struct HdfRemoteService *service = NULL;
int32_t ret = servMgr->GetService(servMgr, "camera_service", &service);
if (ret != HDF_SUCCESS) {
    return ret;
}

// 使用服务...

// 释放服务
HdfRemoteServiceRecycle(service);
```

### 4.2 HDIDeviceManager

**文件**: `interfaces/inner_api/hdi/devmgr_hdi.h`

```c
struct HDIDeviceManager {
    // 加载设备
    int32_t (*LoadDevice)(struct HDIDeviceManager *self, 
                          const char *serviceName);
    
    // 卸载设备
    int32_t (*UnloadDevice)(struct HDIDeviceManager *self, 
                            const char *serviceName);
    
    // 列出所有设备
    int32_t (*ListAllDevice)(struct HDIDeviceManager *self, 
                             struct HdfSBuf *reply);
};

// 获取 DeviceManager 实例
struct HDIDeviceManager *HDIDeviceManagerGet(void);
```

## 5. 服务状态监听

### 5.1 C++ 监听器

```cpp
#include <iservstat_listener_hdi.h>

class MyServiceListener : public IServStatListener {
public:
    int32_t OnReceive(const ServiceStatus& status) override {
        // 处理服务状态变更
        if (status.status == SERVIE_STATUS_START) {
            // 服务启动
        } else if (status.status == SERVIE_STATUS_STOP) {
            // 服务停止
        }
        return 0;
    }
};

// 注册监听器
sptr<MyServiceListener> listener = new MyServiceListener();
servMgr->RegisterServiceStatusListener(listener, DEVICE_CLASS_CAMERA);
```

### 5.2 C 监听器

```c
#include "servstat_listener_hdi.h"

int32_t MyOnReceive(struct ServiceStatusListener *listener, 
                    struct ServiceStatus *status) {
    if (status->status == SERVICE_STATUS_START) {
        // 服务启动
    }
    return HDF_SUCCESS;
}

// 创建监听器
struct ServiceStatusListener listener = {
    .callback = MyOnReceive,
};

// 注册监听器
servMgr->RegisterServiceStatusListener(servMgr, &listener);
```

## 6. 设备类定义

**文件**: `interfaces/inner_api/core/hdf_device_class.h`

```c
enum DeviceClass {
    DEVICE_CLASS_DEFAULT = 0,
    DEVICE_CLASS_AUDIO,           // 音频设备
    DEVICE_CLASS_VIDEO,           // 视频设备
    DEVICE_CLASS_CAMERA,          // 相机设备
    DEVICE_CLASS_DISPLAY,         // 显示设备
    DEVICE_CLASS_INPUT,           // 输入设备
    DEVICE_CLASS_NETWORK,         // 网络设备
    DEVICE_CLASS_SENSOR,          // 传感器设备
    DEVICE_CLASS_STORAGE,         // 存储设备
    DEVICE_CLASS_MISC,            // 杂项设备
    DEVICE_CLASS_UNKNOWN,
};
```

## 7. 错误码定义

**文件**: `interfaces/inner_api/utils/hdf_base.h`

```c
#define HDF_SUCCESS         0       // 成功
#define HDF_FAILURE         (-1)    // 通用失败
#define HDF_ERR_INVALID_PARAM   (-2)    // 无效参数
#define HDF_ERR_MALLOC_FAIL     (-3)    // 内存分配失败
#define HDF_ERR_TIMEOUT         (-4)    // 超时
#define HDF_ERR_NOT_SUPPORT     (-5)    // 不支持
#define HDF_ERR_NOT_IMPLEMENT   (-6)    // 未实现
#define HDF_ERR_INVALID_OBJECT  (-7)    // 无效对象
#define HDF_ERR_NOPERM          (-8)    // 无权限
#define HDF_ERR_IO              (-9)    // IO 错误
#define HDF_ERR_BAD_FD          (-10)   // 无效文件描述符
#define HDF_ERR_NODATA          (-11)   // 无数据
#define HDF_ERR_NOMEM           (-12)   // 内存不足
#define HDF_ERR_BUSY            (-13)   // 设备忙
#define HDF_ERR_NOT_FOUND       (-14)   // 未找到

// 设备相关错误码
#define HDF_DEV_ERR_NO_DEVICE           (-100)
#define HDF_DEV_ERR_NO_DEVICE_SERVICE   (-101)
#define HDF_DEV_ERR_NO_SPACE            (-102)
#define HDF_DEV_ERR_NO_RESOURCE         (-103)
```

## 8. 权限要求

### 8.1 SELinux 权限

使用 HDI 接口需要以下 SELinux 权限（定义在 `devsvc_manager_stub.c:36-95`）：

| 操作 | 所需权限 | 检查函数 |
|------|----------|----------|
| GetService | get service | GetServicePermCheck() |
| AddService | add service | AddServicePermCheck() |
| ListService | list service | ListServicePermCheck() |

### 8.2 调用上下文

HDI 调用会记录调用者信息：
- **PID**: 调用进程 ID (`HdfRemoteGetCallingPid()`)
- **UID**: 调用用户 ID (`HdfRemoteGetCallingUid()`)
- **SID**: 调用安全上下文 (`HdfRemoteGetCallingSid()`)

## 9. 接口清单表

| 接口名称 | 类型 | 同步/异步 | 参数校验 | 权限要求 | 错误码 |
|----------|------|-----------|----------|----------|--------|
| IServiceManager::GetService | C++ | 同步 | 服务名非空 | get service | HDF_SUCCESS / HDF_ERR_NOT_FOUND |
| IServiceManager::ListAllService | C++ | 同步 | 无 | list service | HDF_SUCCESS |
| IServiceManager::RegisterServiceStatusListener | C++ | 同步 | 监听器有效 | 无 | HDF_SUCCESS |
| IDeviceManager::LoadDevice | C++ | 同步 | 服务名非空 | 特权 | HDF_SUCCESS / HDF_ERR_NOT_FOUND |
| IDeviceManager::UnloadDevice | C++ | 同步 | 服务名非空 | 特权 | HDF_SUCCESS |
| HDIServiceManager::GetService | C | 同步 | 服务名非空 | get service | HDF_SUCCESS / HDF_ERR_NOT_FOUND |
| HDIServiceManager::ListAllService | C | 同步 | reply 缓冲区 | list service | HDF_SUCCESS |
| HDIDeviceManager::LoadDevice | C | 同步 | 服务名非空 | 特权 | HDF_SUCCESS |
| HDIDeviceManager::UnloadDevice | C | 同步 | 服务名非空 | 特权 | HDF_SUCCESS |

## 10. 相关文档

- [项目概览](./01_Overview.md)
- [架构说明](./03_Architecture.md)
- [内部 API](./05_Inner_API.md)
- [安全风险](./07_Security.md)

## 11. 参考代码

- `adapter/uhdf2/hdi/src/iservmgr_client.cpp` - C++ 客户端实现
- `adapter/uhdf2/hdi/src/servmgr_client.c` - C 客户端实现
- `adapter/uhdf2/hdi/src/idevmgr_client.cpp` - DeviceManager C++ 实现
- `adapter/uhdf2/hdi/src/devmgr_client.c` - DeviceManager C 实现
