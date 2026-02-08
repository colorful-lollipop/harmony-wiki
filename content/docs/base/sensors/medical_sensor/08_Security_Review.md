# 安全风险评审

## 目的

本文档分析 Medical_Sensor 的安全风险、攻击面和防护机制，并提供修复建议。

---

## 适用范围

本文档适用于需要：
- 理解系统安全机制
- 进行安全审计和风险评估
- 实施安全加固措施

---

## 相关文档

- [攻击面分析](04_AttackSurface.md) - 详细的外部输入清单和攻击路径分析
- [架构说明](02_Architecture.md) - 信任边界和数据流分析
- [N-API 参考](03_N-API_Reference.md) - 参数验证和权限检查点
- [内部 API](05_Internal_API.md) - IPC 接口和安全机制

---

## 威胁模型

```
外部输入（JS API / Native API）
    ↓
参数校验和类型检查
    ↓
权限验证（AccessTokenKit）
    ↓
系统能力层（MedicalSensorService）
    ↓
IPC 通信（Binder）
    ↓
HDI 适配层
    ↓
驱动层（Sensor HDI）
    ↓
硬件传感器
```

---

## 攻击面清单

### 1. N-API 输入攻击面

| 入口点 | 潜在攻击 | 证据 |
|--------|----------|------|
| `on(type, callback, options?)` | 参数注入、类型混淆、回调劫持 | `interfaces/plugin/src/medical_js.cpp:112-157` |
| `off(type, callback?)` | 类型混淆、回调劫持 | `interfaces/plugin/src/medical_js.cpp:159-206` |
| `setOpt(type, option)` | 参数注入 | `interfaces/plugin/src/medical_js.cpp:208-240` |

### 2. IPC 攻击面

| 入口点 | 潜在攻击 | 证据 |
|--------|----------|------|
| `EnableSensor` | 权限绕过、ID 伪造 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:37` |
| `DisableSensor` | 拒绝服务 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:40` |
| `SetOption` | 配置篡改 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:42` |
| `GetSensorState` | 信息泄露 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:44` |
| `TransferDataChannel` | 通道劫持 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:50-51` |
| `DestroySensorChannel` | 资源耗尽 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h:53` |

### 3. HDI 攻击面

| 入口点 | 潜在攻击 | 证据 |
|--------|----------|------|
| `EnableSensor` | 驱动崩溃、数据注入 | `services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h:34` |
| `DisableSensor` | 拒绝服务 | `services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h:36` |
| `SetBatch` | 参数溢出 | `services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h:38` |
| `OnDataEvent` 回调 | 伪造数据、数据篡改 | `services/medical_sensor/hdi_connection/adapter/include/sensor_event_callback.h:30-54` |

### 4. 数据通道攻击面

| 入口点 | 潜在攻击 | 证据 |
|--------|----------|------|
| 共享内存 | 内存泄露、数据篡改 | `frameworks/native/medical_sensor/include/medical_sensor_data_channel.h` |
| Socket | 端口扫描、中间人攻击 | `frameworks/native/medical_sensor/include/medical_sensor_data_channel.h` |

---

## 可被利用点

### 1. 权限检查不完整

**严重性**：高

**问题描述**：当前权限检查仅支持部分传感器类型，未配置权限的传感器默认允许访问。

**证据**：
- 权限映射：`utils/src/permission_util.cpp:35-38`
- 权限检查：`utils/src/permission_util.cpp:40-49`

```cpp
// utils/src/permission_util.cpp:42-44
int32_t PermissionUtil::CheckSensorPermission(AccessTokenID callerToken, int32_t sensorTypeId)
{
    if (sensorPermissions_.find(sensorTypeId) == sensorPermissions_.end()) {
        return PERMISSION_GRANTED;  // ⚠️ 默认允许！
    }
    // ...
}
```

**触发路径**：
1. 攻击者创建恶意应用
2. 调用 `on()` 或 `EnableSensor()` 访问未配置权限的传感器类型
3. 系统返回 `PERMISSION_GRANTED`，成功访问传感器数据

**影响**：
- 未授权访问敏感健康数据（PPG、心率等）
- 隐私泄露风险

