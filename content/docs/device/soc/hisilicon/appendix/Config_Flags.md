# 配置选项

本文档描述海思 SDK 中关键的配置宏、Feature Flags 和编译选项。

## SDK 配置宏

### Hi3861V100 SDK 配置

**代码证据**: `hi3861v100/sdk_liteos/config/system_config.h`

| 宏 | 说明 | 默认值 |
|----|------|--------|
| `LOSCFG_PLATFORM_HI3861V100` | 平台类型 | 定义 |
| `LOSCFG_KERNEL_CPUP` | CPU 使用率统计 | 定义 |
| `LOSCFG_BASE_CORE_SWTMR` | 软件定时器 | 定义 |
| `LOSCFG_BASE_CORE_PIP` | 进程间通信 | 定义 |
| `LOSCFG_BASE_CORE_TLS` | 线程本地存储 | 定义 |

### 配置头文件

| 头文件 | 功能 |
|--------|------|
| `config/system_config.h` | 系统配置 |
| `config/diag/dfx_sal.h` | 诊断配置 |
| `config/diag/dfx_sys.h` | 系统诊断 |
| `config/nv/hi_sal_nv.h` | NV 配置 |
| `config/nv/hi_ft_nv.h` | FT NV 配置 |

## 芯片头文件

### Hi3861V100 芯片头文件

**代码证据**: `hi3861v100/sdk_liteos/include/`

| 头文件 | 功能 |
|--------|------|
| `hi_types.h` | 类型定义 |
| `hi_stdlib.h` | 标准库扩展 |
| `hi_gpio.h` | GPIO |
| `hi_uart.h` | UART |
| `hi_i2c.h` | I2C |
| `hi_spi.h` | SPI |
| `hi_adc.h` | ADC |
| `hi_pwm.h` | PWM |
| `hi_flash_base.h` | Flash |
| `hi_cipher.h` | 加密 |
| `hi_efuse.h` | EFuse |
| `hi_task.h` | 任务 |
| `hi_sem.h` | 信号量 |
| `hi_mux.h` | 互斥锁 |
| `hi_event.h` | 事件 |
| `hi_watchdog.h` | 看门狗 |
| `hi_tsensor.h` | 温度传感器 |
| `hi_at.h` | AT 命令 |
| `hi_diag.h` | 诊断 |
| `hi_hook_fuc.h` | Hook 函数 |
| `hi_wifi_api.h` | WiFi API |
| `hi_net_api.h` | 网络 API |
| `hi_partition_table.h` | 分区表 |

## HDF 配置

### Hi3516DV300 HDF 配置

**路径**: `hi3516dv300/sdk_liteos/hdf_config/hdf.hcs`

```hcs
root {
    host {
        device_manager :: host {
            device_gpio :: device {
                gpioConfig {
                    policy = 2;
                    priority = 10;
                    permission = 0664;
                }
                deviceNode {
                    name = "gpio0";
                    ...
                }
            }
        }
    }
}
```

### 配置说明

| 配置项 | 说明 |
|--------|------|
| `policy` | 驱动策略 (0=core, 1=service, 2=unload) |
| `priority` | 启动优先级 |
| `permission` | 设备节点权限 |

## GN 构建配置

### 根配置

**文件**: `ohos.build`

```json
{
  "subsystem": "vendor",
  "parts": {
    "hardware": {
      "module_list": [
        "//device/soc/hisilicon/common/hal/media:hardware_media_sdk"
      ]
    },
    "hi3861_sdk": {
      "module_list": [
        "//device/soc/hisilicon/hi3861v100/sdk_liteos:wifiiot_sdk"
      ]
    }
  }
}
```

### 芯片配置 (soc.gni)

**文件**: `{chip}/soc.gni`

```gn
# 芯片特定配置
chip_name = "hi3516dv300"

# Include 路径
include_dirs = [
  "//device/soc/hisilicon/{chip}/sdk_linux/include",
  "//device/soc/hisilicon/common/platform/include",
]

# 编译选项
defines = [
  "CHIP_{CHIP}_LITEOS",
]
```

### config.gni

**文件**: `{chip}/sdk_linux/config.gni`

```gn
# 目标配置
target_cpu = "arm"
target_os = "linux"
target_arch = "arm"

# 工具链
Clang_prefix = "arm-linux-gnueabi-"

# SDK 路径
sdk_src_dir = "//device/soc/hisilicon/{chip}/sdk_linux"
```

## Feature Flags

### 安全特性

| 特性 | 宏 | 说明 |
|------|-----|------|
| 安全启动 | `CONFIG_SECURE_BOOT` | 启动镜像签名验证 |
| Flash 加密 | `CONFIG_FLASH_ENCRYPT` | Flash 数据加密 |
| 安全调试 | `CONFIG_SEC_DEBUG` | 安全调试接口 |

### 网络特性

| 特性 | 宏 | 说明 |
|------|-----|------|
| WiFi 支持 | `CONFIG_WIFI` | WiFi 功能 |
| 以太网支持 | `CONFIG_ETH` | 以太网功能 |
| LwIP 支持 | `CONFIG_LWIP` | TCP/IP 协议栈 |

### 媒体特性

| 特性 | 宏 | 说明 |
|------|-----|------|
| 视频编码 | `CONFIG_VENC` | 视频编码 |
| 视频解码 | `CONFIG_VDEC` | 视频解码 |
| ISP 支持 | `CONFIG_ISP` | 图像信号处理 |

## Makefile 配置

### lite.mk

**文件**: `common/platform/lite.mk`

```mk
# 平台类型
PLATFORM := hi3861v100

# 编译选项
CFLAGS += -Wall -Werror

# 头文件路径
INC_FLAGS += -I$(SRC_DIR)/include

# 链接库
LIBS += -lmbedtls -lsecurec
```

## 内存配置

### 堆配置

```c
// 系统堆大小
#define LOSCFG_BASE_HEAP_ADJUST_SIZE 0x10000

// 任务堆栈大小
#define LOSCFG_TASK_STACK_SIZE 8192
```

### 分区表配置

```c
// 分区表定义
#define BOOTLOADER_START     0x0
#define BOOTLOADER_SIZE      0x40000     // 256KB
#define FLASHBOOT_START      0x40000
#define FLASHBOOT_SIZE        0x100000    // 1MB
#define KERNEL_START         0x140000
#define KERNEL_SIZE          0x400000    // 4MB
```

**代码证据**: `hi3861v100/sdk_liteos/include/hi_partition_table.h`

## 调试配置

### 日志级别

| 级别 | 宏 | 用途 |
|------|-----|------|
| ERROR | `AES_LOG_ERROR` | 错误日志 |
| WARN | `AES_LOG_WARN` | 警告日志 |
| INFO | `AES_LOG_INFO` | 信息日志 |
| DEBUG | `AES_LOG_DEBUG` | 调试日志 |

### 诊断开关

| 开关 | 宏 | 说明 |
|------|-----|------|
| 内存调试 | `LOSCFG_MEM_DEBUG` | 内存调试 |
| 任务调试 | `LOSCFG_TASK_DEBUG` | 任务调试 |
| 性能分析 | `LOSCFG_CPUP` | CPU 使用率 |
