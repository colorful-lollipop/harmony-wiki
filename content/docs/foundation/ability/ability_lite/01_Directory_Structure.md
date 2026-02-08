# 目录结构与模块职责

## 目的

本文档描述 ability_lite 的目录结构，说明各模块的职责和关键文件位置。

## 适用范围

- 需要了解代码组织的新开发者
- 需要定位特定功能的维护人员

## 顶层目录结构

```
foundation/ability/ability_lite/
├── ability_lite.gni          # GN 构建变量定义
├── bundle.json               # 组件配置（版本、依赖、特性）
├── README.md / README_zh.md  # 项目说明文档
├── LICENSE / OAT.xml         # 许可证和归属
├── figures/                  # 架构图资源
├── frameworks/               # 框架实现代码
├── interfaces/               # 接口定义
└── services/                 # 系统服务实现
```

## frameworks/ - 框架实现

### frameworks/ability_lite/ - AbilityKit 核心

**职责**: 提供应用开发框架，实现 Ability 生命周期和基础能力

```
frameworks/ability_lite/
├── BUILD.gn
├── include/                  # 内部头文件
│   ├── ability_env_impl.h
│   ├── ability_scheduler.h   # Ability 任务调度
│   ├── ability_slice_manager.h
│   ├── ability_thread.h      # Ability 主线程
│   └── ...
├── src/                      # 标准系统实现
│   ├── ability.cpp           # Ability 基类实现
│   ├── ability_context.cpp
│   ├── ability_loader.cpp    # Ability 注册加载
│   ├── ability_main.cpp      # 应用入口
│   ├── ability_scheduler.cpp
│   ├── ability_slice*.cpp    # AbilitySlice 管理
│   └── ability_thread.cpp    # 主线程实现
├── src/slite/                # LiteOS-M 专用实现
│   ├── slite_ability.cpp
│   └── lite_context.cpp
└── example/                  # 示例应用
    └── entry/
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `src/ability.cpp` | Ability 基类，生命周期回调 |
| `src/ability_loader.cpp` | Ability 注册与加载 |
| `src/ability_thread.cpp` | 应用主线程，IPC 处理 |
| `src/ability_scheduler.cpp` | 接收 AMS 指令并执行 |

### frameworks/abilitymgr_lite/ - AMS 客户端

**职责**: 提供 AbilityKit 与 AMS 通信的客户端接口

```
frameworks/abilitymgr_lite/
├── BUILD.gn
├── include/
│   ├── abilityms_client.h    # AMS 通信客户端
│   └── ability_kit.h
├── src/
│   ├── abilityms_client.cpp  # IPC 客户端实现
│   ├── ability_manager.cpp   # AbilityManager API
│   └── slite/                # LiteOS-M 实现
│       └── ability_manager_client.cpp
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `src/abilityms_client.cpp:1` | 与 AMS 的 IPC 通信 |
| `src/ability_manager.cpp:1` | StartAbility/StopAbility 等 API |

### frameworks/want_lite/ - Want 实现

**职责**: 实现 Want 数据结构，用于 Ability 间信息传递

```
frameworks/want_lite/
├── BUILD.gn
├── include/want_utils.h
└── src/want.cpp              # Want 序列化/反序列化
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `src/want.cpp:1` | Want 数据结构操作，DATA_LENGTH 限制 2048 字节 |

## interfaces/ - 接口定义

### interfaces/kits/ - 对外公开 API

#### interfaces/kits/ability_lite/ - AbilityKit API

**职责**: 应用开发使用的公开头文件

```
interfaces/kits/ability_lite/
├── ability.h                 # Ability 基类定义
├── ability_manager.h         # C API: StartAbility/StopAbility
├── ability_slice.h           # AbilitySlice 定义
├── ability_context.h
├── ability_loader.h          # REGISTER_AA 宏
├── ability_connection.h
├── ability_state.h           # 生命周期状态枚举
├── ability_event_handler.h
├── ability_env.h
├── ability_errors.h          # 错误码定义
└── slite/                    # LiteOS-M API
    ├── slite_ability.h
    ├── ability_manager.h
    └── lite_context.h
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `ability.h:75` | `class Ability` 基类定义 |
| `ability_manager.h:70` | `StartAbility()` / `StopAbility()` |
| `ability_loader.h` | `REGISTER_AA()` 宏 |
| `ability_errors.h` | 错误码枚举 |

#### interfaces/kits/want_lite/ - Want API

```
interfaces/kits/want_lite/
└── want.h                    # Want 结构体和操作函数
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `want.h:57` | `typedef struct { ... } Want` |

#### interfaces/kits/js/ - JavaScript 绑定

```
interfaces/kits/js/
├── napi/
│   ├── BUILD.gn
│   └── js_aafwk.cpp          # N-API 实现
└── declaration/
    ├── BUILD.gn
    └── api/
        └── @ohos.aafwk.d.ts  # TypeScript 声明
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `napi/js_aafwk.cpp:43` | `JSAafwkStartAbility()` |
| `napi/js_aafwk.cpp:67` | `JSAafwkStopAbility()` |
| `napi/js_aafwk.cpp:171` | N-API 模块注册 |

### interfaces/inner_api/ - 内部 API

**职责**: 供其他子系统使用的内部接口

