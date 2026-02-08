# 项目概览

> **目的**：让新人快速理解 USB Manager 是什么、能做什么、如何使用
> **适用范围**：USB Manager v3.1.0
> **更新时间**：2026-02-07

---

## 一句话定义

**USB Manager** 是 OpenHarmony 系统中的 USB 设备管理服务，提供 Host/Device/Port 三大功能模块，通过 JS API 和 C++ 内部接口支持应用层 USB 设备访问、数据传输、权限控制。

**关键词**：
- USB 设备管理
- 权限控制
- 数据传输
- Host/Device/Port 模式

---

## 能力边界

### ✅ 能做什么

#### 1. USB Host 模式（主机模式）

**功能描述**：设备作为 USB Host，连接外部 USB 设备进行通信

**支持的操作**：
| 操作 | 说明 | JS API |
|------|------|--------|
| 设备枚举 | 扫描并列出可用 USB 设备 | `usb.getDevices()` |
| 设备连接 | 打开 USB 设备建立连接 | `usb.connectDevice()` |
| 权限管理 | 请求、检查、移除设备访问权限 | `usb.requestRight()`, `usb.hasRight()` |
| 接口操作 | 声明、释放、切换 USB 接口 | `usb.claimInterface()`, `usb.releaseInterface()` |
| 配置管理 | 设置 USB 设备配置 | `usb.setConfiguration()` |
| 数据传输 | 批量传输、控制传输 | `usb.bulkTransfer()`, `usb.controlTransfer()` |
| 设备控制 | 重置设备、清除端点停止状态 | `usb.resetUsbDevice()`, `usb.clearHalt()` |
| 描述符获取 | 获取原始设备描述符 | `usb.getRawDescriptor()` |
| 文件描述符 | 获取设备文件描述符（用于高级操作） | `usb.getFileDescriptor()` |

**使用场景**：
- 连接 USB 存储设备读写数据
- 连接 USB 摄像头、打印机等外设
- 与 USB 串口设备通信
- 与自定义 USB 设备交互

**证据**：`interfaces/kits/js/napi/src/usb_info.cpp` - Host 功能实现

---

#### 2. USB Device 模式（从机模式）

**功能描述**：设备作为 USB Device，通过 USB 连接到主机

**支持的功能**：
| 功能 | 说明 | JS API |
|------|------|--------|
| 功能切换 | 设置 USB Device 功能模式 | `usbmanager.setCurrentFunctions()` |
| 功能查询 | 获取当前 USB Device 功能 | `usbmanager.getCurrentFunctions()` |
| 功能转换 | 字符串/功能掩码转换 | `usbmanager.usbFunctionsFromString()`, `usbmanager.usbFunctionsToString()` |
| 配件管理 | USB Accessory 模式支持 | `usbmanager.getAccessoryList()`, `usbmanager.openAccessory()` |
| 配件权限 | 配件访问权限管理 | `usbmanager.requestAccessoryRight()`, `usbmanager.hasAccessoryRight()` |

**支持的 USB 功能**：
- **ACM** (USB Communication Class)：串口模拟
- **ECM** (Ethernet Control Model)：以太网模拟
- **MTP** (Media Transfer Protocol)：媒体传输
- **RNDIS** (Remote Network Driver Interface Specification)：网络驱动模拟
- **HDC** (Huawei Debug Bridge)：华为调试桥

**使用场景**：
- 设备作为 USB 串口设备连接电脑
- 设备作为 USB 网络设备共享网络
- 设备作为 USB 存储设备传输文件
- 设备作为 USB 调试设备

**证据**：`interfaces/kits/js/napi/src/usb_info.cpp` - Device 功能实现

---

#### 3. USB Port 管理（端口模式）

**功能描述**：管理 USB-C 端口的角色配置

**支持的操作**：
| 操作 | 说明 | JS API |
|------|------|--------|
| 端口枚举 | 获取所有 USB Port 列表 | `usbmanager.getPorts()` |
| 角色设置 | 设置 Port 的电源角色和数据角色 | `usbmanager.setPortRoles()` |
| 模式查询 | 获取 Port 支持的模式 | `usbmanager.getSupportedModes()` |

**端口角色**：
- **Power Role**：Source（供电）、Sink（受电）
- **Data Role**：Host（主机）、Device（从机）

**使用场景**：
- USB-C PD（Power Delivery）角色切换
- USB OTG 自动角色协商
- USB-C 充电和数据传输模式切换

**证据**：`interfaces/kits/js/napi/src/usb_info.cpp` - Port 功能实现

---

#### 4. USB Serial（串口管理）

**功能描述**：USB 串口设备的专用管理接口

