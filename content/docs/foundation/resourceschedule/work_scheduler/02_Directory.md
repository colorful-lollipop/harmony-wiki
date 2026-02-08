# 目录结构与模块职责

> **目的**: 详细说明 Work Scheduler 模块的目录组织和各层职责
> **适用范围**: 代码开发者、模块维护者、架构师

---

## 顶层目录结构

```
work_scheduler/
├── BUILD.gn                    # 根构建配置
├── bundle.json                  # 组件元数据（依赖、Syscap、ROM/RAM）
├── workscheduler.gni           # 全局配置和 Feature Flags
├── hisysevent.yaml            # HiSysEvent 事件定义
├── CODEOWNERS                 # 代码所有者
├── LICENSE                    # Apache 2.0
├── README.md / README_ZH.md  # 中英文 README
├── figures/                    # 文档图片资源
│
├── frameworks/                 # 框架层（客户端 SDK）
│   ├── BUILD.gn              # 框架层构建配置
│   ├── IWorkSchedService.idl  # IPC 接口定义（客户端调用服务）
│   ├── include/               # 公共头文件
│   │   ├── work_condition.h          # 工作条件类型定义
│   │   ├── work_info.h               # WorkInfo 数据结构
│   │   └── workscheduler_srv_client.h  # 客户端代理类
│   ├── src/                   # 框架实现
│   │   ├── work_info.cpp            # WorkInfo 实现
│   │   └── workscheduler_srv_client.cpp  # IPC 客户端实现
│   └── extension/             # Extension 框架
│       ├── BUILD.gn
│       ├── include/
│       │   ├── work_scheduler_extension.h
│       │   ├── work_scheduler_extension_context.h
│       │   └── [其他扩展头文件]
│       └── src/
│           └── [扩展实现文件]
│
├── interfaces/                 # 对外接口层
│   └── kits/
│       ├── cj/                   # Cangjie 语言 FFI
│       │   ├── BUILD.gn
│       │   └── work_scheduler/
│       │       ├── work_scheduler_ffi.h       # FFI 头文件
│       │       └── work_scheduler_ffi.cpp     # FFI 实现（10 个函数）
│       ├── ets/                  # ArkTS/ETS 接口（Taihe 框架）
│       │   └── taihe/
│       │       ├── work_scheduler/
│       │       │   ├── idl/ohos.resourceschedule.workScheduler.taihe
│       │       │   ├── include/common.h
│       │       │   └── src/[ANI 实现文件]
│       │       └── work_scheduler_extension/
│       │           ├── ets//*.ets                      # ArkTS 类型定义
│       │           ├── include/ani_utils.h
│       │           └── src/ani_*.cpp                  # ANI 桥接实现
│       └── js/                    # JavaScript NAPI 接口
│           ├── BUILD.gn
│           ├── napi/
│           │   ├── include/                   # N-API 头文件
│           │   │   ├── init.h                   # 模块注册定义
│           │   │   ├── start_work.h
│           │   │   ├── stop_work.h
│           │   │   ├── get_work_status.h
│           │   │   ├── obtain_all_works.h
│           │   │   ├── stop_and_clear_works.h
│           │   │   ├── is_last_work_time_out.h
│           │   ├── common.h                  # 通用工具类
│           │   └── common_want.h
│           │   └── src/                       # N-API 实现
│           │       ├── init.cpp                 # 模块初始化和注册
│           │       ├── start_work.cpp
│           │       ├── stop_work.cpp
│           │       ├── get_work_status.cpp
│           │       ├── obtain_all_works.cpp
│           │       ├── stop_and_clear_works.cpp
│           │       ├── is_last_work_time_out.cpp
│           │       ├── common.cpp
│           │       └── common_want.cpp
│           ├── work_scheduler_extension/
│           │   ├── BUILD.gn
│           │   ├── work_scheduler_extension_ability.js
│           │   └── work_scheduler_extension_ability_module.cpp
│           └── work_scheduler_extension_context/
│               ├── BUILD.gn
│               ├── work_scheduler_extension_context.js
│               └── work_scheduler_extension_context_module.cpp
│
├── services/                   # 服务端实现
│   ├── BUILD.gn
│   ├── native/                 # 原生服务实现
│   │   ├── include/
│   │   │   ├── conditions/            # 条件监听器头文件
│   │   │   │   ├── battery_level_listener.h
│   │   │   │   ├── battery_status_listener.h
│   │   │   │   ├── charger_listener.h
│   │   │   │   ├── condition_checker.h
│   │   │   │   ├── group_listener.h
│   │   │   │   ├── icondition_listener.h
│   │   │   │   ├── network_listener.h
│   │   │   │   ├── screen_listener.h
│   │   │   │   ├── storage_listener.h
│   │   │   │   ├── timer_info.h
│   │   │   │   └── timer_listener.h
│   │   │   ├── policy/                # 策略过滤器头文件
│   │   │   │   ├── app_data_clear_listener.h
│   │   │   │   ├── cpu_policy.h
│   │   │   │   ├── ipolicy_filter.h
│   │   │   │   ├── memory_policy.h
│   │   │   │   ├── power_mode_policy.h
│   │   │   │   └── thermal_policy.h
│   │   │   ├── detector_value.h
│   │   │   ├── event_publisher.h
│   │   │   ├── ipolicy_listener.h
│   │   │   ├── policy_type.h
│   │   │   ├── scheduler_bg_task_subscriber.h
│   │   │   ├── watchdog.h
│   │   │   ├── work_bundle_group_change_callback.h
│   │   │   ├── work_conn_manager.h
│   │   │   ├── work_datashare_helper.h
│   │   │   ├── work_event_handler.h
│   │   │   ├── work_policy_manager.h
│   │   │   ├── work_queue.h
│   │   │   ├── work_queue_event_handler.h
│   │   │   ├── work_queue_manager.h
│   │   │   ├── work_sched_config.h
│   │   │   ├── work_sched_data_manager.h
│   │   │   ├── work_scheduler_connection.h
│   │   │   ├── work_scheduler_service.h
│   │   │   ├── work_standby_state_change_callback.h
│   │   │   └── work_status.h
│   │   └── src/
│   │       ├── conditions/            # 条件监听器实现
│   │       │   ├── battery_level_listener.cpp
│   │       │   ├── battery_status_listener.cpp
│   │       │   ├── charger_listener.cpp
│   │       │   ├── condition_checker.cpp
│   │       │   ├── group_listener.cpp
│   │       │   ├── network_listener.cpp
│   │       │   ├── screen_listener.cpp
│   │       │   ├── storage_listener.cpp
│   │       │   └── timer_listener.cpp
│   │       ├── policy/                # 策略过滤器实现
│   │       │   ├── app_data_clear_listener.cpp
│   │       │   ├── cpu_policy.cpp
│   │       │   ├── memory_policy.cpp
│   │       │   ├── power_mode_policy.cpp
│   │       │   └── thermal_policy.cpp
│   │       ├── event_publisher.cpp
│   │       ├── scheduler_bg_task_subscriber.cpp
│   │       ├── watchdog.cpp
│   │       ├── work_bundle_group_change_callback.cpp
│   │       ├── work_conn_manager.cpp
│   │       ├── work_datashare_helper.cpp
│   │       ├── work_event_handler.cpp
│   │       ├── work_policy_manager.cpp
│   │       ├── work_queue.cpp
│   │       ├── work_queue_event_handler.cpp
│   │       ├── work_queue_manager.cpp
│   │       ├── work_sched_config.cpp
│   │       ├── work_sched_data_manager.cpp
│   │       ├── work_scheduler_connection.cpp
│   │       ├── work_scheduler_service.cpp
│   │       ├── work_standby_state_change_callback.cpp
│   │       └── work_status.cpp
│   └── zidl/                     # ZIDL 接口层
│       ├── BUILD.gn
│       ├── IWorkScheduler.idl      # Extension 回调接口定义
│       ├── include/
│       │   ├── work_scheduler_stub_ani.h
│       │   └── work_scheduler_stub_imp.h
│       └── src/
│           ├── work_scheduler_stub_ani.cpp
│           └── work_scheduler_stub_imp.cpp
│
├── sa_profile/                # System Ability 配置
│   ├── BUILD.gn
│   └── 1904.json              # WorkScheduler SA 配置
│
├── utils/                     # 工具模块
│   └── native/
│       ├── BUILD.gn
│       ├── include/
│       │   ├── work_sched_common.h
│       │   ├── work_sched_constants.h
│       │   ├── work_sched_errors.h
│       │   ├── work_sched_hilog.h
│       │   ├── work_sched_hisysevent_report.h
│       │   ├── work_sched_system_policy.h
│       │   └── work_sched_utils.h
│       └── src/
│           ├── work_sched_hisysevent_report.cpp
│           └── work_sched_utils.cpp
│
└── wiki/                      # 文档目录
    ├── README.md
    ├── SUMMARY.md
    ├── [00-09 章节文档]
    ├── appendix/
    │   ├── Callgraphs.md
    │   └── Config_Flags.md
    └── _work/
        ├── NOTES.md
        └── PLAN.md
```

