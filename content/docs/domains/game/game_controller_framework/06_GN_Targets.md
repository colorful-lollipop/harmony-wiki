# 06_GN_Targets - GN 构建目标

## 目的

本文档详细说明 GameController Framework 的 GN 构建目标和依赖关系。

## 适用范围

- 构建工程师
- 需要定制构建的开发者
- 理解产物结构的人员

## 根构建文件

### BUILD.gn（根）

**文件路径**: `/domains/game/game_controller_framework/BUILD.gn`

**证据**: `BUILD.gn:1-30`

```gn
import("//build/ohos.gni")
import("//build/ohos_var.gni")
import("//domains/game/game_controller_framework/game_controller_framework.gni")

group("game_controller_framework_packages") {
  deps = [
    "etc:game_controller_service_business_config",
    "etc/init:gamecontroller_server.cfg",
    "frameworks/native:gamecontroller_client",
    "frameworks/native:gamecontroller_event",
    "frameworks/native:gamecontroller_fwk_client",
    "interfaces/kits/c:ohgame_controller",
    "sa_profile:game_controller_sa_profile",
    "service:gamecontroller_server",
  ]
}
```

## 主要 Targets 清单

### 1. frameworks/native/BUILD.gn

#### Target: gamecontroller_client

**类型**: `ohos_shared_library`

**产物**: `libgamecontroller_client.z.so`

**证据**: `frameworks/native/BUILD.gn:67`

| 属性 | 值 | 说明 |
|------|-----|------|
| **branch_protector_ret** | `pac_ret` | Pointer Authentication Code |
| **sanitize.boundary_sanitize** | `true` | 边界检查 |
| **sanitize.integer_overflow** | `true` | 整数溢出检查 |
| **sanitize.cfi** | `true` | Control Flow Integrity |
| **sanitize.cfi_cross_dso** | `true` | 跨 DSO 的 CFI |
| **sanitize.stack_protector_ret** | `true` | 栈保护 |
| **sanitize.ubsan** | `true` | 未定义行为检查 |
| **cflags** | `-fstack-protector-all`, `-D_FORTIFY_SOURCE=2`, `-O2` | 编译标志 |
| **subsystem_name** | `game` | 子系统 |
| **part_name** | `game_controller_framework` | 部件 |

**源文件**:
```
common/src/gamecontroller_keymapping_model.cpp
common/src/gamecontroller_utils.cpp
sa_client/src/gamecontroller_server_client.cpp
sa_client/src/gamecontroller_server_client_proxy.cpp
+ IDL 生成的 .cpp 文件
```

