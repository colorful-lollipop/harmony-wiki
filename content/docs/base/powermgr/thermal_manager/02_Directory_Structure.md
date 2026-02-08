# 目录结构与模块职责

## 目的

本文档详细说明 thermal_manager 项目的目录结构和各模块的职责，帮助开发者理解代码组织。

## 适用范围

- OpenHarmony thermal_manager 模块
- 非测试代码（test/ 目录除外）

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Module_Boundaries.md](01_Module_Boundaries.md) - 模块边界

---

## 完整目录树

```
/Volumes/lexar/code/d/work/oh/base/powermgr/thermal_manager
│
├── application/                    # Native 应用
│   ├── init/                     # Init 配置
│   │   └── init.cfg
│   │
│   └── protector/                # Thermal Protector（非运行态热管理）
│       ├── include/
│       │   ├── action/          # 动作定义
│       │   │   ├── cpu_action.h
│       │   │   ├── current_action.h
│       │   │   ├── device_control.h
│       │   │   ├── device_control_factory.h
│       │   │   ├── thermal_device_control.h
│       │   │   ├── voltage_action.h
│       │   │   └── ithermal_action.h
│       │   ├── policy/          # 策略定义
│       │   │   ├── protector_base_info.h
│       │   │   ├── protector_thermal_zone_info.h
│       │   │   ├── thermal_kernel_config_file.h
│       │   │   └── thermal_kernel_policy.h
│       │   ├── thermal_protector_timer.h
│       │   ├── thermal_protector_util.h
│       │   ├── thermal_sensor_provider.h
│       │   └── thermal_sensor_provision.h
│       ├── profile/              # 配置文件
│       │   └── thermal_service_config.xml
│       └── src/                 # 实现源码
│           ├── action/
│           │   ├── cpu_action.cpp
│           │   ├── current_action.cpp
│           │   ├── device_control.cpp
│           │   ├── device_control_factory.cpp
│           │   ├── thermal_device_control.cpp
│           │   ├── voltage_action.cpp
│           │   └── ...
│           ├── policy/
│           │   ├── protector_thermal_zone_info.cpp
│           │   ├── thermal_kernel_config_file.cpp
│           │   ├── thermal_kernel_policy.cpp
│           │   └── ...
│           ├── main.cpp
│           ├── thermal_kernel_service.cpp
│           ├── thermal_protector_timer.cpp
│           ├── thermal_protector_utils.cpp
│           ├── thermal_sensor_provision.cpp
│           └── thermal_sensor_provider.cpp
│
├── figures/                       # 架构图
│   └── thermal_manager_architecture.png
│
├── frameworks/                    # Framework 层
│   ├── cj/                       # CJ 语言绑定
│   │   ├── cj_thermal_manager.h
│   │   └── ...
│   │
│   ├── ets/                      # ETS (ArkTS) 语言绑定
│   │   └── taihe/
│   │       ├── thermal/
│   │       │   ├── idl/
│   │       │   ├── include/
│   │       │   ├── src/
│   │       │   └── test/
│   │       └── BUILD.gn
│   │
│   ├── native/                    # Native 层框架
│   │   └── BUILD.gn
│   │
│   └── napi/                     # N-API 层（JS API）
│       ├── BUILD.gn
│       ├── napi_errors.cpp          # 错误处理
│       ├── napi_errors.h
│       ├── napi_utils.cpp          # N-API 工具函数
│       ├── napi_utils.h
│       ├── thermal_manager_napi.cpp  # N-API 主实现
│       └── thermal_manager_napi.h
│
├── interfaces/                    # API 层
│   └── inner_api/               # 内部 API
│       ├── BUILD.gn
│       └── native/include/
│           ├── ithermal_action_callback.h      # 动作回调接口
│           ├── ithermal_level_callback.h      # 热级别回调接口
│           ├── ithermal_temp_callback.h       # 温度回调接口
│           ├── thermal_level_callback_stub.h  # 回调 stub
│           ├── thermal_mgr_client.h           # 客户端 API
│           ├── thermal_level_info.h           # 热级别定义
│           └── thermal_srv_sensor_info.h     # 传感器信息定义
│
├── sa_profile/                    # SA profile（System Ability 配置）
│   ├── 3303.json                   # SA 3303 配置
│   └── BUILD.gn
│
├── services/                     # Thermal Service 代码
│   ├── native/                    # Native 层实现
│   │   ├── include/
│   │   │   ├── thermal_action/
│   │   │   │   ├── action/              # 具体动作实现
│   │   │   │   │   ├── action_soc/    # SOC 动作
│   │   │   │   │   │   ├── action_cpu_big.cpp
│   │   │   │   │   │   ├── action_cpu_boost.cpp
│   │   │   │   │   │   ├── action_cpu_isolate.cpp
│   │   │   │   │   │   ├── action_cpu_lit.cpp
│   │   │   │   │   │   ├── action_cpu_med.cpp
│   │   │   │   │   │   ├── action_cpu_nonvip.cpp
│   │   │   │   │   │   ├── action_gpu.cpp
│   │   │   │   │   │   ├── action_socperf.cpp
│   │   │   │   │   │   ├── action_socperf_resource.cpp
│   │   │   │   │   │   └── soc_action_base.cpp
│   │   │   │   ├── action_airplane.cpp       # 飞行模式
│   │   │   │   ├── action_application_process.cpp  # 应用进程限制
│   │   │   │   ├── action_charger.cpp         # 充电控制
│   │   │   │   ├── action_display.cpp        # 显示控制
│   │   │   │   ├── action_node.cpp            # sysfs 节点控制
│   │   │   │   ├── action_popup.cpp          # 弹窗警告
│   │   │   │   ├── action_shutdown.cpp        # 关机动作
│   │   │   │   ├── action_thermal_level.cpp   # 热级别设置
│   │   │   │   ├── action_voltage.cpp        # 电压控制
│   │   │   │   ├── action_volume.cpp         # 音量控制
│   │   │   │   └── ...
│   │   │   ├── action_popup.h
│   │   │   ├── action_shutdown.h
│   │   │   ├── action_thermal_level.h
│   │   │   ├── action_voltage.h
│   │   │   ├── action_volume.h
│   │   │   ├── ithermal_action.h            # 动作接口
│   │   │   └── thermal_timer.h
│   │   │   ├── thermal_action_factory.cpp  # 动作工厂
│   │   │   ├── thermal_action_manager.cpp   # 动作管理器
│   │   │   └── thermal_timer.cpp
│   │   ├── thermal_action/
│   │   │   └── ithermal_action.h
│   │   ├── thermal_observer/
│   │   │   ├── state_machine/       # 状态机
│   │   │   │   ├── charge_delay_state_collection.cpp
│   │   │   │   ├── charger_state_collection.cpp
│   │   │   │   ├── cust_state_collection.cpp
│   │   │   │   ├── extend_state_collection.cpp
│   │   │   │   ├── scene_state_collection.cpp
│   │   │   │   ├── screen_state_collection.cpp
│   │   │   │   ├── startup_delay_state_collection.cpp
│   │   │   │   ├── state_collection_factory.cpp
│   │   │   │   └── state_machine.cpp
│   │   │   ├── thermal_common_event_receiver.cpp  # 公共事件接收
│   │   │   ├── thermal_observer.cpp             # 观察者主类
│   │   │   ├── thermal_sensor_info.cpp
│   │   │   └── thermal_service_subscriber.cpp
│   │   ├── thermal_policy/
│   │   │   ├── fan_fault_detect.cpp             # 风扇故障检测
│   │   │   ├── thermal_config_base_info.cpp
│   │   │   ├── thermal_config_sensor_cluster.cpp
│   │   │   ├── thermal_policy.cpp                # 策略主类
│   │   │   └── thermal_srv_config_parser.cpp  # 配置解析器
│   │   ├── thermal_srv_sensor_info.h
│   │   ├── action_popup.h
│   │   ├── fan_callback.h
│   │   ├── fan_fault_detect.h
│   │   ├── hdi_service_status_listener.h
│   │   ├── ithermal_level_callback.h
│   │   ├── ithermal_temp_callback.h
│   │   ├── state_machine.h
│   │   ├── thermal_action_manager.h
│   │   ├── thermal_callback.h
│   │   ├── thermal_config_base_info.h
│   │   ├── thermal_config_sensor_cluster.h
│   │   ├── thermal_observer.h
│   │   ├── thermal_policy.h
│   │   ├── thermal_sensor_info.h
│   │   ├── thermal_service_subscriber.h
│   │   ├── thermal_srv_config_parser.h
│   │   ├── thermal_srv_sensor_info.h
│   │   ├── thermal_srv_stub.h
│   │   ├── v1_1/ithermal_interface.h
│   │   └── v1_1/thermal_types.h
│   │   ├── profile/                  # 配置文件
│   │   │   └── thermal_service_config.xml
│   │   ├── thermal_callback.cpp          # HDI 回调
│   │   ├── thermal_mgr_dumper.cpp      # Dump 功能
│   │   └── thermal_service.cpp          # Service 主类
│   │   └── BUILD.gn
│   └── zidl/                         # ZIDL 接口定义
│       ├── include/
│       │   ├── thermal_action_callback_proxy.h
│       │   ├── thermal_action_callback_stub.h
│       │   ├── thermal_level_callback_proxy.h
│       │   ├── thermal_level_callback_stub.h
│       │   ├── thermal_temp_callback_proxy.h
│       │   └── thermal_temp_callback_stub.h
│       └── src/
│           ├── thermal_action_callback_proxy.cpp
│           ├── thermal_action_callback_stub.cpp
│           ├── thermal_level_callback_proxy.cpp
│           ├── thermal_level_callback_stub.cpp
│           ├── thermal_temp_callback_proxy.cpp
│           └── thermal_temp_callback_stub.cpp
│       └── BUILD.gn
│
├── utils/                         # 工具库
│   ├── appmgr/                    # 应用管理工具
│   │   ├── include/
│   │   │   └── thermal_ability_client.h
│   │   ├── src/
│   │   │   └── thermal_ability_client.cpp
│   │   └── BUILD.gn
│   ├── hookmgr/                   # Hook 管理器
│   │   ├── include/
│   │   │   └── thermal_hookmgr.h
│   │   ├── src/
│   │   │   └── thermal_hookmgr.cpp
│   │   └── BUILD.gn
│   ├── native/                    # Native 工具库
│   │   ├── include/
│   │   │   ├── file_operation.h
│   │   │   ├── string_operation.h
│   │   │   ├── thermal_base_info.h
│   │   │   └── thermal_config.h
│   │   ├── src/
│   │   │   ├── file_operation.cpp
│   │   │   ├── string_operation.cpp
│   │   │   ├── thermal_common.cpp
│   │   │   ├── thermal_config.cpp
│   │   │   └── thermal_log.cpp
│   │   └── BUILD.gn
│   └── BUILD.gn
│
├── thermalmgr.gni                 # GN 全局配置
├── bundle.json                   # Bundle 元数据
├── README.md                     # 项目说明文档
└── LICENSE                       # Apache 2.0 许可证
```

