# 目录结构与模块职责

## 总体目录结构

```
foundation/resourceschedule/background_task_mgr/
├── BUILD.gn                    # 根构建脚本
├── bgtaskmgr.gni               # GN变量定义
├── bundle.json                 # 组件配置
├── hisysevent.yaml             # 系统事件配置
├── LICENSE                     # Apache 2.0许可证
├── README.md / README_ZH.md    # 项目说明
│
├── frameworks/                 # 框架层实现
│   ├── common/                 # 通用工具
│   │   ├── include/            # 日志、错误码定义
│   │   └── src/
│   ├── include/                # 对外C++ API头文件
│   └── src/                    # 客户端代理实现
│
├── interfaces/                 # 接口定义层
│   ├── innerkits/              # 对内接口（C++）
│   │   ├── include/            # 数据结构、IDL接口
│   │   └── src/                # 实现代码
│   └── kits/                   # 对外接口（JS/ArkTS/C）
│       ├── napi/               # N-API绑定实现
│       ├── c/                  # C API（NDK）
│       ├── cj/                 # Cangjie FFI
│       └── ets/taihe/          # ArkTS 1.2 ANI绑定
│
├── sa_profile/                 # SA（System Ability）配置
│   ├── 1903.json               # SA ID 1903配置
│   └── BUILD.gn
│
├── services/                   # 服务实现
│   ├── core/                   # 主服务入口
│   ├── common/                 # 公共服务组件
│   ├── transient_task/         # 短时任务子服务
│   ├── continuous_task/        # 长时任务子服务
│   └── efficiency_resources/   # 能效资源子服务
│
├── resources/                  # 资源文件
│   ├── main/                   # HAP资源
│   └── signature/              # 签名配置
│
└── test/                       # 测试代码（本文档不详细覆盖）
```

---

## 核心模块职责

### 1. frameworks/ - 框架层

**职责**: 提供客户端API和IPC代理

| 目录/文件 | 职责说明 |
|-----------|----------|
| `include/background_task_manager.h` | 客户端主类`BackgroundTaskManager`定义 |
| `src/background_task_manager.cpp` | IPC代理封装，提供C++业务API |
| `common/include/bgtaskmgr_inner_errors.h` | 错误码定义（连续、瞬时、能效资源） |

**关键类**:
- `BackgroundTaskManager` - 单例客户端类，封装所有IPC调用
- `BgTaskMgrDeathRecipient` - 服务死亡监听

---

### 2. interfaces/ - 接口层

#### 2.1 innerkits/ - 对内接口

**职责**: 定义IPC数据结构、IDL接口和内部API

| 目录 | 内容 |
|------|------|
| `include/` | 40+头文件，定义所有数据结构和接口 |
| `src/` | 18个实现文件 |

**关键接口与数据结构**:

| 文件 | 定义 |
|------|------|
| `IBackgroundTaskMgr.idl` | IPC接口定义（40+方法） |
| `background_task_mgr_helper.h` | `BackgroundTaskMgrHelper`静态API |
| `continuous_task_param.h` | `ContinuousTaskParam`参数结构 |
| `delay_suspend_info.h` | `DelaySuspendInfo`短时任务信息 |
| `efficiency_resource_info.h` | `EfficiencyResourceInfo`资源信息 |
| `resource_type.h` | 资源类型枚举（CPU/GPS等） |
| `background_mode.h` | 后台模式枚举（9种模式） |

#### 2.2 kits/napi/ - N-API绑定

**职责**: JS/ArkTS接口的Native绑定

| 文件 | 功能 |
|------|------|
| `init.cpp` | 旧版模块注册（backgroundTaskManager） |
| `init_bgtaskmgr.cpp` | 新版模块注册（resourceschedule.backgroundTaskManager） |
| `bg_continuous_task_napi_module.cpp` | 长时任务N-API实现 |
| `request_suspend_delay.cpp` | 短时任务申请N-API |
| `cancel_suspend_delay.cpp` | 取消短时任务N-API |
| `efficiency_resources_operation.cpp` | 能效资源N-API |
| `common.cpp` | 通用工具函数（错误处理、类型转换） |

---

### 3. services/ - 服务层

#### 3.1 core/ - 主服务入口

**文件**: `background_task_mgr_service.h/cpp`

**职责**: 
- System Ability生命周期管理（OnStart/OnStop）
- IPC请求路由分发
- 权限检查入口

**关键方法**:
- `OnStart()` - 启动服务，初始化三大子管理器
- `OnStop()` - 停止服务
- `RequestSuspendDelay()` - 路由到短时任务管理器
- `StartBackgroundRunning()` - 路由到长时任务管理器
- `ApplyEfficiencyResources()` - 路由到能效资源管理器

#### 3.2 transient_task/ - 短时任务子服务

**文件**: `bg_transient_task_mgr.h/cpp`等11个文件

**职责**: 管理短时任务（延迟挂起）

**关键组件**:

| 文件 | 职责 |
|------|------|
| `bg_transient_task_mgr.h/cpp` | 主管理器，单例模式 |
| `timer_manager.h/cpp` | 定时器管理，超时检测 |
| `watchdog.h/cpp` | 看门狗监控 |
| `decision_maker.h/cpp` | 决策引擎，判断申请是否通过 |
| `device_info_manager.h/cpp` | 设备信息管理 |
| `suspend_controller.h/cpp` | 挂起控制逻辑 |
| `pkg_delay_suspend_info.h/cpp` | 应用延时信息记录 |