**依赖**:
- `:game_controller_interface` (IDL 生成）

**外部依赖**:
- `c_utils:utils`
- `eventhandler:libeventhandler`
- `ffrt:libffrt`
- `hilog:libhilog`
- `init:libbegetutil`
- `ipc:ipc_core`
- `ipc:ipc_single`
- `safwk:system_ability_fwk`
- `samgr:samgr_proxy`

**证据**: `frameworks/native/BUILD.gn:99-109`

#### Target: gamecontroller_fwk_client

**类型**: `ohos_shared_library`

**产物**: `libgamecontroller_fwk_client.z.so`

**证据**: `frameworks/native/BUILD.gn:114`

**源文件** (33 个文件）:
```
bundle_info/src/bundle_manager.cpp
key_mapping/src/combination_key_to_touch_handler.cpp
key_mapping/src/crosshair_key_to_touch_handler.cpp
key_mapping/src/dpad_key_to_touch_handler.cpp
key_mapping/src/input_to_touch_client.cpp
key_mapping/src/key_mapping_handle.cpp
key_mapping/src/key_mapping_service.cpp
key_mapping/src/key_to_touch_handler.cpp
key_mapping/src/key_to_touch_manager.cpp
key_mapping/src/keyboard_observation_to_touch_handler.cpp
key_mapping/src/mouse_left_fire_to_touch_handler.cpp
key_mapping/src/mouse_observation_to_touch_handler.cpp
key_mapping/src/mouse_right_key_click_to_touch_handler.cpp
key_mapping/src/mouse_right_key_walking_to_touch_handler.cpp
key_mapping/src/observation_key_to_touch_handler.cpp
key_mapping/src/single_key_to_touch_handler.cpp
key_mapping/src/skill_key_to_touch_handler.cpp
multi_modal_input/src/device_event_callback.cpp
multi_modal_input/src/device_identify_service.cpp
multi_modal_input/src/device_info_service.cpp
multi_modal_input/src/game_device_client.cpp
multi_modal_input/src/input_device_listener.cpp
multi_modal_input/src/multi_modal_input_mgt_service.cpp
multi_modal_input/src/multi_modal_input_monitor.cpp
plugin/src/plugin_callback_manager.cpp
plugin/src/plugin_client.cpp
plugin/src/plugin_manager.cpp
window/src/input_event_callback.cpp
window/src/input_event_client.cpp
window/src/window_info_manager.cpp
window/src/window_input_intercept.cpp
window/src/window_opr_handle.cpp
```

**依赖**:
- `:gamecontroller_client`

**外部依赖**:
- `bundle_framework:appexecfwk_base`
- `bundle_framework:appexecfwk_core`
- `c_utils:utils`
- `eventhandler:libeventhandler`
- `ffrt:libffrt`
- `hilog:libhilog`
- `init:libbegetutil`
- `input:libmmi-client`
- `ipc:ipc_core`
- `ipc:ipc_single`
- `safwk:system_ability_fwk`
- `samgr:samgr_proxy`
- `window_manager:libwm`

**证据**: `frameworks/native/BUILD.gn:173-187`

#### Target: gamecontroller_event

**类型**: `ohos_shared_library`

**产物**: `libgamecontroller_event.z.so`

**证据**: `frameworks/native/BUILD.gn:196`

**源文件**:
```
event/src/entryModule.cpp
event/src/public_event_listener.cpp
```

**依赖**:
- `:gamecontroller_fwk_client`

**外部依赖**:
- `c_utils:utils`
- `common_event_service:cesfwk_innerkits`
- `eventhandler:libeventhandler`
- `ffrt:libffrt`
- `hilog:libhilog`
- `ipc:ipc_core`
- `ipc:ipc_single`

**证据**: `frameworks/native/BUILD.gn:218-232`

### 2. service/BUILD.gn

#### Target: gamecontroller_server

**类型**: `ohos_shared_library`

**产物**: `libgamecontroller_server.z.so`

**shlib_type**: `sa` (System Ability)

**证据**: `service/BUILD.gn:42-85`

| 属性 | 值 | 说明 |
|------|-----|------|
| **install_enable** | `true` | 安装到系统 |
| **shlib_type** | `sa` | System Ability 类型 |
| **subsystem_name** | `game` | 子系统 |
| **part_name** | `game_controller_framework` | 部件 |

**源文件** (10 个文件）:
```
common/src/json_utils.cpp
common/src/permission_utils.cpp
device_manager/src/device_manager.cpp
event/src/event_publisher.cpp
ipc/src/ability_event_handler.cpp
ipc/src/gamecontroller_server_ability.cpp
key_mapping_manager/src/game_support_key_mapping_manager.cpp
key_mapping_manager/src/key_mapping_config_manager.cpp
```

**依赖**:
- `:${game_controller_framework_innerkits_path}:gamecontroller_client`

**外部依赖**:
- `access_token:libaccesstoken_sdk`
- `access_token:libtokenid_sdk`
- `bundle_framework:appexecfwk_base`
- `bundle_framework:appexecfwk_core`
- `c_utils:utils`
- `eventhandler:libeventhandler`
- `ffrt:libffrt`
- `hilog:libhilog`
- `ipc:ipc_core`
- `ipc:ipc_single`
- `json:nlohmann_json_static`
- `safwk:system_ability_fwk`
- `samgr:samgr_proxy`

**证据**: `service/BUILD.gn:54-68`

**编译标志**:
- `ROM_USER_VERSION`: 当 build_variant == "user" 时定义

**证据**: `service/BUILD.gn:72-74`

### 3. interfaces/kits/c/BUILD.gn

#### Target: ohgame_controller

**类型**: `ohos_shared_library`

**产物**: `libohgame_controller.z.so`

**install_enable**: `true`

**relative_install_dir**: `ndk/`

**证据**: `interfaces/kits/c/BUILD.gn:27-68`

**源文件**:
```
$game_controller_framework_capi_path/c/game_device.cpp
$game_controller_framework_capi_path/c/game_device_event.cpp
$game_controller_framework_capi_path/c/game_pad.cpp
$game_controller_framework_capi_path/c/game_pad_event.cpp
$game_controller_framework_path/frameworks/capi/src/game_device_event_proxy.cpp
$game_controller_framework_path/frameworks/capi/src/game_device_proxy.cpp
$game_controller_framework_path/frameworks/capi/src/game_pad_event_proxy.cpp
$game_controller_framework_path/frameworks/capi/src/game_pad_proxy.cpp
```

**依赖**:
- `${game_controller_framework_innerkits_path}:gamecontroller_client`
- `${game_controller_framework_innerkits_path}:gamecontroller_fwk_client`

**外部依赖**:
- `c_utils:utils`
- `hilog:libhilog`

**证据**: `interfaces/kits/c/BUILD.gn:54-64`

### 4. 配置文件 Targets

#### Target: game_controller_service_business_config (group)

**证据**: `etc/BUILD.gn`

**子 Targets**:
1. `custom_key_mapping` (ohos_prebuilt_etc)
   - 源文件: `./config/custom_key_mapping.json`
   - 安装路径: `game_controller/game_controller_service/`

2. `default_key_mapping` (ohos_prebuilt_etc)
   - 源文件: `./config/default_key_mapping.json`
   - 安装路径: `game_controller/game_controller_service/`

3. `device_config` (ohos_prebuilt_etc)
   - 源文件: `./config/device_config.json`
   - 安装路径: `game_controller/game_controller_service/`

4. `game_support_key_mapping` (ohos_prebuilt_etc)
   - 源文件: `./config/game_support_key_mapping.json`
   - 安装路径: `game_controller/game_controller_service/`

**证据**: `etc/BUILD.gn` - 配置文件定义

#### Target: gamecontroller_server.cfg (ohos_prebuilt_etc)

**源文件**: `./gamecontroller_server.cfg`

**安装路径**: `init/`

**证据**: `etc/init/BUILD.gn`

#### Target: game_controller_sa_profile (ohos_sa_profile)

**源文件**: `8450.json`

**SA ID**: 8450

**证据**: `sa_profile/8450.json` 和 `sa_profile/BUILD.gn`

## Target 依赖关系图

```
game_controller_framework_packages (group)
    │
    ├──► etc:game_controller_service_business_config (group)
    │      ├──► custom_key_mapping (ohos_prebuilt_etc)
    │      ├──► default_key_mapping (ohos_prebuilt_etc)
    │      ├──► device_config (ohos_prebuilt_etc)
    │      └──► game_support_key_mapping (ohos_prebuilt_etc)
    │
    ├──► etc/init:gamecontroller_server.cfg (ohos_prebuilt_etc)
    │
    ├──► sa_profile:game_controller_sa_profile (ohos_sa_profile)
    │
    ├──► interfaces/kits/c:ohgame_controller (ohos_shared_library)
    │      ├──► frameworks/native:gamecontroller_client (dep)
    │      └──► frameworks/native:gamecontroller_fwk_client (dep)
    │
    ├──► frameworks/native:gamecontroller_event (ohos_shared_library)
    │      └──► frameworks/native:gamecontroller_fwk_client (dep)
    │
    ├──► frameworks/native:gamecontroller_fwk_client (ohos_shared_library)
    │      └──► frameworks/native:gamecontroller_client (dep)
    │
    ├──► frameworks/native:gamecontroller_client (ohos_shared_library)
    │      └──► frameworks/native:game_controller_interface (idl_gen_interface) (dep)
    │
    └──► service:gamecontroller_server (ohos_shared_library)
           └──► frameworks/native:gamecontroller_client (dep)
```

**证据**: 各 BUILD.gn 文件的 deps 定义

## Target ↔ 产物映射

| Target Name | 输出产物 | 安装路径 | 说明 |
|-------------|----------|----------|------|
| gamecontroller_client | libgamecontroller_client.z.so | /system/lib/ | InnerAPI 和数据模型库 |
| gamecontroller_fwk_client | libgamecontroller_fwk_client.z.so | /system/lib/ | 框架客户端库（设备监听、输入转触控等）|
| gamecontroller_event | libgamecontroller_event.z.so | /system/lib/ | 事件库（由 Window Framework dlopen 加载）|
| gamecontroller_server | libgamecontroller_server.z.so | /system/lib/ | SA 库（独立进程）|
| ohgame_controller | libohgame_controller.z.so | /system/ndk/ | CAPI 库（游戏应用链接）|

**证据**:
- `interfaces/kits/c/BUILD.gn:65`: relative_install_dir = "ndk/"
- `README_zh.md:126-129`: 编译产物列表

## 配置文件

### GNI 变量定义

**文件**: `game_controller_framework.gni`

**证据**: `game_controller_framework.gni`

```gn
game_controller_framework_path = "//domains/game/game_controller_framework"
game_controller_framework_innerkits_path = "${game_controller_framework_path}/frameworks/native"
game_controller_framework_capi_path = "${game_controller_framework_path}/interfaces/kits"
game_controller_service_path = "//domains/game/game_controller_framework"
```

### 业务配置文件

| 文件 | 安装路径 | 用途 |
|------|----------|------|
| custom_key_mapping.json | /system/etc/game_controller/game_controller_service/ | 自定义按键映射配置 |
| default_key_mapping.json | /system/etc/game_controller/game_controller_service/ | 默认按键映射配置 |
| device_config.json | /system/etc/game_controller/game_controller_service/ | 设备识别配置 |
| game_support_key_mapping.json | /system/etc/game_controller/game_controller_service/ | 支持转触控的游戏列表 |

**证据**:
- `README_zh.md:13-17`: 配置文件说明
- `etc/BUILD.gn`: 安装配置

### SA 配置文件

**文件**: `8450.json`

**安装路径**: `/system/profile/`

**证据**: `sa_profile/BUILD.gn`

```json
{
  "process": "gamecontroller_server",
  "systemability": [{
      "name": 8450,
      "libpath": "libgamecontroller_server.z.so",
      "run-on-create": false,
      "distributed": false,
      "dump_level": 1,
      "recycle-strategy": "low-memory",
      "start-on-demand": {"allow-update": false}
  }]
}
```

## 编译配置说明

### 安全相关配置

| 配置 | 值 | 证据 |
|------|-----|------|
| **boundary_sanitize** | `true` | 边界检查 |
| **integer_overflow** | `true` | 整数溢出检查 |
| **cfi** | `true` | Control Flow Integrity |
| **cfi_cross_dso** | `true` | 跨 DSO 的 CFI |
| **stack_protector_ret** | `true` | 栈保护 |
| **ubsan** | `true` | 未定义行为检查 |
| **branch_protector_ret** | `pac_ret` | Pointer Authentication Code |

**证据**: `frameworks/native/BUILD.gn:69-77`

### 编译优化

| 标志 | 值 | 说明 |
|------|-----|------|
| `-fstack-protector-all` | 启用 | 栈保护 |
| `-D_FORTIFY_SOURCE=2` | 启用 | Fortify Source |
| `-O2` | 启用 | 优化级别 2 |

**证据**: `frameworks/native/BUILD.gn:78-81`

### 条件编译

**ROM_USER_VERSION**: 当 `build_variant == "user"` 时定义

**证据**: `service/BUILD.gn:72-74`

## 构建产物详情

### 主要库文件

#### libohgame_controller.z.so (CAPI)

- **目标**: `ohgame_controller`
- **安装路径**: `/system/ndk/libohgame_controller.z.so`
- **大小**: 约 50-100KB（估算）
- **依赖**: libgamecontroller_client.z.so, libgamecontroller_fwk_client.z.so
- **链接方式**: 游戏应用在编译时链接（-lohgame_controller）

**证据**: `interfaces/kits/c/BUILD.gn:27-68`

#### libgamecontroller_event.z.so (事件库）

- **目标**: `gamecontroller_event`
- **安装路径**: `/system/lib/libgamecontroller_event.z.so`
- **大小**: 约 100-200KB（估算）
- **加载方式**: Window Framework 通过 dlopen 动态加载
- **依赖**: libgamecontroller_fwk_client.z.so

**证据**:
- `README_zh.md:81-84`: Window Framework dlopen 加载
- `frameworks/native/BUILD.gn:196-235`

#### libgamecontroller_fwk_client.z.so (框架客户端）

- **目标**: `gamecontroller_fwk_client`
- **安装路径**: `/system/lib/libgamecontroller_fwk_client.z.so`
- **大小**: 约 300-500KB（估算）
- **包含**: 按键映射、设备监听、输入拦截、插件、Bundle 管理

**证据**: `frameworks/native/BUILD.gn:114-90`

#### libgamecontroller_client.z.so (InnerAPI）

- **目标**: `gamecontroller_client`
- **安装路径**: `/system/lib/libgamecontroller_client.z.so`
- **大小**: 约 50-100KB（估算）
- **依赖**: 无（仅依赖 IPC 生成的代码）
- **用途**: 提供 InnerAPI 接口

**证据**: `frameworks/native/BUILD.gn:67-112`

#### libgamecontroller_server.z.so (SA）

- **目标**: `gamecontroller_server`
- **安装路径**: `/system/lib/libgamecontroller_server.z.so`
- **类型**: System Ability (shlib_type = "sa"）
- **进程名**: `gamecontroller_server`
- **大小**: 约 200-300KB（估算）
- **依赖**: libgamecontroller_client.z.so

**证据**:
- `sa_profile/8450.json:2-6`: SA 配置
- `service/BUILD.gn:42-85`: SA 编译配置

## 关键结论

1. **根目标**: `game_controller_framework_packages` 聚合所有子目标
2. **五个主要库**: libohgame_controller.z.so, libgamecontroller_event.z.so, libgamecontroller_fwk_client.z.so, libgamecontroller_client.z.so, libgamecontroller_server.z.so
3. **依赖层次**: ohgame_controller → gamecontroller_fwk_client → gamecontroller_client
4. **配置文件**: 四个 JSON 配置文件安装到系统
5. **SA 特性**: 按需启动（run-on-create: false），低内存回收策略

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物详解
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构

---

**版本**: 1.0 | **更新时间**: 2026-02-06
