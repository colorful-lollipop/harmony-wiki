# Time Service 项目概览

> 项目定位、边界、核心能力、运行环境

---

## 项目定位

**Time Service** 是 OpenHarmony 标准系统（standard）的时间子系统服务，提供系统级时间管理、时区管理和定时器调度能力。

### 所属子系统

- **子系统**：`time`
- **部件名**：`time_service`
- **系统能力**：`SystemCapability.MiscServices.Time`
- **源码路径**：`/base/time/time_service`

### 与 Android 对比

| 功能 | Android | OpenHarmony |
|------|---------|-------------|
| 时间设置 | `AlarmManager.setTime()` | `@ohos.systemTime.setTime()` |
| 定时器 | `AlarmManager` | `@ohos.systemTimer` |
| 时区 | `AlarmManager.setTimeZone()` | `@ohos.systemTime.setTimezone()` |

---

## 功能边界

### 包含功能

1. **系统时间获取/设置**
   - Wall Time（UTC 时间）
   - Boot Time（开机至今，含休眠）
   - Monotonic Time（开机至今，不含休眠）
   - Thread Time（线程 CPU 时间）

2. **时区管理**
   - 获取当前时区
   - 设置系统时区

3. **定时器调度**
   - 创建/启动/停止/销毁定时器
   - 支持重复/单次定时器
   - 支持唤醒/非唤醒模式
   - 支持精确/非精确触发
   - 支持 WantAgent 触发通知

4. **NTP 时间同步**
   - 自动 NTP 时间校准
   - 可信时间获取

5. **定时器代理**
   - 应用后台冻结时延迟定时器
   - 调整定时器触发策略

### 不包含功能

- 闹钟应用（Alarm Clock）- 由上层应用实现
- 日历事件提醒 - 由日历应用实现
- 倒计时 UI - 由应用层实现

---

## 核心能力

### API 矩阵

| 命名空间 | 功能类别 | 主要接口 |
|----------|----------|----------|
| `@ohos.systemTime` | 系统时间 | `setTime`, `getCurrentTime`, `getTimezone` |
| `@ohos.systemTimer` | 定时器 | `createTimer`, `startTimer`, `stopTimer`, `destroyTimer` |
| `@ohos.systemDateTime` | 日期时间 | `setDate`, `getDate` |

### 架构特性

| 特性 | 说明 |
|------|------|
| **SystemAbility** | SA_ID: 3702，系统常驻服务 |
| **IPC 通信** | Binder-based，IDL 接口定义 |
| **多语言支持** | JS/TS (N-API)、C/C++ (NDK)、Cangjie (FFI) |
| **持久化** | 支持定时器重启恢复（RDB/cJSON） |
| **后台管理** | 支持应用冻结时代理定时器 |

---

## 运行环境

### 目标系统

- **适配类型**：standard（标准系统）
- **最小 ROM**：400KB
- **最小 RAM**：2845KB

### 权限要求

| 功能 | 权限名 | 权限级别 |
|------|--------|----------|
| 设置时间 | `ohos.permission.SET_TIME` | system_basic |
| 设置时区 | `ohos.permission.SET_TIME_ZONE` | system_basic |
| 获取时间 | 无 | normal |
| 定时器管理 | 系统应用校验 | system |

### 依赖组件

```
napi, samgr, common_event_service, cJSON
os_account, ipc, ability_base, ability_runtime
relational_store, hilog, hicollie, safwk
c_utils, access_token, hisysevent
device_standby, init, power_manager, runtime_core
```

---

## 关键概念

### 时间类型

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| **Wall Time** | UTC 时间（1970 起） | 显示给用户的时间 |
| **Boot Time** | 开机至今（含休眠） | 计算运行时长 |
| **Monotonic Time** | 开机至今（不含休眠） | 性能计时 |
| **Thread Time** | 线程 CPU 时间 | 性能分析 |

### 定时器类型

| 类型标志 | 值 | 说明 |
|----------|-----|------|
| `TIMER_TYPE_REALTIME` | 1 << 0 | 基于 Wall Time |
| `TIMER_TYPE_WAKEUP` | 1 << 1 | 休眠时唤醒设备 |
| `TIMER_TYPE_EXACT` | 1 << 2 | 精确触发（不批处理） |
| `TIMER_TYPE_IDLE` | 1 << 3 | 低功耗模式触发 |
| `TIMER_TYPE_INEXACT_REMINDER` | 1 << 4 | 非精确提醒 |

### 定时器状态

```
CREATE → START → [TRIGGER] → STOP → DESTROY
   ↓                    ↓
   +--------------------+
        (repeat=true)
```

---

## 相关链接

- [目录结构](./01_Directory_Structure.md) - 源码组织
- [架构说明](./02_Architecture.md) - 组件关系
- [N-API 参考](./03_NAPI_Reference.md) - 接口详情
