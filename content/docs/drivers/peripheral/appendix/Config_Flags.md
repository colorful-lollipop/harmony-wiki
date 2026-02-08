# 配置宏说明

本文档说明各模块使用的关键编译宏和运行时配置。

## 编译期配置

### 1.1 平台配置

| 宏定义 | 说明 | 使用模块 | 证据位置 |
|--------|------|----------|----------|
| `__OHOS__` | OpenHarmony 平台标识 | 所有模块 | `audio/BUILD.gn` |
| `__UNIX_THREAD_SUPPORT__` | Unix 线程支持 | audio | `audio/BUILD.gn` |
| `__LINUX_USER__` | Linux 用户态 | 所有 Linux 模块 | `audio.gni` |
| `__LITEOS__` | LiteOS 内核 | LiteOS 模块 | `audio.gni` |

### 1.2 功能开关

| 宏定义 | 默认值 | 说明 | 相关模块 |
|--------|--------|------|----------|
| `ENABLE_AUDIO_HDMI` | 关闭 | HDMI 音频支持 | audio |
| `ENABLE_AUDIO_SPDIF` | 关闭 | S/PDIF 音频支持 | audio |
| `ENABLE_INPUT_HID` | 开启 | HID 设备支持 | input |
| `ENABLE_SENSOR_BATCH` | 开启 | 传感器批处理 | sensor |

### 1.3 调试配置

| 宏定义 | 说明 | 证据位置 |
|--------|------|----------|
| `HDF_LOG_TAG` | 日志标签 | 各模块日志宏 |
| `HDF_LOG_LEVEL` | 日志级别 | `hdf_trace.h` |
| `INPUT_DEBUG` | 输入调试开关 | input |

---

## 运行时配置

### 2.1 HDF 服务配置

#### 2.1.1 设备信息配置

**文件**: `device_info` 或 `*_hdi.cfg`

**示例**:
```json
{
  "device_info": {
    "devices": [
      {
        "deviceName": "audio_dev",
        "serviceName": "audio_hdi_service",
        "implPath": "/system/lib64/libhdi_audio.z.so"
      }
    ]
  }
}
```

#### 2.1.2 权限配置

**证据位置**: `usb/cfg/usb.para.dac`

```
# USB 设备权限配置
/dev/bus/usb/*  uid=system gid=system mode=0660
```

---

### 2.2 模块特定配置

#### 2.2.1 Audio 模块配置

**配置项**:

| 配置项 | 类型 | 说明 | 证据位置 |
|--------|------|------|----------|
| `audio.primary.active` | bool | 主音频路径激活 | `audio/hal/hdi_passthrough/` |
| `audio.usb.enable` | bool | USB 音频支持 | `audio/hal/hdi_passthrough/` |
| `audio.a2dp.enable` | bool | A2DP 蓝牙音频 | `audio/hal/hdi_binder/` |

#### 2.2.2 Battery 模块配置

**证据位置**: `battery/interfaces/hdi_service/src/battery_config.cpp`

**配置项**:

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `battery.capacity` | int | 电池容量 (mAh) |
| `battery.voltage_max` | int | 最大电压 (mV) |
| `battery.temperature` | int | 初始温度 |

#### 2.2.3 Thermal 模块配置

**证据位置**: `thermal/interfaces/hdi_service/profile/thermal_hdi_config.xml`

**配置项**:

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `thermal.zone.*` | object | 热区配置 |
| `thermal.governor.*` | object | 温控策略 |
| `thermal.throttle.*` | object | 降频阈值 |

---

## 编译选项

### 3.1 GCC/Clang 选项

| 选项 | 值 | 说明 |
|------|-------|------|
| `-Wall` | 开启 | 警告信息 |
| `-Werror` | 开启 | 警告视为错误 |
| `-O2` | 默认 | 优化级别 |
| `-fno-rtti` | 开启 | 禁用运行时类型信息 |
| `-fno-exceptions` | 开启 | 禁用异常 |

### 3.2 链接选项

| 选项 | 说明 |
|------|------|
| `-Wl,-z,relro` | 只读重定位 |
| `-Wl,-z,now` | 立即绑定 |
| `-Wl,--as-needed` | 按需链接 |
| `-fPIC` | 位置无关代码 |

---

## 配置文件路径

### 4.1 系统配置

| 配置类型 | 路径 | 说明 |
|----------|------|------|
| HDF 设备信息 | `/system/etc/device_info` | 设备与服务注册 |
| USB 权限 | `/system/etc/usb.para.dac` | USB DAC 权限 |
| 温控配置 | `/system/etc/thermal_hdi_config.xml` | 温控策略 |

### 4.2 模块配置

| 模块 | 配置路径 |
|------|----------|
| audio | `/data/audio/` |
| battery | `/data/battery/` |
| thermal | `/data/thermal/` |

---

## 环境变量

### 5.1 调试环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `HDF_LOG_LEVEL` | 设置日志级别 | `export HDF_LOG_LEVEL=3` |
| `HDF_TRACING` | 启用追踪 | `export HDF_TRACING=1` |
| `INPUT_DEBUG` | 输入调试开关 | `export INPUT_DEBUG=1` |

---

## 默认参数

### 6.1 Audio 默认参数

| 参数 | 默认值 | 单位 |
|------|--------|------|
| `sampleRate` | 48000 | Hz |
| `channels` | 2 | - |
| `format` | S16_LE | - |
| `periodSize` | 512 | samples |

### 6.2 Sensor 默认参数

| 参数 | 默认值 | 单位 |
|------|--------|------|
| `samplingInterval` | 200000000 | ns (200ms) |
| `reportInterval` | 200000000 | ns (200ms) |

### 6.3 Input 默认参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pollTimeout` | 5000 | ms |
| `maxEventQueue` | 1024 | events |

---

## 兼容性配置

### 7.1 API 版本兼容

| 接口版本 | 头文件路径 | 兼容性 |
|----------|-----------|--------|
| v1.0 | `interfaces/v1_0/` | 基础版本 |
| 2.0 | `interfaces/2.0/` | 向后兼容 v1.0 |

### 7.2 内核兼容

| 内核版本 | 支持状态 | 备注 |
|----------|----------|------|
| Linux 5.10+ | ✅ 支持 | 主要测试平台 |
| Linux 4.19 | ✅ 支持 | 兼容 |
| LiteOS-M | ✅ 支持 | 嵌入式平台 |
