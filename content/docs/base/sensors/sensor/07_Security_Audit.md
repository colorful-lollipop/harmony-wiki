# 安全风险评审

> **目的**: 分析 Sensor 子系统的攻击面、信任边界、可被利用点和修复建议
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统通过权限检查、SA 通信和输入验证来保障安全，但存在一些潜在的可利用点需要注意
> **相关跳转**: [架构说明](02_Architecture.md) | [对外 N-API 参考](03_NAPI_Reference.md)

---

## 攻击面清单

### 1. N-API 接口攻击面

**攻击路径**: 应用 → N-API → Native → IPC → Service

| 攻击面 | 说明 | 检查范围 |
|---------|------|---------|
| 参数注入 | 恶意应用传入恶意参数 | 全部 API |
| 权限绕过 | 试图绕过权限检查 | Subscribe/Enable 接口 |
| 回调劫持 | 劫持或替换回调函数 | Subscribe/once/off 接口 |
| 拒绝服务 | 通过恶意订阅消耗资源 | Subscribe/once/off 接口 |
| 信息泄露 | 获取设备状态、传感器信息 | GetSensorList 等 API |

### 2. IPC 通信攻击面

**攻击路径**: Client ↔ IPC ↔ Service

| 攻击面 | 说明 | 检查范围 |
|---------|------|---------|
| IPC 欺骗 | 伪造客户端身份 | ISensorService 接口 |
| Binder 数据篡改 | 修改传输中的传感器数据 | IPC 数据传输 |
| 死亡通知伪造 | 模拟进程死亡 | DeathRecipient |
| 序列化漏洞 | 恶意构造 Parcel 数据 | IPC 序列化/反序列化 |

### 3. 系统能力攻击面

**攻击路径**: Service → SA Framework → 其他 SA

| 攻击面 | 说明 | 检查范围 |
|---------|------|---------|
| SA 假冒 | 伪造 Sensor Service 身份 | SystemAbility 注册 |
| 权限提升 | 利用权限检查缺陷 | AccessToken 查询 |
| 跨 SA 调用 | 访问受限的 SA 服务 | IPC 跨服务调用 |

### 4. 文件操作攻击面

**攻击路径**: Service → 文件系统

| 攻击面 | 说明 | 检查范围 |
|---------|------|---------|
| 路径遍历 | 访问受限目录 | TransferDataChannel (已验证安全) |
| 文件描述符泄露 | 泄露敏感文件 FD | StreamServer (Socket通道隔离) |
| 任意文件写入 | 写入任意位置 | 无 (无文件写入操作) |

### 5. 硬件驱动交互攻击面

**攻击路径**: Service → HDF → 硬件

| 攻击面 | 说明 | 检查范围 |
|---------|------|---------|
| 恶意传感器数据 | 发送异常数据到驱动 | HDI 回调 |
| 传感器状态篡改 | 修改传感器配置 | HDI Enable/Disable |
| DoS 攻击 | 频繁操作传感器 | HDI 调用 |

---

## 信任边界

### 边界 1: 应用空间 → 用户空间

**信任度**: **不可信**

**说明**:
- 应用空间是沙箱化的，权限受限
- 应用需要明确的权限声明才能访问传感器
- 用户可能授予 user_grant 权限（如 ACTIVITY_MOTION、READ_HEALTH_DATA）

**防御机制**:
- **权限检查**: `PermissionUtil::CheckSensorPermission()`
- **访问令牌验证**: `AccessTokenKit::VerifyAccessToken()`
- **包名验证**: `SensorManager::GetPackageName()`
- **系统调用检查**: `IsSystemCalling()`, `IsSystemServiceCalling()`

> **证据**: `utils/common/src/permission_util.cpp`, `services/src/sensor_service.cpp:596-622`

### 边界 2: 用户空间 → 内核空间

**信任度**: **部分可信**

**说明**:
- 通过 HDI 接口与硬件驱动通信
- HDI 是 OpenHarmony 标准接口，有基本保护机制

**防御机制**:
- **Binder IPC**: 所有 IPC 通过 Binder 框架
- **HDF 框架**: 硬件驱动框架的隔离

**注意事项**:
- 恶意应用可能在同一设备上，通过 N-API 访问 HDI
- 需要确保 HDI 层的安全性

> **证据**: `services/hdi_connection/`

### 边界 3: System Ability 框架

**信任度**: **部分可信**

**说明**:
- System Ability (SA) 是 OpenHarmony 的服务管理机制
- SA 3601 (Sensor Service) 在受保护的进程空间运行
- 非 system API 的敏感操作需要 native token

**防御机制**:
- **SA 注册限制**: 只有系统服务能注册 SA
- **进程隔离**: SA 在独立进程运行
- **权限验证**: 管理员操作需要 MANAGE_SENSOR 权限

