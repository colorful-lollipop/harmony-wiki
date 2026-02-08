# 安全评审

> location_cangjie_wrapper 攻击面、信任边界与安全风险分析

## 概述

本文档对 `location_cangjie_wrapper` 进行安全风险评审，识别潜在攻击面、信任边界和可利用点，并提供修复建议。

### 评审范围

| 范围 | 说明 |
|------|------|
| 代码范围 | `kit/`、`ohos/`、`mock/` 目录下的 Cangjie 代码 |
| 构建配置 | `BUILD.gn`、`bundle.json` |
| 外部依赖 | cangjie_ark_interop、hiviewdfx_cangjie_wrapper、location |

### 排除范围

- 测试代码（`test/` 目录）
- 底层 C++ FFI 实现（位于 `base_location` 仓库）
- 硬件驱动层

---

## 信任边界

### 边界模型

```
┌─────────────────────────────────────────────────────────────────┐
│                         不可信区域                               │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              恶意/脆弱应用层（Cangjie App）                 │  │
│  │  - 伪造位置请求参数                                        │  │
│  │  - 绕过权限检查                                            │  │
│  │  - 频繁调用消耗资源                                        │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ▲                                  │
│                              │ 权限边界                         │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │            location_cangjie_wrapper（信任边界内）          │  │
│  │  - 参数校验                                                │  │
│  │  - 权限验证                                                │  │
│  │  - 错误处理                                                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                位置服务（Location Service）                 │  │
│  │  - 系统服务                                                │  │
│  │  - 硬件抽象层                                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              GNSS 硬件（可信执行环境）                      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 边界说明

| 边界 | 说明 | 防护措施 |
|------|------|----------|
| 应用层 → 封装层 | 外部输入进入 | 权限检查、参数校验 |
| 封装层 → 服务层 | 跨进程调用 | 系统能力检查 |
| 服务层 → 硬件 | 内核调用 | 驱动签名验证 |

---

## 攻击面清单

### 1. N-API/Cangjie 接口

| 攻击面 | 输入 | 风险等级 | 说明 |
|--------|------|----------|------|
| `getCurrentLocation()` | 请求参数（可选） | 高 | 返回敏感位置信息 |
| `isLocationEnabled()` | 无 | 低 | 仅返回布尔值 |

**证据**：`geo_location_manager.cj:46-134`

### 2. 参数注入

| 参数类型 | 字段 | 风险 |
|----------|------|------|
| `CurrentLocationRequest.priority` | 枚举值 | 低（枚举约束） |
| `CurrentLocationRequest.scenario` | 枚举值 | 低（枚举约束） |
| `CurrentLocationRequest.maxAccuracy` | Float32 | 中（范围未校验） |
| `CurrentLocationRequest.timeoutMs` | Int32 | 中（范围未校验） |
| `SingleLocationRequest.locatingTimeoutMs` | Int32 | 中（范围未校验） |

**证据**：`geo_location_manager_common.cj:408-462`、`geo_location_manager_common.cj:507-541`

### 3. 内存操作

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| FFI 数据转换 | 高 | C/C++ 与 Cangjie 数据结构转换 |
| 字符串处理 | 中 | CString 与 String 转换 |
| 指针操作 | 高 | CPointer 内存访问 |

**证据**：`geo_location_manager_ffi.cj:59-105`

### 4. 日志泄露

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| Hilog 日志 | 中 | 位置数据可能写入系统日志 |
| 错误消息 | 低 | BusinessException 消息 |

**证据**：`geo_location_manager_common.cj:28`

---

## 可被利用点分析

### 风险 1：敏感信息泄露（高风险）

**证据**：`geo_location_manager.cj:53-64`

```cj
public static func getCurrentLocation(): Location {
    var errCode: Int32 = 0
    let cLocation = unsafe { FfiOHOSGeoLocationManagerGetCurrentLocation(inout errCode) }
    try {
        if (errCode != SUCCESS_CODE) {
            throw BusinessException(getErrorCode(errCode), getErrorMsg(errCode))
        }
        return cLocation.toLocation()
    } finally {
        cLocation.free()
    }
}
```

**问题**：返回的 `Location` 对象包含精确的经纬度信息，若应用恶意收集可导致用户位置被追踪。

**触发条件**：
1. 用户已授权位置权限
2. 恶意应用调用 API
3. 无节流或频率限制

**影响**：
- 用户隐私泄露
- 位置轨迹追踪
- 定向攻击

**修复建议**：
- 添加位置模糊化选项（返回粗略位置）
- 实现调用频率限制
- 添加使用场景审计日志

---

### 风险 2：参数范围未校验（中风险）

**证据**：`geo_location_manager_common.cj:454-456`

```cj
public init(priority!: LocationRequestPriority = LocationRequestPriority.FirstFix,
    scenario!: LocationRequestScenario = LocationRequestScenario.Unset, maxAccuracy!: Float32 = 0.0,
    timeoutMs!: Int32 = 5000) {
    this.priority = priority
    this.scenario = scenario
    this.maxAccuracy = maxAccuracy
    this.timeoutMs = timeoutMs
}
```

**问题**：`maxAccuracy` 和 `timeoutMs` 参数无范围校验。

**触发条件**：
1. 传入极大值（如 `timeoutMs: 2147483647`）
2. 传入负值（如 `maxAccuracy: -1.0`）

**影响**：
- 资源耗尽（超长超时等待）
- 潜在的整数溢出
- 定位精度误用

**修复建议**：
```cj
// 添加参数校验
if (timeoutMs < 1000 || timeoutMs > 60000) {
    throw BusinessException(400, "timeoutMs must be in [1000, 60000]")
}
if (maxAccuracy < 0.0 || maxAccuracy > 10000.0) {
    throw BusinessException(400, "maxAccuracy must be in [0, 10000]")
}
```

---

### 风险 3：FFI 内存访问越界（中风险）

**证据**：`geo_location_manager_ffi.cj:60-71`

```cj
func toLocation(): Location {
    let cjAdditions = if (additions.isNotNull()) {
        unsafe { Array<String>(additionSize, {i => additions.read(i).toString()}) }
    } else {
        Array<String>()
    }
    // ...
}
```

**问题**：`additions` 和 `additionsMap` 指针未校验是否有效，潜在空指针解引用风险。

**触发条件**：
1. 底层 C++ 返回无效指针
2. 内存已被释放但指针未置空

**影响**：
- 进程崩溃（NullPointerException）
- 潜在的内存读取异常

**修复建议**：
```cj
func toLocation(): Location {
    let cjAdditions: Array<String>
    if (additions.isNotNull() && additionSize > 0) {
        unsafe {
            // 校验内存范围
            if (additionSize > MAX_ADDITIONS_SIZE) {
                throw BusinessException(500, "Invalid additionSize")
            }
            cjAdditions = Array<String>(additionSize, {i => additions.read(i).toString()})
        }
    } else {
        cjAdditions = Array<String>()
    }
    // ...
}
```

---

### 风险 4：日志信息泄露（中风险）

**证据**：`geo_location_manager_common.cj:28`

```cj
let GEO_LOCATION_MANAGER_LOG = HilogChannel(LOG_CORE, LOCATION_LOG_DOMAIN, "CJ-GeoLocationManager")
```

**问题**：若启用详细日志，位置信息可能写入系统日志。

**触发条件**：
1. Hilog 级别设置为 DEBUG 或 INFO
2. 位置数据作为日志内容输出

**影响**：
- 系统日志包含敏感位置信息
- 其他应用可能读取日志

**修复建议**：
- 不在日志中输出完整位置坐标
- 仅输出脱敏位置或操作结果
- 敏感操作使用 DEBUG 级别

---

### 风险 5：BusinessException 信息暴露（低风险）

**证据**：`geo_location_manager_common.cj:568-577`

```cj
func getErrorMsg(code: Int32): String {
    let errorCode = getErrorCode(code)
    if (let Some(v) <- getUniversalErrorMsg(errorCode)) {
        return v
    } else if (ERROR_CODE_MAP.contains(errorCode)) {
        return ERROR_CODE_MAP[errorCode]
    } else {
        return "Unknown error code ${errorCode}"
    }
}
```

**问题**：错误消息可能泄露系统内部状态。

**触发条件**：
1. 返回详细的系统错误码
2. 暴露底层实现细节

**影响**：
- 信息收集（系统指纹）
- 攻击面识别

**修复建议**：
- 使用通用错误消息，避免暴露内部实现
- 错误码映射使用模糊化描述

---

## 权限与安全机制

### 现有安全机制

| 机制 | 实现位置 | 状态 |
|------|----------|------|
| 权限校验 | `geo_location_manager.cj:48` | ✅ 已实现 |
| 能力检查 | `@!APILevel[syscap]` | ✅ 已实现 |
| 错误码映射 | `geo_location_manager_common.cj` | ✅ 已实现 |
| 内存安全释放 | `geo_location_manager_ffi.cj:83-105` | ✅ 已实现 |

### 缺失安全机制

| 机制 | 建议优先级 | 说明 |
|------|------------|------|
| 参数范围校验 | 高 | timeoutMs、maxAccuracy |
| 调用频率限制 | 中 | 防止资源耗尽 |
| 位置模糊化 | 中 | 隐私保护 |
| 安全日志脱敏 | 低 | 日志不输出敏感信息 |

---

## 依赖组件安全评估

### cangjie_ark_interop

| 评估项 | 状态 | 说明 |
|--------|------|------|
| FFI 安全性 | 依赖 | 框架层保障 |
| 异常处理 | 依赖 | BusinessException 机制 |
| 指针安全 | 依赖 | unsafe 标记 |

### hiviewdfx_cangjie_wrapper

| 评估项 | 状态 | 说明 |
|--------|------|------|
| 日志脱敏 | 待评估 | 可能输出敏感信息 |
| 日志级别控制 | 依赖 | 运行时可配置 |

### location（C++ FFI）

| 评估项 | 状态 | 说明 |
|--------|------|------|
| 输入校验 | 依赖 C++ 层 | Cangjie 层已做基本校验 |
| 权限传递 | 依赖系统 | 系统权限机制 |
| 内存安全 | 依赖 C++ 层 | 智能指针管理 |

---

## 安全开发建议

### 1. 应用开发者

```cj
// ✅ 正确示例：先检查位置开关
let isEnabled = GeoLocationManager.isLocationEnabled()
if (!isEnabled) {
    // 引导用户开启位置
    return
}