**修复建议**：
```cpp
int32_t PermissionUtil::CheckSensorPermission(AccessTokenID callerToken, int32_t sensorTypeId)
{
    // 修复：所有传感器类型都必须有明确权限
    if (sensorPermissions_.find(sensorTypeId) == sensorPermissions_.end()) {
        HiLog::Error(LABEL, "Sensor type %{public}d has no permission configured", sensorTypeId);
        return ERR_PERMISSION_DENIED;  // 拒绝访问
    }
    // ...
}
```

---

### 2. 参数类型校验不充分

**严重性**：中

**问题描述**：JS 层的参数校验仅检查类型，未验证数值范围和边界。

**证据**：
- 类型检查：`interfaces/plugin/src/medical_js.cpp:124, 129, 137`

```cpp
// interfaces/plugin/src/medical_js.cpp:133-140
int64_t interval = 200000000;  // 默认值
if (argc == 3) {
    // ⚠️ 未验证 interval 的最小值和最大值
    napi_value value = NapiGetNamedProperty(args[2], "interval", env);
    interval = GetCppInt64(value, env);
}
```

**触发路径**：
1. 攻击者传入极大或极小的 `interval` 值
2. 系统接受该值并传递到驱动层
3. 驱动层可能出现溢出、除零错误或资源耗尽

**影响**：
- 驱动崩溃
- 系统不稳定
- 拒绝服务

**修复建议**：
```cpp
int64_t interval = 200000000;
if (argc == 3) {
    napi_value value = NapiGetNamedProperty(args[2], "interval", env);
    interval = GetCppInt64(value, env);

    // 添加范围校验
    if (interval < 1000 || interval > 10000000000) {  // 1μs ~ 10s
        HiLog::Error(LABEL, "Invalid interval: %{public}lld", interval);
        napi_throw_error(env, "Interval must be between 1000 and 10000000000 ns");
        return nullptr;
    }
}
```

---

### 3. 数据长度未验证

**严重性**：中

**问题描述**：传感器数据的 `dataLen` 未充分验证，可能导致缓冲区溢出。

**证据**：
- 数据复制：`interfaces/plugin/src/medical_js.cpp:61`
- 数据结构：`interfaces/plugin/include/medical_napi_utils.h:31-39`

```cpp
// interfaces/plugin/src/medical_js.cpp:50-64
static void DataCallbackImpl(SensorEvent *event)
{
    uint32_t maxDataLen = MAX_DATA_LEN * sizeof(uint32_t);  // MAX_DATA_LEN = 512
    uint32_t dataLen = (event->dataLen > maxDataLen) ? maxDataLen : event->dataLen;
    // ⚠️ 截断但未验证 dataLen 的合理性
    if (memcpy_s(onCallbackInfo->sensorData, dataLen, data, dataLen) != EOK) {
        HiLog::Error(LABEL, "%{public}s copy data failed", __func__);
        return;
    }
}
```

**触发路径**：
1. 驱动层返回恶意的 `dataLen`（如 UINT32_MAX）
2. 系统调用 `memcpy_s()` 进行数据复制
3. `memcpy_s()` 的安全检查可能不足以防止所有攻击场景

**影响**：
- 缓冲区溢出（虽然 `memcpy_s()` 有一定保护）
- 内存越界访问
- 潜在代码执行

**修复建议**：
```cpp
static void DataCallbackImpl(SensorEvent *event)
{
    uint32_t maxDataLen = MAX_DATA_LEN * sizeof(uint32_t);

    // 添加合理性校验
    if (event->dataLen == 0 || event->dataLen > maxDataLen) {
        HiLog::Error(LABEL, "Invalid dataLen: %{public}u", event->dataLen);
        return;
    }

    uint32_t dataLen = (event->dataLen > maxDataLen) ? maxDataLen : event->dataLen;
    if (memcpy_s(onCallbackInfo->sensorData, dataLen, data, dataLen) != EOK) {
        HiLog::Error(LABEL, "%{public}s copy data failed", __func__);
        return;
    }
}
```

---

### 4. 全局回调信息管理存在竞态条件

