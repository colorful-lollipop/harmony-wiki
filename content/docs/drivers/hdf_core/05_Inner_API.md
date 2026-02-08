# HDF Core 内部 API

## 1. 概述

本文档描述 HDF Core 内部模块之间的接口定义，包括核心框架、OSAL、平台接口等内部 API。

**适用范围**: HDF 框架开发者、驱动开发者（高级）

## 2. 核心对象接口

### 2.1 HdfObject（基础对象）

**文件**: `interfaces/inner_api/core/hdf_object.h`

```c
struct HdfObject {
    int32_t objectId;  // 基础对象 ID
};
```

所有 HDF 对象都继承自 `HdfObject`。

### 2.2 HdfDeviceObject（设备对象）

**文件**: `interfaces/inner_api/host/shared/hdf_device_object.h`

```c
struct HdfDeviceObject {
    struct IDeviceIoService *service;     // 服务接口
    const struct DeviceResourceNode *property;  // 设备属性
    DeviceClass deviceClass;               // 设备类
    void *priv;                            // 私有数据
#ifdef __USER__
    pthread_rwlock_t mutex;                // 服务访问锁
#endif
};
```

### 2.3 HdfDriverEntry（驱动入口）

**文件**: `interfaces/inner_api/host/shared/hdf_driver.h`

```c
struct HdfDriverEntry {
    int32_t moduleVersion;                 // 驱动版本
    const char *moduleName;                // 驱动名称
    int32_t (*Bind)(struct HdfDeviceObject *deviceObject);
    int32_t (*Init)(struct HdfDeviceObject *deviceObject);
    void (*Release)(struct HdfDeviceObject *deviceObject);
};

// 驱动注册宏
#define HDF_INIT(module)  HDF_DRIVER_INIT(module)
```

### 2.4 IDeviceIoService（设备 IO 服务）

**文件**: `interfaces/inner_api/host/shared/hdf_device_desc.h:137-169`

```c
struct IDeviceIoService {
    struct HdfObject object;
    
    // 打开服务
    int32_t (*Open)(struct HdfDeviceIoClient *client);
    
    // 分发命令
    int32_t (*Dispatch)(struct HdfDeviceIoClient *client, 
                        int cmdId, 
                        struct HdfSBuf *data, 
                        struct HdfSBuf *reply);
    
    // 释放服务
    void (*Release)(struct HdfDeviceIoClient *client);
};
```

## 3. OSAL 接口（操作系统适配层）

### 3.1 内存管理

**文件**: `interfaces/inner_api/osal/shared/osal_mem.h`

```c
// 内存分配
void *OsalMemAlloc(uint32_t size);
void *OsalMemCalloc(uint32_t size);
void OsalMemFree(void *mem);

// 内存拷贝/设置
void *OsalMemCopy(void *dest, const void *src, uint32_t size);
void *OsalMemSet(void *mem, uint8_t ch, uint32_t size);
void *OsalMemMove(void *dest, const void *src, uint32_t size);
int OsalMemCmp(const void *buf1, const void *buf2, uint32_t size);
```

### 3.2 互斥锁

**文件**: `interfaces/inner_api/osal/shared/osal_mutex.h`

```c
struct OsalMutex {
    void *mutex;
};

// 互斥锁操作
int32_t OsalMutexInit(struct OsalMutex *mutex);
int32_t OsalMutexLock(struct OsalMutex *mutex);
int32_t OsalMutexUnlock(struct OsalMutex *mutex);
int32_t OsalMutexDestroy(struct OsalMutex *mutex);
```

### 3.3 信号量

**文件**: `interfaces/inner_api/osal/shared/osal_sem.h`

```c
struct OsalSem {
    void *sem;
};

// 信号量操作
int32_t OsalSemInit(struct OsalSem *sem, uint32_t value);
int32_t OsalSemWait(struct OsalSem *sem, uint32_t ms);
int32_t OsalSemPost(struct OsalSem *sem);
int32_t OsalSemDestroy(struct OsalSem *sem);
```

### 3.4 线程

**文件**: `interfaces/inner_api/osal/shared/osal_thread.h`

```c
struct OsalThread {
    void *thread;
};

typedef int32_t (*OsalThreadEntry)(void *arg);

// 线程操作
int32_t OsalThreadCreate(struct OsalThread *thread, 
                         OsalThreadEntry entry, 
                         void *arg);
int32_t OsalThreadStart(struct OsalThread *thread);
int32_t OsalThreadDestroy(struct OsalThread *thread);
```