**支持的操作**：
| 操作 | 说明 | JS API |
|------|------|--------|
| 串口枚举 | 获取所有 USB 串口列表 | `serial.getPortList()` |
| 串口操作 | 打开、关闭、读写串口 | `serial.open()`, `serial.close()`, `serial.read()`, `serial.write()` |
| 属性配置 | 设置波特率、数据位、停止位、校验位 | `serial.setAttribute()`, `serial.getAttribute()` |
| 权限管理 | 串口访问权限管理 | `serial.requestSerialRight()`, `serial.hasSerialRight()` |

**使用场景**：
- 与 USB 转 RS232/485 设备通信
- 与 USB 调试串口设备通信
- 与 USB 模块通信（如 4G/5G 模块）

**证据**：`interfaces/kits/js/napi/src/serial_info.cpp` - Serial 功能实现

---

#### 5. 权限管理

**功能描述**：USB 设备和串口的访问权限控制

**权限模型**：
- **应用首次访问**：弹出权限对话框，用户授权
- **权限存储**：权限信息存储在本地数据库
- **权限验证**：每次访问 USB 设备前检查权限
- **权限撤销**：可移除已授予的权限

**权限 API**：
| 操作 | Host 模式 | Device 模式 | Serial 模式 |
|------|-----------|------------|------------|
| 请求权限 | `usb.requestRight()` | `usbmanager.requestAccessoryRight()` | `serial.requestSerialRight()` |
| 检查权限 | `usb.hasRight()` | `usbmanager.hasAccessoryRight()` | `serial.hasSerialRight()` |
| 添加权限 | `usb.addRight()` | `usbmanager.addAccessoryRight()` | `serial.addSerialRight()` |
| 移除权限 | `usb.removeRight()` | `usbmanager.cancelAccessoryRight()` | `serial.cancelSerialRight()` |

**证据**：`services/native/src/usb_right_manager.cpp` - 权限管理实现

---

### ❌ 不能做什么

| 限制 | 说明 | 原因 |
|------|------|------|
| 不支持 ISO 同步传输（直接接口） | 需要通过 Request 接口间接实现 | ISO 传输需要特殊的异步处理 |
| 不能直接访问内核 USB 驱动 | 必须通过 HAL 层接口 | 安全隔离要求 |
| 不能绕过权限检查访问设备 | 所有敏感操作都需权限验证 | 安全机制 |
| 不支持 USB OTG 自动切换 | 需要应用层通过 Port API 控制角色 | 架构设计 |
| 不支持 USB 3.x 高速特性 | 当前仅支持 USB 2.0 基本功能 | HAL 层限制 |
| 不支持 USB Hub 管理 | 无法直接管理 USB Hub 设备 | 功能范围限制 |

---

## 运行环境

### 系统要求

| 要求 | 版本/配置 | 说明 |
|------|----------|------|
| 操作系统 | OpenHarmony Standard 3.0+ | USB Manager 是标准系统组件 |
| 子系统 | usb | USB 子系统必须启用 |
| 系统能力 | SystemCapability.USB.USBManager | 必须声明 USB 能力 |
| 运行时权限 | ohos.permission.USB_MANAGER | 应用需申请权限 |

### 依赖的系统服务

| 系统服务 | 用途 | 证据位置 |
|---------|------|---------|
| **System Ability Manager** | 注册 USB Service (SA ID: 4201) | `sa_profile/4201.json` |
| **Bundle Manager** | 获取应用信息用于权限验证 | `bundle.json` 依赖 |
| **Access Token 服务** | 验证调用者身份和权限 | `bundle.json` access_token 依赖 |
| **HDF 驱动框架** | 提供 USB HAL 驱动接口 | `bundle.json` hdf_core 依赖 |
| **Data Share** | 存储权限数据库 | `bundle.json` data_share 依赖 |
| **Common Event Service** | 设备插拔事件通知 | `bundle.json` common_event_service 依赖 |
| **HiSysEvent** | 安全事件上报 | `bundle.json` hisysevent 依赖 |

### 编译产物

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|---------|------|
| `libusbservice.z.so` | System Ability | `system/lib64/` | USB Service 主服务 |
| `libusb.z.so` | N-API | `system/lib64/module/` | USB 核心接口 |
| `libusbmanager.z.so` | N-API | `system/lib64/module/` | USB Manager 接口 |
| `libserial.z.so` | N-API | `system/lib64/module/usbmanager/` | Serial 接口 |
| `libusbsrv_client.z.so` | IPC 客户端 | `system/lib64/` | 内部客户端库 |

**证据**：`interfaces/kits/js/napi/BUILD.gn`, `services/BUILD.gn`

---

## 快速开始示例

