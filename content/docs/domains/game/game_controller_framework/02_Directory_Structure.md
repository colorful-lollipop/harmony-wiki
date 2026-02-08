# 02_Directory_Structure - 目录结构与模块职责

## 目的

本文档详细说明 GameController Framework 的目录结构和各模块职责。

## 适用范围

- 所有开发人员
- 需要理解代码组织的架构师

## 完整目录树（非测试）

```
domains/game/game_controller_framework/
├── frameworks/                 # 框架层代码
│   ├── native/              # Native 实现
│   │   ├── bundle_info/       # Bundle 信息查询
│   │   │   ├── include/
│   │   │   │   └── bundle_manager.h
│   │   │   └── src/
│   │   │       └── bundle_manager.cpp
│   │   ├── common/            # 通用工具和定义
│   │   │   ├── include/
│   │   │   │   ├── gamecontroller_client_model.h
│   │   │   │   ├── gamecontroller_constants.h
│   │   │   │   ├── gamecontroller_errors.h
│   │   │   │   ├── gamecontroller_keymapping_model.h
│   │   │   │   ├── gamecontroller_log.h
│   │   │   │   └── gamecontroller_utils.h
│   │   │   └── src/
│   │   │       ├── gamecontroller_keymapping_model.cpp
│   │   │       └── gamecontroller_utils.cpp
│   │   ├── sa_client/          # System Ability 客户端
│   │   │   ├── include/
│   │   │   │   ├── gamecontroller_server_client.h          # InnerAPI 接口定义
│   │   │   │   └── gamecontroller_server_client_proxy.h  # IPC Proxy
│   │   │   └── src/
│   │   │       ├── gamecontroller_server_client.cpp
│   │   │       └── gamecontroller_server_client_proxy.cpp
│   │   ├── key_mapping/        # 按键映射实现
│   │   │   ├── include/
│   │   │   │   ├── key_to_touch_handler.h              # 按键转触控基类
│   │   │   │   ├── key_to_touch_manager.h              # 管理器
│   │   │   │   ├── input_to_touch_client.h              # 转触控客户端
│   │   │   │   ├── key_mapping_handle.h                # 处理句柄
│   │   │   │   ├── key_mapping_service.h                # 映射服务
│   │   │   │   ├── single_key_to_touch_handler.h        # 单按键处理
│   │   │   │   ├── combination_key_to_touch_handler.h   # 组合键处理
│   │   │   │   ├── skill_key_to_touch_handler.h          # 技能键处理
│   │   │   │   ├── observation_key_to_touch_handler.h     # 观察键处理
│   │   │   │   ├── keyboard_observation_to_touch_handler.h # 键盘观察处理
│   │   │   │   ├── mouse_observation_to_touch_handler.h    # 鼠标观察处理
│   │   │   │   ├── mouse_left_fire_to_touch_handler.h      # 鼠标左键开火
│   │   │   │   ├── mouse_right_key_walking_to_touch_handler.h # 鼠标右键行走
│   │   │   │   ├── mouse_right_key_click_to_touch_handler.h   # 鼠标右键点击
│   │   │   │   ├── dpad_key_to_touch_handler.h            # 十字键处理
│   │   │   │   └── crosshair_key_to_touch_handler.h        # 准心键处理
│   │   │   └── src/ (对应实现文件)
│   │   ├── multi_modal_input/  # 多模态输入对接
│   │   │   ├── include/
│   │   │   │   ├── game_device_client.h              # 设备客户端
│   │   │   │   ├── multi_modal_input_mgt_service.h   # 多模态服务
│   │   │   │   ├── multi_modal_input_monitor.h        # 输入监听器
│   │   │   │   ├── device_event_callback.h           # 设备事件回调
│   │   │   │   ├── input_device_listener.h           # 输入设备监听器
│   │   │   │   ├── device_info_service.h            # 设备信息服务
│   │   │   │   └── device_identify_service.h       # 设备识别服务
│   │   │   └── src/ (对应实现文件)
│   │   ├── window/            # 窗口框架对接
│   │   │   ├── include/
│   │   │   │   ├── input_event_client.h              # 输入事件客户端
│   │   │   │   ├── input_event_callback.h             # 输入事件回调
│   │   │   │   ├── window_input_intercept.h          # 窗口输入拦截
│   │   │   │   ├── window_info_manager.h             # 窗口信息管理
│   │   │   │   └── window_opr_handle.h               # 窗口操作句柄
│   │   │   └── src/ (对应实现文件)
│   │   ├── plugin/           # 插件管理
│   │   │   ├── include/
│   │   │   │   ├── plugin_client.h                 # 插件客户端
│   │   │   │   ├── plugin_manager.h                # 插件管理器
│   │   │   │   ├── plugin_event_callback.h          # 插件事件回调
│   │   │   │   └── plugin_callback_manager.h         # 插件回调管理
│   │   │   └── src/ (对应实现文件)
│   │   ├── event/            # 事件处理
│   │   │   ├── include/
│   │   │   │   ├── public_event_listener.h           # 公共事件监听
│   │   │   │   └── gamecontroller_event_log.h
│   │   │   └── src/
│   │   │       ├── entryModule.cpp                   # 事件模块入口
│   │   │       └── public_event_listener.cpp
│   │   ├── capi/             # CAPI 实现层
│   │   │   ├── include/
│   │   │   │   ├── game_device_proxy.h
│   │   │   │   ├── game_device_event_proxy.h
│   │   │   │   ├── game_pad_proxy.h
│   │   │   │   └── game_pad_event_proxy.h
│   │   │   └── src/ (对应实现文件)
│   │   └── BUILD.gn         # 框架层构建配置
├── service/                    # 服务层代码
│   ├── common/             # 服务层公共方法
│   │   ├── include/
│   │   │   ├── json_utils.h                  # JSON 工具
│   │   │   └── permission_utils.h            # 权限工具
│   │   └── src/ (对应实现文件)
│   ├── device_manager/     # 设备管理
│   │   ├── include/
│   │   │   └── device_manager.h
│   │   └── src/
│   │       └── device_manager.cpp
│   ├── event/              # 事件处理能力
│   │   ├── include/
│   │   │   └── event_publisher.h
│   │   └── src/
│   │       └── event_publisher.cpp
│   ├── ipc/                # IPC 接口实现
│   │   ├── include/
│   │   │   ├── gamecontroller_server_ability.h   # SA 实现
│   │   │   └── ability_event_handler.h          # SA 事件处理器
│   │   └── src/
│   │       ├── gamecontroller_server_ability.cpp
│   │       └── ability_event_handler.cpp
│   ├── key_mapping_manager/ # 按键映射配置管理
│   │   ├── include/
│   │   │   ├── key_mapping_config_manager.h       # 配置管理器
│   │   │   └── game_support_key_mapping_manager.h   # 支持映射管理器
│   │   └── src/ (对应实现文件)
│   └── BUILD.gn            # 服务层构建配置
├── interfaces/                 # 接口定义
│   └── kits/              # 对外接口存放目录
│       └── c/           # C API 接口定义
│           ├── game_device.h               # 设备管理 API
│           ├── game_device_event.h         # 设备事件定义
│           ├── game_pad.h                # 手柄输入 API
│           ├── game_pad_event.h           # 手柄事件定义
│           ├── game_controller_type.h      # 通用类型定义
│           └── BUILD.gn
├── sa_profile/                 # System Ability 配置
│   ├── 8450.json            # SA 配置（SA ID）
│   └── BUILD.gn
├── etc/                       # 配置文件
│   ├── config/              # 业务配置
│   │   ├── device_config.json              # 设备配置
│   │   ├── game_support_key_mapping.json   # 支持转触控游戏列表
│   │   ├── default_key_mapping.json       # 默认按键映射
│   │   └── custom_key_mapping.json        # 自定义按键映射
│   ├── init/                # 启动配置
│   │   └── gamecontroller_server.cfg
│   ├── config.json          # 构建配置
│   └── BUILD.gn
├── figures/                    # 图表资源
│   ├── system_arch.PNG              # 系统架构图
│   └── code_arch.PNG                # 代码架构图
├── test/                      # 测试代码（忽略）
│   ├── unittest/
│   ├── fuzztest/
│   └── mock/
├── wiki/                      # Wiki 文档
├── README.md                  # 英文说明
├── README_zh.md               # 中文说明
├── LICENSE                    # Apache 2.0 许可证
├── bundle.json                # 包配置
├── game_controller_framework.gni  # GN 变量定义
└── BUILD.gn                   # 根构建配置
```

