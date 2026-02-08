# 内部 API

> **目的**: 模块间接口定义,依赖方向和稳定性分析
> **适用范围**: 模块开发者、架构师
> **阅读时间**: 40分钟

---

## 概述

### API 分类
1. **Framework → Service 接口**: 运行锁、挂起/唤醒
2. **Service 内部接口**: 模块间协调
3. **平台操作接口**: 内核/proc 文件系统抽象

---

## Framework → Service 接口

### PowerManage Interface

#### 定义位置
- **文件**: `frameworks/include/power_manage_intf_define.h`
- **宏**: `INHERIT_POWERMANAGE_INTERFACE`

#### 接口定义
```c
#define INHERIT_POWERMANAGE_INTERFACE                                           \
    int32_t (*AcquireRunningLockEntryFunc)(IUnknown *iUnknown, RunningLockEntry *entry, int32_t timeoutMs);     \
    int32_t (*ReleaseRunningLockEntryFunc)(IUnknown *iUnknown, RunningLockEntry *entry);                        \
    BOOL (*IsAnyRunningLockHoldingFunc)(IUnknown *iUnknown);                                                    \
    void (*SuspendDeviceFunc)(IUnknown *iUnknown, SuspendDeviceType reason, BOOL suspendImmed);                 \
    void (*WakeupDeviceFunc)(IUnknown *iUnknown, WakeupDeviceType reason, const char* details)
```

#### 方法详解

##### AcquireRunningLockEntryFunc
- **职责**: 获取运行锁
- **参数**:
  - `iUnknown`: 接口实例
  - `entry`: 运行锁条目 (包含锁、身份、状态)
  - `timeoutMs`: 超时时间 (毫秒)
- **返回值**:
  - 成功: EC_SUCCESS (0)
  - 失败: EC_FAILURE (非0)
- **实现位置**:
  - Small: `services/src/power/small/power_manage_feature_impl.c:66-81`
  - Mini: `services/src/power/mini/power_manage_feature_impl.c:`
- **调用链**: Framework → SAMGR → PowerManageFeature → RunningLockMgr

##### ReleaseRunningLockEntryFunc
- **职责**: 释放运行锁
- **参数**:
  - `iUnknown`: 接口实例
  - `entry`: 运行锁条目
- **返回值**: EC_SUCCESS / EC_FAILURE
- **实现位置**: 同上
- **调用链**: Framework → SAMGR → PowerManageFeature → RunningLockMgr

##### IsAnyRunningLockHoldingFunc
- **职责**: 检查是否有任何运行锁被持有
- **参数**:
  - `iUnknown`: 接口实例
- **返回值**: TRUE / FALSE
- **实现位置**: `services/src/power_manage_feature.c:72-75`
- **调用链**: Framework → SAMGR → PowerManageFeature → RunningLockMgr (查询计数器)

##### SuspendDeviceFunc
- **职责**: 挂起设备
- **参数**:
  - `iUnknown`: 接口实例
  - `reason`: 挂起原因 (枚举)
  - `suspendImmed`: 是否立即挂起
- **返回值**: 无
- **实现位置**: `services/src/power_manage_feature.c:82-97`
- **调用链**: Framework → SAMGR → PowerManageFeature → SuspendController
- **安全**: ⚠️ TODO: 应检查权限

##### WakeupDeviceFunc
- **职责**: 唤醒设备
- **参数**:
  - `iUnknown`: 接口实例
  - `reason`: 唤醒原因 (枚举)
  - `details`: 详情字符串 (可为 NULL)
- **返回值**: 无
- **实现位置**: `services/src/power_manage_feature.c:92-98`
- **调用链**: Framework → SAMGR → PowerManageFeature (直接调用)
- **安全**: ⚠️ TODO: 应检查权限

---

## Service 内部接口

### RunningLockMgr 接口

#### 定义位置
- **文件**: `services/include/running_lock_mgr.h`

