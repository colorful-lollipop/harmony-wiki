# 模块边界与核心能力

## 目的

定义 thermal_manager 模块的边界、核心能力、运行环境和关键概念。

## 适用范围

- OpenHarmony thermal_manager 模块
- 标准系统类型 (standard system type)

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构

---

## 模块边界

### 职责范围

| 模块 | 职责 | 不负责 |
|---|---|---|
| **Thermal Manager (N-API)** | 提供温度查询和回调 JS API | 底层热管理策略实现 |
| **Thermal Service** | 核心温度控制（检测、决策、执行） | N-API 接口层、应用业务逻辑 |
| **Thermal Protector** | 非运行态热控制 | 运行态热管理 |
| **Thermal HDI** | 驱动信息上报、指令下发 | 驱动实现细节 |

### 依赖方向

```
Applications (N-API Consumer)
        ↓
    N-API Layer (thermal)
        ↓ (IPC - Binder)
    Thermal Service (SA 3303)
        ↓ (HDI)
    Thermal Drivers (HDF)
```

**关键约束**:
- ✅ 无环依赖
- ✅ 单向依赖（上层依赖下层）
- ✅ Thermal Service 不依赖 N-API

---

## 核心能力

### 1. 温度查询

| 能力 | 描述 | API | 调用者类型 |
|---|---|---|---|
| 查询热级别 | 获取当前设备热级别 | `getThermalLevel()` / `getLevel()` | 应用、其他子系统 |
| 查询传感器温度 | 获取特定传感器温度值 | `GetThermalSensorTemp()` | Thermal Service 内部 |

**证据**: `frameworks/napi/thermal_manager_napi.cpp:193`

### 2. 回调订阅

| 回调类型 | 描述 | 接口 |
|---|---|---|
| 热级别回调 | 热级别变化时通知订阅者 | `IThermalLevelCallback` |
| 温度回调 | 特定传感器温度变化时通知 | `IThermalTempCallback` |
| 动作回调 | 动作执行/状态变化时通知 | `IThermalActionCallback` |

**证据**:
- `interfaces/inner_api/native/include/ithermal_level_callback.h:25`
- `interfaces/inner_api/native/include/ithermal_temp_callback.h:25`

### 3. 策略执行

| 策略类型 | 描述 |
|---|---|
| 级别策略 | 根据温度传感器级别决策热级别 |
| 状态策略 | 根据设备状态（屏幕、充电）调整策略 |
| 动作策略 | 执行相应的热动作（降频、限流、关机） |

**证据**: `services/native/include/thermal_policy/thermal_policy.h:40`

### 4. 动作执行

支持的动作类型：

| 动作 | 实现类 | 功能 |
|---|---|---|
| CPU 频率控制 | `ActionCpuBig`, `ActionCpuMed`, `ActionCpuLit`, `ActionCpuBoost`, `ActionCpuIsolate`, `ActionCpuNonvip` | CPU 降频、核心隔离、提升控制 |
| GPU 频率控制 | `ActionGpu` | GPU 降频 |
| 电压限制 | `ActionVoltage` | 调节供电电压 |
| 电流限制 | `ActionCharger` | 限制充电电流 |
| 显示控制 | `ActionDisplay` | 调节屏幕亮度 |
| 音量控制 | `ActionVolume` | 降低音量 |
| 应用进程限制 | `ActionApplicationProcess` | 限制应用进程 |
| 飞行模式 | `ActionAirplane` | 启用/禁用飞行模式 |
| 关机 | `ActionShutdown` | 执行关机 |
| 弹窗警告 | `ActionPopup` | 显示热警告弹窗 |
| 热级别设置 | `ActionThermalLevel` | 设置系统热级别 |
| 节点控制 | `ActionNode` | sysfs 节点控制 |

**证据**: `services/BUILD.gn:42-61`

---

## 运行环境

### 进程模型

| 组件 | 进程名 | 用户空间/内核空间 |
|---|---|---|
| Thermal Service | powermgr | 用户空间 |
| Thermal Drivers | - | 内核空间 (通过 HDF) |
| Thermal Protector | thermal_protector (独立进程) | 用户空间 |

### 线程模型

**证据来源**: `services/native/include/thermal_observer/thermal_observer.h:101-106`

| 组件 | 线程模型 |
|---|---|
| ThermalObserver | 使用 FFRT (Foundation Function Runtime) 队列 |
| Callback 通知 | 使用 napi_send_event 异步通知 JS |
| Policy 执行 | 主线程同步执行 |

**关键常量**:
```cpp
FFRTQueue g_queue("thermal_service");  // thermal_service.cpp:58
```

### 内存模型

| 区域 | 负责组件 | 大小限制 |
|---|---|---|
| 应用内存 | Thermal Service | ~2048KB (bundle.json) |
| 共享内存 | IPC 通信 | Binder 驱动 |

---

## 权限模型

### 权限检查

**证据**: `services/native/src/thermal_service.cpp:446, 459, 503, 517, 573, 584, 738, 756`

