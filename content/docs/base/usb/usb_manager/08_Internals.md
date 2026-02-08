# 内部 API 文档

## 模块概览

| 模块 | 头文件 | 稳定性 | 导出 |
|------|--------|--------|------|
| UsbSrvClient | usb_srv_client.h | 稳定 | innerkits |
| UsbService | usb_service.h | 稳定 | sa |
| UsbHostManager | usb_host_manager.h | 稳定 | service |
| UsbDeviceManager | usb_device_manager.h | 稳定 | service |
| UsbRightManager | usb_right_manager.h | 稳定 | service |
| UsbPortManager | usb_port_manager.h | 稳定 | service |
| SerialManager | serial_manager.h | 稳定 | service |

## UsbSrvClient

**文件**: `interfaces/innerkits/native/include/usb_srv_client.h`

**类型**: 单例客户端

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetInstance()` | UsbSrvClient& | 获取单例 |
| `OpenDevice(device, pipe)` | int32_t | 打开设备 |
| `GetDevices(deviceList)` | int32_t | 获取设备列表 |
| `HasRight(deviceName)` | bool | 检查权限 |
| `RequestRight(deviceName)` | int32_t | 请求权限 |
| `ClaimInterface(pipe, interface, force)` | int32_t | 声明接口 |
| `BulkTransfer(pipe, endpoint, data, timeout)` | int32_t | 批量传输 |
| `ControlTransfer(pipe, ctrl, data)` | int32_t | 控制传输 |
| `SetCurrentFunctions(funcs)` | int32_t | 设置功能 |
| `GetCurrentFunctions(funcs)` | int32_t | 获取功能 |
| `GetPorts(ports)` | int32_t | 获取端口 |
| `SetPortRole(portId, powerRole, dataRole)` | int32_t | 设置端口角色 |

### 依赖方向

```
UsbSrvClient
    │
    ├──▶ IUsbServer (IPC 接口)
    │
    ├──▶ IPCSkeleton (权限获取)
    │
    └──▶ DeathRecipient (服务死亡监控)
```

## UsbService

**文件**: `services/native/include/usb_service.h`

**类型**: SystemAbility (SA ID: 4201)

### 继承关系

```cpp
class UsbService : public SystemAbility, public UsbServerStub
```

### 关键方法

| 方法 | 条件编译 | 说明 |
|------|----------|------|
| `OnStart()` | - | SA 启动 |
| `OnStop()` | - | SA 停止 |
| `OpenDevice(busNum, devAddr)` | HOST | 打开设备 |
| `Close(busNum, devAddr)` | HOST | 关闭设备 |
| `ClaimInterface(busNum, devAddr, interfaceId, force)` | HOST | 声明接口 |
| `BulkTransferRead/Write()` | HOST | 批量传输 |
| `GetCurrentFunctions()` | DEVICE | 获取功能 |
| `SetCurrentFunctions(funcs)` | DEVICE | 设置功能 |
| `GetPorts()` | PORT | 获取端口 |
| `SetPortRole()` | PORT | 设置端口角色 |

### 依赖组件

```
UsbService
    │
    ├──▶ UsbHostManager (HOST)
    ├──▶ UsbDeviceManager (DEVICE)
    ├──▶ UsbPortManager (PORT)
    ├──▶ UsbRightManager
    ├──▶ SerialManager
    ├──▶ UsbAccessoryManager
    └──▶ UsbdSubscriber
```

## UsbHostManager

**文件**: `services/native/include/usb_host_manager.h`

**职责**: 主机模式 USB 设备管理

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetDevices()` | int32_t | 枚举设备 |
| `OpenDevice()` | bool | 打开设备 |
| `CloseDevice()` | bool | 关闭设备 |
| `ClaimInterface()` | int32_t | 声明接口 |
| `ReleaseInterface()` | int32_t | 释放接口 |
| `BulkTransfer()` | int32_t | 批量传输 |

### 依赖

- `UsbDescriptorParser` - USB 描述符解析
- `UsbdBulkCallBackImpl` - 传输回调

## UsbDeviceManager

**文件**: `services/native/include/usb_device_manager.h`

**职责**: 设备模式功能管理

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetCurrentFunctions()` | int32_t | 获取当前功能 |
| `SetCurrentFunctions()` | int32_t | 设置功能 |
| `UsbFunctionsFromString()` | int32_t | 字符串转功能 |
| `UsbFunctionsToString()` | int32_t | 功能转字符串 |

### 依赖

- `UsbFunctionSwitchWindow` - 功能切换 UI
- `UsbAccessoryManager` - USB 配件

## UsbRightManager

**文件**: `services/native/include/usb_right_manager.h`

**职责**: USB 权限数据库管理

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `HasRight()` | bool | 检查权限 |
| `RequestRight()` | int32_t | 请求权限 |
| `AddDeviceRight()` | bool | 添加权限 |
| `RemoveDeviceRight()` | bool | 移除权限 |
| `CleanUpRightExpired()` | int32_t | 清理过期权限 |

### 依赖

- `UsbRightDatabase` - 权限数据库
- `BundleMgr` - 包信息查询
- `AbilityConnectionStub` - 权限对话框

## UsbPortManager

**文件**: `services/native/include/usb_port_manager.h`

**职责**: USB-C 端口管理

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetPorts()` | int32_t | 获取端口列表 |
| `GetSupportedModes()` | int32_t | 获取支持模式 |
| `SetPortRole()` | int32_t | 设置端口角色 |

### 数据结构

```cpp
struct UsbPort {
    int32_t id;
    int32_t supportedModes;
    int32_t powerRole;
    int32_t dataRole;
};
```

## SerialManager

**文件**: `services/native/include/serial_manager.h`

**职责**: USB 串口设备管理

### 关键方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `OpenSerial()` | int32_t | 打开串口 |
| `CloseSerial()` | int32_t | 关闭串口 |
| `ReadSerial()` | int32_t | 读取数据 |
| `WriteSerial()` | int32_t | 写入数据 |
| `GetAttribute()` | int32_t | 获取属性 |
| `SetAttribute()` | int32_t | 设置属性 |

### 依赖

- `SerialDeathMonitor` - 串口死亡监控
- `IPCSkeleton` - 调用者权限检查

## 接口稳定性标注

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| `IUsbServer` | 稳定 | IDL 生成，保持向后兼容 |
| `UsbSrvClient` | 稳定 | innerkits 导出 |
| `UsbService` | 稳定 | SA 官方接口 |
| `UsbHostManager` | 不稳定 | service 内部实现 |
| `UsbDeviceManager` | 不稳定 | service 内部实现 |

## 错误码定义

**文件**: `utils/native/include/usb_errors.h`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| -1 | ERRCODE_NEGATIVE_ONE | 一般错误 |
| -2 | INVALID_PARAM | 无效参数 |
| -4 | NO_DEVICE | 无设备 |
| -11 | NO_MEM | 内存不足 |
| -12 | NOT_SUPPORT | 不支持 |

## 相关文档

- [架构与数据流](02_Architecture.md)
- [对外接口文档](04_Interface.md)
- [安全风险评估](06_SecurityReview.md)
