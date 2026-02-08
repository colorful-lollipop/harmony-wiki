# API 参考

## 3.1 API 清单

### 公共 API 概览

| API | 功能 | 同步/异步 | Throws | Worker Thread |
|-----|------|---------|--------|---------------|
| [on()](#on---订阅传感器数据) | 订阅传感器数据 | 异步 | 是 | 否 |
| [once()](#once---单次订阅传感器数据) | 单次订阅 | 异步 | 是 | 否 |
| [off()](#off---取消订阅) | 取消订阅 | 同步 | 是 | 否 |
| [getSensorList()](#getsensorlist---获取所有传感器列表) | 获取所有传感器 | 异步 | 是 | 是 |
| [getSingleSensor()](#getsinglesensor---获取单个传感器信息) | 获取单个传感器 | 异步 | 是 | 是 |

### 命名空间

- **Kit 命名空间**: `kit.SensorServiceKit`
- **OHOS 命名空间**: `ohos.sensor`

## 3.2 订阅类 API

### on() - 订阅传感器数据

**位置**: `ohos/sensor/sensor.cj:1653-1666`

```cj
public func on<T>(sensorType: SensorId, callback: Callback1Argument<T>, option!: ?Options = None): Unit
```

**功能**: 订阅指定传感器的持续数据流。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sensorType | SensorId | 是 | 传感器类型枚举 |
| callback | Callback1Argument\<T\> | 是 | 回调函数，接收 Response 数据 |
| option | Options | 否 | 订阅选项（采样间隔等） |

**Options**:

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| interval | IntervalOption | NormalMode | 数据上报间隔 |
| sensorInfoParam | SensorInfoParam | None | 设备/传感器索引 |

**IntervalOption**:

| 枚举值 | 间隔 (ns) | 用途 |
|--------|----------|------|
| GameMode | 20,000,000 | 游戏模式 (50Hz) |
| UIMode | 60,000,000 | UI 模式 (16Hz) |
| NormalMode | 200,000,000 | 普通模式 (5Hz) |
| SensorNumber(v) | 自定义 | 指定纳秒值 |

**Throws**:

| 错误码 | 条件 |
|--------|------|
| 201 | 权限校验失败 |
| 14500101 | 服务异常 (HDF/IPC/数据通道) |

**示例**:

```cj
import ohos.sensor.*

// 订阅加速度计
on(Accelerometer) {
    timestamp, response =>
    let acc = response.getOrThrow()
    println("X: ${acc.x}, Y: ${acc.y}, Z: ${acc.z}")
}

// 订阅带选项
on(Gyroscope, Option(interval: GameMode)) {
    timestamp, response =>
    // ...
}
```

---

### once() - 单次订阅传感器数据

**位置**: `ohos/sensor/sensor.cj:1681-1698`

```cj
public func once<T>(sensorType: SensorId, callback: Callback1Argument<T>): Unit
```

**功能**: 订阅一次传感器数据，收到数据后自动取消订阅。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sensorType | SensorId | 是 | 传感器类型枚举 |
| callback | Callback1Argument\<T\> | 是 | 回调函数 |

**Throws**:

| 错误码 | 条件 |
|--------|------|
| 201 | 权限校验失败 |
| 14500101 | 服务异常 |

**特殊逻辑**:
- 如果已有持续订阅 (`on()`)，只注册 once 回调，下次数据返回时自动清理
- 如果没有订阅，发起新的 FFI 订阅请求

**示例**:

```cj
import ohos.sensor.*

// 单次获取心率
once(HeartRate) {
    timestamp, response =>
    let hr = response.getOrThrow()
    println("Heart Rate: ${hr.heartRate}")
}
```

---

### off() - 取消订阅

**位置**: `ohos/sensor/sensor.cj:1713-1731`

```cj
public func off(sensorType: SensorId, callback!: ?CallbackObject = None): Unit
```

**功能**: 取消传感器订阅。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sensorType | SensorId | 是 | 传感器类型枚举 |
| callback | CallbackObject | 否 | 指定回调（省略则取消所有该传感器回调） |

**Throws**:

| 错误码 | 条件 |
|--------|------|
| 201 | 权限校验失败 |

**示例**:

```cj
import ohos.sensor.*

// 取消所有加速度计订阅
off(Accelerometer)

// 取消指定回调
let myCallback: Callback1Argument<GyroscopeResponse> = ...
off(Gyroscope, Some(myCallback))
```

---

## 3.3 查询类 API

### getSensorList() - 获取所有传感器列表

**位置**: `ohos/sensor/sensor.cj:1770-1781`

```cj
public func getSensorList(): Array<Sensor>
```

**功能**: 获取设备上所有可用的传感器列表。

**返回值**:

| 类型 | 说明 |
|------|------|
| Array\<Sensor\> | 传感器信息数组 |

**Throws**:

| 错误码 | 条件 |
|--------|------|
| 14500101 | 服务异常 |

**Worker Thread**: 此 API 在工作线程执行，适用于耗时操作。

**示例**:

```cj
import ohos.sensor.*

let sensors = getSensorList()
for (sensor in sensors) {
    println("ID: ${sensor.sensorId}, Name: ${sensor.sensorName}")
    println("Vendor: ${sensor.vendorName}, Power: ${sensor.power}mA")
}
```

---

### getSingleSensor() - 获取单个传感器信息

**位置**: `ohos/sensor/sensor.cj:1747-1756`

```cj
public func getSingleSensor(sensorType: SensorId): Sensor
```

**功能**: 获取指定类型传感器的详细信息。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sensorType | SensorId | 是 | 传感器类型枚举 |

**返回值**:

| 类型 | 说明 |
|------|------|
| Sensor | 传感器信息 |

**Throws**:

| 错误码 | 条件 |
|--------|------|
| 14500101 | 服务异常 |
| 14500102 | 传感器不支持 |

**Worker Thread**: 此 API 在工作线程执行。

**示例**:

```cj
import ohos.sensor.*

let acc = getSingleSensor(Accelerometer)
println("Sensor: ${acc.sensorName}")
println("Max Range: ${acc.maxRange}")
println("Min Period: ${acc.minSamplePeriod}ns")
```

---

## 3.4 类型定义

### SensorId - 传感器类型枚举

**位置**: `ohos/sensor/sensor.cj:34-300`

| 枚举成员 | 值 | 权限 | 说明 |
|----------|-----|------|------|
| Accelerometer | 1 | ACCELEROMETER | 加速度计 |
| Gyroscope | 2 | GYROSCOPE | 陀螺仪 |
| AmbientLight | 5 | 无 | 环境光传感器 |
| MagneticField | 6 | 无 | 磁场传感器 |
| Barometer | 8 | 无 | 气压计 |
| Hall | 10 | 无 | 霍尔传感器 |
| Proximity | 12 | 无 | 接近传感器 |
| Humidity | 13 | 无 | 湿度传感器 |
| Orientation | 256 | 无 | 方向传感器 |
| Gravity | 257 | 无 | 重力传感器 |
| LinearAccelerometer | 258 | ACCELEROMETER | 线性加速度计 |
| RotationVector | 259 | 无 | 旋转矢量 |
| AmbientTemperature | 260 | 无 | 环境温度 |
| MagneticFieldUncalibrated | 261 | 无 | 未校准磁场 |
| GyroscopeUncalibrated | 263 | GYROSCOPE | 未校准陀螺仪 |
| SignificantMotion | 264 | 无 | 显著运动 |
| PedometerDetection | 265 | ACTIVITY_MOTION | 计步检测 |
| Pedometer | 266 | ACTIVITY_MOTION | 计步器 |
| HeartRate | 278 | READ_HEALTH_DATA | 心率传感器 |
| WearDetection | 280 | 无 | 佩戴检测 |
| AccelerometerUncalibrated | 281 | ACCELEROMETER | 未校准加速度计 |

**方法**:

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `getValue()` | Int32 | 获取枚举对应的传感器类型 ID |
| `parse(v: Int32)` | SensorId | 从 ID 解析枚举值 |

---

### Response - 传感器数据基类

**位置**: `ohos/sensor/sensor.cj:571-610`

```cj
public open class Response {
    public var timestamp: Int64    // 数据时间戳 (ns)
    public var accuracy: SensorAccuracy  // 数据精度
}
```

---

### SensorAccuracy - 精度枚举

**位置**: `ohos/sensor/sensor.cj:310-371`

| 枚举成员 | 值 | 说明 |
|----------|-----|------|
| AccuracyUnreliable | 0 | 不可靠 |
| AccuracyLow | 1 | 低精度 |
| AccuracyMedium | 2 | 中精度 |
| AccuracyHigh | 3 | 高精度 |

---

### Sensor - 传感器信息类

**位置**: `ohos/sensor/sensor.cj:1506-1636`

| 属性 | 类型 | 说明 |
|------|------|------|
| sensorName | String | 传感器名称 |
| vendorName | String | 供应商名称 |
| firmwareVersion | String | 固件版本 |
| hardwareVersion | String | 硬件版本 |
| sensorId | Int32 | 传感器类型 ID |
| maxRange | Float32 | 最大测量范围 |
| minSamplePeriod | Int64 | 最小采样周期 (ns) |
| maxSamplePeriod | Int64 | 最大采样周期 (ns) |
| precision | Float32 | 精度 |
| power | Float32 | 功耗 (mA) |

---

### 具体 Response 类型

继承自 `Response`，包含各传感器的具体数据字段：

| 类型 | 字段 | 说明 |
|------|------|------|
| AccelerometerResponse | x, y, z (Float32) | 加速度 (m/s²) |
| GyroscopeResponse | x, y, z (Float32) | 角速度 (rad/s) |
| LightResponse | intensity, colorTemperature, infraredLuminance | 光照数据 |
| BarometerResponse | pressure (Float32) | 气压 (hPa) |
| HeartRateResponse | heartRate (UInt32) | 心率 (bpm) |
| PedometerResponse | steps (Int64) | 步数 |
| ... | ... | 其他传感器类型 |

---

## 3.5 错误码

**位置**: `ohos/sensor/error.cj:23-24`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 14500101 | SERVICE_EXCEPTION | 服务异常 (HDF/IPC/数据通道) |
| 14500102 | SENSOR_NO_SUPPORT | 传感器不支持 |
| 201 | (权限错误) | 权限校验失败 |

**错误处理**:

```cj
try {
    on(Accelerometer) { timestamp, response =>
        // ...
    }
} catch (e: BusinessException) {
    match (e.code) {
        case 201 => println("Permission denied")
        case 14500101 => println("Service exception")
        case _ => println("Unknown error: ${e.code}")
    }
}
```
