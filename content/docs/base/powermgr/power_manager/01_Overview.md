# 项目概览

本文档描述 OpenHarmony `power_manager` 模块的定位、目录结构和核心能力。

## 1. 项目定位

### 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **包名** | `@ohos/power_manager` |
| **子系统** | `powermgr` |
| **组件名** | `power_manager` |
| **系统能力** | `SystemCapability.PowerManager.PowerManager.Core` |
| **bundle.json 版本** | 3.1 |
| **目标系统** | Standard (标准系统) |

### 1.2 核心能力

**电源管理**是 OpenHarmony 系统的核心子系统之一，提供以下能力：

1. **设备电源控制**
   - 关机/重启设备
   - 休眠/唤醒设备
   - 亮灭屏控制

2. **运行锁管理**
   - 后台任务保活锁
   - 屏幕距离感应锁
   - 用户空闲检测锁

3. **电源状态管理**
   - 状态机管理
   - 电源模式切换
   - 屏幕超时设置

4. **事件通知**
   - 状态变化回调
   - 关机/休眠异步通知
   - 错误上报

---

## 2. 目录结构

```
/base/powermgr/power_manager/
├── figures/                          # 架构图资源
│   └── power-management-subsystem-architecture.png
│
├── frameworks/                       # 【框架层】客户端 API
│   ├── native/                       # Native C++ 客户端库
│   │   ├── include/
│   │   │   ├── power_mgr_client.h    # 客户端主接口
│   │   │   ├── running_lock.h        # 运行锁接口
│   │   │   └── shutdown_client.h     # 关机客户端
│   │   └── src/
│   │       ├── power_mgr_client.cpp
│   │       ├── running_lock.cpp
│   │       └── shutdown_client.cpp
│   │
│   ├── napi/                         # JavaScript/TypeScript 绑定
│   │   ├── power/                    # 电源模块
│   │   │   ├── power_module.cpp      # 模块注册
│   │   │   ├── power_napi.cpp        # API 实现
│   │   │   ├── power.cpp             # 旧 API (废弃)
│   │   │   └── power.h
│   │   │
│   │   ├── runninglock/              # 运行锁模块
│   │   │   ├── runninglock_module.cpp # 模块注册
│   │   │   ├── runninglock_napi.cpp  # API 实现
│   │   │   ├── runninglock_interface.cpp # 旧 API (废弃)
│   │   │   └── runninglock_napi.h
│   │   │
│   │   └── utils/                    # N-API 工具类
│   │       ├── async_callback_info.cpp
│   │       ├── napi_errors.cpp
│   │       ├── napi_utils.cpp
│   │       └── napi_utils.h
│   │
│   ├── ets/taihe/                    # ArkTS/Taihe 绑定
│   │   ├── power/                    # 电源模块
│   │   │   ├── power_napi.ts
│   │   │   └── BUILD.gn
│   │   └── runninglock/              # 运行锁模块
│   │       ├── runninglock_napi.ts
│   │       └── BUILD.gn
│   │
│   └── cj/                           # Cangjie FFI 绑定
│       ├── power/                    # 电源模块
│       │   ├── power_ffi.cpp
│       │   └── BUILD.gn
│       └── runninglock/              # 运行锁模块
│           ├── runninglock_ffi.cpp
│           └── BUILD.gn
│
├── interfaces/                       # 【接口层】API 定义
│   └── inner_api/                    # 内部 C++ API
│       ├── native/include/            # 头文件
│       │   ├── power_mgr_client.h    # 客户端主接口
│       │   ├── running_lock.h        # 运行锁接口
│       │   ├── power_state_callback.h # 状态回调
│       │   ├── power_mode_callback.h  # 模式回调
│       │   ├── running_lock_info.h   # 运行锁信息
│       │   ├── power_errors.h        # 错误码
│       │   ├── shutdown/             # 关机相关
│       │   │   ├── shutdown_client.h
│       │   │   ├── isync_shutdown_callback.h
│       │   │   └── shutdown_priority.h
│       │   ├── suspend/               # 挂起相关
│       │   │   └── itake_over_suspend_callback.h
│       │   └── hibernate/             # 休眠相关
│       │       └── isync_hibernate_callback.h
│       │
│       └── BUILD.gn                   # 构建配置
│
├── services/                         # 【服务层】核心实现
│   ├── native/                       # 服务端实现
│   │   ├── include/
│   │   │   ├── power_mgr_service.h   # 服务主类
│   │   │   ├── power_state_machine.h # 状态机
│   │   │   ├── power_mode_module.h   # 电源模式
│   │   │   └── power_mgr_ipc_adapter.h # IPC 适配器
│   │   │
│   │   ├── src/                      # 核心实现
│   │   │   ├── power_mgr_service.cpp # 主服务 (113KB)
│   │   │   ├── power_state_machine.cpp # 状态机 (121KB)
│   │   │   ├── power_mgr_ipc_adapter.cpp
│   │   │   ├── power_mgr_factory.cpp
│   │   │   ├── power_mgr_dumper.cpp
│   │   │   ├── power_mgr_notify.cpp
│   │   │   ├── power_hdi_callback.cpp
│   │   │   └── death_recipient_manager.cpp
│   │   │
│   │   ├── runninglock/              # 运行锁管理
│   │   │   ├── running_lock_mgr.cpp
│   │   │   ├── running_lock_inner.cpp
│   │   │   ├── running_lock_proxy.cpp
│   │   │   └── running_lock_timer_handler.cpp
│   │   │
│   │   ├── suspend/                  # 挂起控制
│   │   │   ├── suspend_controller.cpp
│   │   │   ├── suspend_source_parser.cpp
│   │   │   ├── suspend_sources.cpp
│   │   │   ├── sleep_callback_holder.cpp
│   │   │   └── suspend_takeover_callback_holder.cpp
│   │   │
│   │   ├── wakeup/                   # 唤醒控制
│   │   │   ├── wakeup_controller.cpp
│   │   │   ├── wakeup_source_parser.cpp
│   │   │   └── wakeup_sources.cpp
│   │   │
│   │   ├── shutdown/                 # 关机控制
│   │   │   ├── shutdown_controller.cpp
│   │   │   ├── shutdown_dialog.cpp
│   │   │   └── shutdown_callback_holder.cpp
│   │   │
│   │   ├── hibernate/                # 休眠支持
│   │   │   └── hibernate_controller.cpp
│   │   │
│   │   ├── power_mode/               # 电源模式
│   │   │   ├── power_mode_module.cpp
│   │   │   └── power_mode_policy.cpp
│   │   │
│   │   ├── setting/                  # 设置集成
│   │   │   └── setting_helper.cpp
│   │   │
│   │   ├── proximity_sensor_controller/ # 距离传感器
│   │   │   └── proximity_controller_base.cpp
│   │   │
│   │   ├── screenoffpre/             # 灭屏前处理
│   │   │   └── screen_off_pre_controller.cpp
│   │   │
│   │   ├── wakeup_action/            # 唤醒动作
│   │   │   ├── wakeup_action_controller.cpp
│   │   │   └── wakeup_action_source_parser.cpp
│   │   │
│   │   ├── ulsr/                      # ULSR 插件
│   │   │   └── ulsr_callback_holder.cpp
│   │   │
│   │   ├── actions/                   # 设备动作抽象
│   │   │   ├── default/               # 默认实现
│   │   │   │   ├── suspend/
│   │   │   │   ├── display/
│   │   │   │   └── running_lock_action.cpp
│   │   │   ├── standard/
│   │   │   └── idevice_power_action.h
│   │   │
│   │   ├── screen_common_event/       # 公共事件
│   │   │   └── screen_common_event_controller.cpp
│   │   │
│   │   ├── multi_invoker_helper/      # 多调用者助手
│   │   │   └── multi_invoker_helper.cpp
│   │   │
│   │   └── profile/                   # 配置文件
│   │       ├── power_mode_config.xml
│   │       ├── power_suspend.json
│   │       ├── power_wakeup.json
│   │       └── power_vibrator.json
│   │
│   └── zidl/                         # ZIDL IPC 接口
│       ├── include/                   # IPC 头文件
│       │   ├── power_mgr_async_reply.h
│       │   ├── power_state_callback_proxy.h
│       │   ├── power_state_callback_stub.h
│       │   ├── power_mode_callback_*.h
│       │   ├── power_runninglock_callback_*.h
│       │   ├── shutdown_callback_*.h
│       │   ├── sync_sleep_callback_*.h
│       │   ├── sync_hibernate_callback_*.h
│       │   └── ...
│       │
│       └── src/                       # IPC 实现
│           ├── power_state_callback_proxy.cpp
│           ├── power_state_callback_stub.cpp
│           └── ...
│
├── utils/                            # 【工具层】公共组件
│   ├── native/                       # 公共工具
│   │   ├── include/
│   │   │   ├── power_common.h       # 公共宏
│   │   │   ├── power_log.h          # 日志
│   │   │   ├── power_utils.h        # 工具函数
│   │   │   ├── power_mgr_errors.h   # 错误码
│   │   │   ├── sp_singleton.h       # 单例模板
│   │   │   └── power_xcollie.h      # 看门狗
│   │   └── src/
│   │       ├── power_utils.cpp
│   │       └── power_xcollie.cpp
│   │
│   ├── ffrt/                         # FFRT 异步任务
│   │   ├── include/ffrt_utils.h
│   │   ├── ffrt_utils.cpp
│   │   └── BUILD.gn
│   │
│   ├── hookmgr/                      # 钩子管理器
│   │   ├── include/power_hookmgr.h
│   │   ├── power_hookmgr.cpp
│   │   └── BUILD.gn
│   │
│   ├── vibrator/                    # 振动器
│   │   ├── include/power_vibrator.h
│   │   ├── include/vibrator_source_parser.h
│   │   ├── power_vibrator.cpp
│   │   ├── vibrator_source_parser.cpp
│   │   └── BUILD.gn
│   │
│   ├── permission/                  # 权限检查
│   │   ├── include/permission.h
│   │   ├── permission.cpp
│   │   └── BUILD.gn
│   │
│   ├── param/                       # 系统参数
│   │   ├── include/sysparam.h
│   │   ├── sysparam.cpp
│   │   └── BUILD.gn
│   │
│   ├── setting/                     # 设置项
│   │   ├── include/setting_observer.h
│   │   ├── include/setting_provider.h
│   │   ├── setting_observer.cpp
│   │   ├── setting_provider.cpp
│   │   └── BUILD.gn
│   │
│   ├── appmgr/                      # 应用管理
│   │   ├── include/
│   │   ├── app_manager_utils.cpp
│   │   └── BUILD.gn
│   │
│   ├── ability/                     # 能力集成
│   │   ├── include/
│   │   ├── power_ability.cpp
│   │   └── BUILD.gn
│   │
│   ├── shell/                       # Shell 命令
│   │   ├── main.cpp
│   │   ├── power_shell_command.cpp
│   │   └── BUILD.gn
│   │
│   ├── lib_loader/                  # 动态库加载
│   │   ├── interface_loader.cpp
│   │   ├── library_loader.cpp
│   │   └── BUILD.gn
│   │
│   └── intf_wrapper/                # 接口包装
│       ├── power_ext_intf_wrapper.cpp
│       └── BUILD.gn
│
├── power_dialog/                     # 电源对话框 HAP
│   ├── entry/                        # FA 入口
│   ├── signature/                    # 签名
│   ├── hvigfile
│   └── BUILD.gn
│
├── sa_profile/                      # SA 配置文件
│   └── 3301.json                    # PowerMgr SA ID
│
├── etc/                              # 系统配置
│   ├── init/                         # 初始化脚本
│   │   └── power_manager.cfg
│   └── para/                         # 参数配置
│       ├── powermgr_para
│       └── powermgr_para_dac
│
├── test/                             # 测试代码 (不计入业务逻辑)
│   ├── fuzztest/                    # Fuzz 测试
│   ├── unittest/                    # 单元测试
│   ├── systemtest/                  # 系统测试
│   ├── apitest/                     # API 测试
│   └── mock/                        # Mock 工具
│
├── bundle.json                       # 组件配置
├── powermgr.gni                      # GN 特性配置
├── powermanager.yaml                 # HiSysEvent 配置
├── powermanager_POWER_UE.yaml       # HiSysEvent UE 配置
├── README.md                         # 项目说明
└── README_zh.md                     # 中文说明
```

