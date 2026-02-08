# 目录结构与模块职责

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库的目录组织、各模块职责和文件类型
- **适用范围**：vendor/hihope 仓库全部 13 个产品线
- **最后更新**：2025-02-06
- **关键结论**：
  - 所有产品遵循一致的目录结构模式
  - 标准产品与 Mini 系统在目录复杂度上差异明显
  - 功能区域清晰划分：hals/, hdf_config/, security_config/ 等

## 整体目录树

```
vendor/hihope/
├── CODEOWNERS                          # 代码所有者定义
├── LICENSE                             # Apache License 2.0
├── OAT.xml                             # OpenHarmony 归属工具配置
├── README_zh.md                        # 中文文档（产品工程创建指南）
│
├── 2in1_core_system/                   # 2 合 1 设备核心系统
├── dayu210/                            # DAYU210 开发板
├── default_core_system/                 # 默认核心系统
├── ipcamera_core_system/              # IPC 摄像头核心系统
├── nearlink_dk_3863/                   # NearLink 开发套件
│   └── ws63_sample/                   # 28 个教程示例
├── nearlink_dk_3863_xts/               # NearLink XTS 测试套件
├── neptune_iotlink_demo/               # 海王星 IoT 演示
├── rk3568/                             # RK3568 完整系统（最完整）
├── rk3568_mini_system/               # RK3568 精简系统
├── tablet_core_system/                  # 平板设备核心系统
├── tv/                                 # 智能电视核心系统
└── wearable/                           # 可穿戴设备核心系统
```

## 标准产品目录结构

### 标准产品列表

以下产品采用一致的目录结构：
- 2in1_core_system
- dayu210
- default_core_system
- ipcamera_core_system
- tablet_core_system
- tv
- wearable

### 标准目录组织

