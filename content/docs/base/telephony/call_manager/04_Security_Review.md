# 安全评审

**目的**: 基于代码证据进行安全风险分析

---

## 评审范围

| 范围 | 说明 |
|------|------|
| N-API 入口 | `frameworks/js/napi/src/native_module.cpp` |
| SA 服务 | `services/call_manager_service/` |
| 权限校验 | `telephony_permission.h/cpp` |
| IPC 通信 | IPC Skeleton/Proxy |

---

## 攻击面分析

### 输入点清单

| 输入类型 | 来源 | 处理位置 |
|----------|------|----------|
| 电话号码 | JS API 参数 | `call_manager_service.cpp` |
| callId | JS API 参数 | `CallStatusManager` |
| slotId | JS API 参数 | 各 API 处理函数 |
| 通话状态 | IPC 回调 | `CallStatusListener` |
| 音频设备 | 系统回调 | `AudioDeviceManager` |

### 信任边界

```
┌─────────────────────────────────────────┐
│           不可信区域                      │
│  ┌─────────────────────────────────┐     │
│  │     三方应用进程                │     │
│  │  - JS API 参数                 │     │
│  │  - phoneNumber, callId 等     │     │
│  └─────────────────────────────────┘     │
│                  │                       │
│                  ▼                       │
│  ┌─────────────────────────────────┐     │
│  │     N-API 边界                  │     │
│  │  - 参数序列化                   │     │
│  │  - 类型转换                     │     │
│  └─────────────────────────────────┘     │
│                  │                       │
│                  ▼                       │
│  ┌─────────────────────────────────┐     │
│  │     SA 服务进程                 │     │
│  │  - 权限校验 ✓ 已实现           │     │
│  │  - 参数校验 ✓ 已实现           │     │
│  │  - 核心逻辑处理                 │     │
│  └─────────────────────────────────┘     │
│                  │                       │
│                  ▼                       │
│           可信内部区域                    │
└─────────────────────────────────────────┘
```

---

## 安全控制措施

### 权限校验

**实现位置**: `services/call_manager_service/src/call_manager_service.cpp`

```cpp
// 行 62-68: 权限常量定义
static constexpr const char *OHOS_PERMISSION_PLACE_CALL = "ohos.permission.PLACE_CALL";
static constexpr const char *OHOS_PERMISSION_ANSWER_CALL = "ohos.permission.ANSWER_CALL";
static constexpr const char *OHOS_PERMISSION_GET_TELEPHONY_STATE = "ohos.permission.GET_TELEPHONY_STATE";
static constexpr const char *OHOS_PERMISSION_SET_TELEPHONY_STATE = "ohos.permission.SET_TELEPHONY_STATE";

// 行 321-325: 权限检查
if (!TelephonyPermission::CheckPermission(OHOS_PERMISSION_PLACE_CALL)) {
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

**评估**: ✅ 权限校验已实现

### 参数校验

| 校验项 | 实现状态 | 证据 |
|--------|----------|------|
| phoneNumber 格式 | ✅ 已实现 | `libphonenumber` 库 |
| callId 范围 | ✅ 已实现 | `CallStatusManager` |
| slotId 范围 | ✅ 已实现 | API 参数处理 |
| 空值检查 | ✅ 已实现 | N-API 参数解析 |

### CFI 防护

**配置**: `BUILD.gn:18-23`

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
}
branch_protector_ret = "pac_ret"
```

**评估**: ✅ CFI 防护已启用

### 栈保护

```gn
// BUILD.gn:50
cflags_cc = [
  "-fstack-protector-all",
  "-D_FORTIFY_SOURCE=2",
]
```

**评估**: ✅ 栈保护已启用

---

## 潜在风险

### 风险清单

| 风险 ID | 风险描述 | 风险等级 | 证据 |
|---------|----------|----------|------|
| SEC-001 | 电话号码未完全过滤特殊字符 | 中 | `call_manager_service.cpp` |
| SEC-002 | callId 验证可能存在竞态 | 低 | `CallStatusManager` |
| SEC-003 | 紧急号码绕过权限校验 | 高 | `isEmergencyPhoneNumber` |
| SEC-004 | 分布式通信数据未加密 | 中 | `distributed_call/` |

### SEC-001: 电话号码注入

**描述**: 电话号码参数可能包含特殊字符，导致潜在注入

**触发条件**:
```typescript
call.dialCall("123#456"); // 特殊字符
```

**影响**: 可能的呼叫行为异常

**修复建议**:
```cpp
// 增加电话号码白名单校验
bool ValidatePhoneNumber(const std::string& phoneNumber) {
    // 只允许数字和特定分隔符
    std::regex pattern("^[0-9+\\-*#]+$");
    return std::regex_match(phoneNumber, pattern);
}
```

### SEC-003: 紧急号码权限

**描述**: 紧急号码相关 API 不要求权限，可能被滥用

**证据**: `interfaces/kits/js/@ohos.telephony.call.d.ts:336`

```typescript
// 无权限要求
function isEmergencyPhoneNumber(phoneNumber: string, ...): void;
```

**影响**: 恶意应用可能查询紧急号码列表

**修复建议**: 对紧急号码查询添加 `ohos.permission.READ_CONTACTS` 权限要求

---

## 安全建议

### 短期（高优先级）

| 建议 | 优先级 | 工作量 |
|------|--------|--------|
| 增加电话号码格式严格校验 | 高 | 低 |
| 对紧急号码 API 添加权限 | 高 | 低 |
| 添加通话频率限制 | 中 | 中 |

### 中期（改进）

| 建议 | 优先级 | 工作量 |
|------|--------|--------|
| 分布式通信数据加密 | 中 | 高 |
| 增加调用链追踪 | 低 | 中 |
| 安全日志审计 | 低 | 中 |

### 长期（架构）

| 建议 | 优先级 | 工作量 |
|------|--------|--------|
| 引入安全沙箱隔离 | 低 | 高 |
| 通话操作二次确认 | 低 | 中 |

---

## 局限性说明

### 本次评审未覆盖

| 范围 | 说明 |
|------|------|
| test/ 目录 | 根据规范排除 |
| 第三方库 | libphonenumber, cJSON |
| 运行时漏洞 | 需要动态测试 |
| 侧信道攻击 | 需要专门的安全测试 |

### 工具/技术限制

- 静态代码分析有限
- 未进行渗透测试
- 未验证运行时行为

---

## 相关文档

- [API 参考](02_API_Reference.md)
- [架构说明](01_Architecture.md)
- [构建系统](03_Build_System.md)