// ✅ 正确示例：使用超时限制
let request = CurrentLocationRequest(
    timeoutMs: 10000  // 10秒超时
)
let location = GeoLocationManager.getCurrentLocation(request)

// ❌ 错误示例：无超时限制
let location = GeoLocationManager.getCurrentLocation()
```

### 2. 封装层开发者

```cj
// ✅ 正确示例：参数校验
if (timeoutMs < 1000 || timeoutMs > 60000) {
    throw BusinessException(400, "timeoutMs out of range")
}

// ✅ 正确示例：日志不输出敏感信息
GEO_LOCATION_MANAGER_LOG.info("Location request completed")  // 不输出坐标
```

### 3. 系统集成

- 启用位置访问审计日志
- 配置合理的权限授予策略
- 监控异常调用模式

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_Architecture](01_Architecture.md) | 系统架构 |
| [02_API_Reference](02_API_Reference.md) | API 参考 |
| [03_Build](03_Build.md) | 构建配置 |

---

## 安全评审证据

| 风险点 | 证据位置 |
|--------|----------|
| API 权限声明 | `geo_location_manager.cj:48` |
| 参数定义 | `geo_location_manager_common.cj:408-541` |
| FFI 内存操作 | `geo_location_manager_ffi.cj:59-105` |
| 错误码映射 | `geo_location_manager_common.cj:543-577` |
| 日志配置 | `geo_location_manager_common.cj:26-28` |

---

## 评审结论

### 总体评估

| 评估项 | 评级 | 说明 |
|--------|------|------|
| 代码质量 | 良好 | 结构清晰，注释完整 |
| 安全机制 | 基本完善 | 权限、异常机制已实现 |
| 输入校验 | 需加强 | 参数范围未校验 |
| 隐私保护 | 需加强 | 缺少位置模糊化 |

### 待改进项

| 优先级 | 问题 | 建议 |
|--------|------|------|
| 高 | 参数范围未校验 | 添加 timeoutMs、maxAccuracy 范围检查 |
| 中 | 缺少调用频率限制 | 实现 API 调用频率控制 |
| 中 | 位置数据暴露风险 | 添加位置模糊化选项 |
| 低 | 日志可能泄露信息 | 日志输出脱敏处理 |
