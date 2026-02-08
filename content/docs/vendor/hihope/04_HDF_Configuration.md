# HDF（硬件驱动框架）配置详解

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库中的 HDF（硬件驱动框架）配置文件、服务定义和配置语法
- **适用范围**：vendor_hihope 仓库中的 hdf_config/ 目录
- **最后更新**：2025-02-06
- **关键结论**：
  - HDF 使用 HCS（HDF Configuration Source）语言配置驱动服务
  - 包含 UHDF（用户态）和 KHDF（内核态）两种驱动类型
  - rk3568 产品包含最完整的 HDF 配置（1600+ 行）

## HDF 概述

### HDF 定义

HDF（Hardware Driver Foundation，硬件驱动框架）是 OpenHarmony 的统一驱动架构。

**核心特点**：
- **分层架构**：用户态（UHDF）和内核态（KHDF）
- **统一配置**：使用 HCS 语言描述驱动配置
- **动态加载**：支持驱动按需加载和卸载
- **服务管理**：通过系统服务管理器（System Ability Manager）管理驱动服务
- **HDI 接口**：定义标准化的硬件驱动接口（HDF Driver Interface）

**在 vendor 仓库中的位置**：
- 标准产品：`{product}/hdf_config/` 目录
  - `khdf/` - 内核态驱动配置
  - `uhdf/` - 用户态驱动配置

## HCS 配置语言

### HCS 语法基础

HCS（HDF Configuration Source）是 HDF 的配置语言，采用类似 JSON 的树状结构。

#### 基本语法

```hcs
root {
    module = "module_name";           // 模块名
    attr_name = "attribute_value";     // 属性
    config_name {                    // 配置块
        property = "value";            // 属性
    }
}
```

#### 关键关键字

| 关键字 | 说明 | 示例 |
|---------|------|------|
| `module` | 模块名，用于标识驱动模块 | `"module": "rockchip,rk3568_chip"` |
| `policy` | 策略值，0=内核，1=内核+用户，2=用户服务 | `"policy": 2` |
| `priority` | 加载优先级，数字越小优先级越高 | `"priority": 40` |
| `preload` | 预加载策略，0=总是，1=延迟，2=从不 | `"preload": 0` |
| `permission` | 设备节点权限，八进制表示 | `"permission": 0644` |
| `serviceName` | 服务名 | `"serviceName": "HDF_PLATFORM_UART_0"` |
| `moduleName` | 模块名 | `"moduleName": "HDF_PLATFORM_UART"` |
| `deviceMatchAttr` | 设备匹配属性 | `"deviceMatchAttr": "rockchip_rk3568_uart_0"` |

### UHDF（用户态 HDF）配置

**位置**：`{product}/hdf_config/uhdf/`

#### UHDF device_info.hcs 结构（以 rk3568 为例）

**证据**：`rk3568/hdf_config/uhdf/device_info.hcs:27`

```hcs
root {
    device_info {
        match_attr = "hdf_manager";
        host :: host {
            hostName = "platform_host";
            priority = 50;

            // 示例驱动服务配置
            sample_driver_service :: device {
                policy = 2;
                priority = 40;
                preload = 0;
                permission = 0644;
                moduleName = "HDF_SAMPLE_DRIVER";
                serviceName = "HDF_SAMPLE_SERVICE";
                deviceMatchAttr = "rockchip_rk3568_sample";
            }

            audio_hdi_service :: device {
                policy = 2;
                priority = 266;
                preload = 0;
                permission = 0666;
                moduleName = "HDI_AUDIO_PRIMARY";
                serviceName = "HDF_AUDIO_PRIMARY";
            }

            camera_service :: device {
                policy = 2;
                priority = 328;
                preload = 0;
                permission = 0666;
                moduleName = "CAMERA_HOST_SERVICE_1.0";
                serviceName = "CAMERA_HOST_SERVICE";
            }

            // ... 更多服务定义（658 行）
        }
    }
}
```

**服务分类**（证据：`rk3568/hdf_config/uhdf/device_info.hcs`）：

