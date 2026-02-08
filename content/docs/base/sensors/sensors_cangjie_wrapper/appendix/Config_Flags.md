# 配置开关

## B.1 构建配置

### 平台判断

**位置**: `ohos/sensor/BUILD.gn:21-31`

```gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.sensor.cj" ]
} else {
    sources = [
      "error.cj",
      "ffi.cj",
      "log.cj",
      "sensor.cj",
      "sensor_manager.cj",
    ]
}
```

**配置项**:

| 配置 | 条件 | 行为 |
|------|------|------|
| `is_mingw` | Windows MinGW | 使用 mock |
| `is_mac` | macOS | 使用 mock |
| 默认 | Linux/Android | 使用实际实现 |

---

## B.2 编译宏

### 无自定义宏

本项目未定义额外的编译宏，所有功能通过源码条件编译实现。

---

## B.3 特征标志

### API Level 注解

**位置**: `sensor.cj` 各定义处

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Sensors.Sensor"
]
```

| 注解字段 | 值 | 说明 |
|----------|-----|------|
| since | "22" | 最低 API Level |
| syscap | SystemCapability.Sensors.Sensor | 系统能力要求 |
| permission | (可选) | 权限要求 |

---

## B.4 功能开关

### 传感器类型

所有 21 种传感器类型在代码中定义，无运行时开关。

### 回调类型

所有回调类型在 `SensorManager.isMatchType()` 中硬编码支持:

**位置**: `sensor_manager.cj:122-147`

```cj
static func isMatchType(`type`: SensorId, callback: CallbackObject): Bool {
    match (`type`) {
        case Accelerometer => callback is Callback1Argument<AccelerometerResponse>
        case AccelerometerUncalibrated => callback is Callback1Argument<AccelerometerUncalibratedResponse>
        // ... 所有类型
        case _ => false
    }
}
```

---

## B.5 日志配置

### 日志通道

**位置**: `ohos/sensor/log.cj:22`

```cj
let SENSOR_LOG = HilogChannel(0, 0xD001C50, "CJ-Sensor")
```

**日志参数**:

| 参数 | 值 | 说明 |
|------|-----|------|
| domain | 0 | 日志域 |
| tag | 0xD001C50 | 日志标签 |
| name | "CJ-Sensor" | 通道名 |

### 日志级别

| 级别 | 函数 | 用途 |
|------|------|------|
| DEBUG | `SENSOR_LOG.debug()` | 调试信息 |
| INFO | `SENSOR_LOG.info()` | 一般信息 |
| WARN | `SENSOR_LOG.warn()` | 警告 |
| ERROR | `SENSOR_LOG.error()` | 错误 |

---

## B.6 性能配置

### 默认上报间隔

**位置**: `sensor.cj:1477`

```cj
const DEFAULT_REPORTING_INTERVAL = 200_000_000  // 200ms
```

| 配置 | 值 | 说明 |
|------|-----|------|
| DEFAULT_REPORTING_INTERVAL | 200,000,000 ns | 默认 200ms (5Hz) |

### IntervalOption 预设

| 预设 | 值 | 频率 |
|------|------|------|
| GameMode | 20,000,000 ns | 50Hz |
| UIMode | 60,000,000 ns | 16Hz |
| NormalMode | 200,000,000 ns | 5Hz |
| SensorNumber(v) | 自定义 | 用户指定 |

---

## B.7 错误码配置

**位置**: `ohos/sensor/error.cj:23-24`

```cj
const SERVICE_EXCEPTION: Int32 = 14500101
const SENSOR_NO_SUPPORT: Int32 = 14500102
```

| 错误码 | 常量 | 用途 |
|--------|------|------|
| 14500101 | SERVICE_EXCEPTION | 服务异常 |
| 14500102 | SENSOR_NO_SUPPORT | 传感器不支持 |
| 201 | (权限错误) | 权限校验失败 |
