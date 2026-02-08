# SDK 架构说明

本文档描述海思芯片 SDK 的整体架构设计，包括各组件的分层关系和交互方式。

## SDK 架构总览

海思 SDK 采用分层架构设计：

```
┌─────────────────────────────────────────────────────────┐
│                    应用层 (Application)                   │
├─────────────────────────────────────────────────────────┤
│              OpenHarmony 适配层 (Adapter)                │
│  ┌─────────────────────────────────────────────────────┐│
│  │  HAL (Hardware Abstraction Layer)                   ││
│  │  KAL (Kernel Abstraction Layer)                      ││
│  └─────────────────────────────────────────────────────┘├─────────────────────────────────────────────────────────┤
│                     SDK 核心层                            │
│  ┌─────────────────────────────────────────────────────┐│
│  │  MPP (Media Process Platform)                       ││
│  │  平台驱动 (Platform Drivers)                        ││
│  │  启动代码 (Boot)                                     ││
│  │  系统服务 (System Services)                          ││
│  └─────────────────────────────────────────────────────┘├─────────────────────────────────────────────────────────┤
│                     芯片抽象层                            │
│  ┌─────────────────────────────────────────────────────┐│
│  │  芯片寄存器头文件                                    ││
│  │  外设驱动接口                                        ││
│  │  内存映射                                            ││
│  └─────────────────────────────────────────────────────┘└─────────────────────────────────────────────────────────┘
```

## SDK 目录结构模式

### LiteOS SDK 模式 (轻量/小型系统)

```
{chip}/sdk_liteos/
├── app/                    # 应用代码
│   ├── demo/              # 示例应用
│   └── {app_name}/        # 用户应用
├── boot/                   # 启动加载器
│   ├── commonboot/        # 通用启动代码
│   ├── flashboot/         # Flash 启动
│   └── loaderboot/        # 二级加载器
├── components/             # SDK 组件
│   └── at/               # AT 命令组件
├── config/                 # 配置
│   ├── diag/              # 诊断配置
│   ├── nv/                # NV 配置
│   └── system_config.h   # 系统配置
├── include/                # SDK 头文件
│   ├── hi_*.h             # 芯片接口头文件
│   └── *.h                # 其他头文件
├── platform/               # 平台驱动
│   ├── drivers/          # 外设驱动
│   └── system/            # 系统服务
├── third_party/            # 第三方库
│   └── mbedtls/          # mbedtls
└── BUILD.gn               # 构建配置
```

### Linux SDK 模式 (标准系统)

```
{chip}/sdk_linux/
├── out/                    # 输出产物
│   └── lib/               # 库文件
├── sample/                 # 示例代码
│   ├── platform/         # 平台示例
│   └── taurus/           # 芯片示例
├── config.gni             # 构建配置
└── BUILD.gn
```

## 适配层架构

### HAL (Hardware Abstraction Layer)

**位置**: `{chip}/hi3861_adapter/hals/`

提供硬件接口抽象，统一上层调用：

```
hals/
├── update/               # OTA 升级抽象
├── utils/               # 工具类抽象
│   └── file/           # 文件操作
└── iot_hardware/        # IoT 硬件
    └── wifiiot_lite/   # WiFi IoT Lite
        ├── hal_gpio.c   # GPIO 实现
        ├── hal_uart.c  # UART 实现
        ├── hal_i2c.c   # I2C 实现
        ├── hal_spi.c   # SPI 实现
        ├── hal_adc.c   # ADC 实现
        ├── hal_pwm.c   # PWM 实现
        └── hal_flash.c # Flash 实现
```

**代码证据**: `hi3861v100/hi3861_adapter/hals/iot_hardware/wifiiot_lite/`

### KAL (Kernel Abstraction Layer)

**位置**: `{chip}/hi3861_adapter/kal/`

提供内核接口抽象，支持不同 RTOS：

```
kal/
├── cmsis/              # CMSIS-RTOS2 适配
│   ├── cmsis_liteos.c
│   ├── cmsis_liteos2.c
│   ├── cmsis_os.h
│   ├── cmsis_os2.h
│   ├── hos_cmsis_adp.h
│   └── BUILD.gn
└── posix/              # POSIX 接口
    ├── src/           # POSIX 实现
    │   ├── pthread.c
    │   ├── time.c
    │   ├── file.c
    │   └── ...
    ├── include/       # 头文件
    │   ├── pthread.h
    │   ├── time.h
    │   └── ...
    └── BUILD.gn
```

**代码证据**: `hi3861v100/hi3861_adapter/kal/`

## MPP 媒体处理平台

**位置**: `hi3516dv300/sdk_liteos/mpp/`

### 架构

