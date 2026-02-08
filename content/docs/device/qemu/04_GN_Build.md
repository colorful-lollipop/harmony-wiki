# GN 构建系统

## 构建概述

device_qemu 使用 OpenHarmony 标准的 GN (Generate Ninja) 构建系统，同时保持与 LiteOS 传统 Make 构建的兼容性。

### 构建入口文件

| 文件 | 类型 | 说明 |
|------|------|------|
| `drivers/BUILD.gn` | GN | 驱动模块构建入口 |
| `drivers/*.gni` | GNI | 构建配置导入 |
| `*/ohos.build` | JSON | OpenHarmony 子系统配置 |
| `drivers/lite.mk` | Make | LiteOS Make 兼容层 |

### 构建系统依赖

**证据**: `drivers/BUILD.gn:30`
```gn
import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")
```

## 驱动构建配置

### 根构建文件

**文件**: `drivers/BUILD.gn`

```gn
# Copyright (c) 2013-2019 Huawei Technologies Co., Ltd. All rights reserved.
# Copyright (c) 2020-2021 Huawei Device Co., Ltd. All rights reserved.
#
# SPDX-License-Identifier: GPL-2.0-only

import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")

group("drivers") {
  deps = [
    "char",
    "uart",
    "virtio",
  ]
}

config("public") {
}
```

**目标说明**:

| 目标 | 类型 | 输出 | 依赖 |
|------|------|------|------|
| `drivers` | group | - | `char`, `uart`, `virtio` |

### 字符设备驱动 (char/)

**文件**: `drivers/char/BUILD.gn`

```gn
import("//kernel/liteos_a/liteos.gni")

module_switch = defined(LOSCFG_DRIVERS_PLATFORM_CHAR_DEVICE)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  deps = [ "mmz" ]

  public_configs = [ ":public" ]
}

config("public") {
  include_dirs = [ "include" ]
}
```

**目标说明**:

| 目标 | 类型 | 条件开关 | 输出 | 依赖 |
|------|------|----------|------|------|
| `char` | kernel_module | `LOSCFG_DRIVERS_PLATFORM_CHAR_DEVICE` | libchar.a | `mmz` |
| `public` | config | - | - | - |

### 内存管理区域驱动 (char/mmz/)

**文件**: `drivers/char/mmz/BUILD.gn`

```gn
import("//kernel/liteos_a/liteos.gni")

module_switch = defined(LOSCFG_DRIVERS_MMZ_CHAR_DEVICE)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  sources = [ "mmz.c" ]

  public_configs = [ ":public" ]
}

config("public") {
  include_dirs = [ "include" ]
}
```

**目标说明**:

| 目标 | 类型 | 条件开关 | sources | 输出 |
|------|------|----------|---------|------|
| `mmz` | kernel_module | `LOSCFG_DRIVERS_MMZ_CHAR_DEVICE` | `mmz.c` | libmmz.a |

### UART 串口驱动 (uart/)

**文件**: `drivers/uart/BUILD.gn`

```gn
import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")

module_switch = defined(LOSCFG_DRIVERS_HDF_PLATFORM_UART)
module_name = "hdf_uart"
hdf_driver(module_name) {
  sources = [
    "uart.c",
    "uart_pl011.c",
  ]
}
```

**目标说明**:

| 目标 | 类型 | 条件开关 | sources | 输出 |
|------|------|----------|---------|------|
| `hdf_uart` | hdf_driver | `LOSCFG_DRIVERS_HDF_PLATFORM_UART` | `uart.c`, `uart_pl011.c` | libhdf_uart.a |

### VirtIO 虚拟化驱动 (virtio/)

**文件**: `drivers/virtio/BUILD.gn`

```gn
import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")

module_name = get_path_info(rebase_path("."), "name")
hdf_driver(module_name) {
  sources = [
    "fakesdio.c",
    "virtblock.c",
    "virtgpu.c",
    "virtinput.c",
    "virtmmio.c",
    "virtnet.c",
  ]
  if (defined(LOSCFG_HW_RANDOM_ENABLE)) {
    sources += [ "virtrng.c" ]
  }
  include_dirs = [
    "//drivers/hdf_core/framework/model/network/wifi/include/",
    "//drivers/hdf_core/framework/model/network/wifi/platform/include/",
    "//drivers/hdf_core/framework/model/network/wifi/core/components/eapol/",
  ]
}
```

**目标说明**:

| 目标 | 类型 | sources | 条件依赖 | 输出 |
|------|------|---------|----------|------|
| `virtio` | hdf_driver | `fakesdio.c`, `virtblock.c`, `virtgpu.c`, `virtinput.c`, `virtmmio.c`, `virtnet.c` | `LOSCFG_HW_RANDOM_ENABLE` → `virtrng.c` | libvirtio.a |

## 平台构建配置

### ohos.build 结构

**文件**: `*/ohos.build` (以 arm_virt 为例)

```json
{
  "parts": {
    "device_arm_virt": {
      "module_list": [
        "//device/qemu/arm_virt:arm_virt"
      ]
    }
  },
  "subsystem": "device_arm_virt"
}
```

**配置说明**:

| 字段 | 值 | 说明 |
|------|-----|------|
| `subsystem` | `device_arm_virt` | 子系统名称 |
| `parts.device_arm_virt` | object | 部件配置 |
| `module_list` | array | GN 模块路径列表 |

### ohos.build 文件列表

| 平台 | 文件路径 |
|------|----------|
| ARM 虚拟 | `arm_virt/ohos.build` |
| ARM 虚拟 Linux | `arm_virt/linux/ohos.build` |
| ARM MPS2-AN386 | `arm_mps2_an386/ohos.build` |
| ARM MPS3-AN547 | `arm_mps3_an547/ohos.build` |
| RISC-V 32位 | `riscv32_virt/ohos.build` |
| RISC-V 64位 | `riscv64_virt/ohos.build` |
| RISC-V 64位 Linux | `riscv64_virt/linux/ohos.build` |
| x86_64 虚拟 | `x86_64_virt/ohos.build` |
| x86_64 虚拟 Linux | `x86_64_virt/linux/ohos.build` |
| ESP32 | `esp32/ohos.build` |
| SmartL_E802 | `SmartL_E802/ohos.build` |

## LiteOS Make 兼容

### lite.mk 结构

**文件**: `drivers/lite.mk`

```makefile
SOC_COMPANY := $(subst $\",,$(LOSCFG_DEVICE_COMPANY))
SOC_PLATFORM := $(subst $\",,$(LOSCFG_PLATFORM))

DRIVERS_ROOT := $(LITEOSTOPDIR)/../../device/$(SOC_COMPANY)/drivers/

###################### SELF-DEVELOPED DRIVER ######################
LITEOS_BASELIB +=  -lvirtio -lplatform_char
LIB_SUBDIRS    += $(DRIVERS_ROOT)/virtio
LIB_SUBDIRS    += $(DRIVERS_ROOT)/char

###################### HDF DRIVER ######################
ifeq ($(LOSCFG_DRIVERS_HDF_PLATFORM_UART), y)
    LITEOS_BASELIB += -lhdf_uart
    LIB_SUBDIRS    += $(DRIVERS_ROOT)/uart
endif
```

**配置说明**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `SOC_COMPANY` | `$(LOSCFG_DEVICE_COMPANY)` | SoC 厂商配置 |
| `SOC_PLATFORM` | `$(LOSCFG_PLATFORM)` | 平台配置 |
| `DRIVERS_ROOT` | `device/$(SOC_COMPANY)/drivers/` | 驱动根目录 |

**库链接配置**:

| 库 | 链接条件 | 说明 |
|----|----------|------|
| `-lvirtio` | 始终 | VirtIO 驱动库 |
| `-lplatform_char` | 始终 | 字符设备驱动库 |
| `-lhdf_uart` | `LOSCFG_DRIVERS_HDF_PLATFORM_UART=y` | HDF UART 驱动库 |

## Kconfig 配置

### 驱动配置菜单

**文件**: `drivers/Kconfig`

```
# none hdf driver configs
choice
    prompt "Enable Uart"
    default DRIVERS_HDF_PLATFORM_UART
    help
      Enable simple uart (without vfs) only for litekernel.
      Enable general uart (with vfs) for full code.

config DRIVERS_HDF_PLATFORM_UART
    bool "Enable HDF platform uart driver"
    depends on DRIVERS_HDF_PLATFORM
    help
      Answer Y to enable HDF platform uart driver.

config PLATFORM_UART_WITHOUT_VFS
    bool "Simple Uart"
config PLATFORM_NO_UART
    bool "NO Uart"
endchoice

# platform char dev drivers config
config DRIVERS_PLATFORM_CHAR_DEVICE
    bool "Enable Platform Char Device Drivers"
    default y
    depends on FS_VFS
    help
      Enable Platform Char Device Drivers.

config DRIVERS_MMZ_CHAR_DEVICE
    bool "Enable MMZ Platform Char Device Drivers"
    default y
    depends on DRIVERS_PLATFORM_CHAR_DEVICE && FS_VFS
    help
      Enable MMZ Platform Char Device Drivers.
```

### 配置选项表

| 配置选项 | 类型 | 默认值 | 依赖 | 说明 |
|----------|------|--------|------|------|
| `DRIVERS_HDF_PLATFORM_UART` | bool | - | `DRIVERS_HDF_PLATFORM` | 启用 HDF UART 驱动 |
| `PLATFORM_UART_WITHOUT_VFS` | bool | - | - | 简单 UART (无 VFS) |
| `PLATFORM_NO_UART` | bool | - | - | 禁用 UART |
| `DRIVERS_PLATFORM_CHAR_DEVICE` | bool | y | `FS_VFS` | 启用字符设备驱动 |
| `DRIVERS_MMZ_CHAR_DEVICE` | bool | y | `DRIVERS_PLATFORM_CHAR_DEVICE`, `FS_VFS` | 启用 MMZ 驱动 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](02_Directory_Structure.md) | 目录与文件结构 |
| [编译产物](05_Build_Artifacts.md) | 构建产物清单 |
| [支持的平台](07_Platforms.md) | 各平台构建配置 |
| [附录：配置标志](appendix/Config_Flags.md) | 详细配置标志 |
