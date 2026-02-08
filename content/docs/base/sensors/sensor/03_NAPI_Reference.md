# 对外 N-API 完整参考

> **目的**: 描述 Sensor 子系统的对外 JavaScript N-API，包括 API 清单表、参数校验和调用链
> **适用范围**: /base/sensors/sensor/frameworks/js/napi/（排除 test/ 目录）
> **关键结论**: Sensor N-API 导出 46 个方法和 3 个枚举类，支持多种传感器类型的订阅和数据获取
> **相关跳转**: [项目概览](00_Overview.md) | [目录结构](01_Directory_Structure.md)

---

## N-API 模块注册

### 注册点

**文件**: `frameworks/js/napi/src/sensor_js.cpp`

**模块注册**:
```cpp
extern "C" __attribute__((constructor)) SensorModule GetModule()
{
    static napi_module _module = {
        .nm = "sensor",
        .version = 1,
    };
    _module.get = Init;  // Init 函数定义导出属性
    return &_module;
}
napi_module_register(&_module);
```

**Init 函数**:
```cpp
static napi_value Init(napi_env env, napi_value exports)
{
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("on"),  // 注册 on 方法
        DECLARE_NAPI_FUNCTION("once"),  // 注册 once 方法
        DECLARE_NAPI_FUNCTION("off"),  // 注册 off 方法
        // ... 其他方法
        DECLARE_NAPI_FUNCTION("subscribeAccelerometer"),  // 订阅方法
        DECLARE_NAPI_FUNCTION("subscribeGyroscope"),  // 订阅方法
        // ... 其他订阅方法
    };
    napi_define_properties(env, exports, sizeof(desc)/sizeof(napi_property_descriptor), desc);
    // 创建枚举类
    CreateEnumSensorType(env, exports);
    CreateEnumSensorId(env, exports);
    CreateEnumSensorAccuracy(env, exports);
}
```

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:870-880`

---

## JS API 清单表

### 核心订阅方法

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `on(type, callback, options)` | type: SensorType, callback: Callback<Response>, options?: Options | void | 异步 | `On()` | sensor_js.cpp:599 | 持续订阅传感器数据 |
| `once(type, callback)` | type: SensorType, callback: Callback<Response> | void | 异步 | `Once()` | sensor_js.cpp: | 单次订阅传感器数据 |
| `off(type, callback)` | type: SensorType, callback?: Callback<Response> | void | 异步 | `Off()` | sensor_js.cpp: | 取消订阅传感器数据 |

> **TODO**: 补充具体行号

### 运动类传感器订阅/取消

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `subscribeAccelerometer(callback, options)` | callback, options | void | 异步 | `SubscribeAccelerometer()` | sensor_js.cpp: | 订阅加速度传感器 |
| `unsubscribeAccelerometer()` | - | void | 异步 | `UnsubscribeAccelerometer()` | sensor_js.cpp: | 取消订阅加速度传感器 |
| `subscribeGyroscope(callback, options)` | callback, options | void | 异步 | `SubscribeGyroscope()` | sensor_js.cpp: | 订阅陀螺仪 |
| `unsubscribeGyroscope()` | - | void | 异步 | `UnsubscribeGyroscope()` | sensor_js.cpp: | 取消订阅陀螺仪 |
| `subscribeGravity(callback, options)` | callback, options | void | 异步 | `SubscribeGravity()` | sensor_js.cpp: | 订阅重力传感器 |
| `unsubscribeGravity()` | - | void | 异步 | `UnsubscribeGravity()` | sensor_js.cpp: | 取消订阅重力传感器 |
| `subscribeMagnetic(callback, options)` | callback, options | void | 异步 | `SubscribeMagnetic()` | sensor_js.cpp: | 订阅磁力计 |
| `unsubscribeMagnetic()` | - | void | 异步 | `UnsubscribeMagnetic()` | sensor_js.cpp: | 取消订阅磁力计 |

> **TODO**: 补充具体行号

### 姿态类传感器订阅/取消

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `subscribeDeviceOrientation(callback, options)` | callback, options | void | 异步 | `SubscribeDeviceOrientation()` | sensor_js.cpp: | 订阅设备方向传感器 |
| `unsubscribeDeviceOrientation()` | - | void | 异步 | `UnsubscribeDeviceOrientation()` | sensor_js.cpp: | 取消订阅设备方向传感器 |
| `subscribeHall(callback, options)` | callback, options | void | 异步 | `SubscribeHall()` | sensor_js.cpp: | 订阅霍尔传感器 |
| `unsubscribeHall()` | - | void | 异步 | `UnsubscribeHall()` | sensor_js.cpp: | 取消订阅霍尔传感器 |

> **TODO**: 补充具体行号

### 环境类传感器订阅/取消

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `subscribeLight(callback, options)` | callback, options | void | 异步 | `SubscribeLight()` | sensor_js.cpp: | 订阅环境光传感器 |
| `unsubscribeLight()` | - | void | 异步 | `UnsubscribeLight()` | sensor_js.cpp: | 取消订阅环境光传感器 |
| `subscribeBarometer(callback, options)` | callback, options | void | 异步 | `SubscribeBarometer()` | sensor_js.cpp: | 订阅气压传感器 |
| `unsubscribeBarometer()` | - | void | 异步 | `UnsubscribeBarometer()` | sensor_js.cpp: | 取消订阅气压传感器 |

> **TODO**: 补充具体行号

### 其他传感器订阅/取消

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `subscribeProximity(callback, options)` | callback, options | void | 异步 | `SubscribeProximity()` | sensor_js.cpp: | 订阅接近光传感器 |
| `unsubscribeProximity()` | - | void | 异步 | `UnsubscribeProximity()` | sensor_js.cpp: | 取消订阅接近光传感器 |
| `subscribeStepCounter(callback, options)` | callback, options | void | 异步 | `SubscribeStepCounter()` | sensor_js.cpp: | 订阅计步器 |
| `unsubscribeStepCounter()` | - | void | 异步 | `UnsubscribeStepCounter()` | sensor_js.cpp: | 取消订阅计步器 |
| `subscribeHeartRate(callback, options)` | callback, options | void | 异步 | `SubscribeHeartRate()` | sensor_js.cpp: | 订阅心率传感器 |
| `unsubscribeHeartRate()` | - | void | 异步 | `UnsubscribeHeartRate()` | sensor_js.cpp: | 取消订阅心率传感器 |
| `subscribeOnBodyState(callback, options)` | callback, options | void | 异步 | `SubscribeOnBodyState()` | sensor_js.cpp: | 订阅身体状态传感器 |
| `unsubscribeOnBodyState()` | - | void | 异步 | `UnsubscribeOnBodyState()` | sensor_js.cpp: | 取消订阅身体状态传感器 |
| `getOnBodyState()` | - | BodyState | 同步 | `GetOnBodyState()` | sensor_js.cpp: | 获取身体状态 |

> **TODO**: 补充具体行号

### 几何计算方法

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `getGeomagneticField(geomagneticOption)` | GeomagneticOption | GeomagneticField | 同步 | `GetGeomagneticField()` | sensor_js.cpp: | 获取地磁场数据 |
| `getGeomagneticInfo(geomagneticInfo)` | GeomagneticInfo | GeomagneticInfo | 同步 | `GetGeomagneticInfo()` | sensor_js.cpp: | 获取地磁场信息 |
| `transformCoordinateSystem(in, out)` | in: RotationMatrix, out: RotationMatrix | void | 同步 | `TransformCoordinateSystem()` | sensor_js.cpp: | 转换坐标系 |
| `transformRotationMatrix(in, out)` | in: RotationVector, out: RotationMatrix | void | 同步 | `TransformRotationMatrix()` | sensor_js.cpp: | 转换旋转矩阵 |
| `getAngleModify()` | - | float | 同步 | `GetAngleModify()` | sensor_js.cpp: | 获取角度修正值 |
| `getAngleVariation()` | - | float | 同步 | `GetAngleVariation()` | sensor_js.cpp: | 获取角度变化值 |
| `getDirection()` | - | float | 同步 | `GetDirection()` | sensor_js.cpp: | 获取方向值 |
| `getOrientation()` | - | Orientation | 同步 | `GetOrientation()` | sensor_js.cpp: | 获取方向值 |
| `createQuaternion(x, y, z)` | x, y, z: float | Quaternion | 同步 | `CreateQuaternion()` | sensor_js.cpp: | 创建四元数 |
| `getQuaternion()` | - | Quaternion | 同步 | `GetQuaternion()` | sensor_js.cpp: | 获取四元数 |
| `getAltitude()` | - | float | 同步 | `GetAltitude()` | sensor_js.cpp: | 获取高度值 |
| `getDeviceAltitude()` | - | float | 同步 | `GetDeviceAltitude()` | sensor_js.cpp: | 获取设备高度值 |
| `getGeomagneticDip()` | - | float | 同步 | `GetGeomagneticDip()` | sensor_js.cpp: | 获取地磁倾角 |
| `getInclination()` | - | float | 同步 | `GetInclination()` | sensor_js.cpp: | 获取磁倾角 |
| `createRotationMatrix()` | - | RotationMatrix | 同步 | `CreateRotationMatrix()` | sensor_js.cpp: | 创建旋转矩阵 |
| `getRotationMatrix()` | - | RotationMatrix | 同步 | `GetRotationMatrix()` | sensor_js.cpp: | 获取旋转矩阵 |

> **TODO**: 补充具体行号

### 传感器状态监听

| JS API 名称 | 参数 | 返回值 | 同步/异步 | C++ 实现函数 | 文件位置 |
|-------------|------|--------|----------|---------------|----------|
| `onPlugSensor(type, callback)` | type: "sensorStatusChange", callback | void | 异步 | `OnPlugSensor()` | sensor_js.cpp:525 | 监听传感器插拔事件 |
| `offPlugSensor(type, callback)` | type: "sensorStatusChange", callback | void | 异步 | `OffPlugSensor()` | sensor_js.cpp: | 取消监听传感器插拔事件 |

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:525-541`

