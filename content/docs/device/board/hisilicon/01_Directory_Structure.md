# 目录结构与模块职责

## 目的

本文档详细说明 OpenHarmony Hisilicon 板卡仓库的目录结构、各模块的职责以及文件组织方式，帮助开发者快速定位代码。

## 适用范围

本文档适用于：
- 需要修改或扩展板级配置的开发者
- 驱动开发者定位 HAL 接口
- 系统工程师理解模块依赖关系

---

## 顶层目录结构

### 完整目录树（排除测试目录）

```
/Volumes/lexar/code/d/work/oh/device/board/hisilicon/
├── .git/                          # Git 版本控制
├── .gitattributes                 # Git 属性配置
├── LICENSE                       # Apache 2.0 许可证
├── OAT.xml                      # 开源审计工具配置
├── README.md                    # 中文说明（项目介绍）
├── README.en.md                 # 英文说明
│
├── hispark_aries/              # HiSpark Aries 开发板
│   ├── BUILD.gn               # GN 顶层构建配置
│   ├── README_zh.md           # 板卡说明（中文）
│   ├── NOTICE                 # 第三方声明
│   ├── ohos.build             # 老版构建配置
│   ├── liteos_a/             # LiteOS-A 系统配置
│   └── uboot/                # U-Boot 引导加载器
│
├── hispark_pegasus/            # HiSpark Pegasus 开发板
│   ├── README_zh.md           # 板卡说明
│   ├── ohos.build             # 老版构建配置
│   └── liteos_m/             # LiteOS-M 系统配置
│
├── hispark_phoenix/            # HiSpark Phoenix 开发板
│   ├── README_zh.md           # 板卡说明
│   ├── device.gni            # 板卡设备配置
│   ├── docs/                 # 文档资源
│   └── linux/                # Linux 系统配置
│
├── hispark_taurus/             # HiSpark Taurus 开发板（最复杂）
│   ├── BUILD.gn               # GN 顶层构建配置
│   ├── README_zh.md           # 板卡说明
│   ├── device.gni            # 板卡设备配置
│   ├── ohos.build             # 老版构建配置
│   ├── linux/                # Linux 系统配置
│   ├── liteos_a/             # LiteOS-A 系统配置
│   ├── uboot/                # U-Boot 引导加载器
│   ├── audio_drivers/        # 音频驱动 HAL
│   ├── camera/               # 相机驱动 HAL（复杂）
│   └── display_drivers/      # 显示驱动 HAL
│
└── wiki/                      # 文档目录
    ├── README.md              # Wiki 说明
    ├── SUMMARY.md             # 文档导航
    ├── 00_Overview.md        # 项目概览
    ├── 01_Directory_Structure.md  # 本文档
    ├── _work/                # 工作目录
    │   ├── NOTES.md          # 代码证据记录
    │   └── PLAN.md          # 工作计划
    └── appendix/             # 附录文档
```

**证据**:
- 仓库根目录 `ls -la` 输出
- 所有板卡目录的实际文件清单

---

## 板卡目录详细说明

### HiSpark Aries (Hi3518EV300 - 小型系统)

#### 目录结构

