# 攻击面分析

> **目的**：识别 USB Manager 的所有外部输入点和敏感操作，为安全研究提供攻击面地图
> **适用范围**：USB Manager v3.1.0
> **更新时间**：2026-02-07

---

## 外部输入清单

### 1. N-API 接口（应用层输入）

**攻击面等级**：**高**

**入口点**：`@ohos.usb`, `@ohos.usbManager`, `@ohos.usbManager.serial`

**风险说明**：应用层 JS 代码可直接调用 N-API，所有参数均可被恶意构造

#### USB Host 功能输入

| JS API | 参数类型 | 攻击面 | 输入验证位置 |
|--------|---------|--------|-------------|
| `getDevices()` | 无 | 信息泄露（设备列表） | `usb_info.cpp:CoreGetDevices` |
| `connectDevice(device)` | UsbDevice 对象 | 设备 ID 注入、DoS | `usb_info.cpp:CoreConnectDevice` |
| `hasRight(deviceName)` | string (deviceName) | 路径遍历、DoS | `usb_info.cpp:CoreHasRight` |
| `requestRight(deviceName)` | string (deviceName) | 路径遍历、权限提升 | `usb_info.cpp:CoreRequestRight` |
| `closePipe(pipe)` | USBDevicePipe | 资源释放攻击、UAF | `usb_info.cpp:PipeClose` |
| `resetUsbDevice(pipe)` | USBDevicePipe | DoS | `usb_info.cpp:PipeResetDevice` |
| `claimInterface(pipe, interface, force)` | USBDevicePipe, int, bool | 竞态条件、DoS | `usb_info.cpp:PipeClaimInterface` |
| `releaseInterface(pipe, interface)` | USBDevicePipe, int | 资源释放、UAF | `usb_info.cpp:PipeReleaseInterface` |
| `setConfiguration(pipe, config)` | USBDevicePipe, int | DoS | `usb_info.cpp:PipeSetConfiguration` |
| `setInterface(pipe, interface)` | USBDevicePipe, int | DoS | `usb_info.cpp:PipeSetInterface` |
| `bulkTransfer(pipe, endpoint, data, timeout)` | USBDevicePipe, USBEndpoint, Uint8Array, int | 缓冲区溢出、信息泄露、DoS | `usb_info.cpp:PipeBulkTransfer` |
| `controlTransfer(pipe, ctrl, data)` | USBDevicePipe, UsbCtrlTransfer, Uint8Array | 控制命令注入、DoS | `usb_info.cpp:PipeControlTransfer` |
| `usbControlTransfer(pipe, ctrlParams, data)` | USBDevicePipe, UsbCtrlTransfer, Uint8Array | 控制命令注入 | `usb_info.cpp:PipeUsbControlTransfer` |
| `usbCancelTransfer(pipe, endpoint)` | USBDevicePipe, int | 资源竞争 | `usb_info.cpp:UsbCancelTransfer` |
| `usbSubmitTransfer(pipe, transfer, cb)` | USBDevicePipe, UsbTransInfo, callback | 回调注入、内存泄露 | `usb_info.cpp:UsbSubmitTransfer` |
| `getRawDescriptor(pipe)` | USBDevicePipe | 信息泄露 | `usb_info.cpp:PipeGetRawDescriptors` |
| `getFileDescriptor(pipe)` | USBDevicePipe | 文件描述符泄露 | `usb_info.cpp:PipeGetFileDescriptor` |
| `addRight(bundleName, deviceName)` | string, string | 权限提升 | `usb_info.cpp:DeviceAddRight` |
| `removeRight(deviceName)` | string | 权限绕过 | `usb_info.cpp:DeviceRemoveRight` |

#### USB Device 功能输入