---

## 枚举类

### SensorType 枚举

**位置**: `CreateEnumSensorType(env, exports)`

**主要传感器类型**:

| 枚举值 | 说明 | 对应 SensorTypeId | 权限要求 |
|---------|------|------------------|----------|
| `ACCELEROMETER` | 加速度传感器 | SENSOR_TYPE_ID_ACCELEROMETER | ohos.permission.ACCELEROMETER |
| `GYROSCOPE` | 陀螺仪 | SENSOR_TYPE_ID_GYROSCOPE | ohos.permission.GYROSCOPE |
| `AMBIENT_LIGHT` | 环境光传感器 | SENSOR_TYPE_ID_AMBIENT_LIGHT | 无 |
| `ORIENTATION` | 方向传感器 | SENSOR_TYPE_ID_ORIENTATION | 无 |
| `GRAVITY` | 重力传感器 | SENSOR_TYPE_ID_GRAVITY | 无 |
| `LINEAR_ACCELEROMETER` | 线性加速度 | SENSOR_TYPE_ID_LINEAR_ACCELERATION | ohos.permission.ACCELEROMETER |
| `ROTATION_VECTOR` | 旋转矢量 | SENSOR_TYPE_ID_ROTATION_VECTOR | 无 |
| `MAGNETIC_FIELD` | 磁力计 | SENSOR_TYPE_ID_MAGNETIC_FIELD | 无 |
| `BAROMETER` | 气压传感器 | SENSOR_TYPE_ID_BAROMETER | 无 |
| `HEART_RATE` | 心率传感器 | SENSOR_TYPE_ID_HEART_RATE | ohos.permission.READ_HEALTH_DATA |
| `PEDOMETER` | 计步器 | SENSOR_TYPE_ID_PEDOMETER | ohos.permission.ACTIVITY_MOTION |
| `PROXIMITY` | 接近光传感器 | SENSOR_TYPE_ID_PROXIMITY | 无 |
| `HALL` | 霍尔传感器 | SENSOR_TYPE_ID_HALL | 无 |
| `DEVICE_ORIENTATION` | 设备方向 | SENSOR_TYPE_ID_DEVICE_ORIENTATION | 无 |

