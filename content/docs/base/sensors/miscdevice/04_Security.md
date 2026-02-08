# 安全风险评审

## 概述

本文档对 miscdevice 子系统进行安全风险评审，分析攻击面、信任边界和潜在安全风险。

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界示意                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     IPC/Binder     ┌─────────────────────┐      │
│  │   应用层    │ ◄────────────────► │  MiscDevice SA     │      │
│  │  (可信)     │     边界           │  (可信域)           │      │
│  └──────┬──────┘                    └──────────┬────────┘      │
│         │                                      │                │
│         │ Permission Check                      │ HDI Call       │
│         ▼                                      ▼                │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   驱动层 (HAL)                          │     │
│  │           Vibrator Driver | Light Driver               │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 信任级别

| 区域 | 信任级别 | 说明 |
|------|----------|------|
| System Ability (SA) | **高** | 系统级服务，运行在 sensors 进程 |
| Native Client | **高** | 同进程，共享 SA 信任域 |
| JS N-API | **中** | 需经过权限校验 |
| 应用层 | **低** | 不可信输入来源 |

## 攻击面分析

### 1. N-API 接口 (JS/ArkTS)

**入口**: `frameworks/js/napi/vibrator/src/vibrator_js.cpp`

| API | 输入 | 潜在风险 |
|-----|------|----------|
| `startVibration` | VibrateEffect.duration | 整数溢出、资源耗尽 |
| `stopVibration` | VibratorStopMode | 枚举值校验 |
| `isSupportEffect` | effectId | 字符串注入、路径遍历 |
| `startVibration` | VibrateEffect.effectId | effectId 注入 |

**证据**: `vibrator_js.cpp:522-540` - `napi_get_cb_info` 参数解析

```cpp
static napi_value StartVibrate(napi_env env, napi_callback_info info) {
    size_t argc = 3;
    napi_value args[3] = {};
    napi_value thisArg = nullptr;
    // 参数校验不足: effectId 未验证长度和格式
    napi_status status = napi_get_cb_info(env, info, &argc, args, &thisArg, nullptr);
}
```

### 2. C API

**入口**: `frameworks/capi/vibrator.cpp`

| API | 输入 | 潜在风险 |
|-----|------|----------|
| `OH_Vibrator_StartVibration` | Vibrator_Parameter | 结构体字段校验缺失 |
| `OH_Vibrator_IsSupportEffect` | effectId | effectId 长度无限制 |

**证据**: `vibrator.cpp` - C API 实现中缺少边界检查

### 3. Binder IPC

**入口**: `services/miscdevice_service/src/miscdevice_service.cpp`

| 接口 | 潜在风险 |
|------|----------|
| `StartVibrator()` | 反序列化数据校验 |
| `StopVibrator()` | 模式枚举校验 |
| `TransferClientRemoteObject()` | 远程对象传递安全 |

**证据**: `miscdevice_service.cpp:OnRemoteRequest()` - 消息处理

```cpp
// 远程对象直接传递，缺少验证
int32_t TransferClientRemoteObject(const sptr<IRemoteObject> &vibratorServiceClient) {
    // vibratorServiceClient 未验证有效性
}
```

### 4. HDI 调用

**入口**: `services/miscdevice_service/hdi_connection/adapter/src/hdi_connection.cpp`

| 调用 | 潜在风险 |
|------|----------|
| `StartVibration()` | 驱动参数未校验 |
| `StopVibration()` | 停止信号完整性 |

### 5. 权限系统

**入口**: `utils/common/src/permission_util.cpp`

| 检查点 | 潜在风险 |
|--------|----------|
| `CheckVibratePermission()` | 权限缓存误用 |
| `VerifyAccessToken()` | Token 伪造 |

## 已识别风险

### 🔴 高风险

#### 1. effectId 参数注入

**风险等级**: 🔴 高
**CWE**: CWE-78 (OS Command Injection) / CWE-79 (XSS)

**证据**:
- 文件: `frameworks/js/napi/vibrator/src/vibrator_napi_utils.cpp`
- 函数: `ParseString()` - 未校验 effectId 长度和字符集

**触发条件**:
```
1. 攻击者构造超长 effectId (>256 字符)
2. 调用 startVibration() 或 isSupportEffect()
3. 可能导致缓冲区溢出或注入攻击
```

