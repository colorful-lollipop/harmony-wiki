# 安全风险评审

## 6.1 威胁模型

### 系统边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Cangjie 应用                          │   │
│  │  - 可调用 on()/off()/getSensorList() API               │   │
│  │  - 提供回调函数接收传感器数据                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │ API 调用                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │             Sensors Cangjie Wrapper (本项目)            │   │
│  │  - 类型转换与校验                                       │   │
│  │  - 回调注册表管理                                       │   │
│  │  - FFI 调用                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │ FFI 调用                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │               cj_sensor_ffi (C++)                       │   │
│  │  - IPC 通信                                             │   │
│  │  - 权限校验                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │ IPC                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Sensor Service (SA)                     │   │
│  │  - 传感器数据获取                                       │   │
│  │  - 权限最终校验                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │ HDF                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Sensor HDF Driver                       │   │
│  │  - 硬件访问                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 外部输入

| 输入源 | 数据类型 | 信任级别 |
|--------|----------|----------|
| 用户回调参数 | Callback1Argument\<T\> | 低 (来自应用) |
| 传感器类型枚举 | SensorId | 中 (编译时常量) |
| 订阅选项 | Options | 中 |
| 传感器数据 | Response | 高 (来自系统服务) |

---

## 6.2 攻击面清单

| 攻击面 | 接口 | 风险等级 |
|--------|------|----------|
| 回调参数校验 | `on()`, `once()` | 中 |
| 传感器类型转换 | `CSensorCallbackData.convertToResponseOption()` | 中 |
| 内存安全 | `CSensorArray` 操作 | 低 |
| 权限缺失 | 敏感传感器权限校验 | 高 |
| 竞态条件 | 回调注册表并发访问 | 低 |
| 资源耗尽 | 大量订阅/回调 | 中 |
| 回调伪造 | 回调对象类型检查 | 低 |

---

## 6.3 风险清单

### 🔴 高风险

#### 风险 1: 敏感传感器权限校验不完整

**证据**: `ohos/sensor/sensor.cj:38-43`

```cj
@!APILevel[
    since: "22",
    permission: "ohos.permission.ACCELEROMETER",
    syscap: "SystemCapability.Sensors.Sensor"
]
Accelerometer
```

**问题描述**:
- `@!APILevel` 注解中的 `permission` 字段定义了权限要求
- 但权限的实际校验在底层 `cj_sensor_ffi` (C++) 中实现
- Cangjie 层仅通过注解声明，未进行运行时强校验

**触发条件**:
1. 开发者未申请 `ohos.permission.ACCELEROMETER`
2. 直接调用 `on(Accelerometer, callback)`
3. 若底层校验失败，抛出 `BusinessException(201)`

**影响**: 权限声明与实际校验分离，可能导致:
- 文档与实现不一致
- 权限注解遗漏时难以发现

**修复建议**:
- 在 Cangjie 层添加显式的权限检查
- 或者确保文档明确标注权限声明位置

---

#### 风险 2: 传感器类型枚举解析越界

**证据**: `ohos/sensor/sensor.cj:191-221`

```cj
func convertToResponseOption(): ?Response {
    try {
        match (sensorTypeId) {
            case 1 => AccelerometerResponse(this)
            case 2 => GyroscopeResponse(this)
            // ... 已支持类型
            case _ => None  // 未匹配返回 None
        }
    } catch (e: Exception) {
        SENSOR_LOG.error("Fail to convert callback data, ${e}")
        None
    }
}
```

**问题描述**:
- 当 `sensorTypeId` 为未知值时，返回 `None` 而非抛出异常
- 调用方需要处理 `None` 情况，否则可能导致静默失败

**触发条件**:
1. FFI 回调收到未知 `sensorTypeId`
2. `convertToResponseOption()` 返回 `None`
3. `dataCallbackImpl()` 中 `responseData` 为 `None`
4. 部分回调可能被静默丢弃

**影响**:
- 传感器数据可能丢失
- 难以调试的问题（静默失败）

**修复建议**:
```cj
// 修改为抛出异常而非返回 None
case _ => throw BusinessException(14500101, "Unknown sensor type: ${sensorTypeId}")
```

---

### 🟡 中风险

#### 风险 3: 回调注册表竞态条件

**证据**: `ohos/sensor/sensor_manager.cj:36-51`

```cj
func on(target: CallbackObject): Unit {
    synchronized(callBackMutex) {
        for (callback in callbackList) {
            if (refEq(callback, target)) {
                SENSOR_LOG.info("The callback has been subscribed.")
                return
            }
        }
        callbackList.add(target)
    }
}
```

**问题描述**:
- 使用 `synchronized()` 提供互斥访问
- 但 `synchronized` 仅保护列表操作，不保护迭代器

**触发条件** (理论风险):
1. 线程 A: 遍历 `callbackList`
2. 线程 B: `off()` 移除回调
3. 可能导致迭代器失效

**实际影响**: Cangjie 运行时的 `synchronized` 实现提供了足够的安全保障

**修复建议**:
- 当前实现已足够安全
- 建议添加迭代时的拷贝保护

---

#### 风险 4: 内存安全 - C 指针直接读取

**证据**: `ohos/sensor/ffi.cj:273-276`

```cj
func asArray(): Array<Sensor> {
    if (head.isNull() || size <= 0) {
        return Array<Sensor>()
    }
    Array<Sensor>(size) {
        i =>
        let data = unsafe { head.read(i) }  // 直接读取 C 指针
        Sensor(data)
    }
}
```

