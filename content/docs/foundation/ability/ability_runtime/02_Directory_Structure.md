# 目录结构与模块职责

## 顶层目录结构

```
ability_runtime/
├── frameworks/              # 框架实现层
│   ├── native/             # C++ 原生框架
│   ├── js/napi/           # JavaScript N-API 绑定
│   ├── ets/               # ETS/ArkTS ANI 绑定
│   ├── cj/                # CJ FFI 绑定
│   ├── c/                 # C API
│   └── simulator/         # 模拟器
├── interfaces/             # 接口定义层
│   ├── inner_api/         # 内部组件间接口（InnerKit）
│   └── kits/              # 公共 SDK 接口
│       ├── native/        # Native SDK（C/C++）
│       └── c/             # C SDK
├── services/               # 系统服务层
│   ├── abilitymgr/        # AbilityManagerService
│   ├── appmgr/            # AppManagerService
│   ├── common/            # 公共服务组件
│   ├── dataobsmgr/        # DataAbility 观察者管理
│   ├── dialog_ui/         # 系统对话框
│   ├── quickfixmgr/       # 热修复管理
│   ├── sa_profile/        # SA 配置文件
│   └── uripermmgr/        # URI 权限管理
├── agent_runtime_framework/   # Agent 组件框架
├── service_router_framework/  # 服务路由框架
├── js_environment/         # JS 运行时环境
├── ets_environment/       # ETS 运行时环境
├── cj_environment/        # CJ 运行时环境
├── tools/                  # 工具
│   └── aa/                # aa 命令行工具
├── utils/                  # 公共工具
│   ├── server/            # 服务端工具
│   └── global/             # 全局工具
├── figures/                # 架构图等资源
└── test/                   # 测试目录（不纳入文档范围）
```

## frameworks/ 目录详解

### frameworks/native/

原生框架实现，提供 C++ 核心功能。

```
frameworks/native/
├── ability/native/         # Ability 核心实现
│   ├── include/          # 头文件
│   │   ├── ability.h
│   │   ├── ability_context.h
│   │   ├── extension.h
│   │   └── ...
│   ├── ability_thread.cpp
│   ├── ability_context.cpp
│   ├── extension_module.cpp
│   └── ...
├── appkit/               # 应用套件
│   ├── include/
│   │   ├── application.h
│   │   ├── ability_stage.h
│   │   └── test_runner.h
│   └── ...
├── child_process/         # 子进程管理
└── insight_intent/        # 意图框架
```

**模块职责**：
- `ability/native/`：提供 Ability 的原生实现，包括 UIAbility、ExtensionAbility 等
- `appkit/`：提供 Application、AbilityStage、TestRunner 等高层抽象
- `child_process/`：提供子进程创建和管理能力

### frameworks/js/napi/

JavaScript N-API 实现，为 JS 应用提供调用接口。

```
frameworks/js/napi/
├── ability/               # Ability 基础 API
├── ability_context/       # Ability 上下文 API
├── ability_manager/       # Ability 管理 API
├── application/           # 应用相关 API
├── app/                   # 应用套件 API
│   ├── ability_stage/
│   ├── ability_delegator/
│   ├── app_manager/
│   └── ...
├── featureAbility/        # FA 模型 API
├── particleAbility/       # PA 模型 API
├── extension_ability/     # ExtensionAbility API
├── service_extension_ability/  # Service Extension API
├── ui_extension_ability/  # UI Extension API
├── auto_fill_extension_ability/  # 自动填充 Extension API
├── mission_manager/       # 任务管理 API
├── caller/                # 调用者 API
├── callee/                # 被调用者 API
├── wantagent/            # Want 代理 API
├── uri_permission/        # URI 权限 API
├── configuration_constant/  # 配置常量
├── ability_constant/      # Ability 常量
├── errorcode/             # 错误码
├── insight_intent/        # 意图框架 API
├── app_startup/           # 应用启动 API
├── js_dialog_session/     # 对话框会话
├── quick_fix/             # 热修复 API
├── embedded_ui_extension_ability/  # 嵌入式 UI Extension
├── share_extension_ability/  # 分享 Extension
├── photo_editor_extension_ability/  # 图片编辑 Extension
├── ui_service_extension_ability/  # UI 服务 Extension
├── embeddable_ui_ability/  # 可嵌入 UI Ability
├── auto_fill_manager/     # 自动填充管理
├── ability_vertical_panel/  # 竖屏面板
├── kiosk_manager/         # Kiosk 模式管理
├── js_child_process/       # JS 子进程
├── completion_handler_for_abilitystartcallback/  # 启动回调
└── inner/                 # 内部通用实现
    ├── napi_common/       # N-API 公共工具
    ├── napi_ability_common/  # Ability N-API 公共实现
    └── napi_wantagent_common/  # WantAgent 公共实现
```

**模块职责**：为 JS/TS 应用提供调用 Ability 运行时功能的接口

### frameworks/ets/

ETS/ArkTS 原生接口绑定。

