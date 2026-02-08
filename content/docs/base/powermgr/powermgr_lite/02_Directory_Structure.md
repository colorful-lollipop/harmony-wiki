# 目录结构与模块职责

> **目的**: 快速定位代码位置,理解各模块职责
> **适用范围**: 所有开发者,特别是需要定位代码的人
> **阅读时间**: 15分钟

---

## 目录树概览

```
base/powermgr/powermgr_lite
├── figures/              # 架构图
│   └── en-us_image_0000001079710638.png
├── frameworks/           # Framework 层 - 客户端实现
│   ├── include/
│   │   ├── mini/        # Mini 系统特定头文件
│   │   │   ├── power_manage_interface.h
│   │   │   └── screen_saver_interface.h
│   │   ├── small/       # Small 系统特定头文件
│   │   │   ├── power_manage_interface.h
│   │   │   ├── screen_saver_interface.h
│   │   │   └── screen_saver_handler.h
│   │   ├── running_lock_framework.h      # 运行锁 Framework 接口
│   │   ├── screen_saver_intf_define.h    # 屏保接口定义
│   │   ├── running_lock_entry.h         # 运行锁条目定义
│   │   ├── power_mgr.h                # 电源管理接口
│   │   └── hilog_wrapper.h            # 日志封装
│   └── src/
│       ├── mini/                    # Mini 系统实现
│       │   ├── power_manage.c          # 直接调用实现
│       │   └── power_screen_saver.c   # 基础屏保实现
│       ├── small/                   # Small 系统实现
│       │   ├── power_manage.c          # IPC 代理实现
│       │   └── power_screen_saver.c   # 完整屏保实现
│       └── running_lock.c           # 运行锁通用实现
├── interfaces/          # 接口层 - API 定义
│   ├── innerkits/       # 内部接口 (模块间通信)
│   │   ├── running_lock_inner.h      # 内部运行锁接口
│   │   ├── power_manage.h            # 挂起/唤醒 API
│   │   └── power_screen_saver.h     # 屏保 API
│   └── kits/            # 外部接口 (应用开发者使用)
│       ├── running_lock.h            # 公共 C API
│       └── battery/js/builtin/       # JavaScript JSI 绑定
│           ├── BUILD.gn
│           ├── CMakeLists.txt
│           ├── include/
│           │   ├── battery_module.h
│           │   └── battery_impl.h
│           └── src/
│               └── battery_module.cpp     # JS 模块实现
├── services/           # 服务层 - 系统服务实现
│   ├── include/
│   │   ├── power/                # 电源管理服务接口
│   │   │   ├── suspend_controller.h      # 挂起控制器
│   │   │   ├── running_lock_hub.h       # 运行锁中心
│   │   │   └── suspend_ops.h           # 平台操作抽象
│   │   ├── small/               # Small 系统屏保
│   │   │   └── screen_saver_handler.h    # 屏保处理器
│   │   ├── screen_saver_mgr.h            # 屏保管理器 (C++)
│   │   ├── screen_saver_feature.h        # 屏保 Feature
│   │   ├── power_manage_feature.h        # 电源管理 Feature
│   │   └── running_lock_mgr.h          # 运行锁管理器
│   └── src/
│       ├── power/                # 电源管理核心实现
│       │   ├── suspend_controller.c        # 挂起控制器实现
│       │   ├── running_lock_hub.c         # 运行锁中心实现
│       │   ├── auto_suspend.c            # 自动挂起线程
│       │   ├── mini/                   # Mini 平台实现
│       │   │   ├── power_manage_feature_impl.c
│       │   │   ├── running_lock_handler.c    # LiteOS-M 内核接口
│       │   │   └── auto_suspend_loop.c       # 挂起循环
│       │   └── small/                  # Small 平台实现
│       │       ├── power_manage_feature_impl.c  # IPC Stub 实现
│       │       ├── running_lock_handler.c       # /proc 文件操作
│       │       └── auto_suspend_loop.c       # 挂起循环 (Linux)
│       ├── screensaver/          # 屏保服务 (仅 small)
│       │   └── small/
│       │       ├── screen_saver_feature_impl.c  # IPC Stub
│       │       ├── screen_saver_handler.cpp     # C++ 计时器处理
│       │       ├── screen_saver_mgr.cpp         # C++ 管理器
│       │       └── screen_saver_feature.c        # Feature 注册
│       ├── power_manage_feature.c    # 电源 Feature 通用逻辑
│       ├── power_manage_service.c    # 主服务注册
│       ├── running_lock_mgr.c       # 运行锁管理器
│       └── screen_saver_feature.c  # 屏保 Feature 注册
├── utils/               # 工具层 - 通用工具函数
│   ├── include/
│   │   ├── power_mgr_timer_util.h   # POSIX 计时器封装
│   │   └── power_mgr_time_util.h    # 时间转换工具
│   └── src/
│       ├── power_mgr_timer_util.c   # 计时器实现
│       └── power_mgr_time_util.c    # 时间转换实现
├── wiki/               # 文档目录 (本文档所在)
├── _work/              # 工作目录 (生成的笔记和计划)
├── README.md           # 英文项目说明
├── README_zh.md        # 中文项目说明
├── bundle.json         # 组件元数据
├── BUILD.gn            # 根构建文件
├── config.gni          # 构建配置
└── powermgr.gni        # 全局路径和系统类型定义
```

