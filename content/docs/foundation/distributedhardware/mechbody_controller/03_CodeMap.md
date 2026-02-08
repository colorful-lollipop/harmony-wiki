# 目录结构与代码地图

## 1. 顶层目录职责

```
mechbody_controller/
├── bundle.json              # [组件配置] 定义组件元数据、依赖、构建目标
├── mechbody.gni             # [构建配置] GN构建参数定义(feature开关)
├── README.md / README_zh.md # [项目文档] 官方README
├── OAT.xml                  # [合规检查] OpenHarmony开源合规检查配置
├── LICENSE                  # [许可证] Apache 2.0
├── figures/                 # [资源] 架构图等文档图片
├── etc/init/                # [配置] SA初始化配置
├── sa_profile/              # [配置] SystemAbility配置
├── interface/               # [接口层] N-API/ANI对外接口
├── services/                # [核心] 服务实现
└── test/                    # [测试] fuzztest/unittest (本文档不覆盖)
```

---

## 2. 核心代码地图

### 2.1 服务层 (services/)

#### 2.1.1 服务入口

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `services/src/mechbody_controller_service.cpp` | ~1050 | SA服务主实现 | `MechBodyControllerService` |
| `services/include/mechbody_controller_service.h` | ~104 | 服务类定义 | `class MechBodyControllerService` |
| `services/src/mechbody_controller_stub.cpp` | ~350 | IPC请求分发 | `MechBodyControllerStub::OnRemoteRequest` |
| `services/include/mechbody_controller_stub.h` | ~35 | Stub定义 | `class MechBodyControllerStub` |

**关键调用链**: `OnRemoteRequest` (行97) → 命令码分发 → 具体接口实现

```cpp
// services/src/mechbody_controller_stub.cpp:97
std::u16string interfaceToken = data.ReadInterfaceToken();
if (interfaceToken != MECH_SERVICE_IPC_TOKEN) {
    return INVALID_PARAMETERS_ERR;
}
```

#### 2.1.2 接口定义

| 文件 | 职责 | 关键符号 |
|------|------|----------|
| `services/include/mechbody_controller_interface.h` | IPC接口定义 | `class IMechBodyController` |
| `services/include/mechbody_controller_types.h` | 数据结构定义 | `MechInfo`, `EulerAngles`, `RotateByDegreeParam` |
| `services/include/mechbody_controller_enums.h` | 枚举定义 | `MechType`, `AttachmentState`, `MechMode` |
| `services/include/mechbody_controller_ipc_interface_code.h` | IPC命令码 | `IMechBodyControllerCode` |
| `services/include/mechbody_controller_utils.h` | 工具函数 | `GetAnonymStr`, `RadToDegree` |

**关键数据结构**: 
- `MechInfo` (`types.h:37`): 设备信息（ID、MAC、名称、状态）
- `EulerAngles` (`types.h:108`): 欧拉角（yaw/roll/pitch）
- `RotateByDegreeParam` (`types.h:150`): 旋转参数

#### 2.1.3 连接管理 (connect/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `services/src/connect/mc_connect_manager.cpp` | ~280 | 蓝牙连接管理 | `MechConnectManager` |
| `services/include/connect/mc_connect_manager.h` | ~89 | 连接管理器定义 | `class MechConnectManager` |
| `services/src/connect/bluetooth_state_adapter.cpp` | ~180 | 蓝牙状态适配 | `BluetoothStateAdapter` |
| `services/include/connect/bluetooth_state_adapter.h` | ~55 | 适配器定义 | `class BluetoothStateAdapter` |
| `services/src/connect/bluetooth_state_listener.cpp` | ~100 | 蓝牙状态监听 | `BluetoothStateListener` |
| `services/include/connect/bluetooth_state_listener.h` | ~45 | 监听器定义 | `class BluetoothStateListener` |

**设备管理**: `MechConnectManager` 管理已连接设备集合 (`std::set<MechInfo> mechInfos_`)

