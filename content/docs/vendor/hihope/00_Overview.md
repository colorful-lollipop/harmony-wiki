# HiHope Vendor 仓库概览

## 文档信息

- **目的**：为新人提供 OpenHarmony vendor_hihope 仓库的整体认知
- **适用范围**：vendor/hihope 仓库全部内容
- **最后更新**：2025-02-06
- **关键结论**：本仓库为 OpenHarmony 产品配置和 HAL 适配仓库，不包含系统框架核心代码

## 仓库定位

### 核心职责

vendor_hihope 仓库是 **HiHope Open Source Organization** 维护的 **OpenHarmony Vendor Repository**，主要职责为：

1. **产品配置**：为 HiHope 系列硬件产品（海王星 Neptune / 大禹 DAYU）提供 OpenHarmony 系统配置
2. **HAL 适配**：实现硬件抽象层（Hardware Abstraction Layer）接口，连接 OpenHarmony 系统与厂商硬件
3. **HDF 配置**：配置硬件驱动框架（Hardware Driver Foundation）的服务绑定和设备参数
4. **示例与教程**：提供硬件外设使用示例（NearLink DK-3863）

### 仓库边界

**包含内容**：
- ✅ 产品配置文件（config.json, ohos.build）
- ✅ HAL 实现（hals/ 目录）
- ✅ HDF 配置文件（hdf_config/ 目录）
- ✅ 安全策略配置（security_config/）
- ✅ 构建文件（BUILD.gn, .gni）
- ✅ 示例代码（ws63_sample/）

**不包含内容**：
- ❌ N-API（JavaScript API）实现
- ❌ IPC/ServiceAbility 实际 C++ 实现
- ❌ 系统内核代码
- ❌ OpenHarmony 框架核心代码
- ❌ 第三方库实现

**原因**：这些核心代码位于 OpenHarmony 主源码仓库（如 foundation/, base/, drivers/ 目录），vendor 仓库仅负责厂商适配和配置。

## 产品线概览

### 产品分类

| 产品线 | 产品名称 | 类型 | 内核 | SoC/芯片 | 适用场景 | 配置文件 |
|---------|----------|------|------------|----------|----------|
| **2合1** | 2in1_core_system | Standard | RK3568 | 平板/笔记本混合设备 | config.json, ohos.build |
| **平板** | tablet_core_system | Standard | RK3568 | 平板设备 | config.json, ohos.build |
| **电视** | tv | Standard | RK3568 | 智能电视 | config.json, ohos.build |
| **可穿戴** | wearable | Standard | RK3568 | 可穿戴设备 | config.json, ohos.build |
| **IPC 摄像头** | ipcamera_core_system | Standard | RK3568 | IPC 摄像头 | config.json, ohos.build |
| **默认核心** | default_core_system | Standard | RK3568 | 默认核心系统参考 | config.json, ohos.build |
| **DAYU200** | rk3568 | Standard | RK3568 | DAYU200 开发板 | config.json, ohos.build |
| **DAYU210** | dayu210 | Standard | RK3568 | DAYU210 开发板 | config.json, ohos.build |
| **Mini 系统** | rk3568_mini_system | Standard | RK3568 | RK3568 精简系统 | config.json, ohos.build |
| **海王星 IoT** | neptune_iotlink_demo | Mini | LiteOS-M | WinnerMicro Neptune100 | BLE/IoT 演示 | config.json, ohos.build |
| **NearLink DK** | nearlink_dk_3863 | Mini | LiteOS-M | Hisilicon WS63 | NearLink 开发套件 | config.json, ohos.build |
| **NearLink 测试** | nearlink_dk_3863_xts | Mini | LiteOS-M | Hisilicon WS63 | NearLink XTS 测试 | config.json, ohos.build |

### 产品类型说明

#### Standard 系统（9 个）

**定义**：OpenHarmony 标准系统，提供完整系统能力。

**特征**：
- API Level 8 或 9
- 支持完整的图形界面（ArkUI）
- 支持完整的应用框架
- 支持 N-API（JavaScript API）
- 包含丰富的子系统（多媒体、通信、AI 等）

