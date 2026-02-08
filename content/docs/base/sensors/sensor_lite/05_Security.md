# 安全风险评审

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                         用户进程空间                                      │ │
│  │  ┌──────────┐     ┌──────────────────────────────────────────────┐     │ │
│  │  │ Application│────▶│ sensor_client (libsensor_client.so)        │     │ │
│  │  └──────────┘     └──────────────────────────────────────────────┘     │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                    │                                          │
│                                    ▼ IPC                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    sensor_service (独立进程)                             │ │
│  │  - SAMGRlite 服务                                                       │ │
│  │  - 参数校验                                                              │ │
│  │  - HDI 调用                                                             │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                    │                                          │
│                                    ▼ HDI                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    HDF Sensor Driver                                     │ │
│  │                    (硬件访问)                                            │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 外部输入

| 输入类型 | 来源 | 处理方式 |
|---------|------|---------|
| sensorTypeId | 用户进程 (API 参数) | 范围校验 (`>= SENSOR_TYPE_ID_MAX`) |
| user 指针 | 用户进程 (API 参数) | NULL 检查 |
| samplingInterval | 用户进程 (API 参数) | 负数校验 |
| reportInterval | 用户进程 (API 参数) | 负数校验 |
| sensorData | HDI 回调 | 长度校验 (`event->dataLen`) |

## 攻击面分析

### 1. IPC 输入验证不足

**风险等级**: 🔴 高

**描述**: 服务端从 IPC 读取参数后直接使用，未进行充分验证。

**证据**: `services/src/sensor_service.c:52-63`

```c
int32_t ActivateSensorInvoke(SensorFeatureApi *defaultApi, IpcIo *req, IpcIo *reply)
{
    int32_t sensorId = -1;
    ReadInt32(req, &sensorId);  // 从 IPC 读取，未验证范围
    // ...
    SensorUser sensorUser;
    int32_t ret = defaultApi->ActivateSensor(sensorId, &sensorUser);
    // ...
}
```

**触发条件**:
- 攻击者发送 funcId=1 的 IPC 请求
- sensorId 为任意值（在 impl 层才校验）

**影响**:
- 可能导致无效的 HDI 调用
- 潜在的拒绝服务 (DoS)

**修复建议**:
- 在 `Invoke()` 函数中增加 sensorId 范围校验
- 拒绝超出 `[0, SENSOR_TYPE_ID_MAX)` 范围的 ID

---

### 2. SensorUser 结构体未验证

**风险等级**: 🟡 中

**描述**: `ActivateSensorInvoke()` 创建空的 `SensorUser` 结构体传递给 impl 层。

**证据**: `services/src/sensor_service.c:59-60`

```c
SensorUser sensorUser;
int32_t ret = defaultApi->ActivateSensor(sensorId, &sensorUser);
```

**问题**:
- `sensorUser.name` 未初始化
- `sensorUser.callback` 为 NULL
- `sensorUser.userData` 为 NULL

**触发条件**:
- 调用 ActivateSensor/DeactivateSensor 等 API

**影响**:
- 功能异常（回调可能无法正常工作）
- 潜在的空指针解引用

**修复建议**:
- 验证或拒绝空回调的订阅请求
- 在 impl 层增加 user 参数完整性校验

---

### 3. SetBatch/SetMode/SetOption 未实现

**风险等级**: 🟡 中

**描述**: 这三个 API 在 `sensor_service_impl.c` 中仅进行参数校验，未调用底层 HDI。

**证据**: `services/src/sensor_service_impl.c:201-217`

```c
int32_t SetBatchImpl(int32_t sensorId, const SensorUser *user, 
                     int64_t samplingInterval, int64_t reportInterval)
{
    if ((sensorId >= SENSOR_TYPE_ID_MAX) || (sensorId < 0)) {
        return SENSOR_ERROR_INVALID_ID;
    }
    if ((samplingInterval < 0) || (reportInterval < 0)) {
        return SENSOR_ERROR_INVALID_PARAM;
    }
    return SENSOR_OK;  // 未调用 HDI 设置间隔
}
```

**触发条件**:
- 调用 SetBatch/SetMode/SetOption API

**影响**:
- API 调用返回成功但实际无效
- 误导开发者以为配置已生效

**修复建议**:
- 实现完整的 HDI 调用
- 或在 API 文档中明确说明当前为占位实现

---

### 4. 回调链表内存泄漏

**风险等级**: 🟡 中

**描述**: `InsertCallbackNode()` 使用 `malloc()` 分配节点，但失败时仅返回错误码，未清理已分配的节点。

**证据**: `frameworks/src/sensor_agent_proxy.c:77-80`

```c
CallbackNode *nd = (CallbackNode *)malloc(sizeof(CallbackNode));
if (nd == NULL) {
    HILOG_ERROR(HILOG_MODULE_SEN, "%s malloc failed", __func__);
    return SENSOR_ERROR_INVALID_PARAM;
}
```