---

## 模块职责详解

### Frameworks 层 (`frameworks/`)

**职责**: 提供**客户端接口**,屏蔽底层服务差异,为应用开发者提供统一的 API。

#### running_lock.c
- **职责**: 运行锁的客户端生命周期管理
- **关键功能**:
  - 运行锁列表管理 (`g_runningLocks`)
  - 锁创建/销毁验证
  - 锁获取/释放的 IPC 调用或直接调用
  - 线程安全 (使用 `pthread_mutex_t`)
- **依赖**:
  - `interfaces/innerkits/running_lock_inner.h`
  - `frameworks/include/running_lock_entry.h`
  - samgr_lite (small 系统)
- **关键函数**:
  - `CreateRunningLock()` - 创建锁对象
  - `DestroyRunningLock()` - 销毁锁对象
  - `AcquireRunningLock()` - 获取锁
  - `ReleaseRunningLock()` - 释放锁
- **文件**: `frameworks/src/running_lock.c`

#### power_manage.c (mini/small)
- **职责**: 电源管理的客户端实现,平台相关
- **关键功能**:
  - **mini**: 直接调用服务接口,无 IPC
  - **small**: IPC 代理,通过 SAMGR Lite 调用服务
  - 接口单例管理
  - 身份初始化 (PID, token)
- **依赖**:
  - `frameworks/include/${system_type}/power_manage_interface.h`
  - `samgr_lite`
- **关键函数**:
  - `InitIdentity()` - 初始化调用者身份
  - `OnSuspendDevice()` - 挂起请求
  - `OnWakeupDevice()` - 唤醒请求
- **文件**:
  - `frameworks/src/mini/power_manage.c`
  - `frameworks/src/small/power_manage.c`

#### power_screen_saver.c (mini/small)
- **职责**: 屏保客户端实现
- **关键功能**:
  - **mini**: 基础屏保功能
  - **small**: 完整屏保功能,输入事件监听
- **依赖**:
  - `frameworks/include/screen_saver_intf_define.h`
- **文件**:
  - `frameworks/src/mini/power_screen_saver.c`
  - `frameworks/src/small/power_screen_saver.c`

---

### Services 层 (`services/`)

**职责**: 实现**系统服务**,处理来自客户端的请求,协调底层硬件操作。

#### power_manage_service.c
- **职责**: 主服务注册与 SAMGR Lite 集成
- **关键功能**:
  - 注册 `powermgr` 服务到 SAMGR
  - 服务初始化
  - 消息处理框架
- **配置**:
  - 优先级: LEVEL_HIGH
  - 栈大小: 2KB
  - 消息队列: 20
- **文件**: `services/src/power_manage_service.c`

#### power_manage_feature.c
- **职责**: 电源管理 Feature 实现核心逻辑
- **关键功能**:
  - 注册 `powermanage` Feature
  - 处理运行锁获取/释放请求
  - 处理挂起/唤醒请求
  - ⚠️ 权限检查 TODO (未实现)
- **依赖**:
  - `services/include/power_manage_feature.h`
  - `services/src/running_lock_mgr.c`
  - `services/src/power/suspend_controller.c`
- **关键函数**:
  - `GetPowerManageFeatureImpl()` - 获取 Feature 实例
  - `OnAcquireRunningLock()` - 处理锁获取
  - `OnReleaseRunningLock()` - 处理锁释放
  - `OnSuspendDevice()` - 处理挂起请求
  - `OnWakeupDevice()` - 处理唤醒请求
- **文件**: `services/src/power_manage_feature.c`

#### running_lock_mgr.c
- **职责**: 线程安全的运行锁管理器
- **关键功能**:
  - 按类型维护锁向量 (SCREEN, BACKGROUND, PROXIMITY)
  - 锁条目添加/移除
  - 锁计数查询
  - 线程安全 (使用 `pthread_mutex_t`)
- **数据结构**:
  ```c
  static RunningLockEntry* g_runningLocks[RUNNINGLOCK_BUTT];
  static pthread_mutex_t g_mutex;
  ```
