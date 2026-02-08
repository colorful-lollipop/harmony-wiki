# 安全风险评审

> **目的**: 基于 code evidence 评估 battery_manager 项目的安全风险，识别可被利用点并提供修复建议

**适用范围**: 输入校验、权限检查、路径遍历、内存安全、竞态条件、信息泄露

---

## 威胁模型

### 外部输入 → 敏感操作

```
JS 应用（用户空间）
    ↓ N-API (batteryInfo, battery, charger)
    ↓ [输入校验薄弱点]
    ↓ 权限检查[部分缺失]
    ↓ IPC 调用 (SA 3302)
    ↓ [服务端权限检查]
    ↓ HDI 驱动访问
    ↓ 硬件层
```

**信任边界**:
- **用户空间**: JS 应用，不可信输入
- **内核空间**: 电池驱动，可信
- **边界**: N-API 层、IPC 通信、服务端业务逻辑

---

## 攻击面分析

### 1. N-API 攻击面

| 入口点 | 风险描述 | 证据 |
|---------|----------|------|
| `setBatteryConfig()` | 配置方法缺少输入验证，可被滥用修改系统配置 | `frameworks/napi/src/battery_info.cpp:185-213` |
| `getBatteryConfig()` | 配置查询方法无权限检查，可泄露敏感配置信息 | `frameworks/napi/src/battery_info.cpp:215-242` |
| `getStatus()` | 异步方法回调未校验回调类型，可被注入恶意回调 | `frameworks/napi/src/system_battery.cpp:245-254` |
| 所有 Getter 方法 | 无权限检查，任何应用可查询电池信息 | `frameworks/napi/src/battery_info.cpp:37-183` |

### 2. IPC 攻击面

| 入口点 | 风险描述 | 证据 |
|---------|----------|------|
| IBatterySrv.idl 接口 | ZIDL 接口部分方法缺乏参数验证 | `services/zidl/IBatterySrv.idl:32-34` |
| Binder IPC | 依赖 Binder IPC，需关注 Binder 安全 | `services/BUILD.gn:58` |
| DeathRecipient | 服务崩溃重启未清理资源 | `frameworks/native/src/battery_srv_client.cpp:106-114` |

### 3. 文件攻击面

| 入口点 | 风险描述 | 证据 |
|---------|----------|------|
| 配置文件读取 | JSON 配置文件解析未做安全检查 | `services/native/src/battery_config.cpp` |
| 资源文件路径 | 多语言资源路径可能被注入 | `services/native/resources/` |

### 4. 权限攻击面

| 入口点 | 风险描述 | 证据 |
|---------|----------|------|
| 配置方法 | SetBatteryConfig 等方法仅检查系统应用身份，无细粒度权限控制 | `services/native/src/battery_service.cpp:678-719` |
| 通知广播 | 电池状态广播仅设置订阅权限，接收者无身份验证 | `services/native/src/battery_notify.cpp:270-271` |

---

## 可被利用点（共 7 条）

### 1. 配置注入攻击

**位置**: `services/native/src/battery_service.cpp:678-681`

**证据**:
```cpp
BatteryError BatteryService::SetBatteryConfigInner(const std::string& sceneName, const std::string& value)
{
    // 仅检查是否为系统应用
    if (!Permission::IsSystem()) {
        BATTERY_HILOGW(FEATURE_BATT_INFO, "SetBatteryConfig failed, System permission intercept");
        return BatteryError::ERR_SYSTEM_API_DENIED;
    }
    // 未验证 sceneName 和 value 的内容
    return BatteryConfig::SetConfigValue(sceneName, value);
}
```

**风险**: 恶意应用可通过系统应用身份，写入任意配置值

**可利用路径**:
```
JS 应用（系统应用）→ setBatteryConfig("critical", "malicious_value")
    → BatteryService::SetBatteryConfigInner()
    → BatteryConfig::SetConfigValue()
    → 写入配置文件/内存
```

**影响**: 可能修改电池配置导致异常行为、安全绕过或拒绝服务

**修复建议**:
1. 实现白名单机制，仅允许预定义的场景名称
2. 验证配置值的格式和范围
3. 添加配置写入审计日志
4. 考虑为不同配置项定义细粒度权限

