# 项目概览

> **目的**: 提供项目快速理解指南,帮助新开发者快速上手
> **适用范围**: 所有人,特别是初次接触项目的开发者
> **阅读时间**: 15分钟

---

## 核心能力

powermgr_lite 是 OpenHarmony 的轻量级电源管理组件,提供以下核心能力:

1. **休眠唤醒锁管理 (RunningLock)**
   - 保持屏幕常亮 (RUNNINGLOCK_SCREEN)
   - 保持 CPU 运行 (RUNNINGLOCK_BACKGROUND)
   - 距离传感器控制 (RUNNINGLOCK_PROXIMITY_SCREEN_CONTROL)
   - 锁超时与自动释放

2. **设备挂起与唤醒控制**
   - 多种挂起原因支持 (应用超时、电源按钮、合盖等)
   - 多种唤醒原因支持 (电源按钮、手势、插拔等)
   - 立即挂起与延迟挂起选项

3. **屏幕保护器 (Screen Saver, 可选)**
   - 定时激活屏保 (20秒无操作)
   - 输入事件监听与自动重置
   - Ability 集成 (通过 AMS 服务)

---

## 项目定位

### 适用系统
- **mini 系统**: 基于 LiteOS-M (嵌入式设备,内存受限)
  - 静态库编译
  - 无 IPC 机制,直接函数调用
  - 基础屏保实现

- **small 系统**: 基于 LiteOS-A (小型物联网设备)
  - 共享库编译
  - IPC 通信 (基于 Binder)
  - 完整屏保实现 (C++ + 窗口管理器)

### 资源占用
- **ROM**: 22KB (总计)
- **RAM**: ~10KB (运行时)
- **栈大小**: 2KB (主服务线程)
- **消息队列**: 20 (服务消息缓冲)

### 与其他 powermgr 组件的关系
```
powermgr_powermgr_lite (本仓库)
    └── 运行锁管理
    └── 设备挂起/唤醒

powermgr_power_manager (标准版)
    ├── 更完整的电源策略
    ├── 电池管理集成
    └── 热管理集成

powermgr_battery_lite (电池组件)
    └── 电池状态查询
    └── 充电状态上报
```

---

## 运行环境

### 依赖组件
```
powermgr_lite 依赖:
├── utils_lite (通用工具)
├── samgr_lite (系统能力管理器,IPC 基础)
├── ipc (进程间通信,仅 small 系统)
├── hilog_lite (日志,仅 small 系统共享库)
├── window_manager_lite (窗口管理器,屏保功能)
└── bounds_checking_function (安全函数,仅 small 系统)
```

### 内核依赖
- **mini 系统**: LiteOS-M 内核接口
  - `LOS_PmLockRequest()` - 运行锁请求
  - `LOS_PmLockRelease()` - 运行锁释放
  - `LOS_PmSuspend()` - 系统挂起

- **small 系统**: Linux proc 文件系统
  - `/proc/power/power_lock` - 运行锁写入
  - `/proc/power/power_unlock` - 运行锁释放
  - `/sys/power/wakeup_count` - 唤醒计数器

---

## 关键概念

### RunningLock (运行锁)
RunningLock 是防止系统进入挂起状态的机制。应用获取锁后,系统会推迟或阻止挂起。

**生命周期**:
```
创建锁 (CreateRunningLock)
    ↓
获取锁 (AcquireRunningLock) - 唤醒设备(如需)
    ↓
...应用运行...
    ↓
释放锁 (ReleaseRunningLock)
    ↓
销毁锁 (DestroyRunningLock)
```

**类型**:
| 类型 | 用途 | 示例场景 |
|------|------|----------|
| RUNNINGLOCK_SCREEN | 保持屏幕常亮 | 视频播放、阅读应用 |
| RUNNINGLOCK_BACKGROUND | 保持 CPU 运行 | 音乐播放、后台下载 |
| RUNNINGLOCK_PROXIMITY_SCREEN_CONTROL | 距离传感器控制 | 通话应用 |

**标志**:
| 标志 | 值 | 用途 |
|------|-----|------|
| RUNNINGLOCK_FLAG_WAKEUP_WHEN_ACQUIRED | 1 << 0 | 获取锁时自动唤醒设备 |

### Suspend Block Counter (挂起阻塞计数器)
系统通过阻塞计数器决定是否允许挂起:

- `计数器 == 0`: 允许挂起
- `计数器 > 0`: 阻止挂起 (有锁被持有)

**流程**:
```
[每获取一个 RunningLock] → 计数器 +1 → 阻止挂起
[每释放一个 RunningLock] → 计数器 -1 → 可能允许挂起
```