> **证据**: `frameworks/js/napi/src/sensor_js.cpp` (CreateEnumSensorType 函数)

### SensorId 枚举

**位置**: `CreateEnumSensorId(env, exports)`

**说明**: 传感器 ID 枚举，用于标识具体传感器实例

> **证据**: `frameworks/js/napi/src/sensor_js.cpp`

### SensorAccuracy 枚举

**位置**: `CreateEnumSensorAccuracy(env, exports)`

**精度等级**:

| 枚举值 | 说明 | 数值 |
|---------|------|------|
| `ACCURACY_UNRELIABLE` | 不可靠 | 0 |
| `ACCURACY_LOW` | 低精度 | 1 |
| `ACCURACY_MEDIUM` | 中等精度 | 2 |
| `ACCURACY_HIGH` | 高精度 | 3 |

> **证据**: `frameworks/js/napi/src/sensor_js.cpp` (CreateEnumSensorAccuracy 函数)

---

## 参数校验

### type 参数校验

**校验内容**:
- 类型是否为有效的 SensorType/SensorId 枚举值
- 是否为支持的传感器类型

**校验位置**: `On()` 函数中

> **TODO**: 补充具体行号和校验逻辑

### callback 参数校验

**校验内容**:
- 是否为有效的 JavaScript 函数
- 是否为 null 或 undefined