**数据流**:
```
RequestSuspendDelay() → IsCallingInfoLegal() → DecisionMaker::Decide()
                     ↓
              TimerManager::StartTimer() → Watchdog监控
                     ↓
              超时回调 → ForceCancelSuspendDelay()
```

#### 3.3 continuous_task/ - 长时任务子服务

**文件**: `bg_continuous_task_mgr.h/cpp`等6个文件

**职责**: 管理长时任务（持续后台运行）

**关键组件**:

| 文件 | 职责 |
|------|------|
| `bg_continuous_task_mgr.h/cpp` | 主管理器（200+方法） |
| `continuous_task_record.h/cpp` | 任务记录数据结构 |
| `notification_tools.h/cpp` | 通知栏工具 |
| `banner_notification_record.h/cpp` | Banner通知记录 |
| `task_notification_subscriber.h/cpp` | 通知订阅者 |
| `config_change_observer.h/cpp` | 配置变更监听 |

**数据流**:
```
StartBackgroundRunning() → CheckPermission() → CheckBgmodeType()
                      ↓
              Create ContinuousTaskRecord → SendNotification()
                      ↓
              AppStateObserver监控 → OnAppStopped() → StopBackgroundRunning()
```

#### 3.4 efficiency_resources/ - 能效资源子服务

**文件**: `bg_efficiency_resources_mgr.h/cpp`等3个文件

**职责**: 管理能效资源申请

**关键组件**:

| 文件 | 职责 |
|------|------|
| `bg_efficiency_resources_mgr.h/cpp` | 主管理器 |
| `resource_application_record.h/cpp` | 资源申请记录 |
| `resources_subscriber_mgr.h/cpp` | 订阅管理 |

**数据流**:
```
ApplyEfficiencyResources() → CheckResourceInfo() → ApplyEfficiencyResourcesInner()
                       ↓
              UpdateResourcesEndtime() → ReportHisysEvent()
                       ↓
              App死亡回调 → RemoveProcessRecord()
```

#### 3.5 common/ - 公共服务组件

**文件**: 9个公共组件

| 文件 | 职责 |
|------|------|
| `bundle_manager_helper.h/cpp` | Bundle信息查询、权限检查 |
| `app_mgr_helper.h/cpp` | Ability管理器交互 |
| `app_state_observer.h/cpp` | 应用状态监听 |
| `data_storage_helper.h/cpp` | 数据持久化（关系型数据库） |
| `bgtask_config.h/cpp` | 配置管理、签名验证 |
| `system_event_observer.h/cpp` | 系统事件监听 |
| `report_hisysevent_data.h/cpp` | 事件上报 |

---

### 4. sa_profile/ - SA配置

**文件**: `1903.json`

**SA ID**: 1903

**配置**:
```json
{
    "process": "resource_schedule_service",
    "systemability": [{
        "name": 1903,
        "libpath": "libbgtaskmgr_service.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1,
        "extension": ["backup", "restore"]
    }]
}
```

---

### 5. resources/ - 资源文件

**结构**:
```
resources/
├── main/
│   ├── config.json         # HAP配置
│   └── resources/          # 多语言字符串资源
│       ├── base/           # 默认资源
│       ├── zh_CN/          # 中文
│       ├── en_US/          # 英文
│       └── ... (70+语言)
├── signature/
│   └── BackgroundTaskResources.gni
├── BackgroundTaskResources.p7b    # 签名证书
└── BUILD.gn
```

**产物**: `BackgroundTaskResources.hap` 安装到 `app/com.ohos.backgroundtaskmgr.resources`

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                         JS/ArkTS App                         │
└───────────────────────────┬─────────────────────────────────┘
                            │ N-API
┌───────────────────────────▼─────────────────────────────────┐
│                    interfaces/kits/napi                      │
│                    (N-API绑定层)                              │
└───────────────────────────┬─────────────────────────────────┘
                            │ 调用
┌───────────────────────────▼─────────────────────────────────┐
│                   frameworks/                                │
│              BackgroundTaskManager                           │
│                    (客户端API)                                │
└───────────────────────────┬─────────────────────────────────┘
                            │ IPC (Binder)
┌───────────────────────────▼─────────────────────────────────┐
│              interfaces/innerkits/                           │
│              BackgroundTaskMgrProxy                          │
│                    (IPC代理层)                                │
└───────────────────────────┬─────────────────────────────────┘
                            │ OnRemoteRequest
┌───────────────────────────▼─────────────────────────────────┐
│              services/core/                                  │
│         BackgroundTaskMgrService                             │
│            (System Ability 1903)                             │
└───────┬───────────────┬───────────────┬─────────────────────┘
        │               │               │
┌───────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
│ transient_   │ │ continuous_ │ │ efficiency_ │
│ task/        │ │ task/       │ │ resources/  │
│ 短时任务管理  │ │ 长时任务管理 │ │ 能效资源管理 │
└──────────────┘ └─────────────┘ └─────────────┘
```

---

## TODO(需确认)

- [ ] 确认utils目录的实际用途（看起来是符号链接到services/common）
- [ ] 补充每个模块的初始化顺序和依赖条件
- [ ] 确认各子服务的线程模型细节
