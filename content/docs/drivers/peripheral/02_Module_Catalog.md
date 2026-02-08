# 模块目录

本文档列出 `drivers/peripheral` 仓库中的所有外设驱动模块及其简要说明。

## 模块概览

| # | 模块名 | 目录 | 主要功能 |
|---|--------|------|----------|
| 1 | Audio | `audio/` | 音频播放、录音、混音、音效 |
| 2 | Battery | `battery/` | 电池状态监控、充电管理 |
| 3 | Bluetooth | `bluetooth/` | 蓝牙通信 |
| 4 | Camera | `camera/` | 摄像头采集、预览、拍照、录像 |
| 5 | Clearplay | `clearplay/` | DRM 内容解密播放 |
| 6 | Codec | `codec/` | 音视频编解码 |
| 7 | Connected NFC Tag | `connected_nfc_tag/` | NFC 标签连接 |
| 8 | Display | `display/` | 显示图层管理、图形渲染 |
| 9 | Distributed Audio | `distributed_audio/` | 分布式音频 |
| 10 | Distributed Camera | `distributed_camera/` | 分布式摄像头 |
| 11 | Ethernet | `ethernet/` | 以太网连接 |
| 12 | Face Auth | `face_auth/` | 人脸认证 |
| 13 | Fingerprint Auth | `fingerprint_auth/` | 指纹认证 |
| 14 | Format | `format/` | 媒体格式复用/解复用 |
| 15 | Huks | `huks/` | 密钥管理 |
| 16 | Input | `input/` | 输入设备（触摸、按键） |
| 17 | Intelligent Voice | `intelligent_voice/` | 智能语音 |
| 18 | Light | `light/` | 灯光控制 |
| 19 | Location | `location/` | 定位服务 |
| 20 | Low Power Player | `low_power_player/` | 低功耗播放 |
| 21 | Memorytracker | `memorytracker/` | 内存追踪 |
| 22 | MIDI | `midi/` | MIDI 音乐 |
| 23 | Motion | `motion/` | 运动检测 |
| 24 | NFC | `nfc/` | NFC 通信 |
| 25 | Partitionslot | `partitionslot/` | 分区管理 |
| 26 | Pin Auth | `pin_auth/` | PIN 码认证 |
| 27 | Power | `power/` | 电源管理 |
| 28 | Ril | `ril/` | 无线通信（电话、短信） |
| 29 | Secure Element | `secure_element/` | 安全元素 |
| 30 | Sensor | `sensor/` | 传感器数据采集 |
| 31 | Thermal | `thermal/` | 温控管理 |
| 32 | USB | `usb/` | USB 设备管理 |
| 33 | User Auth | `user_auth/` | 用户认证 |
| 34 | Vibrator | `vibrator/` | 振动控制 |
| 35 | WLAN | `wlan/` | 无线局域网 |

---

## 音视频与显示

### Audio 音频驱动

**目录**: `audio/`

**功能描述**:
- 管理声卡驱动的加载和卸载
- 创建音频播放（Render）和录音（Capture）对象
- 选择音频场景（Audio Scene）
- 设置音频属性（采样率、声道等）
- 设置音频音量及增益
- 控制音频播放及录音的启停

**核心接口文件**:
- `interfaces/include/audio_manager.h` - 音频管理器接口
- `interfaces/include/audio_adapter.h` - 音频适配器接口
- `interfaces/include/audio_render.h` - 播放接口
- `interfaces/include/audio_capture.h` - 录音接口

**架构分层**:
- `interfaces/` - HDI 接口定义
- `hal/` - HAL 实现（hdi_passthrough, hdi_binder）
- `hdi_service/` - HDI 服务（primary, event）
- `supportlibs/` - 辅助库

**证据**: `audio/README_zh.md:15-19`

---

### Camera 摄像头驱动

**目录**: `camera/`

**功能描述**:
- 摄像头硬件设备管理
- 图像采集与预览
- 拍照与录像
- 摄像头参数配置
- 分布式摄像头支持

**核心接口文件**:
- `interfaces/hdi_ipc/icamera_interface.h` - 摄像头接口

**证据**: `camera/interfaces/hdi_ipc/icamera_interface.h:20`

```cpp
class ICameraInterface : public IRemoteBroker { ... }
```

---

### Codec 编解码驱动

**目录**: `codec/`

**功能描述**:
- 音视频编解码能力
- 编解码器创建与销毁
- 编码参数配置
- 解码数据输出

**证据**: `codec/README_zh.md`

---

### Display 显示驱动

**目录**: `display/`

**功能描述**:
- 显示图层（Display Layer）管理
- 显示内存（FrameBuffer）管理
- 硬件图形加速
- 显示设备热插拔

