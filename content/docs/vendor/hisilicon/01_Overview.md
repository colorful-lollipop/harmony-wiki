# 项目概述

## 1.1 项目定位

**vendor_hisilicon** 是 OpenHarmony 系统的海思芯片 vendor 仓库，负责存放基于上海海思芯片的各个开发板（产品形态）的配置信息、产品定义以及相关 Demo 案例。

### 核心职责

| 职责 | 说明 |
|-----|------|
| HDF 配置 | 硬件驱动框架（HDF）配置信息 |
| 产品定义 | 各开发板的产品配置（config.json） |
| 硬件适配 | 板级硬件抽象层（HAL）代码 |
| Demo 示例 | 演示程序与开发指南 |

### 仓库定位

```
openharmony/
├── device/                          # 设备层
│   ├── board/                       # 板级支持
│   │   └── hisilicon/              # 本仓库
│   │       ├── hispark_*/          # 各开发板
│   │       └── ...
│   └── soc/                         # SOC 支持
│       └── hisilicon/              # device_soc_hisilicon
└── vendor/                          # 厂商定制
    └── hisilicon/                  # 本仓库
        ├── hispark_*/              # 各产品配置
        └── ...
```

---

## 1.2 产品系列

本仓库支持以下海思芯片开发板：

### LiteOS-M 系列（IoT 设备）

| 开发板 | 芯片 | 系统 | 说明 |
|-------|------|------|------|
| hispark_pegasus | Hi3861V100 | LiteOS-M | WiFi IoT 开发板 |
| hispark_pegasus_mini_system | Hi3861V100 | LiteOS-M | 最小系统 |

### LiteOS 系列（轻量系统）

| 开发板 | 芯片 | 系统 | 说明 |
|-------|------|------|------|
| hispark_aries | Hi3516 | LiteOS | IP Camera 开发板 |
| hispark_taurus | Hi35xx | LiteOS | 标准设备 |
| hispark_taurus_mini_system | Hi35xx | LiteOS | 最小系统 |

### 标准系统系列

| 开发板 | 芯片 | 系统 | 说明 |
|-------|------|------|------|
| hispark_phoenix | Hi3518 | 标准系统 | IP Camera |
| hispark_taurus_standard | Hi35xx | 标准系统 | 标准系统 |
| watchos | - | 标准系统 | 手表产品 |

### Linux 系列

| 开发板 | 芯片 | 系统 | 说明 |
|-------|------|------|------|
| hispark_taurus_linux | Hi35xx | Linux | Linux 系统 |

---

## 1.3 相关仓库

### 设备板级支持

- **[device_board_hisilicon](https://gitee.com/openharmony/device_board_hisilicon)**：开发板公共代码

### SOC 支持

- **[device_soc_hisilicon](https://gitee.com/openharmony/device_soc_hisilicon)**：海思芯片 SOC 支持

### 系统组件

- **[third_party_u-boot](https://gitee.com/openharmony/third_party_u-boot)**：U-Boot 移植

---

## 1.4 系统类型说明

### LiteOS-M

**定位**：轻量级 IoT 设备操作系统

**特点**：
- 资源占用极小（RAM < 64KB）
- 无 MMU 支持
- 适用于 Hi3861V100 等 WiFi 芯片

### LiteOS

**定位**：轻量级设备操作系统

**特点**：
- 支持 MMU
- 适用于 Hi35xx 系列芯片
- 支持丰富的外设驱动

### 标准系统

**定位**：功能完整的设备操作系统

**特点**：
- 支持 ArkUI、分布式能力
- 完整的系统服务框架
- 适用于 Hi35xx 系列芯片

### Linux

**定位**：通用 Linux 系统

**特点**：
- 完整的 Linux 用户空间
- 支持容器化部署
- 适用于 Hi35xx 系列芯片

---

## 1.5 快速开始

### 环境准备

```bash
# 1. 获取 OpenHarmony 源码
git clone https://gitee.com/openharmony/manifest.git
cd manifest
repo init -u https://gitee.com/openharmony/manifest.git -b master
repo sync -c

# 2. 编译特定产品
python build.py --product-name wifiiot          # Pegasus
python build.py --product-name hispark_taurus  # Taurus
```

### 资料链接

- [海思开发板使用说明](https://gitee.com/openharmony/device_board_hisilicon)
- [Hi3861V100 开发指南](https://device.harmonyos.com/cn/docs/documentation/guide/ide-hi3861v100-compile-0000001192526021)
- [Hi3516 开发指南](https://device.harmonyos.com/cn/docs/documentation/guide/ide-hi3516v100-compile-0000001052144981)

---

## 1.6 术语表

| 术语 | 说明 |
|-----|------|
| HDF | Hardware Driver Framework，硬件驱动框架 |
| HAL | Hardware Abstraction Layer，硬件抽象层 |
| SOC | System on Chip，片上系统 |
| config.json | 产品配置文件 |
| .hcs | HDF Configuration Source，HDF 配置源文件 |

---

## 1.7 相关文档

### 内部链接

- [目录结构](./02_Directory_Structure.md)
- [产品系列](./03_Products.md)
- [配置体系](./04_Configuration.md)

### 外部链接

- [OpenHarmony 官方文档](https://www.harmonyos.com/)
- [海思开发者门户](https://www.hisilicon.com/cn/)