| 服务类型 | 服务名示例 | Host | 优先级范围 | 说明 |
|---------|------------|------|-----------|------|
| **音频服务** | audio_hdi_service, audio_hdi_usb_service, audio_hdi_a2dp_service 等 | audio_host | 266-314 | 音频采集、播放、A2DP、音频 PnP |
| **相机服务** | camera_service, hdi_media_layer_service, distributed_camera_service | camera_host | 328-509 | 相机采集、媒体层、分布式相机 |
| **输入服务** | input_service, input_interfaces_service, input_hotplug_service | input_user_host | 364-380 | 输入设备、输入热插拔 |
| **传感器服务** | sensor_interface_service, light_interface_service, vibrator_interface_service | sensor_host/light_host/vibrator_host | 422-446 | 传感器、光线、振动器 |
| **网络服务** | wlan_interface_service, chip_interface_service, wpa_interface_service, hostapd_interface_service | wifi_host/wpa_host/hostapd_host | 211-253 | WiFi 主机、芯片、WPA、HostAPD |
| **USB 服务** | usb_pnp_manager, usb_interface_service, usb_port_interface_service 等 | usb_host | 79-166 | USB PNP 通知、USB 接口、USB 端口 |
| **电源服务** | power_interface_service, battery_interface_service, thermal_interface_service | power_host | 181-197 | 电源管理、电池、温度 |
| **显示服务** | display_composer_service, allocator_service | composer_host/allocator_host | 397-409 | 显示合成、内存分配 |
| **编解码服务** | codec_hdi_omx_service, codec_component_manager_service, codec_hdi_service, codec_image_service | codec_host | 460-487 | OMX 编解码、组件管理、HDI 服务、图像编解码 |
| **认证服务** | face_auth_interface_service, pin_auth_interface_service, user_auth_interface_service, fingerprint_auth_interface_service | face_auth_host/pin_auth_host/user_auth_host/fingerprint_auth_host | 545-590 | 人脸、PIN、用户、指纹认证 |
| **定位服务** | gnss_interface_service, agnss_interface_service, geofence_interface_service, location_host | 545-624 | GNSS/AGNSS、地理围栏 |
| **音频流服务** | daudio_primary_service, daudio_ext_service | daudio_host | 523-530 | 分布式音频主/扩展 |
| **分区服务** | partition_slot_service | partitionslot_host | 639 | 分区槽管理 |
| **语音引擎服务** | intell_voice_trigger_manager_service, intell_voice_engine_manager_service | intell_voice_host | 652-652 | 智能语音触发和引擎 |
| **HDF 内核服务** | hdf_kevent, hdf_input_host, hdf_wifi, hdf_disp 等 | base_host | - | 内核事件、输入、WiFi、显示 |

#### UHDF Camera 配置

**位置**：`hdf_config/uhdf/camera/`

**结构**（证据：`rk3568/hdf_config/uhdf/camera/`）：

```
camera/
├── hdi_impl/                     # HDI 实现
│   └── camera_host_config.hcs
├── pipeline_core/                 # 相机管道
│   ├── config.hcs
│   ├── ipp_algo_config.hcs
│   ├── params.hcs
│   └── device_info.hcs
└── device_info.hcs
```

**配置说明**：
- `hdi_impl/`：HDI（HDF Driver Interface）实现配置
- `pipeline_core/`：相机处理管道配置，包含 IPP 算法和参数配置

### KHDF（内核态 HDF）配置

**位置**：`{product}/hdf_config/khdf/`

#### KHDF device_info.hcs 结构（以 rk3568 为例）

**证据**：`rk3568/hdf_config/khdf/device_info/device_info.hcs:1`