> **证据**: `sa_profile/3601.json`, `services/src/sensor_service.cpp:51`

---

## 可被利用点

### R1: 权限检查绕过风险

**风险等级**: 🟢 低 (已缓解)

**位置**: `services/src/sensor_service.cpp:596-622`

**证据**:
```cpp
// services/src/sensor_service.cpp:596-622
ErrCode SensorService::CheckAuthAndParameter(const SensorDescription &sensorDesc, 
    int64_t samplingPeriodNs, int64_t maxReportDelayNs)
{
    // 1. 系统 API 检查
    if (((g_systemApiSensorCall.find(sensorDesc.sensorType) != g_systemApiSensorCall.end()) ||
        (sensorDesc.sensorType > GL_SENSOR_TYPE_PRIVATE_MIN_VALUE)) && !IsSystemCalling()) {
        SEN_HILOGE("Permission check failed. A non-system application uses the system API");
        return NON_SYSTEM_API;
    }
    
    // 2. 传感器权限检查
    PermissionUtil &permissionUtil = PermissionUtil::GetInstance();
    int32_t ret = permissionUtil.CheckSensorPermission(GetCallingTokenID(), sensorDesc.sensorType);
    if (ret != PERMISSION_GRANTED) {
        // 记录安全审计日志
        HiSysEventWrite(HiSysEvent::Domain::SENSOR, "VERIFY_ACCESS_TOKEN_FAIL", 
            HiSysEvent::EventType::SECURITY, ...);
        return PERMISSION_DENIED;
    }
    
    // 3. 参数验证
    if ((!CheckSensorId(sensorDesc)) || (maxReportDelayNs != 0L && samplingPeriodNs != 0L &&
        ((maxReportDelayNs / samplingPeriodNs) > MAX_EVENT_COUNT))) {
        return ERR_NO_INIT;
    }
    return ERR_OK;
}
```

**触发路径**:
```
JS: sensor.on(ACCELEROMETER, callback)
    ↓
napi: On() [sensor_js.cpp:599]
    ↓
native: SubscribeSensor() [sensor_agent.cpp]
    ↓
ipc: EnableSensor() [ISensorService.idl:22]
    ↓
service: CheckAuthAndParameter() [sensor_service.cpp:596]
    ↓
permission: AccessTokenKit::VerifyAccessToken() [permission_util.cpp:48]
```

**分析**:
- ✅ 每次请求都重新验证 AccessToken，无缓存绕过风险
- ✅ 权限检查失败会记录 HiSysEvent 安全日志
- ✅ 系统 API 有额外的 IsSystemCalling 检查
- ✅ 敏感传感器(计步器、心率)有动态权限监控

**结论**: 权限检查机制完善，未发现绕过风险。

---

### R2: 参数注入风险

**风险等级**: 🟡 中

**位置**: 
- `frameworks/js/napi/src/sensor_js.cpp:569-597` (GetOptionalParameter)
- `frameworks/js/napi/src/sensor_js.cpp:619-621` (sensorType 获取)

**证据**:
```cpp
// frameworks/js/napi/src/sensor_js.cpp:569-597
static bool GetOptionalParameter(napi_env env, napi_value *args, std::string &interval, 
    SensorDescription &sensorDesc)
{
    // 从 JS 对象获取 deviceId, sensorId, location
    napi_value napiDeviceId = GetNamedProperty(env, args[ARG_2], "deviceId");
    GetNativeInt32(env, napiDeviceId, sensorDesc.deviceId);  // 无范围验证!
    
    napi_value napiSensorId = GetNamedProperty(env, args[ARG_2], "sensorId");
    GetNativeInt32(env, napiSensorId, sensorDesc.sensorId);  // 无范围验证!
}

// sensor_js.cpp:619-621
SensorDescription sensorDesc;
sensorDesc.sensorType = INVALID_SENSOR_TYPE;
if (!GetNativeInt32(env, args[0], sensorDesc.sensorType)) {
    // 仅检查类型，未验证范围
}
```

**触发路径**:
```
JS: sensor.on(999999, callback, {deviceId: -1, sensorId: 0x7FFFFFFF})
    ↓
napi: On() → GetOptionalParameter()
    ↓
无范围验证，直接传递到 IPC
    ↓
service: CheckSensorId() 在服务端验证
    ↓
返回 ERR_NO_INIT (安全)
```

**分析**:
- ⚠️ N-API 层对 deviceId/sensorId 缺乏范围验证
- ✅ 但服务端 `CheckSensorId()` 会验证传感器有效性
- ⚠️ 潜在风险：无效参数可能导致 IPC 资源浪费

**建议**:
1. 在 N-API 层添加 deviceId/sensorId 范围检查
2. 验证 sensorType 是否在有效枚举范围内
3. 限制 options 对象只能包含白名单字段

