# 架构设计

## 整体架构

sensor_lite 采用 **客户端-服务器 (Client-Server)** 架构，通过 **SAMGRlite IPC 框架**进行进程间通信。

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Application Layer                                │
│                                                                              │
│   使用 sensor_agent.h 中的 8 个 C API:                                       │
│   - GetAllSensors, ActivateSensor, DeactivateSensor                           │
│   - SetBatch, SubscribeSensor, UnsubscribeSensor                              │
│   - SetMode, SetOption                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────┬─────────────┘
                                                                  │
                                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         sensor_client (共享库)                                │
│                                                                              │
│   文件: frameworks/src/sensor_agent.c                                        │
│                                                                              │
│   职责:                                                                      │
│   - 延迟初始化全局代理 g_proxy                                               │
│   - 封装 IPC 调用（根据内核类型选择不同实现）                                  │
│   - 管理回调链表 (g_callbackNodes)                                           │
│   - 序列化/反序列化 IPC 数据                                                │
│                                                                              │
│   条件编译:                                                                  │
│   - liteos_riscv: sensor_agent_client.c                                      │
│   - liteos_a/linux: sensor_agent_proxy.c                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────┬─────────────┘
                                                                  │
                              IPC (SAMGRlite)
                                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       sensor_service (可执行文件)                              │
│                                                                              │
│   文件: services/src/sensor_service.c                                        │
│                                                                              │
│   职责:                                                                      │
│   - 注册 SAMGRlite 服务 ("sensor_service")                                    │
│   - Invoke() 函数分发 IPC 请求                                               │
│   - g_invokeFuncList[] 路由表 (8 个函数)                                     │
│   - 异步数据上报 (TF_OP_ASYNC)                                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────┬─────────────┘
                                                                  │
                                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    sensor_service_impl.c (业务实现)                            │
│                                                                              │
│   文件: services/src/sensor_service_impl.c                                   │
│                                                                              │
│   职责:                                                                      │
│   - 参数校验 (sensorId, user, interval)                                      │
│   - HDI 条件调用 (#ifdef HAS_HDI_SENSOR_LITE_PRAT)                          │
│   - 传感器列表管理 (g_sensorLists)                                           │
│   - 数据回调注册 (SensorDataCallback)                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────┬─────────────┘
                                                                  │
                              HDI 调用
                                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     HDF Sensor HDI (硬件驱动抽象层)                           │
│                                                                              │
│   组件: drivers/peripheral/sensor/hal:hdi_sensor                            │
│                                                                              │
│   条件编译: has_drivers_peripheral_sensor_part                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 模块职责

### 1. 客户端库 (sensor_client)

**文件**: `frameworks/src/`

| 文件 | 职责 | 内核类型 |
|-----|------|---------|
| `sensor_agent.c` | 8 个 API 的代理入口 | 通用 |
| `sensor_agent_client.c` | IPC 客户端实现 | liteos_riscv |
| `sensor_agent_proxy.c` | IPC 代理实现 | liteos_a/linux |

**关键全局变量**:

| 变量 | 类型 | 职责 |
|-----|------|------|
| `g_proxy` | `void*` | 服务端代理句柄 |
| `g_sensorLists` | `SensorInfo*` | 传感器列表缓存 |
| `g_sensorEvent` | `SensorEvent*` | 传感器事件缓存 |
| `g_callbackNodes[]` | `CallbackNode[]` | 回调链表数组 |

**证据**: `frameworks/src/sensor_agent_proxy.c:24-45`

### 2. 服务端 (sensor_service)

**文件**: `services/src/`

| 文件 | 职责 |
|-----|------|
| `sensor_service.c` | IPC 分发、函数路由、服务注册 |
| `sensor_service_impl.c` | 业务逻辑、HDI 调用 |
| `proc.c` | 服务进程入口 |

**服务注册**:

```c
// services/src/sensor_service.c:227-234
SAMGR_GetInstance()->RegisterService((Service *)&g_sensorService);
SAMGR_GetInstance()->RegisterDefaultFeatureApi(SENSOR_SERVICE, GET_IUNKNOWN(g_sensorService));
SYSEX_SERVICE_INIT(Init);
```

**证据**: `services/src/sensor_service.c:227-234`

### 3. IPC 路由表

```c
// services/src/sensor_service.c:183-192
static InvokeFunc g_invokeFuncList[] = {
    GetAllSensorsInvoke,      // funcId = 0
    ActivateSensorInvoke,     // funcId = 1
    DeactivateSensorInvoke,   // funcId = 2
    SetBatchInvoke,           // funcId = 3
    SubscribeSensorInvoke,    // funcId = 4
    UnsubscribeSensorInvoke,  // funcId = 5
    SetModeInvoke,            // funcId = 6
    SetOptionInvoke,          // funcId = 7
};
```

**证据**: `services/src/sensor_service.c:183-192`

## 数据流

### 同步调用流程 (GetAllSensors)

```
Client  ──IPC(IpcIo)──▶ sensor_service (Invoke) ──▶ sensor_service_impl.c
                │                                        │
                │◀─────── 返回值 ───────────────────────┘
                │
              IpcIo
```

### 异步数据上报流程 (SubscribeSensor)

```
Client ──Subscribe──▶ sensor_service ──HDI Register──▶ HDF Sensor HDI
        │                    │                               │
        │◀──ACK─────────────┘                               │
        │                                              传感器数据到达
        │◀──Async Data(IpcIo)◀──────────────────────────────┘
```

## 线程模型

### 服务端线程配置

```c
// services/src/sensor_service_impl.c:114-118
TaskConfig GetTaskConfig(Service *service)
{
    TaskConfig config = {LEVEL_HIGH, PRI_BELOW_NORMAL, 
                         TASK_CONFIG_STACK_SIZE, TASK_CONFIG_QUEUE_SIZE, 
                         SHARED_TASK};
    return config;
}
```

**证据**: `services/src/sensor_service_impl.c:114-118`

| 配置项 | 值 | 说明 |
|-------|-----|------|
| priority | LEVEL_HIGH | 高优先级任务 |
| scheduling | SHARED_TASK | 共享任务池 |
| stack size | TASK_CONFIG_STACK_SIZE | 默认栈大小 |
| queue size | TASK_CONFIG_QUEUE_SIZE | 默认队列大小 |

## 内存管理

### 客户端回调链表

```c
// frameworks/src/sensor_agent_proxy.c:33-45
typedef struct CallbackNode {
    RecordSensorCallback callback;
    void *next;
} CallbackNode;

CallbackNode g_callbackNodes[(int32_t)SENSOR_TYPE_ID_MAX] = {
    {NULL, NULL}, {NULL, NULL}, ...  // 30 个节点
};
```

**分配**: `InsertCallbackNode()` 中使用 `malloc()` 分配新节点
**释放**: `DeleteCallbackNode()` 中使用 `free()` 释放节点

**证据**: `frameworks/src/sensor_agent_proxy.c:60-86`

---

## 相关文档

- [项目概览](01_Overview.md)
- [API 参考](02_API_Reference.md)
- [构建配置](04_Build.md)
- [安全评审](05_Security.md)