```
hispark_aries/
├── BUILD.gn                          # ✅ GN 顶层构建配置
├── README_zh.md                      # 板卡说明
├── NOTICE                           # 第三方代码声明
├── ohos.build                       # 老版构建配置
│
├── liteos_a/                        # LiteOS-A 系统支持
│   ├── BUILD.gn                     # ✅ GN 配置
│   ├── config.gni                   # ✅ 编译选项配置
│   │
│   ├── board/                      # 板级初始化代码
│   │   ├── board.c                 # ✅ 板级初始化实现
│   │   ├── target_config.h         # ✅ 目标配置
│   │   ├── Makefile               # 编译脚本
│   │   ├── LICENSE                # 许可证
│   │   │
│   │   ├── include/              # 板级头文件
│   │   │   ├── board.h          # ✅ 板级定义
│   │   │   ├── asm/            # 汇编相关
│   │   │   │   └── platform.h   # ✅ 平台定义
│   │   │   └── hisoc/          # HiSilicon SoC 头文件
│   │   │       ├── nand.h       # ✅ NAND 控制器
│   │   │       ├── mmc.h        # ✅ MMC/eMMC 控制器
│   │   │       ├── uart.h       # ✅ UART 驱动
│   │   │       ├── timer.h      # ✅ 定时器
│   │   │       ├── dmac.h       # ✅ DMA 控制器
│   │   │       ├── mmu_config.h # ✅ MMU 配置
│   │   │       ├── cpu.h        # ✅ CPU 相关
│   │   │       ├── flash.h      # ✅ Flash 控制器
│   │   │       ├── clock.h      # ✅ 时钟控制
│   │   │       └── random.h    # ✅ 随机数生成
│   │   │
│   │   └── libs/              # 板级库
│   │       ├── debug/          # Debug 版本库
│   │       └── release/       # Release 版本库
│   │
│   └── drivers/                 # LiteOS 驱动配置
│       └── BUILD.gn           # ✅ GN 配置
│
└── uboot/                         # U-Boot 引导加载器
    ├── Makefile                    # ✅ U-Boot 编译脚本
    ├── out/                       # 编译输出
    │   └── boot/                  # ✅ U-Boot 镜像输出
    │
    ├── prebuilts/                # U-Boot 编译依赖
    │
    ├── reg/                      # U-Boot 配置和 LICENSE
    │
    ├── secureboot_ohos/          # ✅ OpenHarmony 安全启动
    │   └── x509_creater/       # ✅ X.509 证书生成工具
    │
    └── secureboot_release/       # ✅ Release 版本安全启动
        ├── ddr_init/           # ✅ DDR 初始化二进制
        │   ├── boot/
        │   │   └── hi3518ev300/
        │   ├── drv/
        │   │   └── cmd_bin/
        │   └── include/
        ├── rsa2048pem/         # ✅ RSA 2048 位密钥
        └── rsa4096pem/         # ✅ RSA 4096 位密钥
```

#### 模块职责

| 模块 | 职责 | 关键文件 |
|------|------|---------|
| **board** | 板级硬件初始化、配置参数 | `board.c`, `target_config.h`, `include/` |
| **drivers** | LiteOS 内核驱动配置 | `drivers/BUILD.gn` |
| **uboot** | U-Boot 编译、安全启动 | `Makefile`, `secureboot_*/` |
| **secureboot_ohos** | OpenHarmony 安全启动脚本 | `x509_creater/` |
| **secureboot_release** | Release 版本安全启动 | `ddr_init/`, `rsa*pem/` |

**证据**:
- `hispark_aries/README_zh.md:22-32` - 目录结构说明
- `hispark_aries/liteos_a/config.gni:15` - `kernel_type = "liteos_a"`
- `hispark_aries/uboot/secureboot_*/` - 安全启动目录

---

### HiSpark Pegasus (Hi3861V100 - 轻量系统)

#### 目录结构

```
hispark_pegasus/
├── README_zh.md                      # 板卡说明
├── ohos.build                       # 老版构建配置
│
└── liteos_m/                        # LiteOS-M 系统支持（最简）
    └── config.gni                   # ✅ 编译选项配置
```

#### 模块职责

| 模块 | 职责 | 关键文件 |
|------|------|---------|
| **liteos_m** | LiteOS-M 最小化配置 | `config.gni` |

**特点**: 最简配置，无板级初始化代码，依赖 SoC 层提供。

**证据**:
- `hispark_pegasus/README_zh.md` - 仅包含基本说明
- `hispark_pegasus/liteos_m/config.gni` - 唯一的配置文件

---

### HiSpark Phoenix (Hi3751V350 - 标准系统)

#### 目录结构