---

### R3: 回调劫持风险

**风险等级**: 🟡 中

**位置**: 
- `frameworks/js/napi/src/sensor_js.cpp:68-69` (全局回调映射)
- `frameworks/js/napi/src/sensor_js.cpp:599-635` (On函数)

**证据**:
```cpp
// frameworks/js/napi/src/sensor_js.cpp:68-69
static std::unordered_map<SensorDescription, std::unordered_map<napi_ref, AsyncCallbackInfo *>> 
    g_onCallbackInfos;
static std::unordered_map<SensorDescription, std::unordered_map<napi_ref, AsyncCallbackInfo *>> 
    g_onceCallbackInfos;

// On() 函数中的回调覆盖逻辑 (行 ~630)
if (g_onCallbackInfos.find(sensorDesc) != g_onCallbackInfos.end()) {
    // 存在已有回调，可能被覆盖
    auto callbackInfo = g_onCallbackInfos[sensorDesc];
    // ...
}
```

**触发路径**:
```
应用A: sensor.on(ACCELEROMETER, callbackA)  // 注册回调
    ↓
g_onCallbackInfos[ACCELEROMETER] = {callbackARef -> asyncInfoA}
    ↓
应用A再次调用: sensor.on(ACCELEROMETER, callbackB)  // 同一应用覆盖
    ↓
g_onCallbackInfos[ACCELEROMETER] = {callbackBRef -> asyncInfoB}  // callbackA被覆盖
```

**分析**:
- ⚠️ 同一应用内，后注册的回调会覆盖先注册的
- ✅ 不同应用间隔离（通过进程隔离）
- ✅ 覆盖前会清理旧回调资源
- ⚠️ 可能导致数据接收方不确定

**建议**:
1. 在文档中明确说明多次订阅的行为
2. 提供警告日志当覆盖发生时
3. 建议应用使用唯一的回调函数

---

### R4: 拒绝服务攻击风险 (DoS)

**风险等级**: 🟢 低 (已缓解)

**位置**: `services/src/client_info.cpp:282-304`

**证据**:
```cpp
// services/src/client_info.cpp:282-304
bool ClientInfo::UpdateSensorChannel(int32_t pid, sptr<SensorBasicDataChannel> &channel)
{
    // ...
    if (channelMap_.size() >= MAX_SUPPORT_CHANNEL) {  // 最大200个通道
        SEN_HILOGE("The maximum number of channels is reached, channelMap_ size is %{public}u",
            static_cast<uint32_t>(channelMap_.size()));
        return false;
    }
    // ...
}
```

**触发路径**:
```
恶意应用 → 创建大量进程 → 每个进程订阅传感器
    ↓
ClientInfo::UpdateSensorChannel() 检查 channelMap_.size()
    ↓
达到 MAX_SUPPORT_CHANNEL (200) 后拒绝新订阅
    ↓
返回 false，不影响已有服务
```

**分析**:
- ✅ 有通道数量限制 (MAX_SUPPORT_CHANNEL = 200)
- ✅ 超出限制时返回错误，不崩溃
- ✅ 资源分配失败有日志记录
- 🟡 如果200个通道都被恶意占用，正常应用无法订阅

**建议**:
1. 按应用设置通道配额（如每个应用最多5个）
2. 添加速率限制（每分钟最多创建多少次）
3. 实现通道租约机制（超时自动回收）

---

### R5: 信息泄露风险

**风险等级**: 🟢 低 (设计如此)

**位置**: `services/src/sensor_service.cpp:803-815`

**证据**:
```cpp
// services/src/sensor_service.cpp:803-815
ErrCode SensorService::GetSensorList(std::vector<Sensor> &sensorList)
{
    std::vector<Sensor> sensors = GetSensorList();
    int32_t sensorCount = static_cast<int32_t>(sensors.size());
    if (sensorCount > MAX_SENSOR_COUNT) {  // 限制最多200个
        SEN_HILOGD("SensorCount:%{public}u", sensorCount);
        sensorCount = MAX_SENSOR_COUNT;
    }
    for (int32_t i = 0; i < sensorCount; ++i) {
        sensorList.push_back(sensors[i]);  // 返回所有传感器信息
    }
    return ERR_OK;
}
```

**返回信息** (来自 `utils/common/include/sensor.h`):
```cpp
class Sensor {
    int32_t sensorTypeId_;      // 传感器类型
    int32_t sensorId_;          // 传感器ID
    int32_t deviceId_;          // 设备ID
    std::string name_;          // 传感器名称
    std::string vendor_;        // 厂商
    float resolution_;          // 分辨率
    float minRange_;            // 最小范围
    float maxRange_;            // 最大范围
    int32_t flags_;             // 标志位
    int32_t minDelay_;          // 最小延迟
    int32_t maxDelay_;          // 最大延迟
    int32_t fifoMaxEventCount_; // FIFO最大事件数
    int32_t location_;          // 位置
};
```