#### 2.1.4 控制器 (controller/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `services/src/controller/mc_controller_manager.cpp` | ~450 | 控制器管理 | `ControllerManager` |
| `services/include/controller/mc_controller_manager.h` | ~120 | 控制器定义 | `class ControllerManager` |
| `services/src/controller/mc_camera_tracking_controller.cpp` | ~380 | 相机追踪控制 | `CameraTrackingController` |
| `services/include/controller/mc_camera_tracking_controller.h` | ~95 | 追踪控制器定义 | `class CameraTrackingController` |
| `services/src/controller/mc_controller_ipc_death_listener.cpp` | ~80 | IPC死亡监听 | `MechControllerIpcDeathListener` |
| `services/include/controller/mc_controller_ipc_death_listener.h` | ~45 | 监听器定义 | `class MechControllerIpcDeathListener` |

**追踪功能**: `CameraTrackingController` 接收相机数据，转换为追踪指令

#### 2.1.5 运动控制 (motion/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `services/src/motion/mc_motion_manager.cpp` | ~320 | 运动管理 | `MotionManager` |
| `services/include/motion/mc_motion_manager.h` | ~85 | 运动管理器定义 | `class MotionManager` |

**运动规划**: 管理旋转运动的状态机

#### 2.1.6 传输层 (transport/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `services/src/transport/mc_send_adapter.cpp` | ~250 | 发送适配器 | `TransportSendAdapter` |
| `services/include/transport/mc_send_adapter.h` | ~70 | 适配器定义 | `class TransportSendAdapter` |
| `services/src/transport/mc_data_buffer.cpp` | ~180 | 数据缓冲区 | `DataBuffer` |
| `services/src/transport/mc_protocol_convertor.cpp` | ~200 | 协议转换 | `ProtocolConvertor` |
| `services/src/transport/mc_subscription_center.cpp` | ~150 | 订阅中心 | `SubscriptionCenter` |
| `services/src/transport/command/mc_command_base.cpp` | ~120 | 命令基类 | `MechCommandBase` |
| `services/src/transport/command/mc_command_factory.cpp` | ~200 | 命令工厂 | `MechCommandFactory` |
| `services/src/transport/command/mc_get_mech_protocol_ver_cmd.cpp` | ~80 | 获取协议版本 | `GetMechProtocolVerCmd` |

#### 2.1.7 命令实现 (transport/command/)

**协议版本 0x01** (系统级命令):
| 文件 | 职责 |
|------|------|
| `0x01/mc_set_mech_rotation_cmd.cpp` | 设置旋转角度 |
| `0x01/mc_set_mech_rotation_by_speed_cmd.cpp` | 速度旋转 |
| `0x01/mc_set_mech_rotation_trace_cmd.cpp` | 轨迹旋转 |
| `0x01/mc_set_mech_stop_cmd.cpp` | 停止运动 |
| `0x01/mc_register_mech_state_info_cmd.cpp` | 注册状态监听 |
| `0x01/mc_register_mech_position_info_cmd.cpp` | 注册位置监听 |
| `0x01/mc_set_mech_camera_tracking_enable_cmd.cpp` | 启用追踪 |
| `0x01/mc_set_mech_camera_tracking_frame_cmd.cpp` | 追踪帧设置 |
| `0x01/mc_get_mech_limit_info_cmd.cpp` | 获取限制信息 |
| `0x01/mc_get_mech_real_name_cmd.cpp` | 获取设备名称 |

**协议版本 0x02** (普通命令):
| 文件 | 职责 |
|------|------|
| `0x02/mc_normal_set_mech_rotation_by_speed_cmd.cpp` | 普通速度旋转 |
| `0x02/mc_normal_set_mech_rotation_trace_cmd.cpp` | 普通轨迹旋转 |
| `0x02/mc_normal_get_mech_state_info_cmd.cpp` | 获取状态 |
| `0x02/mc_normal_get_mech_pose_info_cmd.cpp` | 获取姿态 |
| `0x02/mc_normal_register_mech_state_info_cmd.cpp` | 注册状态 |
| `0x02/mc_normal_set_mech_camera_tracking_enable_cmd.cpp` | 启用追踪 |

