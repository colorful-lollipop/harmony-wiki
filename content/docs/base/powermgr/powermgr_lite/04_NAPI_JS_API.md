# 对外 API (N-API/JSI)

> **目的**: 完整的对外 API 参考,包括 C API 和 JavaScript JSI 绑定
> **适用范围**: 应用开发者、JS 开发者
> **阅读时间**: 30分钟

---

## 概述

**注意**: powermgr_lite 使用 **JSI (JavaScript Interface)** 而非 N-API。JSI 是 OpenHarmony ACE Engine Lite 的轻量级 JS/C++ 绑定机制。

### API 分类
1. **C API**: `interfaces/kits/running_lock.h` - 主要电源管理接口
2. **JS API**: `interfaces/kits/battery/js/builtin/` - 仅电池状态查询

---

## C API

### 头文件
```c
#include <ohos_types.h>
#include "interfaces/kits/running_lock.h"
#include "interfaces/innerkits/power_manage.h"
```

### RunningLock API

#### 数据类型

```c
// 锁类型
typedef enum {
    RUNNINGLOCK_SCREEN,                       // 保持屏幕常亮
    RUNNINGLOCK_BACKGROUND,                   // 保持 CPU 运行
    RUNNINGLOCK_PROXIMITY_SCREEN_CONTROL,      // 距离传感器控制
    RUNNINGLOCK_BUTT
} RunningLockType;

// 锁标志
typedef enum {
    RUNNINGLOCK_FLAG_NONE = 0,
    RUNNINGLOCK_FLAG_WAKEUP_WHEN_ACQUIRED = 1 << 0,  // 获取时唤醒设备
} RunningLockFlag;

// 锁对象
#define RUNNING_LOCK_NAME_LEN   64

typedef struct {
    char name[RUNNING_LOCK_NAME_LEN];  // 锁名称 (调试用)
    RunningLockType type;              // 锁类型
    RunningLockFlag flag;              // 锁标志
} RunningLock;
```

#### 函数原型

| 函数 | 返回值 | 参数 | 说明 |
|------|--------|------|------|
| `CreateRunningLock` | `const RunningLock*` | `name`, `type`, `flag` | 创建运行锁对象 |
| `DestroyRunningLock` | `void` | `const RunningLock*` | 销毁运行锁对象 |
| `AcquireRunningLock` | `BOOL` | `const RunningLock*` | 获取运行锁 |
| `ReleaseRunningLock` | `BOOL` | `const RunningLock*` | 释放运行锁 |
| `IsRunningLockHolding` | `BOOL` | `const RunningLock*` | 检查锁是否被持有 |

#### API 详细说明

##### CreateRunningLock
```c
const RunningLock *CreateRunningLock(
    const char *name,
    RunningLockType type,
    RunningLockFlag flag
);
```

**参数**:
- `name`: 锁名称 (最大 64 字符)
  - 不能为 NULL
  - 建议使用描述性名称 (如 "video_playback")
- `type`: 锁类型 (见 `RunningLockType` 枚举)
- `flag`: 锁标志 (见 `RunningLockFlag` 枚举)

**返回值**:
- 成功: 返回 `RunningLock*` 指针
- 失败: 返回 `NULL`

**错误条件**:
- `name` 为 NULL
- `type` 无效 (>= `RUNNINGLOCK_BUTT`)
- 内存分配失败
- 超时 (平台相关)

**线程安全**: 是
**证据**: `frameworks/src/running_lock.c:79-103`

##### DestroyRunningLock
```c
void DestroyRunningLock(const RunningLock *lock);
```

**参数**:
- `lock`: 要销毁的锁对象

**前置条件**:
- 锁必须已释放 (`IsRunningLockHolding(lock) == FALSE`)

**线程安全**: 是
**证据**: `frameworks/src/running_lock.c:105-120`

##### AcquireRunningLock
```c
BOOL AcquireRunningLock(const RunningLock *lock);
```

**参数**:
- `lock`: 运行锁对象

**返回值**:
- `TRUE`: 成功获取
- `FALSE`: 失败 (超时或系统错误)

