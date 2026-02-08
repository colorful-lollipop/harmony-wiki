# N-API 接口文档

## 模块概览

| 模块 | N-API 模块名 | 产物 | 注册函数 |
|------|-------------|------|---------|
| USB 核心 | `@ohos.usb` | libusb.z.so | 动态加载 UsbInit |
| USB 管理 | `@ohos.usbManager` | libusbmanager.z.so | UsbInit |
| 串口 | `@ohos.usbManager.serial` | libserial.z.so | SerialInit |

## USB 核心模块 (@ohos.usb)

### 模块注册证据

**文件**: `interfaces/kits/js/napi/src/usb_middle.cpp:26-53`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "usb",
    .nm_register_func = nullptr,  // 动态加载
    .nm_modname = "usb",
};
```

**动态加载**: 使用 `dlopen("libusbmanager.z.so")` + `dlsym("UsbInit")`

### 设备操作 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `getDevices()` | CoreGetDevices | 同步 | 获取 USB 设备列表 |
| `connectDevice(device)` | CoreConnectDevice | 同步 | 连接 USB 设备 |
| `hasRight(deviceName)` | CoreHasRight | 同步 | 检查权限 |
| `requestRight(deviceName)` | CoreRequestRight | Promise | 请求设备权限 |
| `closePipe(pipe)` | PipeClose | 同步 | 关闭设备管道 |
| `resetUsbDevice(pipe)` | PipeResetDevice | 同步 | 重置 USB 设备 |

### 权限管理 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `addRight(bundleName, deviceName)` | DeviceAddRight | 同步 | 添加设备权限 |
| `removeRight(deviceName)` | DeviceRemoveRight | 同步 | 移除设备权限 |

### 接口操作 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `claimInterface(pipe, interface, force)` | PipeClaimInterface | 同步 | 声明接口 |
| `releaseInterface(pipe, interface)` | PipeReleaseInterface | 同步 | 释放接口 |
| `setInterface(pipe, interface)` | PipeSetInterface | 同步 | 设置接口 |
| `setConfiguration(pipe, config)` | PipeSetConfiguration | 同步 | 设置配置 |

### 数据传输 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `bulkTransfer(pipe, endpoint, data, timeout)` | PipeBulkTransfer | Promise | 批量传输 |
| `controlTransfer(pipe, ctrl, data)` | PipeControlTransfer | 同步 | 控制传输 |
| `usbControlTransfer(pipe, ctrlParams, data)` | PipeUsbControlTransfer | 同步 | USB 控制传输 |
| `usbCancelTransfer(pipe, endpoint)` | UsbCancelTransfer | 同步 | 取消传输 |
| `usbSubmitTransfer(pipe, transfer, cb)` | UsbSubmitTransfer | 回调 | 异步传输 |

### 描述符 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `getRawDescriptor(pipe)` | PipeGetRawDescriptors | 同步 | 获取原始描述符 |
| `getFileDescriptor(pipe)` | PipeGetFileDescriptor | 同步 | 获取文件描述符 |

### USB 功能 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `usbFunctionsFromString(funcs)` | CoreUsbFunctionsFromString | 同步 | 功能字符串转掩码 |
| `usbFunctionsToString(funcs)` | CoreUsbFunctionsToString | 同步 | 功能掩码转字符串 |
| `setCurrentFunctions(funcs)` | CoreSetCurrentFunctions | Promise | 设置当前功能 |
| `getCurrentFunctions()` | CoreGetCurrentFunctions | 同步 | 获取当前功能 |

### 端口管理 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `getPorts()` | CoreGetPorts | 同步 | 获取端口列表 |
| `getSupportedModes(portId)` | PortGetSupportedModes | 同步 | 获取支持模式 |
| `setPortRoles(portId, powerRole, dataRole)` | PortSetPortRole | Promise | 设置端口角色 |

### USB 配件 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `getAccessoryList()` | DeviceGetAccessoryList | 同步 | 获取配件列表 |
| `openAccessory(accessory)` | DeviceOpenAccessory | 同步 | 打开配件 |
| `closeAccessory(fd)` | DeviceCloseAccessory | 同步 | 关闭配件 |
| `hasAccessoryRight(accessory)` | DeviceHasAccessoryRight | 同步 | 检查配件权限 |
| `requestAccessoryRight(accessory)` | DeviceRequestAccessoryRight | Promise | 请求配件权限 |

### 常量

| 常量名 | 值类型 | 说明 |
|--------|--------|------|
| `NONE`, `SOURCE`, `SINK` | PowerRoleType | 电源角色 |
| `HOST`, `DEVICE` | DataRoleType | 数据角色 |
| `ACM`, `ECM`, `MTP`, `RNDIS` | FunctionType | USB 功能类型 |
| `USB_REQUEST_TYPE_*` | USBControlRequestType | 控制请求类型 |
| `USB_REQUEST_DIR_*` | USBRequestDirection | 传输方向 |

## USB Manager 模块 (@ohos.usbManager)

### 模块注册证据

**文件**: `interfaces/kits/js/napi/src/usbmanager_middle.cpp:20-28`

```cpp
static napi_module g_moduleManager = {
    .nm_version = 1,
    .nm_filename = "usbManager",
    .nm_register_func = UsbInit,
    .nm_modname = "usbManager",
};
```

## 串口模块 (@ohos.usbManager.serial)

### 模块注册证据

**文件**: `interfaces/kits/js/napi/src/serial_middle.cpp:20-35`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "serial",
    .nm_register_func = SerialInit,
    .nm_modname = "serial",
};
```

