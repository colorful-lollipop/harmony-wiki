# 项目概览

## 目的

本文档提供 OpenHarmony Hisilicon 板卡仓库的整体概览，帮助读者快速理解：
- 项目定位和边界
- 支持的板卡型号和特性
- 核心能力和运行环境
- 关键概念和术语

## 适用范围

本文档适用于：
- 新接触该仓库的开发者
- 需要了解板卡支持的系统工程师
- 选择开发板的方案评估人员

---

## 项目定位

### 仓库性质

**device_board_hisilicon** 是 OpenHarmony 的**板卡级（Board Level）** 仓库，包含基于上海海思（HiSilicon）芯片的多款开发板的板级配置代码。

**核心定位**:
- 硬件抽象层（HAL）的板级适配
- U-Boot 引导加载器配置
- LiteOS/Linux 内核的板级初始化
- 驱动 HAL（相机、音频、显示等）
- 安全启动机制

**不包含**:
- ❌ N-API 模块（应用层 API）
- ❌ IPC/ServiceAbility（系统服务）
- ❌ 应用层权限管理
- ❌ 业务功能实现（如相机应用）

### 与其他仓库的关系

```
┌─────────────────────────────────────────────────────────────┐
│                  OpenHarmony 生态系统                      │
├─────────────────────────────────────────────────────────────┤
│  应用层 (applications)                                      │
│  └── ❌ 本仓库不包含                                          │
├─────────────────────────────────────────────────────────────┤
│  框架层 (foundation)                                       │
│  └── N-API, IPC, 权限管理                                     │
├─────────────────────────────────────────────────────────────┤
│  子系统层 (subsystem)                                       │
│  └── 相机、音频、显示等子系统                                    │
├─────────────────────────────────────────────────────────────┤
│  SoC 层 (device_soc_hisilicon)                             │
│  └── 芯片级驱动、SDK                                          │
├─────────────────────────────────────────────────────────────┤
│  板卡层 (device_board_hisilicon) ← 本仓库                    │
│  └── ✅ 板级配置、U-Boot、HAL、安全启动                          │
├─────────────────────────────────────────────────────────────┤
│  Vendor 层 (vendor_hisilicon)                               │
│  └── 产品配置、HDF 配置                                        │
└─────────────────────────────────────────────────────────────┘
```

**证据**:
- README.md 第 5 行明确说明: "本仓用于存放基于上海海思芯片的开发板相关内容"
- 不存在 N-API、IPC 相关代码（参见 `wiki/_work/NOTES.md` Phase 1 搜索结果）

---

## 支持的板卡

### 板卡清单

| 开发板名称 | SoC 型号 | 系统类型 | 应用领域 | CPU 架构 | 关键特性 |
|-----------|----------|---------|---------|---------|---------|
| **HiSpark Aries** | Hi3518EV300 | 小型系统 (LiteOS-A) | 智慧视觉 | Cortex-A7 | Camera, U-Boot, Secure Boot |
| **HiSpark Pegasus** | Hi3861V100 | 轻量系统 (LiteOS-M) | 智慧 IoT | RISC-V | WiFi, 最小化配置 |
| **HiSpark Phoenix** | Hi3751V350 | 标准系统 (Linux) | 智慧媒体 | Cortex-A53 | TV, GPU, Video Decoder |
| **HiSpark Taurus** | Hi3516DV300 | 小型/标准系统 (LiteOS-A + Linux) | 智慧视觉 | Cortex-A7 | Camera, Audio, Display |

**证据**:
- README.md 第 7-12 行: 板卡信息表
- 各板卡 README_zh.md 文件确认

### 板卡详细介绍

#### HiSpark Aries (Hi3518EV300)

**系统类型**: 小型系统 (LiteOS-A)

**应用领域**: 智慧视觉

**关键特性**:
- 相机 HAL (Media Processing Platform - MPP)
- U-Boot 2020.01
- Secure Boot (RSA 2048/4096)
- SPI NOR Flash 存储

**目录结构**:
```
hispark_aries/
├── liteos_a/              # LiteOS-A 板级配置
│   ├── board/            # 板级初始化
│   └── drivers/          # LiteOS 驱动配置
└── uboot/               # U-Boot + Secure Boot
```

**证据**:
- `hispark_aries/README_zh.md:44` - "当前支持Hi3518EV300芯片"
- `hispark_aries/liteos_a/config.gni:15` - `kernel_type = "liteos_a"`

#### HiSpark Pegasus (Hi3861V100)

**系统类型**: 轻量系统 (LiteOS-M)

**应用领域**: 智慧 IoT

**关键特性**:
- 最小化配置（仅 liteos_m 目录）
- WiFi 连接
- 低功耗

**目录结构**:
```
hispark_pegasus/
└── liteos_m/            # 最小化配置
```

**证据**:
- `hispark_pegasus/README_zh.md:28` - "当前支持Hi3861V100芯片"

#### HiSpark Phoenix (Hi3751V350)

**系统类型**: 标准系统 (Linux)

