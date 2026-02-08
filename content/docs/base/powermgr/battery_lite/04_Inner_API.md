# 内部 API 文档

> **目的**: 说明 battery_lite 内部模块之间的接口、依赖方向和稳定性标注。  
> **适用范围**: 需要深入理解代码实现或进行二次开发的开发者。  
> **关键结论**: 内部 API 分为服务层接口（`IBattery`）、框架层接口（`BatteryInterface`）和公共头文件（`battery_info.h`）。

---

## API 分层结构

```
┌─────────────────────────────────────────────────────┐
│                  应用层 (Applications)               │
├─────────────────────────────────────────────────────┤
│  interfaces/kits/battery_info.h                      │
│  (稳定) 公共函数接口，公开给所有应用使用              │
├─────────────────────────────────────────────────────┤
│  frameworks/native/                                  │
│  (稳定) Native 框架层，封装 IPC 调用                  │
├─────────────────────────────────────────────────────┤
│  services/                                           │
│  (不稳定) 服务层，实现核心电池逻辑                    │
├─────────────────────────────────────────────────────┤
│  samgr_lite IPC                                      │
│  (系统层) 进程间通信基础设施                          │
└─────────────────────────────────────────────────────┘
```

---

## 公共头文件 API (稳定)

### battery_info.h

**位置**: `interfaces/kits/battery_info.h`  
**稳定性**: **高** - 对外公开接口，语义不变  
**适用范围**: 所有应用代码

#### 函数声明

```c
// 获取电池剩余电量 (0-100%)
int32_t GetBatSoc(void);

// 获取充电状态
BatteryChargeState GetChargingStatus(void);

// 获取电池健康状态
BatteryHealthState GetHealthStatus(void);

// 获取连接类型
BatteryPluggedType GetPluggedType(void);

// 获取电池电压 (mV)
int32_t GetBatVoltage(void);

// 获取电池技术型号
char* GetBatTechnology(void);

// 获取电池温度 (0.1℃)
int32_t GetBatTemperature(void);
```

**证据**: `interfaces/kits/battery_info.h:100-106` 声明了所有公共 API。

#### 枚举类型

**BatteryChargeState** (`interfaces/kits/battery_info.h:23-44`)

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `CHARGE_STATE_NONE` | 0 | 放电中 |
| `CHARGE_STATE_ENABLE` | 1 | 充电中 |
| `CHARGE_STATE_DISABLE` | 2 | 未充电 |
| `CHARGE_STATE_FULL` | 3 | 已充满 |
| `CHARGE_STATE_BUTT` | 4 | 枚举上限 |

**BatteryHealthState** (`interfaces/kits/battery_info.h:46-75`)

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `HEALTH_STATE_UNKNOWN` | 0 | 未知 |
| `HEALTH_STATE_GOOD` | 1 | 良好 |
| `HEALTH_STATE_OVERHEAT` | 2 | 过热 |
| `HEALTH_STATE_OVERVOLTAGE` | 3 | 过压 |
| `HEALTH_STATE_COLD` | 4 | 过冷 |
| `HEALTH_STATE_DEAD` | 5 | 失效 |
| `HEALTH_STATE_BUTT` | 6 | 枚举上限 |

**BatteryPluggedType** (`interfaces/kits/battery_info.h:77-98`)

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `PLUGGED_TYPE_NONE` | 0 | 未连接 |
| `PLUGGED_TYPE_AC` | 1 | 交流充电 |
| `PLUGGED_TYPE_USB` | 2 | USB 充电 |
| `PLUGGED_TYPE_WIRELESS` | 3 | 无线充电 |
| `PLUGGED_TYPE_BUTT` | 4 | 枚举上限 |

---

## 框架层 API (稳定)

### battery_framework.h

**位置**: `frameworks/native/include/battery_framework.h`  
**稳定性**: **高** - Native 框架接口  
**适用范围**: Native 应用

#### GetBatteryIUnknown()