**包含产品**：
1. **rk3568** - Rockchip RK3568 最完整配置（最多传感器配置）
2. **dayu210** - DAYU210 开发板
3. **wearable** - 可穿戴设备配置
4. **tablet_core_system** - 平板设备配置
5. **tv** - 智能电视配置
6. **ipcamera_core_system** - IPC 摄像头配置
7. **default_core_system** - 默认核心系统参考
8. **2in1_core_system** - 2合1 混合设备配置
9. **rk3568_mini_system** - RK3568 精简配置（缺少部分功能区域）

#### Mini 系统（3 个）

**定义**：OpenHarmony 轻量级系统，针对资源受限设备。

**特征**：
- API Level 通常为较低版本
- 精简图形界面或无 GUI
- 精简的应用框架
- 不支持完整的 N-API
- 资源占用小

**包含产品**：
1. **neptune_iotlink_demo** - 基于 LiteOS-M 的 BLE/IoT 演示
2. **nearlink_dk_3863** - 基于 LiteOS-M 的 NearLink 开发套件，含 28 个示例教程
3. **nearlink_dk_3863_xts** - NearLink XTS 测试套件

## 核心能力

### Standard 系统核心能力

| 能力类别 | 支持情况 | 说明 |
|----------|---------|------|
| **图形界面** | ✅ 支持 | ArkUI 框架 |
| **应用框架** | ✅ 支持 | Ability/ExtensionAbility |
| **JavaScript API** | ✅ 支持 | N-API 绑定（但不在本仓库） |
| **多媒体** | ✅ 支持 | 相机、音频、视频、图像 |
| **通信** | ✅ 支持 | WiFi、蓝牙、以太网 |
| **传感器** | ✅ 支持 | 加速度、陀螺仪、光线、距离、磁场、温度、湿度、气体 |
| **输入输出** | ✅ 支持 | 触摸、按钮、按键、显示、振动、LED |
| **存储** | ✅ 支持 | eMMC、SD 卡 |
| **AI 能力** | ✅ 支持 | MindSpore（rk3568） |
| **安全** | ✅ 支持 | SELinux、Seccomp、权限管理、签名验证 |

### Mini 系统核心能力

| 能力类别 | 支持情况 | 说明 |
|----------|---------|------|
| **图形界面** | ⚠️ 精简 | 基础 UI 或无 GUI |
| **应用框架** | ⚠️ 精简 | Lite Ability |
| **JavaScript API** | ❌ 不支持 | 不支持完整 N-API |
| **通信** | ✅ 支持 | WiFi（部分）、BLE、UART、SLE（NearLink） |
| **传感器** | ✅ 支持 | GPIO、PWM、ADC、I2C、SPI |
| **存储** | ✅ 支持 | 闪存存储 |
| **系统服务** | ✅ 支持 | 精简系统服务（samgr_lite, hilog_lite, bootstrap_lite） |

## 运行环境

### Standard 系统运行环境

**硬件环境**：
- **CPU 架构**：ARM 64 位（target_cpu: "arm"）
- **SoC 平台**：Rockchip RK3568
- **内存**：通常 2GB - 8GB（取决于具体板卡）
- **存储**：eMMC 闪存 + 可选 SD 卡

**软件环境**：
- **OpenHarmony 版本**：3.0 - 4.0
- **API Level**：8 或 9
- **系统类型**：Standard
- **内核**：Linux 内核

**安全环境**：
- **SELinux**：启用（build_selinux: true）
- **Seccomp**：启用（build_seccomp: true）
- **权限系统**：完整权限模型
- **签名验证**：应用签名和包名绑定

### Mini 系统运行环境

**硬件环境**：
- **CPU 架构**：ARM Cortex-M4 或类似（不同 SoC）
- **内存**：通常 < 512KB（取决于具体板卡）
- **存储**：片上闪存 + 外部存储