| JS API | 参数类型 | 攻击面 | 输入验证位置 |
|--------|---------|--------|-------------|
| `getCurrentFunctions()` | 无 | 信息泄露 | `usb_info.cpp:CoreGetCurrentFunctions` |
| `setCurrentFunctions(funcs)` | int | DoS、功能劫持 | `usb_info.cpp:CoreSetCurrentFunctions` |
| `usbFunctionsFromString(funcs)` | string | 字符串解析漏洞 | `usb_info.cpp:CoreUsbFunctionsFromString` |
| `usbFunctionsToString(funcs)` | int | 缓冲区溢出 | `usb_info.cpp:CoreUsbFunctionsToString` |
| `getAccessoryList()` | 无 | 信息泄露 | `usb_info.cpp:DeviceGetAccessoryList` |
| `openAccessory(accessory)` | USBAccessory | 设备访问攻击 | `usb_info.cpp:DeviceOpenAccessory` |
| `closeAccessory(fd)` | int | 资源释放攻击 | `usb_info.cpp:DeviceCloseAccessory` |
| `hasAccessoryRight(accessory)` | USBAccessory | 权限绕过 | `usb_info.cpp:DeviceHasAccessoryRight` |
| `requestAccessoryRight(accessory)` | USBAccessory | 权限提升 | `usb_info.cpp:DeviceRequestAccessoryRight` |

#### USB Port 功能输入

| JS API | 参数类型 | 攻击面 | 输入验证位置 |
|--------|---------|--------|-------------|
| `getPorts()` | 无 | 信息泄露 | `usb_info.cpp:CoreGetPorts` |
| `getSupportedModes(portId)` | int | 数组越界、DoS | `usb_info.cpp:PortGetSupportedModes` |
| `setPortRoles(portId, powerRole, dataRole)` | int, int, int | DoS、状态劫持 | `usb_info.cpp:PortSetPortRole` |

#### Serial 功能输入

| JS API | 参数类型 | 攻击面 | 输入验证位置 |
|--------|---------|--------|-------------|
| `getPortList()` | 无 | 信息泄露 | `serial_info.cpp:SerialGetPortListNapi` |
| `open(portId)` | int | 竞态条件、资源泄露 | `serial_info.cpp:SerialOpenNapi` |
| `close(portId)` | int | 资源释放攻击 | `serial_info.cpp:SerialCloseNapi` |
| `read(portId, size, timeout)` | int, int, int | 缓冲区溢出、DoS | `serial_info.cpp:SerialReadNapi` |
| `readSync(portId, size, timeout)` | int, int, int | 缓冲区溢出 | `serial_info.cpp:SerialReadSyncNapi` |
| `write(portId, data, timeout)` | int, Uint8Array, int | 缓冲区溢出、DoS | `serial_info.cpp:SerialWriteNapi` |
| `writeSync(portId, data, timeout)` | int, Uint8Array, int | 缓冲区溢出 | `serial_info.cpp:SerialWriteSyncNapi` |
| `getAttribute(portId)` | int | 数组越界 | `serial_info.cpp:SerialGetAttributeNapi` |
| `setAttribute(portId, attr)` | int, UsbSerialAttr | 竞态条件、DoS | `serial_info.cpp:SerialSetAttributeNapi` |
| `hasSerialRight(portId)` | int | 权限绕过 | `serial_info.cpp:SerialHasRightNapi` |
| `requestSerialRight(portId)` | int | 权限提升 | `serial_info.cpp:SerialRequestRightNapi` |
| `addSerialRight(portId)` | int | 权限提升 | `serial_info.cpp:SerialAddRightNapi` |
| `cancelSerialRight(portId)` | int | 权限绕过 | `serial_info.cpp:CancelSerialRightNapi` |

---

### 2. IPC 接口（跨进程输入）

**攻击面等级**：**高**

**入口点**：`IUsbServer` 接口（`interfaces/innerkits/IUsbServer.idl`）

**风险说明**：其他进程可通过 Binder IPC 调用 USB Service，需要验证调用者身份

#### IPC 方法清单（共 95 个方法）

**Host 功能（USB_MANAGER_FEATURE_HOST）**：