**问题描述**:
- 使用 `unsafe` 块直接读取 C 指针
- 若指针越界或无效，可能导致内存访问错误

**触发条件**:
1. `head` 指针无效
2. `size` 与实际数据不匹配
3. FFI 层返回错误数据

**影响**:
- 潜在崩溃风险
- 数据损坏

**修复建议**:
```cj
func asArray(): Array<Sensor> {
    if (head.isNull() || size <= 0) {
        return Array<Sensor>()
    }
    // 添加边界检查
    let safeSize = if (size > MAX_SENSOR_COUNT) MAX_SENSOR_COUNT else size
    Array<Sensor>(safeSize) {
        i =>
        // ...
    }
}
```

---

#### 风险 5: 资源耗尽 - 大量回调注册

**证据**: `ohos/sensor/sensor_manager.cj:42-51`

```cj
func on(target: CallbackObject): Unit {
    synchronized(callBackMutex) {
        for (callback in callbackList) {
            if (refEq(callback, target)) {
                SENSOR_LOG.info("The callback has been subscribed.")
                return
            }
        }
        callbackList.add(target)  // 无数量限制
    }
}
```

**问题描述**:
- 对单个传感器类型的回调数量没有限制
- 恶意应用可能注册大量回调

**触发条件**:
1. 循环调用 `on()` 注册同一传感器
2. 不调用 `off()` 清理

**影响**:
- 内存占用增加
- 传感器数据推送性能下降
- 潜在的 DoS 攻击

**修复建议**:
```cj
let MAX_CALLBACKS_PER_SENSOR = 16

func on(target: CallbackObject): Unit {
    synchronized(callBackMutex) {
        if (callbackList.size >= MAX_CALLBACKS_PER_SENSOR) {
            throw BusinessException(14500101, "Too many callbacks")
        }
        // ...
    }
}
```

---

#### 风险 6: 错误码信任边界穿越

**证据**: `ohos/sensor/sensor.cj:1661-1663`

```cj
let ret: Int32 = unsafe { FfiSensorSubscribeSensor(...) }
if (ret != SUCCESS_CODE) {
    throw BusinessException(ret, getErrorMsg(ret))
}
```

**问题描述**:
- FFI 返回的错误码直接用于创建异常
- 若 FFI 层返回恶意构造的错误码

**触发条件**:
1. FFI 层返回超大错误码 (如 `INT_MAX`)
2. `getErrorMsg()` 可能返回 "Unknown error code X"

**影响**:
- 异常信息泄露
- 潜在的整数溢出

**修复建议**:
```cj
let ret: Int32 = unsafe { FfiSensorSubscribeSensor(...) }
if (ret != SUCCESS_CODE) {
    // 验证错误码范围
    if (ret < 0 || ret > MAX_ERROR_CODE) {
        throw BusinessException(14500101, "Invalid error code from FFI")
    }
    throw BusinessException(ret, getErrorMsg(ret))
}
```

---

### 🟢 低风险

#### 风险 7: 传感器类型 ID 硬编码

**证据**: `ohos/sensor/sensor.cj:248-270`

```cj
public func getValue(): Int32 {
    match (this) {
        case Accelerometer => 1
        case Gyroscope => 2
        // ...
    }
}
```

**说明**: 传感器类型 ID 与系统服务约定，硬编码是合理做法。当前实现安全。

---

## 6.4 安全建议总结

### 优先级排序

| 优先级 | 风险 | 建议 |
|--------|------|------|
| P0 | 权限校验 | 添加显式权限检查 |
| P1 | 类型解析 | 将静默失败改为异常 |
| P1 | 资源耗尽 | 添加回调数量限制 |
| P2 | C 指针安全 | 添加边界检查 |
| P2 | 错误码 | 验证 FFI 返回的错误码 |

### 纵深防御

```
┌─────────────────────────────────────────┐
│  应用层 - 回调注册数量限制                │
├─────────────────────────────────────────┤
│  Wrapper 层 - 类型转换异常               │
├─────────────────────────────────────────┤
│  FFI 层 - 权限校验、参数验证             │
├─────────────────────────────────────────┤
│  SA 层 - 最终权限校验、数据验证          │
├─────────────────────────────────────────┤
│  Driver 层 - 硬件访问控制               │
└─────────────────────────────────────────┘
```

---

## 6.5 检查范围说明

### 已检查范围

| 范围 | 深度 | 说明 |
|------|------|------|
| `ohos/sensor/*.cj` | 完整 | 所有 Cangjie 源码 |
| `kit/SensorServiceKit/*.cj` | 完整 | Kit 层源码 |
| `BUILD.gn` | 完整 | 构建配置 |
| `bundle.json` | 完整 | 包配置 |

### 未检查范围

| 范围 | 原因 |
|------|------|
| 底层 FFI 实现 | 位于 `sensors_sensor` 仓库 |
| Sensor Service (SA) | 独立系统服务 |
| HDF Driver | 内核驱动层 |
| Cangjie 运行时 | 编译器运行时库 |

### 局限性

1. **静态分析**: 本评审基于静态代码分析，未运行时测试
2. **依赖信任**: 假设底层 FFI 和 SA 的安全机制有效
3. **并发测试**: 未进行高并发场景测试
