# Resource Schedule Service 概览

## 目的

本文档介绍 OpenHarmony **resource_schedule_service**（资源调度服务）的定位、边界、核心能力和运行环境。

## 适用范围

- OpenHarmony 3.1+ 版本
- 标准系统（standard）
- 资源调度子系统开发者

---

## 项目定位

**resource_schedule_service** 是 OpenHarmony 资源调度子系统的核心服务，负责：

1. **事件感知与分发**：接收系统各类事件（应用状态、窗口变化、用户交互等），分发给注册的插件处理
2. **资源调度决策**：基于场景识别和策略配置，决定系统资源分配
3. **插件化管理**：支持动态加载插件扩展调度能力
4. **跨进程通信**：提供 IPC 接口供其他服务/应用上报事件和查询状态

### 子系统架构位置

```
OpenHarmony System
├── resourceschedule (资源调度子系统)
│   ├── resource_schedule_service (本仓库) ← 核心引擎
│   ├── device_usage_statistics
│   ├── device_standby
│   └── ...
├── ability (应用框架)
├── window (窗口管理)
└── ...
```

---

## 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| **事件管理** | 接收系统/应用事件，支持同步/异步上报 | `ressched/sched_controller/` |
| **场景识别** | 识别用户交互场景（点击、滑动等） | `ressched/scene_recognize/` |
| **插件框架** | 动态加载插件，事件订阅分发 | `ressched/services/resschedmgr/` |
| **Cgroup 调度** | 进程分组调度策略 | `ressched/plugins/cgroup_sched_plugin/` |
| **性能调频** | SoC 频率调节（CPU/GPU/DDR） | `ressched/plugins/socperf_plugin/` |
| **帧感知** | 流畅度优化，网络延迟控制 | `ressched/plugins/frame_aware_plugin/` |
| **设备待机** | 待机状态管理 | `ressched/plugins/device_standby_plugin/` |

---

## 运行环境

### 系统要求

- **SystemCapability**:
  - `SystemCapability.Resourceschedule.BackgroundProcessManager`
  - `SystemCapability.ResourceSchedule.SystemLoad`

- **依赖服务**:
  - `samgr` (System Ability Manager) - SA 注册与发现
  - `ability_runtime` - 应用生命周期
  - `window_manager` - 窗口状态
  - `background_task_mgr` - 后台任务
  - `audio_framework` - 音频事件
  - 其他见 `bundle.json` deps

### 进程模型

| 进程名 | SA ID | 说明 |
|--------|-------|------|
| `resource_schedule_service` | 1901 | 主调度服务，事件接收与分发 |
| `resource_schedule_executor` | 1918 | 执行器服务，具体调度操作 |

### 启动时机

- **ressched**: 系统启动后由 init 拉起 (`ressched/etc/init/resource_schedule_service.cfg`)
- **ressched_executor**: 按需启动，由 ressched 或其他服务触发

---

## 关键概念

### 1. ResType（资源类型）

资源调度服务处理的所有事件类型定义，超过 200 种：

```cpp
// 文件: ressched/interfaces/innerkits/ressched_client/include/res_type.h

// 应用生命周期
RES_TYPE_APP_STATE_CHANGE = 1
RES_TYPE_ABILITY_STATE_CHANGE = 2
RES_TYPE_PROCESS_STATE_CHANGE = 3

// 窗口事件
RES_TYPE_WINDOW_FOCUS = 6
RES_TYPE_WINDOW_VISIBILITY_CHANGE = 7

// 用户交互
RES_TYPE_CLICK_RECOGNIZE = 12
RES_TYPE_SLIDE_RECOGNIZE = 13
RES_TYPE_KEY_EVENT = 24

// 系统事件
RES_TYPE_SCREEN_STATUS = 8
RES_TYPE_THERMAL_STATE = 35
```

### 2. Plugin（插件）

插件是资源调度能力的扩展单元：

```cpp
// 插件接口定义
class Plugin {
public:
    virtual bool OnPluginInit(std::string& libName) = 0;
    virtual void OnPluginDisable() = 0;
    virtual void OnDispatchResource(const std::shared_ptr<ResData>& data) = 0;
};
```

**现有插件**:
- `libcgroup_sched_plugin.z.so` - Cgroup 调度
- `libsocperf_plugin.z.so` - 性能调频
- `libframe_aware_plugin.z.so` - 帧感知
- `libdevice_standby_plugin.z.so` - 待机管理

### 3. SystemLoadLevel（系统负载等级）

用于表示系统当前负载状态的枚举：

```typescript
// JS API 中的定义
enum SystemLoadLevel {
    LOW = 0,        // 低负载
    NORMAL = 1,     // 正常
    MEDIUM = 2,     // 中等
    HIGH = 3,       // 高负载
    OVERHEATED = 4, // 过热
    WARNING = 5,    // 警告
    EMERGENCY = 6,  // 紧急
    ESCAPE = 7      // 逃逸
}
```

### 4. ReportData 上报模型

```
┌─────────────┐    ReportData     ┌─────────────────┐
│   Client    │ ─────────────────> │  ResSchedService │
│  (App/SA)   │   (resType, value, │    (SA 1901)     │
└─────────────┘    payload)        └─────────────────┘
                                          │
                                          ▼
                                   ┌─────────────┐
                                   │  PluginMgr  │
                                   │ Dispatch to │
                                   │  Plugins    │
                                   └─────────────┘
```

---

## 对外接口概览

### JS API

| 模块 | 说明 | 主要接口 |
|------|------|----------|
| `@ohos.resourceschedule.systemload` | 系统负载查询 | `getLevel()`, `on()`, `off()` |
| `@ohos.resourceschedule.backgroundProcessManager` | 后台进程管理 | `setProcessPriority()`, `setPowerSaveMode()` |

### C++ Inner API

| 类 | 说明 | 主要方法 |
|----|------|----------|
| `ResSchedClient` | 客户端 IPC 代理 | `ReportData()`, `ReportSyncEvent()`, `GetSystemloadLevel()` |
| `ResSchedExeClient` | 执行器客户端 | `SendRequestSync()`, `KillProcess()` |

---

## 代码证据

### 核心服务注册

```cpp
// 文件: ressched/services/resschedservice/src/res_sched_service_ability.cpp:73-98
void ResSchedServiceAbility::OnStart() {
    ResSchedMgr::GetInstance().Init();
    NotifierMgr::GetInstance().Init();
    EventListenerMgr::GetInstance().Init();
    service_ = new (std::nothrow) ResSchedService();
    service_->InitAllowIpcReportRes();
    if (!Publish(service_)) {  // <-- SA 注册
        RESSCHED_LOGE("ResSchedServiceAbility::OnStart register to system ability manager failed!");
    }
    // ...
}
```

### SystemAbility ID

```json
// 文件: ressched/sa_profile/1901.json
{
    "name": 1901,
    "path": "/system/lib64/libresschedsvc.z.so"
}
```

---

## 相关链接

- [架构设计](01_Architecture.md) - 组件图与数据流
- [目录结构](02_Directory_Structure.md) - 代码组织
- [N-API 参考](03_NAPI_Reference.md) - JS API 详情
- [内部 API](04_Inner_API.md) - C++ 接口