```hcs
#include "device_info/device_info.hcs"

root {
    module = "rockchip,rk3568_chip";

    platform :: host {
        hostName = "platform_host";
        priority = 50;

        // 平台设备
        HDF_PLATFORM_GPIO_MANAGER :: device { ... }
        HDF_PLATFORM_WATCHDOG_0 :: device { ... }
        HDF_PLATFORM_RTC :: device { ... }
        HDF_PLATFORM_UART_0 :: device { ... }
        HDF_PLATFORM_I2C_MANAGER :: device { ... }
        HDF_PLATFORM_ADC_MANAGER :: device { ... }
        HDF_PLATFORM_SPI_0 :: device { ... }
        HDF_PLATFORM_MMC_2 :: device { ... }

        // 显示设备
        hdf_disp :: device { ... }
        hdf_bl :: device { ... }

        // 输入设备
        hdf_input_host :: device { ... }
        HDF_INPUT_MANAGER :: device { ... }
        hdf_touch_gt911_service :: device { ... }
        hdf_input_event1 :: device { ... }
        hdf_input_event2 :: device { ... }

        // 传感器管理器
        hdf_sensor_manager_ap :: device { ... }

        // 网络设备
        hdfwifi :: device { ... }
        ap6275s :: device { ... }

        // 相机
        hdfcamera :: device { ... }
        hdfcamera0 :: device { ... }

        // 音频
        dai_service :: device { ... }
        hdmi_dai_service :: device { ... }
        codec_service_0 :: device { ... }
        codec_service_1 :: device { ... }
        audio_usb_service_0 :: device { ... }
        dsp_service_0 :: device { ... }
        hdmi_dma_service_0 :: device { ... }
        usb_dma_service_0 :: device { ... }

        // 振动器
        hdf_misc_vibrator :: device { ... }
        hdf_misc_linear_vibrator :: device { ... }
        hdf_misc_drv2605l_vibrator :: device { ... }

        // 光线
        hdf_light :: device { ... }

        // 其他
        hdf_sensor_als :: device { ... }
        hdf_sensor_proximity :: device { ... }
        hdf_sensor_magnetic :: device { ... }
        hdf_sensor_temperature :: device { ... }
        hdf_sensor_humidity :: device { ... }
        hdf_usb_pnp_notify_service :: device { ... }
        hdf_usb_net_service :: device { ... }
        hdf_audio_codec_primary_dev0 :: device { ... }
        hdf_audio_codec_hdmi_dev0 :: device { ... }
        hdf_audio_codec_usb_dev0 :: device { ... }
        hdf_audio_render :: device { ... }
        hdf_audio_capture :: device { ... }
        hdf_audio_control :: device { ... }
        hdf_misc_drv2605l_vibrator :: device { ... }
    }
}
```

#### 平台设备配置

**证据**：`rk3568/hdf_config/khdf/platform/`（901 行）

**GPIO 配置**（证据：`rk3568/hdf_config/khdf/platform/i2c_config.hcs:1`）：
```hcs
HDF_PLATFORM_GPIO_MANAGER :: device {
    policy = 0;              // 内核态
    priority = 10;
    permission = 0666;
    moduleName = "HDF_PLATFORM_GPIO_MANAGER";
    serviceName = "HDF_PLATFORM_GPIO_MANAGER";
}
```

**UART 配置**：
- `HDF_PLATFORM_UART_0,1,3` - 3 个 UART 控制器
- 配置参数：波特率、数据位、停止位等

**I2C 配置**：
- `HDF_PLATFORM_I2C_MANAGER` - I2C 管理器
- 配置参数：总线速度、设备地址

**SPI 配置**：
- `HDF_PLATFORM_SPI_0,1,2,3` - 4 个 SPI 控制器
- 配置参数：模式、频率、数据位宽

**ADC 配置**：
- `HDF_PLATFORM_ADC_MANAGER` - ADC 管理器
- 配置参数：采样率、分辨率

**PWM 配置**：
- `HDF_PLATFORM_PWM_0,1,2,3,4` - 5 个 PWM 控制器
- 配置参数：频率、占空比

**Watchdog 配置**：
- `HDF_PLATFORM_WATCHDOG_0` - 看门狗
- 配置参数：超时时间

**RTC 配置**：
- `HDF_PLATFORM_RTC` - 实时时钟
- 配置参数：初始时间设置

#### 传感器配置

**传感器管理器**（证据：`rk3568/hdf_config/khdf/sensor/sensor_manager_ap.hcs:1`）：
```hcs
HDF_SENSOR_MGR_AP :: device {
    policy = 0;
    priority = 426;
    permission = 0666;
    moduleName = "HDF_SENSOR_MGR_AP";
    serviceName = "HDF_SENSOR_MGR_AP";
}
```

**具体传感器配置**（60+ 个传感器配置）：