### 3.5 时间

**文件**: `interfaces/inner_api/osal/shared/osal_time.h`

```c
// 延时
void OsalMSleep(uint32_t ms);
void OsalUSleep(uint32_t us);
void OsalNDelay(uint32_t ns);

// 获取时间
uint64_t OsalGetTime(void);
int32_t OsalGetTimeOfDay(struct OsalTimespec *tv);
```

### 3.6 自旋锁（内核态）

**文件**: `interfaces/inner_api/osal/shared/osal_spinlock.h`

```c
struct OsalSpinlock {
    void *spinlock;
};

int32_t OsalSpinLockInit(struct OsalSpinlock *spinlock);
void OsalSpinLock(struct OsalSpinlock *spinlock);
void OsalSpinUnlock(struct OsalSpinlock *spinlock);
void OsalSpinLockIrqSave(struct OsalSpinlock *spinlock, uint32_t *flags);
void OsalSpinUnlockIrqRestore(struct OsalSpinlock *spinlock, uint32_t flags);
void OsalSpinLockDestroy(struct OsalSpinlock *spinlock);
```

## 4. 平台接口（Platform API）

### 4.1 GPIO 接口

**文件**: `framework/include/platform/gpio_if.h`

```c
// GPIO 电平
enum GpioValue {
    GPIO_VAL_LOW  = 0,
    GPIO_VAL_HIGH = 1,
};

// GPIO 方向
enum GpioDirType {
    GPIO_DIR_IN  = 0,
    GPIO_DIR_OUT = 1,
};

// GPIO 中断触发类型
enum GpioIrqType {
    GPIO_IRQ_TRIGGER_RISING  = OSAL_IRQF_TRIGGER_RISING,
    GPIO_IRQ_TRIGGER_FALLING = OSAL_IRQF_TRIGGER_FALLING,
    GPIO_IRQ_TRIGGER_HIGH    = OSAL_IRQF_TRIGGER_HIGH,
    GPIO_IRQ_TRIGGER_LOW     = OSAL_IRQF_TRIGGER_LOW,
};

// GPIO 中断处理函数类型
typedef int32_t (*GpioIrqFunc)(uint16_t gpio, void *data);

// GPIO 操作
int32_t GpioRead(uint16_t gpio, uint16_t *val);
int32_t GpioWrite(uint16_t gpio, uint16_t val);
int32_t GpioSetDir(uint16_t gpio, uint16_t dir);
int32_t GpioGetDir(uint16_t gpio, uint16_t *dir);
int32_t GpioSetIrq(uint16_t gpio, uint16_t mode, GpioIrqFunc func, void *arg);
int32_t GpioUnsetIrq(uint16_t gpio, void *arg);
int32_t GpioEnableIrq(uint16_t gpio);
int32_t GpioDisableIrq(uint16_t gpio);
int32_t GpioGetByName(const char *gpioName);
```

### 4.2 I2C 接口

**文件**: `framework/include/platform/i2c_if.h`

```c
struct I2cMsg {
    uint16_t addr;      // 从机地址
    uint16_t flags;     // 标志位
    uint16_t len;       // 数据长度
    uint8_t *buf;       // 数据缓冲区
};

// I2C 操作
int32_t I2cRead(uint32_t busId, uint16_t addr, uint8_t *data, uint16_t len);
int32_t I2cWrite(uint32_t busId, uint16_t addr, uint8_t *data, uint16_t len);
int32_t I2cTransfer(uint32_t busId, struct I2cMsg *msgs, uint16_t count);
int32_t I2cOpen(uint32_t busId);
void I2cClose(uint32_t busId);
```

### 4.3 SPI 接口

**文件**: `framework/include/platform/spi_if.h`

```c
struct SpiMsg {
    uint8_t *wbuf;      // 写缓冲区
    uint8_t *rbuf;      // 读缓冲区
    uint32_t len;       // 数据长度
    uint32_t speed;     // 速度（可选）
    uint16_t delayUs;   // 延迟（可选）
};

// SPI 操作
int32_t SpiRead(uint32_t busId, uint32_t csId, uint8_t *data, uint32_t len);
int32_t SpiWrite(uint32_t busId, uint32_t csId, uint8_t *data, uint32_t len);
int32_t SpiTransfer(uint32_t busId, uint32_t csId, struct SpiMsg *msgs, uint32_t count);
int32_t SpiOpen(uint32_t busId, uint32_t csId);
void SpiClose(uint32_t busId, uint32_t csId);
```