**触发条件**:
- 内存不足时的订阅请求

**影响**:
- 潜在内存泄漏（如果前面的节点分配成功）
- 订阅失败但错误处理不当

**修复建议**:
- 完善内存分配失败的处理逻辑
- 添加进程退出时的资源清理

---

### 5. 全局单例 g_proxy 线程安全

**风险等级**: 🟢 低

**描述**: `sensor_agent.c` 中的 `g_proxy` 全局变量在首次访问时初始化，存在潜在的竞态条件。

**证据**: `frameworks/src/sensor_agent.c:22-25`

```c
int32_t GetAllSensors(SensorInfo **sensorInfo, int32_t *count)
{
    if (g_proxy == NULL) {
        g_proxy = GetServiceProxy();  // 非原子操作
    }
    return GetAllSensorsByProxy(g_proxy, sensorInfo, count);
}
```

**触发条件**:
- 多线程同时调用 API（LiteOS-M 可能不支持多线程）

**影响**:
- 竞态条件导致 g_proxy 被覆盖
- 潜在的段错误

**修复建议**:
- 使用原子操作或互斥锁保护初始化过程

---

### 6. IPC 缓冲区大小限制

**风险等级**: 🟢 低

**描述**: `MAX_IO_SIZE` 限制可能导致大传感器数据无法传输。

**证据**: `frameworks/include/sensor_agent_proxy.h:36`

```c
#define MAX_IO_SIZE 0x100  // 1024 bytes
```

**触发条件**:
- 传感器数据超过 1024 字节

**影响**:
- 数据截断
- 潜在的内存溢出

**修复建议**:
- 增加缓冲区大小
- 或分块传输大数据

---

### 7. 无访问控制机制

**风险等级**: 🟡 中

**描述**: 当前实现未检查调用者的权限或身份。

**问题**:
- 任何进程都可以调用 sensor_lite API
- 无签名验证或 bundle name 检查
- 无 capability 检查

**触发条件**:
- 恶意应用调用传感器 API

**影响**:
- 隐私泄露（获取传感器数据）
- 传感器资源耗尽

**修复建议**:
- 集成 OpenHarmony 权限系统
- 添加 `ohos.permission.USE_SENSOR` 权限检查
- 验证调用者 UID/Bundle Name

---

### 8. 敏感信息泄露风险

**风险等级**: 🟢 低

**描述**: `SensorInfo` 结构体包含传感器硬件信息，可能被恶意应用利用。

**证据**: `interfaces/kits/native/include/sensor_agent_type.h:112-122`

```c
typedef struct SensorInfo {
    char sensorName[SENSOR_NAME_MAX_LEN];      // 传感器名称
    char vendorName[SENSOR_NAME_MAX_LEN];       // 厂商名称
    char firmwareVersion[VERSION_MAX_LEN];      // 固件版本
    // ...
} SensorInfo;
```

**触发条件**:
- 调用 GetAllSensors API

**影响**:
- 设备硬件信息泄露
- 潜在的固件漏洞利用

**修复建议**:
- 限制敏感信息的返回
- 仅返回必要的信息给普通应用

---

## 安全建议汇总

### 紧急 (高风险)

| # | 建议 | 优先级 |
|---|------|-------|
| 1 | 在 Invoke() 中增加 sensorId 范围校验 | P0 |
| 2 | 集成权限检查机制 | P0 |
| 3 | 完善内存分配失败处理 | P1 |

### 重要 (中风险)

| # | 建议 | 优先级 |
|---|------|-------|
| 4 | 实现 SetBatch/SetMode/SetOption 的 HDI 调用 | P1 |
| 5 | 完善 SensorUser 参数校验 | P1 |
| 6 | 添加线程安全保护 | P2 |

### 建议 (低风险)

| # | 建议 | 优先级 |
|---|------|-------|
| 7 | 扩大 IPC 缓冲区 | P2 |
| 8 | 限制敏感信息返回 | P2 |

---

## 检查范围说明

本次安全评审覆盖范围：

- ✅ `services/src/sensor_service.c` - IPC 分发层
- ✅ `services/src/sensor_service_impl.c` - 业务实现层
- ✅ `frameworks/src/sensor_agent.c` - 客户端 API 层
- ✅ `frameworks/src/sensor_agent_proxy.c` - 客户端代理层
- ✅ `interfaces/kits/native/include/` - API 头文件

未覆盖范围：

- ❌ HDF Sensor HDI 内部实现 (`drivers/peripheral/sensor`)
- ❌ SAMGRlite IPC 框架内部
- ❌ 内核层传感器驱动
- ❌ 测试代码

---

## 相关文档

- [项目概览](01_Overview.md)
- [API 参考](02_API_Reference.md)
- [架构设计](03_Architecture.md)
- [构建配置](04_Build.md)