```c
/**
 * @brief 获取电池服务的 IUnknown 接口
 * @return IUnknown* 电池服务接口，失败返回 NULL
 * @note 内部使用互斥锁保护，多线程安全
 */
static inline IUnknown *GetBatteryIUnknown(void)
{
    IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(
        BATTERY_SERVICE, BATTERY_INNER);
    if (iUnknown == NULL) {
        POWER_HILOGE("[SERVICE:%s]:BatteryClient::ChargingApiGet iUnknown is null", 
                     BATTERY_SERVICE);
        return NULL;
    }
    return iUnknown;
}
```

**证据**: `frameworks/native/include/battery_framework.h:31-39`。

### batterymgr_intf_define.h

**位置**: `frameworks/native/include/batterymgr_intf_define.h`  
**稳定性**: **高** - 接口定义宏  
**适用范围**: 框架内部

#### INHERIT_BATTERY_INTERFACE

```c
#define INHERIT_BATTERY_INTERFACE                                                  \
    int32_t (*GetBatSocFunc)(IUnknown *iUnknown);                                  \
    BatteryChargeState (*GetChargingStatusFunc)(IUnknown *iUnknown);               \
    BatteryHealthState (*GetHealthStatusFunc)(IUnknown *iUnknown);                 \
    BatteryPluggedType (*GetPluggedTypeFunc)(IUnknown *iUnknown);                  \
    int32_t (*GetBatVoltageFunc)(IUnknown *iUnknown);                              \
    char* (*GetBatTechnologyFunc)(IUnknown *iUnknown);                             \
    int32_t (*GetBatTemperatureFunc)(IUnknown *iUnknown)
```

**证据**: `frameworks/native/include/batterymgr_intf_define.h:26-33`。

---

## 服务层 API (不稳定)

### ibattery.h

**位置**: `services/include/ibattery.h`  
**稳定性**: **中** - 服务内部接口，可能变更  
**适用范围**: 服务框架内部

#### IBattery 接口结构

```c
typedef struct IBattery {
    int32_t (*GetSoc)();
    BatteryChargeState (*GetChargingStatus)();
    BatteryHealthState (*GetHealthStatus)();
    BatteryPluggedType (*GetPluggedType)();
    int32_t (*GetVoltage)();
    char* (*GetTechnology)();
    int32_t (*GetTemperature)();
    int (*TurnOnLed)(int red, int green, int blue);
    int (*TurnOffLed)();
    int (*SetLedColor)(int red, int green, int blue);
    int (*GetLedColor)(int* red, int* green, int* blue);
    void (*ShutDown)();
    void (*UpdateBatInfo)(BatInfo*);
} IBattery;
```

**证据**: `services/include/ibattery.h:45-59`。

#### BatInfo 数据结构

```c
typedef struct {
    int32_t batSoc;              // 电池电量
    int32_t batVoltage;          // 电池电压
    int32_t BatTemp;             // 电池温度
    int32_t batCapacity;         // 电池容量
    BatteryChargeState chargingStatus;  // 充电状态
    BatteryPluggedType pluggedType;     // 连接类型
    char BatTechnology[64];      // 电池技术 (长度固定)
    BatteryHealthState healthStatus;    // 健康状态
} BatInfo;
```

**证据**: `services/include/ibattery.h:26-43`。

#### 接口创建/销毁

```c
/**
 * @brief 创建电池接口实例
 * @return IBattery* 电池接口指针，失败返回 NULL
 */
IBattery *NewBatterInterfaceInstance(void);

/**
 * @brief 释放电池接口实例
 * @return int32_t 错误码，0 表示成功
 */
int32_t FreeBatterInterfaceInstance(void);
```

**证据**: `services/include/ibattery.h:61-62`。

---

## 依赖方向

```
                    interfaces/kits/battery_info.h
                              ↓
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│                  Native 应用层代码                        │
│         (调用 GetBatSoc() 等公共 API)                      │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│            frameworks/native/battery_framework.c          │
│            (封装 SAMgr IPC 调用)                          │
│                                                      │
│  GetBatteryIUnknown() ──→ SAMGR_GetFeatureApi()        │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│            services/battery_manage_feature.c             │
│            (特征实现，代理调用)                           │
│                                                      │
│  BatterySocImpl() ──→ NewBatterInterfaceInstance()     │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│              services/battery_device.c                    │
│              (服务实现，真实数据)                         │
│                                                      │
│  GetSocImpl() ──→ battInfo.batSoc                      │
└─────────────────────────────────────────────────────────┘
```