### 4.4 UART 接口

**文件**: `framework/include/platform/uart_if.h`

```c
// UART 属性
struct UartAttribute {
    uint32_t baudRate;
    uint32_t dataBits;
    uint32_t stopBits;
    uint32_t parity;
};

// UART 操作
int32_t UartRead(uint32_t port, uint8_t *data, uint32_t len);
int32_t UartWrite(uint32_t port, uint8_t *data, uint32_t len);
int32_t UartGetBaud(uint32_t port, uint32_t *baudRate);
int32_t UartSetBaud(uint32_t port, uint32_t baudRate);
int32_t UartGetAttribute(uint32_t port, struct UartAttribute *attribute);
int32_t UartSetAttribute(uint32_t port, struct UartAttribute *attribute);
int32_t UartSetTransMode(uint32_t port, uint32_t mode);
int32_t UartOpen(uint32_t port);
void UartClose(uint32_t port);
```

## 5. IPC 接口

### 5.1 HdfRemoteService

**文件**: `interfaces/inner_api/ipc/hdf_remote_service.h`

```c
struct HdfRemoteService {
    struct HdfObject object;
    struct HdfObject *target;
    struct HdfRemoteDispatcher *dispatcher;
    uint64_t index;
};

struct HdfRemoteDispatcher {
    int32_t (*Dispatch)(struct HdfRemoteService *service, 
                        int code, 
                        struct HdfSBuf *data, 
                        struct HdfSBuf *reply);
    int32_t (*DispatchAsync)(struct HdfRemoteService *service, 
                             int code, 
                             struct HdfSBuf *data);
};

// 远程服务操作
struct HdfRemoteService *HdfRemoteServiceObtain(
    struct HdfObject *target, 
    struct HdfRemoteDispatcher *dispatcher);
void HdfRemoteServiceRecycle(struct HdfRemoteService *service);
int32_t HdfRemoteServiceAddDeathRecipient(
    struct HdfRemoteService *service, 
    struct HdfDeathRecipient *recipient);
void HdfRemoteServiceRemoveDeathRecipient(
    struct HdfRemoteService *service, 
    struct HdfDeathRecipient *recipient);

// 获取调用者信息
pid_t HdfRemoteGetCallingPid(void);
uid_t HdfRemoteGetCallingUid(void);
char *HdfRemoteGetCallingSid(void);
```

### 5.2 HdfSBuf（序列化缓冲区）

**文件**: `interfaces/inner_api/utils/hdf_sbuf.h`

```c
// 创建/销毁
struct HdfSBuf *HdfSBufCreate(uint32_t capacity);
void HdfSBufRecycle(struct HdfSBuf *sbuf);

// 写入操作
bool HdfSbufWriteInt8(struct HdfSBuf *sbuf, int8_t value);
bool HdfSbufWriteInt16(struct HdfSBuf *sbuf, int16_t value);
bool HdfSbufWriteInt32(struct HdfSBuf *sbuf, int32_t value);
bool HdfSbufWriteInt64(struct HdfSBuf *sbuf, int64_t value);
bool HdfSbufWriteUint8(struct HdfSBuf *sbuf, uint8_t value);
bool HdfSbufWriteUint16(struct HdfSBuf *sbuf, uint16_t value);
bool HdfSbufWriteUint32(struct HdfSBuf *sbuf, uint32_t value);
bool HdfSbufWriteUint64(struct HdfSBuf *sbuf, uint64_t value);
bool HdfSbufWriteBuffer(struct HdfSBuf *sbuf, const void *data, uint32_t size);
bool HdfSbufWriteString(struct HdfSBuf *sbuf, const char *value);
bool HdfSbufWriteRemoteService(struct HdfSBuf *sbuf, struct HdfRemoteService *service);

// 读取操作
int8_t HdfSbufReadInt8(struct HdfSBuf *sbuf);
int16_t HdfSbufReadInt16(struct HdfSBuf *sbuf);
int32_t HdfSbufReadInt32(struct HdfSBuf *sbuf);
int64_t HdfSbufReadInt64(struct HdfSBuf *sbuf);
uint8_t HdfSbufReadUint8(struct HdfSBuf *sbuf);
uint16_t HdfSbufReadUint16(struct HdfSBuf *sbuf);
uint32_t HdfSbufReadUint32(struct HdfSBuf *sbuf);
uint64_t HdfSbufReadUint64(struct HdfSBuf *sbuf);
const void *HdfSbufReadBuffer(struct HdfSBuf *sbuf, uint32_t *size);
const char *HdfSbufReadString(struct HdfSBuf *sbuf);
struct HdfRemoteService *HdfSbufReadRemoteService(struct HdfSBuf *sbuf);
```

