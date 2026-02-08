# 配置体系

本文档详细说明 Hisilicon Vendor 仓库中的各类配置文件及其配置方法。

---

## 4.1 配置类型总览

| 配置类型 | 格式 | 位置 | 用途 |
|---------|------|------|------|
| [产品配置](#42-产品配置-configjson) | JSON | `*/config.json` | 产品定义、子系统、组件 |
| [HDF 驱动配置](#43-hdf-驱动配置) | .hcs | `*/hdf_config/` | 设备驱动配置 |
| [HAL 配置](#44-硬件抽象层-hal-配置) | JSON/C | `*/hals/` | 板级硬件适配 |
| [预安装配置](#45-预安装配置) | JSON | `*/preinstall-config/` | 预装应用、权限 |
| [GN 构建配置](#46-gn-构建配置) | GN | `*.gni`, `BUILD.gn` | 构建目标定义 |
| [内核配置](#47-内核配置) | - | `*/kernel_configs/` | Linux/LiteOS 内核 |
| [初始化配置](#48-初始化配置) | - | `*/init_configs/` | 系统初始化 |
| [电源配置](#49-电源配置) | - | `*/power_config/` | 电源管理 |
| [文件系统配置](#410-文件系统配置) | YAML | `*/fs.yml` | 文件系统定义 |

---

## 4.2 产品配置 (config.json)

### 4.2.1 配置结构

**格式**：JSON

**位置**：
- `hispark_pegasus/config.json` [证据]
- `hispark_taurus_standard/config.json` [证据]
- `hispark_phoenix/config.json`

### 4.2.2 完整示例

```json
{
  "product_name": "wifiiot_hispark_pegasus",      // [必填] 产品名称
  "type": "mini",                                 // [必填] 系统类型: mini/light/standard/linux
  "version": "3.0",                               // [必填] 产品版本
  "ohos_version": "OpenHarmony 1.0",              // [必填] OpenHarmony 版本
  "device_company": "hisilicon",                  // [必填] 芯片厂商
  "device_build_path": "...",                    // [必填] 设备构建路径
  "board": "hispark_pegasus",                    // [必填] 开发板名称
  "kernel_type": "liteos_m",                     // [必填] 内核类型: liteos_m/liteos/linux
  "kernel_is_prebuilt": true,                    // [可选] 是否预编译内核
  "kernel_version": "",                          // [可选] 内核版本
  "subsystems": [                                // [必填] 子系统列表
    {
      "subsystem": "applications",
      "components": [
        { "component": "wifi_iot_sample_app", "features": [] }
      ]
    }
  ],
  "third_party_dir": "...",                      // [可选] 第三方目录
  "product_adapter_dir": "..."                   // [可选] 产品适配目录
}
```

### 4.2.3 字段说明

| 字段 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| product_name | string | 是 | 产品唯一标识 |
| type | string | 是 | 系统类型 |
| version | string | 是 | 产品版本号 |
| device_company | string | 是 | 芯片厂商 |
| board | string | 是 | 开发板名称 |
| kernel_type | string | 是 | 内核类型 |
| subsystems | array | 是 | 子系统配置 |

### 4.2.4 系统类型

| type 值 | 对应系统 | 说明 |
|--------|---------|------|
| mini | LiteOS-M | 轻量级 IoT 系统 |
| light | LiteOS | 轻量级设备系统 |
| standard | 标准系统 | 完整功能系统 |
| linux | Linux | 通用 Linux 系统 |

---

## 4.3 HDF 驱动配置

### 4.3.1 配置概述

**用途**：定义 Hardware Driver Framework 设备驱动配置

**格式**：.hcs (HDF Configuration Source)

**位置**：`*/hdf_config/`

### 4.3.2 目录结构

```
hdf_config/
├── khdf/                                    # 内核态配置
│   ├── hdf.hcs                              # 配置入口 [证据：hdf.hcs:1-25]
│   ├── device_info/
│   │   └── device_info.hcs
│   ├── platform/
│   │   ├── i2c_config.hcs
│   │   ├── uart_config.hcs
│   │   ├── spi_config.hcs
│   │   ├── pwm_config.hcs
│   │   ├── sdio_config.hcs
│   │   └── emmc_config.hcs
│   ├── wifi/
│   │   ├── wlan_platform.hcs               # WiFi 平台配置 [证据]
│   │   └── wlan_chip_hi3881.hcs             # WiFi 芯片配置 [证据]
│   ├── sensor/
│   │   └── sensor_config.hcs
│   ├── audio/
│   │   ├── audio_config.hcs
│   │   ├── codec_config.hcs
│   │   ├── dai_config.hcs
│   │   ├── dma_config.hcs
│   │   └── dsp_config.hcs
│   ├── light/
│   │   └── light_config.hcs
│   ├── vibrator/
│   │   └── vibrator_config.hcs
│   ├── input/
│   │   └── input_config.hcs
│   └── lcd/
│       └── lcd_config.hcs
└── uhdf/                                    # 用户态配置
    ├── hdf.hcs                              # 配置入口
    ├── camera/
    │   └── ...
    └── usb/
        ├── usb_ecm_acm.hcs                  # USB ECM/ACM 配置
        └── usb_pnp_device.hcs               # USB 即插即用配置
```

### 4.3.3 配置入口示例

**khdf/hdf.hcs** [证据：hispark_taurus_standard/hdf_config/khdf/hdf.hcs]

```hcs
#include "device_info/device_info.hcs"
#include "platform/i2c_config.hcs"
#include "platform/hi35xx_watchdog_config.hcs"
#include "platform/hi35xx_pwm_config.hcs"
#include "platform/hi35xx_uart_config.hcs"
#include "platform/sdio_config.hcs"
#include "platform/emmc_config.hcs"
#include "platform/hi35xx_spi_config.hcs"
#include "input/input_config.hcs"
#include "wifi/wlan_platform.hcs"
#include "wifi/wlan_chip_hi3881.hcs"
#include "sensor/sensor_config.hcs"
#include "audio/audio_config.hcs"
#include "audio/codec_config.hcs"
#include "audio/dai_config.hcs"
#include "audio/dma_config.hcs"
#include "audio/dsp_config.hcs"
#include "light/light_config.hcs"
#include "vibrator/vibrator_config.hcs"
#include "vibrator/linear_vibrator_config.hcs"
#include "lcd/lcd_config.hcs"

root {
    module = "hisilicon,hi35xx_chip";       // 模块名 [证据：hdf.hcs:24]
}
```

### 4.3.4 GN 构建配置

**uhdf/BUILD.gn** [证据：hispark_taurus_standard/hdf_config/uhdf/BUILD.gn]

```gn
import("//drivers/hdf_core/adapter/uhdf2/hcs/hcs.gni")

hdf_hcb("hdf_default.hcb") {
  source = "./hdf.hcs"
  part_name = "product_hispark_taurus_standard"
  subsystem_name = "product_hisilicon"
}

hdf_cfg("hdf_devhost.cfg") {
  source = "./hdf.hcs"
  part_name = "product_hispark_taurus_standard"
  subsystem_name = "product_hisilicon"
}

group("hdf_config") {
  deps = [
    ":hdf_default.hcb",
    ":hdf_devhost.cfg",
  ]
}
```

---

## 4.4 硬件抽象层 (HAL) 配置

### 4.4.1 音频配置

**位置**：`*/hals/audio/`

**文件列表**：

| 文件 | 用途 |
|-----|------|
| audio_adapter.json | 音频适配器配置 [证据：hispark_taurus_standard/hals/audio/audio_adapter.json] |
| audio_paths.json | 音频路径配置 |
| audio_effect.json | 音效配置 |
| alsa_adapter.json | ALSA 适配器配置 |
| alsa_paths.json | ALSA 路径配置 |

---

## 4.5 预安装配置

### 4.5.1 配置目录

**位置**：`*/preinstall-config/`

**文件列表**：

| 文件 | 用途 |
|-----|------|
| install_list.json | 预安装应用列表 [证据] |
| install_list_permissions.json | 预装应用权限 |
| install_list_capability.json | 预装应用能力 |
| uninstall_list.json | 卸载列表 |

---

## 4.6 GN 构建配置

### 4.6.1 构建入口

**BUILD.gn** [证据：hispark_pegasus_mini_system/BUILD.gn]

```gn
# Copyright (C) 2020 Hisilicon (Shanghai) Technologies Co., Ltd. All rights reserved.

group("hispark_pegasus_mini_system") {
}
```

### 4.6.2 产品构建

**hispark_taurus_standard/BUILD.gn** [证据]

```gn
# Copyright (C) 2023 Hisilicon (Shanghai) Technologies Co., Ltd. All rights reserved.

group("hispark_taurus_standard") {
  deps = [ "preinstall-config:preinstall-config" ]
}
```

---

## 4.7 内核配置

**位置**：`*/kernel_configs/`

**用途**：存放 Linux 或 LiteOS 内核配置文件

---

## 4.8 初始化配置

**位置**：`*/init_configs/`

**用途**：存放系统初始化配置文件

---

## 4.9 电源配置

**位置**：`*/power_config/`

**用途**：存放电源管理配置文件

---

## 4.10 文件系统配置

### 4.10.1 配置格式

**位置**：`*/fs.yml`

**格式**：YAML

### 4.10.2 示例

```yaml
# 文件系统定义
fs:
  type: fat
  mount_point: /sdcard
  device: /dev/mmcblk0p1
```

---

## 4.11 相关文档

- [HDF 配置详解](./04a_HDF_Configuration.md)
- [产品配置详解](./04b_Product_Configuration.md)
- [构建指南](./06_Build.md)