---

## 模块职责

### 1. N-API 层 (frameworks/napi/)

**职责**: 提供给 JavaScript 应用调用的 API 接口

**主要组件**:
- `thermal_manager_napi.cpp` - N-API 主实现
- `ThermalLevelCallback` - 热级别回调类
- `napi_utils.cpp` - N-API 工具函数
- `napi_errors.cpp` - 错误处理

**证据**: `frameworks/napi/thermal_manager_napi.cpp:16-297`

---

### 2. 内部 API 层 (interfaces/inner_api/)

**职责**: 提供内部 C/C++ API 供其他 Native 组件使用

**主要组件**:
- `ThermalMgrClient` - 客户端单例，连接到 Thermal Service
- 回调接口定义:
  - `IThermalLevelCallback` - 热级别变化回调
  - `IThermalTempCallback` - 温度变化回调
  - `IThermalActionCallback` - 动作执行回调

**证据**: `interfaces/inner_api/native/include/thermal_mgr_client.h:26`

---

### 3. Thermal Service (services/native/)

**职责**: 实现核心热管理服务，作为 System Ability (SA 3303) 运行

**主要子模块**:

#### 3.1 ThermalObserver
**职责**: 观察温度传感器数据，通知订阅者

**证据**: `services/native/include/thermal_observer/thermal_observer.h:34`

