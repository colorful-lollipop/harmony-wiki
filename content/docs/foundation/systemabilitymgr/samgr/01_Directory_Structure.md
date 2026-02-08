# 目录结构与模块职责

## 顶层目录结构

```
samgr/
├── bundle.json              # 组件描述和构建配置
├── config.gni               # 构建变量配置
├── Cargo.toml               # Rust 包配置
├── hisysevent.yaml          # 系统事件配置
│
├── etc/                     # 配置文件
├── figures/                 # 文档图片
├── frameworks/              # 框架实现（客户端代码）
├── interfaces/              # 接口定义（InnerKits）
├── services/                # 服务实现
├── utils/                   # 工具代码
└── wiki/                    # 本文档
```

## 各目录详细说明

### 1. interfaces/innerkits/ - 接口定义

对外暴露的 API，供其他组件和应用程序调用。

#### samgr_proxy/ - 核心代理接口

| 文件 | 说明 |
|------|------|
| `if_system_ability_manager.h` | **核心接口**：`ISystemAbilityManager` 定义，包含所有 SA 管理方法 |
| `iservice_registry.h` | 服务注册接口 `IServiceRegistry` |
| `system_ability_manager_proxy.h` | 客户端代理类 `SystemAbilityManagerProxy` |
| `system_ability_definition.h` | **SA ID 定义**：所有子系统的 SA ID 常量（300+ 个） |
| `samgr_ipc_interface_code.h` | IPC 命令码定义 |
| `samgr_err_code.h` | 错误码定义 |
| `isystem_ability_load_callback.h` | 加载回调接口 |
| `isystem_ability_status_change.h` | 状态变化监听接口 |

#### common/ - 公共工具

| 文件 | 说明 |
|------|------|
| `sa_profiles.h` | SA 配置文件结构定义 |
| `parse_util.h` | 配置文件解析工具 |

#### dynamic_cache/ - 动态缓存

| 文件 | 说明 |
|------|------|
| `dynamic_cache.h/cpp` | 动态 SA 缓存实现 |

#### rust/ - Rust 绑定

| 文件 | 说明 |
|------|------|
| `src/lib.rs` | Rust API 入口 |
| `src/cxx/*.cpp` | C++ 包装层 |

### 2. services/ - 服务实现

#### samgr/native/ - 主服务实现

**核心文件**：

| 文件 | 职责 |
|------|------|
| `main.cpp` | 服务入口点 |
| `system_ability_manager.cpp/h` | **核心类**：`SystemAbilityManager` 实现 |
| `system_ability_manager_stub.cpp/h` | IPC Stub 实现，处理远程调用 |
| `system_ability_manager_util.cpp/h` | 工具函数 |
| `ability_death_recipient.cpp/h` | 死亡监听（Death Recipient） |

**状态管理 (schedule/)**：

| 文件 | 职责 |
|------|------|
| `system_ability_state_scheduler.cpp/h` | 状态调度器，管理 SA 生命周期 |
| `system_ability_state_machine.cpp/h` | 状态机实现 |
| `system_ability_event_handler.cpp/h` | 事件处理器 |

**设备状态收集 (collect/)**：

| 文件 | 职责 |
|------|------|
| `device_status_collect_manager.cpp/h` | 收集管理器 |
| `device_param_collect.cpp/h` | 设备参数收集 |
| `device_networking_collect.cpp/h` | 网络状态收集（条件编译） |
| `common_event_collect.cpp/h` | 通用事件收集（条件编译） |
| `device_switch_collect.cpp/h` | 设备开关事件 |
| `ref_count_collect.cpp/h` | 引用计数收集 |

#### lsamgr/ - 本地 SA 管理

| 文件 | 职责 |
|------|------|
| `local_abilitys.cpp/h` | 本地 SA 管理 |
| `local_ability_manager_proxy.cpp` | 本地管理代理 |

#### dfx/ - 诊断与跟踪

| 文件 | 职责 |
|------|------|
| `hisysevent_adapter.cpp/h` | HiSysEvent 事件上报适配 |
| `samgr_xcollie.cpp` | XCollie 看门狗集成 |

### 3. frameworks/native/ - 客户端框架

供客户端使用的代理实现。

| 文件 | 职责 |
|------|------|
| `system_ability_manager_proxy.cpp` | 客户端代理实现 |
| `system_ability_load_callback_stub.cpp` | 加载回调 Stub |
| `system_ability_status_change_stub.cpp` | 状态变化 Stub |
| `system_process_status_change_stub.cpp` | 进程状态 Stub |

### 4. utils/native/ - 工具代码

| 文件 | 职责 |
|------|------|
| `tools.cpp/h` | 通用工具函数 |

### 5. etc/ - 配置文件

| 文件 | 职责 |
|------|------|
| `samgr_standard.cfg` | 服务启动配置（boot-mode, critical） |
| `samgr.para` | 系统参数定义 |
| `samgr.para.dac` | DAC 访问控制参数 |

## 模块依赖关系

```
                    ┌──────────────────┐
                    │   应用/客户端     │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  samgr_proxy     │  ← 客户端库
                    │  (frameworks/)   │
                    └────────┬─────────┘
                             │ IPC
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼─────┐ ┌──────▼──────┐ ┌────▼─────┐
     │   samgr      │ │  lsamgr     │ │   dfx    │
     │  (services/) │ │ (services/) │ │(services)│
     └──────────────┘ └─────────────┘ └──────────┘
```

## 关键文件速查

### 接口定义
- **主接口**: `interfaces/innerkits/samgr_proxy/include/if_system_ability_manager.h:39`
- **SA ID**: `interfaces/innerkits/samgr_proxy/include/system_ability_definition.h:32`
- **错误码**: `interfaces/innerkits/common/include/samgr_err_code.h:20`

### 核心实现
- **服务主类**: `services/samgr/native/include/system_ability_manager.h:61`
- **服务入口**: `services/samgr/native/source/main.cpp`
- **IPC Stub**: `services/samgr/native/source/system_ability_manager_stub.cpp`

### 构建配置
- **组件配置**: `bundle.json`
- **构建变量**: `config.gni`
- **服务变量**: `services/samgr/var.gni`