```
MPP (Media Process Platform)
├── component/           # 组件层
│   ├── vi/             # Video Input
│   ├── vpss/          # Video Process Subsystem
│   ├── venc/          # Video Encoder
│   ├── vdec/          # Video Decoder
│   ├── vo/            # Video Output
│   └── region/        # Region (OSD)
├── hal/                # 硬件抽象层
│   ├── vi_hal/        # VI HAL
│   ├── vpss_hal/      # VPSS HAL
│   └── ...
└── ko/                 # 内核模块
```

### 数据流

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Camera   │───▶│    VI   │───▶│  VPSS   │───▶│  VENC   │
│ Sensor   │    │         │    │         │    │         │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
                                           │
                                           ▼
                                     ┌─────────┐
                                     │  File   │
                                     │ /Stream │
                                     └─────────┘
```

## 启动架构

### 启动流程

```
ROM Bootloader
    │
    ▼
Loaderboot (二级加载器)
    │
    ├── 初始化时钟、内存
    ├── 加载 Flashboot
    └── 验证签名
    │
    ▼
Flashboot (Flash 启动加载器)
    │
    ├── 初始化外设
    ├── 加载内核镜像
    └── 验证内核签名
    │
    ▼
Kernel (LiteOS/Linux)
    │
    ├── 初始化系统
    ├── 挂载文件系统
    └── 启动应用
    │
    ▼
Application
```

### 启动代码结构

**代码证据**: `hi3861v100/sdk_liteos/boot/`

```
boot/
├── commonboot/           # 通用启动代码
│   ├── adc_drv.h
│   ├── hi_types.h
│   ├── hi3861_platform.h
│   ├── hi_cipher.h
│   └── ...
├── flashboot/           # Flash 启动
│   ├── drivers/
│   │   ├── flash/      # Flash 驱动
│   │   ├── gpio/       # GPIO 驱动
│   │   ├── efuse/      # EFuse 驱动
│   │   └── lsadc/      # LSADC 驱动
│   ├── upg/            # 升级相关
│   ├── secure/         # 安全启动
│   ├── startup/       # 启动入口
│   └── BUILD.gn
├── loaderboot/         # 二级加载器
│   ├── drivers/
│   ├── secure/
│   ├── fixed/
│   ├── startup/
│   └── BUILD.gn
└── BUILD.gn
```

## 升级架构

### OTA 升级流程

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   升级包      │───▶──│   校验       │───▶──│   烧写       │
│   下载       │      │   验证签名   │      │   分区写入   │
└──────────────┘      └──────────────┘      └──────────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │   回滚       │
                     │   失败恢复   │
                     └──────────────┘
```

### 升级相关代码

**代码证据**: `hi3861v100/sdk_liteos/platform/system/upg/`

| 文件 | 功能 |
|------|------|
| `upg_common.c` | 升级通用接口 |
| `upg_check.c` | 升级包校验 |
| `upg_check_secure.c` | 安全校验 |
| `upg_check_boot_bin.c` | 启动镜像校验 |
| `upg_check_file.c` | 文件校验 |
| `upg_user_verify.c` | 用户校验 |
| `upg_start_up.c` | 启动升级 |
| `kernel_crypto.c` | 内核加密 |
| `kernel_crypto.h` | 加密头文件 |

## 头文件组织

### SDK 头文件

**代码证据**: `hi3861v100/sdk_liteos/include/`

| 头文件 | 功能 |
|--------|------|
| `hi_types.h` | 类型定义 |
| `hi_stdlib.h` | 标准库扩展 |
| `hi_gpio.h` | GPIO 接口 |
| `hi_uart.h` | UART 接口 |
| `hi_i2c.h` | I2C 接口 |
| `hi_spi.h` | SPI 接口 |
| `hi_adc.h` | ADC 接口 |
| `hi_pwm.h` | PWM 接口 |
| `hi_flash_base.h` | Flash 接口 |
| `hi_cipher.h` | 加密接口 |
| `hi_efuse.h` | EFuse 接口 |
| `hi_task.h` | 任务接口 |
| `hi_sem.h` | 信号量接口 |
| `hi_mux.h` | 互斥锁接口 |
| `hi_event.h` | 事件接口 |

## 模块依赖

### 依赖方向图

```
Application
    │
    ├── depends on ──────────────┐
    │                           │
Adapter (HAL/KAL)               │
    │                           │
    ├── depends on ─────────────┼──────────┐
    │                           │          │
Platform Drivers                │          │
    │                           │          │
    ├── depends on ─────────────┤          │
    │                           │          │
Chip Headers (hi_*.h)          │          │
    │                           │          │
    └── depends on ─────────────┴──────────┘
                  │
                  ▼
           芯片寄存器映射
```

## 相关文档

- 目录结构: [03_Directory_Structure.md](03_Directory_Structure.md)
- HAL 模块: [04_HAL_Modules.md](04_HAL_Modules.md)
- 构建系统: [07_Build_System.md](07_Build_System.md)