---

## 各层职责说明

### Frameworks 层（框架层）

**位置**: `frameworks/`

**职责**:
- 提供 Work Scheduler 服务的**客户端 SDK**
- 封装 IPC 调用，简化应用层使用
- 定义 WorkInfo 和 WorkCondition 数据结构
- 实现 WorkSchedulerExtension 框架（支持应用扩展）

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `WorkSchedulerSrvClient` | `src/workscheduler_srv_client.cpp` | IPC 客户端代理，单例模式，管理 SA 连接 |
| `WorkInfo` | `src/work_info.cpp` | 任务信息封装（id、bundleName、abilityName、条件等） |
| `WorkCondition` | `include/work_condition.h` | 条件类型枚举（网络、电池、充电、存储） |
| `WorkSchedulerExtension` | `extension/src/work_scheduler_extension.cpp` | Extension 框架基础类 |
| `JsWorkSchedulerExtension` | `extension/src/js_work_scheduler_extension.cpp` | JS Extension 实现 |

---

### Interfaces 层（接口层）

**位置**: `interfaces/kits/`

**职责**:
- 提供**多语言接口绑定**（JS/TS/CJ/ArkTS）
- N-API 模块注册和初始化
- 参数解析和校验
- 错误码映射

**子模块职责**:

#### 1. JS/NAPI 模块 (`interfaces/kits/js/`)
- **主模块**: `resourceschedule.workScheduler`
- **职责**: 6 个核心 API 方法 + 4 个枚举类型
- **输出**: `libworkscheduler.so`