---

## 线程安全说明

### 互斥锁保护

框架层使用互斥锁保护接口获取：

```c
// battery_framework.c
static pthread_mutex_t g_mutex = PTHREAD_MUTEX_INITIALIZER;
static BatteryInterface *g_intf = NULL;

static BatteryInterface *GetBatteryInterface(void)
{
    pthread_mutex_lock(&g_mutex);  // 加锁
    if (g_intf != NULL) {
        pthread_mutex_unlock(&g_mutex);  // 快速路径解锁
        return g_intf;
    }
    // ... 初始化接口
    pthread_mutex_unlock(&g_mutex);  // 解锁
    return g_intf;
}
```

**证据**: `frameworks/native/src/mini/battery_framework.c:22-44`。

### 服务层线程模型

服务运行在独立线程，采用消息队列处理请求：

```c
static TaskConfig GetTaskConfig(Service *service)
{
    TaskConfig config = {
        LEVEL_HIGH,                    // 优先级级别
        PRI_BELOW_NORMAL,               // 线程优先级
        TASK_CONFIG_STACK_SIZE,        // 栈大小 0x800
        TASK_CONFIG_QUEUE_SIZE,        // 队列大小 20
        SHARED_TASK                    // 共享任务模式
    };
    return config;
}
```

**证据**: `services/src/battery_device.c:54-59`。

---

## 错误码定义

### 服务层错误码

| 常量 | 值 | 说明 |
|------|-----|------|
| `BATTERY_OK` | 0 | 成功 |
| `BATTERY_ERROR_UNKNOWN` | -1 | 未知错误 |
| `BATTERY_ERROR_INVALID_ID` | -2 | 无效 ID |
| `BATTERY_ERROR_INVALID_PARAM` | -3 | 无效参数 |

**证据**: `services/include/battery_device.h:37-40`。

### 框架层返回值

| 返回值 | 含义 |
|--------|------|
| `EC_SUCCESS` (0) | 成功 |
| `EC_FAILURE` (-1) | 失败 |
| `NULL` | 接口不可用 |

**证据**: `frameworks/native/src/mini/battery_framework.c:39,49`。

---

## 可替换点

### HAL 层替换

电池数据获取逻辑位于 `services/src/battery_device.c`，可替换为真实的 HAL 调用：

```c
// 当前实现：返回模拟数据
int32_t GetSocImpl(void)
{
    return battInfo.batSoc;  // 直接返回静态数据
}

// 替换为 HAL 调用示例
int32_t GetSocImpl(void)
{
    return HAL_Battery_GetSOC();  // 调用硬件抽象层
}
```

### LED 控制替换

LED 控制接口当前为空实现，可替换为真实 GPIO 控制：

```c
// 当前实现：空操作
int TurnOnLedImpl(int red, int green, int blue)
{
    (void)red;
    (void)green;
    (void)blue;
    return BATTERY_OK;
}

// 替换为 GPIO 控制
int TurnOnLedImpl(int red, int green, int blue)
{
    GPIO_WritePin(LED_RED_PIN, red > 0 ? GPIO_HIGH : GPIO_LOW);
    GPIO_WritePin(LED_GREEN_PIN, green > 0 ? GPIO_HIGH : GPIO_LOW);
    GPIO_WritePin(LED_BLUE_PIN, blue > 0 ? GPIO_HIGH : GPIO_LOW);
    return BATTERY_OK;
}
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [N-API 接口](03_N_API.md) | JS API 详细说明 |
| [架构说明](02_Architecture.md) | 组件和数据流 |
| [GN 构建](05_GN_Build.md) | 构建 Targets 和产物 |
| [安全评审](06_Security_Review.md) | API 安全考量 |