### 示例 1：USB Host 设备枚举和数据传输

**场景**：连接 USB 存储设备并读取数据

```javascript
// 导入 USB API
import usb from '@ohos.usb';

// 1. 获取设备列表
const deviceList = usb.getDevices();
console.log(`Found ${deviceList.length} USB devices`);

if (deviceList.length === 0) {
    console.log('No USB device found');
    return;
}

// 2. 选择第一个设备
const device = deviceList[0];

// 3. 请求设备访问权限
try {
    const hasRight = await usb.requestRight(device.name);
    if (!hasRight) {
        console.log('Permission denied');
        return;
    }
} catch (error) {
    console.error('Request right failed:', error);
    return;
}

// 4. 打开设备
const pipe = usb.connectDevice(device);
console.log('Device connected:', pipe);

// 5. 声明接口（假设使用第一个接口）
const interface = device.configs[0].interfaces[0];
usb.claimInterface(pipe, interface, true);

// 6. 执行批量传输
const inEndpoint = interface.endpoints.find(ep => ep.direction === 0x80); // IN 端点
const data = new Uint8Array(1024);
const timeout = 15000; // 15 秒超时

try {
    const length = await usb.bulkTransfer(pipe, inEndpoint, data, timeout);
    if (length >= 0) {
        console.log(`Received ${length} bytes`);
        console.log('Data:', data.subarray(0, length));
    } else {
        console.log('Transfer failed');
    }
} catch (error) {
    console.error('Transfer error:', error);
}

// 7. 清理资源
usb.releaseInterface(pipe, interface);
usb.closePipe(pipe);
console.log('Device closed');
```

**关键点**：
- ✅ 必须先请求权限才能访问设备
- ✅ 使用 `claimInterface` 声明接口才能进行数据传输
- ✅ `bulkTransfer` 返回 Promise，需要异步处理
- ✅ 传输完成后必须释放接口和关闭管道

**证据**：`README.md:82-151` - Host 开发示例

---

### 示例 2：USB Device 功能切换

**场景**：将设备设置为 USB Device 模式，启用 ACM（串口）功能

```javascript
// 导入 USB API
import usbManager from '@ohos.usbManager';

// 定义 USB 功能常量
const FunctionType = {
    ACM: 0x01,
    ECM: 0x02,
    MTP: 0x04,
    RNDIS: 0x08,
    HDC: 0x10
};

// 1. 查询当前功能
try {
    const currentFuncs = usbManager.getCurrentFunctions();
    console.log('Current functions:', currentFuncs);
} catch (error) {
    console.error('Get current functions failed:', error);
}

// 2. 设置 USB 功能（启用 ACM）
const targetFuncs = FunctionType.ACM;

try {
    await usbManager.setCurrentFunctions(targetFuncs);
    console.log('Set current functions success');
} catch (error) {
    console.error('Set current functions failed:', error);
}

// 3. 验证功能设置
try {
    const newFuncs = usbManager.getCurrentFunctions();
    console.log('New functions:', newFuncs);

    if (newFuncs === targetFuncs) {
        console.log('Function switch success');
    } else {
        console.log('Function switch failed');
    }
} catch (error) {
    console.error('Verify functions failed:', error);
}
```

**关键点**：
- ✅ USB 功能使用位掩码表示，支持组合多个功能
- ✅ `setCurrentFunctions` 返回 Promise，需要异步处理
- ✅ 功能切换需要一定时间，建议验证设置结果

**证据**：`README.md:155-166` - Device 开发示例

---

### 示例 3：USB Serial 串口通信

**场景**：通过 USB 串口与设备通信

```javascript
// 导入 Serial API
import serial from '@ohos.usbManager.serial';

// 1. 获取串口列表
try {
    const portList = serial.getPortList();
    console.log('Serial ports:', portList);

    if (portList.length === 0) {
        console.log('No serial port found');
        return;
    }

    // 2. 选择第一个串口
    const portId = portList[0].portId;
    console.log('Using port:', portId);

    // 3. 请求串口权限
    const hasRight = await serial.requestSerialRight(portId);
    if (!hasRight) {
        console.log('Permission denied');
        return;
    }

    // 4. 打开串口
    await serial.open(portId);
    console.log('Serial port opened');

    // 5. 配置串口参数（9600, 8N1）
    const attr = {
        baudRate: serial.BaudRates.BAUD_RATE_9600,
        dataBits: serial.DataBits.DATA_BITS_8,
        stopBits: serial.StopBits.STOP_BITS_1,
        parity: serial.Parity.PARITY_NONE
    };

    await serial.setAttribute(portId, attr);
    console.log('Serial port configured');

    // 6. 写入数据
    const sendData = 'Hello Serial';
    const writeBuffer = new Uint8Array(sendData.length);
    for (let i = 0; i < sendData.length; i++) {
        writeBuffer[i] = sendData.charCodeAt(i);
    }

    const writeLength = await serial.write(portId, writeBuffer, 5000);
    console.log(`Wrote ${writeLength} bytes`);

    // 7. 读取数据
    const readBuffer = new Uint8Array(1024);
    const readLength = await serial.read(portId, readBuffer, 5000);
    console.log(`Read ${readLength} bytes`);
    console.log('Data:', new TextDecoder().decode(readBuffer.subarray(0, readLength)));

    // 8. 关闭串口
    await serial.close(portId);
    console.log('Serial port closed');

} catch (error) {
    console.error('Serial operation error:', error);
}
```

