# 目录结构

## 2.1 顶层目录

```
/vendor/hisilicon/
├── hispark_aries/                    # [LiteOS] IP Camera 开发板
├── hispark_pegasus/                   # [LiteOS-M] WiFi IoT 开发板
├── hispark_pegasus_mini_system/       # [LiteOS-M] 最小系统
├── hispark_phoenix/                   # [标准系统] IP Camera
├── hispark_taurus/                    # [LiteOS] 标准设备
├── hispark_taurus_linux/              # [Linux] Linux 系统
├── hispark_taurus_mini_system/        # [LiteOS] 最小系统
├── hispark_taurus_standard/           # [标准系统] 标准系统
├── watchos/                           # [标准系统] 手表产品
├── README_zh.md                       # 中文说明
├── LICENSE                            # Apache 2.0 许可证
└── OAT.xml                            # 开源许可文件
```

---

## 2.2 典型产品目录结构

### 2.2.1 LiteOS-M 产品（hispark_pegasus）

```
hispark_pegasus/
├── BUILD.gn                           # GN 构建入口 [证据：BUILD.gn:3]
├── config.json                        # 产品配置 [证据：config.json]
├── hals/                              # 硬件抽象层
│   ├── audio/                         # 音频适配
│   ├── display/                      # 显示适配
│   └── ...
├── demo/                             # Demo 示例
│   ├── easy_wifi_demo/               # WiFi 示例
│   ├── environment_demo/             # 传感器示例
│   ├── mutex_demo/                   # 互斥锁示例
│   ├── mqtt_demo/                    # MQTT 示例
│   └── ...
└── ohos.build                         # OpenHarmony 构建入口 [证据：ohos.build]
```

### 2.2.2 标准系统产品（hispark_taurus_standard）

```
hispark_taurus_standard/
├── BUILD.gn                           # GN 构建入口 [证据：BUILD.gn:3]
├── config.json                        # 产品配置 [证据：config.json]
├── hals/                              # 硬件抽象层
│   ├── audio/                         # 音频适配
│   │   ├── audio_adapter.json         # 音频适配配置 [证据：hals/audio/audio_adapter.json]
│   │   ├── audio_paths.json           # 音频路径配置
│   │   ├── audio_effect.json          # 音效配置
│   │   └── alsa_*.json                # ALSA 配置
│   └── ...
├── hdf_config/                        # HDF 驱动配置
│   ├── khdf/                         # 内核态 HDF 配置
│   │   ├── hdf.hcs                   # HDF 配置入口 [证据：hdf_config/khdf/hdf.hcs]
│   │   ├── device_info/              # 设备信息
│   │   ├── platform/                 # 平台驱动 (I2C/UART/SPI/...)
│   │   ├── wifi/                     # 无线驱动
│   │   ├── sensor/                   # 传感器驱动
│   │   ├── audio/                    # 音频驱动
│   │   ├── light/                    # 灯光驱动
│   │   ├── vibrator/                 # 振动驱动
│   │   ├── input/                    # 输入设备
│   │   └── lcd/                      # LCD 驱动
│   └── uhdf/                         # 用户态 HDF 配置
│       ├── hdf.hcs                   # UHDF 配置入口
│       ├── camera/                   # 摄像头配置
│       └── usb/                      # USB 配置
├── preinstall-config/                 # 预安装配置
│   ├── install_list.json             # 安装列表 [证据：preinstall-config/install_list.json]
│   ├── install_list_permissions.json # 权限列表
│   ├── install_list_capability.json  # 能力列表
│   └── uninstall_list.json            # 卸载列表
├── power_config/                      # 电源配置
├── updater_config/                    # 升级配置
│   └── build_cfg.gni                 # 升级构建配置
├── ohos.build                         # OpenHarmony 构建入口
└── product.gni                       # 产品 GN 配置
```

### 2.2.3 LiteOS 产品（hispark_taurus）

```
hispark_taurus/
├── BUILD.gn                           # GN 构建入口
├── config.json                        # 产品配置
├── hals/                              # 硬件抽象层
│   └── audio/
├── hdf_config/                        # HDF 配置
│   ├── khdf/                         # 内核态配置
│   └── ...
├── kernel_configs/                    # 内核配置
│   ├── liteos_m/
│   └── linux/
├── init_configs/                      # 初始化配置
└── fs.yml                             # 文件系统配置 [证据：fs.yml]
```

---

## 2.3 关键目录详解

### 2.3.1 hals/ - 硬件抽象层

**用途**：存放板级硬件适配代码和配置

**组织方式**：