### WakeupHolder (唤醒持有者)
系统级内核锁,用于强制阻止或允许挂起:

```
SuspendController持有WakeupHolder → 系统无法挂起
SuspendController释放WakeupHolder → 系统可以挂起
```

### SAMGR Lite (System Ability Manager Lite)
轻量级服务注册与消息路由框架:

```
应用 ──→ Framework ──→ SAMGR ──→ PowerManageFeature
                     (代理)    (消息路由)     (服务端实现)
```

---

## 架构分层

```
┌─────────────────────────────────────────────────────────────────┐
│                    应用层 / JS 应用层                       │
│         (使用 running_lock.h API / battery JS API)          │
└────────────────────┬────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────┐
│                  Frameworks 层                              │
│   ┌─────────────────┐     ┌──────────────────┐         │
│   │ running_lock.c   │     │ power_manage.c   │         │
│   │ (锁生命周期)     │     │ (IPC 代理)      │         │
│   └────────┬────────┘     └────────┬──────────┘         │
│            │                       │                      │
└────────────┼───────────────────────┼──────────────────────┘
             │                       │
┌────────────▼───────────────────────▼──────────────────────┐
│              SAMGR Lite (IPC 消息路由)                     │
└────────────┬───────────────────────────────────────────────┘
             │
┌────────────▼───────────────────────────────────────────────┐
│                 Services 层                                │
│  ┌──────────────────────┐   ┌──────────────────┐      │
│  │ PowerManageFeature   │   │ScreenSaverFeature│      │
│  │ - 运行锁管理         │   │ - 屏保管理        │      │
│  │ - 挂起/唤醒控制      │   └──────────────────┘      │
│  └────────┬──────────────┘                              │
│           │                                            │
│  ┌────────▼──────────────┐                              │
│  │ RunningLockMgr       │ - 线程安全锁管理              │
│  │ - 锁向量管理       │                              │
│  │ - 计数器维护       │                              │
│  └────────┬──────────────┘                              │
│           │                                            │
│  ┌────────▼──────────────────────┐                      │
│  │ SuspendController         │ - 挂起控制              │
│  │ - WakeupHolder 管理      │                      │
│  └────────┬──────────────────────┘                      │
│           │                                            │
│  ┌────────▼──────────────┐                              │
│  │ RunningLockHub     │ - 平台操作桥接             │
│  │ - 调用平台 Ops     │                              │
│  └──────────────────────┘                              │
│                                                      │
│  ┌──────────────────────┐                              │
│  │ Platform Ops (平台特定)                             │
│  │ - mini: LOS_PmLock*()                              │
│  │ - small: /proc/power/* 文件操作                   │
│  └──────────────────────┘                              │
└───────────────────────────────────────────────────────────────┘
```

---

## 数据流示例

### 场景 1: 应用获取运行锁
```
1. 应用调用: AcquireRunningLock(lock)
   ↓
2. Framework 检查锁存在性 (g_runningLocks)
   ↓
3. [小系统] 通过 SAMGR 发送 IPC 消息
   [小系统] 或直接调用
   ↓
4. PowerManageFeature: OnAcquireRunningLock()
   ↓
5. RunningLockMgr: AcquireEntry(entry)
   - 验证 entry 有效性
   - 添加到 g_runningLocks[type] 向量
   ↓
6. RunningLockHub: Lock(name)
   - g_runningLockOps->Acquire(name)
   ↓
7. SuspendController: IncSuspendBlockCounter()
   - 阻塞计数器 +1
   ↓
8. 平台 Ops: 实际锁获取
   - [mini] LOS_PmLockRequest()
   - [small] write("/proc/power/power_lock", name)
```

### 场景 2: 设备挂起
```
1. 应用调用: SuspendDevice(reason, suspendImmed)
   ↓
2. Framework 调用服务接口
   ↓
3. PowerManageFeature: OnSuspendDevice(reason, suspendImmed)
   ↓
4. [TODO: 应检查权限]
   ↓
5. SuspendController: DisableSuspend()
   - 释放 WakeupHolder
   ↓
6. AutoSuspend 线程检查条件:
   - if (g_suspendBlockCounter == 0 && g_suspendEnabled)
   - if (g_waitingSuspendCondition == TRUE)
   → 触发挂起
```

---

## 相关文档

- [01_Project_Boundaries](01_Project_Boundaries.md) - 项目边界与适用场景详解
- [02_Directory_Structure](02_Directory_Structure.md) - 代码结构导航
- [03_Architecture](03_Architecture.md) - 完整架构说明
- [04_NAPI_JS_API](04_NAPI_JS_API.md#c-api) - 运行锁 API 参考