---

## 3. 模块职责划分

### 3.1 分层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                     │
│   JS/TS 应用    ArkTS 应用    Cangjie 应用    Native 应用     │
└─────────────────────┬───────────────────┬─────────────────────┘
                      │                   │
                      ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      框架层 (Frameworks)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  N-API   │  │ETS/Taihe │  │CJ FFI    │  │ Native   │     │
│  │ (JS/TS) │  │ (ArkTS)  │  │ (Cangjie)│  │ (C++)    │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
└───────┼─────────────┼─────────────┼─────────────┼───────────┘
        │             │             │             │
        └─────────────┴─────────────┴─────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    接口层 (Interfaces/Inner API)                 │
│              PowerMgrClient, RunningLock, Callbacks              │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       服务层 (Services)                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ZIDL (IPC 层)                               │   │
│  │        Proxy/Stub - 基于 IPowerMgr.idl 生成              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Native 服务实现                              │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────┐ │   │
│  │  │PowerMgr│ │ State  │ │Running │ │Suspend │ │Wakeup│ │   │
│  │  │Service │ │Machine │ │Lock    │ │Ctrl   │ │Ctrl  │ │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └──────┘ │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────┐ │   │
│  │  │Shutdown│ │Power   │ │Proximity│ │Hibernate│ │ULSR │ │   │
│  │  │Ctrl    │ │Mode    │ │Sensor  │ │Support │ │Plugin│ │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └──────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     工具层 (Utils)                               │
│  Common│FFRT│Vibrator│Permission│Setting│HookMgr│Shell...     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   HDI/Hardware 层                               │
│              drivers_interface_power, HDF                        │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 各层职责