## 6. 服务管理接口

### 6.1 DevSvcManager（服务管理器）

**文件**: `framework/core/manager/include/devsvc_manager.h`

```c
struct IDevSvcManager {
    struct HdfObject object;
    struct IDevSvcManagerSuper *super;
    
    // 添加服务
    int32_t (*AddService)(struct IDevSvcManager *inst, 
                          struct HdfDeviceObject *serviceObject, 
                          const struct HdfServiceInfo *info);
    
    // 获取服务
    struct HdfDeviceObject *(*GetService)(struct IDevSvcManager *inst, 
                                          const char *svcName);
    
    // 移除服务
    int32_t (*RemoveService)(struct IDevSvcManager *inst, 
                             const char *svcName, 
                             struct HdfDeviceObject *serviceObject);
    
    // 列出所有服务
    void (*ListAllService)(struct IDevSvcManager *inst, struct HdfSBuf *reply);
    
    // 注册服务状态监听器
    int32_t (*RegsterServListener)(struct IDevSvcManager *inst, 
                                   struct ServStatListenerHolder *listener);
    
    // 注销服务状态监听器
    int32_t (*UnregsterServListener)(struct IDevSvcManager *inst, 
                                     struct ServStatListenerHolder *listener);
};

// 获取服务管理器实例
struct IDevSvcManager *DevSvcManagerGetInstance(void);
```

### 6.2 DevMgrService（设备管理服务）

**文件**: `framework/core/manager/include/devmgr_service.h`

```c
struct IDevmgrService {
    struct HdfObject object;
    struct IDevmgrServiceSuper *super;
    
    // 附加设备主机
    int32_t (*AttachDeviceHost)(struct IDevmgrService *inst, 
                                uint16_t hostId, 
                                struct IDevHostService *hostService);
    
    // 附加设备
    int32_t (*AttachDevice)(struct IDevmgrService *inst, 
                            struct HdfDeviceToken *token);
    
    // 分离设备
    int32_t (*DetachDevice)(struct IDevmgrService *inst, 
                            uint32_t deviceId);
    
    // 加载设备
    int32_t (*LoadDevice)(struct IDevmgrService *inst, 
                          const char *serviceName);
    
    // 卸载设备
    int32_t (*UnloadDevice)(struct IDevmgrService *inst, 
                            const char *serviceName);
    
    // 列出所有设备
    int32_t (*ListAllDevice)(struct IDevmgrService *inst, 
                             struct HdfSBuf *reply);
};
```

## 7. 模块依赖关系

### 7.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                        Driver Layer                         │
│                   (驱动实现，调用平台接口)                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Platform Layer                          │
│              (GPIO/I2C/SPI/UART/PWM 接口)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       OSAL Layer                            │
│           (Mem/Mutex/Sem/Thread/Time/Spinlock)              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Kernel/OS Layer                        │
│              (Linux/LiteOS/UniProton 内核)                   │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 接口稳定性

| 接口层级 | 稳定性 | 说明 |
|----------|--------|------|
| Platform API | 稳定 | 向后兼容，可放心使用 |
| OSAL API | 稳定 | 向后兼容 |
| HDI API | 稳定 | 对外接口，版本化管理 |
| Core Internal | 不稳定 | 框架内部使用，可能变更 |
| Driver Models | 中等 | 各模型独立演进 |

## 8. 相关文档

- [项目概览](./01_Overview.md)
- [架构说明](./03_Architecture.md)
- [HDI 接口](./04_HDI_API.md)
- [GN 构建](./06_GN_Build.md)