#### 3.2 ThermalPolicy
**职责**: 基于传感器温度和设备状态决策热级别和策略

**证据**: `services/native/include/thermal_policy/thermal_policy.h:40`

#### 3.3 ThermalActionManager
**职责**: 管理和执行所有热动作（CPU、GPU、电压等）

**证据**: `services/native/include/thermal_action/thermal_action_manager.h:38`

#### 3.4 StateMachine
**职责**: 管理设备状态（屏幕、充电、场景）

**证据**: `services/native/include/thermal_observer/state_machine/`

#### 3.5 ThermalConfigParser
**职责**: 解析 XML 配置文件

**证据**: `services/native/src/thermal_policy/thermal_srv_config_parser.cpp`

#### 3.6 ThermalService 主类
**职责**: SA 入口，协调各模块，提供 IPC 接口

**证据**: `services/native/include/thermal_service.h:56`

---

### 4. ZIDL 层 (services/zidl/)

**职责**: 定义 IPC 接口的 Proxy 和 Stub

**主要组件**:
- 回调 Stub（服务端）:
  - `ThermalTempCallbackStub`
  - `ThermalLevelCallbackStub`
  - `ThermalActionCallbackStub`
- 回调 Proxy（客户端）:
  - `ThermalTempCallbackProxy`
  - `ThermalLevelCallbackProxy`
  - `ThermalActionCallbackProxy`