```
hispark_phoenix/
├── README_zh.md                      # 板卡说明（详细）
├── device.gni                       # ✅ 板卡设备配置
│
├── docs/                            # 文档资源
│   ├── figures/                     # 图片资源
│   │   └── zn-cn_image_Hi3751V350.png  # ✅ 板卡外观图
│   └── public_sys-resources/         # 公共系统资源
│
├── linux/                           # Linux 系统支持
│   ├── BUILD.gn                     # ✅ GN 顶层配置
│   ├── ohos.build                   # 老版构建配置
│   │
│   ├── boot/                        # 预编译镜像分区文件
│   │
│   ├── system/                      # 系统资源配置
│   │   └── cfg/                    # ✅ 系统配置文件
│   │
│   └── updater/                     # 升级子系统配置
│       └── cfg/                    # ✅ 升级配置文件
│
└── peripherals/                      # 外设适配（文档提及）
    └── bluetooth/                  # 蓝牙适配器
        └── rtkbt/                 # Realtek 蓝牙栈
```

#### 模块职责

| 模块 | 职责 | 关键文件 |
|------|------|---------|
| **device.gni** | 定义 SoC 名称、相机支持 | `device.gni` |
| **docs** | 板卡文档、图片资源 | `figures/`, `public_sys-resources/` |
| **linux** | Linux 系统镜像、配置、升级 | `boot/`, `system/cfg/`, `updater/cfg/` |
| **peripherals** | 外设适配（蓝牙等） | `bluetooth/rtkbt/` |

**证据**:
- `hispark_phoenix/README_zh.md:29-43` - 目录结构说明
- `hispark_phoenix/device.gni:14-15` - `soc_company = "hisilicon"`, `soc_name = "hi3751v350"`

---

### HiSpark Taurus (Hi3516DV300 - 小型/标准系统)

#### 目录结构（最复杂）