**优先级**: 高

---

### 2. 配置信息泄露

**位置**: `frameworks/napi/src/battery_info.cpp:215-242`

**证据**:
```cpp
static napi_value GetBatteryConfig(napi_env env, napi_callback_info info)
{
    // 仅参数类型检查，无权限验证
    if (argc != 1 || !NapiUtils::CheckValueType(env, argv[INDEX_0], napi_string)) {
        error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
        return nullptr;
    }

    std::string sceneName = NapiUtils::GetStringFromNapi(env, argv[INDEX_0]);
    BatteryError code = g_battClient.GetBatteryConfig(sceneName, result);

    // 即使服务端返回错误，也返回结果值
    if (code != BatteryError::ERR_OK) {
        error.ThrowError(env, code);
    }

    // 返回配置值给调用者
    return napiValue;
}
```

**风险**: 任何应用可读取电池配置信息，包括敏感参数

**可利用路径**:
```
JS 应用（用户应用）→ getBatteryConfig("charging_limit")
    → BatteryService::GetBatteryConfig()
    → 返回配置值给应用
    → 泄露内部配置参数
```

**影响**: 泄露内部系统配置，可能被恶意利用

**修复建议**:
1. 添加权限检查，仅允许特定配置访问
2. 对敏感配置返回脱敏值
3. 记录配置访问日志
4. 考虑将配置接口移动到系统服务

**优先级**: 中

---

### 3. 异步回调注入

**位置**: `frameworks/napi/src/system_battery.cpp:101-120`

**证据**:
```cpp
bool SystemBattery::CreateCallbackRef(napi_env env, napi_value options)
{
    // 仅检查 options 是否为 object
    RETURN_IF_WITH_RET(!CheckValueType(env, options, napi_object), false);

    // 未验证回调函数是否为有效的 JavaScript 函数
    napi_value succCallBack = GetOptionsFunc(env, options, "success");
    if (succCallBack != nullptr) {
        napi_create_reference(env, succCallBack, 1, &successRef_);
    }
    // fail/complete 回调类似...
}
```

**风险**: 恶意应用可传入恶意回调函数，执行任意 JS 代码

**可利用路径**:
```
JS 应用（恶意）→ getStatus({ success: maliciousFunction })
    → CreateCallbackRef() 创建引用
    → 异步执行时调用恶意函数
    → 在服务进程中执行恶意代码
```

**影响**: 恶意代码在服务进程上下文中执行，可能窃取敏感信息或提升权限

**修复建议**:
1. 验证回调函数来源（必须是本模块定义的函数）
2. 限制回调函数只能访问特定数据
3. 在独立线程池中执行回调，避免在服务线程直接执行
4. 使用 JS 沙箱或严格模式限制回调能力

**优先级**: 高

---

### 4. 通知身份伪造

**位置**: `services/native/src/battery_notify.cpp:270-271`

**证据**:
```cpp
void BatteryNotify::PublishChangedEventInner()
{
    // 仅设置订阅者权限要求，不验证通知创建者身份
    Want want;
    want.SetParam(CommonEventName::COMMON_EVENT_BATTERY_CHANGED_INNER, eventData);
    subscriber_->SetSubscriberPermissions("ohos.permission.POWER_OPTIMIZATION");
    // 发布事件
    CommonEventManager::PublishCommonEvent(want);
}
```

**风险**: 任何应用都可以伪造电池状态事件

**可利用路径**:
```
恶意应用（用户空间）
    → 订阅电池变化事件
    → 发布伪造的电池状态事件（如电量100%）
    → 其他订阅的应用收到虚假事件，误报或触发异常行为
```

**影响**: 误导其他应用，触发异常的系统行为

**修复建议**:
1. 验证通知发布者身份
2. 使用系统提供的应用签名验证
3. 仅允许特权系统应用发布电池事件
4. 添加事件来源标识和验证机制
5. 考虑使用 CommonEvent 的系统级权限控制

**优先级**: 中

---

### 5. 整数溢出风险

**位置**: `frameworks/napi/src/battery_info.cpp:194-211`