**证据**: `services/zidl/include/` 目录

---

### 5. 工具库 (utils/)

#### 5.1 Native 工具 (utils/native/)
**职责**: 提供通用工具函数

**主要组件**:
- `FileOperation` - 文件操作（读/写）
- `StringOperation` - 字符串处理
- `ThermalCommon` - 通用热管理常量

**证据**: `utils/native/src/` 目录

#### 5.2 AppMgr (utils/appmgr/)
**职责**: 应用管理相关工具

**证据**: `utils/appmgr/` 目录

#### 5.3 HookMgr (utils/hookmgr/)
**职责**: Hook 管理器，用于配置解密等

**证据**: `utils/hookmgr/src/thermal_hookmgr.cpp`

---

### 6. Thermal Protector (application/protector/)

**职责**: 非运行态（关机/重启）下的简化热管理

**主要组件**:
- `main.cpp` - Protector 主程序
- `ThermalKernelService` - 内核服务
- 动作实现（CPU、电压、电流等）
- 策略实现

**证据**: `application/protector/` 目录

---

### 7. SA Profile (sa_profile/)

**职责**: 定义 System Ability 配置

**主要文件**:
- `3303.json` - SA 3303 配置

**证据**: `sa_profile/3303.json`

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────┐
│              Applications / Other Modules          │
└───────────────┬───────────────────────────────┘
                │
                ↓
┌───────────────────────────────────────────────────┐
│        frameworks/napi (N-API Layer)         │
│  ┌───────────────────────────────────────┐   │
│  │ interfaces/inner_api/ (Client)        │   │
│  └───────────────────────────────────────┘   │
└───────────────┬───────────────────────────────┘
                │ (IPC - Binder)
                ↓
┌───────────────────────────────────────────────────┐
│         services/native (Thermal Service)     │
│  ┌─────────┬───────────────────┬─────────┐│
│  │Observer │  Policy         │Actions  ││
│  └─────────┴───────────────────┴─────────┘│
└───────────────┬───────────────────────────────┘
                │ (HDI)
                ↓
┌───────────────────────────────────────────────────┐
│         Thermal Drivers (HDF)                │
└───────────────────────────────────────────────────┘
```

**依赖规则**:
- ✅ N-API 层依赖 inner_api
- ✅ inner_api 通过 IPC 连接 Thermal Service
- ✅ Thermal Service 不依赖 N-API
- ✅ Thermal Service 依赖 utils
- ✅ Thermal Protector 独立运行

---

## 代码统计

| 类型 | 文件数 | 说明 |
|---|---|---|
| N-API 文件 | 6 | .cpp/.h 文件 |
| 内部 API 文件 | 7 | .h 文件 |
| Service 源文件 | ~85 | .cpp 文件（排除测试） |
| Service 头文件 | ~40 | .h 文件 |
| ZIDL 文件 | 12 | .cpp/.h 文件 |
| Protector 文件 | ~25 | .cpp/.h 文件 |
| 工具库文件 | ~15 | .cpp/.h 文件 |

---

## 总结

thermal_manager 采用清晰的分层架构：

1. **接口层** (N-API + Inner API) - 提供标准化接口
2. **服务层** (Thermal Service) - 核心业务逻辑
3. **驱动层** (HDI) - 与硬件通信
4. **工具层** (Utils) - 通用功能

各模块职责明确，依赖关系单向，便于理解和维护。
