# 目录结构与模块职责

## 文档信息

- **目的**: 介绍项目的目录组织、各模块职责和文件分布
- **适用范围**: 新人了解代码布局、开发者定位代码、架构师理解模块边界
- **关键结论**:
  1. 项目采用分层架构：Server/Service/Stack/Hardware
  2. Profile 按功能模块独立实现
  3. 测试代码完全隔离在 test/ 目录
- **相关文档**: [00_Overview](00_Overview.md), [02_Architecture](02_Architecture.md), [04_GN_Targets](04_GN_Targets.md)

---

## 顶层目录结构

```
bluetooth_service/
├── bluetooth.gni              # Feature flags 定义
├── bundle.json                # 组件元数据与依赖
├── hisysevent.yaml           # HiSysEvent 事件定义
├── LICENSE                   # Apache 2.0 许可证
├── README.md                 # 项目说明（占位）
├── sa_profile/               # System Ability 配置
├── services/                 # 核心代码（不含测试）
│   ├── bluetooth/           # 蓝牙服务主目录
│   │   ├── common/         # 公共头文件和日志
│   │   ├── etc/           # 配置文件
│   │   │   └── init/    # 进程启动配置
│   │   ├── external/       # 外部依赖封装
│   │   ├── hardware/      # HDI 硬件接口层
│   │   ├── ipc/          # IPC Skeleton/Proxy
│   │   ├── server/       # IPC Server (SA 实现)
│   │   ├── service/      # 业务服务层
│   │   │   ├── src/
│   │   │   │   ├── a2dp_snk/     # A2DP Sink Profile
│   │   │   │   ├── a2dp_src/     # A2DP Source Profile
│   │   │   │   ├── avrcp_ct/     # AVRCP CT Profile
│   │   │   │   ├── avrcp_tg/     # AVRCP TG Profile
│   │   │   │   ├── base/         # 基础实现
│   │   │   │   ├── ble/          # BLE 实现
│   │   │   │   ├── classic/      # Classic 实现
│   │   │   │   ├── common/       # 公共组件
│   │   │   │   ├── dialog/       # 配对对话框
│   │   │   │   ├── gatt/        # GATT Profile
│   │   │   │   ├── gavdp/        # A2DP 音频传输
│   │   │   │   ├── hfp_ag/       # HFP AG Profile
│   │   │   │   ├── hfp_hf/       # HFP HF Profile
│   │   │   │   ├── hid_host/     # HID Host Profile
│   │   │   │   ├── obex/        # OBEX 协议
│   │   │   │   ├── pan/         # PAN Profile
│   │   │   │   ├── permission/   # 权限检查
│   │   │   │   ├── sock/        # Socket 支持
│   │   │   │   ├── transport/   # 传输层
│   │   │   │   └── util/        # 工具类
│   │   │   └── include/        # 服务层头文件
│   │   └── stack/         # 蓝牙协议栈
│   │       ├── include/     # 协议栈头文件
│   │       ├── platform/    # 平台抽象层
│   │       └── src/       # 协议栈源码
│   └── bluetooth_lite/    # LiteOS 支持
└── test/                  # 测试代码（本文档不引用）
    ├── example/           # 示例应用
    ├── fuzztest/         # 模糊测试
    ├── moduletest/        # 模块测试
    └── unittest/         # 单元测试
```

---

## 根目录文件说明

| 文件 | 说明 | 关键内容 |
|------|------|----------|
| `bluetooth.gni` | Feature flags 配置 | Profile 开关、条件编译选项 |
| `bundle.json` | 组件元数据 | 版本、依赖、编译产物定义 |
| `hisysevent.yaml` | HiSysEvent 定义 | 事件类型、参数说明 |
| `LICENSE` | 许可证 | Apache License 2.0 |
| `README.md` | 项目说明（占位） | Gitee 平台说明模板 |

**证据**:
- `bluetooth.gni:14-32` - Feature flags
- `bundle.json:1-122` - 完整元数据
- `hisysevent.yaml:14-84` - HiSysEvent 事件

---

## services/bluetooth/ 详解

### common/ - 公共头文件

**职责**: 公共类型定义、日志宏、工具函数

