# 常见问题

## 7.1 构建问题

### 问题 1: Windows/macOS 编译失败

**现象**:
```
error: Could not find source file: error.cj
```

**原因**: 在 Windows/macOS 平台使用了条件编译逻辑

**位置**: `ohos/sensor/BUILD.gn:21-31`

```gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.sensor.cj" ]
} else {
    sources = [
      "error.cj",
      "ffi.cj",
      // ...
    ]
}
```

**说明**: Windows 和 macOS 平台使用 mock 实现，不支持实际传感器功能。

**解决**:
- 在 Linux 或 Android 设备上编译
- 或者确认是否需要 Windows/macOS 支持

---

### 问题 2: 找不到 cj_sensor_ffi 依赖

**现象**:
```
ninja: error: dependency '//base/sensors/sensors_sensor:cj_sensor_ffi' not found
```

**原因**: `sensors_sensor` 子系统未正确编译

**解决**:
1. 确保已拉取 `sensors_sensor` 仓库
2. 检查 `sensor` 组件是否在构建配置中启用
3. 重新同步构建配置:
```bash
hb set
hb build
```

---

### 问题 3: bundle.json 配置错误

**现象**:
```
error: Invalid bundle.json: missing required field 'name'
```

**解决**: 检查 `bundle.json` 格式是否正确

---

## 7.2 运行时问题

### 问题 4: 权限被拒绝 (错误码 201)

**现象**:
```
BusinessException: Permission verification failed
```

**原因**: 未申请所需权限

**涉及传感器**:

| 传感器 | 所需权限 |
|--------|----------|
| Accelerometer | `ohos.permission.ACCELEROMETER` |
| Gyroscope | `ohos.permission.GYROSCOPE` |
| Pedometer | `ohos.permission.ACTIVITY_MOTION` |
| HeartRate | `ohos.permission.READ_HEALTH_DATA` |

**解决**: 在应用的 `module.json5` 中添加权限声明:

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCELEROMETER"
      }
    ]
  }
}
```

---

### 问题 5: 传感器不支持 (错误码 14500102)

**现象**:
```
BusinessException: The sensor is not supported by the device
```

**原因**: 设备不具备该传感器硬件

**解决**:
1. 先调用 `getSensorList()` 查询可用传感器
2. 检查目标传感器是否在列表中

```cj
let sensors = getSensorList()
for (sensor in sensors) {
    println("ID: ${sensor.sensorId}, Name: ${sensor.sensorName}")
}
```

---

### 问题 6: 服务异常 (错误码 14500101)

**现象**:
```
BusinessException: Service exception
```

**可能原因**:
1. Sensor HDF 服务异常
2. Sensor Service IPC 异常
3. 传感器数据通道异常

**排查步骤**:
1. 检查传感器驱动是否正常加载
2. 检查 Sensor Service 是否运行
3. 查看日志:
```bash
hilog | grep "CJ-Sensor"
```

---

### 问题 7: 回调未被调用

**现象**: 订阅后回调函数从未执行

**可能原因**:
1. 传感器不支持
2. 传感器未使能
3. 回调类型不匹配

**排查**:
1. 检查 `on()` 返回值是否成功
2. 确认回调类型正确:
```cj
// 错误: 使用了错误的 Response 类型
on(Accelerometer) { timestamp, response =>
    let acc = response.getOrThrow() as GyroscopeResponse  // 类型错误
}

// 正确: 使用匹配的 Response 类型
on(Accelerometer) { timestamp, response =>
    let acc = response.getOrThrow() as AccelerometerResponse
}
```

---

### 问题 8: once() 订阅后立即返回数据

**现象**: `once()` 调用后回调立即被触发多次

**说明**: 这是预期行为。如果已有持续订阅 (`on()`)，`once()` 会复用该订阅，回调会在每次数据推送时被调用。

**解决方案**: 如果需要严格单次获取，确保先调用 `off()` 取消所有订阅。

---

## 7.3 调试方法

### 日志查看

**启用日志**:

```cj
import ohos.log.*

// 使用 SENSOR_LOG 打印调试信息
SENSOR_LOG.info("Callback invoked with data")
```

**过滤日志**:

```bash
hilog | grep "CJ-Sensor"
```

---

### 断点调试

1. 使用 DevEco Studio 设置断点
2. 断点位置建议:
   - `sensor.cj:1653` (`on()` 入口)
   - `sensor_manager.cj:166` (`dataCallbackImpl`)
   - `ffi.cj:290` (FFI 函数)

---

### API 调用追踪

**启用 FFI 日志**:

在 `sensor.cj` 中添加调试输出:

```cj
public func on<T>(sensorType: SensorId, callback: Callback1Argument<T>, option!: ?Options = None): Unit {
    SENSOR_LOG.debug("on() called for sensorType: ${sensorType.getValue()}")
    // ...
}
```

---

## 7.4 性能问题

### 问题 9: 传感器数据回调频繁导致性能下降

**原因**: 采样频率设置过高

**解决**: 使用适当的 `IntervalOption`:

| 场景 | 推荐 | 频率 |
|------|------|------|
| 游戏 | GameMode | 50Hz |
| UI 交互 | UIMode | 16Hz |
| 后台监控 | NormalMode | 5Hz |

```cj
// 使用 UIMode 降低频率
on(Accelerometer, Option(interval: UIMode)) {
    // ...
}
```

---

### 问题 10: 内存占用过高

**可能原因**:
1. 大量未取消的订阅
2. 回调中创建了临时对象
3. 传感器数据未及时处理

**优化建议**:
1. 及时调用 `off()` 取消订阅
2. 避免在回调中创建大量对象
3. 使用 `once()` 替代持续订阅（如果适用）

---

## 7.5 迁移与兼容性

### API Level 兼容性

- **最低 API Level**: 22
- 检查 API 可用性:
```cj
// API Level >= 30 可用
if (sdk.getApiVersion() >= 30) {
    // 使用新 API
}
```

---

### ArkTS 差异

| 能力 | ArkTS | Cangjie |
|------|-------|---------|
| 订阅传感器 | 支持 | 支持 |
| 获取传感器列表 | 支持 | 支持 |
| 获取地磁数据 | 支持 | **不支持** |
| 基于气压获取海拔 | 支持 | **不支持** |

详见 [README.md#Constraints](../README.md#constraints)