| 传感器类型 | 配置名 | 芯片型号示例 | 说明 |
|----------|---------|-------------|------|
| **加速度** | HDF_SENSOR_ACCEL_MXC6655XA, HDF_SENSOR_ACCEL_BMI270 | mxc6655xa, bmi270 | 3 轴加速度测量 |
| **陀螺仪** | HDF_SENSOR_GYRO, HDF_SENSOR_GYRO_BMI270 | bmi270 | 角速度测量 |
| **磁力** | HDF_SENSOR_MAGNETIC, HDF_SENSOR_MAGNETIC_LSM303 | lsm303 | 磁场强度和方向测量 |
| **光线** | HDF_SENSOR_ALS, HDF_SENSOR_ALS_BH1745 | bh1745 | 环境光强度测量 |
| **距离** | HDF_SENSOR_PROXIMITY, HDF_SENSOR_PROXIMITY_APDS9960 | apds9960 | 接近距离测量 |
| **温度** | HDF_SENSOR_TEMPERATURE, HDF_SENSOR_TEMPERATURE_AHT20, HDF_SENSOR_TEMPERATURE_SHT30 | aht20, sht30 | 环境温度测量 |
| **湿度** | HDF_SENSOR_HUMIDITY, HDF_SENSOR_HUMIDITY_AHT20, HDF_SENSOR_HUMIDITY_SHT30 | aht20, sht30 | 环境湿度测量 |
| **气体** | HDF_SENSOR_GAS, HDF_SENSOR_GAS_BME688 | bme688 | 空气质量测量 |
| **气压** | HDF_SENSOR_BAROMETER, HDF_SENSOR_BAROMETER_BMP581 | bmp581 | 大气压测量 |

#### 显示配置

**证据**：`rk3568/hdf_config/khdf/display/`（253 行）**

```hcs
HDF_DISP :: device {
    policy = 0;
    priority = 253;
    moduleName = "HDF_DISP";
    serviceName = "HDF_DISP";
}
```

**背光配置**：
```hcs
HDF_BL :: device {
    policy = 0;
    priority = 306;
    moduleName = "HDF_BL";
    serviceName = "HDF_BL";
}
```

#### 网络配置

**WiFi 配置**（证据：`rk3568/hdf_config/khdf/network/`（389 行））：

```hcs
HDF_WIFI :: device {
    policy = 0;
    priority = 380;
    moduleName = "HDF_WIFI";
    serviceName = "HDF_WIFI";
}

HDF_WLAN_CHIPS_AP6275S :: device {
    policy = 0;
    priority = 389;
    moduleName = "HDF_WLAN_CHIPS_AP6275S";
    serviceName = "HDF_WLAN_CHIPS_AP6275S";
}
```

#### 音频配置

**证据**：`rk3568/hdf_config/khdf/audio/`（120 行）**

**DAI 配置**：
```hcs
DAI_RK3568 :: device {
    policy = 0;
    priority = 699;
    moduleName = "DAI_RK3568";
    serviceName = "HDF_AUDIO_DAI";
}
```

**Codec 配置**：
```hcs
CODEC_RK809 :: device {
    policy = 0;
    priority = 719;
    moduleName = "CODEC_RK809";
    serviceName = "HDF_AUDIO_CODEC";
}

HDMI_CODEC :: device {
    policy = 0;
    priority = 728;
    moduleName = "AUDIO_HDMI_CODEC";
    serviceName = "HDF_AUDIO_CODEC";
}

AUDIO_USB_CODEC :: device {
    policy = 0;
    priority = 737;
    moduleName = "AUDIO_USB_CODEC";
    serviceName = "HDF_AUDIO_CODEC";
}
```

**DSP 配置**：
```hcs
DSP_RK3568 :: device {
    policy = 0;
    priority = 747;
    moduleName = "DSP_RK3568";
    serviceName = "HDF_AUDIO_DSP";
}
```

**音频流配置**：
```hcs
HDF_AUDIO_STREAM :: device {
    policy = 0;
    priority = 824;
    moduleName = "HDF_AUDIO_STREAM";
    serviceName = "HDF_AUDIO_STREAM";
}

HDF_AUDIO_CAPTURE :: device {
    policy = 0;
    priority = 834;
    moduleName = "HDF_AUDIO_CAPTURE";
    serviceName = "HDF_AUDIO_CAPTURE";
}

HDF_AUDIO_CONTROL :: device {
    policy = 0;
    priority = 844;
    moduleName = "HDF_AUDIO_CONTROL";
    serviceName = "HDF_AUDIO_CONTROL";
}
```

