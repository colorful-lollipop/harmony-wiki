# 内部 C++ API

本文档描述 Power Manager 模块的内部 C++ API，用于 Native 应用和系统服务调用。

## 1. 概述

### 1.1 头文件位置

```
interfaces/inner_api/native/include/
├── power_mgr_client.h          # 客户端主接口
├── running_lock.h              # 运行锁接口
├── running_lock_info.h         # 运行锁信息
├── power_state_callback.h      # 状态回调
├── power_mode_callback.h       # 模式回调
├── ipower_state_callback.h    # IPower 状态回调
├── ipower_mode_callback.h     # IPower 模式回调
├── iproximity_controller.h     # 距离控制器
├── iscreen_off_pre_callback.h # 灭屏前回调
├── iscreen_common_event_controller.h
├── power_errors.h             # 错误码
├── shutdown/                  # 关机相关
│   ├── shutdown_client.h
│   ├── isutdown_client.h
│   ├── isync_shutdown_callback.h
│   ├── iasync_shutdown_callback.h
│   ├── itake_over_shutdown_callback.h
│   ├── shutdown_priority.h
│   ├── takeover_info.h
│   └── async_shutdown_callback_stub.h
├── suspend/                   # 挂起相关
│   ├── itake_over_suspend_callback.h
│   ├── sleep_priority.h
│   └── sync_sleep_callback_ipc_interface_code.h
└── hibernate/                 # 休眠相关
    └── isync_hibernate_callback.h
```

### 1.2 使用方式

```cpp
#include "power_mgr_client.h"
#include "running_lock.h"

using namespace OHOS::PowerMgr;
```

---

## 2. PowerMgrClient

**头文件**: `interfaces/inner_api/native/include/power_mgr_client.h`

### 2.1 类定义

```cpp
class PowerMgrClient : public IPowerMgr
{
public:
    static PowerMgrClient& GetInstance();

    // 设备控制
    bool RebootDevice(const std::string& reason);
    bool ShutdownDevice(const std::string& reason);
    bool WakeupDevice(const std::string& deviceId, const std::string& wakeupReason);
    bool SuspendDevice(bool immediate);
    bool HibernateDevice(bool immediate);

    // 状态查询
    bool IsScreenOn();
    bool IsActive();
    PowerState GetState();

    // 电源模式
    bool SetPowerMode(PowerMode mode);
    PowerMode GetPowerMode();

    // 回调注册
    sptr<IPowerStateCallback> RegisterPowerStateCallback(
        const sptr<IPowerStateCallback>& callback);
    sptr<IPowerModeCallback> RegisterPowerModeCallback(
        const sptr<IPowerModeCallback>& callback);

    // 屏幕控制
    bool SetScreenOffTime(int32_t timeout);
    bool RefreshActivity(const std::string& reason);
};
```

### 2.2 使用示例

```cpp
#include "power_mgr_client.h"

auto& client = PowerMgrClient::GetInstance();

// 重启设备
client.RebootDevice("user");

// 唤醒设备
client.WakeupDevice("", "powerkey");

// 挂起设备
client.SuspendDevice(true);

// 查询状态
if (client.IsActive()) {
    // 设备活跃
}

// 注册状态回调
auto callback = new PowerStateCallbackImpl();
client.RegisterPowerStateCallback(callback);
```

### 2.3 详细接口

#### 设备控制

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `RebootDevice(reason)` | string | bool | 重启设备 |
| `ShutdownDevice(reason)` | string | bool | 关机设备 |
| `WakeupDevice(deviceId, reason)` | string, string | bool | 唤醒设备 |
| `SuspendDevice(immediate)` | bool | bool | 挂起设备 |
| `HibernateDevice(immediate)` | bool | bool | 休眠设备 |

#### 状态查询

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `IsScreenOn()` | - | bool | 屏幕是否点亮 |
| `IsActive()` | - | bool | 设备是否活跃 |
| `GetState()` | - | PowerState | 当前电源状态 |
| `GetStateMachineInfo()` | - | PowerStateMachineInfo | 状态机信息 |

#### 电源模式

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `SetPowerMode(mode)` | PowerMode | bool | 设置电源模式 |
| `GetMode()` | - | PowerMode | 获取电源模式 |

#### 回调注册

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `RegisterPowerStateCallback(cb)` | IPowerStateCallback | sptr | 注册状态回调 |
| `RegisterPowerModeCallback(cb)` | IPowerModeCallback | sptr | 注册模式回调 |
| `RegisterShutdownCallback(cb)` | IShutdownCallback | sptr | 注册关机回调 |

#### 屏幕控制

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `SetScreenOffTime(ms)` | int32_t | bool | 设置灭屏超时 |
| `RefreshActivity(reason)` | string | bool | 刷新活动状态 |

---

## 3. RunningLock

**头文件**: `interfaces/inner_api/native/include/running_lock.h`

### 3.1 类定义

```cpp
class RunningLock : public RefBase
{
public:
    virtual bool Lock(uint32_t timeout) = 0;
    virtual void Unlock() = 0;
    virtual bool IsUsed() const = 0;
    virtual const std::string& GetName() const = 0;
    virtual RunningLockType GetType() const = 0;
    virtual void SetWorkTriggerList(WorkMode workMode) = 0;

protected:
    virtual ~RunningLock() = default;
};
```

### 3.2 RunningLockMgr

**头文件**: `interfaces/inner_api/native/include/running_lock.h`

```cpp
class RunningLockMgr : public RefBase
{
public:
    static sptr<RunningLock> CreateRunningLock(
        const std::string& name,
        RunningLockType type,
        const WorkTriggerList& triggerList = {});

    static bool IsSupported(RunningLockType type);
    static void Dump(std::string& result);
};
```