- **关键函数**:
  - `RunningLockMgrInit()` - 初始化管理器
  - `RunningLockMgrAcquireEntry()` - 添加锁条目
  - `RunningLockMgrReleaseEntry()` - 移除锁条目
  - `RunningLockMgrGetLockCount()` - 查询某类型锁数量
- **文件**: `services/src/running_lock_mgr.c`

#### suspend_controller.c
- **职责**: 挂起状态控制器
- **关键功能**:
  - 持有/释放 `WAKEUP_HOLDER`
  - 挂起使能控制
  - 协调 RunningLockHub 操作
- **机制**:
  - `WAKEUP_HOLDER` 是系统级内核锁
  - 持有时阻止挂起
  - 释放时允许挂起
- **关键函数**:
  - `SuspendControllerInit()` - 初始化
  - `EnableSuspend()` - 使能挂起
  - `DisableSuspend()` - 禁止挂起
- **文件**: `services/src/power/suspend_controller.c`

#### running_lock_hub.c
- **职责**: 运行锁与挂起操作的桥梁
- **关键功能**:
  - 调用平台运行锁操作
  - 增/减挂起阻塞计数器
  - 初始化平台 Ops
- **关键函数**:
  - `RunningLockHubLock()` - 获取锁时调用
  - `RunningLockHubUnlock()` - 释放锁时调用
  - `RunningLockHubInit()` - 初始化 Hub
- **文件**: `services/src/power/running_lock_hub.c`

#### auto_suspend.c
- **职责**: 自动挂起线程管理
- **关键功能**:
  - 创建后台挂起线程
  - 周期性检查挂起条件
  - 处理阻塞计数器变化
- **线程配置**:
  - 检查间隔: 500ms
  - 线程类型: 分离线程
- **同步**:
  - `pthread_mutex_t g_mutex`
  - `pthread_cond_t g_cond`
- **文件**: `services/src/power/auto_suspend.c`

#### 平台特定实现

##### mini 系统 (`services/src/power/mini/`)

**power_manage_feature_impl.c**
- **职责**: mini 系统电源管理 Feature 实现
- **关键功能**:
  - 直接函数调用 (无 IPC)
  - 调用 `RunningLockMgr` 和 `SuspendController`
- **文件**: `services/src/power/mini/power_manage_feature_impl.c`

**running_lock_handler.c**
- **职责**: LiteOS-M 内核接口封装
- **关键功能**:
  - `LOS_PmLockRequest()` - 内核锁请求
  - `LOS_PmLockRelease()` - 内核锁释放
- **文件**: `services/src/power/mini/running_lock_handler.c`

**auto_suspend_loop.c**
- **职责**: LiteOS-M 挂起循环
- **关键功能**:
  - `LOS_PmSuspend()` - 内核挂起调用
- **文件**: `services/src/power/mini/auto_suspend_loop.c`

##### small 系统 (`services/src/power/small/`)

**power_manage_feature_impl.c**
- **职责**: small 系统电源管理 Feature IPC Stub
- **关键功能**:
  - `FeatureInvoke()` - IPC 函数分发
  - `AcquireInvoke()` / `ReleaseInvoke()` - 锁操作
  - `SuspendInvoke()` / `WakeupInvoke()` - 挂起/唤醒
- **文件**: `services/src/power/small/power_manage_feature_impl.c`

**running_lock_handler.c**
- **职责**: Linux proc 文件系统接口
- **关键功能**:
  - `/proc/power/power_lock` - 写入锁请求
  - `/proc/power/power_unlock` - 写入锁释放
- **文件**: `services/src/power/small/running_lock_handler.c`

**auto_suspend_loop.c**
- **职责**: Linux 挂起循环
- **关键功能**:
  - `sleep()` 模拟挂起
- **文件**: `services/src/power/small/auto_suspend_loop.c`

#### 屏保服务 (`services/src/screensaver/`)

**screen_saver_feature.c**
- **职责**: 屏保 Feature 注册
- **关键功能**:
  - 注册 `screensaver` Feature
- **文件**: `services/src/screensaver/small/screen_saver_feature.c`

**screen_saver_mgr.cpp**
- **职责**: 屏保 C++ 管理器
- **关键功能**:
  - 计时器管理
  - Ability 启动/停止
- **文件**: `services/src/screensaver/small/screen_saver_mgr.cpp`

**screen_saver_handler.cpp**
- **职责**: 屏保 C++ 处理器
- **关键功能**:
  - POSIX 计时器使用
  - 输入事件监听
  - AMS (Ability Manager Service) 集成
- **文件**: `services/src/screensaver/small/screen_saver_handler.cpp`

**screen_saver_feature_impl.c**
- **职责**: 屏保 IPC Stub
- **关键功能**:
  - `SetStateInvoke()` - 处理屏保状态设置
- **文件**: `services/src/screensaver/small/screen_saver_feature_impl.c`

---

