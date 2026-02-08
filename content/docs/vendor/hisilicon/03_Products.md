# 产品系列

本文档详细介绍 Hisilicon Vendor 支持的所有开发板和产品形态。

---

## 3.1 产品总览

| 开发板 | 芯片 | 系统类型 | 适用场景 | 配置路径 |
|-------|------|---------|---------|---------|
| hispark_pegasus | Hi3861V100 | LiteOS-M | WiFi IoT | [config.json](hispark_pegasus/config.json) |
| hispark_pegasus_mini_system | Hi3861V100 | LiteOS-M | 最小系统 | [config.json](hispark_pegasus_mini_system/config.json) |
| hispark_aries | Hi3516 | LiteOS | IP Camera | [config.json](hispark_aries/config.json) |
| hispark_taurus | Hi35xx | LiteOS | 标准设备 | [config.json](hispark_taurus/config.json) |
| hispark_taurus_mini_system | Hi35xx | LiteOS | 最小系统 | [config.json](hispark_taurus_mini_system/config.json) |
| hispark_phoenix | Hi3518 | 标准系统 | IP Camera | [config.json](hispark_phoenix/config.json) |
| hispark_taurus_standard | Hi35xx | 标准系统 | 标准系统 | [config.json](hispark_taurus_standard/config.json) |
| hispark_taurus_linux | Hi35xx | Linux | Linux 系统 | [config.json](hispark_taurus_linux/config.json) |
| watchos | - | 标准系统 | 手表产品 | [config.json](watchos/config.json) |

---

## 3.2 LiteOS-M 系列

### 3.2.1 hispark_pegasus（WiFi IoT 开发板）

**芯片**：Hi3861V100

**系统**：LiteOS-M

**特点**：
- WiFi 连接能力
- 丰富的外设接口（I2C、SPI、UART、PWM、GPIO）
- 适用于智能家电、传感器等 IoT 场景

**产品配置**：

```json
{
  "product_name": "wifiiot_hispark_pegasus",    // [证据：config.json:2]
  "type": "mini",                                // [证据：config.json:3]
  "kernel_type": "liteos_m",                     // [证据：config.json:9]
  "kernel_is_prebuilt": true                    // [证据：config.json:10]
}
```

**支持子系统**：

| 子系统 | 组件 | 说明 |
|-------|------|------|
| applications | wifi_iot_sample_app | WiFi IoT 示例应用 |
| iothardware | peripheral | 外设支持 |
| hiviewdfx | hilog_lite, hievent_lite, blackbox_lite, hidumper_lite | 系统调试 |
| security | device_auth, huks | 安全框架 |
| communication | wifi_lite, dsoftbus, wifi_aware | 通信能力 |
| startup | bootstrap_lite, init | 启动框架 |

**Demo 示例**：

