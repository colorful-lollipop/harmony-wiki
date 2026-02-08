# CUPS 库概览

## 原始库信息

| 属性 | 值 |
|------|-----|
| **名称** | CUPS (Common UNIX Printing System) |
| **上游版本** | v2.4.14 |
| **上游地址** | https://github.com/OpenPrinting/cups/releases/download/v2.4.14/cups-2.4.14-source.tar.gz |
| **许可证** | Apache License 2.0 |
| **上游维护者** | OpenPrinting |

## 原始功能简介

CUPS 是 Unix/Linux 系统的标准打印系统，提供：

- **打印作业管理**：作业提交、队列管理、状态查询、取消操作
- **打印机抽象**：统一的打印机设备 URI 模型
- **IPP 协议支持**：Internet Printing Protocol 实现
- **PPD 驱动支持**：PostScript Printer Description 解析
- **打印过滤器**：图像格式转换、页面缩放、色彩处理
- **Web 管理界面**：CUPS Web 管理控制台

---

## CUPS 在 OpenHarmony 中的定位

### 所属子系统

```
thirdparty (第三方组件)
    │
    └──► cups - OpenHarmony 打印子系统的核心依赖
```

### 功能层级

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Applications)                   │
│         打印对话框、应用内打印功能、系统打印设置             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  打印框架层 (Print Framework)              │
│              base/print/print_fwk/                         │
│         打印作业管理、打印机发现、PPD 驱动管理               │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ cups_enable = true 时依赖
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     CUPS 层 (v2.4.14)                       │
│              third_party/cups/                              │
│       作业调度、IPP 协议、USB/网络后端、过滤器               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   OpenHarmony 系统服务                      │
│              hilog (日志), usb_manager (USB), ipc          │
└─────────────────────────────────────────────────────────────┘
```

### OH 中的核心功能

| 功能 | 支持状态 | 说明 |
|-----|---------|------|
| **IPP 打印** | ✅ 支持 | 网络打印机、IPP Everywhere、AirPrint |
| **USB 打印** | ✅ 支持 | OH USB 服务集成，取代 usblp |
| **PPD 驱动** | ✅ 支持 | 完整 PPD 解析和驱动管理 |
| **打印过滤器** | ✅ 支持 | cups-filters 图像转换 |
| **云打印** | ⚠️ 部分 | 基础支持，需进一步适配 |
| **蓝牙打印** | ❌ 未支持 | 需额外开发 |

---

## OH 特有适配概述

### 1. USB 打印支持 (核心适配)

```
┌─────────────────────────────────────────────────────────────┐
│                    OH USB 打印架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CUPS Backend (usb-oh.c)                                   │
│        │                                                     │
│        ├──► OH USB Manager (usb_manager.cxx)                │
│        │         │                                           │
│        │         └──► OHOS::USB::UsbSrvClient              │
│        │                   │                                 │
│        │                   └──► libusb (底层)               │
│                                                             │
│   替代方案：传统 Linux usblp 内核模块                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**相关 Patch**：
- `ohos-usb-print.patch` - USB 后端实现
- `ohos-usb-manager.patch` - OH USB 服务 C++ 封装

### 2. 日志系统集成

```
┌─────────────────────────────────────────────────────────────┐
│                    OH 日志集成架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CUPS 日志 (cupsdWriteErrorLog)                           │
│        │                                                     │
│        ├──► hilog-helper.c                                  │
│        │         │                                           │
│        │         └──► HiLogPrint(LOG_CORE, "cupslog")      │
│        │                                                     │
│   日志脱敏 (cups-log-datamasking.patch)                     │
│        ├──► 匿名化用户名                                    │
│        ├──► 脱敏作业名称                                     │
│        └──► 脱敏敏感数据                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**相关 Patch**：
- `ohos-hilog-print.patch` - HiLog 集成
- `cups-log-datamasking.patch` - 日志隐私保护

### 3. 网络适配

```
┌─────────────────────────────────────────────────────────────┐
│                   网络功能适配                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   IP 冲突处理                                               │
│   ├──► httpConnect3() - 支持网络接口绑定                    │
│   ├──► SO_BINDTODEVICE - 绑定到特定 NIC                     │
│   └──► nic= 参数 - 指定网络接口                             │
│                                                             │
│   IPP 认证                                                  │
│   ├──► 移除文件凭证存储                                     │
│   └──► 环境变量传递认证信息                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**相关 Patch**：
- `ohos_ip_conflict.patch` - IP 冲突和 NIC 绑定
- `ohos-ipp-authenticate.patch` - 认证适配

---

## OH Feature 开关

在 `base/print/print_fwk/print.gn` 中配置：

```gn
# CUPS 功能开关
cups_enable = true  # 启用 CUPS 支持 (默认 true)

# CUPS 内部功能 (cups.gni)
cups_feature_pstops_filter = false   # PS 过滤 (默认关闭)
cups_feature_virtual_printer = false # 虚拟打印机 (默认关闭)
enable_cups_lpd_backend = true       # LPD 后端 (默认开启)
enable_cups_socket_backend = true    # Socket 后端 (默认开启)
```

---

## OH 版本与上游版本对比

| 特性 | 上游 CUPS v2.4.14 | OpenHarmony CUPS |
|-----|-------------------|------------------|
| **基础打印** | ✅ | ✅ |
| **IPP 协议** | ✅ | ✅ |
| **USB 打印** | ❌ (依赖 usblp) | ✅ (OH USB 服务) |
| **日志系统** | syslog | HiLog + 数据脱敏 |
| **认证存储** | 文件 | 环境变量 |
| **网络绑定** | 基础 | 支持 NIC 指定 |
| **安全补丁** | 发布时版本 | 9 个 CVE 后向移植 |
| **OH 特有文件** | 无 | usb_manager.cxx, hilog-helper.c 等 |

---

## 相关组件

| 组件 | 关系 | 说明 |
|-----|------|------|
| **cups-filters** | 依赖 | 图像/PDF 过滤器库 |
| **print_fwk** | 消费者 | 打印框架主服务 |
| **hilog** | 依赖 | OH 日志系统 |
| **usb_manager** | 依赖 | OH USB 管理服务 |
| **openssl** | 依赖 | 加密库 |
| **libusb** | 依赖 | USB 访问库 |

---

## 下一步

- **[02_Patches.md](02_Patches.md)** - 深入了解 OH 适配细节
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看依赖关系