## 模块职责详解

### frameworks/ - 框架层

#### native/common - 通用模块

**职责**: 提供通用工具、数据模型和错误定义

**证据**:
- `frameworks/native/common/include/gamecontroller_client_model.h`: 客户端数据模型
- `frameworks/native/common/include/gamecontroller_keymapping_model.h`: 按键映射数据模型
- `frameworks/native/common/include/gamecontroller_errors.h`: 错误码定义
- `frameworks/native/common/include/gamecontroller_constants.h`: 常量定义（如 GAME_CONTROLLER_SA_ID = 8450）

**编译产物**: 打包到 libgamecontroller_client.z.so

#### native/sa_client - SA 客户端

**职责**: 提供 InnerAPI 接口，与 GameControllerSA 进行 IPC 通信

**关键类**:
- `GameControllerServerClient`: InnerAPI 客户端（单例模式）
- `GameControllerServerClientProxy`: IPC Proxy

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:25`: DelayedSingleton 单例
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:34-99`: InnerAPI 接口定义

**编译产物**: 打包到 libgamecontroller_client.z.so

#### native/key_mapping - 按键映射实现

**职责**: 实现输入转触控的各种处理器

**处理器类型**:
- `SingleKeyToTouchHandler`: 单按键转触控
- `CombinationKeyToTouchHandler`: 组合键转触控
- `SkillKeyToTouchHandler`: 技能键转触控
- `ObservationKeyToTouchHandler`: 观察键转触控
- `KeyboardObservationToTouchHandler`: 键盘观察处理
- `MouseObservationToTouchHandler`: 鼠标观察处理
- `MouseLeftFireToTouchHandler`: 鼠标左键开火
- `MouseRightKeyWalkingToTouchHandler`: 鼠标右键行走
- `MouseRightKeyClickToTouchHandler`: 鼠标右键点击
- `DpadKeyToTouchHandler`: 十字键处理
- `CrosshairKeyToTouchHandler`: 准心键处理