#### 接口定义
```c
void RunningLockMgrInit(void);
int32_t RunningLockMgrAcquireEntry(RunningLockEntry *entry, int32_t timeoutMs);
int32_t RunningLockMgrReleaseEntry(RunningLockEntry *entry);
uint32_t RunningLockMgrGetLockCount(RunningLockType type);
uint32_t RunningLockMgrGetTotalLockCount();

static inline BOOL RunningLockMgrIsLockHolding(RunningLockType type) {
    return (RunningLockMgrGetLockCount(type) > 0) ? TRUE : FALSE;
}

static inline BOOL RunningLockMgrIsAnyLockHolding() {
    return (RunningLockMgrGetTotalLockCount() > 0) ? TRUE : FALSE;
}
```

#### 方法详解

##### RunningLockMgrInit
- **职责**: 初始化锁管理器
- **参数**: 无
- **返回值**: 无
- **实现位置**: `services/src/running_lock_mgr.c:20-25`
- **线程安全**: 是 (初始化时加锁)

##### RunningLockMgrAcquireEntry
- **职责**: 添加运行锁条目
- **参数**:
  - `entry`: 运行锁条目 (已初始化 identity)
  - `timeoutMs`: 超时时间
- **返回值**: EC_SUCCESS / EC_FAILURE
- **实现位置**: `services/src/running_lock_mgr.c:27-43`
- **逻辑**:
  1. 验证 entry 有效性 (`IsValidRunningLockEntry`)
  2. 加锁
  3. 添加到 `g_runningLocks[type]` 向量
  4. 递增 `g_runningLockCounts[type]`
  5. 解锁
- **线程安全**: 是 (pthread_mutex)

##### RunningLockMgrReleaseEntry
- **职责**: 移除运行锁条目
- **参数**:
  - `entry`: 运行锁条目
- **返回值**: EC_SUCCESS / EC_FAILURE
- **实现位置**: `services/src/running_lock_mgr.c:45-70`
- **逻辑**:
  1. 加锁
  2. 查找并从向量移除 entry
  3. 递减计数器
  4. 解锁
- **线程安全**: 是

##### RunningLockMgrGetLockCount
- **职责**: 查询某类型锁数量
- **参数**: `type`: 运行锁类型
- **返回值**: 锁数量 (uint32_t)
- **实现位置**: `services/src/running_lock_mgr.c:72-78`
- **线程安全**: 是

### SuspendController 接口

#### 定义位置
- **文件**: `services/include/power/suspend_controller.h`

#### 接口定义
```c
void SuspendControllerInit(void);
void EnableSuspend(void);
void DisableSuspend(void);
```

#### 方法详解

##### SuspendControllerInit
- **职责**: 初始化挂起控制器
- **参数**: 无
- **返回值**: 无
- **实现位置**: `services/src/power/suspend_controller.c:18-30`
- **行为**: 获取 `WAKEUP_HOLDER` (阻止挂起)

##### EnableSuspend
- **职责**: 使能挂起
- **参数**: 无
- **返回值**: 无
- **实现位置**: `services/src/power/suspend_controller.c:32-44`
- **行为**:
  1. 设置 `g_suspendEnabled = TRUE`
  2. 释放 `WAKEUP_HOLDER`
- **线程安全**: 是 (pthread_mutex)

##### DisableSuspend
- **职责**: 禁止挂起
- **参数**: 无
- **返回值**: 无
- **实现位置**: `services/src/power/suspend_controller.c:46-57`
- **行为**:
  1. 设置 `g_suspendEnabled = FALSE`
  2. 持有 `WAKEUP_HOLDER`
- **线程安全**: 是

### RunningLockHub 接口

#### 定义位置
- **文件**: `services/include/power/running_lock_hub.h`

#### 接口定义
```c
void RunningLockHubLock(const char* name);
void RunningLockHubUnlock(const char* name);
BOOL RunningLockHubInit(struct AutoSuspendOps* suspendOps);
```

#### 方法详解

##### RunningLockHubLock
- **职责**: 获取运行锁 (平台操作 + 阻塞计数)
- **参数**:
  - `name`: 锁名称
