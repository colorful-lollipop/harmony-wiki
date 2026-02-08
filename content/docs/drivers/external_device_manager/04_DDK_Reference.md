# DDK C API 接口参考

本文档描述扩展外部设备管理模块提供的 Driver Development Kit（DDK）C API 接口，供驱动开发者调用以访问 USB、HID、USB Serial、SCSI 等外部设备。DDK 接口采用 C 语言设计，遵循 OpenHarmony NDK 规范。

## DDK 概述

DDK（Driver Development Kit）是一套为设备驱动开发者提供的底层硬件访问接口。与 N-API（面向应用开发者）不同，DDK 面向驱动扩展 Ability 的开发者，提供直接操作硬件的能力。目前支持以下设备类型：

| DDK 模块 | 头文件 | 库文件 | 权限 |
|---------|--------|--------|------|
| Base DDK | ddk_api.h | libddk_base.z.so | 无 |
| USB DDK | usb_ddk_api.h | libusb_ndk.z.so | ohos.permission.ACCESS_DDK_USB |
| HID DDK | hid_ddk_api.h | libhid.z.so | ohos.permission.ACCESS_DDK_HID |
| USB Serial DDK | usb_serial_api.h | libusb_serial_ndk.z.so | ohos.permission.ACCESS_DDK_USB_SERIAL |
| SCSI DDK | scsi_peripheral_api.h | libscsi.z.so | ohos.permission.ACCESS_DDK_SCSI_PERIPHERAL |

## Base DDK

Base DDK 提供共享内存（Ashmem）管理功能，可用于在进程间高效传输大数据。

### 头文件

```c
#include <ddk_api.h>
#include <ddk_types.h>
```

### 类型定义

```c
typedef struct DDK_Ashmem {
    int32_t ashmemFd;           // 共享内存文件描述符
    const uint8_t *address;      // 映射地址
    const uint32_t size;         // 内存大小
    uint32_t offset;             // 当前偏移
    uint32_t bufferLength;       // 缓冲区长度
    uint32_t transferredLength;  // 已传输长度
} DDK_Ashmem;

typedef enum {
    DDK_SUCCESS = 0,
    DDK_FAILURE = 28600001,
    DDK_INVALID_PARAMETER = 28600002,
    DDK_INVALID_OPERATION = 28600003,
    DDK_NULL_PTR = 28600004
} DDK_RetCode;
```

### 接口函数

**OH_DDK_CreateAshmem**

创建匿名共享内存。

```c
DDK_RetCode OH_DDK_CreateAshmem(const uint8_t *name, uint32_t size, DDK_Ashmem **ashmem);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| name | `const uint8_t *` | 共享内存名称 |
| size | `uint32_t` | 共享内存大小（字节） |
| ashmem | `DDK_Ashmem **` | 输出参数，返回创建的共享内存句柄 |

| 返回值 | 说明 |
|--------|------|
| `DDK_SUCCESS` | 创建成功 |
| `DDK_INVALID_PARAMETER` | 参数无效（name 为空、size 为 0、ashmem 为空） |
| `DDK_FAILURE` | 创建失败（errno 包含具体错误） |

**OH_DDK_MapAshmem**

映射共享内存到进程地址空间。

```c
DDK_RetCode OH_DDK_MapAshmem(DDK_Ashmem *ashmem, const uint8_t ashmemMapType);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| ashmem | `DDK_Ashmem *` | 共享内存句柄 |
| ashmemMapType | `const uint8_t` | 映射类型（0-7，对应 PROT_READ、PROT_WRITE 等组合） |

| 返回值 | 说明 |
|--------|------|
| `DDK_SUCCESS` | 映射成功 |
| `DDK_NULL_PTR` | ashmem 为空 |
| `DDK_INVALID_OPERATION` | 映射类型非法或映射失败 |

**OH_DDK_UnmapAshmem**

取消共享内存映射。

```c
DDK_RetCode OH_DDK_UnmapAshmem(DDK_Ashmem *ashmem);
```

**OH_DDK_DestroyAshmem**

销毁共享内存。