```
hispark_taurus/
├── BUILD.gn                          # ✅ GN 顶层构建配置（支持双内核）
├── README_zh.md                      # 板卡说明
├── device.gni                       # ✅ 板卡设备配置
│
├── linux/                            # Linux 系统支持
│   ├── BUILD.gn                     # ✅ GN 顶层配置
│   ├── config.gni                   # ✅ 编译选项配置
│   ├── ohos.build                   # 老版构建配置
│   │
│   ├── distributedhardware/          # ✅ 分布式硬件配置
│   │   └── BUILD.gn
│   │
│   ├── images/                     # 镜像文件
│   │   └── BUILD.gn              # ✅ GN 配置
│   │
│   ├── system/                     # 系统配置
│   │   ├── BUILD.gn             # ✅ GN 配置
│   │   └── cfg/                 # 系统配置文件
│   │
│   └── updater/                    # 升级配置
│       ├── BUILD.gn             # ✅ GN 配置
│       └── cfg/                 # ✅ 升级配置文件
│
├── liteos_a/                        # LiteOS-A 系统支持
│   ├── BUILD.gn                     # ✅ GN 配置
│   ├── config.gni                   # ✅ 编译选项配置
│   │
│   ├── board/                      # 板级初始化
│   │   ├── board.c                # ✅ 板级初始化实现
│   │   ├── target_config.h        # ✅ 目标配置
│   │   ├── Makefile              # 编译脚本
│   │   ├── LICENSE               # 许可证
│   │   │
│   │   ├── include/             # 板级头文件
│   │   │   ├── board.h         # ✅ 板级定义
│   │   │   ├── asm/           # 汇编相关
│   │   │   │   ├── dma.h      # ✅ DMA 控制器
│   │   │   │   └── platform.h # ✅ 平台定义
│   │   │   ├── system_config.h  # ✅ 系统配置
│   │   │   ├── platform_config.h # ✅ 平台配置
│   │   │   ├── reset_shell.h  # ✅ Reset Shell
│   │   │   └── hisoc/        # ✅ SoC 头文件
│   │   │       ├── nand.h     # NAND 控制器
│   │   │       ├── sys_ctrl.h # 系统控制器
│   │   │       ├── usb3.h    # USB 3.0
│   │   │       ├── net.h      # 网络控制器
│   │   │       ├── spinand.h  # SPI NAND
│   │   │       ├── mmc.h      # MMC/eMMC
│   │   │       ├── uart.h     # UART
│   │   │       ├── spinor.h   # SPI NOR
│   │   │       ├── timer.h    # 定时器
│   │   │       ├── dmac.h     # DMA 控制器
│   │   │       ├── mmu_config.h # MMU 配置
│   │   │       ├── cpu.h      # CPU 相关
│   │   │       ├── flash.h    # Flash 控制器
│   │   │       ├── clock.h    # 时钟控制
│   │   │       └── random.h  # 随机数生成
│   │   │
│   │   └── libs/              # 板级库
│   │       ├── debug/          # Debug 版本
│   │       └── release/       # Release 版本
│   │
│   └── drivers/                  # LiteOS 驱动配置
│       └── BUILD.gn           # ✅ GN 配置
│
├── uboot/                           # U-Boot 引导加载器
│   ├── Makefile                      # ✅ 编译脚本
│   ├── out/                         # 编译输出
│   │   └── boot/                   # ✅ U-Boot 镜像
│   ├── prebuilts/                  # U-Boot 编译依赖
│   ├── reg/                        # U-Boot 配置
│   ├── secureboot_ohos/             # ✅ OpenHarmony 安全启动
│   │   └── x509_creater/        # ✅ X.509 证书生成
│   ├── secureboot_release/          # ✅ Release 安全启动
│   │   ├── ddr_init/            # ✅ DDR 初始化
│   │   │   ├── boot/hi3518ev300/
│   │   │   ├── drv/cmd_bin/
│   │   │   └── include/
│   │   ├── rsa2048pem/          # ✅ RSA 2048 密钥
│   │   └── rsa4096pem/          # ✅ RSA 4096 密钥
│   └── prebuilts/                # 预编译工具
│       └── mkimage              # ✅ U-Boot 镜像生成工具
│
├── audio_drivers/                  # ✅ 音频驱动 HAL（复杂）
│   ├── BUILD.gn                 # ✅ GN 配置
│   │
│   ├── codec/                    # 编解码器驱动
│   │   ├── hi3516/            # ✅ Hi3516 SoC 编解码器
│   │   │   ├── include/
│   │   │   │   └── hi3516_codec.h
│   │   │   └── src/
│   │   │       ├── hi3516_codec_adapter.c  # ✅ 适配器
│   │   │       ├── hi3516_codec_impl.c      # ✅ 实现
│   │   │       └── hi3516_codec_ops.c      # ✅ 操作
│   │   │
│   │   └── tfa9879/           # ✅ TFA9879 编解码器
│   │       ├── include/
│   │       └── src/
│   │           ├── tfa9879_codec_adapter.c  # ✅ 适配器
│   │           └── tfa9879_codec_ops.c      # ✅ 操作
│   │
│   ├── dsp/                      # DSP 驱动
│   │   ├── include/
│   │   └── src/
│   │
│   └── soc/                      # SoC 音频驱动
│       ├── include/
│       └── src/
│
├── camera/                         # ✅ 相机驱动 HAL（最复杂模块）
│   ├── BUILD.gn                 # ✅ GN 顶层配置
│   │
│   ├── demo/                    # 相机演示代码
│   │   └── include/
│   │       └── project_camera_demo.h
│   │
│   ├── device_manager/           # ✅ 相机设备管理器
│   │   ├── BUILD.gn           # ✅ GN 配置
│   │   ├── include/
│   │   │   ├── imx335.h      # ✅ Sony IMX335 传感器
│   │   │   ├── imx600.h      # ✅ Sony IMX600 传感器
│   │   │   └── project_hardware.h
│   │   └── src/
│   │       ├── imx335.cpp     # ✅ IMX335 驱动实现
│   │       └── imx600.cpp     # ✅ IMX600 驱动实现
│   │
│   ├── driver_adapter/          # ✅ 驱动适配层
│   │   ├── BUILD.gn          # ✅ GN 配置
│   │   ├── include/
│   │   │   ├── ivpss_object.h      # ✅ VPSS 对象
│   │   │   ├── ivo_object.h        # ✅ VO 对象
│   │   │   ├── isys_object.h       # ✅ 输入系统
│   │   │   ├── ivi_object.h        # ✅ VI 对象
│   │   │   ├── mpi_adapter.h       # ✅ MPI 适配器
│   │   │   └── ivenc_object.h      # ✅ 编码对象
│   │   └── src/
│   │
│   ├── libs/                    # 相机库
│   │   ├── linux/
│   │   └── liteos_a/
│   │
│   └── pipeline_core/            # ✅ 相机流水线核心
│       ├── BUILD.gn            # ✅ GN 配置
│       ├── src/
│       │   └── ipp_algo_example.c  # ✅ IPP 算法示例
│       └── pipeline_impl/src/strategy/config/
│           ├── config.c        # 配置文件（编译生成）
│           └── params.c        # 参数文件（编译生成）
│
└── display_drivers/                # 显示驱动 HAL
    └── BUILD.gn                 # ✅ GN 配置
```