- [easy_wifi_demo](./05_Demos.md#easy_wifi_demo)：WiFi STA/AP 模式
- [environment_demo](./05_Demos.md#environment_demo)：环境传感器
- [mqtt_demo](./05_Demos.md#mqtt_demo)：MQTT 通信
- [coap_demo](./05_Demos.md#coap_demo)：CoAP 通信

### 3.2.2 hispark_pegasus_mini_system（最小系统）

**芯片**：Hi3861V100

**系统**：LiteOS-M

**特点**：
- 最小系统配置
- 无额外外设驱动
- 适用于资源受限场景

---

## 3.3 LiteOS 系列

### 3.3.1 hispark_aries（IP Camera 开发板）

**芯片**：Hi3516

**系统**：LiteOS

**特点**：
- 视频编解码能力
- Camera 接口
- 适用于 IP Camera 场景

### 3.3.2 hispark_taurus（标准设备）

**芯片**：Hi35xx

**系统**：LiteOS

**特点**：
- 丰富的计算能力
- 多种外设支持
- 适用于通用智能设备

### 3.3.3 hispark_taurus_mini_system（最小系统）

**芯片**：Hi35xx

**系统**：LiteOS

**特点**：
- 最小系统配置
- 适用于资源受限场景

---

## 3.4 标准系统系列

### 3.4.1 hispark_phoenix（IP Camera）

**芯片**：Hi3518

**系统**：标准系统

**特点**：
- 完整系统服务
- ArkUI 支持
- 分布式能力
- 适用于高端 IP Camera

**产品配置**：

```json
{
  "product_name": "hispark_phoenix",
  "type": "standard",                           // [证据：config.json]
  "enable_ramdisk": true,
  "subsystems": [
    {
      "subsystem": "arkui",
      "components": [ { "component": "ui_lite" } ]
    },
    {
      "subsystem": "security",
      "components": [ { "component": "certificate_manager" } ]
    }
  ]
}
```

### 3.4.2 hispark_taurus_standard（标准系统）

**芯片**：Hi35xx

**系统**：标准系统

**特点**：
- 完整系统框架
- 支持 HDF 驱动
- 适用于标准智能设备

**HDF 配置**：

| 配置模块 | 路径 | 说明 |
|---------|------|------|
| 设备信息 | hdf_config/khdf/device_info/ | device_info.hcs |
| 平台驱动 | hdf_config/khdf/platform/ | I2C/UART/SPI/... |
| WiFi 驱动 | hdf_config/khdf/wifi/ | wlan_platform.hcs, wlan_chip_hi3881.hcs |
| 传感器 | hdf_config/khdf/sensor/ | sensor_config.hcs |
| 音频 | hdf_config/khdf/audio/ | audio_config.hcs |
| 灯光 | hdf_config/khdf/light/ | light_config.hcs |
| 振动 | hdf_config/khdf/vibrator/ | vibrator_config.hcs |
| 输入设备 | hdf_config/khdf/input/ | input_config.hcs |
| LCD | hdf_config/khdf/lcd/ | lcd_config.hcs |
| USB | hdf_config/uhdf/ | usb_ecm_acm.hcs |

### 3.4.3 watchos（手表产品）

**芯片**：-

**系统**：标准系统

**特点**：
- 手表形态产品
- 屏幕显示
- 电源管理

---

## 3.5 Linux 系列

### 3.5.1 hispark_taurus_linux

**芯片**：Hi35xx

**系统**：Linux

**特点**：
- 完整 Linux 用户空间
- 容器化支持
- 适用于 Linux 应用场景

---

## 3.6 产品配置对比

| 特性 | LiteOS-M | LiteOS | 标准系统 | Linux |
|-----|---------|--------|---------|-------|
| 系统类型 | mini | light | standard | linux |
| RAM 要求 | < 64KB | < 1MB | > 128MB | > 256MB |
| MMU 支持 | 无 | 有 | 有 | 有 |
| ArkUI | 不支持 | 部分 | 完全支持 | 完全支持 |
| 典型芯片 | Hi3861V100 | Hi3516 | Hi35xx | Hi35xx |
| 适用设备 | 传感器、智能家电 | 摄像头、门锁 | 手机、平板 | 网关、服务器 |

---

## 3.7 芯片选型指南

### 按资源需求

| 场景 | 推荐芯片 | 说明 |
|-----|---------|------|
| 超低功耗传感器 | Hi3861V100 | RAM < 64KB |
| 智能门锁/摄像头 | Hi3516 | RAM < 1MB |
| 智能音箱/平板 | Hi35xx (标准系统) | RAM > 128MB |
| 网关/服务器 | Hi35xx (Linux) | 完整 Linux |

### 按连接需求

| 场景 | 推荐芯片 | 说明 |
|-----|---------|------|
| WiFi IoT | Hi3861V100 | 内置 WiFi |
| 以太网 | Hi3516/Hi35xx | 以太网 PHY |
| 视频相关 | Hi3516/Hi3518 | 视频编解码 |

---

## 3.8 相关文档

- [目录结构](./02_Directory_Structure.md)
- [配置体系](./04_Configuration.md)
- [Demo 示例](./05_Demos.md)
- [构建指南](./06_Build.md)
