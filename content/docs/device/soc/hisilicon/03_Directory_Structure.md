# 目录结构

本文档描述 `device_soc_hisilicon` 仓库的目录结构和模块组织。

## 顶层目录

```
/device/soc/hisilicon
├── common/              # 通用 HAL 和平台驱动（跨芯片复用）
├── hi3516dv300/        # Hi3516DV300 芯片专用代码
├── hi3518ev300/        # Hi3518EV300 芯片专用代码
├── hi3751v350/         # Hi3751V350 芯片专用代码
├── hi3861v100/         # Hi3861V100 芯片专用代码
├── ws63v100/           # WS63V100 芯片专用代码
├── ohos.build          # OpenHarmony 子系统构建定义
├── README.md           # 项目说明
└── LICENSE             # 许可证
```

## common 目录结构

**路径**: `common/`

### hal/ - 硬件抽象层

**用途**: 提供 OpenHarmony HDI (Hardware Driver Interface) 实现

```
common/hal/
├── BUILD.gn           # HAL 顶层构建配置
├── LICENSE            # 许可证
├── README.md          # 说明文档
├── ai/                # AI 推理 HDI
├── display/           # 显示 HDI
├── media/             # 媒体 HDI (音频/相机/编解码)
├── middleware/        # 中间件 (FFmpeg 适配)
├── multimedia/        # 多媒体库
├── update/            # OTA 升级服务
└── usb/               # USB 驱动
```

### platform/ - 平台驱动

**用途**: 提供芯片无关的外设驱动实现

```
common/platform/
├── BUILD.gn           # 平台驱动顶层构建
├── Kconfig            # 内核配置
├── lite.mk            # LiteOS Makefile
├── adc/               # ADC 驱动
├── dmac/              # DMA 控制器驱动
├── gpio/              # GPIO 驱动
├── hieth-sf/          # 以太网驱动
├── hisi_sdk/          # 海思 SDK 适配
├── i2c/               # I2C 驱动
├── i2s/               # I2S 音频驱动
├── libs/              # 静态库
├── mipi_csi/          # MIPI CSI 驱动
├── mipi_dsi/          # MIPI DSI 驱动
├── mmc/               # MMC/SD 驱动
├── mtd/               # MTD 存储驱动
├── pin/               # 引脚复用配置
├── pwm/               # PWM 驱动
├── rtc/               # RTC 驱动
├── spi/               # SPI 驱动
├── timer/             # 定时器驱动
├── uart/              # UART 驱动
├── watchdog/          # 看门狗驱动
└── wifi/              # WiFi 驱动
```

## hi3861v100 目录结构

**路径**: `hi3861v100/`

```
hi3861v100/
├── BUILD.gn                  # 根构建配置
├── hi3861_adapter/           # OpenHarmony 适配层
│   ├── hals/                # HAL 实现
│   │   ├── update/          # OTA 升级
│   │   ├── utils/           # 工具
│   │   │   └── file/       # 文件操作
│   │   ├── iot_hardware/    # IoT 硬件 (WiFi IoT Lite)
│   │   │   └── wifiiot_lite/
│   │   │       ├── BUILD.gn
│   │   │       ├── hal_*.c  # 各外设 HAL 实现
│   │   │       └── inc/     # 头文件
│   │   └── communication/  # 通信模块
│   │       ├── wifi_lite/
│   │       │   ├── wifiservice/
│   │       │   └── wifiaware/
│   │       └── BUILD.gn
│   ├── kal/                 # KAL (内核抽象层)
│   │   ├── cmsis/          # CMSIS-RTOS2 适配
│   │   │   ├── cmsis_*.c
│   │   │   ├── hos_cmsis_adp.h
│   │   │   └── BUILD.gn
│   │   ├── posix/          # POSIX 接口
│   │   │   ├── src/        # POSIX 实现
│   │   │   ├── include/    # 头文件
│   │   │   └── BUILD.gn
│   │   └── BUILD.gn
│   └── BUILD.gn
└── sdk_liteos/              # LiteOS SDK
    ├── app/                # 应用层
    │   ├── demo/           # 示例应用
    │   │   ├── include/    # 示例头文件
    │   │   └── src/        # 示例源码
    │   └── wifiiot_app/    # WiFi IoT 应用
    ├── boot/               # 启动加载器
    │   ├── commonboot/     # 通用启动代码
    │   ├── flashboot/      # Flash 启动
    │   │   ├── drivers/    # 驱动 (Flash/GPIO/EFuse 等)
    │   │   ├── upg/        # 升级相关
    │   │   └── secure/     # 安全启动
    │   └── loaderboot/     # 二级加载器
    │       ├── drivers/    # 驱动
    │       ├── secure/     # 安全
    │       └── fixed/      # 固定代码
    ├── components/         # SDK 组件
    │   └── at/            # AT 命令组件
    ├── config/             # 配置
    │   ├── diag/          # 诊断配置
    │   ├── nv/            # NV 配置
    │   └── system_config.h
    ├── include/            # SDK 头文件
    │   ├── hi_*.h         # 芯片接口头文件
    │   └── hi_stdlib.h
    ├── platform/           # 平台驱动
    │   ├── drivers/       # 外设驱动 (UART/I2C/SPI/PWM/ADC)
    │   ├── system/        # 系统服务 (升级/分区表)
    │   └── ...
    ├── third_party/        # 第三方库
    │   └── mbedtls/       # mbedtls
    └── BUILD.gn
```