**分析**:
- 🟡 返回了传感器硬件配置信息
- 🟡 可能用于设备指纹识别
- ✅ 但这是传感器框架的设计功能
- ✅ 敏感传感器仍需权限才能订阅

**建议**:
1. 这是一个设计权衡，不是安全漏洞
2. 如需增强隐私，可考虑添加 SENSOR_INFO 权限
3. 对于高隐私场景，限制传感器列表访问

---

## 输入校验检查

### 已实现的校验

| 校验项 | 实现位置 | 状态 |
|---------|----------|------|
| 传感器类型 | `CreateEnumSensorType()` | ✅ 已实现 |
| 参数类型 | `On()` 函数中 | ✅ 已实现 |
| 采样间隔 | `GetOptionalParameter()` | ✅ 已实现 |
| 权限检查 | `CheckSensorPermission()` | ✅ 已实现 |

### 缺失的校验

| 校验项 | 建议 | 优先级 |
|---------|------|--------|
| deviceId 范围验证 | 确保只能访问本地设备 | 高 |
| sensorId 有效性验证 | 防止越界访问 | 高 |
| callback 来源验证 | 防止回调劫持 | 中 |
| 采样频率上限 | 防止 DoS | 中 |
| 订阅数量限制 | 防止资源耗尽 | 中 |

---

## 权限安全机制

### Access Token 验证

**实现**: `AccessTokenKit::VerifyAccessToken()`

**保护**:
- 验证调用者是否拥有所需权限
- 支持 system_grant 和 user_grant 权限
- 支持权限撤销通知

> **证据**: `utils/common/src/permission_util.cpp:41-54`

### 系统调用检查

**实现**:
- `IsSystemCalling()` - 检查是否为系统调用
- `IsSystemServiceCalling()` - 检查是否为系统应用
- `IsNativeToken()` - 检查是否为 native token

**保护**:
- 保护管理 API (SuspendSensors, ResetSensors)
- 防止普通应用调用敏感操作

> **证据**: `services/src/sensor_service.cpp:572-594`

---

## 数据安全

### 数据传输保护

**IPC 数据传输**:
- 使用 Binder IPC（有权限控制）
- FileDescriptor 通道（高性能数据流）

**数据隔离**:
- 每个客户端有独立的数据通道
- FIFO 缓存隔离数据

> **证据**: `services/include/fifo_cache_data.h`

### 敏感数据保护

| 数据类型 | 保护措施 | 证据 |
|---------|---------|------|
| 传感器数据 | 权限检查、沙箱隔离 | `permission_util.cpp` |
| 设备信息 | SA 通信保护 | `sa_profile/3601.json` |
| 权限状态 | 动态权限监控 | `services/src/sensor_service.cpp:1197-1239` |

---

## 检查范围

### 已检查范围

| 范围 | 覆盖度 |
|------|--------|
| N-API 接口权限检查 | ✅ 完整 |
| 管理员操作权限检查 | ✅ 完整 |
| 动态权限撤销处理 | ✅ 完整 |
| 输入参数校验 | ✅ 基本覆盖 |
| IPC 通信安全 | ✅ 通过 Binder 框架 |

### 未检查范围

| 范围 | 说明 | 建议 |
|------|------|--------|
|deviceId 范围验证 | 需要确保只能访问本地设备 | 高优先级 |
| Callback 来源验证 | 防止劫持 | 中优先级 |
| HDI 层安全性 | 需要确认 HDF 保护机制 | 高优先级 |
| 内存安全 | 检查潜在的内存安全问题 | 中优先级 |

---

## 修复优先级

### 高优先级修复

1. **权限绕过防护**
   - 加强每次权限检查
   - 添加日志审计

2. **回调劫持防护**
   - 验证回调来源
   - 防止覆盖

3. **参数注入防护**
   - 严格验证 deviceId 和 sensorId
   - 限制敏感参数

### 中优先级修复

1. **资源耗尽防护**
   - 实施订阅数量限制
   - 添加速率控制

2. **信息泄露防护**
   - 限制返回的设备信息
   - 添加调用频率限制

### 低优先级增强

1. **日志和监控**
   - 添加安全审计日志
   - 监控异常行为

2. **文档和测试**
   - 编写安全使用指南
   - 进行安全测试

---

## 相关跳转

- [对外 N-API 参考](03_NAPI_Reference.md) - 查看 API 安全要求
- [内部 API](04_Internal_API.md) - 查看内部接口安全
- [架构说明](02_Architecture.md) - 理解信任边界和数据流
