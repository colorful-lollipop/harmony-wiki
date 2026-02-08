# 编译产物

## 5.1 产物概览

### 产物清单

| 产物类型 | 目标名称 | 说明 |
|----------|----------|------|
| Cangjie 共享库 | `libohos.sensor.z.so` | OHOS 层接口库 |
| Cangjie 共享库 | `libkit.SensorServiceKit.z.so` | Kit 层接口库 |
| SDK 产物 | 见 SDK 复制任务 | 分发给开发者 |

### 产物与 Target 对应关系

```
Target: ohos.sensor
    └── 产物: libohos.sensor.z.so

Target: kit.SensorServiceKit
    └── 产物: libkit.SensorServiceKit.z.so

Target: copy_sdk_sensors_cangjie_libs
    └── 产物: SDK 包 (ohos.sensor)

Target: copy_sdk_sensors_cangjie_libs_kit
    └── 产物: SDK 包 (kit.SensorServiceKit)
```

---

## 5.2 产物详情

### libohos.sensor.z.so

**Target**: `//base/sensors/sensors_cangjie_wrapper/ohos/sensor:ohos.sensor`

**类型**: Cangjie 共享库 (.z.so)

**包含模块**:

| 模块 | 功能 |
|------|------|
| `ohos.sensor` | 传感器 API 入口 |
| `ohos.sensor.SensorId` | 传感器类型枚举 |
| `ohos.sensor.AccelerometerResponse` | 加速度响应 |
| `ohos.sensor.on()` | 订阅 API |
| `ohos.sensor.off()` | 取消订阅 API |
| `ohos.sensor.getSensorList()` | 获取传感器列表 |

**依赖关系**:

```
libohos.sensor.z.so
├── libohos.hilog.z.so (hiviewdfx_cangjie_wrapper)
├── libohos.business_exception.z.so (cangjie_ark_interop)
├── libohos.callback_invoke.z.so (cangjie_ark_interop)
└── libcj_sensor_ffi.so (sensor)
```

---

### libkit.SensorServiceKit.z.so

**Target**: `//base/sensors/sensors_cangjie_wrapper/kit/SensorServiceKit:kit.SensorServiceKit`

**类型**: Cangjie 共享库 (.z.so)

**包含模块**:

| 模块 | 功能 |
|------|------|
| `kit.SensorServiceKit` | Kit 命名空间 (重导出 ohos.sensor) |

**依赖关系**:

```
libkit.SensorServiceKit.z.so
└── libohos.sensor.z.so (重导出)
```

---

## 5.3 安装路径

### 系统路径

编译产物安装到 OpenHarmony 系统镜像的以下路径：

```
/system/lib/module/
├── libohos.sensor.z.so
└── libkit.SensorServiceKit.z.so
```

### SDK 路径

SDK 产物安装到以下路径：

```
prebuilts/sdk/
├── ohos/
│   ├── ohos.sensor/
│   │   ├── index.d.ts       # 类型定义
│   │   └── package.json     # 包配置
│   └── ...
└── kit/
    └── SensorServiceKit/
        ├── index.d.ts
        └── package.json
```

---

## 5.4 运行时加载

### 加载顺序

```
Cangjie 应用
    │
    ├── 加载 libkit.SensorServiceKit.z.so
    │       │
    │       └── 依赖 libohos.sensor.z.so
    │               │
    │               └── 依赖 libohos.hilog.z.so
    │               └── 依赖 libohos.business_exception.z.so
    │               └── 依赖 libcj_sensor_ffi.so (C++)
    │
    └── 运行时调用
            │
            ├── on() → FfiSensorSubscribeSensor() → IPC → Sensor SA
            ├── off() → FfiSensorUnSubscribeSensor()
            └── getSensorList() → FfiSensorGetAllSensors()
```

### 动态依赖

| 依赖库 | 加载时机 | 说明 |
|--------|----------|------|
| `libkit.SensorServiceKit.z.so` | 应用启动 | Kit API |
| `libohos.sensor.z.so` | 自动加载 | 依赖传递 |
| `libcj_sensor_ffi.so` | 运行时 | C++ FFI 实现 |
| `libohos.hilog.z.so` | 自动加载 | 日志框架 |

---

## 5.5 产物验证

### 文件检查

```bash
# 检查产物是否存在
ls -la out/.../system/lib/module/libohos.sensor.z.so
ls -la out/.../system/lib/module/libkit.SensorServiceKit.z.so

# 检查依赖
ldd out/.../system/lib/module/libohos.sensor.z.so
```

### 预期输出

```
libohos.sensor.z.so:
    libohos.hilog.z.so => ...
    libohos.business_exception.z.so => ...
    libcj_sensor_ffi.so => ...
    libpthread.so => ...
    libcjrt.so => ...
```

---

## 5.6 资源占用

### ROM 占用

| 产物 | 大小 (约) |
|------|-----------|
| libohos.sensor.z.so | ~80KB |
| libkit.SensorServiceKit.z.so | ~5KB |
| **总计** | **~160KB** |

### RAM 占用

| 组件 | 占用 (约) |
|------|-----------|
| 运行时库 | ~50KB |
| 传感器数据缓存 | ~40KB |
| 回调注册表 | ~10KB |
| 其他 | ~44KB |
| **总计** | **~144KB** |

---

## 5.7 调试产物

### 符号表

调试版本产物包含完整符号信息：

```
out/.../lib.unstripped/
├── libohos.sensor.z.so
└── libkit.SensorServiceKit.z.so
```

### 使用符号表

```bash
# 查看导出符号
nm -D out/.../lib.unstripped/libohos.sensor.z.so

# 示例输出:
# 0000000000001a30 T FfiSensorSubscribeSensor
# 0000000000001b20 T FfiSensorUnSubscribeSensor
# 0000000000001c10 T FfiSensorGetAllSensors
```