**管理器**:
- `KeyToTouchManager`: 统一管理所有转触控处理器
- `KeyMappingService`: 映射服务实现
- `InputToTouchClient`: 转触控客户端

**证据**:
- `frameworks/native/key_mapping/include/key_to_touch_manager.h`: 管理器定义
- `frameworks/native/key_mapping/src/key_mapping_service.cpp`: 映射服务实现

**编译产物**: 打包到 libgamecontroller_fwk_client.z.so

#### native/multi_modal_input - 多模态输入对接

**职责**: 对接 MultiModalInput 服务，实现设备监听和识别

**关键类**:
- `MultiModalInputMonitor`: 多模态输入监听器
- `GameDeviceClient`: 设备客户端
- `DeviceEventListener`: 设备事件监听
- `InputDeviceListener`: 输入设备监听器
- `DeviceInfoService`: 设备信息服务
- `DeviceIdentifyService`: 设备识别服务
- `MultiModalInputMgtService`: 多模态管理服务

**证据**:
- `README_zh.md:95-99`: DeviceMonitor 对接多模输入，实现设备上线下线监听

**编译产物**: 打包到 libgamecontroller_fwk_client.z.so

#### native/window - 窗口框架对接

**职责**: 对接 Window Framework，实现输入事件拦截

**关键类**:
- `WindowInputIntercept`: 窗口输入拦截
- `InputEventClient`: 输入事件客户端
- `InputEventCallback`: 输入事件回调
- `WindowInfoManager`: 窗口信息管理
- `WindowOprHandle`: 窗口操作句柄

