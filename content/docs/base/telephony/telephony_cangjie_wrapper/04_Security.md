# 安全风险评审

## 评审范围

| 范围 | 包含 | 不包含 |
|------|------|--------|
| 源码 | telephony_cangjie_wrapper (本仓库) | call_manager 原生实现 |
| 文件 | .cj 源码, BUILD.gn | 测试文件, mock 存根 |
| 能力 | N/A (仅调用) | 权限声明 |

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────┐
│                   不可信区域                             │
│  应用层: 仓颉应用提供的 phoneNumber 等输入参数            │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼ [FFI 边界]
┌─────────────────────────────────────────────────────────┐
│                   信任边界                               │
│  telephony_cangjie_wrapper 封装层                       │
│  - 输入校验                                              │
│  - 错误码映射                                            │
│  - 异常封装                                             │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼ [外部依赖]
┌─────────────────────────────────────────────────────────┐
│                   可信区域                               │
│  call_manager 原生库                                     │
│  - 权限校验 (SystemCapability)                          │
│  - 业务逻辑                                              │
└─────────────────────────────────────────────────────────┘
```

### 数据流

| 数据 | 来源 | 流向 | 敏感度 |
|------|------|------|--------|
| phoneNumber | 应用输入 | FFI → call_manager | 中 (电话号码) |
| slotId | 应用输入 | FFI → call_manager | 低 |
| countryCode | 应用输入 | FFI → call_manager | 低 |
| CallState | call_manager | 返回应用 | 低 |
| Bool 结果 | call_manager | 返回应用 | 低 |

---

## 风险点分析

### ✅ 已校验/低风险项

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 输入校验 | ✅ 已覆盖 | call_manager 负责 |
| 空指针处理 | ✅ Safe | Cangjie 无空指针 |
| 缓冲区溢出 | ✅ Safe | Cangjie FFI 类型安全 |
| 权限声明 | ✅ 已验证 | syscap 在注解中声明 |

---

### ⚠️ 潜在风险点

#### 1. 电话号码未本地校验（依赖 call_manager）

**证据**：`call.cj:57` - `makeCall()` 直接调用 FFI

```cj
public static func makeCall(phoneNumber: String): Unit {
    unsafe {
        try (cNumber = LibC.mallocCString(phoneNumber).asResource()) {
            let errCode = FfiOHOSTelephonyCallMakeCall(cNumber.value)
```

**触发路径**：
1. 应用调用 `Call.makeCall("恶意构造号码")`
2. phoneNumber 未在封装层校验
3. 直接传递给 FFI 函数

**潜在影响**：
- 无效号码 → call_manager 返回错误
- 恶意构造号码 → 依赖 call_manager 防护

**当前缓解**：
- call_manager 负责输入校验
- BusinessException 捕获异常分支

**修复建议**：
```
1. 在封装层增加号码格式校验 (正则: ^\+?[0-9]*$)
2. 限制号码长度 (建议 ≤ 20 位)
3. 过滤特殊字符
```

---

#### 2. 紧急号码判断可被滥用

**证据**：`call.cj:162` - `isEmergencyPhoneNumber()` 返回 Bool

```cj
public static func isEmergencyPhoneNumber(phoneNumber: String, ...): Bool {
    unsafe {
        var result = true
        try (...) {
            result = FfiOHOSTelephonyCallIsEmergencyPhoneNumber(...)
        }
        return result
    }
}
```

**触发路径**：
1. 应用循环调用 `isEmergencyPhoneNumber()`
2. 触发 call_manager 服务调用

**潜在影响**：
- DoS 攻击：频繁调用耗尽资源
- 隐私泄露：可探测紧急号码列表

**当前缓解**：
- 部分 API 在 workerthread 执行

**修复建议**：
```
1. 实施调用频率限制
2. 考虑本地缓存紧急号码列表
3. 添加结果缓存机制
```

---

#### 3. 错误信息泄露系统细节

**证据**：`number_format_options.cj:57` - `getErrorMsg()`

```cj
protected func getErrorMsg(code: Int32): String {
    let errCode = getErrorCode(code)
    if (let Some(v) <- getUniversalErrorMsg(errCode)) {
        return v
    } else if (ERROR_CODE_MAP.contains(errCode)) {
        return ERROR_CODE_MAP[errCode]
    } else {
        return "Unknown error code: ${errCode}"  // 可能泄露内部错误码
    }
}
```

**触发路径**：
1. call_manager 返回内部错误码
2. 封装层直接透传未知错误码

**潜在影响**：
- 信息泄露：暴露 call_manager 内部状态
- 帮助攻击者理解系统结构

**当前缓解**：
- 大部分错误码已标准化 (8300xxx)

**修复建议**：
```
1. 过滤所有内部错误码，只返回标准化错误
2. 未知错误返回通用错误消息
3. 考虑日志记录而非返回给应用
```

---

#### 4. SIM 卡槽索引未校验范围

**证据**：`number_format_options.cj:96` - `EmergencyNumberOptions(slotId!)`

```cj
public init(slotId!: Int32 = 0) {
    this.slotId = slotId  // 无范围校验
}
```

**触发路径**：
1. 应用构造 `EmergencyNumberOptions(slotId: -1)`
2. 传递给 FFI

**潜在影响**：
- 负数 slotId → 依赖 call_manager 处理
- 超大数值 → 未定义行为

**当前缓解**：
- call_manager 应该处理边界情况

**修复建议**：
```cj
public init(slotId!: Int32 = 0) {
    if (slotId < 0) {
        throw BusinessException(8300001, "slotId must be non-negative")
    }
    this.slotId = slotId
}
```

---

#### 5. 国家代码未校验格式

**证据**：`number_format_options.cj:199` - `NumberFormatOptions(countryCode!)`

```cj
public init(countryCode!: String = "CN") {
    this.countryCode = countryCode  // 无格式校验
}
```

**触发路径**：
1. 应用传入无效国家代码
2. 传递给 FFI → call_manager

**潜在影响**：
- 无效国家代码 → call_manager 返回错误
- 可能触发异常路径

**修复建议**：
```cj
public init(countryCode!: String = "CN") {
    // ISO 3166-1 格式: 2字母大写
    if (countryCode.length != 2 || !countryCode.match("^[A-Z]{2}$")) {
        throw BusinessException(8300001, "Invalid country code format")
    }
    this.countryCode = countryCode
}
```

---

## 安全改进建议汇总

| 优先级 | 风险点 | 建议 |
|--------|--------|------|
| P1 | 紧急号码滥用 | 添加调用频率限制 |
| P2 | 国家代码格式 | 增加 ISO 3166-1 校验 |
| P2 | slotId 范围 | 增加非负校验 |
| P3 | 错误码泄露 | 过滤内部错误码 |
| P3 | 号码格式 | 增加本地校验 |

---

## 检查局限性说明

1. **未检查 call_manager 原生实现** - 安全策略由 call_manager 维护
2. **未检查权限声明** - SystemCapability 声明由系统管理
3. **未检查运行时行为** - DoS、隐私等运行时问题需安全测试验证
4. **mock 实现未纳入评审** - Windows/Mac 存根不用于生产环境
