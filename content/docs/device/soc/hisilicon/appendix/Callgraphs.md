# 关键调用链

本文档描述海思 SDK 中的关键调用路径，从入口到核心逻辑。

## 应用启动调用链

### Hi3861V100 应用启动

```
app_main.c (应用入口)
    │
    ├── OHOS_Main()  ──────────────────────┐
    │                                      │
    ├── AppInit()                          │
    │    ├── BoardInit()                   │
    │    │    ├── IoT_GPIO_Init()          │
    │    │    ├── IoT_UART_Init()         │
    │    │    └── IoT_I2C_Init()           │
    │    ├──WifiService_Init()             │
    │    │    ├── HiWifi_Init()           │
    │    │    └── HiWifi_RegEventCb()     │
    │    └── TasksInit()                  │
    │         ├── Wifi_Task()             │
    │         ├── Uart_Task()             │
    │         └── ...                     │
    │                                      │
    └── LOS_KernelStart() ──────────────────┘
              │
              ▼
      LiteOS Kernel
```

**代码证据**: `hi3861v100/sdk_liteos/app/wifiiot_app/src/app_main.c`

## OTA 升级调用链

```
OTA 升级触发
     │
     ├── Upgrade_Check()
     │    ├── Verify_Signature()     ← upg_check_secure.c
     │    ├── Check_Version()        ← upg_check.c
     │    └── Check_Integrity()      ← kernel_crypto.c
     │
     ├── Upgrade_Prepare()
     │    ├── Load_Partition()       ← flashboot
     │    └── Decrypt_Package()      ← kernel_crypto.c
     │
     └── Upgrade_Burn()
          ├── Write_Flash()          ← hi_flashboot_flash.c
          └── Verify_Flash()        ← upg_check_file.c
```

**代码证据**: `hi3861v100/sdk_liteos/platform/system/upg/`

## 媒体处理调用链 (MPP)

```
┌──────────────────────────────────────────────────────────────────┐
│  应用层 (APP)                                                     │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  MPP 组件层 (Component)                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │    VI    │──▶│  VPSS    │──▶│  VENC    │──▶│   File   │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  MPP HAL 层 (HAL)                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ VI_HAL   │  │VPSS_HAL  │  │VENC_HAL  │  │  ...     │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  芯片寄存器操作                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  寄存器读写、时钟配置、DMA 传输                              │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

**代码证据**: `hi3516dv300/sdk_liteos/mpp/`

## GPIO 操作调用链

```
应用调用
    │
    ├── hi_gpio_set_dir(pin, direction)   ← hi_gpio.h
    │       └── gpio_hal.c
    │              └── hi_gpio.c
    │                     └── 寄存器操作 (基地址 + 偏移)
    │
    ├── hi_gpio_write(pin, value)
    │       └── 同上
    │
    └── hi_gpio_read(pin)
             └── 同上
```

**代码证据**: `common/platform/gpio/src/hi_gpio.c`

## 启动调用链

```
ROM Bootloader
     │
     ▼
Loaderboot (二级加载器)
     │
     ├── init_critical_hw()
     │    ├── init_clock()      ← 时钟初始化
     │    ├── init_memory()     ← 内存初始化
     │    └── init_uart()       ← 串口初始化
     │
     ├── load_flashboot()
     │    ├── read_flash(addr, buf, len)
     │    └── verify_signature()
     │
     └── jump_to_flashboot()
              │
              ▼
Flashboot (Flash 启动加载器)
     │
     ├── init_platform()
     │    ├── init_drivers()
     │    ├── init_pmic()
     │    └── init_memory()
     │
     ├── load_kernel()
     │    ├── read_flash()
     │    └── verify_kernel()
     │
     └── boot_kernel()
              │
              ▼
LiteOS Kernel
```

**代码证据**: `hi3861v100/sdk_liteos/boot/`

## 安全验证调用链

```
启动验证
     │
     ├── Boot_Verify()
     │    ├── Verify_Loaderboot_Signature()  ← load_crypto.c
     │    ├── Verify_Flashboot_Signature()
     │    └── Verify_Kernel_Signature()
     │
     └── Chain_Of_Trust
          ├── ROM_Key
          │    └── Burned_In_EFuse
          │
          ├── Loaderboot_Key
          │    └── Signed_By_ROM_Key
          │
          ├── Flashboot_Key
          │    └── Signed_By_Loaderboot_Key
          │
          └── Kernel_Key
               └── Signed_By_Flashboot_Key
```

**代码证据**: `hi3861v100/sdk_liteos/boot/loaderboot/secure/`

## HDI 调用链

```
OpenHarmony Framework
     │
     ├── HDI_GetService()
     │    └── dlopen("libhdi_xxx.z.so")
     │
     ├── IHdi_XXX::Method()
     │    └── 实现层 (common/hal/)
     │            └── 芯片 SDK (hi*/sdk_*/)
     │                    └── 寄存器操作
     │
     └── 返回结果
```

**代码证据**: `common/hal/*/`

## 外设数据流

### 摄像头数据流

```
Camera Sensor
     │
     MIPI CSI
     │
     ▼
VI (Video Input)     ← mpp/component/vi/
     │
     ▼
VPSS (Video Process) ← mpp/component/vpss/
     │
     ▼
VENC (Encoder)      ← mpp/component/venc/
     │
     ▼
H.264/H.265 码流  → File / Network
```

### WiFi 数据流

```
WiFi Hardware
     │
     MAC/PHY
     │
     ▼
WiFi Driver         ← common/platform/wifi/
     │
     ▼
WiFi Service        ← hi3861_adapter/hals/communication/wifi_lite/
     │
     ▼
LwIP Stack          ← third_party/lwip/
     │
     ▼
Application
```
