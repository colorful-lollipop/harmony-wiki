# 07_Build_Artifacts - 编译产物

## 目的

本文档详细说明 GameController Framework 的编译产物、安装路径和运行时加载关系。

## 适用范围

- 构建工程师
- 部署人员
- 运维人员

## 产物总览

| 产物名称 | 大小估算 | 安装路径 | 用途 | 加载方式 |
|----------|----------|----------|------|----------|
| libohgame_controller.z.so | ~50-100KB | /system/ndk/ | CAPI 库，游戏应用链接使用 |
| libgamecontroller_event.z.so | ~100-200KB | /system/lib/ | 事件监听库，Window Framework dlopen 加载 |
| libgamecontroller_fwk_client.z.so | ~300-500KB | /system/lib/ | 框架客户端库，包含核心业务逻辑 |
| libgamecontroller_client.z.so | ~50-100KB | /system/lib/ | InnerAPI 库，SA 客户端 |
| libgamecontroller_server.z.so | ~200-300KB | /system/lib/ | System Ability 服务库 |
| device_config.json | ~1-5KB | /system/etc/game_controller/game_controller_service/ | 设备识别配置 |
| default_key_mapping.json | ~5-10KB | /system/etc/game_controller/game_controller_service/ | 默认按键映射配置 |
| custom_key_mapping.json | ~5-10KB | /system/etc/game_controller/game_controller_service/ | 自定义按键映射配置 |
| game_support_key_mapping.json | ~1-5KB | /system/etc/game_controller/game_controller_service/ | 支持转触控游戏列表 |
| gamecontroller_server.cfg | ~0.5KB | /system/etc/init/ | SA 启动配置 |
| 8450.json | ~0.5KB | /system/profile/ | SA 配置文件 |

**证据**:
- `README_zh.md:126-129`: 编译产物列表
- `BUILD.gn` 和各子目录的 BUILD.gn 文件: 产物定义和安装路径

## 主要产物详解

### 1. libohgame_controller.z.so (CAPI)

**Target**: `ohgame_controller`

**证据**: `interfaces/kits/c/BUILD.gn:27`

**安装路径**: `/system/ndk/libohgame_controller.z.so`

**安装配置**:
- `relative_install_dir`: `"ndk/"`

**大小**: 约 50-100KB

**包含内容**:
- C API 实现（game_device.cpp, game_pad.cpp, game_device_event.cpp, game_pad_event.cpp）
- C++ Proxy 实现（game_device_proxy.cpp, game_pad_proxy.cpp, etc.）
- 外部依赖：libgamecontroller_client.z.so, libgamecontroller_fwk_client.z.so

**用途**:
- 游戏应用在编译时通过 `-lgame_controller` 链接
- 游戏应用调用 C API 函数（OH_GameDevice_*, OH_GamePad_*）

**加载时机**: 应用启动时由应用链接加载器加载

**证据**:
- `interfaces/kits/c/BUILD.gn:41-50`: sources 列表
- `interfaces/kits/c/BUILD.gn:54-57`: deps 定义

### 2. libgamecontroller_event.z.so (事件监听库）

**Target**: `gamecontroller_event`

**证据**: `frameworks/native/BUILD.gn:196`

**安装路径**: `/system/lib/libgamecontroller_event.z.so`

**大小**: 约 100-200KB

**包含内容**:
- 事件模块入口（entryModule.cpp）
- 公共事件监听器（public_event_listener.cpp）

**外部依赖**: libgamecontroller_fwk_client.z.so

**用途**:
- 处理公共事件
- 由 Window Framework 通过 dlopen 动态加载

**加载时机**: 应用启动时由 Window Framework dlopen

**证据**:
- `README_zh.md:81-84`: Window Framework dlopen 加载
- `frameworks/native/BUILD.gn:214-216`: deps 和 sources

### 3. libgamecontroller_fwk_client.z.so (框架客户端）

**Target**: `gamecontroller_fwk_client`

**证据**: `frameworks/native/BUILD.gn:114`