#### 2.1.8 其他核心文件

| 文件 | 行数 | 职责 |
|------|------|------|
| `services/src/ble_send_manager.cpp` | ~180 | BLE GATT发送管理 |
| `services/include/ble_send_manager.h` | ~70 | BLE发送器定义 |
| `services/src/utils/load_mechbody_adapter.cpp` | ~100 | 动态库加载 |
| `services/include/utils/load_mechbody_adapter.h` | ~35 | 加载器定义 |
| `services/src/dotReport/hisysevent_utils.cpp` | ~150 | 事件上报 |
| `services/include/dotReport/hisysevent_utils.h` | ~40 | 事件工具定义 |
| `services/src/mechbody_controller_utils.cpp` | ~80 | 通用工具实现 |

---

### 2.2 接口层 (interface/)

#### 2.2.1 N-API 接口

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `interface/napi/mech_manager/js_mech_manager.cpp` | ~1750 | N-API主实现 | `MechManager` 类 |
| `interface/napi/mech_manager/js_mech_manager.h` | ~167 | N-API头文件 | `class MechManager` |
| `interface/napi/mech_manager/js_mech_manager_client.cpp` | ~900 | IPC客户端 | `MechClient` 类 |
| `interface/napi/mech_manager/js_mech_manager_client.h` | ~150 | 客户端定义 | `class MechClient` |
| `interface/napi/mech_manager/js_mech_manager_stub.cpp` | ~150 | JS回调Stub | `JsMechManagerStub` |
| `interface/napi/mech_manager/js_mech_manager_stub.h` | ~60 | Stub定义 | `class JsMechManagerStub` |
| `interface/napi/mech_manager/js_mech_manager_service.cpp` | ~200 | 服务回调处理 | - |
| `interface/napi/mech_manager/js_mech_manager_service.h` | ~45 | 服务头文件 | - |

**N-API注册点**: `js_mech_manager.cpp:1731-1739`
```cpp
static napi_module mechManagerModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "mechbody_controller",
    .nm_priv = nullptr,
    .reserved = {0}
};
```

**JS API映射表**: `js_mech_manager.cpp:1705-1725`
```cpp
DECLARE_NAPI_FUNCTION("on", MechManager::On),
DECLARE_NAPI_FUNCTION("off", MechManager::Off),
DECLARE_NAPI_FUNCTION("getAttachedMechDevices", MechManager::GetAttachedDevices),
DECLARE_NAPI_FUNCTION("rotate", MechManager::Rotate),
// ... 共18个API
```

#### 2.2.2 ANI (ArkTS) 接口

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `interface/ets/mech_manager/src/ani_mech_manager.cpp` | ~1100 | ANI主实现 | `AniMechManager` 类 |
| `interface/ets/mech_manager/include/ani_mech_manager.h` | ~140 | ANI头文件 | `class AniMechManager` |
| `interface/ets/mech_manager/src/ani_mech_manager_client.cpp` | ~600 | ANI客户端 | `AniMechClient` 类 |
| `interface/ets/mech_manager/include/ani_mech_manager_client.h` | ~110 | 客户端定义 | `class AniMechClient` |
| `interface/ets/mech_manager/src/ani_mech_manager_stub.cpp` | ~120 | ANI回调Stub | `AniMechManagerStub` |
| `interface/ets/mech_manager/src/ani_constructor.cpp` | ~80 | 构造函数 | 模块初始化 |
| `interface/ets/mech_manager/src/ohos.distributedHardware.mechanicManager.impl.cpp` | ~200 | IDL实现 | 自动生成 |

---

### 2.3 配置与构建

#### 2.3.1 SA配置

| 文件 | 职责 |
|------|------|
| `sa_profile/8550.json` | SA ID 8550 配置 |
| `sa_profile/BUILD.gn` | SA profile构建 |
| `etc/init/mechbody.cfg` | 服务初始化配置、权限配置 |
| `etc/init/BUILD.gn` | 配置文件构建 |