```c
DDK_RetCode OH_DDK_DestroyAshmem(DDK_Ashmem *ashmem);
```

## USB DDK

USB DDK 提供 USB 设备的完整访问接口，包括设备枚举、接口声明、控制传输和批量传输等。

### 头文件

```c
#include <usb_ddk_api.h>
#include <usb_ddk_types.h>
```

### 错误码

```c
typedef enum {
    USB_DDK_SUCCESS = 0,
    USB_DDK_NO_PERM = 201,
    USB_DDK_INVALID_PARAMETER = 401,
    USB_DDK_MEMORY_ERROR = 27400001,
    USB_DDK_INVALID_OPERATION = 27400002,
    USB_DDK_IO_FAILED = 27400003,
    USB_DDK_TIMEOUT = 27400004,
} UsbDdkErrCode;
```

### 接口函数

**OH_Usb_Init**

初始化 USB DDK。

```c
int32_t OH_Usb_Init(void);
```

**OH_Usb_Release**

释放 USB DDK 资源。

```c
void OH_Usb_Release(void);
```

**OH_Usb_GetDeviceDescriptor**

获取 USB 设备描述符。

```c
int32_t OH_Usb_GetDeviceDescriptor(uint64_t deviceId, struct UsbDeviceDescriptor *desc);
```

**OH_Usb_GetConfigDescriptor**

获取 USB 配置描述符。

```c
int32_t OH_Usb_GetConfigDescriptor(uint64_t deviceId, uint8_t configIndex, 
    struct UsbDdkConfigDescriptor **config);
```

**OH_Usb_FreeConfigDescriptor**

释放配置描述符内存。

```c
void OH_Usb_FreeConfigDescriptor(struct UsbDdkConfigDescriptor *config);
```

**OH_Usb_ClaimInterface**

声明接口，获取操作该接口的句柄。

```c
int32_t OH_Usb_ClaimInterface(uint64_t deviceId, uint8_t interfaceIndex, 
    uint64_t *interfaceHandle);
```

**OH_Usb_ReleaseInterface**

释放接口声明。

```c
int32_t OH_Usb_ReleaseInterface(uint64_t interfaceHandle);
```

**OH_Usb_SelectInterfaceSetting**

选择接口的备用设置。

```c
int32_t OH_Usb_SelectInterfaceSetting(uint64_t interfaceHandle, uint8_t settingIndex);
```

**OH_Usb_GetCurrentInterfaceSetting**

获取当前选中的备用设置。

```c
int32_t OH_Usb_GetCurrentInterfaceSetting(uint64_t interfaceHandle, uint8_t *settingIndex);
```

**OH_Usb_SendControlReadRequest**

发送控制读取请求（端点 0）。

```c
int32_t OH_Usb_SendControlReadRequest(uint64_t interfaceHandle, 
    const struct UsbControlRequestSetup *setup, uint32_t timeout, 
    uint8_t *data, uint32_t *dataLen);
```

**OH_Usb_SendControlWriteRequest**

发送控制写入请求（端点 0）。

```c
int32_t OH_Usb_SendControlWriteRequest(uint64_t interfaceHandle, 
    const struct UsbControlRequestSetup *setup, uint32_t timeout, 
    const uint8_t *data, uint32_t dataLen);
```

**OH_Usb_SendPipeRequest**

发送数据请求（批量/中断传输）。

```c
int32_t OH_Usb_SendPipeRequest(const struct UsbRequestPipe *pipe, 
    UsbDeviceMemMap *devMmap);
```

**OH_Usb_SendPipeRequestWithAshmem**

通过 Ashmem 发送数据请求。

```c
int32_t OH_Usb_SendPipeRequestWithAshmem(const struct UsbRequestPipe *pipe, 
    DDK_Ashmem *ashmem);
```

**OH_Usb_CreateDeviceMemMap**

创建设备内存映射。

```c
int32_t OH_Usb_CreateDeviceMemMap(uint64_t deviceId, size_t size, 
    UsbDeviceMemMap **devMmap);
```

**OH_Usb_DestroyDeviceMemMap**

销毁设备内存映射。