- **返回值**: 无
- **实现位置**: `services/src/power/running_lock_hub.c:22-34`
- **行为**:
  1. 调用 `g_runningLockOps->Acquire(name)`
  2. 调用 `g_suspendOps->IncSuspendBlockCounter()`
- **线程安全**: 否 (调用者应确保)

##### RunningLockHubUnlock
- **职责**: 释放运行锁 (平台操作 + 阻塞计数)
- **参数**:
  - `name`: 锁名称
- **返回值**: 无
- **实现位置**: `services/src/power/running_lock_hub.c:36-44`
- **行为**:
  1. 调用 `g_runningLockOps->Release(name)`
  2. 调用 `g_suspendOps->DecSuspendBlockCounter()`
- **线程安全**: 否

### AutoSuspendOps 接口

#### 定义位置
- **文件**: `services/include/power/suspend_ops.h`

#### 接口定义
```c
struct AutoSuspendOps {
    void (*Enable)(void);
    void (*IncSuspendBlockCounter)(void);
    void (*DecSuspendBlockCounter)(void);
};

typedef BOOL (*AutoSuspendLoop)(AutoSuspendWait waitFunc);
```

#### 方法详解

##### Enable
- **职责**: 使能挂起
- **参数**: 无
- **实现**: `SuspendController::EnableSuspend()`

##### IncSuspendBlockCounter
- **职责**: 递增阻塞计数器
- **参数**: 无
- **实现**: 在 `RunningLockHubLock()` 中调用
- **效果**: 阻止系统挂起

##### DecSuspendBlockCounter
- **职责**: 递减阻塞计数器
- **参数**: 无
- **实现**: 在 `RunningLockHubUnlock()` 中调用
- **效果**: 可能允许系统挂起 (如果计数器归零)

### RunningLockOps 接口

#### 定义位置
- **文件**: `services/include/power/suspend_ops.h`

#### 接口定义
```c
struct RunningLockOps {
    void (*Acquire)(const char *name);
    void (*Release)(const char *name);
};

struct RunningLockOps* RunningLockOpsInit(void);
```

#### 方法详解

##### Acquire
- **职责**: 平台锁获取
- **参数**:
  - `name`: 锁名称
- **返回值**: 无
- **实现位置**:
  - Mini: `services/src/power/mini/running_lock_handler.c:RunningLockRequest()`
  - Small: `services/src/power/small/running_lock_handler.c:RunningLockRequest()`
- **平台操作**:
  - Mini: `LOS_PmLockRequest(name, timeout)`
  - Small: `write(fd, name, strlen(name))` → `/proc/power/power_lock`

##### Release
- **职责**: 平台锁释放
- **参数**:
  - `name`: 锁名称
- **返回值**: 无
- **实现位置**: 同上
- **平台操作**:
  - Mini: `LOS_PmLockRelease(name)`
  - Small: `write(fd, name, strlen(name))` → `/proc/power/power_unlock`

---

## ScreenSaver 接口 (可选)

### ScreenSaver 接口

#### 定义位置
- **文件**: `frameworks/include/screen_saver_intf_define.h`

#### 接口定义
```c
#define INHERIT_SCREENSAVER_INTERFACE                                       \
    int32_t (*SetScreenSaverStateFunc)(IUnknown *iUnknown, BOOL enable)
```

#### 方法详解

##### SetScreenSaverStateFunc
- **职责**: 设置屏保状态
- **参数**:
  - `iUnknown`: 接口实例
  - `enable`: TRUE 启用, FALSE 禁用
- **返回值**: EC_SUCCESS / EC_FAILURE
- **实现位置**:
  - Small: `services/src/screensaver/small/screen_saver_feature_impl.c:41-67`
- **调用链**: Framework → SAMGR → ScreenSaverFeature → ScreenSaverMgr

---

## 依赖方向

### 无环依赖图

