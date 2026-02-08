# 安全风险评审

> 基于代码证据的电池统计模块安全风险分析与修复建议。

## 目录

- [威胁模型概述](#威胁模型概述)
- [信任边界](#信任边界)
- [攻击面分析](#攻击面分析)
- [安全控制措施](#安全控制措施)
- [可利用风险点](#可利用风险点)
- [安全建议](#安全建议)

## 威胁模型概述

### 数据流图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   JS/ArkTS  │────▶│  N-API Layer│────▶│ IPC Client  │────▶│ IPC Binder  │
│   (App)     │     │  (Native)   │     │  (Proxy)    │     │  (Kernel)   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Dump CLI   │◀────│   Shell     │◀────│   Service   │◀────│  IPC Stub   │
│  (Debug)    │     │   Dump      │     │   (SA 3304) │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### 外部输入

| 输入源 | 格式 | 用途 |
|--------|------|------|
| JS 参数 | number | uid, type 等 |
| IPC 参数 | Parcel | uid, type, args 等 |
| Shell Dump | string[] | 调试命令 |
| HiSysEvent | JSON | 事件监听 |
| 配置文件 | JSON | 功耗参数 |

**证据**: `services/native/include/battery_stats_parser.h` 和 `services/profile/power_average.json`

## 信任边界

### 边界 1: JS 层 → N-API 层

- **信任度**: 高
- **控制**: 类型检查、参数数量限制
- **风险**: 恶意 JS 代码传入异常参数

### 边界 2: N-API 层 → IPC 层

- **信任度**: 中
- **控制**: IPC 代理层
- **风险**: 服务端权限检查依赖

### 边界 3: IPC 客户端 → IPC 服务端

- **信任度**: 高
- **控制**: Binder IPC + Parcel 序列化验证
- **风险**: 进程间数据篡改 (理论上由系统保证)

### 边界 4: 服务端内部

- **信任度**: 高
- **控制**: Permission::IsSystem() 检查
- **风险**: 内部逻辑错误

## 攻击面分析

### N-API 接口

| API | 参数 | 潜在风险 |
|-----|------|---------|
| `getBatteryStats` | callback (可选) | 回调中资源泄露 |
| `getAppPowerValue` | uid (number) | UID 范围未校验 |
| `getAppPowerPercent` | uid (number) | UID 范围未校验 |
| `getHardwareUnitPowerValue` | type (number) | type 范围未校验 |
| `getHardwareUnitPowerPercent` | type (number) | type 范围未校验 |

### IPC 接口

| 方法 | 输入 | 潜在风险 |
|------|------|---------|
| `GetBatteryStatsIpc` | 无 | 无 |
| `GetAppStatsMahIpc` | uid | 负数 UID |
| `GetPartStatsMahIpc` | type | 越界 type |
| `ShellDumpIpc` | args[] | 参数注入 |
| `SetOnBatteryIpc` | isOnBattery | 无 |

### Dump 接口

| 参数 | 限制 | 风险 |
|------|------|------|
| argc | MAX 10 | 拒绝服务 |
| args | string[] | 命令注入 |

### 文件 I/O

| 文件 | 路径 | 风险 |
|------|------|------|
| 电池统计数据 | `/data/service/el0/stats/battery_stats.json` | 路径固定，但需文件权限 |
| 功耗配置 | `services/profile/power_average.json` | 启动时加载 |

## 安全控制措施

### 权限检查

**证据**: `services/native/src/battery_stats_service.cpp:195-197`

```cpp
if (!Permission::IsSystem()) {
    lastError_ = static_cast<int32_t>(StatsError::ERR_SYSTEM_API_DENIED);
    return StatsUtils::DEFAULT_VALUE;
}
```

| API | 权限检查位置 | 检查方式 |
|-----|-------------|---------|
| `GetBatteryStats()` | Line 195 | Permission::IsSystem() |
| `Dump()` | Line 209 | Permission::IsSystem() |
| `GetAppStatsMah()` | Line 232 | Permission::IsSystem() |
| `GetPartStatsMah()` | Line 254 | Permission::IsSystem() |
| `GetTotalTimeSecond()` | Line 275 | Permission::IsSystem() |
| `SetOnBattery()` | Line 341 | Permission::IsSystem() |

### 参数校验

**证据**: `frameworks/napi/src/napi_utils.cpp:48-57`

```cpp
bool NapiUtils::CheckValueType(napi_env& env, napi_value& value, napi_valuetype checkType)
{
    napi_valuetype valueType = napi_undefined;
    napi_typeof(env, value, &valueType);
    if (valueType != checkType) {
        STATS_HILOGW(COMP_FWK, "Parameter type error");
        return false;
    }
    return true;
}
```

### Parcel 验证

**证据**: `frameworks/native/src/battery_stats_info.cpp:227-236`

```cpp
ParcelableBatteryStatsList* ParcelableBatteryStatsList::Unmarshalling(Parcel& parcel) {
    int32_t size = parcel.ReadInt32();
    if (size < PARAM_ZERO || size > PARAM_MAX_NUM) {  // PARAM_MAX_NUM = 2000
        STATS_HILOGE(COMP_FWK, "size is invalid, size=%{public}d", size);
        return nullptr;
    }
    // ...
}
```

### IPC 看门狗

**证据**: `utils/native/src/stats_xcollie.cpp`

```cpp
StatsXCollie::StatsXCollie(const std::string &logTag, bool isRecovery) {
    const int DFX_DELAY_S = 60;  // 60 second timeout
    unsigned int flag = HiviewDFX::XCOLLIE_FLAG_LOG;
    if (isRecovery) {
        flag = HiviewDFX::XCOLLIE_FLAG_LOG | HiviewDFX::XCOLLIE_FLAG_RECOVERY;
    }
    id_ = HiviewDFX::XCollie::GetInstance().SetTimer(logTag_, DFX_DELAY_S, nullptr, nullptr, flag);
}
```

所有 IPC 方法使用 60 秒超时看门狗。

## 可利用风险点

### 风险 1: UID 参数未校验范围 (中等风险)

**证据**: `frameworks/napi/src/battery_stats.cpp:148-149`

```cpp
int32_t jsValue;
napi_get_value_int32(env_, argv[index], &jsValue);
```

| 属性 | 值 |
|------|-----|
| **代码位置** | `frameworks/napi/src/battery_stats.cpp:148-149` |
| **触发方式** | 传入负数或超大 UID 值 |
| **影响** | 可能导致数组越界访问或错误的统计结果 |
| **修复建议** | 添加 UID 范围校验 (`uid > 0 && uid < MAX_UID`) |

### 风险 2: ConsumptionType 参数未校验范围 (中等风险)

**证据**: `frameworks/napi/src/battery_stats.cpp:117-118, 128-129`

```cpp
BatteryStatsInfo::ConsumptionType naviveType = BatteryStatsInfo::ConsumptionType(type);
double partStatsMah = BatteryStatsClient::GetInstance().GetPartStatsMah(naviveType);
```

| 属性 | 值 |
|------|-----|
| **代码位置** | `frameworks/napi/src/battery_stats.cpp:117-118, 128-129` |
| **触发方式** | 传入非法的 ConsumptionType 整数值 |
| **影响** | 枚举值越界，可能导致崩溃或错误统计 |
| **修复建议** | 添加 type 范围校验 (`type >= CONSUMPTION_TYPE_INVALID && type <= CONSUMPTION_TYPE_ALARM`) |

### 风险 3: Shell Dump 参数数量限制绕过 (低风险)

**证据**: `frameworks/native/src/battery_stats_client.cpp:188-191`

```cpp
uint32_t argc = args.size();
if (argc >= PARAM_MAX_NUM) {  // PARAM_MAX_NUM = 10
    STATS_HILOGE(COMP_FWK, "params exceed limit, argc=%{public}u", argc);
    return dumpshell;
}
```

| 属性 | 值 |
|------|-----|
| **代码位置** | `frameworks/native/src/battery_stats_client.cpp:188-191` |
| **触发方式** | 传入超过 10 个参数 |
| **影响** | 返回错误信息，但不影响系统稳定性 |
| **当前控制** | 已有限制，行为正确 |
| **建议** | 此风险已缓解，无需额外修复 |

### 风险 4: 服务死亡后客户端状态不一致 (低风险)

**证据**: `frameworks/native/src/battery_stats_client.cpp:56-75`

```cpp
void BatteryStatsClient::ResetProxy(const wptr<IRemoteObject>& remote)
{
    std::lock_guard<std::mutex> lock(mutex_);
    STATS_RETURN_IF(proxy_ == nullptr);
    auto serviceRemote = proxy_->AsObject();
    if ((serviceRemote != nullptr) && (serviceRemote == remote.promote())) {
        serviceRemote->RemoveDeathRecipient(deathRecipient_);
        proxy_ = nullptr;
    }
}
```

| 属性 | 值 |
|------|-----|
| **代码位置** | `frameworks/native/src/battery_stats_client.cpp:56-75` |
| **触发方式** | SA 崩溃后再次调用 API |
| **影响** | 可能触发重新连接，但存在竞态窗口 |
| **当前控制** | DeathRecipient 机制 |
| **建议** | 后续版本可考虑增加重连重试机制 |

### 风险 5: 缺少输入消毒 (低风险)

**证据**: `services/native/src/battery_stats_dumper.cpp` (未发现消毒逻辑)

| 属性 | 值 |
|------|-----|
| **代码位置** | `services/native/src/battery_stats_dumper.cpp` |
| **触发方式** | Shell dump 传入特殊字符 |
| **影响** | 可能导致日志输出异常 |
| **当前控制** | 参数数量限制 |
| **建议** | 添加参数白名单校验 |

## 安全建议

### 高优先级

1. **添加 UID 范围校验**
   - 位置: `battery_stats.cpp` 的 `GetAppOrPartStats()` 函数
   - 建议: `if (uid <= 0 || uid > MAX_UID) return ERR_PARAM_INVALID`

2. **添加 ConsumptionType 范围校验**
   - 位置: `battery_stats.cpp` 的 `GetPartStatsMah()` 和 `GetPartStatsPercent()`
   - 建议: `if (type < CONSUMPTION_TYPE_INVALID || type > CONSUMPTION_TYPE_ALARM) return ERR_PARAM_INVALID`

### 中优先级

3. **Shell Dump 参数白名单**
   - 位置: `battery_stats_dumper.cpp`
   - 建议: 定义允许的 dump 命令列表，拒绝不在列表中的命令

4. **增加 IPC 调用日志**
   - 位置: `battery_stats_service.cpp`
   - 建议: 记录所有 IPC 调用的 UID、参数、结果，便于审计

### 低优先级

5. **增加重连重试机制**
   - 位置: `battery_stats_client.cpp`
   - 建议: SA 死亡后首次调用自动重试

6. **输入长度限制**
   - 位置: N-API 层
   - 建议: 对字符串类输入添加长度限制

## 相关文档

- [概览](./00_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 参考](./03_NAPI.md)
- [故障排查](./08_Troubleshooting.md)
- [SUMMARY](./SUMMARY.md)