| IPC 方法 | 参数 | 风险 | 证据位置 |
|---------|------|------|---------|
| `GetDevices` | [out]UsbDevice[] | 信息泄露 | `IUsbServer.idl:27` |
| `OpenDevice` | busNum, devAddr | DoS | `IUsbServer.idl:28` |
| `Close` | busNum, devAddr | 资源释放 | `IUsbServer.idl:29` |
| `ResetDevice` | busNum, devAddr | DoS | `IUsbServer.idl:30` |
| `ClaimInterface` | busNum, devAddr, interfaceid, force | 竞态条件 | `IUsbServer.idl:31` |
| `SetInterface` | busNum, devAddr, interfaceid, altIndex | DoS | `IUsbServer.idl:32` |
| `ReleaseInterface` | busNum, devAddr, interfaceid | 资源释放 | `IUsbServer.idl:33` |
| `SetActiveConfig` | busNum, devAddr, configId | DoS | `IUsbServer.idl:34` |
| `ManageGlobalInterface` | disable | DoS | `IUsbServer.idl:35` |
| `ManageDevice` | vendorId, productId, disable | 策略绕过 | `IUsbServer.idl:36` |
| `ManageDevicePolicy` | trustList | 策略注入 | `IUsbServer.idl:37` |
| `ManageInterfaceType` | disableType, disable | 策略注入 | `IUsbServer.idl:38` |
| `UsbAttachKernelDriver` | busNum, devAddr, interfaceid | 内核操作 | `IUsbServer.idl:39` |
| `UsbDetachKernelDriver` | busNum, devAddr, interfaceid | 内核操作 | `IUsbServer.idl:40` |
| `ClearHalt` | busNum, devAddr, interfaceid, endpointId | DoS | `IUsbServer.idl:41` |
| `GetActiveConfig` | busNum, devAddr, [out]configId | 信息泄露 | `IUsbServer.idl:42` |
| `GetRawDescriptor` | busNum, devAddr, [out]bufferData | 信息泄露 | `IUsbServer.idl:43` |
| `GetFileDescriptor` | busNum, devAddr, [out]fd | 文件描述符泄露 | `IUsbServer.idl:44` |
| `GetDeviceSpeed` | busNum, devAddr, [out]speed | 信息泄露 | `IUsbServer.idl:45` |
| `GetInterfaceActiveStatus` | busNum, devAddr, interfaceid, [out]unactivated | 信息泄露 | `IUsbServer.idl:46` |
| `BulkTransferRead` | busNum, devAddr, ep, [out]buffData, timeOut | 缓冲区溢出 | `IUsbServer.idl:47` |
| `BulkTransferWrite` | busNum, devAddr, ep, buffData, timeOut | 缓冲区溢出 | `IUsbServer.idl:48` |
| `BulkTransferReadwithLength` | busNum, devAddr, ep, length, [out]buffData, timeOut | 缓冲区溢出 | `IUsbServer.idl:49` |
| `ControlTransfer` | busNum, devAddr, ctrlParams, [inout]bufferData | 控制命令注入 | `IUsbServer.idl:50` |
| `UsbControlTransfer` | busNum, devAddr, ctrlParams, [inout]bufferData | 控制命令注入 | `IUsbServer.idl:51` |
| `RequestQueue` | busNum, devAddr, ep, clientData, bufferData | 内存损坏 | `IUsbServer.idl:52` |
| `RequestWait` | busNum, devAddr, timeOut, [inout]clientData, [inout]bufferData | 竞态条件 | `IUsbServer.idl:53` |
| `RequestCancel` | busNum, devAddr, interfaceid, endpointId | 资源竞争 | `IUsbServer.idl:54` |
| `UsbCancelTransfer` | busNum, devAddr, endpoint | 资源竞争 | `IUsbServer.idl:55` |
| `UsbSubmitTransfer` | busNum, devAddr, info, cb, fd, memSize | 回调注入 | `IUsbServer.idl:56` |
| `RegBulkCallback` | busNum, devAddr, ep, cb | 回调注入 | `IUsbServer.idl:57` |
| `UnRegBulkCallback` | busNum, devAddr, ep | 资源释放 | `IUsbServer.idl:58` |
| `BulkRead` | busNum, devAddr, ep, ashmem, memSize | 缓冲区溢出 | `IUsbServer.idl:59` |
| `BulkWrite` | busNum, devAddr, ep, ashmem, memSize | 缓冲区溢出 | `IUsbServer.idl:60` |
| `BulkCancel` | busNum, devAddr, ep | 资源竞争 | `IUsbServer.idl:61` |
| `HasRight` | deviceName, [out]hasRight | 权限绕过 | `IUsbServer.idl:62` |
| `RequestRight` | deviceName | 权限提升 | `IUsbServer.idl:63` |
| `RemoveRight` | deviceName | 权限绕过 | `IUsbServer.idl:64` |
| `AddRight` | bundleName, deviceName | 权限提升 | `IUsbServer.idl:65` |
| `AddAccessRight` | tokenId, deviceName | 权限提升 | `IUsbServer.idl:66` |

