# OpenHarmony Print Scan Framework - 内部架构与 Inner API

**目的**: 理解框架内部架构、模块接口、依赖关系和稳定性分级

**适用范围**: OpenHarmony Print Scan Framework 3.1 - 内部实现层

---

## 目录

- [架构分层](#架构分层)
- [模块依赖关系](#模块依赖关系)
- [打印服务内部架构](#打印服务内部架构)
- [扫描服务内部架构](#扫描服务内部架构)
- [SANE 服务内部架构](#sane-服务内部架构)
- [接口稳定性说明](#接口稳定性说明)
- [关键数据结构](#关键数据结构)

---

## 架构分层

```
┌─────────────────────────────────────────────────────────────┐
│                  应用层 (JS/ArkTS)                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ N-API Bindings
┌────────────────────┴────────────────────────────────────────┐
│              接口层 (interfaces/)                  │
│  - napi (print_napi, scan_napi)              │
│  - jsnapi (printextension, printextensionctx)    │
│  - ndk (ohprint, ohscan)                       │
└────────────────────┬────────────────────────────────────────┘
                     │ IPC (Binder)
                     ↓
┌────────────────────┴────────────────────────────────────────┐
│            内部实现层 (frameworks/innerkitsimpl/)       │
│  - print_impl (PrintServiceProxy)                   │
│  - scan_impl (ScanServiceProxy)                    │
│  - print_callback_stub, scan_callback_stub          │
│  - print_sync_load_callback, sane_service_load_callback │
└────────────────────┬────────────────────────────────────────┘
                     │ IPC (Binder)
                     ↓
┌────────────────────┴────────────────────────────────────────┐
│              服务层 (services/)                       │
│  - PrintServiceAbility (SA: PRINT_SERVICE_ID)        │
│  - ScanServiceAbility (SA: SCAN_SERVICE_ID)         │
│  - SaneServerManager (SA: SANE_SERVICE_ID)          │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────┴────────────────────────────────────────┐
│              后端层 (外部依赖)                    │
│  - CUPS (third_party_cups)                       │
│  - SANE Backends (third_party_backends)              │
│  - Vendor Drivers (框架内部)                         │
│  - USB/Network 协议栈                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 模块依赖关系

### 打印模块依赖图

```
print_napi (N-API 层)
    ↓ depends on
print_helper, print_models, print_client
    ↓
print_client (内部实现层)
    ↓ IPC calls
PrintServiceAbility (服务层)
    ↓ uses
print_cups_client, vendor_manager, print_system_data
    ↓
CUPS / Vendor Drivers (后端层)
```

### 扫描模块依赖图

```
scan_napi (N-API 层)
    ↓ depends on
scan_helper, scan_models, scan_client
    ↓
scan_client (内部实现层)
    ↓ IPC calls
ScanServiceAbility (服务层)
    ↓ uses
sane_manager_client, scan_usb_manager
    ↓
SaneServerManager (SANE 服务层)
    ↓
SANE Backends (外部依赖)
```

---

## 打印服务内部架构

### 核心组件

| 组件 | 文件位置 | 职责 | 稳定性 |
|--------|----------|------|--------|
| **PrintServiceAbility** | `services/print_service/include/print_service_ability.h` | 打印服务 SA 实现，管理所有打印功能 | 稳定 |
| **PrintServiceHelper** | `services/print_service/include/print_service_helper.h` | 辅助类，提供 Bundle 查询、权限检查等 | 稳定 |
| **PrintCUPSClient** | `services/print_service/include/print_cups_client.h` | CUPS 客户端，与 CUPS 打印系统交互 | 稳定 |
| **VendorManager** | `services/print_service/include/vendor_manager.h` | 厂商驱动管理器，管理多种打印驱动 | 稳定 |
| **PrintSystemData** | `services/print_service/include/print_system_data.h` | 打印系统数据管理，打印机列表、任务队列 | 稳定 |
| **PrintUserData** | `services/print_service/include/print_user_data.h` | 打印用户数据管理，用户偏好设置 | 稳定 |
| **PrintCallerAppMonitor** | `services/print_service/include/print_caller_app_monitor.h` | 调用应用监控，监控应用生命周期 | 稳定 |
| **PrintSecurityGuardManager** | `services/print_service/include/print_security_guard_manager.h` | 安全守卫管理器，与 Security Guard 集成 | 稳定 |

### 厂商驱动类型

| 驱动类型 | 文件 | 说明 |
|---------|------|------|
| **BSUNI Driver** | `vendor_bsuni_driver.cpp` | Brocadesoft 通用驱动 |
| **PPD Driver** | `vendor_ppd_driver.cpp` | PPD 文件驱动 |
| **IPP Everywhere Driver** | `vendor_ipp_everywhere.cpp` | IPP Everywhere 协议驱动 |
| **WLAN Group Driver** | `vendor_wlan_group.cpp` | WLAN 组打印机驱动 |
| **SMB Printer** | `smb_printer_discoverer.cpp` | SMB 网络打印机支持 |

### IPC 接口

| 接口 | Stub 位置 | Proxy 位置 | 说明 |
|--------|----------|------------|------|
| **IPrintService** | `print_service_stub.cpp` | `print_service_proxy.cpp` | 打印服务 IPC 接口 |
| **IPrintCallback** | `print_callback_stub.cpp` | `print_callback_proxy.cpp` | 打印回调 IPC 接口 |
| **IPrintExtensionCallback** | `print_extension_callback_stub.cpp` | - | 打印扩展回调 IPC 接口 |

---

## 扫描服务内部架构

### 核心组件

| 组件 | 文件位置 | 职责 | 稳定性 |
|--------|----------|------|--------|
| **ScanServiceAbility** | `services/scan_service/include/scan_service_ability.h` | 扫描服务 SA 实现，管理所有扫描功能 | 稳定 |
| **SaneManagerClient** | `services/scan_service/include/sane_manager_client.h` | SANE 服务客户端，加载和管理 SANE 服务 | 稳定 |
| **ScanUSBManager** | `services/scan_service/include/scan_usb_manager.h` | USB 扫描器管理，USB 设备发现和管理 | 稳定 |
| **ESCLDriverManager** | `services/scan_service/include/escl_driver_manager.h` | ESCL 驱动管理器，管理 ESCL 扫描器 | 稳定 |
| **ScanSystemData** | `services/scan_service/include/scan_system_data.h` | 扫描系统数据管理，扫描器列表、任务队列 | 稳定 |
| **CallerAppMonitor** | `services/scan_service/include/caller_app_monitor.h` | 调用应用监控，监控应用生命周期 | 稳定 |

### IPC 接口

| 接口 | Stub 位置 | Proxy 位置 | 说明 |
|--------|----------|------------|------|
| **IScanService** | `scan_service_stub.cpp` | `scan_service_proxy.cpp` | 扫描服务 IPC 接口 |
| **IScanCallback** | `scan_callback_stub.cpp` | `scan_callback_proxy.cpp` | 扫描回调 IPC 接口 |
| **ISaneBackends** | `services/sane_service/src/sane_service_ability.cpp` | `sane_manager_client.cpp` | SANE 后端服务 IPC 接口 |

### 扫描流程

```
发现扫描器:
ScanServiceAbility.GetScannerList()
  → ScanUSBManager.DiscoverUSB()
  → ESCLDriverManager.DiscoverNetwork()
  → 返回扫描器列表

执行扫描:
ScanServiceAbility.StartScan()
  → SaneManagerClient.OpenScanner()
  → SaneServerManager.SetOption()
  → SaneServerManager.StartScan()
  → 扫描进度回调
  → 返回扫描结果
```

---

## SANE 服务内部架构

### 核心组件

| 组件 | 文件位置 | 职责 | 稳定性 |
|--------|----------|------|--------|
| **SaneServerManager** | `services/sane_service/include/sane_service_ability.h` | SANE 后端 SA 实现，管理 SANE 后端 | 稳定 |
| **ScanPictureData** | `services/sane_service/include/scan_picture_data.h` | 扫描图片数据管理 | 稳定 |
| **ScanTask** | `services/scan_service/include/scan_task.h` | 扫描任务管理 | 稳定 |

### 服务关系

```
ScanServiceAbility (扫描服务)
    ↓ uses
SaneManagerClient (SANE 服务客户端)
    ↓ IPC calls
SaneServerManager (SANE 后端服务)
    ↓ loads
SANE Backends (外部扫描驱动)
```

---

## 接口稳定性说明

### 稳定接口（对外公开）

| 接口 | 声明位置 | 稳定性说明 |
|--------|----------|----------|
| **IPrintService** | `print_service_stub.h` | SA 接口，跨大版本稳定 |
| **IScanService** | `scan_service_stub.h` | SA 接口，跨大版本稳定 |
| **ISaneBackends** | `sane_service_ability.h:28` | SA 接口，跨大版本稳定 |
| **IPrintCallback** | `print_callback_stub.h` | 回调接口，跨大版本稳定 |
| **IScanCallback** | `scan_callback_stub.h` | 回调接口，跨大版本稳定 |

### 不稳定接口（内部实现）

| 接口 | 声明位置 | 稳定性说明 |
|--------|----------|----------|
| **PrintServiceHelper** | `print_service_helper.h` | 内部辅助类，可能重构 |
| **VendorDriverBase** | `vendor_driver_base.h` | 厂商驱动基类，接口可能变化 |
| **Helper 类** | 各 helper 目录 | 内部工具类，可能调整 |
| **Model 类** | models 目录 | 数据模型，可能随需求变化 |

### NDK 接口稳定性

| 接口 | 位置 | 稳定性说明 |
|--------|------|----------|
| **ohprint (NDK)** | `interfaces/kits/ndk/ohprint/include/` | 打印 NDK 接口，相对稳定 |
| **ohscan (NDK)** | `interfaces/kits/ndk/ohscan/include/` | 扫描 NDK 接口，相对稳定 |

### 可替换点

| 模块 | 可替换性 | 说明 |
|--------|---------|------|
| **Vendor Drivers** | 是 | 厂商驱动可以独立开发和替换 |
| **SANE Backends** | 是 | SANE 后端可以独立开发和替换 |
| **Print Extensions** | 是 | 打印扩展可以独立开发和替换 |
| **Helper Functions** | 否 | 内部辅助函数，不建议替换 |

---

## 关键数据结构

### 打印相关结构

| 数据结构 | 定义位置 | 说明 |
|-----------|----------|------|
| **PrinterInfo** | `print_models/include/` | 打印机信息（ID、名称、状态、能力） |
| **PrintJob** | `print_models/include/` | 打印任务（ID、文件列表、属性、状态） |
| **PrinterCapability** | `print_models/include/` | 打印机能力（支持的纸张大小、分辨率等） |
| **PrintAttributes** | `print_models/include/` | 打印属性（页面大小、颜色模式、双面模式） |
| **PrintExtensionInfo** | `print_models/include/` | 打印扩展信息（ID、名称、类型） |

### 扫描相关结构

| 数据结构 | 定义位置 | 说明 |
|-----------|----------|------|
| **ScanDeviceInfo** | `scan_models/include/` | 扫描设备信息（ID、名称、类型、状态） |
| **ScanParameters** | `scan_models/include/` | 扫描参数（分辨率、颜色模式、格式） |
| **ScanOptionDescriptor** | `scan_models/include/` | 扫描选项描述（名称、类型、范围、约束） |
| **ScanOptionValue** | `scan_models/include/` | 扫描选项值 |
| **ScanProgress** | `scan_models/include/` | 扫描进度（已完成页数、总页数） |

---

## 线程模型

### 打印服务线程模型

| 线程/队列 | 用途 |
|------------|------|
| **主线程** | SA 服务主循环，处理 IPC 调用 |
| **事件处理线程** | `PrintServiceHelper` 使用 EventHandler 处理异步事件 |
| **操作队列线程** | `OperationQueue` 处理异步操作 |
| **CUPS 工作线程** | CUPS 客户端的工作线程池 |
| **事件发布线程** | 发布打印机事件和任务状态变化 |

### 扫描服务线程模型

| 线程/队列 | 用途 |
|------------|------|
| **主线程** | SA 服务主循环，处理 IPC 调用 |
| **SANE 扫描线程** | SANE 后端扫描线程，执行实际扫描操作 |
| **事件处理线程** | `ScanServiceAbility` 使用 EventHandler 处理异步事件 |
| **设备发现线程** | USB 和网络设备发现线程 |

---

## 错误传播机制

### 错误码映射

```
N-API 层错误 (E_PRINT_*)
    ↓
内部实现错误
    ↓
服务层错误
    ↓
IPC 错误 (E_PRINT_RPC_FAILURE)
    ↓
应用层错误显示
```

### 异步回调错误处理

```
异步操作回调:
1. 检查错误码
2. 调用 napi_reject_internal 或 napi_resolve
3. 传递错误信息和错误码
4. 应用层通过 catch 或 Promise rejection 获取错误
```

---

**相关链接**:
- [项目概览](00_Overview.md)
- [对外 API](04_External_API.md)
- [目录结构](02_Directory_Structure.md)
- [GN Targets](06_GN_Targets.md)