**行为**:
- 如果设置了 `RUNNINGLOCK_FLAG_WAKEUP_WHEN_ACQUIRED`,设备会被唤醒
- 超时时间由平台决定 (通常 5 秒)

**线程安全**: 是
**证据**: `frameworks/src/running_lock.c:37-48`

##### ReleaseRunningLock
```c
BOOL ReleaseRunningLock(const RunningLock *lock);
```

**参数**:
- `lock`: 运行锁对象

**返回值**:
- `TRUE`: 成功释放
- `FALSE`: 失败 (锁未持有或系统错误)

**线程安全**: 是
**证据**: `frameworks/src/running_lock.c:50-61`

##### IsRunningLockHolding
```c
BOOL IsRunningLockHolding(const RunningLock *lock);
```

**参数**:
- `lock`: 运行锁对象

**返回值**:
- `TRUE`: 锁当前被持有
- `FALSE`: 锁未被持有

**线程安全**: 是
**证据**: `interfaces/kits/running_lock.h:64`

### Power Management API

#### 数据类型

```c
// 挂起原因
typedef enum {
    SUSPEND_DEVICE_REASON_APPLICATION = 0,     // 应用主动请求
    SUSPEND_DEVICE_REASON_DEVICE_ADMIN = 1,    // 设备管理员
    SUSPEND_DEVICE_REASON_TIMEOUT = 2,           // 超时挂起
    SUSPEND_DEVICE_REASON_LID_SWITCH = 3,        // 合盖事件
    SUSPEND_DEVICE_REASON_POWER_BUTTON = 4,      // 电源按钮
    SUSPEND_DEVICE_REASON_HDMI = 5,              // HDMI 事件
    SUSPEND_DEVICE_REASON_SLEEP_BUTTON = 6,    // 睡眠按钮
    SUSPEND_DEVICE_REASON_ACCESSIBILITY = 7,     // 无障碍服务
    SUSPEND_DEVICE_REASON_FORCE_SUSPEND = 8,     // 强制挂起
} SuspendDeviceType;

// 唤醒原因
typedef enum {
    WAKEUP_DEVICE_UNKNOWN = 0,               // 未知
    WAKEUP_DEVICE_POWER_BUTTON = 1,          // 电源按钮
    WAKEUP_DEVICE_APPLICATION = 2,            // 应用主动唤醒
    WAKEUP_DEVICE_PLUGGED_IN = 3,           // 充电器插入
    WAKEUP_DEVICE_GESTURE = 4,               // 手势识别
    WAKEUP_DEVICE_CAMERA_LAUNCH = 5,        // 相机启动
    WAKEUP_DEVICE_WAKE_KEY = 6,              // 唤醒键
    WAKEUP_DEVICE_WAKE_MOTION = 7,           // 唤醒动作
    WAKEUP_DEVICE_HDMI = 8,                  // HDMI 事件
    WAKEUP_DEVICE_LID = 9,                  // 开盖事件
} WakeupDeviceType;
```

#### 函数原型

| 函数 | 返回值 | 参数 | 说明 |
|------|--------|------|------|
| `SuspendDevice` | `void` | `reason`, `suspendImmed` | 挂起设备 |
| `WakeupDevice` | `void` | `reason`, `details` | 唤醒设备 |

#### API 详细说明

##### SuspendDevice
```c
void SuspendDevice(
    SuspendDeviceType reason,
    BOOL suspendImmed
);
```

**参数**:
- `reason`: 挂起原因 (见 `SuspendDeviceType` 枚举)
- `suspendImmed`: 是否立即挂起
  - `TRUE`: 立即挂起 (忽略运行锁)
  - `FALSE`: 等待所有运行锁释放后挂起

**安全注意**: ⚠️ **当前无权限检查**
- 任何能调用此 API 的进程都可以挂起设备
- 证据: `services/src/power_manage_feature.c:82-97` 有 TODO 注释

**线程安全**: 是
**证据**: `frameworks/src/small/power_manage.c:131-138` (IPC 调用)

