# 项目概览

## 1.1 项目定位

**Generic Sensor Service Cangjie Wrapper** 是 OpenHarmony 系统中为 **Cangjie 语言** 开发者提供传感器能力的封装层。

### 核心能力

| 能力 | 说明 |
|------|------|
| 订阅传感器数据 | 持续接收传感器数据推送 |
| 单次订阅 | 获取一次传感器数据后自动取消 |
| 获取传感器信息 | 查询设备支持的传感器列表和属性 |

### 定位层级

```
┌─────────────────────────────────────┐
│  Cangjie 应用层                      │
│  (开发者使用 SensorServiceKit)        │
├─────────────────────────────────────┤
│  Sensors Cangjie Wrapper (本项目)    │
│  - 封装 FFI 接口                     │
│  - 类型转换与校验                    │
│  - 回调管理                          │
├─────────────────────────────────────┤
│  sensors_sensor (cj_sensor_ffi)     │
│  - C++ FFI 实现                      │
│  - IPC 通信                          │
├─────────────────────────────────────┤
│  Sensor Service (SA)               │
│  - 传感器服务系统能力                │
├─────────────────────────────────────┤
│  HDF Driver                         │
│  - 硬件抽象层驱动                    │
└─────────────────────────────────────┘
```

### 支持的传感器 (21 种)

| 类别 | 传感器 |
|------|--------|
| 运动 | Accelerometer, Gyroscope, LinearAccelerometer, Gravity, RotationVector |
| 校正 | AccelerometerUncalibrated, GyroscopeUncalibrated, MagneticFieldUncalibrated |
| 环境 | Light, AmbientTemperature, Humidity, Barometer |
| 位置 | MagneticField, Orientation, Proximity, Hall |
| 健康 | HeartRate, Pedometer, PedometerDetection |
| 检测 | SignificantMotion, WearDetection |

## 1.2 运行环境

### 系统要求

- **操作系统**: OpenHarmony (标准设备)
- **系统能力**: `SystemCapability.Sensors.Sensor`
- **API Level**: 22+
- **语言版本**: Cangjie Beta

### 硬件要求

- 设备必须配备相应的传感器硬件
- 部分传感器需要权限授权

### 依赖组件

| 组件 | 用途 |
|------|------|
| `cangjie_ark_interop` | 注解类定义、异常类 |
| `hiviewdfx_cangjie_wrapper` | 日志接口 (Hilog) |
| `sensor` | 底层 C++ FFI (`cj_sensor_ffi`) |

## 1.3 目录结构

```
base/sensors/sensors_cangjie_wrapper/
│
├── figures/                          # 文档资源
│   └── sensors_cangjie_wrapper_architecture_en.png
│
├── kit/                              # 对外 Kit API
│   └── SensorServiceKit/
│       ├── index.cj                  # 包入口，重导出 ohos.sensor
│       └── BUILD.gn
│
├── ohos/                             # OHOS 层接口
│   └── sensor/
│       ├── index.cj                 # 包入口
│       ├── ffi.cj                   # FFI 接口定义 (C 结构、foreign 函数)
│       ├── sensor.cj                # 传感器 API、数据类型定义
│       ├── sensor_manager.cj        # 回调管理器
│       ├── error.cj                 # 错误码定义
│       ├── log.cj                   # 日志通道
│       └── BUILD.gn
│
├── mock/                             # Mock 实现 (Windows/Mac)
│   └── ohos.sensor.cj
│
├── test/                             # 测试目录 (不纳入文档)
│   └── sensor/
│
├── BUILD.gn                          # 根构建配置
├── bundle.json                       # 包配置
├── README.md / README_zh.md          # 项目说明
└── LICENSE                           # Apache 2.0
```

### 关键文件说明

| 文件 | 职责 |
|------|------|
| `kit/SensorServiceKit/index.cj` | Kit 入口，导出 `SensorServiceKit` 命名空间 |
| `ohos/sensor/sensor.cj` | API 实现、传感器类型枚举、Response 类 |
| `ohos/sensor/ffi.cj` | C 结构定义、FFI 函数声明 |
| `ohos/sensor/sensor_manager.cj` | 回调注册表、数据分发 |
| `ohos/sensor/error.cj` | 错误码映射 |
| `ohos/sensor/log.cj` | 日志通道初始化 |

## 1.4 版本与约束

### 版本信息

- **当前版本**: 6.1 (见 `bundle.json`)
- **API 版本**: 22+ (见 `@!APILevel` 注解)
- **状态**: Beta

### 已知限制

与 ArkTS API 相比，以下能力暂不支持：

1. 获取地球上特定位置的地球磁场信息
2. 基于气压值获取海拔高度

### 权限要求

| 传感器 | 所需权限 |
|--------|----------|
| Accelerometer | `ohos.permission.ACCELEROMETER` |
| LinearAccelerometer | `ohos.permission.ACCELEROMETER` |
| AccelerometerUncalibrated | `ohos.permission.ACCELEROMETER` |
| Gyroscope | `ohos.permission.GYROSCOPE` |
| GyroscopeUncalibrated | `ohos.permission.GYROSCOPE` |
| Pedometer | `ohos.permission.ACTIVITY_MOTION` |
| PedometerDetection | `ohos.permission.ACTIVITY_MOTION` |
| HeartRate | `ohos.permission.READ_HEALTH_DATA` |

## 1.5 快速开始

### 基本用法

```cj
import ohos.sensor.*

// 获取传感器列表
let sensors = getSensorList()
for (sensor in sensors) {
    println("Sensor: ${sensor.sensorName}, Vendor: ${sensor.vendorName}")
}

// 订阅加速度计数据
on(Accelerometer) {
    timestamp, response =>
    let acc = response.getOrThrow()
    println("X: ${acc.x}, Y: ${acc.y}, Z: ${acc.z}")
}

// 取消订阅
off(Accelerometer)
```

### 完整示例见

- [API 参考](03_API_Reference.md)
- [官方开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/device/sensor/cj-sensor-guidelines.md)
