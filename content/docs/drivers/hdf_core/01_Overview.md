# HDF Core 项目概览

## 1. 项目定位

**HDF Core**（Hardware Driver Foundation Core）是 OpenHarmony 驱动子系统的核心框架，提供跨平台的硬件驱动开发环境，支持"一次开发，多系统部署"。

### 核心定位
- **驱动框架核心**: 提供驱动加载、服务管理、消息模型的基础能力
- **跨 OS 迁移**: 屏蔽不同操作系统（Linux、LiteOS、UniProton）的差异
- **标准化接口**: 通过 HDI（Hardware Driver Interface）向系统服务提供统一硬件访问接口
- **多系统支持**: 适配 standard、small、mini 三种系统类型

## 2. 核心能力

### 2.1 驱动生命周期管理
```c
// 驱动入口结构体 (hdf_device_desc.h:202-240)
struct HdfDriverEntry {
    int32_t moduleVersion;
    const char *moduleName;
    int32_t (*Bind)(struct HdfDeviceObject *deviceObject);
    int32_t (*Init)(struct HdfDeviceObject *deviceObject);
    void (*Release)(struct HdfDeviceObject *deviceObject);
};
```

- **Bind**: 绑定驱动设备与功能接口
- **Init**: 初始化驱动
- **Release**: 释放驱动资源

### 2.2 服务管理
- **服务注册**: 驱动可向框架注册服务，供其他组件调用
- **服务发现**: 支持按名称、按设备类发现服务
- **服务订阅**: 支持服务状态变更监听

### 2.3 IPC 通信
- 基于 OpenHarmony Binder IPC 框架
- 支持同步/异步调用
- 支持跨进程服务访问

### 2.4 配置管理
- **HCS**（HDF Configuration Source）配置源
- 支持设备树形式的层次化配置
- 编译时配置解析（hc-gen 工具）

### 2.5 平台抽象
- GPIO、I2C、SPI、UART、PWM 等平台驱动接口
- OSAL（Operating System Abstraction Layer）操作系统适配层

## 3. 运行环境

### 3.1 支持的操作系统
| OS | 内核态 (KHDF) | 用户态 (UHDF) |
|----|--------------|--------------|
| Linux | ✅ | ✅ |
| LiteOS-A | ✅ | ❌ |
| LiteOS-M | ✅ | ❌ |
| UniProton | ✅ | ❌ |

### 3.2 支持的系统类型
- **standard**: 标准系统（手机、平板等）
- **small**: 小型系统（智能手表、智能音箱等）
- **mini**: 轻量系统（传感器、模组等）

### 3.3 资源占用
- **ROM**: 735KB
- **RAM**: 1350KB

### 3.4 依赖组件
- hilog / hilog_lite（日志）
- c_utils（通用工具）
- init（启动初始化）
- ipc（进程间通信）
- samgr（服务管理）
- selinux_adapter（安全增强）
- hicollie（看门狗）
- bounds_checking_function（安全函数库）

## 4. 关键概念

### 4.1 架构分层

```
┌─────────────────────────────────────────────────────────────┐
│                    System Services                          │
│         (使用 HDI 接口访问硬件)                               │
├─────────────────────────────────────────────────────────────┤
│                    HDI Layer                                │
│    IServiceManager / IDeviceManager / IDriverInterface      │
├─────────────────────────────────────────────────────────────┤
│                    UHDF (User Space)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   DevMgr    │  │  DevHost    │  │  Service Manager    │ │
│  │  (设备管理)  │  │  (驱动主机)  │  │    (服务管理)        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    KHDF (Kernel Space)                      │
│         Core Framework / Driver Models / Platform           │
├─────────────────────────────────────────────────────────────┤
│                    Hardware                                 │
│              GPIO / I2C / SPI / UART / etc.                 │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 核心组件

#### DevMgr（设备管理器）
- 系统唯一，以 System Ability 形式运行（SA ID: 5100）
- 负责设备加载、卸载、查询
- 管理 DevHost 进程生命周期

#### DevHost（驱动主机）
- 每个 host 配置对应一个进程
- 运行实际驱动代码
- 通过 IPC 与 DevMgr 通信

#### Service Manager（服务管理器）
- 管理驱动服务的注册与发现
- 提供服务状态监听机制

### 4.3 驱动模型

HDF 提供多种通用驱动模型：

| 模型 | 说明 | 路径 |
|------|------|------|
| Audio | 音频框架 | framework/model/audio |
| Camera | 相机框架 | framework/model/camera |
| Display | 显示框架 | framework/model/display |
| Input | 输入框架 | framework/model/input |
| Network | WLAN 框架 | framework/model/network |
| Sensor | 传感器框架 | framework/model/sensor |
| Storage | 存储框架 | framework/model/storage |
| USB | USB 框架 | framework/model/usb |

### 4.4 接口类型

#### 对外接口（HDI）
- C++ 接口: `IServiceManager`, `IDeviceManager`
- C 接口: `HDIServiceManager`, `HDIDeviceManager`
- 位置: `interfaces/inner_api/hdi/`

#### 平台接口
- GPIO: `GpioRead`, `GpioWrite`, `GpioSetIrq` (framework/include/platform/gpio_if.h)
- I2C: `I2cRead`, `I2cWrite`, `I2cTransfer`
- SPI: `SpiRead`, `SpiWrite`, `SpiTransfer`
- UART: `UartRead`, `UartWrite`, `UartSetBaud`

#### 内部接口
- Core: `HdfObject`, `HdfDeviceObject`
- OSAL: `OsalMemAlloc`, `OsalMutexLock`, `OsalThreadCreate`

## 5. 安全特性

### 5.1 SELinux 集成
- 服务操作权限检查（Add/Get/List Service）
- 基于 Calling SID 的访问控制

### 5.2 进程隔离
- 驱动运行在独立 DevHost 进程
- 进程崩溃不影响其他驱动

### 5.3 路径安全
- 模块加载路径验证（realpath + 前缀检查）
- 防止路径遍历攻击

### 5.4 权限位图
- 硬件访问权限细粒度控制
- I2C、SPI、GPIO 等独立权限位

## 6. 版本信息

- **当前版本**: 4.0
- **License**: Apache License 2.0
- **双许可**: GPL / BSD（可选）

## 7. 相关文档

- [目录结构](./02_Directory_Structure.md)
- [架构说明](./03_Architecture.md)
- [HDI 接口](./04_HDI_API.md)
- [安全风险](./07_Security.md)

## 8. 参考链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Driver 子系统](https://gitee.com/openharmony/docs/blob/master/en/readme/driver.md)
- [HDF 开发指南](https://gitee.com/openharmony/docs/blob/master/en/device-dev/driver/driver-hdf-manage.md)