#### 模块职责

| 模块 | 职责 | 关键文件 | 复杂度 |
|------|------|---------|--------|
| **linux** | Linux 系统镜像、配置、分布式硬件、升级 | `linux/` | 中 |
| **liteos_a/board** | LiteOS-A 板级初始化、硬件抽象 | `board/` | 中 |
| **uboot** | U-Boot 编译、Secure Boot | `uboot/` | 中 |
| **camera** | 相机 HAL（最复杂，包含设备管理、适配层、流水线） | `camera/` | ⭐⭐⭐ |
| **audio_drivers** | 音频 HAL（编解码器、DSP、SoC 驱动） | `audio_drivers/` | ⭐⭐ |
| **display_drivers** | 显示 HAL | `display_drivers/` | ⭐ |

**证据**:
- `hispark_taurus/README_zh.md:22-36` - 目录结构说明
- `hispark_taurus/device.gni:14-37` - 板卡设备配置（包含相机支持）
- `hispark_taurus/camera/` 目录结构和文件清单

---

## 模块职责总结

### 按功能分类

#### 1. 板级初始化
**目录**: `liteos_a/board/`

**职责**:
- 板级硬件初始化
- CPU 和总线配置
- 外设控制器初始化
- 内存和时钟配置

**关键文件**:
- `board.c` - 初始化代码
- `target_config.h` - 目标配置
- `include/hisoc/*.h` - SoC 硬件抽象

**支持板卡**: HiSpark Aries, HiSpark Taurus

#### 2. U-Boot + Secure Boot
**目录**: `uboot/`

**职责**:
- U-Boot 2020.01 编译
- DDR 初始化
- Secure Boot 验证
- X.509 证书生成
- RSA 密钥管理

**关键文件**:
- `Makefile` - 编译脚本
- `secureboot_ohos/x509_creater/` - 证书生成
- `secureboot_release/ddr_init/` - DDR 初始化
- `secureboot_release/rsa*pem/` - RSA 密钥

**支持板卡**: HiSpark Aries, HiSpark Taurus

#### 3. 相机 HAL
**目录**: `camera/`

**职责**:
- 相机传感器驱动（IMX335, IMX600）
- MPI (Media Processing Interface) 适配
- 相机流水线管理
- IPP (Image Post Processing) 算法
- HDF (Hardware Driver Foundation) 适配

**关键组件**:
- `device_manager/` - 传感器驱动
- `driver_adapter/` - MPI 适配层
- `pipeline_core/` - 流水线核心

**支持板卡**: HiSpark Taurus（主要）, HiSpark Aries（基础 MPP 库）