## hi3516dv300 目录结构

**路径**: `hi3516dv300/`

```
hi3516dv300/
├── soc.gni              # SoC 配置 (include 路径等)
├── sdk_liteos/          # LiteOS SDK
│   ├── app/            # 应用代码
│   ├── boot/           # 启动加载器
│   ├── config/         # 配置
│   ├── hdf_config/     # HDF 配置 (.hcs)
│   ├── include/        # 头文件
│   ├── mpp/            # 媒体处理平台
│   │   ├── component/  # 组件层
│   │   ├── hal/        # HAL
│   │   ├── ko/         # 内核模块
│   │   └── BUILD.gn
│   ├── platform/       # 平台驱动
│   ├── third_party/    # 第三方
│   └── BUILD.gn
├── sdk_linux/          # Linux SDK
│   ├── sample/         # 示例
│   ├── config.gni      # 配置
│   └── BUILD.gn
└── uboot/              # U-Boot
```

## 目录职责归类

| 目录类型 | 位置 | 职责 |
|---------|------|------|
| 通用 HAL | `common/hal/` | HDI 接口实现，供所有芯片使用 |
| 平台驱动 | `common/platform/` | GPIO/I2C/SPI 等外设驱动 |
| 芯片适配 | `{chip}/hi*_adapter/` | OpenHarmony 适配层 |
| 芯片 SDK | `{chip}/sdk_*os/` | 各系统 SDK |
| 启动代码 | `{chip}/sdk_*/boot/` | Loaderboot/Flashboot |
| 构建配置 | `{chip}/BUILD.gn`, `*.gni` | GN 构建脚本 |

## 模块依赖关系

```
┌─────────────────────────────────────────┐
│          应用代码 (各芯片 app/)           │
├─────────────────────────────────────────┤
│       OpenHarmony 适配层 (adapter/)       │
│  ┌─────────────────────────────────────┐│
│  │  HAL (hals/) - 外设抽象              ││
│  │  KAL (kal/) - 内核抽象               ││
│  └─────────────────────────────────────┘├─────────────────────────────────────────┤
│              芯片 SDK (sdk_*/)           │
│  ┌─────────────────────────────────────┐│
│  │  platform/ - 平台驱动                 ││
│  │  mpp/ - 媒体处理                      ││
│  │  boot/ - 启动加载器                   ││
│  │  include/ - 头文件                    ││
│  └─────────────────────────────────────┘├─────────────────────────────────────────┤
│         通用代码 (common/)               │
│  ┌─────────────────────────────────────┐│
│  │  hal/ - 通用 HAL (HDI)               ││
│  │  platform/ - 通用驱动                 ││
│  └─────────────────────────────────────┘└─────────────────────────────────────────┘
```

## 相关文档

- 芯片说明: [02_Chips.md](02_Chips.md)
- HAL 模块: [04_HAL_Modules.md](04_HAL_Modules.md)
- 平台驱动: [05_Platform_Drivers.md](05_Platform_Drivers.md)