##### WakeupDevice
```c
void WakeupDevice(
    WakeupDeviceType reason,
    const char* details
);
```

**参数**:
- `reason`: 唤醒原因 (见 `WakeupDeviceType` 枚举)
- `details`: 唤醒详情字符串 (可为 NULL)
  - 用于调试和日志
  - 最大长度未定义

**安全注意**: ⚠️ **当前无权限检查**
- 任何能调用此 API 的进程都可以唤醒设备
- 证据: `services/src/power_manage_feature.c:92-98` 有 TODO 注释

**线程安全**: 是
**证据**: `frameworks/src/small/power_manage.c:140-147` (IPC 调用)

---

## JavaScript JSI API

### 概述

**注意**: 当前仅暴露**电池状态** API,无电源控制接口。

### 模块名称
```javascript
import battery from '@ohos.powermgr_lite';
```

### API 列表

| JS API 名称 | C++ 实现 | 同步/异步 | 说明 |
|------------|----------|-----------|------|
| `battery.getStatus` | `BatteryModule::GetStatus()` | 异步 (callback) | 获取电池状态 |

### API 详细说明

#### battery.getStatus

**原型**:
```javascript
battery.getStatus({
    success: function(data) {
        // data.charging: boolean - 是否正在充电
        // data.level: number - 电量 (0-100)
    },
    complete: function() {
        // 完成回调 (可选)
    }
});
```

**参数**:
- `success`: 成功回调 (必需)
  - 参数 `data`: 对象,包含 `charging` 和 `level`
- `complete`: 完成回调 (可选)

**返回值**:
- 无 (通过回调返回结果)

**错误处理**:
- 如果参数无效,函数静默失败 (无错误回调)

**同步/异步**:
- 异步 (通过 callback)
- 无 Promise 支持

**证据**: `interfaces/kits/battery/js/builtin/src/battery_module.cpp:42-58`

#### 实现细节

**C++ 实现**:
```cpp
// 注册
void InitBatteryModule(JSIValue exports) {
    JSI::SetModuleAPI(exports, "getStatus", BatteryModule::GetStatus);
}

// 实现
JSIValue BatteryModule::GetStatus(
    const JSIValue thisVal,
    const JSIValue* args,
    uint8_t argsNum
) {
    // 1. 参数验证
    if ((args == nullptr) || (argsNum == 0) || JSI::ValueIsUndefined(args[0])) {
        return JSI::CreateUndefined();
    }

    // 2. 调用 C 接口
    double level = 0;
    bool charging = false;
    (void)GetBatteryStatus(&charging, &level);

    // 3. 构造返回对象
    JSIValue result = JSI::CreateObject();
    JSI::SetBooleanProperty(result, "charging", charging);
    JSI::SetNumberProperty(result, "level", level);

    // 4. 调用成功回调
    SuccessCallBack(thisVal, args[0], result);
    JSI::ReleaseValue(result);

    return JSI::CreateUndefined();
}
```

**电池接口**:
```cpp
// 外部实现 (不在本仓库)
// 声明在: interfaces/kits/battery/js/builtin/include/battery_impl.h
extern int GetBatteryStatus(bool* charging, double* level);
```

---

## 内部接口映射

### C API → 内部调用链

#### AcquireRunningLock 流程
```
C API: AcquireRunningLock(lock)
    ↓
Framework: frameworks/src/running_lock.c:37-48
    ↓ (验证锁存在性)
    ↓
[Small] Framework: frameworks/src/small/power_manage.c:123-139
    ↓ (通过 SAMGR Lite IPC)
    ↓
Service: services/src/power/small/power_manage_feature_impl.c:66-81
    ↓ (FeatureInvoke → AcquireInvoke)
    ↓
Service: services/src/running_lock_mgr.c:27-43
    ↓ (RunningLockMgrAcquireEntry)
    ↓
Service: services/src/power/running_lock_hub.c:22-34
    ↓ (RunningLockHubLock)
    ↓
Platform: services/src/power/small/running_lock_handler.c:30-61
    ↓ (write("/proc/power/power_lock", name))
```