```
{standard_product}/
├── bluetooth/                          # 蓝牙 HAL 和配置
├── default_app_config/                # 默认应用配置
│   └── default_app.json            # 默认应用列表
├── etc/                               # 系统配置文件
│   └── param/                       # 系统参数
├── hals/                              # 硬件抽象层（核心内容）
│   ├── audio/                        # 音频 HAL
│   │   ├── BUILD.gn
│   │   ├── audio_adapter.json       # 音频适配器配置
│   │   ├── audio_effect.json        # 音效配置
│   │   ├── audio_paths.json         # 音频路径配置
│   │   ├── product.gni             # 音频产品配置
│   │   └── config/                # 音频策略配置
│   │       ├── arm/                # ARM 策略
│   │       └── arm64/              # ARM64 策略
│   └── codec/                        # 编解码器 HAL
│       ├── BUILD.gn
│       └── product.gni
├── hdf_config/                        # 硬件驱动框架配置
│   ├── khdf/                        # 内核态 HDF 配置
│   │   ├── device_info/              # 设备信息
│   │   │   └── device_info.hcs    # 设备信息配置（901 行）
│   │   ├── audio/                  # 音频驱动配置
│   │   ├── camera/                 # 相机驱动配置
│   │   ├── device_info/            # 设备信息
│   │   ├── input/                  # 输入驱动配置
│   │   ├── lcd/                    # LCD 显示配置
│   │   ├── light/                  # 光线传感器配置
│   │   ├── platform/               # 平台设备配置
│   │   │   ├── adc_config_linux.hcs
│   │   │   ├── emmc_config.hcs
│   │   │   ├── i2c_config.hcs
│   │   │   ├── pwm_config.hcs
│   │   │   ├── rk3568_spi_config.hcs
│   │   │   ├── rk3568_uart_config.hcs
│   │   │   ├── rk3568_watchdog_config.hcs
│   │   │   ├── sdio_config.hcs
│   │   │   ├── i2c_config.hcs
│   │   │   └── spi_config.hcs
│   │   ├── sensor/                 # 传感器配置
│   │   │   ├── accel/           # 加速度
│   │   │   │   ├── mxc6655xa_config.hcs
│   │   │   │   └── bmi160_config.hcs
│   │   │   ├── als/             # 环境光
│   │   │   │   └── bh1745_config.hcs
│   │   │   ├── gas/             # 气体传感器
│   │   │   │   └── bme688_config.hcs
│   │   │   ├── gyro/            # 陀螺仪
│   │   │   ├── magnetic/       # 磁力
│   │   │   │   └── lsm303_config.hcs
│   │   │   ├── proximity/      # 距离
│   │   │   │   └── apds9960_config.hcs
│   │   │   ├── temperature/    # 温度
│   │   │   │   ├── aht20_config.hcs
│   │   │   │   └── sht30_config.hcs
│   │   │   └── humidity/      # 湿度
│   │   │       ├── aht20_config.hcs
│   │   │       └── sht30_config.hcs
│   │   ├── sensor_config.hcs
│   │   ├── sensor_common.hcs
│   │   └── sensor/           # 所有传感器配置目录
│   │       └── sensor_config.hcs
│   │           ├── als/          # 光线
│   │           ├── gas/          # 气体
│   │           ├── humidity/     # 湿度
│   │           ├── temperature/   # 温度
│   │           ├── proximity/    # 距离
│   │           ├── accel/        # 加速度
│   │           ├── gyro/         # 陀螺仪
│   │           ├── magnetic/     # 磁力
│   │           └── barometer/    # 气压
│   ├── vibrator/                    # 振动器配置
│   │   ├── vibrator_config.hcs
│   │   ├── drv2605l_linear_vibrator_config.hcs
│   │   └── linear_vibrator_config.hcs
│   ├── wifi/                        # WiFi 配置
│   │   ├── wlan_chip_ap6275s.hcs
│   │   ├── wlan_chip_hi3881.hcs
│   │   └── wlan_platform.hcs
│   ├── audio/                       # 音频驱动配置
│   │   ├── analog_headset_config.hcs
│   │   ├── audio_config.hcs
│   │   ├── codec_config.hcs
│   │   ├── dai_config.hcs
│   │   ├── dma_config.hcs
│   │   └── dsp_config.hcs
│   ├── light/                      # 光线配置
│   │   └── light_config.hcs
│   ├── input/                      # 输入配置
│   │   ├── input_config.hcs
│   │   └── input_event1_config.hcs（触摸）
│   │   └── input_event2_config.hcs（红外）
│   ├── lcd/                        # LCD 配置
│   │   └── lcd_config.hcs
│   ├── camera/                     # 相机配置
│   │   ├── camera_config.hcs
│   │   └── device_info.hcs
│   ├── hdf_test/                   # HDF 测试配置
│   │   ├── emmc_test_config.hcs
│   │   ├── i2c_test_config.hcs
│   │   ├── spi_test_config.hcs
│   │   ├── pwm_test_config.hcs
│   │   ├── adc_test_config.hcs
│   │   ├── gpio_test_config.hcs
│   │   ├── uart_test_config.hcs
│   │   ├── rtc_test_config.hcs
│   │   ├── sdio_test_config.hcs
│   │   ├── watchdog_test_config.hcs
│   │   ├── hdf_config_test.hcs
│   │   ├── hdf_test_manager/
│   │   │   └── device_info.hcs
│   │   └── hdf.hcs
│   └── device_info/              # 设备信息配置
│       └── device_info.hcs
│   └── uhdf/                     # 用户态 HDF 配置
│       ├── camera/                # 相机配置
│       │   ├── hdi_impl/          # HDI 实现
│       │   │   └── camera_host_config.hcs
│       │   ├── pipeline_core/     # 相机管道
│       │   │   ├── config.hcs
│       │   │   ├── ipp_algo_config.hcs
│       │   │   └── params.hcs
│       │   └── device_info.hcs
│       └── media_codec/              # 媒体编解码
│           ├── codec_component_capabilities.hcs
│           ├── codec_adapter_capabilities.hcs
│           ├── codec_hdi1.0_capabilities.hcs
│           ├── image_codec_capabilities.hcs
│           ├── media_codec_capabilities.hcs
│           ├── device_info.hcs
│           ├── hdfs.hcs
│           ├── hdfs.hcs
│           ├── hdfs.hcs
│           └── hdfs.hcs
├── image_conf/                      # 镜像构建配置
├── preinstall-config/                # 预安装应用配置
│   ├── install_list.json            # 预安装应用列表
│   ├── install_list_permissions.json  # 预安装应用权限
│   └── install_list_capability.json  # 预安装应用能力
├── resourceschedule/                  # 资源调度配置
│   ├── cgroup_sched/               # Cgroup 调度配置
│   ├── ressched/                  # 资源调度器配置
│   └── soc_perf/                  # SoC 性能配置
├── security_config/                  # 安全配置
│   ├── critical_reboot_process_list.json   # 关键重启进程列表
│   ├── high_privilege_process_list.json    # 高权限进程列表
│   └── sanitizer_check_list.gni          # Security sanitizer 配置
├── updater_config/                   # OTA 更新配置
│   └── build_cfg.gni               # 更新构建配置
├── window_config/                    # 窗口管理配置
├── config.json                        # 产品配置文件
└── ohos.build                        # 产品构建配置
```

