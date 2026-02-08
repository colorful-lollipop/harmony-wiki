# 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  @ohos.usb  │  │usbManager   │  │@ohos.usbManager.serial│
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
└─────────┼────────────────┼───────────────────┼─────────────┘
          │                │                   │
          ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                   N-API Layer                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  libusb.so  │  │libusbmanager│  │    libserial.so      │  │
│  │ (动态加载)   │  │   .z.so     │  │                     │  │
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
│  │              │  │               │ │                    │  │
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

## 模块职责

| 模块 | 路径 | 职责 |
|------|------|------|
| UsbService | services/native | SystemAbility 主服务 |
| UsbHostManager | services/native | 主机模式设备管理 |
| UsbDeviceManager | services/native | 设备模式功能管理 |
| UsbPortManager | services/native | USB-C 端口配置 |
| UsbRightManager | services/native | USB 权限数据库 |
| SerialManager | services/native | USB 串口管理 |
| UsbSrvClient | interfaces/innerkits | 客户端 IPC 代理 |
| usbmanager NAPI | interfaces/kits/js/napi | JS API 导出 |

## 数据流

### 设备枚举流程
```
App → N-API → UsbSrvClient → IPC → UsbService → HAL → 设备列表
```

### 权限请求流程
```
App → N-API → UsbService → UsbRightManager → (弹窗) → 权限授予
```

### 数据传输流程
```
App → N-API (bulkTransfer) → UsbService → HAL → USB 设备
```

## Feature Flags

| 标志 | 默认值 | 描述 |
|------|--------|------|
| usb_manager_feature_host | true | 主机模式 |
| usb_manager_feature_device | true | 设备模式 |
| usb_manager_feature_port | true | 端口管理 |
| usb_manager_pass_through | true | 直通模式 |

## 线程模型

- **主线程**: SystemAbility 事件循环
- **工作线程**: USB 传输操作
- **回调线程**: HDI 异步回调

## 相关文档

- [项目概览](01_Overview.md)
- [对外接口文档](04_Interface.md)
- [构建与产物](07_Build.md)
- [安全风险评估](06_SecurityReview.md)