**校验位置**: `On()` 函数中

> **TODO**: 补充具体行号和校验逻辑

### options 参数校验

**校验内容**:
- interval 是否为有效数值（纳秒）
- 采样间隔是否在合理范围内
- 是否包含 sensorInfoParam（设备 ID 和传感器 ID）

**校验位置**: `GetOptionalParameter()` 函数

**采样间隔预设**:
- `normal`: 200000000 ns (200 ms)
- `ui`: 60000000 ns (60 ms)
- `game`: 20000000 ns (20 ms)

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:55-59,569-597`

### 几何计算方法参数校验

**校验内容**:
- 输入参数类型检查（矩阵、矢量等）
- 数值范围检查
- 维度验证（如旋转矩阵必须是 3x3）

**校验位置**: 各几何计算函数中

> **TODO**: 补充具体行号和校验逻辑

---

## 异步模式

### on/once/off 方法

**模式**: 异步回调

**回调执行**:
- 通过 `AsyncCallbackInfo` 结构管理异步回调
- 回调在 N-API 线程中执行
- 数据通过 IPC 从服务端异步传递到客户端

> **TODO**: 补充异步回调机制的详细说明

### subscribeXxx 方法

**模式**: 异步订阅

**订阅流程**:
1. 参数校验
2. 创建 AsyncCallbackInfo
3. 调用 Native SubscribeSensor
4. 将回调信息保存到全局映射
5. 返回成功

**取消订阅流程**:
1. 查找对应的订阅信息
2. 调用 Native UnsubscribeSensor
3. 清理回调引用
4. 从全局映射中删除

> **TODO**: 补充具体行号和流程

---

## 调用链

### on(type) 调用链

```mermaid
graph TD
    A[JS应用] -->|调用 sensor.on|
    B[N-API层<br/>sensor_js.cpp] -->|参数校验|
    C[Native客户端<br/>sensor_agent.cpp] -->|创建订阅请求|
    D[SensorServiceClient<br/>IPC 客户端] -->|IPC 调用|
    E[SensorService<br/>libsensor_service.z.so] -->|权限检查|
    F{权限检查<br/>PermissionUtil} -->|调用 AccessTokenKit|
    G[HDI连接<br/>sensor_hdi_connection.cpp] -->|启用硬件|
    H[物理传感器] -->|采集数据|
    I[数据回调<br/>SensorDataCallback] -->|IPC 传输|
    J[N-API 回调] -->|执行 JS callback|
    K[应用] -->|接收传感器数据|