| 层级 | 职责 | 关键组件 |
|------|------|----------|
| **应用层** | 业务调用 | JS/TS/ArkTS/CJ/Native 应用 |
| **框架层** | API 暴露 | N-API, Taihe, CJ FFI, Native Client |
| **接口层** | 接口定义 | Inner API 头文件, IPC 接口 |
| **服务层** | 业务逻辑 | PowerMgrService, 状态机, 各控制器 |
| **工具层** | 公共能力 | 日志, 权限, FFRT, 设置 |
| **HDI 层** | 硬件交互 | Power HDF 驱动 |

---

## 4. 依赖关系

### 4.1 外部依赖

```json
// bundle.json 依赖组件
"ability_runtime",      // 能力运行时
"access_token",         // 权限管理
"battery_manager",      // 电池管理
"common_event_service", // 公共事件
"display_manager",      // 显示管理
"drivers_interface_power", // Power HDI
"ets_runtime",          // ETS 运行时
"ffrt",                // 异步任务
"hiview",              // 日志系统
"init",                // 初始化
"input",               // 输入管理
"ipc",                 // IPC 框架
"safwk",               // SA 框架
"samgr",               // 服务管理
"sensor",              // 传感器
"device_standby",      // 设备待机
"window_manager",      // 窗口管理
```