```c
void OH_Usb_DestroyDeviceMemMap(UsbDeviceMemMap *devMmap);
```

**OH_Usb_GetDevices**

获取设备列表。

```c
int32_t OH_Usb_GetDevices(struct Usb_DeviceArray *devices);
```

## HID DDK

HID DDK 提供人机交互设备（HID）的创建、事件发送和读写接口。

### 头文件

```c
#include <hid_ddk_api.h>
#include <hid_ddk_types.h>
```

### 错误码

```c
typedef enum {
    HID_DDK_SUCCESS = 0,
    HID_DDK_NO_PERM = 201,
    HID_DDK_INVALID_PARAMETER = 401,
    HID_DDK_FAILURE = 27300001,
    HID_DDK_NULL_PTR = 27300002,
    HID_DDK_INVALID_OPERATION = 27300003,
    HID_DDK_TIMEOUT = 27300004,
    HID_DDK_INIT_ERROR = 27300005,
    HID_DDK_SERVICE_ERROR = 27300006,
    HID_DDK_MEMORY_ERROR = 27300007,
    HID_DDK_IO_ERROR = 27300008,
    HID_DDK_DEVICE_NOT_FOUND = 27300009
} Hid_DdkErrCode;
```

### 类型定义

```c
typedef struct Hid_EmitItem {
    uint16_t type;   // 事件类型 (HID_EV_*)
    uint16_t code;   // 事件代码
    uint32_t value;  // 事件值
} Hid_EmitItem;

typedef struct Hid_Device {
    const char *deviceName;
    uint16_t vendorId;
    uint16_t productId;
    uint16_t version;
    uint16_t bustype;
    Hid_DeviceProp *properties;
    uint16_t propLength;
} Hid_Device;

typedef struct Hid_DeviceHandle Hid_DeviceHandle;
```

### 接口函数

**OH_Hid_CreateDevice**（自 API 11）

创建虚拟 HID 设备。

```c
int32_t OH_Hid_CreateDevice(Hid_Device *hidDevice, Hid_EventProperties *hidEventProperties);
```

**OH_Hid_EmitEvent**

发送 HID 事件。

```c
int32_t OH_Hid_EmitEvent(int32_t deviceId, const Hid_EmitItem items[], uint16_t length);
```

**OH_Hid_DestroyDevice**

销毁 HID 设备。

```c
int32_t OH_Hid_DestroyDevice(int32_t deviceId);
```

**OH_Hid_Init**（自 API 16）

初始化 HID DDK。

```c
int32_t OH_Hid_Init(void);
```

**OH_Hid_Release**

释放 HID DDK 资源。

```c
int32_t OH_Hid_Release(void);
```

**OH_Hid_Open**

打开物理 HID 设备。

```c
int32_t OH_Hid_Open(uint64_t deviceId, uint8_t interfaceIndex, Hid_DeviceHandle **dev);
```

**OH_Hid_Close**

关闭 HID 设备。

```c
int32_t OH_Hid_Close(Hid_DeviceHandle **dev);
```

**OH_Hid_Write**

向 HID 设备写入数据。

```c
int32_t OH_Hid_Write(Hid_DeviceHandle *dev, uint8_t *data, uint32_t length, uint32_t *bytesWritten);
```

**OH_Hid_ReadTimeout**

带超时的读取 HID 设备。

```c
int32_t OH_Hid_ReadTimeout(Hid_DeviceHandle *dev, uint8_t *data, uint32_t bufSize, 
    int timeout, uint32_t *bytesRead);
```

**OH_Hid_Read**

读取 HID 设备（阻塞）。

```c
int32_t OH_Hid_Read(Hid_DeviceHandle *dev, uint8_t *data, uint32_t bufSize, uint32_t *bytesRead);
```

**OH_Hid_SendReport**

发送报告到 HID 设备。

```c
int32_t OH_Hid_SendReport(Hid_DeviceHandle *dev, Hid_ReportType reportType, 
    const uint8_t *data, uint32_t length);
```

**OH_Hid_GetReport**

从 HID 设备获取报告。

```c
int32_t OH_Hid_GetReport(Hid_DeviceHandle *dev, Hid_ReportType reportType, 
    uint8_t *data, uint32_t bufSize);
```