#### 2. Extension NAPI (`interfaces/kits/js/napi/*extension*`)
- **ExtensionAbility**: `WorkSchedulerExtensionAbility`
- **ExtensionContext**: `application.WorkSchedulerExtensionContext`
- **职责**: Extension 框架的 N-API 绑定

#### 3. CJ FFI 模块 (`interfaces/kits/cj/`)
- **职责**: Cangjie 语言绑定
- **输出**: `libcj_work_scheduler_ffi.so`
- **接口**: 10 个 FFI 函数（V1 + V2 版本）

#### 4. ArkTS/Taihe 模块 (`interfaces/kits/ets/`)
- **职责**: ArkTS 原生接口（未来方向）
- **输出**: `libwork_scheduler.so` (ANI), `libwork_scheduler_extension_ani.so`
- **生成**: IDL 编译器自动生成 ANI 代码

---

### Services 层（服务层）

**位置**: `services/`

**职责**:
- 实现 **Work Scheduler System Ability**（SA 1904）
- 管理**任务队列**和调度决策
- 实现**条件监听器**（网络、电池、屏幕等）
- 实现**策略过滤器**（CPU、内存、温度、功耗）
- 处理**权限检查**和身份验证
- 管理与**Extension 的回调连接**

**核心组件**:

#### 1. 条件监听器 (`native/src/conditions/`)
| 监听器 | 触发条件 | 依赖系统服务 |
|--------|---------|-------------|
| `NetworkListener` | 网络类型 | netmanager_base |
| `BatteryLevelListener` | 电池电量 | battery_manager |
| `BatteryStatusListener` | 电池状态 | battery_manager |
| `ChargerListener` | 充电类型 | battery_manager |
| `StorageListener` | 存储状态 | data_share |
| `ScreenListener` | 屏幕状态 | eventhandler/ffrt |
| `TimerListener` | 定时器触发 | time_service |
| `GroupListener` | 应用组变更 | device_usage_statistics |

