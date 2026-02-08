# 传感器 Demo

本文档详细说明 environment_demo 的传感器使用。

---

## 5.b.1 Demo 概述

**位置**：`hispark_pegasus/demo/environment_demo/`

**功能**：环境传感器检测，支持 MQ-2 烟雾传感器、光照传感器、温湿度传感器等。

---

## 5.b.2 支持的传感器

| 传感器 | 功能 | 头文件 |
|-------|------|-------|
| MQ-2 | 烟雾/可燃气体检测 | app_demo_mq2.h |
| GL5537-1 | 光照强度检测 | app_demo_gl5537_1.h |
| AHT20 | 温湿度检测 | app_demo_aht20.h |
| I2C OLED | 显示输出 | app_demo_i2c_oled.h |

---

## 5.b.3 文件结构

```
environment_demo/
├── app_demo_environment.c       // 主程序
├── app_demo_environment.h      // 头文件
├── app_demo_mq2.c               // MQ-2 传感器
├── app_demo_mq2.h
├── app_demo_gl5537_1.c          // 光照传感器
├── app_demo_gl5537_1.h
├── app_demo_aht20.c             // 温湿度传感器
├── app_demo_aht20.h
├── app_demo_config.c            // 配置
├── app_demo_config.h
├── app_demo_multi_sample.c      // 多路采样
├── app_demo_multi_sample.h
├── iot_adc.h                    // ADC 接口
├── iot_gpio_ex.h                // GPIO 扩展
├── hal_iot_adc.c                // ADC HAL
├── hal_iot_gpio_ex.c            // GPIO HAL
└── task_start.c                 // 任务入口
```

---

## 5.b.4 接口说明

### 5.b.4.1 ADC 接口

**iot_adc.h**

```c
// 读取 ADC 值
unsigned int AdcRead(unsigned int channel, unsigned short *data);
```

### 5.b.4.2 GPIO 扩展接口

**iot_gpio_ex.h**

```c
// 设置 GPIO 方向
unsigned int GpioSetDir(unsigned int port, unsigned int dir);

// 读取 GPIO
unsigned int GpioRead(unsigned int port, unsigned int *val);

// 写入 GPIO
unsigned int GpioWrite(unsigned int port, unsigned int val);
```

---

## 5.b.5 相关文档

- [Demo 示例](./05_Demos.md)
- [配置体系](../04_Configuration.md)