**证据**:
- `README_zh.md:88-94`: InputMonitor 对接窗口 Framework，向窗口注册需要拦截监听的输入事件

**编译产物**: 打包到 libgamecontroller_fwk_client.z.so

#### native/plugin - 插件管理

**职责**: 管理插件机制（用于扩展）

**关键类**:
- `PluginManager`: 插件管理器
- `PluginClient`: 插件客户端
- `PluginEventCallback`: 插件事件回调
- `PluginCallbackManager`: 插件回调管理

**编译产物**: 打包到 libgamecontroller_fwk_client.z.so

#### native/bundle_info - Bundle 信息

**职责**: 查询应用 Bundle 信息

**关键类**:
- `BundleManager`: Bundle 管理器

**编译产物**: 打包到 libgamecontroller_fwk_client.z.so

#### native/event - 事件处理

**职责**: 处理公共事件

**关键类**:
- `PublicEventListener`: 公共事件监听器

**证据**:
- `frameworks/native/event/src/entryModule.cpp`: 事件模块入口

**编译产物**: 打包到 libgamecontroller_event.z.so

#### native/capi - CAPI 实现

**职责**: 实现 C Native API，将 C++ 框架能力暴露给 C 接口

**Proxy 类**:
- `GameDeviceProxy`: 设备 API 代理
- `GameDeviceEventProxy`: 设备事件代理
- `GamePadProxy`: 手柄 API 代理
- `GamePadEventProxy`: 手柄事件代理

**编译产物**: 打包到 libohgame_controller.z.so

### service/ - 服务层

#### service/common - 公共方法

**职责**: 提供服务层通用工具

**关键类**:
- `JsonUtils`: JSON 文件读写工具
- `PermissionUtils`: 权限检查工具

**证据**:
- `service/common/include/json_utils.h`: JSON 工具定义
- `service/common/include/permission_utils.h`: 权限工具定义

**编译产物**: 打包到 libgamecontroller_server.z.so

#### service/device_manager - 设备管理

**职责**: SA 端的设备管理，实现设备识别

**关键类**:
- `DeviceManager`: 设备管理器

**证据**:
- `README_zh.md:63-65`: DeviceManager 对游戏外设进行设备类别识别

**编译产物**: 打包到 libgamecontroller_server.z.so

#### service/key_mapping_manager - 配置管理

**职责**: 管理按键映射配置文件

**关键类**:
- `KeyMappingConfigManager`: 配置管理器
- `GameSupportKeyMappingManager`: 支持映射管理器

**证据**:
- `README_zh.md:13-17`: 保存配置到 JSON 文件

**编译产物**: 打包到 libgamecontroller_server.z.so

#### service/ipc - IPC 接口实现

**职责**: 实现 System Ability 和 IPC 处理

**关键类**:
- `GameControllerServerAbility`: SA 实现类
- `AbilityEventHandler`: SA 事件处理器

**证据**:
- `service/ipc/include/gamecontroller_server_ability.h:26`: 继承 SystemAbility 和 GameControllerServerInterfaceStub
- `sa_profile/8450.json:5`: SA ID = 8450

**编译产物**: 打包到 libgamecontroller_server.z.so

#### service/event - 事件发布

**职责**: 发布事件到系统

**关键类**:
- `EventPublisher`: 事件发布器

**编译产物**: 打包到 libgamecontroller_server.z.so

### interfaces/ - 接口定义

#### kits/c - C API 接口