**安装路径**: `/system/lib/libgamecontroller_fwk_client.z.so`

**大小**: 约 300-500KB

**包含内容** (33 个源文件）:
- **key_mapping/** (15 个文件）: 按键映射处理器
  - combination_key_to_touch_handler.cpp
  - crosshair_key_to_touch_handler.cpp
  - dpad_key_to_touch_handler.cpp
  - input_to_touch_client.cpp
  - key_mapping_handle.cpp
  - key_mapping_service.cpp
  - key_to_touch_handler.cpp
  - key_to_touch_manager.cpp
  - keyboard_observation_to_touch_handler.cpp
  - mouse_left_fire_to_touch_handler.cpp
  - mouse_observation_to_touch_handler.cpp
  - mouse_right_key_click_to_touch_handler.cpp
  - mouse_right_key_walking_to_touch_handler.cpp
  - observation_key_to_touch_handler.cpp
  - single_key_to_touch_handler.cpp
  - skill_key_to_touch_handler.cpp

- **multi_modal_input/** (6 个文件）: 多模态输入
  - device_event_callback.cpp
  - device_identify_service.cpp
  - device_info_service.cpp
  - game_device_client.cpp
  - input_device_listener.cpp
  - multi_modal_input_mgt_service.cpp
  - multi_modal_input_monitor.cpp

- **window/** (5 个文件）: 窗口输入拦截
  - input_event_callback.cpp
  - input_event_client.cpp
  - window_input_intercept.cpp
  - window_info_manager.cpp
  - window_opr_handle.cpp

- **plugin/** (3 个文件）: 插件管理
  - plugin_callback_manager.cpp
  - plugin_client.cpp
  - plugin_manager.cpp

- **bundle_info/** (1 个文件）: Bundle 信息
  - bundle_manager.cpp

**外部依赖**: libgamecontroller_client.z.so

**用途**:
- 实现核心业务逻辑
- 包含设备监听、输入拦截、按键映射、插件管理
- 被 libohgame_controller.z.so 和 libgamecontroller_event.z.so 依赖

**证据**:
- `frameworks/native/BUILD.gn:131-163`: sources 列表（33 个文件）
- `frameworks/native/BUILD.gn:171`: deps 定义

### 4. libgamecontroller_client.z.so (InnerAPI）

**Target**: `gamecontroller_client`

**证据**: `frameworks/native/BUILD.gn:67`

**安装路径**: `/system/lib/libgamecontroller_client.z.so`

**大小**: 约 50-100KB

**包含内容**:
- **common/** (2 个文件）: 通用工具和数据模型
  - gamecontroller_keymapping_model.cpp
  - gamecontroller_utils.cpp

- **sa_client/** (2 个文件）: SA 客户端
  - gamecontroller_server_client.cpp
  - gamecontroller_server_client_proxy.cpp

- **IDL 生成的代码**: IPC stub/proxy

**外部依赖**: 无（仅依赖 IPC 生成的代码）

**用途**:
- 提供 InnerAPI 接口（供终端厂商使用）
- 实现 IPC 客户端
- 被游戏应用、终端厂商服务、SA 进程共享

**依赖它的产物**:
- libohgame_controller.z.so
- libgamecontroller_fwk_client.z.so
- libgamecontroller_server.z.so

**证据**:
- `frameworks/native/BUILD.gn:84-93`: sources 列表
- `frameworks/native/BUILD.gn:97`: deps 定义（game_controller_interface）
- `interfaces/kits/c/BUILD.gn:54-57`: ohgame_controller 依赖它
- `service/BUILD.gn:53`: gamecontroller_server 依赖它

### 5. libgamecontroller_server.z.so (SA）

**Target**: `gamecontroller_server`

**证据**: `service/BUILD.gn:42`

**安装路径**: `/system/lib/libgamecontroller_server.z.so`

**类型**: System Ability（`shlib_type = "sa"`）

**大小**: 约 200-300KB

**包含内容** (10 个源文件）:
- **common/** (2 个文件）: 服务层通用工具
  - json_utils.cpp
  - permission_utils.cpp

- **device_manager/** (1 个文件）: 设备管理
  - device_manager.cpp

- **event/** (1 个文件）: 事件发布
  - event_publisher.cpp

- **ipc/** (2 个文件）: IPC 和 SA 生命周期
  - ability_event_handler.cpp
  - gamecontroller_server_ability.cpp

- **key_mapping_manager/** (2 个文件）: 配置管理
  - game_support_key_mapping_manager.cpp
  - key_mapping_config_manager.cpp

**外部依赖**: libgamecontroller_client.z.so

**用途**:
- 实现 System Ability 服务
- 提供设备识别、配置管理、事件发布
- 作为独立进程运行

**证据**:
- `service/BUILD.gn:45-46`: sources 列表（10 个文件）
- `service/BUILD.gn:53`: deps 定义
- `sa_profile/8450.json:2-6`: SA 进程配置

## 配置文件产物

### JSON 配置文件

| 文件 | 安装路径 | 大小 | 用途 | 管理 Target |
|------|----------|------|------|----------|
| device_config.json | /system/etc/game_controller/game_controller_service/ | ~1-5KB | 设备识别信息 | DeviceManager |
| default_key_mapping.json | /system/etc/game_controller/game_controller_service/ | ~5-10KB | 默认按键映射规则 | KeyMappingConfigManager |
| custom_key_mapping.json | /system/etc/game_controller/game_controller_service/ | ~5-10KB | 自定义按键映射规则 | KeyMappingConfigManager |
| game_support_key_mapping.json | /system/etc/game_controller/game_controller_service/ | ~1-5KB | 支持转触控游戏列表 | GameSupportKeyMappingManager |

**证据**:
- `etc/BUILD.gn`: ohos_prebuilt_etc targets 定义
- `README_zh.md:13-17`: 配置文件说明

### SA 配置文件

| 文件 | 安装路径 | 大小 | 用途 |
|------|----------|------|------|
| gamecontroller_server.cfg | /system/etc/init/ | ~0.5KB | SA 启动配置 |
| 8450.json | /system/profile/ | ~0.5KB | SA 系统能力配置 |

**证据**:
- `etc/init/BUILD.gn`: ohos_prebuilt_etc target
- `sa_profile/BUILD.gn`: ohos_sa_profile target
- `sa_profile/8450.json`: 完整配置

## 运行时加载关系

### 应用进程加载链示例

```
应用启动
    │
    ├─► 链接 libohgame_controller.z.so（编译时链接）
    │
    ├─► Window Framework
    │      └─► dlopen("/system/lib/libgamecontroller_event.z.so")
    │             ├─► 加载 libgamecontroller_event.z.so
    │             ├─► 加载 libgamecontroller_fwk_client.z.so（依赖）
    │             └─► 加载 libgamecontroller_client.z.so（依赖）
    │
    └─► 应用调用 OH_GameDevice_* / OH_GamePad_* C API
              └─► 通过 CAPI Proxy 调用 Framework 实现
```

**证据**:
- `README_zh.md:81-84`: Window Framework dlopen 加载
- `frameworks/native/BUILD.gn:218`: libgamecontroller_event 依赖 gamecontroller_fwk_client
- `interfaces/kits/c/BUILD.gn:54-57`: libohgame_controller 依赖 gamecontroller_client 和 gamecontroller_fwk_client

### SA 进程加载链示例

```
应用调用 InnerAPI 或外设连接
    │
    ├─► GameControllerServerClient (libgamecontroller_client.z.so）
    │
    ├─► SystemAbilityManager::LoadSystemAbility(8450)
    │      └─► 拉起 gamecontroller_server 进程
    │             ├─► 加载 libgamecontroller_server.z.so
    │             └─► 加载 libgamecontroller_client.z.so（共享数据模型）
    │
    └─► IPC 调用（通过 IPC Stub/Proxy）
```

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client_proxy.h`: IPC 加载逻辑
- `sa_profile/8450.json`: SA ID = 8450，run-on-create: false（按需启动）
- `service/BUILD.gn:53`: gamecontroller_server 依赖 gamecontroller_client

## 产物依赖图

```
应用进程产物依赖：
┌─────────────────────────────────────┐
│ libohgame_controller.z.so        │
│  ├── libgamecontroller_client.z.so │
│  └── libgamecontroller_fwk_client.z.so
│         └── libgamecontroller_client.z.so
└─────────────────────────────────────┘

SA 进程产物依赖：
┌─────────────────────────────┐
│ libgamecontroller_server.z.so │
│  └── libgamecontroller_client.z.so (共享)
└─────────────────────────────────┘

跨进程依赖：
应用进程 libgamecontroller_client.z.so ──IPC──► SA 进程 libgamecontroller_server.z.so
```

**证据**: 各 BUILD.gn 文件的 deps 定义

## 编译输出目录

### 标准编译输出

**命令**:
```bash
./build.sh --product-name rk3568 --ccache --build-target game_controller_framework --build-variant root
```

**证据**: `README_zh.md:121`

**输出路径**: `/out/rk3568/game/game_controller_framework/`

**产物清单**:
- `libgamecontroller_client.z.so`
- `libgamecontroller_service.z.so`
- `libgamecontroller_event.z.so`
- `libohgame_controller.z.so`

### 系统安装路径

| 产物 | 安装路径 |
|------|----------|
| libohgame_controller.z.so | /system/ndk/libohgame_controller.z.so |
| libgamecontroller_event.z.so | /system/lib/libgamecontroller_event.z.so |
| libgamecontroller_fwk_client.z.so | /system/lib/libgamecontroller_fwk_client.z.so |
| libgamecontroller_client.z.so | /system/lib/libgamecontroller_client.z.so |
| libgamecontroller_server.z.so | /system/lib/libgamecontroller_server.z.so |
| *.json 配置文件 | /system/etc/game_controller/game_controller_service/ |
| gamecontroller_server.cfg | /system/etc/init/ |
| 8450.json | /system/profile/ |

**证据**:
- `interfaces/kits/c/BUILD.gn:65`: relative_install_dir = "ndk/"
- `etc/BUILD.gn`: 安装路径配置
- `sa_profile/BUILD.gn`: SA 配置安装

## 运行时大小估算

### 应用进程内存占用

```
应用启动后加载：
- libohgame_controller.z.so: ~50-100KB 代码段
- libgamecontroller_event.z.so: ~100-200KB
- libgamecontroller_fwk_client.z.so: ~300-500KB
- libgamecontroller_client.z.so: ~50-100KB（共享）
───────────────────────────────────────────────────
总计: ~500-900KB（代码段）
```

### SA 进程内存占用

```
SA 进程（按需启动）：
- libgamecontroller_server.z.so: ~200-300KB 代码段
- libgamecontroller_client.z.so: ~50-100KB（共享）
- 配置文件缓存: ~10-50KB
───────────────────────────────────────────────────
总计: ~260-450KB（基础）
```

**证据**: 基于源文件数量和编译配置估算

## 关键结论

1. **五个主要 .so 库**：
   - libohgame_controller.z.so: CAPI 库
   - libgamecontroller_event.z.so: 事件库（dlopen 加载）
   - libgamecontroller_fwk_client.z.so: 框架客户端（最大）
   - libgamecontroller_client.z.so: InnerAPI 库（共享）
   - libgamecontroller_server.z.so: SA 库（独立进程）

2. **配置文件安装到系统**：4 个 JSON 文件

3. **加载方式**：
   - 应用进程：链接 + dlopen（事件库）
   - SA 进程：按需启动，IPC 通信

4. **编译产物路径**：/out/rk3568/game/game_controller_framework/

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [06_GN_Targets.md](./06_GN_Targets.md) - GN 构建目标详解

---

**版本**: 1.0 | **更新时间**: 2026-02-06