**关键文件**:
- `log.h` - 日志接口定义
- `bluetooth_log.h` - 蓝牙日志宏

---

### etc/init/ - 启动配置

**职责**: System Ability 启动配置

**关键文件**:
- `bluetooth_service.cfg` - 进程启动配置

**关联**:
- `services/bluetooth/server/BUILD.gn:102` - `etc:etc` 依赖
- `sa_profile/1130.json` - SA 定义

---

### external/ - 外部依赖封装

**职责**: 封装外部依赖，提供统一接口

**子目录**:
- `dummy/` - 空实现/桩实现（用于编译配置）

**关键文件**:
- `include/stub/` - 外部接口桩定义
- `src/` - 桩实现

**用途**: 允许在无完整依赖时编译（用于测试/CI）

---

### hardware/ - HDI 硬件接口层

**职责**: 封装 HDI 接口调用，与蓝牙 HAL 交互

**关键文件**:

| 文件 | 说明 |
|------|------|
| `include/bt_vendor_lib.h` | 厂商库接口 |
| `include/bluetooth_hdi.h` | HDI 接口定义 |
| `include/bluetooth_hci_callbacks.h` | HCI 回调接口 |
| `src/bluetooth_hdi.cpp` | HDI 实现 |
| `src/bluetooth_hci_callbacks.cpp` | HCI 回调处理 |

**证据**:
- `services/bluetooth/hardware/include/` - 头文件列表

**关联**:
- `services/bluetooth/service/` - 通过 HDI 调用硬件
- `drivers_interface_bluetooth` - HDI 接口定义（外部依赖）

---

### ipc/ - IPC Skeleton/Proxy

**职责**: 生成 IPC 通信代码（Skeleton 和 Proxy）

**关键模式**: 每个 Profile 一个 Stub + Proxy

**示例文件**:

| Profile | Stub | Proxy | Observer Proxy |
|---------|-------|--------|----------------|
| Host | `bluetooth_host_stub.h/cpp` | - | `bluetooth_host_observer_proxy.h/cpp` |
| GATT Server | `bluetooth_gatt_server_stub.h/cpp` | - | `bluetooth_gatt_server_callback_proxy.h/cpp` |
| GATT Client | `bluetooth_gatt_client_stub.h/cpp` | - | `bluetooth_gatt_client_callback_proxy.h/cpp` |
| BLE Advertiser | `bluetooth_ble_advertiser_stub.h/cpp` | - | `bluetooth_ble_advertise_callback_proxy.h/cpp` |
| BLE Central Manager | `bluetooth_ble_central_manager_stub.h/cpp` | - | `bluetooth_ble_central_manager_callback_proxy.h/cpp` |
| A2DP Source | `bluetooth_a2dp_src_stub.h/cpp` | - | `bluetooth_a2dp_src_observer_proxy.h/cpp` |
| Socket | `bluetooth_socket_stub.h/cpp` | - | `bluetooth_socket_observer_proxy.h/cpp` |

**证据**:
- `services/bluetooth/ipc/BUILD.gn:34-105` - Stub/Proxy 源文件
- `services/bluetooth/ipc/include/` - 所有头文件

**输出**: `libbtipc_service.a` (静态库)

**关联**:
- `services/bluetooth/server/` - Server 层继承 Stub
- `bluetooth 框架` - 客户端使用 Proxy

---

### server/ - IPC Server (SA 实现)

**职责**: 实现 System Ability（SA），处理 IPC 请求，进行权限检查

**核心类**:

| 类 | 职责 | 文件 |
|-----|------|------|
| `BluetoothHostServer` | SA 主类，实现 `SystemAbility` 和 `BluetoothHostStub` | `include/bluetooth_host_server.h:33` |
| 各 Profile Server | 处理 Profile 特定请求 | `include/bluetooth_*_server.h` |

**Profile Server 列表**:

| Profile | Server 类 | 文件 |
|---------|----------|------|
| Host | `BluetoothHostServer` | `bluetooth_host_server.h/cpp` |
| GATT Server | `BluetoothGattServerServer` | `bluetooth_gatt_server_server.h/cpp` |
| GATT Client | `BluetoothGattClientServer` | `bluetooth_gatt_client_server.h/cpp` |
| BLE Advertiser | `BluetoothBleAdvertiserServer` | `bluetooth_ble_advertiser_server.h/cpp` |
| BLE Central Manager | `BluetoothBleCentralManagerServer` | `bluetooth_ble_central_manager_server.h/cpp` |
| A2DP Source | `BluetoothA2dpSourceServer` | `bluetooth_a2dp_source_server.h/cpp` |
| A2DP Sink | `BluetoothA2dpSinkServer` | `bluetooth_a2dp_sink_server.h/cpp` |
| AVRCP CT | `BluetoothAvrcpCtServer` | `bluetooth_avrcp_ct_server.h/cpp` |
| AVRCP TG | `BluetoothAvrcpTgServer` | `bluetooth_avrcp_tg_server.h/cpp` |
| HFP AG | `BluetoothHfpAgServer` | `bluetooth_hfp_ag_server.h/cpp` |
| HFP HF | `BluetoothHfpHfServer` | `bluetooth_hfp_hf_server.h/cpp` |
| HID Host | `BluetoothHidHostServer` | `bluetooth_hid_host_server.h/cpp` |
| PAN | `BluetoothPanServer` | `bluetooth_pan_server.h/cpp` |
| Socket | `BluetoothSocketServer` | `bluetooth_socket_server.h/cpp` |

**证据**:
- `services/bluetooth/server/BUILD.gn:37-48` - Server 源文件
- `services/bluetooth/server/include/` - Server 头文件

**其他关键文件**:

| 文件 | 职责 |
|------|------|
| `bluetooth_utils_server.cpp` | 工具函数 |
| `bluetooth_host_dumper.cpp` | Dump 实现（调试） |
| `bluetooth_ble_filter_matcher.cpp` | BLE 广播过滤器匹配 |
| `remote_observer_list.h` | 远程观察者列表管理 |
| `bluetooth_hitrace.cpp` | 性能追踪 |

**输出**: `libbluetooth_server.z.so` (共享库)

**关联**:
- `sa_profile/1130.json` - SA 配置
- `services/bluetooth/ipc/` - 继承 Stub
- `services/bluetooth/service/` - 调用业务逻辑

---

### service/ - 业务服务层

**职责**: 实现核心业务逻辑、Profile 管理、状态机、协议调用

**子模块详解**:

#### src/ble/ - BLE 实现

**关键文件**:

| 文件 | 说明 |
|------|------|
| `ble_adapter.cpp` | BLE 适配器实现 |
| `ble_advertiser_impl.cpp` | 广播实现 |
| `ble_central_manager_impl.cpp` | 中心管理器实现 |
| `ble_config.cpp` | BLE 配置 |
| `ble_properties.cpp` | BLE 属性管理 |
| `ble_security.cpp` | BLE 安全 |
| `ble_utils.cpp` | 工具函数 |

#### src/classic/ - Classic 实现

**关键文件**:

| 文件 | 说明 |
|------|------|
| `classic_adapter.cpp` | Classic 适配器实现 |
| `classic_adapter_properties.cpp` | Classic 属性管理 |
| `classic_remote_device.cpp` | 远程设备管理 |
| `classic_config.cpp` | Classic 配置 |
| `classic_utils.cpp` | 工具函数 |

#### src/common/ - 公共组件

**职责**: 状态机、配置管理、电源管理

**关键文件**:

| 文件 | 说明 |
|------|------|
| `adapter_manager.cpp` | 适配器管理器 |
| `adapter_state_machine.cpp` | 适配器状态机 |
| `profile_service_manager.cpp` | Profile 服务管理 |
| `power_manager.cpp` | 电源管理 |
| `power_state_machine.cpp` | 电源状态机 |
| `sys_state_machine.cpp` | 系统状态机 |

#### src/permission/ - 权限检查

**职责**: 权限验证、Token 管理、调用方身份识别

**关键文件**:

| 文件 | 说明 |
|------|------|
| `permission_manager.cpp` | 权限管理器 |
| `permission_helper.cpp` | 权限辅助函数 |
| `auth_center.cpp` | 认证中心 |
| `permission_utils.cpp` | 权限工具 |

**证据**:
- `services/bluetooth/service/src/permission/` - 所有权限文件

