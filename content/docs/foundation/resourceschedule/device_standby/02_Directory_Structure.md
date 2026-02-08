# 目录结构与模块职责

## 顶层目录

```
/foundation/resourceschedule/device_standby/
├── frameworks/          # API接口框架
├── interfaces/         # 接口定义
│   ├── innerkits/       # 对内接口
│   └── kits/            # 对外接口
├── plugins/            # 插件模块
├── services/           # 核心服务
├── utils/              # 工具模块
├── sa_profile/         # SA配置
├── wiki/               # 文档目录
├── bundle.json         # 部件描述
├── standby_service.gni # GN构建配置
└── LICENSE
```

## 模块职责说明

### frameworks/

**职责**：提供对外接口框架能力

**子模块**：

| 目录/文件 | 职责 |
|-----------|------|
| `include/` | 头文件目录 |
| `standby_service_subscriber_proxy.cpp` | 订阅者代理实现 |

**稳定性**：对外接口，保持稳定

### interfaces/

**职责**：提供对外和对内的 API 接口

#### interfaces/innerkits/

**对内接口** - 供系统其他服务调用

| 头文件 | 职责 |
|--------|------|
| `allow_info.h` | 豁免信息结构 |
| `allow_type.h` | 豁免类型枚举 |
| `resource_request.h` | 资源请求结构 |
| `standby_service_client.h` | 待机服务客户端 |
| `standby_service_subscriber_stub.h` | 订阅存根 |
| `standby_state.h` | 待机状态结构 |

#### interfaces/kits/

**对外接口** - 供应用调用

| 目录/文件 | 职责 |
|-----------|------|
| `ani/` | ArkTS API (Taihe) |
| `napi/` | Node.js API |

### plugins/

**职责**：状态监控、决策、转换、执行策略

**子模块**：

| 目录 | 职责 |
|------|------|
| `ext/` | 基础接口（状态/策略/监听器适配器） |
| `standby_state/` | 设备状态管理 |
| `message_listener/` | 消息监听（输入/后台任务） |
| `extend_constraints/` | 扩展约束监控（充电/传感器） |
| `strategy/` | 策略管理（网络/锁/定时器） |

**插件架构**：

```
┌─────────────────────────────────────────────────────────┐
│                    Plugin Layer                         │
├─────────────────┬─────────────────┬─────────────────────┤
│  State Manager  │ Listener Manager│ Strategy Manager   │
│  (状态管理)      │ (消息监听)      │ (策略执行)          │
├─────────────────┼─────────────────┼─────────────────────┤
│ - working_state │ - input_listener│ - network_strategy │
│ - sleep_state   │ - bg_task_lis.  │ - running_lock_.. │
│ - nap_state     │                 │ - timer_strategy   │
│ - dark_state    │                 │                    │
│ - maintenance_..│                 │                    │
└─────────────────┴─────────────────┴─────────────────────┘
```

### services/

**职责**：核心服务实现

#### services/core/

**核心功能**

| 文件 | 职责 |
|------|------|
| `ability_manager_helper.cpp` | 能力管理器辅助 |
| `app_mgr_helper.cpp` | 应用管理器辅助 |
| `app_state_observer.cpp` | 应用状态观察者 |
| `bundle_manager_helper.cpp` | 包管理器辅助 |
| `common_event_observer.cpp` | 公共事件观察者 |
| `allow_record.cpp` | 豁免记录管理 |
| `standby_service.cpp` | 待机服务主类 |
| `standby_service_impl.cpp` | 待机服务实现 |

#### services/common/

**公共组件**

| 文件 | 职责 |
|------|------|
| `device_standby_switch.cpp` | 待机开关控制 |
| `time_provider.cpp` | 时间提供者 |
| `timed_task.cpp` | 定时任务 |
| `background_task_helper.cpp` | 后台任务辅助 |

#### services/notification/

**通知机制**

| 文件 | 职责 |
|------|------|
| `standby_state_subscriber.cpp` | 待机状态订阅者 |

### utils/

**职责**：通用工具和配置

#### utils/common/

**通用工具**

| 文件 | 职责 |
|------|------|
| `common_constant.cpp` | 常量定义 |
| `ipc_util.cpp` | IPC 工具 |
| `report_data_utils.cpp` | 数据上报工具 |

#### utils/policy/

**策略配置**

| 文件 | 职责 |
|------|------|
| `json_utils.cpp` | JSON 工具 |
| `standby_config_manager.cpp` | 配置管理器 |

### sa_profile/

**职责**：系统能力配置

| 文件 | 职责 |
|------|------|
| `1914.json` | SA ID 1914 配置 |

## 代码规模统计

| 模块 | 代码行数（估算） |
|------|-----------------|
| frameworks | ~500 行 |
| interfaces | ~2000 行 |
| plugins | ~5000 行 |
| services | ~6000 行 |
| utils | ~2000 行 |
| **总计** | ~15500 行 |

## 关键依赖方向

```
interfaces/kits/napi
        ↓
interfaces/innerkits (StandbyServiceClient)
        ↓
services/core (StandbyServiceImpl)
        ↓
plugins (状态/监听/策略)
```

> **说明**：依赖方向应避免环，plugin 层可被 services 调用，但不反向依赖。