### 4.2 模块间依赖

```
frameworks/napi        → interfaces/inner_api
frameworks/native      → interfaces/inner_api
services/native        → interfaces/inner_api, utils/*
services/zidl         → interfaces/inner_api
utils/*               → 无 (基础工具)
```

---

## 5. 相关仓库

根据 `README.md`：

- [powermgr_display_manager](https://gitee.com/openharmony/powermgr_display_manager)
- [powermgr_battery_manager](https://gitee.com/openharmony/powermgr_battery_manager)
- [powermgr_thermal_manager](https://gitee.com/openharmony/powermgr_thermal_manager)
- [powermgr_battery_statistics](https://gitee.com/openharmony/powermgr_battery_statistics)
- [powermgr_battery_lite](https://gitee.com/openharmony/powermgr_battery_lite)
- [powermgr_powermgr_lite](https://gitee.com/openharmony/powermgr_powermgr_lite)

---

## 6. 快速开始

### 6.1 编译

```bash
# 在 OpenHarmony 构建环境中
hb set
hb build -f

# 或单独编译 power_manager
gn gen out
ninja -C out //base/powermgr/power_manager:services
```

### 6.2 运行

SA 自动启动，进程名为 `powermgr`。

### 6.3 调试

```bash
# 查看服务状态
hdc shell sa_ps | grep powermgr

# 查看日志
hdc shell hilog | grep PowerMgr

# Shell 调试工具
hdc shell power-shell
```

---

## 7. 关键文件索引

| 功能 | 文件路径 |
|------|----------|
| 服务主入口 | `services/native/src/power_mgr_service.cpp` |
| 状态机 | `services/native/src/power_state_machine.cpp` |
| N-API 注册 (Power) | `frameworks/napi/power/power_module.cpp` |
| N-API 注册 (RunningLock) | `frameworks/napi/runninglock/runninglock_module.cpp` |
| GN 配置 | `powermgr.gni` |
| 组件配置 | `bundle.json` |
| SA 配置 | `sa_profile/3301.json` |