#### 4. 音频 HAL
**目录**: `audio_drivers/`

**职责**:
- 音频编解码器驱动
- DSP 驱动
- SoC 音频控制器驱动

**关键组件**:
- `codec/` - 编解码器（Hi3516, TFA9879）
- `dsp/` - DSP 驱动
- `soc/` - SoC 音频驱动

**支持板卡**: HiSpark Taurus

#### 5. 显示 HAL
**目录**: `display_drivers/`

**职责**:
- 显示控制器驱动
- HDMI/LVDS 输出

**支持板卡**: HiSpark Taurus（目录存在，内容较少）

#### 6. 系统镜像与配置
**目录**: `linux/`

**职责**:
- Linux 系统镜像分区
- 系统资源配置
- 升级子系统配置
- 分布式硬件配置

**支持板卡**: HiSpark Phoenix, HiSpark Taurus

---

## 文件类型说明

### GN 构建文件

| 文件类型 | 用途 | 示例 |
|---------|------|------|
| **BUILD.gn** | GN 构建目标定义 | `hispark_aries/BUILD.gn` |
| **.gni** | GN 配置变量（可被 import） | `hispark_taurus/device.gni`, `config.gni` |
| **ohos.build** | 老版构建配置（向后兼容） | `hispark_aries/ohos.build` |

### 配置文件

| 文件类型 | 用途 | 示例 |
|---------|------|------|
| **config.gni** | 内核编译选项（工具链、编译标志） | `hispark_aries/liteos_a/config.gni` |
| **device.gni** | 板卡设备配置（SoC 名称、相机支持） | `hispark_taurus/device.gni` |
| **.hcs** | HDF 配置文件（被 hc_gen 处理） | （在 vendor 层，不在此仓库） |

### 源代码文件

| 文件类型 | 用途 | 示例 |
|---------|------|------|
| **board.c** | 板级初始化实现 | `hispark_taurus/liteos_a/board/board.c` |
| **target_config.h** | 目标配置宏 | `hispark_taurus/liteos_a/board/target_config.h` |
| **imx335.cpp** | 相机传感器驱动 | `hispark_taurus/camera/device_manager/src/imx335.cpp` |
| **hi3516_codec_*.c** | 音频编解码器驱动 | `hispark_taurus/audio_drivers/codec/hi3516/src/` |

---

## 依赖关系

### 板卡 → SoC 依赖

```
板卡层
├── board.c
│   └── include/hisoc/*.h          // 引用 SoC 层头文件
│
├── camera/device_manager/
│   └── imx335.cpp              // 使用 MPP 接口
│
└── audio_drivers/codec/
    └── hi3516_codec_impl.c      // 使用 HAL 接口
    │
    └──依赖→ SoC 层 (device_soc_hisilicon)
        ├── mpp/
        ├── audio/
        └── display/
```

**证据**:
- `hispark_taurus/BUILD.gn:10-11` - `deps += ["//device/soc/hisilicon/hi3516dv300/sdk_linux:hispark_taurus_sdk"]`
- `hispark_taurus/device.gni:17` - `import("//device/soc/${soc_company}/${soc_name}/soc.gni")`

### 板卡 → Vendor 依赖

```
板卡层
└── camera/pipeline_core/
    └── BUILD.gn
        ├── product_config_path = "//vendor/hisilicon/hispark_taurus_standard"
        └── 依赖→ HDF 配置 (vendor 层)
            ├── hdf_config/uhdf/camera/*.hcs
            └── camera_host_config.hcs
```

**证据**:
- `hispark_taurus/camera/BUILD.gn:41-43` - `$product_config_path/hdf_config/uhdf/camera/...`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和板卡介绍
- [架构说明](02_Architecture.md) - 系统架构和组件关系
- [内部 API](03_Inner_API.md) - 模块接口和依赖方向
- [GN Targets](04_GN_Targets.md) - 构建系统和目标依赖
