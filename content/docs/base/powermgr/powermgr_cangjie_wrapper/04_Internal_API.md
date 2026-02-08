# 内部 API

> **目的**: 了解 powermgr_cangjie_wrapper 的内部模块接口和依赖关系
> **适用范围**: 模块重构、依赖分析、稳定性评估
> **最后更新**: 2025-02-06

---

## 核心模块

### 1. BatteryInfo 类 (ohos/battery_info/battery_info.cj)

**职责**: 提供电池信息查询的公共 API

**接口稳定性**: **稳定** (公共 API)

**证据**: `ohos/battery_info/battery_info.cj:34-170`

#### 公共接口 (public)

| 接口类型 | 名称 | 稳定性 | 说明 |
|---------|------|-------|------|
| 静态属性 | `batterySoc: Int32` | 稳定 | 电池电量百分比 |
| 静态属性 | `chargingStatus: BatteryChargeState` | 稳定 | 充电状态 |
| 静态属性 | `healthStatus: BatteryHealthState` | 稳定 | 健康状态 |
| 静态属性 | `pluggedType: BatteryPluggedType` | 稳定 | 充电器类型 |
| 静态属性 | `voltage: Int32` | 稳定 | 电池电压 |
| 静态属性 | `technology: String` | 稳定 | 电池技术类型 |
| 静态属性 | `batteryTemperature: Int32` | 稳定 | 电池温度 |
| 静态属性 | `isBatteryPresent: Bool` | 稳定 | 电池是否存在 |
| 静态属性 | `batteryCapacityLevel: BatteryCapacityLevel` | 稳定 | 容量等级 |
| 静态属性 | `nowCurrent: Int32` | 稳定 | 当前电流 |

#### 枚举类型 (public)

| 枚举名称 | 静态方法 | 稳定性 |
|---------|---------|-------|
| `BatteryPluggedType` | `parse(value: Int32)` | 稳定 |
| `BatteryChargeState` | `parse(value: Int32)` | 稳定 |
| `BatteryHealthState` | `parse(value: Int32)` | 稳定 |
| `BatteryCapacityLevel` | `parse(value: Int32)` | 稳定 |

**证据**: `ohos/battery_info/battery_info.cj:181-449`

---

### 2. native.cj - FFI 声明 (ohos/battery_info/native.cj)

**职责**: 声明 FFI 外部函数，绑定 C 层接口

**接口稳定性**: **内部接口** (不应直接使用)

**证据**: `ohos/battery_info/native.cj:20-40`

#### FFI 函数声明 (foreign)

| 函数名 | 返回值 | 稳定性 | 说明 |
|-------|-------|-------|------|
| `FfiBatteryInfoBatterySOC()` | `Int32` | 内部 | 获取电池电量 |
| `FfiBatteryInfoGetChargingState()` | `Int32` | 内部 | 获取充电状态 |
| `FfiBatteryInfoGetHealthState()` | `Int32` | 内部 | 获取健康状态 |
| `FfiBatteryInfoGetPluggedType()` | `Int32` | 内部 | 获取充电器类型 |
| `FfiBatteryInfoGetVoltage()` | `Int32` | 内部 | 获取电压 |
| `FfiBatteryInfoGetBatteryNowCurrent()` | `Int32` | 内部 | 获取电流 |
| `FfiBatteryInfoGetTechnology()` | `CString` | 内部 | 获取电池技术 |
| `FfiBatteryInfoGetBatteryTemperature()` | `Int32` | 内部 | 获取电池温度 |
| `FfiBatteryInfoGetBatteryPresent()` | `Bool` | 内部 | 检查电池存在 |
| `FfiBatteryInfoGetCapacityLevel()` | `Int32` | 内部 | 获取容量等级 |

**依赖外部组件**: `battery_manager:cj_battery_info_ffi`

---

## 模块依赖方向

```
┌─────────────────────────────────────────────────────────┐
│                    Cangjie 应用                        │
│                  (依赖方)                              │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ 依赖 ohos.battery_info 包
                     │
┌────────────────────▼────────────────────────────────────┐
│         powermgr_cangjie_wrapper                       │
│                                                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │  battery_info.cj                             │   │
│  │  - import ohos.business_exception.BusinessException  │
│  │  - import ohos.labels.APILevel              │   │
│  │  - import std.deriving.Derive                │   │
│  └──────────────────┬────────────────────────────┘   │
│                     │                                  │
│  ┌──────────────────▼────────────────────────────┐   │
│  │  native.cj (FFI 声明)                          │   │
│  │  - foreign func FfiBatteryInfoBatterySOC()   │   │
│  │  - ... (9个更多FFI函数)                       │   │
│  └──────────────────┬────────────────────────────┘   │
│                     │                                  │
└─────────────────────┼──────────────────────────────────┘
                      │
                      │ FFI 调用 (依赖)
                      │
┌─────────────────────▼──────────────────────────────────┐
│          battery_manager:cj_battery_info_ffi            │
│                  (外部依赖)                             │
└──────────────────────────────────────────────────────────┘
```

### 依赖分析

#### 依赖的外部组件

1. **cangjie_ark_interop:ohos.business_exception**
   - **用途**: 提供业务异常类
   - **依赖类型**: Cangjie 库
   - **接口**: `BusinessException(code: Int32, message: String)`
   - **证据**: `ohos/battery_info/BUILD.gn:31`, `ohos/battery_info/battery_info.cj:21`