**关键点**：
- ✅ 串口操作需要权限，使用 `requestSerialRight` 请求
- ✅ 串口参数（波特率、数据位等）使用 `setAttribute` 配置
- ✅ `read` 和 `write` 都是异步操作，返回 Promise
- ✅ 必须在操作完成后关闭串口

**证据**：`interfaces/kits/js/napi/src/serial_info.cpp` - Serial API 实现

---

## 架构概览

### 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│                   应用层（Application）                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  @ohos.usb  │  │usbManager   │  │@ohos.usbManager.serial│  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
└─────────┼────────────────┼───────────────────┼─────────────┘
          │                │                   │
          ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                   N-API Layer                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  libusb.z   │  │libusbmanager│  │    libserial.z      │  │
│  │  .so        │  │.z.so        │  │                     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                Inner API Layer (IPC)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            libusbsrv_client.z.so                      │  │
│  │    (通过 IUsbServer 接口与 SA 通信)                   │  │
│  └────────────────────┬─────────────────────────────────┘  │
└───────────────────────┼────────────────────────────────────┘
                         │ IPC (Binder)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                USB Service (SA ID: 4201)                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │UsbHostManager│  │UsbDeviceManager│ │UsbRightManager     │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬─────────────┘  │
│         │                │                 │                │
│  ┌──────┴──────┐  ┌──────┴──────┐  ┌──────┴─────────────┐  │
│  │UsbPortManager│  │SerialManager │  │UsbAccessoryManager │  │
│  └──────────────┘  └─────────────┘  └────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    USB HAL Layer                            │
│         (drivers_peripheral USB Driver)                     │
└─────────────────────────────────────────────────────────────┘
```

**层次职责**：

| 层次 | 组件 | 职责 |
|------|------|------|
| **应用层** | 应用代码 | 调用 JS API 实现 USB 功能 |
| **N-API 层** | libusb.z.so 等 | 参数校验、类型转换、异步处理 |
| **IPC 层** | libusbsrv_client.z.so | 跨进程通信、序列化 |
| **服务层** | UsbService (SA) | 业务逻辑、权限管理、设备管理 |
| **HAL 层** | USB Driver | 硬件访问、内核驱动接口 |

**证据**：`README.md:11-20` - 架构说明

---

## 关键概念

### USB 权限机制

**目的**：保护 USB 设备访问，防止恶意应用滥用

**权限流程**：
```
1. 应用首次访问 USB 设备
   ↓
2. 弹出权限对话框（UI）
   ↓
3. 用户授权/拒绝
   ↓
4. 权限结果存储到数据库
   ↓
5. 后续访问直接查询数据库
```

**权限存储**：
- 位置：`usb_right_database`
- 格式：Bundle Name / Device Name / Token ID 映射
- 有效期：永久（除非用户手动撤销）

**证据**：`services/native/src/usb_right_manager.cpp`

---

### USB 模式切换

**Host 模式**：设备作为 USB Host，连接外设（如 U 盘、摄像头）

**Device 模式**：设备作为 USB Device，连接主机（如电脑）

**Port 模式**：USB-C 端口角色配置（Source/Sink, Host/Device）

**切换方式**：
- 应用层调用：`usbmanager.setPortRoles()`
- 系统自动协商：USB OTG 检测（需硬件支持）

**证据**：`services/native/src/usb_port_manager.cpp`

---

## 相关文档

- [架构与数据流](02_Architecture.md) - 详细的架构设计和数据流向
- [对外接口文档](04_Interface.md) - 完整的 API 文档和使用示例
- [攻击面分析](05_AttackSurface.md) - 外部输入点和安全风险
- [目录结构与代码地图](03_CodeMap.md) - 快速定位代码位置

---

**更新时间**：2026-02-07
**证据来源**：`README.md`, `bundle.json`, `interfaces/kits/js/napi/`