```
interfaces/inner_api/abilitymgr_lite/
├── ability_service_interface.h   # AMS 服务接口定义
├── ability_main.h
└── slite/                        # LiteOS-M 内部 API
    ├── ability_manager_inner.h
    ├── ability_record_observer.h
    └── bms_helper.h
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `ability_service_interface.h:44` | `AmsCommand` 枚举 |
| `ability_service_interface.h:70` | `struct AmsInterface` |

## services/ - 系统服务

### services/abilitymgr_lite/ - AMS 实现

**职责**: Ability Manager Service 实现，运行在 foundation 进程

```
services/abilitymgr_lite/
├── BUILD.gn
├── include/                      # 服务内部头文件
│   ├── ability_mgr_service.h     # AMS 服务类
│   ├── ability_mgr_feature.h     # SAMGR Feature 实现
│   ├── ability_mgr_handler.h     # 消息处理器
│   ├── ability_stack_manager.h   # Ability 栈管理
│   ├── ability_worker.h          # 任务工作器
│   ├── app_manager.h             # 应用进程管理
│   ├── app_record.h              # 应用记录
│   ├── ability_message_id.h      # 消息 ID 定义
│   ├── client/                   # 服务客户端
│   │   ├── app_spawn_client.h    # AppSpawn 通信
│   │   ├── bundlems_client.h     # BundleMS 通信
│   │   └── wms_client.h          # 窗口管理器通信
│   ├── task/                     # 生命周期任务
│   │   ├── ability_start_task.h
│   │   ├── ability_stop_task.h
│   │   ├── ability_terminate_task.h
│   │   └── ... (共 15+ 个任务)
│   └── util/                     # 工具类
│       └── abilityms_helper.h
├── src/                          # 服务实现
│   ├── ability_mgr_service.cpp   # 服务注册
│   ├── ability_mgr_feature.cpp   # IPC 命令处理
│   ├── ability_mgr_handler.cpp   # 消息分发
│   ├── ability_stack_manager.cpp
│   ├── app_manager.cpp
│   ├── app_record.cpp            # 权限加载
│   ├── client/
│   ├── task/                     # 任务实现
│   └── util/
├── tools/                        # aa 命令行工具
│   ├── BUILD.gn
│   └── src/
│       ├── ability_tool.cpp
│       └── main.cpp
└── unittest/                     # 单元测试（忽略）
```

**关键文件**:
| 文件 | 职责 |
|------|------|
| `src/ability_mgr_service.cpp:46` | AMS 服务注册到 SAMGR |
| `src/ability_mgr_feature.cpp:147` | `StartAbilityInvoke()` IPC 入口 |
| `src/ability_mgr_feature.cpp:88` | `Invoke()` 命令分发 |
| `src/app_record.cpp:60` | `LoadPermissions()` 权限加载 |
| `src/task/ability_start_task.cpp` | 启动 Ability 任务 |

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────┐
│  Application (JS/Native)                                │
└─────────────────────┬───────────────────────────────────┘
                      │ uses
                      ▼
┌─────────────────────────────────────────────────────────┐
│  AbilityKit (frameworks/ability_lite)                   │
│  - Ability 生命周期                                     │
│  - AbilitySlice 管理                                    │
└─────────────────────┬───────────────────────────────────┘
                      │ uses
                      ▼
┌─────────────────────────────────────────────────────────┐
│  AbilityManager Client (frameworks/abilitymgr_lite)     │
│  - IPC 客户端                                           │
└─────────────────────┬───────────────────────────────────┘
                      │ IPC (IpcIo)
                      ▼
┌─────────────────────────────────────────────────────────┐
│  AMS Service (services/abilitymgr_lite)                 │
│  - Ability 调度                                         │
│  - 进程管理                                             │
│  - 栈管理                                               │
└─────────────────────────────────────────────────────────┘
```

## 按功能定位文件

### 生命周期管理
| 功能 | 文件位置 |
|------|----------|
| 状态定义 | `interfaces/kits/ability_lite/ability_state.h` |
| 基类回调 | `interfaces/kits/ability_lite/ability.h:87-122` |
| 状态调度 | `frameworks/ability_lite/src/ability_scheduler.cpp` |
| 启动任务 | `services/abilitymgr_lite/src/task/ability_start_task.cpp` |
| 停止任务 | `services/abilitymgr_lite/src/task/ability_stop_task.cpp` |

### IPC 通信
| 功能 | 文件位置 |
|------|----------|
| 服务接口 | `interfaces/inner_api/abilitymgr_lite/ability_service_interface.h` |
| 客户端 | `frameworks/abilitymgr_lite/src/abilityms_client.cpp` |
| 服务端 | `services/abilitymgr_lite/src/ability_mgr_feature.cpp` |
| 命令枚举 | `interfaces/inner_api/abilitymgr_lite/ability_service_interface.h:44` |

### 权限管理
| 功能 | 文件位置 |
|------|----------|
| 权限加载 | `services/abilitymgr_lite/src/app_record.cpp:60` |
| 权限查询 | `services/abilitymgr_lite/src/app_record.cpp:76` |

### N-API / JS 绑定
| 功能 | 文件位置 |
|------|----------|
| 模块注册 | `interfaces/kits/js/napi/js_aafwk.cpp:171` |
| startAbility | `interfaces/kits/js/napi/js_aafwk.cpp:43` |
| stopAbility | `interfaces/kits/js/napi/js_aafwk.cpp:67` |
| TS 声明 | `interfaces/kits/js/declaration/api/@ohos.aafwk.d.ts` |

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](02_Architecture.md)
- [GN 构建目标](06_GN_Targets.md)