**职责**: 定义对外 C Native API 接口

**文件**:
- `game_device.h`: 设备管理 API（OH_GameDevice_*）
- `game_device_event.h`: 设备事件类型定义
- `game_pad.h`: 手柄输入 API（OH_GamePad_*）
- `game_pad_event.h`: 手柄事件类型定义
- `game_controller_type.h`: 通用类型定义（如 GameController_ErrorCode）

**证据**:
- `interfaces/kits/c/game_device.h:58`: OH_GameDevice_GetAllDeviceInfos 接口
- `interfaces/kits/c/game_pad.h:52`: OH_GamePad_LeftShoulder_RegisterButtonInputMonitor 接口

**编译产物**: libohgame_controller.z.so（NDK 库）

### sa_profile/ - SA 配置

**职责**: 定义 System Ability 配置

**文件**:
- `8450.json`: SA 配置文件
  - 进程名: gamecontroller_server
  - SA ID: 8450
  - 库路径: libgamecontroller_server.z.so
  - run-on-create: false（按需启动）
  - start-on-demand: true
  - recycle-strategy: low-memory

**证据**:
- `sa_profile/8450.json:2-15`: 完整配置

### etc/ - 配置文件

#### etc/config - 业务配置

**职责**: 存储业务配置数据

**文件**:
- `device_config.json`: 设备识别配置
- `game_support_key_mapping.json`: 支持转触控的游戏列表
- `default_key_mapping.json`: 默认按键映射规则
- `custom_key_mapping.json`: 自定义按键映射规则

**证据**:
- `README_zh.md:13-17`: 配置文件说明

#### etc/init - 启动配置

**职责**: 定义 SA 启动配置

**文件**:
- `gamecontroller_server.cfg`: SA 启动配置

## 模块依赖关系

### 编译依赖（GN 层面）

```
libohgame_controller.z.so
  ├── depends: libgamecontroller_client.z.so
  └── depends: libgamecontroller_fwk_client.z.so

libgamecontroller_event.z.so
  └── depends: libgamecontroller_fwk_client.z.so

libgamecontroller_fwk_client.z.so
  └── depends: libgamecontroller_client.z.so

libgamecontroller_client.z.so
  └── depends: IPC 生成的代码（IGameControllerServerInterface）

libgamecontroller_server.z.so
  └── depends: libgamecontroller_client.z.so
```

**证据**:
- `interfaces/kits/c/BUILD.gn:54-57`: libohgame_controller.z.so 依赖
- `frameworks/native/BUILD.gn:218`: libgamecontroller_event.z.so 依赖
- `frameworks/native/BUILD.gn:171`: libgamecontroller_fwk_client.z.so 依赖
- `service/BUILD.gn:53`: libgamecontroller_server.z.so 依赖

### 运行时依赖（模块层面）

```
应用进程
  ├── libohgame_controller.z.so (CAPI)
  ├── libgamecontroller_event.z.so (事件监听)
  ├── libgamecontroller_fwk_client.z.so (框架客户端)
  └── libgamecontroller_client.z.so (InnerAPI 客户端)
       └── IPC → GameControllerSA

GameControllerSA 进程
  ├── libgamecontroller_server.z.so (SA 实现)
  └── libgamecontroller_client.z.so (共享数据模型)
```

## 关键结论

1. **两层架构**: Frameworks（客户端）+ Service（SA）
2. **三个主要库**: libohgame_controller.z.so (CAPI), libgamecontroller_client.z.so (InnerAPI), libgamecontroller_event.z.so (事件)
3. **SA 库**: libgamecontroller_server.z.so（独立进程）
4. **配置驱动**: JSON 文件存储配置
5. **SA ID**: 8450

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [01_Project_Position.md](./01_Project_Position.md) - 项目定位
- [03_Architecture.md](./03_Architecture.md) - 架构图

---

**版本**: 1.0 | **更新时间**: 2026-02-06