**严重性**：中

**问题描述**：`g_onCallbackInfos` 全局 map 的访问未充分同步，可能存在竞态条件。

**证据**：
- 全局 map：`interfaces/plugin/src/medical_js.cpp:40`
- 多处访问：`interfaces/plugin/src/medical_js.cpp:54-57, 147-152`

```cpp
// interfaces/plugin/src/medical_js.cpp:40
static std::map<int32_t, struct AsyncCallbackInfo*> g_onCallbackInfos;

// ⚠️ 多线程访问无锁保护
static void DataCallbackImpl(SensorEvent *event)
{
    int32_t sensorTypeId = event->sensorTypeId;
    if (g_onCallbackInfos.find(sensorTypeId) == g_onCallbackInfos.end()) {
        return;  // 竞态：可能在检查后被删除
    }
    AsyncCallbackInfo *onCallbackInfo = g_onCallbackInfos[sensorTypeId];
    // ⚠️ 可能已经被删除
    // ...
}

static napi_value On(napi_env env, napi_callback_info info)
{
    // ⚠️ 无锁保护
    g_onCallbackInfos[sensorTypeId] = asyncCallbackInfo;
    // ...
}
```

**触发路径**：
1. 线程 A 调用 `off()` 删除回调（行 197-200）
2. 线程 B 同时在 `DataCallbackImpl()` 中访问回调（行 54-64）
3. 竞态条件导致使用已释放的内存

**影响**：
- Use-After-Free 漏洞
- 潜在任意代码执行
- 系统崩溃

**修复建议**：
```cpp
// 使用互斥锁保护全局 map
static std::mutex g_callbackMutex;
static std::map<int32_t, struct AsyncCallbackInfo*> g_onCallbackInfos;

static void DataCallbackImpl(SensorEvent *event)
{
    std::lock_guard<std::mutex> lock(g_callbackMutex);
    int32_t sensorTypeId = event->sensorTypeId;
    if (g_onCallbackInfos.find(sensorTypeId) == g_onCallbackInfos.end()) {
        return;
    }
    // ...
}

static napi_value On(napi_env env, napi_callback_info info)
{
    std::lock_guard<std::mutex> lock(g_callbackMutex);
    g_onCallbackInfos[sensorTypeId] = asyncCallbackInfo;
    // ...
}
```

---

### 5. Bundle Name 检查为空实现

**严重性**：低

**问题描述**：`GetPackageNameFromUid()` 为空实现，无法进行应用白名单控制。

**证据**：
- 空实现：`services/medical_sensor/src/medical_manager.cpp:191-194`

```cpp
// services/medical_sensor/src/medical_manager.cpp:191-194
void MedicalSensorManager::GetPackageNameFromUid(int32_t uid, std::string &packageName)
{
    HiLog::Debug(LABEL, "%{public}s begin", __func__);
    // ⚠️ 完全空实现！
}
```

**触发路径**：
1. 恶意应用访问传感器
2. 系统尝试获取 Bundle Name 进行白名单检查
3. 空实现导致检查失败，默认允许访问

**影响**：
- 无法实施应用白名单机制
- 无法进行细粒度访问控制
- 增加隐私泄露风险

**修复建议**：

```cpp
void MedicalSensorManager::GetPackageNameFromUid(int32_t uid, std::string &packageName)
{
    HiLog::Debug(LABEL, "%{public}s begin", __func__);

    // 使用 BundleManager 或相关 API 获取 Bundle Name
    auto bundleMgr = BundleMgr::GetInstance();
    if (bundleMgr == nullptr) {
        HiLog::Error(LABEL, "%{public}s bundleMgr is null", __func__);
        return;
    }

    // 根据 UID 查询 Bundle 信息
    BundleInfo bundleInfo;
    if (bundleMgr->GetBundleInfoForUid(uid, bundleInfo) != ERR_OK) {
        HiLog::Error(LABEL, "%{public}s get bundle info failed for uid: %{public}d",
                    __func__, uid);
        return;
    }

    packageName = bundleInfo.name;
}
```

---

## 信任边界

### 1. 应用层 → 系统能力层