```
frameworks/ets/
├── ani/                   # ArkTS Native Interface
│   ├── ability/
│   ├── ability_context/
│   ├── ability_manager/
│   ├── app/
│   ├── featureAbility/
│   ├── ui_ability/
│   ├── ui_extension/
│   ├── service_extension/
│   ├── wantagent/
│   ├── want/
│   ├── mission_manager/
│   ├── caller_complex/
│   ├── child_process_manager/
│   ├── auto_fill_manager/
│   ├── dialog_request_info/
│   ├── quick_fix_manager/
│   └── ...
├── ets/                   # ETS 运行时
└── taihe/                 # 分布式相关
```

**模块职责**：为 ArkTS 应用提供类型安全的原生接口调用

### frameworks/cj/

CJ 语言 FFI 绑定。

```
frameworks/cj/
├── ffi/                   # FFI 实现
│   ├── ability/
│   ├── app/
│   │   ├── app_manager/
│   │   ├── errormanager/
│   │   ├── recovery/
│   │   └── ...
│   ├── context/
│   ├── want_agent/
│   ├── ark_interop_helper/
│   └── ...
└── environments/          # 运行时环境
```

## interfaces/ 目录详解

### interfaces/inner_api/

内部组件间接口（InnerKit），仅限系统组件使用。

```
interfaces/inner_api/
├── ability_manager/       # AbilityManager 接口
│   ├── include/
│   │   ├── ability_manager_client.h
│   │   ├── ability_connect_callback_stub.h
│   │   ├── launch_param.h
│   │   ├── mission_info.h
│   │   └── ...
│   └── BUILD.gn
├── app_manager/           # AppManager 接口
│   ├── include/
│   │   ├── app_mgr_client.h
│   │   └── page_state_data.h
│   └── BUILD.gn
├── extension_manager/     # Extension 管理接口
├── mission_manager/       # 任务管理接口
├── uri_permission/        # URI 权限接口
├── quick_fix/            # 热修复接口
├── runtime/               # 运行时接口
├── napi_base_context/     # N-API 上下文基础
├── ani_base_context/      # ANI 上下文基础
├── error_utils/           # 错误码工具
├── wantagent/             # WantAgent 接口
├── dataobs_manager/        # Data 观察者管理
├── connectionobs_manager/  # 连接观察者管理
├── foreground_app_obs_manager/  # 前台应用观察
├── auto_fill_manager/     # 自动填充管理
├── session_handler/       # 会话管理
├── page_config_manager/   # 页面配置管理
└── ...
```

### interfaces/kits/

公共 SDK 接口，对应用开放。

```
interfaces/kits/
├── native/               # Native SDK（C/C++）
│   └── ability/
│       ├── native/       # 基础能力接口
│       │   ├── ability_context.h
│       │   ├── extension.h
│       │   ├── service_extension.h
│       │   └── ...
│       ├── appkit/       # 应用套件接口
│       │   ├── application.h
│       │   ├── ability_stage.h
│       │   └── ...
│       └── ability_runtime/  # 运行时接口
│           ├── context_constant.h
│           ├── start_options.h
│           └── ...
└── c/                    # C SDK
    └── ability_runtime/
        ├── ability_runtime_common.h
        ├── application_context.h
        ├── context_constant.h
        └── native_child_process.h
```

## services/ 目录详解

### services/abilitymgr/

Ability 管理服务实现。

```
services/abilitymgr/
├── include/              # 头文件
│   ├── ability_manager_service.h
│   ├── ability_manager_stub.h
│   ├── ability_manager_client.h
│   ├── ability_connect_manager.h
│   ├── ability_event_handler.h
│   ├── lifecycle_deal.h
│   ├── mission_list_manager.h
│   ├── pending_want_manager.h
│   └── ...
├── src/                 # 实现文件
│   ├── ability_manager_service.cpp   # 主服务实现
│   ├── ability_manager_stub.cpp      # IPC Stub
│   ├── ability_manager_client.cpp     # 客户端
│   ├── ability_connect_manager.cpp    # 连接管理
│   ├── ability_record.cpp             # Ability 记录
│   ├── lifecycle_deal.cpp             # 生命周期处理
│   ├── mission/                      # 任务管理
│   ├── start_ability_handler/         # 启动处理
│   ├── interceptor/                   # 拦截器
│   ├── scene_board/                  # 场景板
│   ├── ui_extension/                 # UI 扩展管理
│   └── ...
├── etc/                  # 配置文件
└── BUILD.gn
```

### services/appmgr/

应用管理服务实现。

```
services/appmgr/
├── include/
│   ├── app_mgr_service.h
│   ├── app_mgr_service_inner.h
│   ├── app_running_record.h
│   ├── module_running_record.h
│   └── ...
├── src/
│   ├── app_mgr_service.cpp
│   ├── app_mgr_service_inner.cpp
│   ├── app_running_record.cpp
│   └── ...
└── BUILD.gn
```

## GN 构建文件清单

主要 GN 配置文件：

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口 |
| `ability_runtime.gni` | 全局配置和路径定义 |
| `services/BUILD.gn` | 服务层构建配置 |
| `frameworks/native/BUILD.gn` | Native 框架构建配置 |
| `frameworks/js/napi/BUILD.gn` | N-API 构建配置 |
| `frameworks/ets/ani/BUILD.gn` | ANI 构建配置 |
| `interfaces/inner_api/BUILD.gn` | Inner API 构建配置 |

## 相关文档

- [项目定位](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [GN Targets](06_GN_Targets.md)