**软件环境**：
- **OpenHarmony 版本**：1.0 - 3.0
- **API Level**：较低版本
- **系统类型**：Mini
- **内核**：LiteOS-M（轻量级实时操作系统）

**安全环境**：
- **权限系统**：精简权限模型
- **HUKS（密钥管理）**：精简版（huks_use_lite_storage = true）

## 关键概念

### 1. HAL（Hardware Abstraction Layer）

**定义**：硬件抽象层，为上层应用提供统一的硬件访问接口，屏蔽底层硬件差异。

**作用**：
- 连接 OpenHarmony 系统框架与厂商硬件
- 实现平台相关的硬件驱动接口
- 提供 C 语言接口给上层框架调用

**HAL 类型**（在 vendor 仓库中）：
- **Audio HAL**：音频编解码和音频流处理
- **Codec HAL**：音频编解码器接口
- **Utils HAL**：系统参数和令牌管理（仅在 IoT 产品中）

**证据**：
- 文件：`nearlink_dk_3863/hals/utils/token/hal_token.c:16`（OEMReadToken, OEMWriteToken 函数）
- 文件：`nearlink_dk_3863/hals/utils/sys_param/hal_sys_param.c:21`（HalGetSerial 函数）
- 目录：`neptune_iotlink_demo/hals/`, `nearlink_dk_3863/hals/`

### 2. HDF（Hardware Driver Foundation）

**定义**：硬件驱动框架，OpenHarmony 的统一驱动架构。

**特点**：
- **内核驱动（KHDF）**：运行在内核态，提供硬件访问
- **用户态驱动（UHDF）**：运行在用户态，提供框架服务
- **HCS 配置**：使用 HCS（HDF Configuration Source）语言描述驱动配置
- **HDI 接口**：统一的驱动接口定义语言

**配置层次**：
- **UHDF（用户态）**：`hdf_config/uhdf/` 目录下的 .hcs 文件
- **KHDF（内核态）**：`hdf_config/khdf/` 目录下的 .hcs 文件

**证据**：
- 文件：`rk3568/hdf_config/uhdf/device_info.hcs:27`（module = "rockchip,rk3568_chip"）
- 目录：`rk3568/hdf_config/`（包含 100+ 个 .hcs 配置文件）
- 文件：`2in1_core_system/hdf_config/uhdf/hdf.hcs`（HDF 配置包含示例）

### 3. SystemAbility

**定义**：系统级服务能力，提供跨进程的系统能力。

**作用**：
- 提供系统基础服务（软总线、分布式服务）
- 跨进程通信（通过 IPC/Binder）
- 系统能力注册和发现

**实现位置**：
- **不在 vendor 仓库**：实际实现在 OpenHarmony 框架中（foundation/ability/ability_runtime）
- **配置在 vendor 仓库**：SA 注册配置（.json 文件）

**证据**：
- 文件：`rk3568_mini_system/demo_foundation/foundation.json`（SA ID 8501: SoftBus 服务）
- 配置：`nearlink_dk_3863/config.json:34`（systemabilitymgr 子系统）
- 说明：vendor 仓库仅包含 SA 配置，不包含实际实现

### 4. N-API（Native API）

**定义**：OpenHarmony 提供的 JavaScript/ArkTS 原生 API 绑定机制。

**作用**：
- 允许 ArkTS 应用调用 C/C++ 原生代码
- 提供高性能系统接口
- 封装系统底层能力

**实现位置**：
- **不在 vendor 仓库**：实际实现在 OpenHarmony 框架中
- **vendor 仓库引用**：在 sanitizer_check_list.gni 中列出外部 N-API 模块

**证据**：
- 文件：`rk3568/security_config/sanitizer_check_list.gni`（列出了 ability_napi, camera_napi, audio_framework_napi 等）
- 说明：vendor 仓库使用但不实现 N-API

### 5. SELinux

**定义**：Security-Enhanced Linux，提供强制访问控制（MAC）机制。

**作用**：
- 进程间隔离
- 文件和设备访问控制
- 最小权限原则

