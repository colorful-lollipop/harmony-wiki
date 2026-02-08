# 编译产物说明

本文档描述 `device_soc_hisilicon` 仓库的编译产物清单、输出路径和运行时加载关系。

## 产物总览

| 产物类型 | 说明 | 格式 |
|---------|------|------|
| SDK 库文件 | HAL/HDI 实现库 | `.so`, `.a` |
| 可执行文件 | 应用程序、工具 | `.elf`, `.bin` |
| 内核模块 | 驱动模块 | `.ko` |
| 启动镜像 | Bootloader 镜像 | `.bin` |
| 头文件 | SDK 接口定义 | `.h` |

## HAL 产物

### 显示 HDI

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_display.z.so` | `common/hal/display/source/display_device/libs/` | 显示 HDI 库 |

### 媒体 HDI

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_audio.z.so` | `common/hal/media/audio/hi3751v350/linux_standard/libs/` | 音频 HDI |
| `libhdi_camera.so` | `common/hal/media/camera/[版本]/libs/` | 相机 HDI |
| `libhdi_videodisplayer.so` | `common/hal/media/videodisplay/[版本]/libs/` | 视频显示 HDI |

### 多媒体库

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_media.so` | `common/hal/multimedia/[版本]/libs/` | 多媒体基础库 |

### AI HDI

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_ai.z.so` | `common/hal/ai/libs/` | AI 推理库 |

## SDK 产物

### Hi3861V100 SDK

| 产物 | 路径 | 说明 |
|------|------|------|
| `wifiiot_sdk` | `hi3861v100/sdk_liteos/out/` | WiFi IoT SDK 可执行文件 |
| `libwifi_lite.a` | `hi3861v100/sdk_liteos/out/` | WiFi 静态库 |
| `libhal.a` | `hi3861v100/hi3861_adapter/out/` | HAL 静态库 |
| `libkal.a` | `hi3861v100/hi3861_adapter/out/` | KAL 静态库 |

### Hi3516DV300 SDK

| 产物 | 路径 | 说明 |
|------|------|------|
| `libmpp.a` | `hi3516dv300/sdk_liteos/mpp/out/` | MPP 媒体库 |
| `ko/` | `hi3516dv300/sdk_liteos/mpp/ko/` | 内核模块目录 |

### 第三方库

| 产物 | 路径 | 说明 |
|------|------|------|
| `libmbedcrypto.so` | `third_party/mbedtls/out/` | mbedtls 加密库 |
| `libmbedtls.so` | `third_party/mbedtls/out/` | mbedtls 库 |
| `libsecurec.so` | `sdk_linux/out/lib/` | 安全内存库 |

## 启动镜像

### Hi3861V100 启动镜像

| 产物 | 路径模式 | 说明 |
|------|---------|------|
| `loaderboot.bin` | `sdk_liteos/out/` | 二级加载器 |
| `flashboot.bin` | `sdk_liteos/out/` | Flash 启动加载器 |
| `u-boot.bin` | `uboot/` | U-Boot (部分芯片) |

### 镜像结构

```
┌─────────────────────────────────────┐
│           Flash 分区布局             │
├─────────────────────────────────────┤
│  Loaderboot  (64KB)                 │
├─────────────────────────────────────┤
│  Flashboot (256KB)                 │
├─────────────────────────────────────┤
│  Kernel    (2~4MB)                 │
├─────────────────────────────────────┤
│  Rootfs    (4~8MB)                 │
├─────────────────────────────────────┤
│  Userdata  (可调整)                 │
└─────────────────────────────────────┘
```

## 头文件产物

### SDK 头文件

| 路径 | 说明 |
|------|------|
| `sdk_liteos/include/` | LiteOS SDK 头文件 |
| `sdk_linux/include/` | Linux SDK 头文件 |
| `adapter/hals/*/inc/` | HAL 头文件 |
| `adapter/kal/*/` | KAL 头文件 |

### 关键头文件

| 头文件 | 功能 |
|--------|------|
| `hi_gpio.h` | GPIO 接口 |
| `hi_uart.h` | UART 接口 |
| `hi_i2c.h` | I2C 接口 |
| `hi_spi.h` | SPI 接口 |
| `hi_adc.h` | ADC 接口 |
| `hi_pwm.h` | PWM 接口 |
| `hi_flash_base.h` | Flash 接口 |
| `hi_cipher.h` | 加密接口 |
| `hi_task.h` | 任务接口 |

## 运行时加载关系

### HDI 库加载

```
Framework (foundation/)
    │
    ├── dlopen ────────────────┐
    │                          │
    ▼                          ▼
libhdi_media.so ────────── libhdi_display.so
    │                          │
    ├── depends on ────────────┤
    │                          │
    ▼                          ▼
libhilog.so             libhdi_drm.so
```

### SDK 库加载

```
Application
    │
    ├── static link ───────────┐
    │                          │
    ▼                          ▼
libhal.a (HAL)            libkal.a (KAL)
    │                          │
    ├── depends on ────────────┤
    │                          │
    ▼                          ▼
    └──────────────────────────┘
              │
              ▼
      Platform Drivers (芯片 SDK)
              │
              ▼
      芯片寄存器操作
```

## 安装路径

### 标准系统安装

| 组件 | 安装路径 | 说明 |
|------|---------|------|
| HDI 库 | `/system/lib/` | 系统共享库 |
| HAL 库 | `/vendor/lib/` | 厂商库 |
| 驱动 | `/vendor/lib/modules/` | 内核模块 |

### 轻量系统安装

| 组件 | 安装路径 | 说明 |
|------|---------|------|
| SDK | `/bin/` | 可执行文件 |
| 库 | `/lib/` | 静态库 |
| 配置 | `/etc/` | 配置文件 |

## 版本兼容性

### SDK 版本

| 芯片 | SDK 版本 | OpenHarmony 版本 |
|------|---------|------------------|
| Hi3861V100 | LiteOS SDK | 轻量系统 |
| Hi3516DV300 | LiteOS SDK / Linux SDK | 小型/标准系统 |
| Hi3751V350 | Linux SDK | 标准系统 |

### 产物版本管理

HDI 库使用版本后缀区分：
- `.z.so`: 版本化共享库
- `.so`: 非版本化共享库

## 相关文档

- 构建系统: [07_Build_System.md](07_Build_System.md)
- SDK 架构: [06_SDK_Architecture.md](06_SDK_Architecture.md)
- 安全评审: [09_Security_Review.md](09_Security_Review.md)