## USB Serial DDK

USB Serial DDK 提供 USB 转串口设备的通信接口。

### 头文件

```c
#include <usb_serial_api.h>
#include <usb_serial_types.h>
```

### 错误码

```c
typedef enum {
    USB_SERIAL_DDK_NO_PERM = 201,
    USB_SERIAL_DDK_INVALID_PARAMETER = 401,
    USB_SERIAL_DDK_SUCCESS = 31600000,
    USB_SERIAL_DDK_INVALID_OPERATION = 31600001,
    USB_SERIAL_DDK_INIT_ERROR = 31600002,
    USB_SERIAL_DDK_SERVICE_ERROR = 31600003,
    USB_SERIAL_DDK_MEMORY_ERROR = 31600004,
    USB_SERIAL_DDK_IO_ERROR = 31600005,
    USB_SERIAL_DDK_DEVICE_NOT_FOUND = 31600006,
} UsbSerial_DdkRetCode;
```

### 类型定义

```c
typedef struct UsbSerial_Params {
    uint32_t baudRate;   // 波特率
    uint8_t nDataBits;   // 数据位 (5-8)
    uint8_t nStopBits;   // 停止位 (1-2)
    uint8_t parity;     // 校验位 (0:无, 1:奇, 2:偶)
} UsbSerial_Params;

typedef enum {
    USB_SERIAL_NO_FLOW_CONTROL = 0,
    USB_SERIAL_SOFTWARE_FLOW_CONTROL = 1,
    USB_SERIAL_HARDWARE_FLOW_CONTROL = 2,
} UsbSerial_FlowControl;

typedef enum {
    USB_SERIAL_PARITY_NONE = 0,
    USB_SERIAL_PARITY_ODD = 1,
    USB_SERIAL_PARITY_EVEN = 2,
} UsbSerial_Parity;
```

### 接口函数

**OH_UsbSerial_Init**

初始化 USB Serial DDK。

```c
int32_t OH_UsbSerial_Init(void);
```

**OH_UsbSerial_Release**

释放 USB Serial DDK 资源。

```c
int32_t OH_UsbSerial_Release(void);
```

**OH_UsbSerial_Open**

打开 USB。

```c
 串口设备int32_t OH_Open(uint64_UsbSerial_t deviceId, uint8_t interfaceIndex, 
    UsbSerial_Device **dev);
```

**OH_UsbSerial_Close**

关闭 USB 串口设备。

```c
int32_t OH_UsbSerial_Close(UsbSerial_Device **dev);
```

**OH_UsbSerial_Read**

读取数据。

```c
int32_t OH_UsbSerial_Read(UsbSerial_Device *dev, uint8_t *buff, 
    uint32_t bufferSize, uint32_t *bytesRead);
```

**OH_UsbSerial_Write**

写入数据。

```c
int32_t OH_UsbSerial_Write(UsbSerial_Device *dev, uint8_t *buff, 
    uint32_t bufferSize, uint32_t *bytesWritten);
```

**OH_UsbSerial_SetBaudRate**

设置波特率。

```c
int32_t OH_UsbSerial_SetBaudRate(UsbSerial_Device *dev, uint32_t baudRate);
```

**OH_UsbSerial_SetParams**

设置串口参数。

```c
int32_t OH_UsbSerial_SetParams(UsbSerial_Device *dev, UsbSerial_Params *params);
```

**OH_UsbSerial_SetTimeout**

设置超时。

```c
int32_t OH_UsbSerial_SetTimeout(UsbSerial_Device *dev, int timeout);
```

**OH_UsbSerial_SetFlowControl**

设置流控。

```c
int32_t OH_UsbSerial_SetFlowControl(UsbSerial_Device *dev, 
    UsbSerial_FlowControl flowControl);
```

**OH_UsbSerial_Flush**

刷新缓冲区。

```c
int32_t OH_UsbSerial_Flush(UsbSerial_Device *dev);
```

## SCSI Peripheral DDK

SCSI Peripheral DDK 提供 SCSI 存储设备的访问接口。

