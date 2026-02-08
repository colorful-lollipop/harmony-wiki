# 项目概览

> **目的**: 快速了解 powermgr_cangjie_wrapper 项目的定位、能力和边界
> **适用范围**: 新人入门、架构决策参考
> **最后更新**: 2025-02-06

---

## 项目定位

**powermgr_cangjie_wrapper** 是 OpenHarmony 上基于 Cangjie（仓颉）编程语言的电池信息 API 封装层。

**核心价值**: 为 Cangjie 应用开发者提供简洁的电池状态查询接口，隐藏底层 C 实现细节。

**技术定位**:
- **语言**: Cangjie（仓颉）Beta 特性
- **调用方式**: Cangjie FFI → C FFI → 底层驱动
- **API 风格**: 静态属性（无实例化）
- **执行模式**: 同步调用（无异步/Promise）

**证据**: `ohos/battery_info/battery_info.cj:34-170`

---

## 核心能力

### 支持的功能

| 功能类别 | 具体能力 | API |
|---------|---------|-----|
| **电池信息** | 电池电量百分比 | `BatteryInfo.batterySoc` |
| **充电状态** | 充电/未充电/充满 | `BatteryInfo.chargingStatus` |
| **健康状态** | 健康/过热/过压/低温/死亡 | `BatteryInfo.healthStatus` |
| **充电器类型** | AC/USB/无线/未知 | `BatteryInfo.pluggedType` |
| **硬件参数** | 电压 (µV) | `BatteryInfo.voltage` |
| **硬件参数** | 电流 (mA) | `BatteryInfo.nowCurrent` |
| **硬件参数** | 温度 (0.1℃) | `BatteryInfo.batteryTemperature` |
| **电池技术** | 电池技术类型 | `BatteryInfo.technology` |
| **电池存在** | 电池是否存在 | `BatteryInfo.isBatteryPresent` |
| **容量等级** | 满电/高/正常/低/警告/严重/关机 | `BatteryInfo.batteryCapacityLevel` |

**证据**: `ohos/battery_info/battery_info.cj:34-170`

### 不支持的功能

对比 ArkTS 版本的电池 API，当前 Cangjie 版本不支持：

- ❌ 重启服务（系统重启和关机）
- ❌ 系统电源管理服务（系统电源状态管理、休眠锁）
- ❌ 显示相关功耗调整（背光亮度、屏幕开关）
- ❌ 节能模式
- ❌ 电池状态检测和上报
- ❌ 关机充电
- ❌ 温控管理
- ❌ 功耗统计（软件/硬件/应用级）
- ❌ 轻量级设备支持

**证据**: `README.md:47-56`

---

## 运行环境

### 目标系统

- **系统类型**: 标准设备 (standard)
- **API Level**: 22
- **SystemCapability**: `SystemCapability.PowerManager.BatteryManager.Core`

**证据**: `bundle.json:17-19`, `ohos/battery_info/battery_info.cj:30-33`

### 资源占用

- **ROM**: 100KB
- **RAM**: 72KB

**证据**: `bundle.json:20-21`

### 构建环境

- **构建系统**: GN (Generate Ninja)
- **编译器**: cjc (Cangjie Compiler)
- **条件编译**: Windows/macOS 使用 mock 实现

**证据**: `BUILD.gn:14,21-28`

---

## 关键概念

### Cangjie FFI

Cangjie 的外部函数接口（Foreign Function Interface），允许 Cangjie 代码调用 C 语言函数。

**声明方式**:
```cangjie
foreign {
    func FfiBatteryInfoBatterySOC(): Int32
}
```

**证据**: `ohos/battery_info/native.cj:20-40`

### 静态属性 API

所有 API 都通过静态属性暴露，无需实例化 `BatteryInfo` 类。

**调用方式**:
```cangjie
let soc = BatteryInfo.batterySoc
let charging = BatteryInfo.chargingStatus
```

**证据**: `ohos/battery_info/battery_info.cj:34-170`

### 枚举值解析

C 层返回 Int32 值，Cangjie 枚举通过 `parse()` 方法转换为对应的枚举项。

**解析模式**:
```cangjie
static func parse(value: Int32): BatteryChargeState {
    match (value) {
        case 0 => UnknownChargeState
        case 1 => Enabled
        case 2 => Disabled
        case 3 => Full
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**证据**: `ohos/battery_info/battery_info.cj:276-284`

### 业务异常

当参数无效时，抛出 `BusinessException(401, "Parameter error.")`，异常会向上传播到调用方。

**证据**: `ohos/battery_info/battery_info.cj:225,282,359,446`

---

## 项目边界

### 依赖的上游组件

- `battery_manager` - 电池管理服务，提供 C FFI 接口
- `cangjie_ark_interop` - Cangjie 互操作库，提供 `BusinessException` 和 `APILevel`

**证据**: `bundle.json:24-26`, `ohos/battery_info/BUILD.gn:30-35`

### 下游服务对象

- Cangjie 应用开发者
- 通过 `ohos.battery_info` 包访问电池信息

**证据**: `ohos/battery_info/battery_info.cj:18,34`

### 不涉及的领域

- ❌ 驱动层实现（由 `battery_manager` 负责）
- ❌ 系统电源管理策略（由 `power_manager` 负责）
- ❌ 测试用例（在 `test/` 目录）
- ❌ 开发工具和脚本

---

## 相关跳转

- [模块职责](01_Module_Responsibilities.md) - 了解目录结构和模块划分
- [架构说明](02_Architecture.md) - 查看组件图和数据流
- [对外 API](03_Public_API.md) - API 详细清单和使用说明
- [安全风险评审](07_Security_Review.md) - 了解安全风险和修复建议