### 串口 API

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|----------|----------|------|
| `getPortList()` | SerialGetPortListNapi | 同步 | 获取串口列表 |
| `open(portId)` | SerialOpenNapi | Promise | 打开串口 |
| `close(portId)` | SerialCloseNapi | Promise | 关闭串口 |
| `read(portId, size, timeout)` | SerialReadNapi | Promise | 读取数据 |
| `readSync(portId, size, timeout)` | SerialReadSyncNapi | 同步 | 同步读取 |
| `write(portId, data, timeout)` | SerialWriteNapi | Promise | 写入数据 |
| `writeSync(portId, data, timeout)` | SerialWriteSyncNapi | 同步 | 同步写入 |
| `getAttribute(portId)` | SerialGetAttributeNapi | 同步 | 获取属性 |
| `setAttribute(portId, attr)` | SerialSetAttributeNapi | 同步 | 设置属性 |
| `hasSerialRight(portId)` | SerialHasRightNapi | 同步 | 检查串口权限 |
| `requestSerialRight(portId)` | SerialRequestRightNapi | Promise | 请求串口权限 |
| `addSerialRight(portId)` | SerialAddRightNapi | 同步 | 添加串口权限 |
| `cancelSerialRight(portId)` | CancelSerialRightNapi | 同步 | 取消串口权限 |

### 串口常量

| 常量名 | 值类型 | 说明 |
|--------|--------|------|
| `StopBits` | Enum | 停止位 (1, 1.5, 2) |
| `Parity` | Enum | 校验位 (None, Odd, Even) |
| `DataBits` | Enum | 数据位 (5, 6, 7, 8) |
| `BaudRates` | Enum | 波特率 |

## 异步工作模式

### Promise 模式示例

```javascript
usb.requestRight(deviceName).then((hasRight) => {
    console.log("Permission granted:", hasRight);
}).catch((error) => {
    console.error("Request failed:", error);
});
```

### Callback 模式示例

```javascript
usb.bulkTransfer(pipe, endpoint, data, 15000).then((length) => {
    console.log("Transfer complete:", length);
});
```

### 异步上下文结构

**文件**: `interfaces/kits/js/napi/include/usb_async_context.h`

```cpp
struct USBAsyncContext {
    napi_env env;
    napi_async_work work;
    napi_deferred deferred;
    napi_status status;
};

struct USBRightAsyncContext : USBAsyncContext {
    std::string deviceName;
    bool hasRight = false;
};
```

## 错误码

错误码定义在 `utils/native/src/usb_napi_errors.cpp`，通过 `CreateBusinessError()` 返回。

## 相关文档

- [架构与数据流](02_Architecture.md)
- [内部实现细节](08_Internals.md)
- [构建与产物](07_Build.md)