2. **cangjie_ark_interop:ohos.labels**
   - **用途**: 提供 API Level 标注
   - **依赖类型**: Cangjie 库
   - **接口**: `@!APILevel[since: "22", syscap: "..."]`
   - **证据**: `ohos/battery_info/BUILD.gn:32`, `ohos/battery_info/battery_info.cj:20`

3. **battery_manager:cj_battery_info_ffi**
   - **用途**: 提供电池管理 FFI C 接口
   - **依赖类型**: C 库
   - **接口**: 10 个 FFI 函数
   - **证据**: `ohos/battery_info/BUILD.gn:35`, `bundle.json:25`

#### 被依赖方

- Cangjie 应用开发者
  - 通过 `import ohos.battery_info` 导入模块
  - 使用 `BatteryInfo.<property>` 查询电池信息

---

## 接口稳定性

### 稳定接口 (Stable)

以下接口属于公共 API，承诺向后兼容：

- ✅ `BatteryInfo` 类的所有静态属性
- ✅ 所有枚举类型及其值
- ✅ 枚举的 `parse()` 静态方法

**判断依据**:
- 使用 `public` 修饰符
- 有 `@!APILevel` 标注
- 文档化（有注释）

**证据**: `ohos/battery_info/battery_info.cj:34-449`

### 内部接口 (Internal)

以下接口不应被外部直接使用：

- ⚠️ `native.cj` 中的所有 FFI 函数声明
- ⚠️ 枚举的内部实现细节（如 `match` 表达式）

**判断依据**:
- 使用 `foreign` 声明
- 没有被 `public` 导出
- 实现细节，非契约

**证据**: `ohos/battery_info/native.cj:20-40`

### 可替换点

1. **Mock 实现**
   - **位置**: `mock/ohos.battery_info.cj`
   - **替换方式**: 通过条件编译 `is_mingw || is_mac`
   - **用途**: Windows/macOS 开发环境
   - **证据**: `ohos/battery_info/BUILD.gn:21-28`

2. **FFI 实现**
   - **位置**: `battery_manager:cj_battery_info_ffi` (外部组件)
   - **替换方式**: 修改 `external_deps`
   - **用途**: 替换底层数据源
   - **证据**: `ohos/battery_info/BUILD.gn:35`

---

## 线程模型与并发

### 线程安全

**Cangjie 层**:
- ✅ 无共享状态
- ✅ 每次调用独立
- ✅ 无竞态条件

**FFI 层**:
- ✅ 由 C 层 `battery_manager` 保证线程安全
- ⚠️ TODO(需确认): C 层的具体线程安全实现

### 并发策略

**同步阻塞**: 所有 API 调用都是同步的，阻塞当前线程

**无异步支持**: 目前不支持异步/Promise 机制

**建议**: 高频查询应考虑使用定时器而非轮询

---

## 资源生命周期

### 内存管理

**CString 管理**:
- `FfiBatteryInfoGetTechnology()` 返回 `CString`
- 在 Cangjie 层转换为 `String`
- 显式调用 `LibC.free(cStr)` 释放 C 内存

**证据**: `ohos/battery_info/battery_info.cj:110-116`

**其他类型**:
- `Int32`, `Bool`: 值类型，无内存管理
- 枚举: 值类型，无内存管理

### 资源泄漏风险

- ⚠️ **已缓解**: `FfiBatteryInfoGetTechnology()` 的 CString 被正确释放
- ⚠️ **TODO(需确认)**: 其他 FFI 函数是否涉及资源分配

---

## 错误传播机制

### 错误类型

**BusinessException**:
- **来源**: `ohos.business_exception.BusinessException`
- **错误码**: 401 (参数错误)
- **错误消息**: "Parameter error."

**证据**: `ohos/battery_info/battery_info.cj:225, 282, 359, 446`

### 错误传播路径

```
C 层返回无效值
  ↓
枚举 parse() 接收
  ↓
match 表达式匹配失败
  ↓
case _ 分支执行
  ↓
throw BusinessException(401, "Parameter error.")
  ↓
异常向上传播到调用方
```

### 错误处理建议

**应用层**:
```cangjie
try {
    let status = BatteryInfo.chargingStatus
} catch (e: BusinessException) {
    // 处理参数错误
    println("错误: ${e.code} - ${e.message}")
}
```

**证据**: 枚举的 `parse()` 方法会抛出异常

---

## 接口契约

### FFI 契约

**C 层承诺**:
- 返回 Int32 值在枚举定义范围内
- 返回的 CString 必须由调用方释放
- `FfiBatteryInfoGetTechnology()` 返回有效的 C 字符串

**Cangjie 层承诺**:
- 调用 FFI 前不修改状态
- 及时释放 C 层分配的内存
- 将无效值转换为异常

**证据**: `ohos/battery_info/native.cj:20-40`, `ohos/battery_info/battery_info.cj:110-116`

### 枚举值契约

**C 层承诺**:
- `BatteryPluggedType`: 0-3
- `BatteryChargeState`: 0-3
- `BatteryHealthState`: 0-5
- `BatteryCapacityLevel`: 1-7

**超出范围的后果**: 抛出 `BusinessException(401, "Parameter error.")`

**证据**: `ohos/battery_info/battery_info.cj:219-227, 276-284, 351-361, 437-447`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [模块职责](01_Module_Responsibilities.md) - 目录结构和模块划分
- [架构说明](02_Architecture.md) - 查看组件图和数据流
- [对外 API](03_Public_API.md) - 公共接口详细说明
- [GN Targets](05_GN_Targets.md) - 构建配置和依赖