```

**关键节点**:
- **N-API 层**: `sensor_js.cpp::On()`
- **Native 客户端**: `sensor_agent.cpp::SubscribeSensor()`
- **IPC 客户端**: `sensor_service_client.cpp`
- **Sensor 服务**: `sensor_service.cpp::EnableSensor()`
- **权限检查**: `permission_util.cpp::CheckSensorPermission()`
- **HDI 连接**: `hdi_connection.cpp`
- **回调执行**: `sensor_js.cpp` 中的异步回调机制

> **TODO**: 添加具体行号引用

### subscribeAccelerometer() 调用链

```mermaid
graph TD
    A[JS应用] -->|调用 subscribeAccelerometer|
    B[N-API层] -->|创建特定传感器订阅|
    C[Native客户端] -->|调用 SubscribeSensor<br/>sensorTypeId=ACCELEROMETER|
    D[SensorServiceClient] -->|IPC 调用|
    E[SensorService] -->|检查权限<br/>ACCELEROMETER|
    F{权限通过} -->|启用加速度传感器|
    G[HDI连接] -->|启用硬件|
    H[加速度传感器] -->|采集数据|
    I[数据回调] -->|IPC 传输|
    J[N-API 回调] -->|执行 JS callback<br/>data: AccelerometerData|
    K[应用] -->|接收加速度数据|
```

> **TODO**: 补充具体行号引用

### off(type) 调用链

```mermaid
graph TD
    A[JS应用] -->|调用 sensor.off|
    B[N-API层] -->|参数校验|
    C[Native客户端] -->|查找订阅信息|
    D[SensorServiceClient] -->|IPC 调用 DisableSensor|
    E[SensorService] -->|停止数据上报|
    F[HDI连接] -->|禁用硬件|
    G[传感器] -->|停止采集|
    H[SensorService] -->|清理客户端资源|
    I[N-API层] -->|清理回调引用|
    J[应用] -->|返回成功|
```

> **TODO**: 补充具体行号引用

---

## 错误码

### N-API 错误码

**位置**: `frameworks/js/napi/include/sensor_napi_error.h`

**主要错误** (待补充完整列表):

| 错误码 | 含义 | 说明 |
|---------|------|------|
| `PERMISSION_DENIED` | 权限拒绝 | 缺少必要的权限 |
| `PARAMETER_ERROR` | 参数错误 | 参数类型或值不正确 |
| `SENSOR_NATIVE_GET_SERVICE_ERR` | 获取服务失败 | Sensor 服务不可用 |
| `SENSOR_SUBSCRIBE_FAILURE` | 订阅失败 | 订阅传感器失败 |

> **TODO**: 补充完整错误码表

### Native 错误码

**位置**: `interfaces/inner_api/sensor_errors.h`

**主要错误**:

| 错误码 | 含义 | 说明 |
|---------|------|------|
| `ERR_OK` | 成功 | 操作成功 |
| `PERMISSION_DENIED` | 权限拒绝 | 缺少必要的权限 |
| `PARAMETER_ERROR` | 参数错误 | 参数类型或值不正确 |
| `SENSOR_NATIVE_GET_SERVICE_ERR` | 获取服务失败 | Sensor 服务不可用 |

> **证据**: `interfaces/inner_api/sensor_errors.h`

---

## 权限要求

### 传感器权限映射

| 传感器类型 | 权限名称 | 敏感度 | 检查位置 |
|----------|----------|--------|----------|
| 加速度传感器（所有类型） | ohos.permission.ACCELEROMETER | system_grant | `permission_util.cpp::CheckSensorPermission()` |
| 陀螺仪（所有类型） | ohos.permission.GYROSCOPE | system_grant | `permission_util.cpp::CheckSensorPermission()` |
| 计步器、活动检测 | ohos.permission.ACTIVITY_MOTION | user_grant | `permission_util.cpp::CheckSensorPermission()` |
| 心率传感器 | ohos.permission.READ_HEALTH_DATA | user_grant | `permission_util.cpp::CheckSensorPermission()` |

> **证据**: `utils/common/src/permission_util.cpp:18-30`

### 权限检查流程

```
JS API 调用
    ↓
N-API 层
    ↓
Native 客户端
    ↓
IPC 调用 SensorService
    ↓
服务端权限检查
    ↓
AccessTokenKit::VerifyAccessToken()
    ↓
权限状态查询
    ↓
返回结果（允许/拒绝）
```

> **证据**: `services/src/sensor_service.cpp:596-622`

---

## 相关跳转

- [项目概览](00_Overview.md) - 查看传感器类型和权限列表
- [内部 API](04_Internal_API.md) - 查看 Native 接口定义
- [安全风险评审](07_Security_Audit.md) - 查看权限安全机制
