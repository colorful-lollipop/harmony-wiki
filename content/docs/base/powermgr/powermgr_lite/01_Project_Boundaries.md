# 项目定位与边界

> **目的**: 明确项目适用场景、边界和限制
> **适用范围**: 项目决策者、架构师、集成开发者
> **阅读时间**: 20分钟

---

## 项目定位

powermgr_lite 是 OpenHarmony 的**轻量级电源管理子系统**,面向以下场景:

### 目标设备类型

| 设备类型 | 典型硬件 | 资源限制 | powermgr_lite 适用性 |
|----------|----------|----------|-------------------|
| 智能手表 | 512KB RAM, 4MB Flash | 极高 | ✓ 完全适用 |
| 物联网传感器 | 1MB RAM, 8MB Flash | 高 | ✓ 完全适用 |
| 智能音箱 | 4MB RAM, 32MB Flash | 中 | ✓ 完全适用 |
| 工业控制器 | 8MB RAM, 64MB Flash | 中 | ✓ 完全适用 |
| 平板电脑 | 2GB+ RAM | 低 | ⚠ 建议使用 power_manager (标准版) |
| 智能手机 | 4GB+ RAM | 极低 | ✗ 不适用,使用 power_manager |

### 核心价值主张
1. **资源高效**: ROM 22KB, RAM 10KB
2. **功能专注**: 仅提供核心运行锁和挂起/唤醒能力
3. **双系统支持**: 同时支持 mini (LiteOS-M) 和 small (LiteOS-A)
4. **可裁剪**: 屏保功能可选 (`enable_screensaver` flag)

---

## 边界定义

### 功能边界 (在项目内)
- ✓ RunningLock 生命周期管理
- ✓ 设备挂起控制
- ✓ 设备唤醒控制
- ✓ 屏幕保护器 (可选)
- ✓ 运行锁计数与阻塞机制

### 功能边界 (不在项目内)
- ✗ 电池管理 (参考: powermgr_battery_lite)
- ✗ 充电控制 (参考: powermgr_battery_manager)
- ✗ 热管理 (参考: powermgr_thermal_manager)
- ✗ 亮灭屏控制 (参考: powermgr_display_manager)
- ✗ 复杂电源策略 (参考: powermgr_power_manager)
- ✗ 应用级电源优化 (参考: 应用框架层)

### 系统边界

#### mini 系统 (LiteOS-M)
**运行环境**:
- 无用户空间进程隔离
- 无虚拟内存
- 无 Linux 内核接口
- 任务调度器: LiteOS 调度器

**能力限制**:
- 无 IPC 机制 (通过 samgr_lite 直接调用)
- 无进程权限检查
- 无文件系统操作
- 无动态加载

#### small 系统 (LiteOS-A)
**运行环境**:
- 基于 Linux 内核
- 支持多进程
- 支持虚拟内存
- 标准 POSIX 接口

**能力扩展**:
- ✓ IPC 通信 (基于 Binder)
- ✓ 文件系统操作 (/proc, /sys)
- ✓ 动态库加载
- ⚠️ 权限检查 (TODO 未实现)

---

## 核心能力详解

### 1. RunningLock 管理

#### 能力范围
- 创建/销毁运行锁对象
- 获取/释放运行锁
- 查询锁状态
- 多种锁类型支持

#### 应用场景
| 场景 | 锁类型 | 超时 | 唤醒 |
|------|-------|------|------|
| 视频播放 | RUNNINGLOCK_SCREEN | 无 | 可选 |
| 音乐播放 | RUNNINGLOCK_BACKGROUND | 无 | 无 |
| 后台下载 | RUNNINGLOCK_BACKGROUND | 5分钟 | 无 |
| 通话中 | RUNNINGLOCK_PROXIMITY_SCREEN_CONTROL | 无 | 无 |
| 语音助手 | RUNNINGLOCK_SCREEN | 30秒 | 是 |