**Device 功能（USB_MANAGER_FEATURE_DEVICE）**：

| IPC 方法 | 参数 | 风险 | 证据位置 |
|---------|------|------|---------|
| `GetCurrentFunctions` | [out]funcs | 信息泄露 | `IUsbServer.idl:68` |
| `SetCurrentFunctions` | funcs | DoS、功能劫持 | `IUsbServer.idl:69` |
| `UsbFunctionsFromString` | funcs | 字符串解析漏洞 | `IUsbServer.idl:70` |
| `UsbFunctionsToString` | funcs | 缓冲区溢出 | `IUsbServer.idl:71` |
| `AddAccessoryRight` | tokenId, access | 权限提升 | `IUsbServer.idl:72` |
| `HasAccessoryRight` | access, [out]checkResult | 权限绕过 | `IUsbServer.idl:73` |
| `RequestAccessoryRight` | access, [out]checkResult | 权限提升 | `IUsbServer.idl:74` |
| `CancelAccessoryRight` | access | 权限绕过 | `IUsbServer.idl:75` |
| `GetAccessoryList` | [out]accessList | 信息泄露 | `IUsbServer.idl:76` |
| `OpenAccessory` | access, [out]fd | 设备访问攻击 | `IUsbServer.idl:77` |
| `CloseAccessory` | fd | 资源释放攻击 | `IUsbServer.idl:78` |

**Port 功能（USB_MANAGER_FEATURE_PORT）**：

| IPC 方法 | 参数 | 风险 | 证据位置 |
|---------|------|------|---------|
| `GetPorts` | [out]ports | 信息泄露 | `IUsbServer.idl:80` |
| `GetSupportedModes` | portId, [out]supportedModes | 数组越界 | `IUsbServer.idl:81` |
| `SetPortRole` | portId, powerRole, dataRole | DoS、状态劫持 | `IUsbServer.idl:82` |

**Serial 功能**：

| IPC 方法 | 参数 | 风险 | 证据位置 |
|---------|------|------|---------|
| `SerialOpen` | portId, serialRemote | 竞态条件 | `IUsbServer.idl:84` |
| `SerialClose` | portId | 资源释放攻击 | `IUsbServer.idl:85` |
| `SerialRead` | portId, [out]buffData, size, [out]actualSize, timeout | 缓冲区溢出 | `IUsbServer.idl:86` |
| `SerialWrite` | portId, buffData, size, [out]actualSize, timeout | 缓冲区溢出 | `IUsbServer.idl:87` |
| `SerialGetAttribute` | portId, [out]attribute | 数组越界 | `IUsbServer.idl:88` |
| `SerialSetAttribute` | portId, attribute | 竞态条件、DoS | `IUsbServer.idl:89` |
| `SerialGetPortList` | [out]serialPortList | 信息泄露 | `IUsbServer.idl:90` |
| `AddSerialRight` | tokenId, portId | 权限提升 | `IUsbServer.idl:91` |
| `HasSerialRight` | portId, [out]hasRight | 权限绕过 | `IUsbServer.idl:92` |
| `RequestSerialRight` | portId, [out]hasRight | 权限提升 | `IUsbServer.idl:93` |
| `CancelSerialRight` | portId | 权限绕过 | `IUsbServer.idl:94` |

---

### 3. 配置文件（静态输入）

**攻击面等级**：**中**

#### SA Profile 配置

**文件位置**：`sa_profile/4201.json`