**核心接口文件**:
- `hdi_service/device/include/interfaces/idisplay_device_callback.h`

**证据**: `display/hdi_service/device/include/interfaces/idisplay_device_callback.h:23`

```cpp
class DisplayRegisterCallbackBase : public IRemoteBroker { ... }
```

---

### Format 格式驱动

**目录**: `format/`

**功能描述**:
- 媒体文件复用（Multiplexing）
- 媒体文件解复用（Demultiplexing）
- 码流分析

**证据**: `format/README_zh.md`

---

## 输入与传感

### Input 输入驱动

**目录**: `input/`

**功能描述**:
- 输入设备管理（打开、关闭）
- 输入设备列表查询
- 输入事件上报（注册/注销回调）
- 设备信息查询（类型、厂商、芯片）
- 电源状态控制
- 手势模式设置
- 电容自检测试

**核心接口文件**:
- `interfaces/include/input_manager.h` - 设备管理
- `interfaces/include/input_reporter.h` - 事件上报
- `interfaces/include/input_controller.h` - 设备控制

**接口清单**:
| 接口 | 功能 | 证据位置 |
|------|------|----------|
| `OpenInputDevice` | 打开输入设备 | `input_manager.h` |
| `CloseInputDevice` | 关闭输入设备 | `input_manager.h` |
| `GetInputDeviceList` | 获取设备列表 | `input_manager.h` |
| `RegisterReportCallback` | 注册事件回调 | `input_reporter.h` |
| `SetPowerStatus` | 设置电源状态 | `input_controller.h` |
| `GetDeviceType` | 获取设备类型 | `input_controller.h` |

**证据**: `input/README_zh.md:15-17`

---

### Sensor 传感器驱动

**目录**: `sensor/`

**功能描述**:
- 传感器信息查询（类型、厂商、版本）
- 传感器使能/去使能
- 数据采样间隔配置
- 数据上报模式设置
- 传感器数据订阅/取消订阅

**核心接口文件**:
- `interfaces/include/sensor_if.h` - 传感器接口
- `interfaces/include/sensor_type.h` - 类型定义

**接口清单**:
| 接口 | 功能 | 证据位置 |
|------|------|----------|
| `GetAllSensors` | 获取所有传感器 | `sensor_if.h` |
| `Enable` | 使能传感器 | `sensor_if.h` |
| `Disable` | 去使能传感器 | `sensor_if.h` |
| `SetBatch` | 设置采样参数 | `sensor_if.h` |
| `Register` | 注册数据回调 | `sensor_if.h` |

**证据**: `sensor/README_zh.md:13-21`

---

### Motion 运动检测

**目录**: `motion/`

**功能描述**:
- 运动状态检测
- 姿态识别
- 运动数据上报

**核心接口文件**:
- `interfaces/v1_0/imotion_interface_vdi.h`
- `interfaces/v1_0/imotion_callback_vdi.h`

**证据**: `motion/interfaces/v1_0/imotion_interface_vdi.h`

---

## 通信与连接

### WLAN 无线局域网

**目录**: `wlan/`

**功能描述**:
- WLAN 驱动加载
- 无线网络连接管理
- WiFi 特性查询

**子模块**:
- `hostapd/` - SoftAP 支持
- `wpa/` - WPA Supplicant

**证据**: `wlan/README_zh.md`

---

### Bluetooth 蓝牙

**目录**: `bluetooth/`

**功能描述**:
- 蓝牙设备管理
- 蓝牙通信

**证据**: `bluetooth/` 目录结构

---

### USB 通用串行总线

**目录**: `usb/`

**功能描述**:
- USB Host/Device 管理
- 设备配置
- 数据读写
- 串行通信

**子模块**:
- `serial/` - USB 串行驱动
- `test/` - 测试代码

**证据**: `usb/serial/include/usb_serial.h`

---

### NFC 近场通信

**目录**: `nfc/`

**功能描述**:
- NFC 通信
- 标签读写

**证据**: `nfc/` 目录结构

---

### Ril 无线接口层

**目录**: `ril/`

**功能描述**:
- 通话管理
- SIM 卡管理
- 短信/ MMS 管理
- 蜂窝数据
- 无线网络搜索

**证据**: `ril/README_zh.md:25`

---

## 安全认证

### Fingerprint Auth 指纹认证

**目录**: `fingerprint_auth/`

**功能描述**:
- 指纹录入与删除
- 指纹比对
- 认证结果回调

**核心接口文件**:
- `hdi_service/include/fingerprint_auth_hdi.h`
- `hdi_service/include/fingerprint_auth_interface_service.h`