**应用领域**: 智慧媒体（智能电视）

**关键特性**:
- 多核 ARM Cortex-A53 CPU
- Mali T450 GPU
- 视频解码器 (H.264, H.265, MPEG2, RMVB, AVS+)
- 音频解码和音效处理
- 支持 NTSC/PAL/SECAM 制式解调
- 支持 DVB-T/T2/S/S2 等全球数字 Demod

**目录结构**:
```
hispark_phoenix/
├── docs/                 # 文档资源
├── linux/
│   ├── boot/            # 预编译镜像
│   ├── system/cfg/      # 系统配置
│   └── updater/cfg/     # 升级配置
└── peripherals/bluetooth/rtkbt/  # 蓝牙适配器
```

**证据**:
- `hispark_phoenix/README_zh.md:21` - 详细硬件规格描述
- `hispark_phoenix/device.gni:15` - `soc_name = "hi3751v350"`

#### HiSpark Taurus (Hi3516DV300)

**系统类型**: 小型系统 + 标准系统 (LiteOS-A + Linux)

**应用领域**: 智慧视觉

**关键特性**:
- 相机 HAL（最复杂模块）
- 相机传感器适配（Sony IMX335, IMX600）
- 音频驱动 HAL（Hi3516 编解码器、TFA9879）
- 显示驱动
- U-Boot + Secure Boot

**目录结构**:
```
hispark_taurus/
├── linux/               # Linux 系统支持
├── liteos_a/           # LiteOS-A 支持
├── uboot/              # U-Boot + Secure Boot
├── camera/             # 相机 HAL（复杂）
├── audio_drivers/      # 音频驱动 HAL
└── display_drivers/    # 显示驱动
```

**证据**:
- `hispark_taurus/README_zh.md:52` - "当前支持Hi3516DV300芯片"
- `hispark_taurus/BUILD.gn:6-14` - 支持 Linux 和 LiteOS-A 两种内核
- `hispark_taurus/device.gni:28` - `is_support_mpi = true` (相机支持)

---

## 核心能力

### 1. 板级初始化

**功能**: 为 OpenHarmony 内核提供板级硬件初始化

**实现位置**:
- LiteOS-A: `liteos_a/board/board.c`, `target_config.h`
- LiteOS-M: 无（最简配置）
- Linux: 使用板级 HAL，无独立初始化文件

**关键任务**:
- CPU 和总线初始化
- 内存控制器配置
- 时钟和电源管理
- 外设初始化（UART, GPIO, SPI, I2C 等）

**证据**:
- `hispark_taurus/liteos_a/board/board.c:1` - 板级初始化实现
- `hispark_aries/liteos_a/board/` - 板级头文件和库

### 2. U-Boot 引导加载

**功能**: 提供 U-Boot 引导加载器和 OpenHarmony 安全启动支持

**支持板卡**: HiSpark Aries, HiSpark Taurus

**实现位置**: `uboot/` 目录

**关键特性**:
- U-Boot 2020.01 编译输出
- DDR 初始化
- Secure Boot（OpenHarmony 版本和 Release 版本）
- X.509 证书生成
- RSA 密钥管理（2048/4096 位）

**证据**:
- `hispark_aries/README_zh.md:48-53` - U-Boot 编译说明
- `hispark_aries/uboot/secureboot_ohos/x509_creater/` - X.509 证书生成
- `hispark_aries/uboot/secureboot_release/rsa2048pem/` - RSA 2048 密钥

### 3. 驱动 HAL (Hardware Abstraction Layer)

**功能**: 为 OpenHarmony 子系统提供硬件抽象接口

**支持板卡**: HiSpark Taurus（最完整）

#### 相机 HAL

**实现位置**: `hispark_taurus/camera/`

**核心组件**:
- `device_manager/` - 相机设备管理器
  - Sony IMX335 传感器驱动 (`imx335.cpp`)
  - Sony IMX600 传感器驱动 (`imx600.cpp`)
- `driver_adapter/` - 驱动适配层
  - MPI (Media Processing Interface) 适配器
  - VI (Video Input) 对象
  - VO (Video Output) 对象
  - VPSS (Video Process Sub-System) 对象
  - VENC (Video Encode) 对象
- `pipeline_core/` - 相机流水线核心
  - IPP (Image Post Processing) 算法示例

**证据**:
- `hispark_taurus/camera/device_manager/src/imx335.cpp:1` - IMX335 驱动
- `hispark_taurus/camera/driver_adapter/include/mpi_adapter.h:1` - MPI 适配器
- `hispark_taurus/device.gni:28` - `is_support_mpi = true`

#### 音频 HAL

**实现位置**: `hispark_taurus/audio_drivers/`

**核心组件**:
- `codec/` - 编解码器驱动
  - Hi3516 SoC 编解码器 (`hi3516_codec_*.c`)
  - TFA9879 编解码器 (`tfa9879_codec_*.c`)
- `dsp/` - DSP 驱动
- `soc/` - SoC 音频驱动