```
interfaces/kits (公共 API)
    ↓
frameworks
    ↓
services (依赖 interfaces/innerkits)
    ├─→ power_manage_service
    ├─→ power_manage_feature
    ├─→ running_lock_mgr
    ├─→ suspend_controller
    └─→ running_lock_hub
        ↓
utils
    ├─→ power_mgr_timer_util
    └─→ power_mgr_time_util
        ↓
platform (内核 / proc)
```

### 依赖矩阵

| 模块 | 依赖模块 | 依赖类型 |
|------|----------|----------|
| frameworks | interfaces/innerkits, samgr_lite, hilog_lite | 库链接 |
| power_manage_service | samgr_lite, hilog_lite | 库链接 |
| power_manage_feature | frameworks/include, running_lock_mgr, suspend_controller | 头文件包含 |
| running_lock_mgr | frameworks/include/running_lock_entry.h | 头文件包含 |
| suspend_controller | running_lock_hub | 头文件包含 |
| running_lock_hub | platform ops | 头文件包含 |
| auto_suspend | suspend_controller | 头文件包含 |
| screen_saver_feature | frameworks/include/screen_saver_intf_define.h | 头文件包含 |
| screen_saver_mgr | utils/timer_util, AMS | 库链接 |
| utils | 无依赖 | 独立 |

---

## 接口稳定性

### 稳定接口 (长期支持)

| 接口 | 稳定性 | 理由 |
|------|--------|------|
| `CreateRunningLock` | 高 | 核心功能,长期支持 |
| `DestroyRunningLock` | 高 | 核心功能,长期支持 |
| `AcquireRunningLock` | 高 | 核心功能,长期支持 |
| `ReleaseRunningLock` | 高 | 核心功能,长期支持 |
| `SuspendDevice` | 高 | 核心功能,长期支持 |
| `WakeupDevice` | 高 | 核心功能,长期支持 |
| `battery.getStatus` | 高 | 仅电池 API,稳定 |

### 不稳定接口 (可能变化)

| 接口 | 稳定性 | 理由 |
|------|--------|------|
| `SetScreenSaverState` | 中 | 可选功能,可能移除 |
| `IsRunningLockHolding` | 高 | 查询接口,稳定 |
| IPC 内部接口 | 低 | 实现细节,可能重构 |

### 可替换点

#### 平台 Ops
- **接口**: `struct RunningLockOps`
- **可替换**: 通过 `RunningLockHubInit()` 传入新实现
- **位置**: `services/src/power/running_lock_hub.c:46-55`

#### 挂起循环
- **接口**: `AutoSuspendLoop` 函数指针
- **可替换**: 通过传入不同的等待函数
- **位置**: `services/src/power/auto_suspend.c:54-93`

---

## 线程安全分析

| 接口 | 线程安全 | 同步机制 |
|------|----------|----------|
| `CreateRunningLock` | 是 | `pthread_mutex_t g_mutex` (frameworks/src/running_lock.c) |
| `DestroyRunningLock` | 是 | 同上 |
| `AcquireRunningLock` | 是 | 同上 |
| `ReleaseRunningLock` | 是 | 同上 |
| `IsRunningLockHolding` | 是 | 同上 |
| `SuspendDevice` | 是 | IPC 序列化 + SAMGR |
| `WakeupDevice` | 是 | 同上 |
| `RunningLockMgrAcquireEntry` | 是 | `pthread_mutex_t g_mutex` (services/src/running_lock_mgr.c) |
| `RunningLockMgrReleaseEntry` | 是 | 同上 |
| `EnableSuspend` | 是 | `pthread_mutex_t g_mutex` (services/src/power/suspend_controller.c) |
| `DisableSuspend` | 是 | 同上 |
| `RunningLockHubLock` | 否 | 调用者应确保同步 |
| `RunningLockHubUnlock` | 否 | 同上 |

---

## 相关文档

- [02_Directory_Structure](02_Directory_Structure.md) - 模块职责
- [03_Architecture](03_Architecture.md) - 架构与数据流
- [06_GN_Targets](06_GN_Targets.md) - 构建依赖