#### API 能力
```c
// 创建锁
const RunningLock *lock = CreateRunningLock(
    "video_playback",              // 名称 (64 字符限制)
    RUNNINGLOCK_SCREEN,              // 类型
    RUNNINGLOCK_FLAG_WAKEUP_WHEN_ACQUIRED  // 标志
);

// 获取锁
BOOL ret = AcquireRunningLock(lock);
// ret == TRUE: 成功获取
// ret == FALSE: 失败 (超时或系统错误)

// 释放锁
ReleaseRunningLock(lock);

// 销毁锁
DestroyRunningLock(lock);
```

### 2. 设备挂起控制

#### 能力范围
- 多种挂起原因支持 (9 种)
- 立即挂起或延迟挂起
- 与运行锁协调 (有锁时阻止挂起)

#### 挂起原因
| 原因代码 | 枚举值 | 典型场景 |
|---------|-------|----------|
| SUSPEND_DEVICE_REASON_APPLICATION | 0 | 应用主动请求挂起 |
| SUSPEND_DEVICE_REASON_TIMEOUT | 2 | 用户无操作超时 |
| SUSPEND_DEVICE_REASON_POWER_BUTTON | 4 | 电源按钮按下 |
| SUSPEND_DEVICE_REASON_LID_SWITCH | 3 | 笔记本合盖 |
| SUSPEND_DEVICE_REASON_FORCE_SUSPEND | 8 | 强制挂起 (系统级) |

#### 挂起协调机制
```
[有运行锁被持有]
    ↓
SuspendController 持有 WakeupHolder
    ↓
系统无法进入挂起状态

[所有运行锁释放]
    ↓
SuspendController 释放 WakeupHolder
    ↓
AutoSuspend 线程检查: g_suspendBlockCounter == 0
    ↓
系统可以进入挂起状态
```

### 3. 设备唤醒控制

#### 能力范围
- 多种唤醒原因支持 (9 种)
- 唤醒详情字符串传递
- 与屏保协调

#### 唤醒原因
| 原因代码 | 枚举值 | 典型场景 |
|---------|-------|----------|
| WAKEUP_DEVICE_POWER_BUTTON | 1 | 电源按钮按下 |
| WAKEUP_DEVICE_GESTURE | 4 | 手势识别 |
| WAKEUP_DEVICE_PLUGGED_IN | 3 | 充电器插入 |
| WAKEUP_DEVICE_APPLICATION | 2 | 应用主动唤醒 |
| WAKEUP_DEVICE_WAKE_KEY | 6 | 唤醒键按下 |