**证据**:
```cpp
static napi_value GetRemainingChargeTime(napi_env env, napi_callback_info info)
{
    napi_value napiValue = nullptr;
    int64_t time = g_battClient.GetRemainingChargeTime();
    
    // 直接使用服务返回的 int64_t 值，未检查范围
    NAPI_CALL(env, napi_create_int64(env, time, &napiValue));
    
    return napiValue;
}
```

**风险**: 服务返回异常值可能溢出，导致 JS 层获得错误数据

**可利用路径**:
```
底层驱动故障或攻击
    → 返回异常大/小的剩余时间值
    → GetRemainingChargeTime() 返回溢出值
    → JS 应用收到错误数据，可能造成逻辑错误
```

**影响**: 导致应用逻辑错误，可能被利用触发异常行为

**修复建议**:
1. 在服务端验证返回值的合理范围
2. 在 N-API 层进行边界检查
3. 对关键值使用最大/最小值钳制
4. 添加异常值检测和日志

**优先级**: 中

---

### 6. 信息泄露（Dump 接口）

**位置**: `services/native/include/battery_service.h:71`，`services/native/src/battery_dump.cpp`

**证据**:
```cpp
class BatteryService {
public:
    int32_t Dump(int fd, const std::vector<std::u16string> &args) override;
};

// Dump 实现
int32_t BatteryService::Dump(int fd, const std::vector<std::u16string> &args)
{
    // 部分敏感信息未做权限检查
    if (!Permission::IsSystem()) {
        return ERR_PERMISSION_DENIED;
    }
    
    // 可输出所有电池信息、配置等
    dprintf(fd, "Battery Info:\n");
    dprintf(fd, "  capacity: %d\n", batteryInfo_.GetCapacity());
    dprintf(fd, "  technology: %s\n", batteryInfo_.GetTechnology().c_str());
    // ... 输出其他敏感信息
}
```

**风险**: 未授权应用可获取系统电池信息、配置等敏感数据

**可利用路径**:
```
恶意应用（非系统应用）
    → 通过 Dump 接口请求电池信息
    → 如果通过某种方式获取系统应用身份
    → 绕过权限检查获取敏感信息
```

**影响**: 泄露敏感系统信息，可能被用于其他攻击

**修复建议**:
1. Dump 接口应只对系统应用开放
2. 添加更细粒度的 Dump 权限控制
3. 对输出信息进行脱敏处理
4. 记录 Dump 访问审计日志
5. 考虑使用 Binder 调用者 UID 验证

**优先级**: 高

---

### 7. 竞态条件（TODO）

**位置**: `services/native/src/battery_service.cpp` 多处

**证据**:
```cpp
class BatteryService {
    std::shared_mutex mutex_;  // 保护电池信息更新
    BatteryInfo batteryInfo_;
    BatteryInfo lastBatteryInfo_;
    // 多线程访问 batteryInfo_ 但 lastBatteryInfo_ 未受保护
};

void BatteryService::HandleBatteryInfo() {
    {
        std::lock_guard<std::shared_mutex> lock(mutex_);
        batteryInfo_ = ...
    }  // 锁离开
    // 计算...（无保护）
    HandleCapacity(..., batteryInfo_, lastBatteryInfo_);
}
```

**风险**: `lastBatteryInfo_` 未受 mutex 保护，可能与其他变量产生竞态

**可利用路径**: 未知，需要进一步分析

**影响**: 可能导致数据不一致或逻辑错误

**修复建议**:
1. 将 `lastBatteryInfo_` 也纳入 mutex 保护范围
2. 减少临界区大小
3. 使用原子操作替代互斥锁
4. 进行代码审查，识别所有未受保护的共享状态

**优先级**: 中

---

## 输入校验评估

### N-API 层

