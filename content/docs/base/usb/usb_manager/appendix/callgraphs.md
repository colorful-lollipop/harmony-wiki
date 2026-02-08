# 关键调用链

## 设备枚举调用链

```
JS: usb.getDevices()
    │
    ▼
C++: CoreGetDevices (usb_info.cpp:...)
    │
    ▼
UsbSrvClient::GetDevices()
    │
    ▼
IPC: CheckSystemAbility(4201) → SendRequest(USB_FUN_GET_DEVICES)
    │
    ▼
UsbService::GetDevices()
    │
    ▼
UsbHostManager::GetDevices()
    │
    ▼
HDI: GetAllDevices()
    │
    ▼
返回: std::vector<UsbDevice>
```

## 权限请求调用链

```
JS: usb.requestRight(deviceName)
    │
    ▼
C++: CoreRequestRight (usb_info.cpp:...)
    │
    ▼
UsbSrvClient::RequestRight()
    │
    ▼
IPC: SendRequest(USB_FUN_REQUEST_RIGHT)
    │
    ▼
UsbService::RequestRight()
    │
    ▼
UsbRightManager::RequestRight()
    │
    ├──▶ 检查权限数据库
    ├──▶ 弹窗请求用户授权
    └──▶ 写入权限数据库
```

## 数据传输调用链

```
JS: usb.bulkTransfer(pipe, endpoint, data, timeout)
    │
    ▼
C++: PipeBulkTransfer (usb_info.cpp:...)
    │
    ▼
UsbSrvClient::BulkTransfer()
    │
    ▼
IPC: SendRequest(USB_FUN_BULK_TRANSFER)
    │
    ▼
UsbService::BulkTransferRead/Write()
    │
    ▼
UsbHostManager::BulkTransfer()
    │
    ▼
HDI: BulkTransfer()
    │
    ▼
返回: 传输数据
```

## 功能切换调用链

```
JS: usb.setCurrentFunctions(funcs)
    │
    ▼
C++: CoreSetCurrentFunctions (usb_info.cpp:...)
    │
    ▼
UsbSrvClient::SetCurrentFunctions()
    │
    ▼
IPC: SendRequest(USB_FUN_SET_CURRENT_FUNCTIONS)
    │
    ▼
UsbService::SetCurrentFunctions()
    │
    ▼
UsbDeviceManager::SetCurrentFunctions()
    │
    ├──▶ 验证函数组合
    ├──▶ UsbFunctionSwitchWindow::Show()
    └──▶ HDI: SetCurrentFunctions()
```

## 端口角色配置调用链

```
JS: usb.setPortRoles(portId, powerRole, dataRole)
    │
    ▼
C++: PortSetPortRole (usb_info.cpp:...)
    │
    ▼
UsbSrvClient::SetPortRole()
    │
    ▼
IPC: SendRequest(USB_FUN_SET_PORT_ROLE)
    │
    ▼
UsbService::SetPortRole()
    │
    ▼
UsbPortManager::SetPortRole()
    │
    ├──▶ 验证角色组合
    └──▶ HDI: SetPortRole()
```

## 串口打开调用链

```
JS: serial.open(portId)
    │
    ▼
C++: SerialOpenNapi (serial_info.cpp:...)
    │
    ▼
UsbSrvClient::SerialOpen()
    │
    ▼
IPC: SendRequest(USB_FUN_SERIAL_OPEN)
    │
    ▼
UsbService::SerialOpen()
    │
    ├──▶ 权限检查
    ├──▶ SerialManager::OpenSerial()
    └──▶ 注册 SerialDeathMonitor
```

## System Ability 生命周期

```
系统启动
    │
    ├──▶ SAFwk 加载 SA 4201
    │
    ├──▶ UsbService::OnStart()
    │       │
    │       ├──▶ InitUsbd() - 连接 HAL
    │       ├──▶ InitUsbRight() - 初始化权限
    │       └──▶ SubscribeSystemAbility() - 订阅 SA 变化
    │
    └──▶ UsbService 就绪
```

## 回调注册流程

```
应用注册传输回调
    │
    ▼
UsbSrvClient::RegBulkCallback()
    │
    ▼
IPC: SendRequest(USB_FUN_REG_BULK_CALLBACK)
    │
    ▼
UsbService::RegBulkCallback()
    │
    ├──▶ 创建 UsbdBulkCallBackImpl
    └──▶ HDI: RegisterBulkCallback()
            │
            └──▶ 数据传输完成 → OnBulkWriteCallback/OnBulkReadCallback
                    │
                    └──▶ IPC: SendRequest(CMD_USBD_BULK_CALLBACK_*)
                            │
                            └──▶ 应用回调执行
```