#### 振动器配置

**证据**：`rk3568/hdf_config/khdf/vibrator/`（35 行）**

```hcs
HDF_VIBRATOR :: device {
    policy = 0;
    priority = 858;
    moduleName = "HDF_VIBRATOR";
    serviceName = "HDF_VIBRATOR";
}

HDF_LINEAR_VIBRATOR :: device {
    policy = 0;
    priority = 869;
    moduleName = "HDF_LINEAR_VIBRATOR";
    serviceName = "HDF_LINEAR_VIBRATOR";
}

HDF_DRV2605L_VIBRATOR :: device {
    policy = 0;
    priority = 880;
    moduleName = "HDF_DRV2605L_VIBRATOR";
    serviceName = "HDF_DRV2605L_VIBRATOR";
}
```

#### 光线配置

**证据**：`rk3568/hdf_config/khdf/light/`（10 行）**

```hcs
HDF_LIGHT :: device {
    policy = 0;
    priority = 894;
    moduleName = "HDF_LIGHT";
    serviceName = "HDF_LIGHT";
}
```

#### 输入配置

**输入管理器**（证据：`rk3568/hdf_config/khdf/input/input_config.hcs:1`）：
```hcs
HDF_INPUT_MANAGER :: device {
    policy = 0;
    priority = 321;
    moduleName = "HDF_INPUT_MANAGER";
    serviceName = "HDF_INPUT_MANAGER";
}
```

**触摸配置**：
```hcs
HDF_TOUCH_GT911 :: device {
    policy = 0;
    priority = 344;
    moduleName = "HDF_TOUCH_GT911";
    serviceName = "HDF_TOUCH";
}
```

**红外输入**：
```hcs
HDF_INFRARED :: device {
    policy = 0;
    priority = 364;
    moduleName = "HDF_INFRARED";
    serviceName = "HDF_INFRARED";
}
```

#### 存储配置

**证据**：`rk3568/hdf_config/khdf/storage/`（171 行）**

**eMMC 配置**：
```hcs
HDF_PLATFORM_MMC_2 :: device {
    policy = 0;
    priority = 171;
    moduleName = "HDF_PLATFORM_SDIO";
    serviceName = "HDF_PLATFORM_SDIO";
}

HDF_PLATFORM_EMMC :: device {
    policy = 0;
    priority = 181;
    moduleName = "HDF_PLATFORM_EMMC";
    serviceName = "HDF_PLATFORM_EMMC";
}
```

### HDF 配置说明

#### 1. Policy 值

| Policy 值 | 含义 | 典型使用场景 |
|----------|------|-------------|
| `0` | 内核态，仅在内核态运行 | 平台设备、内核驱动 |
| `1` | 内核态 + 用户态，可同时访问 | 需要内核和用户交互的驱动 |
| `2` | 用户态服务 | 用户态 HDF 服务 |

#### 2. Priority 值

| 优先级范围 | 典型用途 | 示例服务 |
|----------|---------|---------|
| `10-50` | 平台基础设备 | GPIO, UART, I2C, SPI, RTC, Watchdog |
| `51-100` | 传感器管理 | 传感器管理器 |
| `100-200` | 输入设备 | 触摸、红外输入 |
| `200-300` | 显示设备 | LCD 显示、背光 |
| `260-314` | 音频服务 | 音频采集、A2DP、音频 PnP |
| `328-509` | 相机服务 | 相机采集、媒体层 |
| `523-652` | 分布式音频、认证服务 | 音频扩展、认证接口 |
| `639-894` | 其他设备 | 分区管理、振动器、光线 |

#### 3. Permission 值

| 权限值 | 八进制 | 含义 | 典型用途 |
|---------|-------|------|
| `0644` | rw-r--r-- | 标准设备，可读写 |
| `0660` | rw-rw-r-- | 音频设备，用户可读写 |
| `0666` | rw-rw-rw- | USB 设备，用户可全面访问 |
| `0640` | rw-rw---- | 受限设备，用户可读写，其他只读 |
| `0644` | rw-r--r-- | 输入设备，可读写 |

#### 4. Device Match Attr

