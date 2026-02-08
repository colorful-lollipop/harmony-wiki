# OpenHarmony USB Manager

## 项目定位

USB Manager 是 OpenHarmony 驱动子系统的核心组件，负责 USB 设备管理、权限控制与数据传输。

## 核心能力

| 能力 | 描述 | 支持模式 |
|------|------|---------|
| 设备枚举 | 扫描并列出可用 USB 设备 | Host |
| 权限管理 | USB 设备访问权限控制 | Host |
| 数据传输 | Bulk/Control/Interrupt/Isochronous 传输 | Host |
| 功能切换 | USB 功能模式切换 (MTP/ACM/RNDIS等) | Device |
| 端口管理 | USB-C 端口角色配置 | Both |
| 串口支持 | USB 串行通信 | Host |

## 运行环境

- **子系统**: usb
- **版本**: 3.1.0 (bundle.json)
- **License**: Apache-2.0
- **目标系统**: OpenHarmony Standard

## 关键概念

### 主机模式 (Host Mode)
设备作为 USB Host，连接外部 USB 设备进行通信。

### 设备模式 (Device Mode)
设备作为 USB Device，通过 USB 连接到主机。

### USB 权限
- 应用首次访问 USB 设备时需要请求权限
- 权限存储在 `usb_right_manager` 数据库
- 支持临时权限和永久权限

## 相关文档

- [架构与数据流](02_Architecture.md)
- [对外接口文档](04_Interface.md)
- [安全风险评估](06_SecurityReview.md)