**信任边界**：应用（沙盒）→ MedicalSensorService（系统服务）

**安全机制**：
- AccessToken 权限验证
- UID/PID 追踪
- 死亡通知（客户端崩溃检测）

**证据**：
- 权限检查：`utils/src/permission_util.cpp:40-49`
- UID/PID 获取：`services/medical_sensor/src/medical_service.cpp:160, 206`

### 2. 系统能力层 → HDI 适配层

**信任边界**：MedicalSensorService → 传感器驱动

**安全机制**：
- HDI 接口标准化
- 死亡通知和自动重连
- 数据通道隔离

**证据**：
- HDI 连接：`services/medical_sensor/hdi_connection/adapter/src/hdi_connection.cpp:48-70`

### 3. HDI 适配层 → 驱动层

**信任边界**：Medical_Sensor 服务 → 传感器驱动进程

**安全机制**：
- IPC 隔离
- 驱动签名验证（由系统提供）
- 数据校验和格式化

**证据**：
- HDI 接口：`services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h:26-58`

---

## 安全机制

### 1. 权限检查

**机制**：使用 `AccessTokenKit::VerifyAccessToken()` 验证调用者权限。

**检查位置**：
- 所有传感器操作接口（EnableSensor、DisableSensor、SetOption 等）
- 通过 `PermissionUtil::CheckSensorPermission()` 统一检查

**证据**：
- 权限检查：`services/medical_sensor/src/medical_service_stub.cpp:75-144`

**支持的权限**：
- `ohos.permission.READ_HEALTH_DATA`

**证据**：
- 权限定义：`utils/src/permission_util.cpp:32-38`

### 2. 权限使用记录

**机制**：使用 `PrivacyKit::AddPermissionUsedRecord()` 记录权限使用情况（成功/失败次数）。

**证据**：
- 权限记录：`utils/src/permission_util.cpp:51-61`

### 3. 进程隔离

**机制**：
- MedicalSensorService 运行在独立进程（`sensors`）
- 应用通过 IPC（Binder）跨进程通信

**证据**：
- SA 进程：`sa_profile/3605.xml:16`

### 4. 死亡通知

**机制**：
- 客户端死亡通知：服务端检测到客户端崩溃，自动清理资源
- 服务端死亡通知：客户端检测到服务崩溃，自动重连

**证据**：
- 客户端死亡：`services/medical_sensor/src/medical_service.cpp:452-481`
- 服务端死亡：`frameworks/native/medical_sensor/src/medical_service_client.cpp:65-68`

### 5. 数据通道保护

**机制**：使用共享内存（Ashmem）或 Socket 进行数据传输，减少 IPC 开销和拷贝风险。

**证据**：
- 数据通道：`frameworks/native/medical_sensor/include/medical_sensor_data_channel.h`

---

## 检查范围与局限性

### 已检查范围

✅ N-API 输入校验
✅ IPC 接口安全
✅ 权限检查机制
✅ HDI 适配层安全
✅ 数据通道安全
✅ 全局变量线程安全
✅ 参数范围校验

### 未检查范围

⚠️ 驱动层实现（由厂商提供）
⚠️ 硬件安全性（物理攻击防护）
⚠️ 网络传输安全（如适用）
⚠️ 数据加密（传感器数据未加密传输）

---

## 安全加固建议

### 1. 完善权限检查

- 所有传感器类型必须配置明确权限
- 未配置权限的类型应默认拒绝访问

### 2. 增强参数校验

- 添加数值范围检查（interval、option 等）
- 验证数据长度合理性
- 添加特殊字符过滤（防止注入）

### 3. 线程安全加固

- 保护所有全局变量的访问（使用互斥锁）
- 添加原子操作保护关键状态

### 4. 实现 Bundle Name 检查

- 实现 `GetPackageNameFromUid()`
- 支持应用白名单机制

### 5. 增强错误处理

- 所有敏感操作失败应记录详细日志
- 避免在错误路径中泄露敏感信息

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [N-API 参考](03_N-API_Reference.md) - JS API 文档
- [常见问题](08_Troubleshooting.md) - 安全相关故障排查