**关联**:
- `services/bluetooth/server/` - Server 层调用权限检查
- `access_token` 组件 - Token 管理

#### src/sock/ - Socket 支持

**关键文件**:

| 文件 | 说明 |
|------|------|
| `socket.cpp` | Socket 实现 |
| `socket_gap_client.cpp` | GAP 客户端 |
| `socket_gap_server.cpp` | GAP 服务器 |
| `socket_sdp_client.cpp` | SDP 客户端 |
| `socket_sdp_server.cpp` | SDP 服务器 |
| `socket_service.cpp` | Socket 服务 |
| `socket_listener.cpp` | Socket 监听器 |

#### src/transport/ - 传输层

**关键文件**:

| 文件 | 说明 |
|------|------|
| `transport_factory.cpp` | 传输工厂 |
| `transport_l2cap.cpp` | L2CAP 传输 |
| `transport_rfcomm.cpp` | RFCOMM 传输 |

#### src/util/ - 工具类

**关键文件**:

| 文件 | 说明 |
|------|------|
| `state_machine.cpp` | 状态机基类 |
| `timer.cpp` | 定时器 |
| `dispatcher.cpp` | 事件分发器 |
| `bluetooth_common_event_helper.cpp` | 公共事件辅助 |
| `xml_parse.cpp` | XML 解析（OBEX 配置） |

#### src/gatt/ - GATT Profile

**关键文件**:

| 文件 | 说明 |
|------|------|
| `gatt_client_profile.cpp` | GATT 客户端 Profile |
| `gatt_server_profile.cpp` | GATT 服务器 Profile |
| `gatt_client_service.cpp` | GATT 客户端服务 |
| `gatt_server_service.cpp` | GATT 服务器服务 |
| `gatt_connection_manager.cpp` | 连接管理 |
| `gatt_database.cpp` | GATT 数据库 |
| `gatt_cache.cpp` | GATT 缓存 |

#### src/gavdp/ - A2DP 音频传输

**关键文件**:

| 文件 | 说明 |
|------|------|
| `a2dp_profile.cpp` | A2DP Profile |
| `a2dp_source.cpp` | A2DP 源端 |
| `a2dp_sink.cpp` | A2DP 接收端 |
| `a2dp_avdtp.cpp` | AVDTP 协议 |
| `a2dp_sdp.cpp` | SDP 协议 |

#### src/hfp_ag/, src/hfp_hf/ - HFP Profile

**职责**: 免提协议实现

**关键文件**:

| 目录 | 说明 |
|------|------|
| `hfp_ag/` | HFP AG（车载端） |
| `hfp_hf/` | HFP HF（手机端） |

#### src/avrcp_ct/, src/avrcp_tg/ - AVRCP Profile

**职责**: 音视频远程控制

**关键文件**:

| 目录 | 说明 |
|------|------|
| `avrcp_ct/` | AVRCP 控制器 |
| `avrcp_tg/` | AVRCP 目标 |

#### src/hid_host/ - HID Host Profile

**职责**: HID 设备支持

**关键文件**:

| 文件 | 说明 |
|------|------|
| `hid_host_service.cpp` | HID Host 服务 |
| `hid_host_hogp.cpp` | HOGP 实现 |
| `hid_host_l2cap_connection.cpp` | L2CAP 连接 |

#### src/pan/ - PAN Profile

**职责**: 个人区域网络

**关键文件**:

| 文件 | 说明 |
|------|------|
| `pan_service.cpp` | PAN 服务 |
| `pan_bnep.cpp` | BNEP 协议 |
| `pan_network.cpp` | 网络层 |

#### src/obex/ - OBEX 协议

**职责**: 对象交换协议（OPP/PBAP）

**关键文件**:

| 文件 | 说明 |
|------|------|
| `obex_client.cpp` | OBEX 客户端 |
| `obex_server.cpp` | OBEX 服务器 |
| `obex_mp_client.cpp` | 多部分客户端 |
| `obex_mp_server.cpp` | 多部分服务器 |
| `obex_transport.cpp` | 传输层 |
| `obex_socket_transport.cpp` | Socket 传输 |

#### src/dialog/ - 配对对话框