#### 2. 策略过滤器 (`native/src/policy/`)
| 过滤器 | 过滤条件 | 实现逻辑 |
|--------|---------|---------|
| `CpuPolicy` | CPU 使用率 | 超过阈值则过滤 |
| `MemoryPolicy` | 内存占用 | 超过阈值则过滤 |
| `ThermalPolicy` | 温度 | 超过阈值则过滤 |
| `PowerModePolicy` | 功耗模式 | 节能模式时过滤 |
| `AppDataClearListener` | 数据清理 | 应用数据清理时暂停任务 |

#### 3. 队列管理 (`native/src/`)
| 组件 | 职责 |
|------|------|
| `WorkQueueManager` | 管理所有任务队列 |
| `WorkQueue` | 单个任务队列（按 UID 分组） |
| `WorkPolicyManager` | 协调条件监听和策略过滤 |
| `WorkSchedulerConnection` | 管理 Extension 回调连接 |
| `WorkSchedulerService` | SA 主服务类（SystemAbility + WorkSchedServiceStub） |

---

### Utils 层（工具层）

**位置**: `utils/native/`

**职责**:
- 提供**通用工具函数**
- 定义**常量和错误码**
- 封装**HiLog 日志**
- 封装**HiSysEvent 事件上报**

**关键工具**:
| 模块 | 功能 | 文件 |
|------|------|------|
| 日志封装 | HiLog 统一接口 | `work_sched_hilog.h/.cpp` |
| 事件上报 | HiSysEvent 统一接口 | `work_sched_hisysevent_report.h/.cpp` |
| Bundle 工具 | 获取应用信息 | `work_sched_utils.h/.cpp` |
| 系统检查 | 应用身份判断 | `work_sched_utils.h/.cpp` |
| 常量定义 | Magic numbers, 常量 | `work_sched_constants.h` |
| 错误码 | 错误码定义 | `work_sched_errors.h` |

---

## 目录设计原则

### 分层架构

```
应用层（N-API）
    ↓ 调用
框架层（Frameworks）→ IPC 客户端
    ↓ IPC 通信
服务层（Services）→ SA 实现
    ↓ 内部调用
工具层（Utils）→ 通用工具
```

### 依赖方向

- **上层依赖下层**: N-API → Frameworks → Services → Utils
- **无循环依赖**: 通过接口和 IDL 解耦
- **可选依赖**: 策略和条件监听器可通过 Feature Flags 独立启用/禁用

---

## 代码组织最佳实践

| 原则 | 应用 |
|------|------|
| **职责单一** | 每个模块职责明确（条件监听 vs 策略过滤） |
| **接口隔离** | 通过头文件和 IDL 定义公共接口 |
| **配置驱动** | Feature Flags 控制可选功能 |
| **分层清晰** | interfaces/frameworks/services/utils 层次分明 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 模块概览
- [03_Architecture.md](03_Architecture.md) - 架构设计详解
- [04_External_API.md](04_External_API.md) - 对外 API 清单

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| 目录结构 | `wiki/_work/NOTES.md:31-102` (目录树) |
| 框架层职责 | `frameworks/BUILD.gn:34-74` (workschedclient) |
| 接口层职责 | `interfaces/kits/js/BUILD.gn:22-68` (workscheduler) |
| 服务层职责 | `services/BUILD.gn:27-152` (workschedservice) |
| 工具层职责 | `utils/native/BUILD.gn:20-52` (workschedutils) |