| 方法 | 校验类型 | 完整性 | 证据 |
|------|---------|------|------|
| 所有 Getter | 类型检查 | ⚠️ 弱 | `frameworks/napi/src/battery_info.cpp` |
| setBatteryConfig | 数量 + 类型检查 | ✅ 较好 | `frameworks/napi/src/battery_info.cpp:185-213` |
| getBatteryConfig | 数量 + 类型检查 | ⚠️ 无权限 | `frameworks/napi/src/battery_info.cpp:215-242` |
| isBatteryConfigSupported | 数量 + 类型检查 | ⚠️ 无权限 | `frameworks/napi/src/battery_info.cpp:244-272` |
| getStatus | 数量 + 类型检查 | ⚠️ 回调未验证 | `frameworks/napi/src/system_battery.cpp:245-254` |

### IPC 层

| 方法 | 校验类型 | 完整性 | 证据 |
|------|---------|------|------|
| 所有查询接口 | 无输入 | N/A | `services/zidl/IBatterySrv.idl:18-35` |
| SetBatteryConfig | 服务端系统应用检查 | ✅ 较好 | `services/native/src/battery_service.cpp:678-681` |
| GetBatteryConfig | 服务端系统应用检查 | ✅ 较好 | `services/native/src/battery_service.cpp:696-699` |
| IsBatteryConfigSupported | 服务端系统应用检查 | ✅ 较好 | `services/native/src/battery_service.cpp:719-721` |

---

## 权限检查评估

### 权限使用情况

| 接口 | 是否检查权限 | 权限类型 | 证据 |
|------|-----------|----------|------|
| N-API 配置方法 | ⚠️ 部分 | 系统应用身份检查（粗粒度） | `services/native/src/battery_service.cpp:678-719` |
| N-API 查询方法 | ❌ 否 | 无权限检查 | `frameworks/napi/src/battery_info.cpp:37-183` |
| N-API 异步回调 | ❌ 否 | 回调类型未验证 | `frameworks/napi/src/system_battery.cpp:245-254` |
| IPC 配置方法 | ✅ 是 | 系统应用身份检查 | `services/native/src/battery_service.cpp:678-681` |
| IPC 查询方法 | ❌ 否 | 依赖服务端检查 | `services/zidl/IBatterySrv.idl:18-35` |
| Dump 接口 | ✅ 是 | 系统应用身份检查 | `services/native/include/battery_service.h:71` |

---

## 内存安全评估

### 潜在问题

| 问题 | 位置 | 严重性 | 说明 |
|------|------|--------|------|
| 字符串长度未检查 | GetStringFromNapi | 中 | 可能导致缓冲区溢出 |
| 指针未初始化 | 多处 | 低 | 部分路径可能未初始化变量 |
| 引用计数错误 | 多处 | 中 | 回调引用可能泄漏 |
| 配置文件解析 | BatteryConfig | 低 | cJSON 解析未做安全检查 |

---

## 修复优先级建议

### 高优先级

1. **修复配置注入**: 实现白名单机制和输入验证
2. **修复回调注入**: 验证回调函数来源和类型
3. **修复信息泄露**: 添加 Dump 接口权限检查和审计

### 中优先级

1. **改进权限控制**: 为 N-API 查询方法添加细粒度权限
2. **修复通知伪造**: 验证通知发布者身份
3. **修复竞态条件**: 保护所有共享状态变量
4. **改进输入校验**: 添加长度和范围检查

### 低优先级

1. **代码审查**: 全面审查内存安全问题
2. **日志增强**: 添加更多安全相关日志
3. **测试覆盖**: 增加模糊测试和安全测试

---

## 检查范围与局限性

### 已检查范围

- ✅ N-API 层参数校验和错误处理
- ✅ IPC 层接口定义和权限检查
- ✅ 服务端配置管理和权限验证
- ✅ 通知模块权限设置
- ✅ 配置文件解析（BatteryConfig）
- ✅ Dump 接口权限检查
- ✅ 线程安全（部分）

### 未检查范围（需后续分析）

- ⚠️ HDI 层安全（依赖底层驱动）
- ⚠️ 充电模块（charger）的安全实现
- ⚠️ Hook 机制的安全性和使用场景
- ⚠️ 完整的竞态条件分析

---

## 相关跳转

- [目录结构](02_Directory_Structure.md)
- [系统架构](03_Architecture.md)
- [N-API 文档](04_NAPI_API.md)

---

**返回**: [导航](SUMMARY.md)