**配置位置**：
- **HDF 配置**：在 .hcs 文件中配置 SELinux 上下文
- **系统镜像**：在 image_conf/ 中配置文件上下文

**证据**：
- 文件：`rk3568/config.json:12`（build_selinux: true）
- 文件：`rk3568/hdf_config/uhdf/device_info.hcs`（包含 "secon" 配置）
- 目录：`rk3568/image_conf/`（包含 system_image_conf.txt 等）

## 架构层次

### OpenHarmony 整体架构（本仓库部分）

```
┌─────────────────────────────────────────────────────────────┐
│                  应用层                              │
│              （ArkTS/C++ 应用）                    │
└────────────────────────┬────────────────────────────────┘
                     │
        ┌────────────────▼────────────────┐
        │   系统框架层         │  [不在 vendor 仓库]
        │  (Foundation/框架)       │
        │                         │
        └────────────────┬────────────────┘
                     │
        ┌────────────────▼────────────────┐
        │    HAL 层               │  [vendor 仓库核心]
        │  (hals/ 目录)           │
        │  - Audio HAL             │
        │  - Codec HAL             │
        │  - Utils HAL             │
        └────────────────┬────────────────┘
                     │
        ┌────────────────▼────────────────┐
        │   HDF 层                │  [vendor 仓库核心]
        │  (hdf_config/ .hcs)      │
        │  - UHDF (用户态驱动)      │
        │  - KHDF (内核态驱动)      │
        └────────────────┬────────────────┘
                     │
        ┌────────────────▼────────────────┐
        │    硬件层               │
        │  (厂商硬件)              │
        │  - SoC 芯片              │
        │  - 外设 (GPIO/I2C/SPI等)  │
        └────────────────────────────────┘
```

**说明**：
- 系统框架层位于 OpenHarmony 主源码仓库（不在此 vendor 仓库）
- HAL 层和 HDF 配置是 vendor 仓库的核心内容
- HAL 层通过标准接口连接上层框架和下层 HDF 驱动

## 产品选择指南

### 如何选择产品配置

| 使用场景 | 推荐产品 | 原因 |
|---------|----------|------|
| **RK3568 开发** | rk3568 | 最完整的传感器和外设配置，包含所有 HDF 配置 |
| **DAYU210 开发** | dayu210 | DAYU210 开发板专用配置 |
| **平板产品开发** | tablet_core_system | 平板设备特定配置 |
| **可穿戴设备开发** | wearable | 可穿戴设备优化配置 |
| **智能电视开发** | tv | 电视设备特定配置 |
| **IPC 摄像头开发** | ipcamera_core_system | IPC 摄像头优化配置 |
| **NearLink 学习开发** | nearlink_dk_3863 | 包含 28 个循序渐进教程 |
| **LiteOS-M 学习** | neptune_iotlink_demo | 轻量级 BLE/IoT 演示 |
| **系统移植参考** | default_core_system | 默认配置可作为参考模板 |
| **精简系统** | rk3568_mini_system | 去除部分功能区域的精简配置 |

## 下一步

阅读完本文档后，建议按以下顺序深入了解：

1. **[目录结构详解](02_Directory_Structure.md)** - 了解完整的目录组织
2. **[HAL 实现说明](03_HAL_Implementation.md)** - 学习硬件抽象层
3. **[HDF 配置详解](04_HDF_Configuration.md)** - 理解驱动框架配置
4. **[GN Targets 梳理](06_GN_Targets.md)** - 学习构建系统
5. **[安全风险评审](08_Security_Review.md)** - 了解安全机制

## 相关跳转

- [产品定位详解](01_Product_Positioning.md)
- [目录结构详解](02_Directory_Structure.md)
- [HAL 实现](03_HAL_Implementation.md)
- [HDF 配置](04_HDF_Configuration.md)
- [内部 API](05_Internal_API.md)
- [GN Targets](06_GN_Targets.md)
- [编译产物](07_Build_Artifacts.md)
- [安全评审](08_Security_Review.md)
- [常见问题](09_Common_Issues.md)
- [返回 Wiki 首页](SUMMARY.md)