**影响**:
- 内存损坏 (CWE-120)
- 拒绝服务
- 潜在代码执行

**修复建议**:
```cpp
// 添加 effectId 长度校验
constexpr size_t MAX_EFFECT_ID_LEN = 256;
if (effectId.length() > MAX_EFFECT_ID_LEN) {
    return napi_throw_error(env, "401", "Parameter validation failed: effectId too long");
}

// 添加字符白名单校验
static const std::regex EFFECT_ID_PATTERN("^[a-z0-9_.]+$");
if (!std::regex_match(effectId, EFFECT_ID_PATTERN)) {
    return napi_throw_error(env, "401", "Parameter validation failed: invalid effectId");
}
```

#### 2. duration 整数溢出

**风险等级**: 🔴 高
**CWE**: CWE-190 (Integer Overflow)

**证据**:
- 文件: `frameworks/js/napi/vibrator/src/vibrator_napi_utils.cpp`
- 函数: `ParseInt32()` - 未检查负数或超大值

**触发条件**:
```
1. 攻击者传入负数 duration (-1)
2. 传入超大 duration (>INT_MAX)
3. 导致未定义行为或资源耗尽
```

**影响**:
- 振动器失控
- 拒绝服务
- 电池耗尽

**修复建议**:
```cpp
// 添加范围校验
constexpr int32_t MIN_DURATION = 0;
constexpr int32_t MAX_DURATION = 3600000; // 1小时上限

if (duration < MIN_DURATION || duration > MAX_DURATION) {
    return napi_throw_error(env, "401", "Parameter validation failed: invalid duration");
}
```

### 🟡 中风险

#### 3. 客户端远程对象未验证

**风险等级**: 🟡 中
**CWE**: CWE-346 (Origin Validation Error)

**证据**:
- 文件: `services/miscdevice_service/src/miscdevice_service.cpp`
- 函数: `TransferClientRemoteObject()`

**触发条件**:
```
1. 恶意应用传递伪造的 IRemoteObject
2. SA 接受并缓存该对象
3. 可能导致回调劫持
```

**影响**:
- 回调劫持
- 权限提升 (如果伪造系统服务)

**修复建议**:
```cpp
int32_t TransferClientRemoteObject(const sptr<IRemoteObject> &vibratorServiceClient) {
    // 验证远程对象类型
    auto descriptor = vibratorServiceClient->GetObjectDescriptor();
    if (descriptor != "IVibratorClient") {
        return PERMISSION_DENIED;
    }
    
    // 验证调用权限
    // ...
}
```

#### 4. 振动持续时间无上限

**风险等级**: 🟡 中
**CWE**: CWE-400 (Resource Exhaustion)

**证据**:
- 文件: `services/miscdevice_service/src/miscdevice_service.cpp`
- 函数: `StartVibrator()` - 未限制最大振动时长

**触发条件**:
```
1. 应用请求持续振动 (duration = 无限)
2. 用户无法取消
3. 电池快速耗尽、设备发热
```

**影响**:
- 拒绝服务 (电池/设备)
- 用户体验损害

**修复建议**:
```cpp
// 在 SA 层添加全局时长限制
constexpr int32_t MAX_CONTINUOUS_DURATION_MS = 300000; // 5分钟

if (duration > 0 && duration != VIBRATOR_DURATION_INFINITE) {
    if (duration > MAX_CONTINUOUS_DURATION_MS) {
        HiSysEventWrite(..., "Duration exceeded limit");
        return PARAMETER_ERROR;
    }
}
```

#### 5. 缺少调用审计日志

**风险等级**: 🟡 中
**CWE**: 缺失安全审计

**证据**:
- 文件: `services/miscdevice_service/src/miscdevice_service.cpp`
- 函数: 多个 API 缺少完整审计日志

**触发条件**:
```
1. 安全事件发生后无法追溯
2. 难以定位恶意调用来源
```

**影响**:
- 安全事件无法溯源
- 合规性问题

**修复建议**:
```cpp
// 添加 HiSysEvent 审计
HiSysEventWrite(
    HiSysEvent::Domain::MISCDEVICE,
    "VIBRATOR_API_CALL",
    HiSysEvent::EventType::BEHAVIOR,
    "CALLER_PKG", packageName,
    "API", "StartVibrator",
    "PARAMS", effect.ToString()
);
```

### 🟢 低风险

#### 6. 错误信息泄露