**配置内容**：
```json
{
    "process": "usb_service",
    "systemability": [{
        "name": 4201,
        "libpath": "libusbservice.z.so",
        "run-on-create": false,
        "auto-restart": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**风险**：
- libpath 路径劫持
- 进程名欺骗
- auto-restart DoS 攻击

#### Init 配置

**文件位置**：`services/usb_service.cfg`

**风险**：
- 服务启动参数注入
- 权限配置绕过

#### 系统参数

**文件位置**：`etc/param/`

**风险**：
- Feature Flag 篡改
- 系统参数注入

---

### 4. USB 设备输入（硬件输入）

**攻击面等级**：**极高**

**入口点**：USB 物理端口

**攻击向量**：
1. **恶意 USB 设备**
   - BadUSB（键盘攻击）
   - USB 描述符漏洞（缓冲区溢出）
   - USB 串行攻击

2. **设备描述符注入**
   - 恶意构造的端点信息
   - 超长配置描述符
   - 特殊 USB 类描述符

**相关代码**：
- 描述符解析：`services/native/src/usb_descriptor_parser.cpp`
- 设备枚举：`services/native/src/usb_host_manager.cpp`
- 设备插拔通知：`services/native/src/usb_connection_notifier.cpp`

---

### 5. HAL 层接口（驱动输入）

**攻击面等级**：**高**

**入口点**：`drivers_interface_usb` HAL 接口

**风险说明**：USB Manager 调用 HAL 层接口与内核驱动通信，HAL 层输入未充分验证可能导致内核漏洞

**相关接口**：
- `IUsbClient`（Host 模式）
- `IUsbFunction`（Device 模式）
- `IUsbPort`（Port 管理）
- `ISerial`（串口管理）

**证据**：`services/BUILD.gn` 中依赖 `drivers_interface_usb`

---

## 敏感操作清单

### 1. 系统调用（特权操作）

| 操作 | 代码位置 | 权限要求 | 风险 |
|------|---------|---------|------|
| 文件描述符操作 | `PipeGetFileDescriptor()` | USB 设备访问 | 文件描述符泄露 |
| 内核驱动交互 | HAL 层调用 | USB 设备访问 | 内核漏洞利用 |
| 数据库写入 | `usb_right_database.cpp` | 系统权限 | 数据库注入 |

---

### 2. 特权接口调用

| 接口 | 用途 | 权限要求 | 代码位置 |
|------|------|---------|---------|
| `ManageDevice` | 设备黑/白名单管理 | EDM 权限 | `usb_service.cpp` |
| `ManageDevicePolicy` | 策略管理 | EDM 权限 | `usb_service.cpp` |
| `UsbAttachKernelDriver` | 附加内核驱动 | Root 权限 | `usb_service.cpp` |
| `UsbDetachKernelDriver` | 分离内核驱动 | Root 权限 | `usb_service.cpp` |
| `AddAccessRight` | 添加访问权限 | 系统权限 | `usb_service.cpp` |

**证据**：`IUsbServer.idl:36-40, 66`

---

### 3. 数据库操作

| 操作 | 数据库 | 代码位置 | 风险 |
|------|-------|---------|------|
| 查询权限 | usb_right_database | `usb_right_database.cpp` | SQL 注入 |
| 添加权限 | usb_right_database | `usb_right_database.cpp` | SQL 注入 |
| 删除权限 | usb_right_database | `usb_right_database.cpp` | SQL 注入 |

---

### 4. 网络操作

**当前状态**：无网络操作

**潜在风险**：如未来增加网络功能，需注意：
- 远程代码执行
- 中间人攻击
- 数据泄露

---

### 5. 资源管理操作

| 操作 | 风险 | 代码位置 |
|------|------|---------|
| 内存分配 | Use-After-Free | `usb_info.cpp` |
| 互斥锁操作 | 死锁 | `usb_service.cpp:291-294` |
| 文件描述符管理 | 文件描述符泄露 | `usb_info.cpp:PipeGetFileDescriptor` |
| 异步任务管理 | 任务泄露 | `usb_info.cpp:PipeBulkTransfer` |

---

## 信任边界

### 信任域划分

```
┌─────────────────────────────────────────────────────────────┐
│                   不可信域（应用层）                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  用户应用（可能被恶意应用或 Root 用户控制）              ││
│  └────────────────────┬────────────────────────────────────┘│
└───────────────────────┼─────────────────────────────────────┘
                        │ N-API 边界
                        ▼