#### 安全注意
⚠️ **当前实现未进行权限检查** (见 [08_Security_Assessment.md#权限检查缺失](08_Security_Assessment.md#权限检查缺失))
- 任何能访问 IPC 接口的进程都可以唤醒设备
- 需要通过系统权限机制 (如 DAC) 限制访问

### 4. 屏幕保护器 (可选)

#### 能力范围 (仅 small 系统)
- 定时激活屏保 (20 秒无操作)
- 输入事件监听与自动重置
- 通过 AMS 启动屏保 Ability
- 计时器管理

#### 应用场景
- 智能屏/音箱待机屏保
- 信息展示屏保 (时钟、天气)
- 低功耗模式演示

#### 限制
- 仅在 `enable_screensaver = true` 时编译
- 依赖 window_manager_lite 和 AMS 服务
- 不适用于 mini 系统

---

## 运行环境

### mini 系统 (LiteOS-M)

#### 特性
- **单地址空间**: 应用与内核共享地址空间
- **无进程隔离**: 没有独立的用户空间
- **直接调用**: Framework 直接调用 Service,无 IPC

#### 平台接口
| 操作 | LiteOS-M 接口 | 文件位置 |
|------|--------------|---------|
| 锁请求 | `LOS_PmLockRequest(name, timeout)` | `services/src/power/mini/running_lock_handler.c` |
| 锁释放 | `LOS_PmLockRelease(name)` | 同上 |
| 系统挂起 | `LOS_PmSuspend(timeout)` | `services/src/power/mini/auto_suspend_loop.c` |

#### 依赖
```
mini 系统依赖:
├── utils_lite (静态链接)
├── samgr_lite (静态链接)
├── hilog_lite (静态链接, 内核日志)
└── LiteOS-M 内核 (LOS_Pm* 接口)
```

### small 系统 (LiteOS-A)

#### 特性
- **多进程**: 应用与系统服务独立进程
- **IPC 通信**: 基于 Binder 的 SAMGR Lite
- **用户空间**: 标准用户态进程

#### 平台接口
| 操作 | Linux 接口 | 文件位置 |
|------|-----------|---------|
| 锁请求 | `write(fd, name, len)` → `/proc/power/power_lock` | `services/src/power/small/running_lock_handler.c:58` |
| 锁释放 | `write(fd, name, len)` → `/proc/power/power_unlock` | 同上:86 |
| 挂起计数 | `/sys/power/wakeup_count` | 读取唤醒次数 |

#### 依赖
```
small 系统依赖:
├── utils_lite (共享库)
├── samgr_lite (共享库)
├── ipc (共享库, Binder)
├── hilog_lite (共享库)
├── window_manager_lite (共享库, 屏保)
└── bounds_checking_function (共享库, 安全函数)
```

---

## Feature Flags

### enable_screensaver
- **默认值**: `false`
- **位置**: `config.gni:16`
- **效果**: 编译时决定是否包含屏保功能
- **影响**:
  - TRUE: 编译 `screen_saver_feature.c` 和相关 C++ 代码
  - FALSE: 屏保功能完全排除
- **适用系统**: 仅 small 系统

### is_liteos_m / is_liteos_a
- **自动检测**: 基于 `ohos_kernel_type` 变量
- **位置**: `powermgr.gni:32-45`
- **效果**: 决定编译目标和系统类型
- **映射**:
  - `ohos_kernel_type == "liteos_m"` → `is_liteos_m = true`, `system_type = "mini"`
  - 其他 → `is_liteos_a = true`, `system_type = "small"`

---

## 限制与已知问题

### 功能限制
1. **无电源策略**: 不支持复杂电源策略 (如应用白名单、策略配置)
2. **无电池管理**: 不提供电池状态、充电状态等 (参考 battery_lite)
3. **权限检查缺失**: 挂起/唤醒操作未验证调用者权限 (安全风险)
4. **单服务**: 仅单个 `powermgr` 服务,无备份或故障转移

### 已知问题
1. **IPC 数据验证不足**: Small 系统 IPC 数据缺乏严格的边界验证 (见 [08_Security_Assessment.md#ipc-输入验证不足](08_Security_Assessment.md#ipc-输入验证不足))
2. **锁名称长度未限制**: `CreateRunningLock` 未在 `strcpy_s` 前验证名称长度
3. **屏保功能仅 Linux**: Mini 系统无屏保支持

---

## 升级路径

### 从 powermgr_lite 迁移到 power_manager (标准版)
如果项目需求超出 powermgr_lite 的能力边界,建议迁移到标准版:

| 能力需求 | 推荐方案 |
|----------|----------|
| 需要电池管理 | 集成 powermgr_battery_lite |
| 需要复杂电源策略 | 迁移到 powermgr_power_manager |
| 需要热管理 | 集成 powermgr_thermal_manager |
| 需要多设备支持 | 使用 power_manager (支持多电源域) |

---

## 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Directory_Structure](02_Directory_Structure.md) - 代码结构导航
- [06_GN_Targets.md#feature-flags](06_GN_Targets.md#feature-flags) - 编译选项详解
- [08_Security_Assessment.md](08_Security_Assessment.md) - 安全风险详情