**风险等级**: 🟢 低
**CWE**: CWE-209 (Information Exposure Through Error Message)

**证据**:
- 文件: `frameworks/js/napi/vibrator/src/vibrator_napi_error.cpp`
- 函数: `GetErrorMessage()` - 可能返回内部路径信息

#### 7. 竞态条件 (DoS)

**风险等级**: 🟢 低
**CWE**: CWE-367 (Time-of-check Time-of-use)

**证据**:
- 文件: `services/miscdevice_service/src/vibration_priority_manager.cpp`
- 振动优先级管理可能存在竞态

#### 8. 资源泄漏 (Fuzz 测试)

**风险等级**: 🟢 低
**CWE**: CWE-775 (Missing Release of Resource)

**证据**: 存在 fuzz 测试 (`test/fuzztest/service/`)，说明已关注 IPC 边界安全

## 安全控制措施

### 已实现控制

| 控制 | 实现位置 | 效果 |
|------|----------|------|
| 权限检查 | `permission_util.cpp` | 防止未授权访问 |
| Token 验证 | `AccessTokenKit` | 身份认证 |
| HiSysEvent 事件 | `hisysevent.yaml` | 安全事件上报 |
| CFI 防护 | `BUILD.gn` | 控制流保护 |
| PAC/BTI | `BUILD.gn` | 返回地址保护 |

### 建议补充控制

| 控制 | 建议实现位置 | 优先级 |
|------|-------------|--------|
| 参数长度校验 | `vibrator_napi_utils.cpp` | 🔴 高 |
| 范围校验 | `miscdevice_service.cpp` | 🔴 高 |
| 远程对象验证 | `miscdevice_service.cpp` | 🟡 中 |
| API 调用审计 | `miscdevice_service.cpp` | 🟡 中 |
| 时长限制 | `miscdevice_service.cpp` | 🟡 中 |

## 安全测试覆盖

### Fuzz 测试

| 测试文件 | 覆盖接口 | 状态 |
|----------|----------|------|
| `vibratestub_fuzzer.cpp` | Vibrate | ✅ |
| `stopvibratorallstub_fuzzer.cpp` | StopVibrator | ✅ |
| `issupporteffectstub_fuzzer.cpp` | IsSupportEffect | ✅ |
| `getvibratorliststub_fuzzer.cpp` | GetVibratorList | ✅ |
| `geteffectinfostub_fuzzer.cpp` | GetEffectInfo | ✅ |

### 单元测试权限覆盖

| 测试文件 | 权限测试 |
|----------|----------|
| `vibrator_test.cpp` | ✅ PERMISSION_DENIED |
| `light_agent_test.cpp` | ✅ Token 验证 |

## 合规性说明

### OpenHarmony 安全要求

- ✅ 权限模型: 使用 `ohos.permission.VIBRATE` system_grant
- ✅ 审计日志: HiSysEvent 支持
- ✅ 安全编译: CFI, PAC, 符号隐藏
- ⚠️ 参数校验: 部分 API 缺少完整校验 (见风险 #1, #2)

### 个人数据保护

- ✅ 无个人敏感数据收集
- ⚠️ 调用包名记录: 需符合数据最小化原则

## 风险评估总结

| 风险 ID | 风险名称 | 等级 | 状态 |
|---------|----------|------|------|
| SEC-001 | effectId 注入 | 🔴 高 | 需修复 |
| SEC-002 | duration 溢出 | 🔴 高 | 需修复 |
| SEC-003 | 远程对象未验证 | 🟡 中 | 建议修复 |
| SEC-004 | 振动时长无限制 | 🟡 中 | 建议修复 |
| SEC-005 | 缺少 API 审计 | 🟡 中 | 建议修复 |
| SEC-006 | 错误信息泄露 | 🟢 低 | 接受 |
| SEC-007 | 竞态条件 | 🟢 低 | 接受 |
| SEC-008 | 资源泄漏 | 🟢 低 | 接受 |

## 附录

### 参考标准

- CWE v4.14 (Common Weakness Enumeration)
- OWASP Mobile Top 10
- OpenHarmony Security Guidelines

### 相关文件

- `hisysevent.yaml` - 安全事件定义
- `sensors_errors.h` - 错误码定义
- `permission_util.cpp` - 权限检查实现
- `BUILD.gn` - 安全编译配置