**证据**: `fingerprint_auth/hdi_service/include/fingerprint_auth_hdi.h`

---

### Face Auth 人脸认证

**目录**: `face_auth/`

**功能描述**:
- 人脸检测
- 人脸比对
- 活体检测

**核心接口文件**:
- `hdi_service/include/face_auth_hdi.h`
- `hdi_service/include/face_auth_interface_service.h`

**证据**: `face_auth/hdi_service/include/face_auth_hdi.h`

---

### User Auth 用户认证

**目录**: `user_auth/`

**功能描述**:
- 用户身份验证
- 认证凭据管理

**证据**: `user_auth/` 目录结构

---

### Pin Auth PIN 认证

**目录**: `pin_auth/`

**功能描述**:
- PIN 码验证
- PIN 码管理

**证据**: `pin_auth/` 目录结构

---

### Secure Element 安全元素

**目录**: `secure_element/`

**功能描述**:
- 安全元素访问
- 密钥存储

**证据**: `secure_element/` 目录结构

---

## 电源与功耗

### Power 电源管理

**目录**: `power/`

**功能描述**:
- 电源状态管理
- 功耗优化
- 唤醒源管理

**核心接口文件**:
- `interfaces/hdi_service/include/` - HDI 接口

**证据**: `power/interfaces/hdi_service/include/` 目录

---

### Battery 电池管理

**目录**: `battery/`

**功能描述**:
- 电池状态监控
- 充电状态管理
- 电源信息查询

**核心接口文件**:
- `interfaces/include/batteryd_api.h`
- `interfaces/hdi_service/include/battery_interface_impl.h`

**证据**: `battery/interfaces/include/batteryd_api.h`

---

### Thermal 温控管理

**目录**: `thermal/`

**功能描述**:
- 温度监控
- 温控策略执行
- 热管理配置

**核心接口文件**:
- `interfaces/hdi_service/include/thermal_interface_impl.h`

**证据**: `thermal/interfaces/hdi_service/include/thermal_interface_impl.h`

---

## 其他外设

### Vibrator 振动器

**目录**: `vibrator/`

**功能描述**:
- 振动控制
- 振动模式设置
- 振动强度控制

**核心接口文件**:
- `interfaces/` - HDI 接口定义

**证据**: `vibrator/interfaces/` 目录

---

### Light 灯光控制

**目录**: `light/`

**功能描述**:
- 灯光类型管理
- 灯光亮度控制
- 灯光颜色设置

**核心接口文件**:
- `interfaces/include/light_if.h`
- `interfaces/include/light_type.h`

**证据**: `light/interfaces/include/light_if.h`

---

### Location 定位服务

**目录**: `location/`

**功能描述**:
- GNSS 定位
- 地理围栏
- 位置信息查询

**子模块**:
- `gnss/` - GNSS 定位
- `geofence/` - 地理围栏

**证据**: `location/` 目录结构

---

### Intelligent Voice 智能语音

**目录**: `intelligent_voice/`

**功能描述**:
- 语音识别
- 语音控制

**证据**: `intelligent_voice/` 目录结构

---

## 基础组件

### Base 基础组件

**目录**: `base/`

**功能描述**:
- 公共数据结构（buffer_handle）
- 追踪日志（hdf_trace）
- 基础类型定义

**核心头文件**:
- `buffer_handle.h` - 内存句柄
- `hdf_trace.h` - 追踪日志

**证据**: `base/buffer_handle.h`, `base/hdf_trace.h`

---

### Adapter 适配层

**目录**: `adapter/`

**功能描述**:
- 平台适配代码
- OS 适配层

**证据**: `adapter/` 目录结构

---

## 模块依赖关系图

```
                    ┌─────────────────────────────────────┐
                    │           System Services            │
                    │  (AudioService, InputService, etc.) │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │         drivers_framework           │
                    │     (HDF, IPC, Device Manager)     │
                    └──────────────┬──────────────────────┘
                                   │
        ┌──────────────┬───────────┼───────────┬──────────────┐
        │              │           │           │              │
   ┌────▼────┐    ┌────▼────┐ ┌───▼───┐ ┌────▼────┐    ┌────▼────┐
   │  Audio  │    │  Input   │ │Sensor │ │ Display │    │ Camera  │
   └─────────┘    └─────────┘ └───┬───┘ └─────────┘    └─────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │       drivers_adapter               │
                    │      (HDI-Adapter, KHDF)           │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │     Linux Kernel Drivers           │
                    └─────────────────────────────────────┘
```

---

**下一节**: [HDI 接口说明](03_HDI_Interfaces.md) - 了解详细接口定义