┌─────────────────────────────────────────────────────────────┐
│              半可信域（N-API 层）                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  libusb.z.so / libusbmanager.z.so / libserial.z.so   ││
│  │  - 参数校验                                        ││
│  │  - 类型转换                                        ││
│  │  - 错误处理                                        ││
│  └────────────────────┬────────────────────────────────────┘│
└───────────────────────┼─────────────────────────────────────┘
                        │ IPC 边界（Binder）
                        ▼
┌─────────────────────────────────────────────────────────────┐
│              可信域（System Ability 层）                    │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  UsbService (SA ID: 4201)                           ││
│  │  - IPCSkeleton 权限检查                              ││
│  │  - Access Token 验证                                 ││
│  │  - 业务逻辑处理                                      ││
│  └────────────────────┬────────────────────────────────────┘│
└───────────────────────┼─────────────────────────────────────┘
                        │ HAL 边界
                        ▼
┌─────────────────────────────────────────────────────────────┐
│               高可信域（HAL 驱动层）                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  USB HAL (drivers_peripheral)                         ││
│  │  - 内核驱动接口                                      ││
│  │  - 硬件访问                                         ││
│  └────────────────────┬────────────────────────────────────┘│
└───────────────────────┼─────────────────────────────────────┘
                        │ 硬件边界
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                不可信硬件域                                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  USB 设备（可能被恶意设备控制）                         ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 边界保护机制

#### N-API 边界保护

**保护机制**：
1. 参数类型验证（N-API 自动处理）
2. 数组长度检查
3. Null 指针检查

**代码位置**：`interfaces/kits/js/napi/src/napi_util.cpp`

**局限性**：
- ✗ 未验证业务逻辑（如 deviceName 格式）
- ✗ 未验证数值范围（如 portId 范围）

#### IPC 边界保护

**保护机制**：
1. **IPCSkeleton** 获取调用者身份
   ```cpp
   // usb_service.cpp:71
   uint32_t tokenId = IPCSkeleton::GetCallingTokenID();
   ```

2. **Access Token** 验证
   - Token 权限检查
   - Token 有效期验证
   - Token 类型验证

3. **权限数据库** 查询
   - UsbRightManager 权限验证
   - 权限授予记录

**代码位置**：
- `services/native/src/usb_service.cpp:71`
- `services/native/src/usb_right_manager.cpp`

**局限性**：
- ✗ Token 劫持风险
- ✗ 权限检查竞态条件
- ✗ 数据库注入风险

#### HAL 边界保护

**保护机制**：
1. HDF 驱动框架隔离
2. 内核权限检查
3. SELinux 策略

**代码位置**：`drivers_interface_usb`

**局限性**：
- ✗ HAL 接口输入验证不足
- ✗ 内核漏洞直接影响

---

## 关键结论

### 高风险攻击面（优先关注）

1. **N-API 输入验证不足**
   - 所有 JS API 参数未充分验证
   - 缓冲区操作可能溢出
   - 优先级：**极高**

2. **IPC 权限检查竞态**
   - 权限检查和设备操作非原子
   - TOCTOU 漏洞可能
   - 优先级：**高**

3. **恶意 USB 设备**
   - 描述符解析可能溢出
   - BadUSB 攻击
   - 优先级：**极高**

4. **权限管理绕过**
   - UsbRightManager 权限检查逻辑复杂
   - 权限数据库可能被篡改
   - 优先级：**高**

### 中等风险攻击面

1. **资源管理不当**
   - 异步任务可能泄露
   - 文件描述符未正确关闭

2. **并发安全问题**
   - 多线程访问共享资源
   - 互斥锁使用不当

---

## 相关文档

- [安全风险评估](06_SecurityReview.md) - 详细安全漏洞分析
- [架构说明](02_Architecture.md) - 信任边界和架构设计
- [对外接口文档](04_Interface.md) - N-API 和 IPC 接口详情

---

**更新时间**：2026-02-07
**证据来源**：`interfaces/kits/js/napi/`, `interfaces/innerkits/`, `services/native/`