**职责**: UI 交互支持

**关键文件**:

| 文件 | 说明 |
|------|------|
| `bluetooth_dialog.cpp` | 对话框实现 |
| `dialog_pair.cpp` | 配对对话框 |
| `dialog_switch.cpp` | 开关对话框 |
| `bluetooth_ability_connection.cpp` | Ability 连接 |

#### include/ - 服务层头文件

**职责**: 接口定义

**关键文件**:

| 文件 | 说明 |
|------|------|
| `interface_adapter_manager.h` | 适配器管理器接口 |
| `interface_adapter_ble.h` | BLE 适配器接口 |
| `interface_adapter_classic.h` | Classic 适配器接口 |
| `interface_profile.h` | Profile 基础接口 |
| `interface_profile_*.h` | 各 Profile 接口 |

**输出**: `libbtservice.z.so` (共享库)

**关联**:
- `services/bluetooth/server/` - 调用服务层
- `services/bluetooth/stack/` - 调用协议栈

---

### stack/ - 蓝牙协议栈

**职责**: 实现蓝牙核心协议（HCI、L2CAP、GAP、SMP 等）

**子模块**:

#### src/hci/ - Host Controller Interface

**职责**: HCI 命令和事件处理

**子目录**:
- `cmd/` - HCI 命令
- `evt/` - HCI 事件
- `acl/` - ACL 数据

#### src/l2cap/ - L2CAP 协议

**职责**: 逻辑链路控制和适配协议

#### src/rfcomm/ - RFCOMM 协议

**职责**: 串口仿真协议

#### src/sdp/ - Service Discovery Protocol

**职责**: 服务发现协议

#### src/gap/ - Generic Access Profile

**职责**: 通用访问协议

#### src/smp/ - Security Manager Protocol

**职责**: 安全管理协议（BLE 配对）

#### src/att/ - Attribute Protocol

**职责**: 属性协议（GATT 基础）

#### src/avctp/ - Audio/Video Control Transport Protocol

**职责**: 音视频控制传输协议

#### src/avdtp/ - Audio/Video Distribution Transport Protocol

**职责**: 音视频分发传输协议（A2DP 基础）

#### src/btm/ - Bluetooth Manager

**职责**: 蓝牙管理器

**platform/ - 平台抽象**

**职责**: 平台相关代码封装（Linux 等）

**输出**: `libbtstack.z.so` (共享库，推断)

**关联**:
- `services/bluetooth/service/` - 服务层调用协议栈
- `services/bluetooth/hardware/` - 硬件层提供底层支持

---

### bluetooth_lite/ - LiteOS 支持

**职责**: 适配 LiteOS 平台

---

## test/ - 测试目录

**说明**: 本文档不引用 test/ 目录内容

**子目录**:

| 目录 | 说明 |
|------|------|
| `example/` | 示例应用 |
| `fuzztest/` | 模糊测试 |
| `moduletest/` | 模块测试 |
| `unittest/` | 单元测试 |

---

## 目录组织原则

### 分层架构体现

```
services/bluetooth/
├── server/      # SA 入口、IPC 接口
├── service/     # 业务逻辑、Profile 实现
├── stack/       # 协议栈实现
└── hardware/    # 硬件接口层
```

### Profile 独立性

每个 Profile 独立实现于 `service/src/` 下的子目录：
- 便于维护和扩展
- 可独立启用/禁用（通过 Feature flags）
- 清晰的职责边界

### 公共组件复用

`common/` 和 `util/` 目录包含：
- 状态机框架
- 配置管理
- 工具函数
- 避免重复代码

---

## 总结

bluetooth_service 项目采用清晰的分层架构，目录结构明确反映模块职责：

1. **server/** - System Ability 和 IPC 接口
2. **service/** - 业务逻辑和 Profile 实现
3. **stack/** - 蓝牙协议栈
4. **hardware/** - 硬件接口层
5. **ipc/** - IPC 代码生成
6. **test/** - 测试代码（本文档不引用）

**相关文档**:
- 架构详解: [02_Architecture](02_Architecture.md)
- 内部接口: [03_Internal_API](03_Internal_API.md)
- 构建系统: [04_GN_Targets](04_GN_Targets.md)
