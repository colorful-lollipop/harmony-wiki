# 配置标志

## Kconfig 配置选项

### 驱动配置菜单

**文件**: `drivers/Kconfig`

```
┌─────────────────────────────────────────────────────────────────┐
│                    Drivers Configuration Menu                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Enable Uart                                               │  │
│  │  ┌─────────────────────────────────────────────────────┐   │  │
│  │  │ [*] Enable HDF platform uart driver                 │   │  │
│  │  │ [ ] Simple Uart                                      │   │  │
│  │  │ [ ] NO Uart                                          │   │  │
│  │  └─────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Platform Char Device Drivers                             │  │
│  │  ┌─────────────────────────────────────────────────────┐   │  │
│  │  │ [*] Enable Platform Char Device Drivers           │   │  │
│  │  │ [*] Enable MMZ Platform Char Device Drivers       │   │  │
│  │  └─────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Net Device                                               │  │
│  │  ┌─────────────────────────────────────────────────────┐   │  │
│  │  │ [*] Enable Net Device                               │   │  │
│  │  └─────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### UART 配置选项

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|--------|------|--------|------|------|
| `DRIVERS_HDF_PLATFORM_UART` | bool | - | `DRIVERS_HDF_PLATFORM` | 启用 HDF UART 驱动 |
| `PLATFORM_UART_WITHOUT_VFS` | bool | - | - | 简单 UART (无 VFS) |
| `PLATFORM_NO_UART` | bool | - | - | 禁用 UART |

**证据**: `drivers/Kconfig:3-19`
```kconfig
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
```

### 字符设备配置选项

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|--------|------|--------|------|------|
| `DRIVERS_PLATFORM_CHAR_DEVICE` | bool | y | `FS_VFS` | 启用字符设备驱动 |
| `DRIVERS_MMZ_CHAR_DEVICE` | bool | y | `DRIVERS_PLATFORM_CHAR_DEVICE`, `FS_VFS` | 启用 MMZ 驱动 |

**证据**: `drivers/Kconfig:21-41`
```kconfig
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

### 网络设备配置选项

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|--------|------|--------|------|------|
| `DRIVERS_NETDEV` | bool | y | `DRIVERS`, `NET_LWIP_SACK` | 启用网络设备 |

**证据**: `drivers/Kconfig:36-41`
```kconfig
config DRIVERS_NETDEV
    bool "Enable Net Device"
    default y
    depends on DRIVERS && NET_LWIP_SACK
    help
      Answer Y to enable LiteOS support net device.
```

### MTD 存储配置选项

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|--------|------|--------|------|------|
| `DRIVERS_MTD` | bool | n | `DRIVERS`, `FS_VFS` | 启用 MTD 支持 |
| `DRIVERS_MTD_SPI_NOR` | bool | y | `DRIVERS_MTD` | 启用 SPI NOR Flash |
| `DRIVERS_MTD_SPI_NOR_HISFC350` | bool | - | `DRIVERS_MTD_SPI_NOR` | 启用 Hisfc350 |
| `DRIVERS_MTD_SPI_NOR_HIFMC100` | bool | - | `DRIVERS_MTD_SPI_NOR` | 启用 Hifmc100 |
| `DRIVERS_MTD_NAND` | bool | n | `DRIVERS_MTD` | 启用 NAND Flash |

**证据**: `drivers/Kconfig:43-86`
```kconfig
config DRIVERS_MTD
    bool "Enable MTD"
    default n
    depends on DRIVERS && FS_VFS
    help
      Answer Y to enable LiteOS support jffs2 multipartion.

# spi nor
config DRIVERS_MTD_SPI_NOR
    bool "Enable MTD spi_nor flash"
    default y
    depends on DRIVERS_MTD
```

## GN 构建配置标志

### 条件编译标志

| 标志 | 用途 | 使用位置 | 说明 |
|------|------|----------|------|
| `LOSCFG_DRIVERS_PLATFORM_CHAR_DEVICE` | 字符设备开关 | `drivers/char/BUILD.gn` | 条件编译 char 模块 |
| `LOSCFG_DRIVERS_MMZ_CHAR_DEVICE` | MMZ 开关 | `drivers/char/mmz/BUILD.gn` | 条件编译 mmz 模块 |
| `LOSCFG_DRIVERS_HDF_PLATFORM_UART` | UART 开关 | `drivers/uart/BUILD.gn` | 条件编译 uart 模块 |
| `LOSCFG_HW_RANDOM_ENABLE` | RNG 开关 | `drivers/virtio/BUILD.gn` | 条件编译 virtrng |

**证据**: `drivers/char/BUILD.gn:23`
```gn
module_switch = defined(LOSCFG_DRIVERS_PLATFORM_CHAR_DEVICE)
```

**证据**: `drivers/virtio/BUILD.gn:39-41`
```gn
if (defined(LOSCFG_HW_RANDOM_ENABLE)) {
  sources += [ "virtrng.c" ]
}
```

### 模块名称配置

| 变量 | 用途 | 示例 |
|------|------|------|
| `module_name` | 驱动模块名称 | `module_name = "hdf_uart"` |
| `module_switch` | 编译开关 | `module_switch = defined(LOSCFG_...)` |

**证据**: `drivers/uart/BUILD.gn:27-28`
```gn
module_switch = defined(LOSCFG_DRIVERS_HDF_PLATFORM_UART)
module_name = "hdf_uart"
```

## CMake/Make 配置标志

### lite.mk 配置变量

| 变量 | 说明 | 来源 |
|------|------|------|
| `SOC_COMPANY` | SoC 厂商 | `LOSCFG_DEVICE_COMPANY` |
| `SOC_PLATFORM` | 平台名称 | `LOSCFG_PLATFORM` |
| `DRIVERS_ROOT` | 驱动根目录 | `LITEOSTOPDIR/../../device/$(SOC_COMPANY)/drivers/` |

**证据**: `drivers/lite.mk:14-17`
```makefile
SOC_COMPANY := $(subst $\",,$(LOSCFG_DEVICE_COMPANY))
SOC_PLATFORM := $(subst $\",,$(LOSCFG_PLATFORM))

DRIVERS_ROOT := $(LITEOSTOPDIR)/../../device/$(SOC_COMPANY)/drivers/
```

### 库链接配置

| 库 | 条件 | 说明 |
|----|------|------|
| `-lvirtio` | 始终 | VirtIO 驱动库 |
| `-lplatform_char` | 始终 | 字符设备驱动库 |
| `-lhdf_uart` | `LOSCFG_DRIVERS_HDF_PLATFORM_UART=y` | HDF UART 驱动库 |

**证据**: `drivers/lite.mk:20-28`
```makefile
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

## 相关文档

| 文档 | 说明 |
|------|------|
| [GN 构建](04_GN_Build.md) | 构建配置详解 |
| [编译产物](05_Build_Artifacts.md) | 构建产物 |
| [常见问题](08_Troubleshooting.md) | 配置相关问题排查 |