#### SuspendDevice 流程
```
C API: SuspendDevice(reason, suspendImmed)
    ↓
[Small] Framework: frameworks/src/small/power_manage.c:131-138
    ↓ (通过 SAMGR Lite IPC)
    ↓
Service: services/src/power/small/power_manage_feature_impl.c:82-97
    ↓ (FeatureInvoke → SuspendInvoke)
    ↓
Service: services/src/power/suspend_controller.c:32-57
    ↓ (DisableSuspend)
    ↓
Platform: WakeupHolder 释放
```

---

## 错误码

### C API 错误码
| 返回值 | 含义 | 说明 |
|--------|------|------|
| `FALSE` | 操作失败 | API 失败时返回 `FALSE` |
| `TRUE` | 操作成功 | API 成功时返回 `TRUE` |
| `NULL` | 创建失败 | `CreateRunningLock` 失败时返回 `NULL` |

### JS API 错误处理
- 无显式错误码
- 失败时静默 (通过回调返回 undefined)

---

## 权限要求

### C API
| API | 权限要求 | 实际状态 |
|------|----------|---------|
| CreateRunningLock | 需要电源管理权限 | ⚠️ 未实现 |
| AcquireRunningLock | 需要电源管理权限 | ⚠️ 未实现 |
| ReleaseRunningLock | 需要电源管理权限 | ⚠️ 未实现 |
| SuspendDevice | 需要 CAP_SYS_ADMIN | ⚠️ 未实现 |
| WakeupDevice | 需要 CAP_SYS_ADMIN | ⚠️ 未实现 |

**注意**: 当前实现缺少任何权限检查 (见 [08_Security_Assessment.md#权限检查缺失](08_Security_Assessment.md#权限检查缺失))

### JS API
- 电池状态 API **无权限要求**
- 所有应用都可以调用

---

## 攻击面

### C API
- ❌ IPC 边界: 无权限验证 (任何进程可调用)
- ❌ 输入验证不足: 锁名称长度未严格限制
- ❌ 整数溢出: 大小计算可能溢出

### JS API
- ✓ 有限暴露: 仅电池状态,无电源控制
- ✓ 无输入: 只读操作

详细分析见 [08_Security_Assessment.md](08_Security_Assessment.md)。

---

## 使用示例

### C API 示例: 视频播放
```c
#include <ohos_types.h>
#include "interfaces/kits/running_lock.h"

void PlayVideo() {
    // 创建锁
    const RunningLock *lock = CreateRunningLock(
        "video_playback",
        RUNNINGLOCK_SCREEN,
        RUNNINGLOCK_FLAG_WAKEUP_WHEN_ACQUIRED
    );

    if (lock == NULL) {
        printf("Failed to create running lock\n");
        return;
    }

    // 获取锁
    BOOL ret = AcquireRunningLock(lock);
    if (ret == FALSE) {
        printf("Failed to acquire running lock\n");
        DestroyRunningLock(lock);
        return;
    }

    // 播放视频...
    printf("Playing video...\n");

    // 释放锁
    ReleaseRunningLock(lock);

    // 销毁锁
    DestroyRunningLock(lock);
}
```

### C API 示例: 挂起设备
```c
#include "interfaces/innerkits/power_manage.h"

void SuspendForSleep() {
    // 挂起设备 (等待运行锁释放)
    SuspendDevice(SUSPEND_DEVICE_REASON_TIMEOUT, FALSE);
}
```

### JS API 示例: 获取电池状态
```javascript
import battery from '@ohos.powermgr_lite';

battery.getStatus({
    success: function(data) {
        console.log("Charging: " + data.charging);
        console.log("Level: " + data.level + "%");
    },
    complete: function() {
        console.log("Battery status check complete");
    }
});
```

---

## 相关文档

- [00_Overview](00_Overview.md) - 核心能力概述
- [03_Architecture](03_Architecture.md) - 架构与调用链
- [08_Security_Assessment.md](08_Security_Assessment.md) - 安全风险
