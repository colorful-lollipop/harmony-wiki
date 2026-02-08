# 编译产物

本文档说明 `drivers/peripheral` 各模块的编译产物、安装路径和运行时加载关系。

## 7.1 编译产物概览

### 7.1.1 产物类型

| 产物类型 | 说明 | 文件扩展名 |
|----------|------|------------|
| **动态库** | HDI 服务实现 | `.z.so` |
| **静态库** | HAL 实现（可选） | `.a` |
| **头文件** | 接口定义 | `.h` |
| **配置文件** | 设备描述 | `.xml`, `.json` |

---

## 7.2 各模块编译产物

### 7.2.1 Audio 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_audio.z.so` | `out/{product}/drivers/peripheral/audio/` | 音频 HDI 服务动态库 |
| `audio_interface` | `interfaces/include/` | 音频接口头文件包 |
| `audio_interface_v2_0` | `interfaces/2.0/include/` | 音频 2.0 接口头文件 |

**运行时加载**:
```
System Service (AudioService)
        │
        ▼ dlopen()
libhdi_audio.z.so
        │
        ▼ depends on
libhdi_audio_adapter.z.so (drivers_adapter)
        │
        ▼ ioctl
Linux Kernel Audio Driver
```

---

### 7.2.2 Input 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_input.z.so` | `out/{product}/drivers/peripheral/input/` | 输入 HDI 服务动态库 |
| `input_interface` | `interfaces/include/` | 输入接口头文件包 |

**运行时加载**:
```
System Service (InputService)
        │
        ▼ dlopen()
libhdi_input.z.so
        │
        ▼ depends on
Linux Kernel Input Driver (/dev/input/*)
```

---

### 7.2.3 Sensor 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_sensor.z.so` | `out/{product}/drivers/peripheral/sensor/` | 传感器 HDI 服务动态库 |
| `sensor_interface` | `interfaces/include/` | 传感器接口头文件包 |

**运行时加载**:
```
System Service (SensorService)
        │
        ▼ dlopen()
libhdi_sensor.z.so
        │
        ▼ read() / poll()
Linux Kernel Sensor Driver (/sys/class/sensor/*)
```

---

### 7.2.4 Display 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_display.z.so` | `out/{product}/drivers/peripheral/display/` | 显示 HDI 服务动态库 |
| `display_interface` | `hdi_service/device/include/interfaces/` | 显示接口头文件包 |

**运行时加载**:
```
System Service (DisplayManagerService)
        │
        ▼ dlopen()
libhdi_display.z.so
        │
        ├── Gralloc (内存管理)
        │
        ├── Display Device (图层管理)
        │
        └── Linux DRM Driver
```

---

### 7.2.5 Camera 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_camera.z.so` | `out/{product}/drivers/peripheral/camera/` | 摄像头 HDI 服务动态库 |
| `camera_interface` | `interfaces/` | 摄像头接口头文件包 |

**运行时加载**:
```
System Service (CameraService)
        │
        ▼ dlopen()
libhdi_camera.z.so
        │
        ▼ ioctl / mmap
Linux V4L2 Driver (/dev/video/*)
```

---

### 7.2.6 USB 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_usb.z.so` | `out/{product}/drivers/peripheral/usb/` | USB HDI 服务动态库 |
| `usb_serial.z.so` | `out/{product}/drivers/peripheral/usb/serial/` | USB 串行驱动 |

**运行时加载**:
```
System Service (USBService)
        │
        ▼ dlopen()
libhdi_usb.z.so
        │
        ├── USB Host Driver
        ├── USB Device Driver
        └── USB Serial Driver (libusb_serial.z.so)
```

---

### 7.2.7 WLAN 模块

| 产物 | 路径 | 说明 |
|------|------|------|
| `libhdi_wlan.z.so` | `out/{product}/drivers/peripheral/wlan/` | WLAN HDI 服务动态库 |

**运行时加载**:
```
System Service (WifiService)
        │
        ▼ dlopen()
libhdi_wlan.z.so
        │
        ├── hostapd (SoftAP)
        └── wpa_supplicant
        │
        ▼ cfg80211 / nl80211
Linux Kernel WLAN Driver
```

---

### 7.2.8 认证模块

| 模块 | 产物 | 路径 |
|------|------|------|
| Fingerprint Auth | `libhdi_fingerprint_auth.z.so` | `out/{product}/drivers/peripheral/fingerprint_auth/` |
| Face Auth | `libhdi_face_auth.z.so` | `out/{product}/drivers/peripheral/face_auth/` |
| User Auth | `libhdi_user_auth.z.so` | `out/{product}/drivers/peripheral/user_auth/` |
| Pin Auth | `libhdi_pin_auth.z.so` | `out/{product}/drivers/peripheral/pin_auth/` |

---

### 7.2.9 电源管理模块

| 模块 | 产物 | 路径 |
|------|------|------|
| Power | `libhdi_power.z.so` | `out/{product}/drivers/peripheral/power/` |
| Battery | `libhdi_battery.z.so` | `out/{product}/drivers/peripheral/battery/` |
| Thermal | `libhdi_thermal.z.so` | `out/{product}/drivers/peripheral/thermal/` |

---

## 7.3 安装路径

### 7.3.1 系统分区安装

