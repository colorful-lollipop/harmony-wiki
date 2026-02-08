# 后台任务管理 - 首页概览

## 项目定位

后台任务管理模块（background_task_mgr）是OpenHarmony**资源调度子系统**的核心组件，负责管理应用后台任务的生命周期，提供统一的任务申请、管理和调度能力。

**仓库路径**: `foundation/resourceschedule/background_task_mgr`  
**模块名称**: `@ohos/background_task_mgr`  
**子系统**: ResourceSchedule  
**版本**: 3.1  
**许可证**: Apache License 2.0

---

## 核心能力

本模块提供三类后台任务管理能力：

### 1. 短时任务（Transient Task）

**功能**: 延迟挂起机制  
**场景**: 应用退到后台后需要完成不可中断的短时间任务  
**约束**:
- 单次最大运行时长：3分钟
- 全天默认配额：10分钟（系统根据场景动态调整）
- 仅支持前台或退后台被挂起前申请

**关键代码**:
- 管理器: `services/transient_task/include/bg_transient_task_mgr.h`
- N-API: `interfaces/kits/napi/src/request_suspend_delay.cpp`

### 2. 长时任务（Continuous Task）

**功能**: 后台持续运行保障  
**场景**: 用户可感知的后台业务（如音频播放、导航、上传下载等）  
**后台模式**: 支持9种类型

| 模式 | 值 | 说明 | 权限 |
|------|-----|------|------|
| DATA_TRANSFER | 0 | 数据传输 | 普通应用 |
| AUDIO_PLAYBACK | 1 | 音频播放 | 普通应用 |
| AUDIO_RECORDING | 2 | 音频录制 | 普通应用 |
| LOCATION | 3 | 定位导航 | 普通应用 |
| BLUETOOTH_INTERACTION | 4 | 蓝牙传输 | 普通应用 |
| MULTI_DEVICE_CONNECTION | 5 | 分布式互联 | 普通应用 |
| WIFI_INTERACTION | 6 | WLAN传输 | SystemApi |
| VOIP | 7 | 音视频通话 | SystemApi |
| TASK_KEEPING | 8 | 计算任务 | 特定设备 |

**关键代码**:
- 管理器: `services/continuous_task/include/bg_continuous_task_mgr.h`
- N-API: `interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp`

### 3. 能效资源（Efficiency Resources）

**功能**: 特权资源申请  
**场景**: 应用需要申请特定系统资源特权

| 资源类型 | 值 | 特权说明 |
|----------|-----|----------|
| CPU | 1 | 申请后不被挂起 |
| COMMON_EVENT | 2 | 挂起状态下公共事件不被代理 |
| TIMER | 4 | 挂起状态下计时器不被代理 |
| WORK_SCHEDULER | 8 | 延迟任务更长执行时间 |
| BLUETOOTH | 16 | 挂起状态下蓝牙资源不被代理 |
| GPS | 32 | 挂起状态下GPS资源不被代理 |
| AUDIO | 64 | 挂起状态下音频资源不被代理 |
| RUNNING_LOCK | 128 | 运行锁 |
| SENSOR | 256 | 传感器 |

**关键代码**:
- 管理器: `services/efficiency_resources/include/bg_efficiency_resources_mgr.h`
- N-API: `interfaces/kits/napi/src/efficiency_resources_operation.cpp`

---

## 运行环境

### SystemCapability

- `SystemCapability.ResourceSchedule.BackgroundTaskManager.ContinuousTask`
- `SystemCapability.ResourceSchedule.BackgroundTaskManager.TransientTask`
- `SystemCapability.ResourceSchedule.BackgroundTaskManager.EfficiencyResourcesApply`

### 适配系统类型

- `standard`（标准系统）

### 资源需求

- ROM: 2048KB
- RAM: 10240KB

### 依赖组件

核心依赖（来自`bundle.json`）：

| 组件 | 用途 |
|------|------|
| ability_runtime | Ability生命周期管理 |
| access_token | 权限校验 |
| bundle_framework | 应用包信息获取 |
| ipc/samgr/safwk | SA框架和IPC通信 |
| distributed_notification_service | 通知服务 |
| relational_store | 数据持久化 |

---

## 关键概念

### System Ability (SA)

后台任务管理服务以System Ability形式运行：

- **SA ID**: 1903
- **进程**: `resource_schedule_service`
- **动态库**: `libbgtaskmgr_service.z.so`
- **启动方式**: `run-on-create: true`（系统启动时自动启动）

配置位置: `sa_profile/1903.json`

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (不可信)                          │
│         JS/ArkTS App → N-API → BackgroundTaskManager         │
└──────────────────────────┬──────────────────────────────────┘
                           │ IPC (Binder)
┌──────────────────────────▼──────────────────────────────────┐
│                   Service层 (系统服务)                        │
│  BackgroundTaskMgrService → 三大子管理器                      │
│  - BgTransientTaskMgr                                       │
│  - BgContinuousTaskMgr                                      │
│  - BgEfficiencyResourcesMgr                                 │
└─────────────────────────────────────────────────────────────┘
```

### 核心对象关系

```
BackgroundTaskMgrService (SA主服务)
    ├── BgTransientTaskMgr (短时任务管理)
    │   ├── TimerManager (定时器管理)
    │   ├── Watchdog (看门狗监控)
    │   └── DecisionMaker (决策引擎)
    ├── BgContinuousTaskMgr (长时任务管理)
    │   ├── ContinuousTaskRecord (任务记录)
    │   └── NotificationTools (通知工具)
    └── BgEfficiencyResourcesMgr (能效资源管理)
        ├── ResourceApplicationRecord (资源申请记录)
        └── ResourcesSubscriberMgr (订阅管理)
```

---

## 快速链接

- [目录结构与模块职责](01_Directory_Structure.md)
- [架构详细说明](02_Architecture.md)
- [N-API接口参考](03_NAPI_Reference.md)
- [安全风险评审](06_Security.md)

---

## TODO(需确认)

- [ ] 确认支持的最低API版本
- [ ] 确认各类型设备的具体配额策略
- [ ] 补充SA权限配置详情