```
hals/
├── audio/                    # 音频 HAL
│   ├── audio_adapter.json   # 适配器配置
│   ├── audio_paths.json     # 路径配置
│   └── *.c / *.h            # C 语言实现
├── display/                 # 显示 HAL
├── sensor/                  # 传感器 HAL
└── utils/                   # 工具 HAL
    ├── token/               # Token 管理
    └── sys_param/           # 系统参数
```

### 2.3.2 hdf_config/ - HDF 驱动配置

**用途**：存放 Hardware Driver Framework 配置

**组织方式**：

```
hdf_config/
├── khdf/                          # 内核态配置 (.hcs)
│   ├── hdf.hcs                    # 配置入口 [证据：hdf.hcs:1-25]
│   ├── device_info/               # 设备信息配置
│   ├── platform/                  # 平台驱动配置
│   │   ├── i2c_config.hcs
│   │   ├── uart_config.hcs
│   │   ├── spi_config.hcs
│   │   └── ...
│   ├── wifi/                      # WiFi 驱动配置
│   ├── sensor/                    # 传感器配置
│   ├── audio/                     # 音频配置
│   ├── light/                     # 灯光配置
│   ├── vibrator/                  # 振动配置
│   ├── input/                     # 输入配置
│   └── lcd/                       # LCD 配置
└── uhdf/                          # 用户态配置 (.hcs)
    ├── hdf.hcs                    # 配置入口
    ├── camera/                    # 摄像头配置
    └── usb/                       # USB 配置
```

### 2.3.3 demo/ - 示例代码

**用途**：存放各功能的演示程序

**Demo 列表**：

| Demo | 功能 | 系统 | 关键文件 |
|-----|------|------|---------|
| easy_wifi_demo | WiFi STA/AP 模式 | LiteOS-M | wifi_starter.c [证据：demo/easy_wifi_demo/src/wifi_starter.c] |
| environment_demo | 传感器环境检测 | LiteOS-M | app_demo_environment.c |
| mutex_demo | 互斥锁 | LiteOS-M | mutex.c |
| mqtt_demo | MQTT 通信 | LiteOS-M | - |
| coap_demo | CoAP 通信 | LiteOS-M | - |

---

## 2.4 配置文件说明

### 2.4.1 产品配置 (config.json)

**格式**：JSON

**用途**：定义产品信息、子系统、组件

**示例结构**：

```json
{
  "product_name": "wifiiot_hispark_pegasus",     // 产品名称
  "type": "mini",                                 // 系统类型 [证据：config.json:3]
  "version": "3.0",                               // 版本
  "ohos_version": "OpenHarmony 1.0",            // OpenHarmony 版本
  "device_company": "hisilicon",                 // 厂商
  "device_build_path": "...",                    // 构建路径
  "board": "hispark_pegasus",                   // 开发板
  "kernel_type": "liteos_m",                     // 内核类型
  "kernel_is_prebuilt": true,                    // 是否预编译内核
  "subsystems": [                                // 子系统列表
    {
      "subsystem": "applications",
      "components": [...]
    },
    // ...
  ],
  "third_party_dir": "...",                      // 第三方目录
  "product_adapter_dir": "..."                   // 产品适配目录
}
```

### 2.4.2 HDF 配置 (.hcs)

**格式**：HDF Configuration Source

**用途**：定义设备驱动配置

**示例结构**：

```hcs
#include "device_info/device_info.hcs"
#include "platform/i2c_config.hcs"

root {
    module = "hisilicon,hi35xx_chip";          // 模块名 [证据：hdf.hcs:24]
}
// ...
```

---

## 2.5 目录职责速查

| 目录 | 职责 | 关键文件 |
|-----|------|---------|
| `*/hals/` | 硬件抽象层 | audio_adapter.json, *.c/*.h |
| `*/hdf_config/` | 驱动配置 | hdf.hcs, *.hcs |
| `*/kernel_configs/` | 内核配置 | - |
| `*/init_configs/` | 初始化配置 | - |
| `*/preinstall-config/` | 预安装配置 | install_list.json |
| `*/power_config/` | 电源配置 | - |
| `*/updater_config/` | 升级配置 | build_cfg.gni |
| `*/demo/` | 示例代码 | *.c, BUILD.gn |
| `*/BUILD.gn` | 构建入口 | - |
| `*/config.json` | 产品配置 | - |
| `*/fs.yml` | 文件系统 | - |

---

## 2.6 相关文档

- [产品系列](./03_Products.md)
- [配置体系](./04_Configuration.md)
- [Demo 示例](./05_Demos.md)