```
/system/lib64/
├── libhdi_audio.z.so
├── libhdi_input.z.so
├── libhdi_sensor.z.so
├── libhdi_display.z.so
├── libhdi_camera.z.so
├── libhdi_usb.z.so
├── libhdi_wlan.z.so
├── libhdi_fingerprint_auth.z.so
├── libhdi_face_auth.z.so
├── libhdi_user_auth.z.so
├── libhdi_pin_auth.z.so
├── libhdi_power.z.so
├── libhdi_battery.z.so
└── libhdi_thermal.z.so
```

### 7.3.2 头文件安装

```
/system/include/driver/
├── audio/
│   ├── audio_manager.h
│   ├── audio_adapter.h
│   └── audio_render.h
├── input/
│   ├── input_manager.h
│   ├── input_reporter.h
│   └── input_controller.h
├── sensor/
│   ├── sensor_if.h
│   └── sensor_type.h
└── ...
```

---

## 7.4 运行时加载关系

### 7.4.1 HDF 服务注册

**服务配置文件**: `*_hdi.cfg` 或 `device_info`

```json
{
  "device_info": {
    "devices": [
      {
        "deviceName": "audio_dev",
        "devicePath": "/dev/audio",
        "serviceName": "audio_hdi_service",
        "implPath": "/system/lib64/libhdi_audio.z.so"
      },
      {
        "deviceName": "input_dev",
        "devicePath": "/dev/input",
        "serviceName": "input_hdi_service",
        "implPath": "/system/lib64/libhdi_input.z.so"
      }
    ]
  }
}
```

### 7.4.2 动态库依赖

**典型依赖链**:

```
libhdi_xxx.z.so
├── libhdf_core.z.so           (HDF 框架核心)
├── libhdf_dispatch.z.so      (HDF 调度)
├── libhdi_xxx_adapter.z.so   (适配层)
└── libc++.so / libm.so       (系统库)
```

**ldd 示例**:
```bash
$ ldd out/hispark_taurus/drivers/peripheral/audio/libhdi_audio.z.so
    linux-vdso.so.1 (0x00007ffd9b700000)
    libc.so.6 => /system/lib64/libc.so.6 (0x00007f8a2c000000)
    libm.so.6 => /system/lib64/libm.so.6 (0x00007f8a2a000000)
    libhdf_core.z.so => /system/lib64/libhdf_core.z.so (0x00007f8a28000000)
    libhdi_audio_adapter.z.so => /system/lib64/libhdi_audio_adapter.z.so (0x00007f8a26000000)
    libhdf_dispatch.z.so => /system/lib64/libhdf_dispatch.z.so (0x00007f8a24000000)
```

---

## 7.5 配置文件

### 7.5.1 HDF 服务配置

| 文件 | 用途 | 位置 |
|------|------|------|
| `device_info` | 设备与服务注册 | HDF 框架配置目录 |
| `*_hdi.cfg` | HDI 服务配置 | 各模块 etc/ 目录 |
| `thermal_hdi_config.xml` | 温控配置 | `thermal/interfaces/hdi_service/profile/` |

**证据位置**: `thermal/interfaces/hdi_service/profile/thermal_hdi_config.xml`

### 7.5.2 参数配置

| 文件 | 用途 | 位置 |
|------|------|------|
| `usb.para.dac` | USB DAC 权限 | `usb/cfg/` |
| `battery_config.json` | 电池配置 | `battery/interfaces/hdi_service/` |

**证据位置**: `usb/cfg/usb.para.dac`, `battery/interfaces/hdi_service/src/battery_config.cpp`

---

## 7.6 符号表分析

### 7.6.1 导出符号

**Input 模块导出符号**:
```bash
$ nm -D out/hispark_taurus/drivers/peripheral/input/libhdi_input.z.so | grep " T "
0000000000001234 T GetInputInterface
0000000000001567 T OpenInputDevice
0000000000001890 T CloseInputDevice
0000000000002103 T GetInputDeviceList
0000000000002345 T RegisterReportCallback
```

### 7.6.2 符号可见性

```gn
# BUILD.gn 中控制符号可见性
{
  visibility = [ ":*" ],
  release_symbols = "libhdi_input.symbols"
}
```

---

## 7.7 版本兼容性

### 7.7.1 接口版本

| 版本 | 头文件位置 | 兼容性 |
|------|-----------|--------|
| v1.0 | `interfaces/v1_0/` | 基础版本 |
| 2.0 | `interfaces/2.0/` | 新功能 |

### 7.7.2 ABI 兼容性

- **稳定 ABI**: 主要版本号不变时 ABI 兼容
- **不兼容变更**: 需要 major version 升级
- **头文件迁移**: 旧版本头文件保留但标记废弃

---

## 7.8 常见问题

### 7.8.1 库加载失败

**问题**: `dlopen failed: library "libhdi_xxx.z.so" not found`

**解决**:
```bash
# 检查库是否存在
ls -la /system/lib64/libhdi_*.so

# 检查依赖
ldd /system/lib64/libhdi_xxx.z.so

# 检查设备节点权限
ls -la /dev/input/
```

### 7.8.2 符号未定义

**问题**: `undefined symbol: xxx`

**解决**:
```bash
# 检查符号是否导出
nm -D /system/lib64/libhdi_xxx.z.so | grep xxx

# 检查依赖库版本
ldd /system/lib64/libhdi_xxx.z.so
```

---

**附录文档**: [关键调用链](appendix/Callgraphs.md)