### 标准文件类型

| 文件扩展名 | 类型 | 说明 | 位置 |
|----------|------|------|------|
| *.hcs | HDF 配置源文件 | hdf_config/khdf/ 和 hdf_config/uhdf/ |
| *.json | 配置文件（JSON 格式） | config.json, preinstall-config/, security_config/ |
| *.xml | 音频策略文件（XML 格式） | hals/audio/config/ |
| *.gn | GN 构建文件 | 所有 BUILD.gn 文件 |
| *.gni | GN 配置文件 | product.gni, sanitizer_check_list.gni 等 |
| *.c | C 源代码（仅 IoT 产品） | nearlink_dk_3863/hals/, neptune_iotlink_demo/hals/ |
| *.h | C 头文件（仅 IoT 产品） | nearlink_dk_3863/hals/, neptune_iotlink_demo/hals/ |

## 模块职责详解

### 1. hals/ 目录（硬件抽象层）

**职责**：实现硬件抽象层接口，为 OpenHarmony 系统提供统一硬件访问能力。

**子模块职责**：

#### 1.1 audio/ - 音频 HAL

**职责**：提供音频采集、播放、音效处理和编解码接口。

**文件结构**：
```
hals/audio/
├── BUILD.gn                              # 音频 HAL 构建文件
├── audio_adapter.json                      # 音频适配器配置
├── audio_effect.json                       # 音效配置
├── audio_paths.json                        # 音频路径配置
└── config/                               # 音频策略配置
    ├── arm/
    │   └── audio_policy_config.xml
    └── arm64/
        └── audio_policy_config.xml
```

**配置文件说明**：
- `audio_adapter.json`：定义音频适配器（如 ALSA）路径和配置
- `audio_effect.json`：定义支持的音效类型和参数
- `audio_paths.json`：定义音频设备路径（如设备节点路径）
- `audio_policy_config.xml`：定义音频策略（如音量、路由、焦点）

**证据**：
- 文件：`rk3568/hals/audio/BUILD.gn:14`（ohos_prebuilt_etc targets）
- 文件：`rk3568/hals/audio/product.gni`（包含产品特定配置）

#### 1.2 codec/ - 编解码器 HAL

**职责**：提供音频、视频编解码硬件抽象。

**文件结构**：
```
hals/codec/
├── BUILD.gn      # 编解码器 HAL 构建文件
└── product.gni    # 编解码器产品配置
```

**说明**：标准产品的 codec/ 目录通常仅包含配置文件，实际编解码驱动在 HDF 层。

#### 1.3 utils/ - 实用 HAL（仅 IoT 产品）

**职责**：提供系统参数管理和令牌管理的 HAL 接口。

**文件结构**（仅 neptune_iotlink_demo 和 nearlink_dk_3863）：
```
hals/utils/
├── sys_param/           # 系统参数 HAL
│   ├── hal_sys_param.c
│   ├── hal_sys_param.h
│   ├── vendor.para
│   └── BUILD.gn
└── token/               # 令牌 HAL
    ├── hal_token.c
    ├── hal_token.h
    └── BUILD.gn
```