### 3.3 使用示例

```cpp
#include "running_lock.h"

auto& client = PowerMgrClient::GetInstance();

// 创建运行锁
sptr<RunningLock> lock = client.CreateRunningLock(
    "BackgroundDownload",
    RunningLockType::RUNNINGLOCK_BACKGROUND
);

if (lock != nullptr) {
    // 加锁（无限期）
    lock->Lock(0);

    // 业务逻辑...

    // 解锁
    lock->Unlock();
}

// 检查是否支持
if (RunningLockMgr::IsSupported(RunningLockType::RUNNINGLOCK_BACKGROUND_USER_IDLE)) {
    // 支持用户空闲锁
}
```

### 3.4 运行锁类型

```cpp
enum class RunningLockType : uint32_t {
    RUNNINGLOCK_BACKGROUND = 1,           // 后台保活锁
    RUNNINGLOCK_PROXIMITY_SCREEN_CONTROL, // 距离感应锁
    RUNNINGLOCK_BACKGROUND_USER_IDLE,     // 用户空闲检测锁
};
```

---

## 4. 回调接口

### 4.1 IPowerStateCallback

**头文件**: `interfaces/inner_api/native/include/ipower_state_callback.h`

```cpp
class IPowerStateCallback : public IRemoteBroker
{
public:
    virtual void OnPowerStateChanged(PowerState state) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.IPowerStateCallback");
};
```

### 4.2 IPowerModeCallback

**头文件**: `interfaces/inner_api/native/include/ipower_mode_callback.h`

```cpp
class IPowerModeCallback : public IRemoteBroker
{
public:
    virtual void OnPowerModeChanged(PowerMode mode) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.IPowerModeCallback");
};
```

### 4.3 IShutdownCallback

**头文件**: `interfaces/inner_api/native/include/shutdown/isync_shutdown_callback.h`

```cpp
class ISyncShutdownCallback : public IRemoteBroker
{
public:
    virtual void OnShutdownCallback(ShutdownCallbackType type,
        const std::string& reason) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.ISyncShutdownCallback");
};
```

### 4.4 ITakeOverSuspendCallback

**头文件**: `interfaces/inner_api/native/include/suspend/itake_over_suspend_callback.h`

```cpp
class ITakeOverSuspendCallback : public IRemoteBroker
{
public:
    virtual bool TakeOverSuspend(bool isImmediate) = 0;
    virtual void TakeOverWakeup() = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.powermgr.ITakeOverSuspendCallback");
};
```

---

## 5. 关机相关 API

**头文件目录**: `interfaces/inner_api/native/include/shutdown/`

### 5.1 ShutdownClient

```cpp
class ShutdownClient
{
public:
    static ShutdownClient& GetInstance();

    bool Shutdown(const std::string& reason, bool isReboot);
    bool RegisterShutdownCallback(ShutdownPriority priority,
        const sptr<ISyncShutdownCallback>& callback);
    bool UnregisterShutdownCallback(const sptr<ISyncShutdownCallback>& callback);
};
```

### 5.2 TakeoverInfo

```cpp
struct TakeoverInfo {
    bool isReboot;              // 是否重启
    std::string reason;         // 原因
    int32_t uid;                // 调用方 UID
    int32_t pid;                // 调用方 PID
    std::string bundleName;      // 包名
};
```

---

## 6. 电源状态

### 6.1 PowerState 枚举

```cpp
enum class PowerState : uint32_t {
    AWAKE = 0,           // 唤醒状态
    ACTIVE,              // 活跃状态
    INACTIVE,            // 非活跃
    SUSPENDING,          // 挂起中
    SUSPEND,             // 已挂起
    DOZING,              // 低功耗显示
    SLEEPING,            // 休眠中
    SHUTTING_DOWN,       // 关机中
    SHUTDOWN,            // 已关机
};
```

### 6.2 PowerMode 枚举

```cpp
enum class PowerMode : uint32_t {
    NORMAL_MODE = 0,             // 正常模式
    POWER_SAVE_MODE = 1,        // 省电模式
    PERFORMANCE_MODE = 2,       // 性能模式
    EXTREME_POWER_SAVE_MODE = 3, // 超级省电模式
    CUSTOM_POWER_SAVE_MODE = 4,  // 自定义模式
};
```

---

## 7. 错误码

**头文件**: `interfaces/inner_api/native/include/power_errors.h`

```cpp
enum {
    ERR_OK = 0,
    ERR_PARAM_INVALID = 401,         // 参数无效
    ERR_NO_INIT = 1001,              // 未初始化
    ERR_SERVICE_NOT_READY = 1002,    // 服务未就绪
    ERR_IPC = 4603001,              // IPC 错误
    ERR_SHELL_PID_DIED = 4603002,    // Shell 进程死亡
};
```

---

## 8. 关键实现文件

| 接口 | 实现文件 | 说明 |
|------|----------|------|
| PowerMgrClient | `frameworks/native/power_mgr_client.cpp` | 客户端实现 |
| RunningLock | `frameworks/native/running_lock.cpp` | 运行锁实现 |
| ShutdownClient | `frameworks/native/shutdown_client.cpp` | 关机客户端 |
| PowerMgrService | `services/native/src/power_mgr_service.cpp` | 服务端 |

---

## 9. 使用注意事项

1. **单例模式**: `PowerMgrClient` 是单例，通过 `GetInstance()` 获取
2. **线程安全**: 大部分方法可在任意线程调用，详情见具体实现
3. **回调释放**: 注册回调后需自行管理生命周期
4. **权限检查**: 部分 API 需要权限验证