### Interfaces 层 (`interfaces/`)

**职责**: 定义**API 接口**,模块间通信协议。

#### kits/running_lock.h
- **职责**: 公共 C API (应用开发者使用)
- **关键定义**:
  - `RunningLock` 结构体
  - `RunningLockType` 枚举
  - `RunningLockFlag` 枚举
  - `RUNNING_LOCK_NAME_LEN` 常量
- **导出函数**:
  - `CreateRunningLock()`
  - `DestroyRunningLock()`
  - `AcquireRunningLock()`
  - `ReleaseRunningLock()`
  - `IsRunningLockHolding()`
- **文件**: `interfaces/kits/running_lock.h`

#### innerkits/power_manage.h
- **职责**: 内部挂起/唤醒 API
- **关键定义**:
  - `SuspendDeviceType` 枚举 (9 种原因)
  - `WakeupDeviceType` 枚举 (9 种原因)
- **导出函数**:
  - `SuspendDevice(reason, suspendImmed)`
  - `WakeupDevice(reason, details)`
- **文件**: `interfaces/innerkits/power_manage.h`

#### innerkits/running_lock_inner.h
- **职责**: 内部运行锁接口
- **导出函数**:
  - `IsAnyRunningLockHolding()`
- **文件**: `interfaces/innerkits/running_lock_inner.h`

#### innerkits/power_screen_saver.h
- **职责**: 屏保 API
- **导出函数**:
  - `SetScreenSaverState(enable)`
- **文件**: `interfaces/innerkits/power_screen_saver.h`

#### kits/battery/js/builtin/
- **职责**: JavaScript JSI 绑定
- **文件**:
  - `include/battery_module.h` - JS 模块定义
  - `include/battery_impl.h` - 电池实现接口
  - `src/battery_module.cpp` - JS 模块实现
- **导出 JS API**:
  - `battery.getStatus()`
- **文件**: `interfaces/kits/battery/js/builtin/`

---

### Utils 层 (`utils/`)

**职责**: 提供**通用工具函数**,供其他模块复用。

#### power_mgr_timer_util.c/h
- **职责**: POSIX 计时器封装
- **关键功能**:
  - `PowerMgrTimerCreate()` - 创建计时器
  - `PowerMgrTimerStart()` - 启动计时器
  - `PowerMgrTimerStop()` - 停止计时器
  - `PowerMgrTimerDestroy()` - 销毁计时器
- **文件**:
  - `utils/include/power_mgr_timer_util.h`
  - `utils/src/power_mgr_timer_util.c`

#### power_mgr_time_util.c/h
- **职责**: 时间转换工具
- **关键函数**:
  - `MsToTimeSpec()` - 毫秒转 timespec
  - `TimeSpecToMs()` - timespec 转毫秒
- **文件**:
  - `utils/include/power_mgr_time_util.h`
  - `utils/src/power_mgr_time_util.c`

---

## 依赖方向

```
interfaces/kits (公共 API)
    ↓
frameworks (客户端)
    ↓ (IPC 或直接调用)
samgr_lite (IPC 框架)
    ↓ (IPC 路由)
services (服务端)
    ↓ (平台调用)
内核 / proc 文件系统
```

**依赖关系**:
1. **interfaces/kits** ← 无依赖 (最上层)
2. **frameworks** → interfaces/innerkits
3. **frameworks** → samgr_lite (small 系统)
4. **services** → frameworks (使用内部接口)
5. **services** → utils (工具函数)
6. **services** → 内核 / proc (平台操作)

---

## 快速导航

### 我想...

| 需求 | 查看位置 |
|------|---------|
| 了解如何使用 RunningLock API | [04_NAPI_JS_API.md#c-api](04_NAPI_JS_API.md#c-api) |
| 修改运行锁逻辑 | `services/src/running_lock_mgr.c` |
| 添加新的锁类型 | `interfaces/kits/running_lock.h` (枚举) + `services/src/running_lock_mgr.c` (向量) |
| 修改挂起逻辑 | `services/src/power/suspend_controller.c` |
| 添加新的挂起原因 | `interfaces/innerkits/power_manage.h` (枚举) + `services/src/power_manage_feature.c` (处理) |
| 理解 IPC 流程 | [03_Architecture.md#ipc-通信](03_Architecture.md#ipc-通信) |
| 修改 JS API | `interfaces/kits/battery/js/builtin/src/battery_module.cpp` |
| 添加新的 JS API | `interfaces/kits/battery/js/builtin/` (参考 battery_module 模式) |
| 修改构建配置 | `BUILD.gn`, `config.gni`, `powermgr.gni` |

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 完整架构说明
- [05_Internal_API](05_Internal_API.md) - 内部接口详解
- [06_GN_Targets](06_GN_Targets.md) - 构建配置