**HAL 函数说明**（证据）：
- `hal_token.c:20`：`OEMReadToken()` - OEM 读取令牌（stub 实现）
- `hal_token.c:28`：`OEMWriteToken()` - OEM 写入令牌（stub 实现）
- `hal_token.c:36`：`OEMGetAcKey()` - OEM 获取访问密钥（stub 实现）
- `hal_token.c:44`：`OEMGetProdId()` - OEM 获取产品 ID（stub 实现）
- `hal_sys_param.c:21`：`HalGetSerial()` - 获取设备序列号

### 2. hdf_config/ 目录（硬件驱动框架配置）

**职责**：使用 HCS（HDF Configuration Source）语言配置 HDF 驱动服务。

**子目录职责**：

#### 2.1 khdf/ - 内核态 HDF 配置

**职责**：配置内核态 HDF 驱动服务（运行在内核空间）。

**配置文件数量**（以 rk3568 为例）：
- 901 行 device_info.hcs（包含 80+ 个服务定义）
- 100+ 个 .hcs 传感器配置文件

**服务类型**：
- 平台设备（Platform Devices）：GPIO, UART, I2C, SPI, ADC, PWM, Watchdog, RTC
- 传感器（Sensors）：60+ 个传感器配置（加速度、陀螺仪、磁力、光线、距离、温度、湿度、气体、气压）
- 网络设备（Network Devices）：WiFi 芯片配置
- 音频设备（Audio Devices）：DAI, Codec, DMA, DSP
- 显示设备（Display Devices）：LCD, 背光, 输入设备
- 存储设备（Storage Devices）：MMC, SDIO
- 其他设备（Others）：触摸、红外、USB PNP 通知等

**证据**：
- 文件：`rk3568/hdf_config/khdf/device_info.hcs:1`（include "device_info/device_info.hcs"）
- 目录：`rk3568/hdf_config/khdf/sensor/`（包含所有传感器配置）

#### 2.2 uhdf/ - 用户态 HDF 配置

**职责**：配置用户态 HDF 驱动服务（运行在用户空间）。

**配置文件**（以 rk3568 为例）：
- 658 行 device_info.hcs（包含 65+ 个服务定义）
- 媒体编解码配置（media_codec/ 子目录）
- 相机配置（camera/ 子目录）

**服务类型**：
- 音频服务：audio_hdi_service, audio_hdi_usb_service, audio_hdi_a2dp_service, audio_manager_service, effect_model_service
- 相机服务：camera_service, hdi_media_layer_service, distributed_camera_service
- 输入服务：input_service, input_interfaces_service
- 传感器服务：sensor_interface_service, light_interface_service, vibrator_interface_service
- 网络服务：wlan_interface_service, chip_interface_service, wpa_interface_service, hostapd_interface_service
- USB 服务：usb_pnp_manager, usb_interface_service, usb_port_interface_service 等
- 电源服务：power_interface_service, battery_interface_service, thermal_interface_service

**证据**：
- 文件：`rk3568/hdf_config/uhdf/device_info.hcs:27`（module = "sample_driver_service" 等）
- 文件：`rk3568/hdf_config/uhdf/hdfs.hcs`（包含 host 配置）

### 3. security_config/ 目录（安全配置）

**职责**：配置系统安全策略、进程权限和 SELinux 策略。

**文件说明**：

#### 3.1 critical_reboot_process_list.json

**职责**：定义安全关键进程列表，这些进程在系统异常时需要特殊处理。

**进程类型**：
- 系统服务：accesstoken_service, samgr, foundation, appspawn
- 硬件服务：render_service, storage_daemon, hdf_devmgr

**证据**：
- 文件：`rk3568/security_config/critical_reboot_process_list.json:1`（JSON 结构）

#### 3.2 high_privilege_process_list.json

**职责**：定义高权限进程列表，这些进程具有 root/system 权限。

**高权限进程**（证据：`rk3568/security_config/high_privilege_process_list.json`）：
- root 权限：appspawn, cjappspawn, nativespawn, hybridspawn, console, netsysnative, misc, hdcd
- system 权限：render_service, media_service, resource_schedule_service, ueventd

**权限说明**：
- uid/gid：root 或 system
- 进程类型：守护进程、系统服务

#### 3.3 sanitizer_check_list.gni

**职责**：定义 Security sanitizer 配置，为特定模块提供 CFI bypass。

