# 目录结构

## 顶层目录概览

```
kernel/linux/config/
├── linux-4.19/                 # Linux 4.19.y 内核配置
│   └── arch/
│       └── arm/
│           └── configs/        # ARM 架构配置文件
├── linux-5.10/                 # Linux 5.10.y 内核配置
│   ├── arch/
│   │   └── arm/
│   │       └── configs/        # ARM 架构配置文件
│   └── type/                   # 系统形态配置目录
│       ├── small_defconfig     # 小系统配置模板
│       └── standard_defconfig  # 标准系统配置模板
├── linux-6.6/                  # Linux 6.6 内核配置
│   ├── base_defconfig          # 基础通用配置
│   ├── arch/                   # 架构相关配置
│   ├── type/                   # 系统形态配置
│   └── rk3568/                 # rk3568 芯片平台配置
├── hispark_taurus/             # Hi3516DV300 开发板配置
├── rk3568/                     # rk3568 开发板配置
├── qemu/                       # QEMU 模拟器配置
├── imx8mm/                     # i.MX8MMini 开发板配置
├── unionpi_tiger/             # UnionPi Tiger 开发板配置
├── yangfan/                    # 杨帆开发板配置
├── LICENSE                     # Apache-2.0 许可证
├── README.md                   # 英文项目说明
└── README_zh.md               # 中文项目说明
```

## 目录职责说明

本仓库的目录结构遵循清晰的职责划分原则，不同目录承担不同的配置职责。

### linux-*.* 版本目录

`linux-4.19/`、`linux-5.10/` 和 `linux-6.6/` 目录分别对应不同版本的 Linux 内核配置。早期版本（4.19 和 5.10）主要采用传统结构，在 `arch/arm/configs/` 目录下存放 ARM 架构的 defconfig 文件。新版本（6.6）则采用了新的分层结构，包含 `base_defconfig`、`type/`、`arch/` 等子目录，体现了 OpenHarmony 配置分层架构的演进。

需要注意的是，并非所有内核版本都包含完整的目录结构。例如，`linux-4.19/` 目前只包含 `arch/arm/configs/` 子目录，没有独立的 `type/` 目录，这是因为 OpenHarmony 对 4.19 版本的支持相对简化。

### 开发板/芯片配置目录

根目录下的 `hispark_taurus/`、`rk3568/`、`qemu/`、`imx8mm/`、`unionpi_tiger/` 和 `yangfan/` 目录分别对应不同开发板或芯片平台的配置。这些目录的存在表明 OpenHarmony 已经为该平台提供了官方或社区支持的配置模板。

每个开发板目录下通常包含 `arch/` 子目录，存放该平台的芯片架构配置。例如，`rk3568/arch/arm64_defconfig` 是 RK3568 芯片的 ARM64 位架构配置文件。

## 配置文件类型

### 通用配置模板

| 文件路径 | 类型 | 说明 |
|---------|------|------|
| `linux-5.10/arch/arm/configs/standard_common_defconfig` | 标准系统通用配置 | 适用于标准系统的默认配置 |
| `linux-5.10/arch/arm/configs/small_common_defconfig` | 小系统通用配置 | 适用于小系统的默认配置 |
| `linux-6.6/base_defconfig` | 基础通用配置 | OpenHarmony 特性依赖的必选配置 |
| `linux-6.6/type/small_defconfig` | 小系统形态配置 | 小系统的差异化配置 |
| `linux-6.6/type/standard_defconfig` | 标准系统形态配置 | 标准系统的差异化配置 |

### 开发板配置模板

| 目录 | 芯片/平台 | 说明 |
|------|----------|------|
| `hispark_taurus/` | Hi3516DV300 | 海思 Hi3516DV300 开发板 |
| `rk3568/` | RK3568 | 瑞芯微 RK3568 开发板 |
| `qemu/` | QEMU ARM | QEMU ARM 模拟器 |
| `imx8mm/` | i.MX8MMini | 恩智浦 i.MX8MMini 开发板 |
| `unionpi_tiger/` | UnionPi Tiger | UnionPi Tiger 开发板 |
| `yangfan/` | 杨帆 | 杨帆开发板 |

### 配置文件规模

以下是部分配置文件的代码行数统计（基于 defconfig 文件的实际内容）。

```
standard_common_defconfig: 81,852 bytes (约 2000+ 配置项)
small_common_defconfig:    79,628 bytes (约 1900+ 配置项)
hispark_taurus_standard_defconfig: 86,264 bytes
hispark_taurus_small_defconfig:    94,672 bytes
qemu-arm-linux_standard_defconfig: 176,971 bytes
```

从文件规模可以看出，标准系统配置包含约 2000 个左右的内核配置项，小系统配置略有精简。QEMU 的配置文件最大，这是因为需要支持更多的模拟硬件和调试功能。

## 目录演进历史

本仓库的目录结构经历了两次主要的演进。

第一次演进发生在 OpenHarmony 3.x 版本时期，引入了 `type/` 目录来区分不同系统形态的配置。在此之前，标准系统和小系统的配置混在一起，难以维护。分离后，配置管理变得更加清晰。

第二次演进发生在 OpenHarmony 4.x 版本时期，引入了 `base_defconfig` 作为基础通用配置层。这一层配置包含了 OpenHarmony 特性所必需的内核选项以及安全红线特性，确保所有平台都满足基本的系统要求和安全标准。

## 与代码证据的对应关系

本节描述的所有目录结构都可以在仓库中找到直接证据。顶层目录结构可以通过 `ls -la` 命令验证，文件内容可以通过 `cat` 命令读取。各配置文件的实际配置项数量可以通过统计 `CONFIG_` 前缀的行数来确认。