**证据**:
- `hispark_taurus/audio_drivers/codec/hi3516/src/hi3516_codec_impl.c:1` - Hi3516 编解码器实现
- `hispark_taurus/audio_drivers/codec/tfa9879/src/tfa9879_codec_ops.c:1` - TFA9879 操作实现

#### 显示 HAL

**实现位置**: `hispark_taurus/display_drivers/`

**状态**: 目录存在但内容较少（可能依赖 SoC 层实现）

### 4. 安全启动 (Secure Boot)

**功能**: 防止恶意固件加载，确保系统完整性

**支持板卡**: HiSpark Aries, HiSpark Taurus

**实现机制**:
- U-Boot 阶段验证内核镜像签名
- RSA 公钥/私钥对（2048 和 4096 位）
- X.509 证书管理

**证据**:
- `hispark_aries/uboot/secureboot_ohos/` - OpenHarmony 安全启动
- `hispark_aries/uboot/secureboot_release/rsa2048pem/` - RSA 2048 密钥
- `hispark_aries/uboot/secureboot_release/rsa4096pem/` - RSA 4096 密钥

---

## 运行环境

### 支持的操作系统

| 操作系统 | 板卡支持 | 配置文件 | 证据 |
|---------|---------|---------|------|
| LiteOS-M | HiSpark Pegasus | `hispark_pegasus/liteos_m/config.gni` | ✅ |
| LiteOS-A | HiSpark Aries, HiSpark Taurus | `hispark_aries/liteos_a/config.gni`<br>`hispark_taurus/liteos_a/config.gni` | ✅ |
| Linux | HiSpark Phoenix, HiSpark Taurus | `hispark_phoenix/linux/`<br>`hispark_taurus/linux/` | ✅ |

### CPU 架构

| 架构 | 板卡 | 配置 | 证据 |
|-----|------|------|-----|
| Cortex-A7 | HiSpark Aries, HiSpark Taurus (LiteOS-A) | `board_cpu = "cortex-a7"` | `hispark_aries/liteos_a/config.gni:21` |
| Cortex-A53 | HiSpark Phoenix | 标准系统，多核 | `hispark_phoenix/README_zh.md:21` |
| RISC-V | HiSpark Pegasus (LiteOS-M) | Hi3861V100 | `hispark_pegasus/README_zh.md:28` |

### 存储类型

| 存储 | 板卡 | 类型 | 证据 |
|-----|------|------|-----|
| SPI NOR | HiSpark Aries | `storage_type = "spinor"` | `hispark_aries/liteos_a/config.gni:61` |

---

## 关键概念

### 1. 板卡（Board）vs SoC（System on Chip）

**板卡（Board）**: 包含 SoC 芯片的外设开发板
- 本仓库定位
- 包含板级配置、外设驱动、启动加载器

**SoC（System on Chip）**: 芯片本身
- 仓库: `device_soc_hisilicon`
- 包含芯片级驱动、SDK、HAL 核心实现

**证据**:
- README.md 第 37-39 行 - 相关仓链接明确区分 Board 和 SoC

### 2. MPP (Media Processing Platform)

**定义**: HiSilicon 提供的媒体处理平台，提供相机、编解码、显示等硬件抽象接口

**用途**: 相机 HAL 的底层接口

**证据**:
- `hispark_aries/BUILD.gn:6` - `//device/soc/hisilicon/hi3518ev300/mpp:copy_mpp_libs`
- `hispark_taurus/device.gni:28` - `is_support_mpi = true`

### 3. MPI (Media Processing Interface)

**定义**: MPP 的 C 语言接口层

**用途**: 在相机 HAL 中调用 MPP 功能

**证据**:
- `hispark_taurus/camera/driver_adapter/include/mpi_adapter.h` - MPI 适配器头文件

### 4. HDF (Hardware Driver Foundation)

**定义**: OpenHarmony 硬件驱动框架

**用途**: 统一驱动接口，支持不同内核（LiteOS、Linux）

**证据**:
- `hispark_taurus/camera/BUILD.gn:4` - `import("//drivers/hdf_core/adapter/uhdf2/uhdf.gni")`

### 5. Secure Boot

**定义**: 安全启动机制，验证固件签名

**实现**: U-Boot 阶段验证，使用 RSA + X.509

**证据**:
- `hispark_aries/uboot/secureboot_*/` - 安全启动脚本和密钥

---

## 项目边界

### ✅ 本仓库负责

- 板级配置和初始化
- U-Boot 编译和配置
- 驱动 HAL 适配（相机、音频、显示）
- Secure Boot 配置
- GN 构建配置

### ❌ 本仓库不负责

- N-API 模块（应用层 API）
- IPC/ServiceAbility（系统服务）
- 应用层权限管理
- 业务功能实现
- SoC 芯片级驱动
- Vendor 产品配置

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 详细的目录组织和模块职责
- [架构说明](02_Architecture.md) - 系统架构和组件关系
- [GN Targets](04_GN_Targets.md) - 构建系统和目标依赖
- [安全评审](06_Security_Review.md) - Secure Boot 和安全风险分析