**配置模块**（证据：`rk3568/security_config/sanitizer_check_list.gni`）：
- `socket_permission` - Socket 权限管理
- `power_permission` - 电源权限
- `uri_permission_mgr` - URI 权限管理
- `dlp_permission_service` - DLP 权限服务
- `ohdlp_permission` - OpenHarmony DLP 权限

### 4. preinstall-config/ 目录（预安装应用配置）

**职责**：定义预安装的系统应用及其权限和能力。

**文件说明**：

#### 4.1 install_list.json

**职责**：定义预安装应用列表和安装参数。

**证据**：
- 文件：`wearable/preinstall-config/install_list.json`（331 行，包含 13 个应用配置）
- 应用示例：com.ohos.screenshot, com.ohos.medialibrary.medialibrarydata, com.ohos.callui 等

#### 4.2 install_list_permissions.json

**职责**：定义预安装应用的权限要求。

**权限类型**（证据：`wearable/preinstall-config/install_list_permissions.json`）：
- 媒体权限：READ_MEDIA, WRITE_MEDIA, MEDIA_LOCATION
- 相机权限：CAMERA, MICROPHONE
- 位置权限：LOCATION, LOCATION_IN_BACKGROUND, APPROXIMATELY_LOCATION
- 连接权限：INTERNET, GET_WIFI_INFO, GET_NETWORK_INFO, ACCESS_BLUETOOTH
- 联系人权限：READ_CONTACTS, WRITE_CONTACTS, READ_CALL_LOG, WRITE_CALL_LOG
- 通话权限：SEND_MESSAGES, ANSWER_CALL, RECEIVE_SMS
- 系统权限：GET_INSTALLED_BUNDLE_LIST, GET_BUNDLE_INFO_PRIVILEGED
- 安全权限：ACCESS_PIN_AUTH, ACCESS_BIOMETRIC, ACCESS_UDID

#### 4.3 install_list_capability.json

**职责**：定义预安装应用的能力扩展。

**能力类型**：
- allowAppUsePrivilegeExtension：允许使用特权 API
- singleton：单实例应用
- keepAlive：保持运行
- allowAppDesktopIconHide：可隐藏桌面图标
- allowAppRunWhenDeviceFirstLocked：首次锁屏时可运行
- allowFormVisibleNotify：表单可见通知

### 5. resourceschedule/ 目录（资源调度配置）

**职责**：配置系统资源调度策略，优化性能和功耗。

**子目录职责**：

#### 5.1 cgroup_sched/ - Cgroup 调度配置

**职责**：配置 Linux Cgroup 资源隔离和调度策略。

#### 5.2 ressched/ - 资源调度器配置

**职责**：配置应用资源调度策略（如 CPU 频率限制、应用调度优先级）。

#### 5.3 soc_perf/ - SoC 性能配置

**职责**：配置 SoC 性能调优参数（如 DVFS、Governor 配置）。

### 6. bluetooth/ 目录（蓝牙 HAL）

**职责**：提供蓝牙硬件抽象层实现。

**标准产品**：
- 仅包含配置文件（BUILD.gn）
- 实际 HAL 实现在 OpenHarmony 框架中

**dayu210 特例**：
- ✅ 包含完整源代码实现（`dayu210/bluetooth/src/`）
- ✅ 包含头文件（`dayu210/bluetooth/include/utils/`）

**证据**：
- 文件：`dayu210/bluetooth/BUILD.gn`（包含 source_set）
- 目录：`dayu210/bluetooth/src/`（实际 C++ 源代码）

### 7. image_conf/ 目录（镜像构建配置）

**职责**：配置系统镜像构建参数和文件上下文。

**配置类型**：
- 系统镜像配置（system_image_conf.txt）
- Ramdisk 镜像配置（ramdisk_image_conf.txt）
- 更新镜像配置（updater_ramdisk_image_conf.txt）

### 8. window_config/ 目录（窗口管理配置）

**职责**：配置窗口管理器和显示相关参数。

### 9. etc/ 目录（系统配置文件）

**职责**：存放系统级配置文件。

**子目录**：
- param/：系统参数文件