#### 2.3.2 GN构建

| 文件 | 职责 |
|------|------|
| `mechbody.gni` | 构建参数(feature开关) |
| `services/BUILD.gn` | 核心服务构建目标 |
| `interface/napi/mech_manager/BUILD.gn` | N-API构建 |
| `interface/ets/mech_manager/BUILD.gn` | ANI构建 |

---

## 3. 功能到代码的映射

### 3.1 设备发现与连接

```
getAttachedMechDevices (N-API)
    ↓ js_mech_manager.cpp:472
js_mech_manager_client.cpp
    ↓ IPC
mechbody_controller_service.cpp:319 GetAttachedDevices
    ↓
mc_connect_manager.cpp:90 GetConnectMechList
    ↓
std::set<MechInfo> mechInfos_
```

### 3.2 运动控制

```
rotate (N-API)
    ↓ js_mech_manager.cpp:194
js_mech_manager_client.cpp
    ↓ IPC
mechbody_controller_service.cpp:463 RotateByDegree
    ↓
mc_motion_manager.cpp (创建运动任务)
    ↓
mc_send_adapter.cpp (发送命令)
    ↓
ble_send_manager.cpp (BLE GATT发送)
```

### 3.3 相机追踪

```
setCameraTrackingEnabled (N-API)
    ↓ js_mech_manager.cpp:1127
js_mech_manager_client.cpp
    ↓ IPC
mechbody_controller_service.cpp:339 SetTrackingEnabled
    ↓
mc_camera_tracking_controller.cpp
    ↓
Camera Service (人脸检测数据)
    ↓
Command Factory → 具体Command
    ↓
蓝牙发送
```

### 3.4 事件监听

```
on("attachStateChange", callback) (N-API)
    ↓ js_mech_manager.cpp:56
js_mech_manager_stub.cpp (注册回调Stub)
    ↓ IPC
mechbody_controller_service.cpp:209 RegisterAttachStateChangeCallback
    ↓
mc_controller_ipc_death_listener.cpp (死亡监听)
    ↓
deviceAttachCallback_[tokenId] = callback
```

---

## 4. 代码导航图

### 4.1 按功能导航

**运动控制相关**:
- 接口定义: `services/include/mechbody_controller_interface.h:46-58`
- 服务实现: `services/src/mechbody_controller_service.cpp:463-554`
- 运动管理: `services/src/motion/mc_motion_manager.cpp`
- 旋转命令: `services/src/transport/command/0x01/mc_set_mech_rotation_cmd.cpp`
- N-API: `interface/napi/mech_manager/js_mech_manager.cpp:194-282`

**追踪相关**:
- 接口定义: `services/include/mechbody_controller_interface.h:39-44`
- 服务实现: `services/src/mechbody_controller_service.cpp:339-395`
- 追踪控制器: `services/src/controller/mc_camera_tracking_controller.cpp`
- N-API: `interface/napi/mech_manager/js_mech_manager.cpp:1127-1210`

**连接管理相关**:
- 连接管理器: `services/src/connect/mc_connect_manager.cpp`
- 蓝牙适配: `services/src/connect/bluetooth_state_adapter.cpp`
- BLE发送: `services/src/ble_send_manager.cpp`

**权限相关**:
- 权限名定义: `services/src/mechbody_controller_service.cpp:39`
- 权限检查: `services/src/mechbody_controller_service.cpp:212-216`
- 系统应用检查: `interface/napi/mech_manager/js_mech_manager.cpp:1595-1600`

---

## 5. 文件依赖关系

```
mechbody_controller_service.h
    ├── mechbody_controller_stub.h
    ├── mechbody_controller_types.h
    │   └── mechbody_controller_utils.h
    ├── mechbody_controller_interface.h
    └── controller/mc_controller_manager.h

js_mech_manager.h
    ├── js_mech_manager_client.h
    ├── js_mech_manager_stub.h
    └── js_mech_manager_service.h
        └── (依赖services层types.h)
```

---

*文档创建时间: 2025-02-07*