| API | 权限要求 | 检查方式 |
|---|---|---|
| `SubscribeThermalTempCallback` | 系统应用 | `Permission::IsSystem()` |
| `UnSubscribeThermalTempCallback` | 系统应用 | `Permission::IsSystem()` |
| `SubscribeThermalActionCallback` | 系统应用 | `Permission::IsSystem()` |
| `UnSubscribeThermalActionCallback` | 系统应用 | `Permission::IsSystem()` |
| `SetScene` | 系统应用 | `Permission::IsSystem()` |
| `UpdateThermalState` | 系统应用 | `Permission::IsSystem()` |
| `Dump` | 系统应用 + BootCompleted | `Permission::IsSystem()` |
| `GetThermalLevel` | 无 | 无检查 |
| `SubscribeThermalLevelCallback` | 无 | 无检查 |

### 权限检查实现

```cpp
// services/native/src/thermal_service.cpp:446
int32_t ThermalService::SubscribeThermalTempCallback(
    const std::vector<std::string>& typeList, const sptr<IThermalTempCallback>& callback)
{
    if (!Permission::IsSystem()) {
        THERMAL_HILOGE(COMP_SVC, "Permission::IsSystem() failed");
        return ERR_FAIL;
    }
    auto uid = IPCSkeleton::GetCallingUid();
    // ... 订阅逻辑
}
```

**调用者识别**:
- `IPCSkeleton::GetCallingUid()` - 获取调用者 UID
- `IPCSkeleton::GetCallingPid()` - 获取调用者 PID

---

## 系统能力

### 系统能力声明

- **系统能力 ID**: `SystemCapability.PowerManager.ThermalManager`
- **子系统**: powermgr
- **组件**: thermal_manager

**证据**: `bundle.json:18`

---

## 关键概念

### 1. Sensor Cluster（传感器集群）

将多个传感器组合为一个集群，用于级别决策。

**证据**: `services/native/include/thermal_policy/thermal_config_sensor_cluster.h`

```xml
<sensor_cluster name="warm_base" sensor="shell">
    <item level="1" threshold="35000" threshold_clr="33000"/>
    <item level="2" threshold="37000" threshold_clr="35000"/>
</sensor_cluster>
```

### 2. Thermal Level（热级别）

0-7 的枚举值，表示设备热状态。

| 级别 | 值 | 触发动作示例 |
|---|---|---|
| COOL | 0 | 无动作，正常运行 |
| NORMAL | 1 | 可能轻微限频 |
| WARM | 2 | CPU/GPU 降频 |
| HOT | 3 | 显著限流，降低亮度 |
| OVERHEATED | 4 | 激进限制，显示警告 |
| WARNING | 5 | 应用进程限制 |
| EMERGENCY | 6 | 准备关机 |
| ESCAPE | 7 | 执行关机 |

**证据**: `frameworks/napi/thermal_manager_napi.cpp:150-157`

### 3. State Machine（状态机）

根据设备状态（屏幕、充电等）调整热策略。

**状态类型**:
- 屏幕状态：SCREEN_ON / SCREEN_OFF
- 充电状态：CHARGING / NOT_CHARGING
- 启动延迟：STARTUP_DELAY
- 场景状态：SCENE

**证据**: `services/native/include/thermal_observer/state_machine/`

### 4. Mock 仿真模式

用于测试的温度仿真模式。

**配置项**:
```xml
<base>
    <item tag="sim_tz" value="0/1"/>
</base>
```

**证据**: `services/native/src/thermal_srv_config_parser.cpp:159-176`

### 5. Fan Fault Detection（风扇故障检测）

检测风扇故障并采取保护措施。

**证据**: `services/native/include/thermal_policy/fan_fault_detect.h`

---

## 约束与限制

### 输入限制

| 参数 | 限制 |
|---|---|
| 温度传感器列表 | 无明确上限 |
| 回调数量 | 每个类型可多个订阅者 |
| 配置文件大小 | 受 libxml2 解析限制 |

### 输出限制

| 输出 | 限制 |
|---|---|
| 热级别 | 0-7 |
| 温度值 | int32_t，单位: 毫摄氏度 |
| 动作执行频率 | 受策略配置控制 |

### 性能约束

- 回调通知：异步执行，不阻塞主线程
- 策略执行：主线程同步执行
- 文件操作：使用安全函数（snprintf_s）

---

## 总结

Thermal Manager 是一个边界清晰的系统服务，通过标准化的 N-API 提供有限的温度查询和回调接口，核心热管理逻辑封装在 Thermal Service 中，通过 IPC 与上层交互，通过 HDI 与底层驱动通信。权限模型严格，仅系统应用可调用敏感接口。

**核心边界**:
- ✅ 职责清晰：分层明确，无交叉职责
- ✅ 依赖单向：上层依赖下层，无环依赖
- ✅ 接口稳定：内部接口保持兼容性
- ✅ 权限严格：敏感操作需系统权限