### 10. demo_foundation/ 目录（仅 rk3568_mini_system）

**职责**：演示 Foundation 进程配置。

**文件说明**：
- `foundation.json`：定义 SystemAbility 配置（SA ID 8501: SoftBus）

**证据**：
- 文件：`rk3568_mini_system/demo_foundation/foundation.json`

## Mini 系统目录结构

### neptune_iotlink_demo 目录结构

```
neptune_iotlink_demo/
├── ble/                        # BLE 源代码
│   └── source/
├── hals/                       # HAL 实现
│   └── utils/
│       ├── sys_param/       # 系统参数
│       └── token/          # 令牌管理
├── hdf_config/                 # HDF 配置
│   ├── BUILD.gn
│   └── hdf.hcs
├── kernel_configs/            # 内核配置
├── config.json                # 产品配置
└── ohos.build                # 构建配置
```

### nearlink_dk_3863 目录结构

```
nearlink_dk_3863/
├── hals/                       # HAL 实现
│   └── utils/
│       ├── sys_param/       # 系统参数
│       └── token/          # 令牌管理
├── ws63_sample/               # 28 个教程示例
│   ├── 00_thread/          # 线程教程
│   ├── 01_timer/           # 定时器教程
│   ├── ...
│   ├── 27_sle_oled/        # SLE OLED 教程
│   └── paho_mqtt/          # Paho MQTT 库
├── config.json                # 产品配置
└── ohos.build                # 构建配置
```

### nearlink_dk_3863_xts 目录结构

```
nearlink_dk_3863_xts/
├── BUILD.gn                  # XTS 测试构建
├── config.json                # 产品配置
└── ohos.build                # 构建配置
```

### rk3568_mini_system 目录结构

```
rk3568_mini_system/
├── bluetooth/                 # 蓝牙配置
├── demo_foundation/           # Foundation 演示
│   └── foundation.json
├── hals/                      # HAL 实现
│   └── audio/
├── hdf_config/              # HDF 配置
│   ├── khdf/
│   └── uhdf/
├── updater_config/          # 更新配置
│   └── build_cfg.gni
├── config.json             # 产品配置
└── ohos.build              # 构建配置
```

**缺失目录**（对比 rk3568）：
- ❌ 无 resourceschedule/
- ❌ 无 security_config/
- ❌ 无 window_config/

## 目录职责总结表

| 目录 | 核心职责 | 文件类型 | 证据路径 |
|------|---------|----------|----------|
| **hals/** | 硬件抽象层实现 | *.c/*.h/*.gn/*.json/*.xml | neptune_iotlink_demo/hals/ |
| **hdf_config/khdf/** | 内核态 HDF 驱动配置 | *.hcs | rk3568/hdf_config/khdf/device_info.hcs:1 |
| **hdf_config/uhdf/** | 用户态 HDF 服务配置 | *.hcs | rk3568/hdf_config/uhdf/device_info.hcs:1 |
| **security_config/** | 安全策略和权限配置 | *.json/*.gni | rk3568/security_config/ |
| **preinstall-config/** | 预安装应用配置 | *.json | wearable/preinstall-config/ |
| **resourceschedule/** | 资源调度配置 | （各类配置文件） | rk3568/resourceschedule/ |
| **bluetooth/** | 蓝牙配置 | BUILD.gn（dayu210 含源码） | dayu210/bluetooth/BUILD.gn |
| **image_conf/** | 镜像构建配置 | *.txt | rk3568/image_conf/ |
| **config.json** | 产品配置 | JSON | {product}/config.json |
| **ohos.build** | 构建配置 | JSON | {product}/ohos.build |

## 配置文件优先级

配置文件按以下优先级被系统读取和应用：

1. **config.json** - 产品主配置
2. **ohos.build** - 产品构建配置
3. **BUILD.gn** - GN 构建入口
4. **hcs 文件** - HDF 驱动配置（按设备树加载）
5. **xml/json 配置** - 模块特定配置（音频策略、权限等）

## 相关跳转

- [HAL 实现说明](03_HAL_Implementation.md)
- [HDF 配置详解](04_HDF_Configuration.md)
- [返回 Wiki 首页](SUMMARY.md)