### 头文件

```c
#include <scsi_peripheral_api.h>
#include <scsi_peripheral_types.h>
```

### 错误码

```c
typedef enum {
    SCSIPERIPHERAL_DDK_SUCCESS = 31700000,
    SCSIPERIPHERAL_DDK_NO_PERM = 201,
    SCSIPERIPHERAL_DDK_INVALID_PARAMETER = 401,
    SCSIPERIPHERAL_DDK_MEMORY_ERROR = 31700001,
    SCSIPERIPHERAL_DDK_INVALID_OPERATION = 31700002,
    SCSIPERIPHERAL_DDK_IO_ERROR = 31700003,
    SCSIPERIPHERAL_DDK_TIMEOUT = 31700004,
    SCSIPERIPHERAL_DDK_INIT_ERROR = 31700005,
    SCSIPERIPHERAL_DDK_SERVICE_ERROR = 31700006,
    SCSIPERIPHERAL_DDK_DEVICE_NOT_FOUND = 31700007,
} ScsiPeripheral_DdkErrCode;
```

### 常量定义

```c
#define SCSIPERIPHERAL_MIN_DESCRIPTOR_FORMAT_SENSE 8
#define SCSIPERIPHERAL_MIN_FIXED_FORMAT_SENSE 18
#define SCSIPERIPHERAL_MAX_CMD_DESC_BLOCK_LEN 16
#define SCSIPERIPHERAL_MAX_SENSE_DATA_LEN 252
#define SCSIPERIPHERAL_VENDOR_ID_LEN 8
#define SCSIPERIPHERAL_PRODUCT_ID_LEN 16
#define SCSIPERIPHERAL_PRODUCT_REV_LEN 4
```

### 接口函数

**OH_ScsiPeripheral_Init**

初始化 SCSI DDK。

```c
int32_t OH_ScsiPeripheral_Init(void);
```

**OH_ScsiPeripheral_Release**

释放 SCSI DDK 资源。

```c
int32_t OH_ScsiPeripheral_Release(void);
```

**OH_ScsiPeripheral_Open**

打开 SCSI 设备。

```c
int32_t OH_ScsiPeripheral_Open(uint64_t deviceId, uint8_t interfaceIndex, 
    ScsiPeripheral_Device **dev);
```

**OH_ScsiPeripheral_Close**

关闭 SCSI 设备。

```c
int32_t OH_ScsiPeripheral_Close(ScsiPeripheral_Device **dev);
```

**OH_ScsiPeripheral_TestUnitReady**

测试设备就绪状态。

```c
int32_t OH_ScsiPeripheral_TestUnitReady(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_TestUnitReadyRequest *request, ScsiPeripheral_Response *response);
```

**OH_ScsiPeripheral_Inquiry**

查询设备信息（INQUIRY 命令）。

```c
int32_t OH_ScsiPeripheral_Inquiry(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_InquiryRequest *request, ScsiPeripheral_InquiryInfo *inquiryInfo, 
    ScsiPeripheral_Response *response);
```

**OH_ScsiPeripheral_ReadCapacity10**

读取设备容量（READ CAPACITY 10 命令）。

```c
int32_t OH_ScsiPeripheral_ReadCapacity10(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_ReadCapacityRequest *request, ScsiPeripheral_CapacityInfo *capacityInfo, 
    ScsiPeripheral_Response *response);
```

**OH_ScsiPeripheral_Read10**

读取数据块（READ 10 命令）。

```c
int32_t OH_ScsiPeripheral_Read10(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_IORequest *request, ScsiPeripheral_Response *response);
```

**OH_ScsiPeripheral_Write10**

写入数据块（WRITE 10 命令）。

```c
int32_t OH_ScsiPeripheral_Write10(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_IORequest *request, ScsiPeripheral_Response *response);
```

**OH_ScsiPeripheral_SendRequestByCdb**

发送自定义 SCSI 命令。

```c
int32_t OH_ScsiPeripheral_SendRequestByCdb(ScsiPeripheral_Device *dev, 
    ScsiPeripheral_Request *request, ScsiPeripheral_Response *response);
```

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