**作用**：用于内核设备树匹配，确保驱动绑定到正确的硬件。

**示例**：`rockchip_rk3568_uart_0` - 匹配 RK3568 的 UART 0 控制器

## HDF 服务管理

### 服务启动顺序

1. 系统启动
2. DevMgr 启动（`hdf_devmgr`）
3. 加载 HDF 配置文件（.hcs）
4. 按优先级加载驱动（priority）
5. 服务注册到 System Ability Manager

### 服务加载命令

```bash
# 查看 HDF 服务列表
hdc shell
hdf -h

# 查看服务状态
hdc shell hilog -T HDF
```

### HDF 配置验证

#### 1. 语法验证

使用 HDF 配置编译工具验证 HCS 文件：

```bash
# HDF 配置验证（在构建时自动进行）
hcs -o device_info.hcs
```

#### 2. 逻辑验证

检查以下内容：
- 服务名唯一性
- priority 值合理性
- policy 值正确性
- permission 值符合要求

### 常见配置问题

| 问题 | 可能原因 | 解决方法 |
|------|---------|---------|
| 服务未加载 | .hcs 文件路径错误或权限错误 | 检查 `device_info.hcs` 中的 include 路径 |
| 服务加载失败 | 驱动 .so 文件缺失 | 检查 `BUILD.gn` 和驱动构建配置 |
| 权限被拒绝 | permission 值过高或过低 | 调整 permission 值 |
| 设备节点不存在 | deviceMatchAttr 不匹配 | 检查设备树和匹配属性 |
| 多个服务冲突 | serviceName 重复 | 确保 serviceName 唯一 |

## HDF 配置开发指南

### 1. 添加新驱动配置

**步骤**：

1. **确定驱动类型**：内核态（KHDF）还是用户态（UHDF）
2. **选择 Host 名称**：
   - 内核态：platform_host, display_host, audio_host 等
   - 用户态：sample_host, audio_host, camera_host 等
3. **编写 HCS 配置**：
   ```hcs
   {service_name} :: device {
       policy = 2;
       priority = 50;
       permission = 0666;
       moduleName = "YOUR_MODULE";
       serviceName = "YOUR_SERVICE";
       deviceMatchAttr = "chip_specific_attr";
   }
   ```
4. **注册到 device_info.hcs**：
   ```hcs
   root {
       module = "rockchip,rk3568_chip";
       host :: host {
           hostName = "{your_host}";
           {service_name} :: device { ... }
       }
   }
   ```
5. **实现驱动代码**：
   - 使用 HDF 框架 API
   - 实现 HDF 驱动入口函数
   - 实现 HDI 接口（如需要）
6. **配置构建**：
   ```gn
   ohos_shared_library("your_driver") {
     sources = ["your_driver.c"]
     external_deps = ["hdf_core:libhdf"]
     subsystem_name = "product_{product}"
     part_name = "product_{product}"
   }
   ```

### 2. 配置优先级策略

| 驱动类型 | 推荐优先级 | 原因 |
|---------|-----------|------|
| **平台基础设备** | 10-50 | 必须早期初始化，其他服务依赖 |
| **传感器** | 51-100 | 在平台设备之后，音频/显示之前 |
| **输入设备** | 100-200 | 需要输入服务管理器已加载 |
| **显示设备** | 200-300 | 在传感器之后 |
| **音频** | 260-314 | 稍后加载，等待其他音频组件 |
| **相机** | 328-509 | 晚加载，需要多个依赖就绪 |
| **系统服务** | 523-652 | 最后加载，作为系统服务 |

### 3. Mini 系统 HDF 配置

对于 neptune_iotlink_demo 等系统：

**配置文件**：`neptune_iotlink_demo/hdf_config/hdf.hcs`（证据：`neptune_iotlink_demo/hdf_config/BUILD.gn:8`）

```hcs
root {
    module = "winnermicro,neptune100";
}
```

**特点**：
- ✅ 非常简洁（仅 29 行）
- ⚠️ 主要为内核驱动配置
- ⚠️ 不包含用户态 HDF 服务

## 相关跳转

- [目录结构详解](02_Directory_Structure.md)
- [HAL 实现说明](03_HAL_Implementation.md)
- [返回 Wiki 首页](SUMMARY.md)
