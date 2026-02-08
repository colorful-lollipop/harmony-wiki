# HDF 配置详解

本文档详细说明 HDF (Hardware Driver Framework) 配置文件的语法和结构。

---

## 4.a.1 HDF 概述

**HDF** (Hardware Driver Framework) 是 OpenHarmony 的硬件驱动框架，提供统一的驱动开发模型。

### 4.a.1.1 HDF 架构

```
+-------------------------------------------+
|              用户空间                      |
|  +-------------------------------------+  |
|  | UHDF (用户态驱动)                    |  |
|  +-------------------------------------+  |
+-------------------------------------------+
                   ↓ IPC
+-------------------------------------------+
|              内核空间                      |
|  +-------------------------------------+  |
|  | KHDF (内核态驱动)                    |  |
|  +-------------------------------------+  |
|  +-------------------------------------+  |
|  | 驱动配置 (.hcs)                     |  |
|  +-------------------------------------+  |
+-------------------------------------------+
```

### 4.a.1.2 配置类型

| 类型 | 位置 | 说明 |
|-----|------|------|
| KHDF | `hdf_config/khdf/` | 内核态驱动配置 |
| UHDF | `hdf_config/uhdf/` | 用户态驱动配置 |

---

## 4.a.2 .hcs 语法

### 4.a.2.1 基本语法

**.hcs** (HDF Configuration Source) 是 HDF 的配置语言。

```hcs
#include "other_config.hcs"           // 包含其他配置

root {                                // 根节点
    module = "module_name";           // 模块名
    attr = value;                     // 属性

    // 子节点
    node_name {
        attr1 = value1;
        attr2 = value2;
    }
}
```

### 4.a.2.2 配置入口

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
    module = "hisilicon,hi35xx_chip";       // [证据：hdf.hcs:24]
}
```

---

## 4.a.3 设备配置示例

### 4.a.3.1 设备信息配置

**device_info/device_info.hcs**

```hcs
root {
    device_info {
        attr = value;
    }
}
```

### 4.a.3.2 平台驱动配置

**platform/i2c_config.hcs**

```hcs
root {
    module = "i2c_driver";
    i2c_config {
        bus_id = 0;
        speed = 100000;        // 100kHz
    }
}
```

### 4.a.3.3 WiFi 配置

**wifi/wlan_platform.hcs** [证据]

```hcs
#include "wlan_chip_hi3881.hcs"

root {
    module = "wlan_platform";
    wlan_platform {
        // WiFi 平台配置
    }
}
```

---

## 4.a.4 GN 构建配置

### 4.a.4.1 UHDF 构建

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

## 4.a.5 常用配置模块

| 模块 | 路径 | 用途 |
|-----|------|------|
| device_info | khdf/device_info/ | 设备信息 |
| platform | khdf/platform/ | I2C/UART/SPI/PWM |
| wifi | khdf/wifi/ | 无线网络 |
| sensor | khdf/sensor/ | 传感器 |
| audio | khdf/audio/ | 音频 |
| light | khdf/light/ | 灯光 |
| vibrator | khdf/vibrator/ | 振动 |
| input | khdf/input/ | 输入设备 |
| lcd | khdf/lcd/ | 显示 |
| camera | uhdf/camera/ | 摄像头 |
| usb | uhdf/usb/ | USB |

---

## 4.a.6 相关文档

- [配置体系](./04_Configuration.md)
- [产品配置详解](./04b_Product_Configuration.md)
- [构建指南](./06_Build.md)
