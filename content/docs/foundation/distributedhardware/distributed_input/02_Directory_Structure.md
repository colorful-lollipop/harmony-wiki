# 目录结构 (Directory Structure)

## 目的

本文档提供 distributed_input 模块的完整目录结构和各模块职责说明。

## 适用范围

本文档适用于以下场景：
- 新加入项目的开发者了解代码组织
- 代码导航和模块定位
- 理解各目录的职责和依赖关系

## 关键结论

1. **分层架构**: 代码按照基础层、服务层、接口层分层组织
2. **职责清晰**: 每个目录有明确的职责划分（constants、services、interfaces 等）
3. **源码分离**: 每个模块都有 include/ 和 src/ 目录，头文件和实现分离
4. **测试独立**: 测试代码完全独立于源码（已排除在本文档外）

## 目录树（排除测试）

```
distributed_input/
├── common/                          # 常量定义和公共函数
│   └── include/
├── dfx_utils/                       # DFX 业务相关实现
│   ├── include/
│   └── src/
├── figures/                         # 文档资源（架构图等）
├── frameworks/                      # 回调函数定义
│   └── include/
├── inputdevicehandler/              # 能力查询接口实现
│   ├── include/
│   └── src/
├── interfaces/                      # 对外接口模块
│   ├── inner_kits/                  # 内部 SDK (DistributedInputKit)
│   │   ├── include/
│   │   └── src/
│   └── ipc/                         # IPC 接口实现
│       ├── include/
│       └── src/
├── sa_profile/                      # SA 配置信息
├── services/                        # SA 具体实现
│   ├── common/                      # 公共常量定义
│   ├── sink/                        # Sink 侧 SA 实现
│   │   ├── inputcollector/          # 事件采集
│   │   ├── sinkmanager/             # Sink 侧业务管理
│   │   └── transport/               # Sink 侧事件发送
│   ├── source/                      # Source 侧 SA 实现
│   │   ├── inputinject/             # 事件注入
│   │   ├── sourcemanager/           # Source 侧业务管理
│   │   └── transport/               # Source 侧事件接收
│   ├── state/                       # 状态管理
│   └── transportbase/               # 驱动事件数据传输接口
├── sinkhandler/                     # Sink 侧部件接入接口
├── sourcehandler/                   # Source 侧部件接入接口
└── utils/                           # 工具类实现
    ├── include/
    └── src/
```

## 模块职责详细说明

### 1. common/ - 公共常量和函数

**职责**: 定义整个模块共享的常量、数据结构、枚举和工具函数。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `constants_dinput.h` | 360+ | 核心常量：RawEvent、InputDevice 结构、输入类型枚举、会话状态、缓冲区大小 |
| `dinput_errcode.h` | - | 错误码定义（详见[安全评审](08_Security_Review.md)错误码部分） |
| `dinput_hitrace.h` | - | HiTrace 日志定义 |
| `input_hub.h/cpp` | - | 输入设备集线器，用于读取事件 |
| `white_list_util.h/cpp` | 334 | 键组合白名单过滤逻辑 |
| `input_check_param.h/cpp` | - | 参数校验逻辑 |
| `dinput_ipc_interface_code.h` | - | IPC 接口命令码定义 |

**依赖**: 无（基础层）

**证据**: [README_zh.md:72](../README_zh.md:72) 提到 common/ 为公共常量定义

---

### 2. frameworks/ - 回调函数定义

**职责**: 定义 Inner SDK 使用的回调接口，用于异步操作和事件通知。

**关键文件**:

| 文件 | 说明 | 用途 |
|------|------|------|
| `i_distributed_source_input.h` | Source 侧 IPC 接口定义（Init、Release、Start/StopRemoteInput 等） | [services/source/sourcemanager/include/distributed_input_source_manager.h:50](../services/source/sourcemanager/include/distributed_input_source_manager.h:50) |
| `i_distributed_sink_input.h` | Sink 侧 IPC 接口定义（InitSink、NotifyStartDScreen 等） | [services/sink/sinkmanager/include/distributed_input_sink_manager.h:50](../services/sink/sinkmanager/include/distributed_input_sink_manager.h:50) |
| `i_prepare_d_input_call_back.h` | Prepare 回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:25](../interfaces/inner_kits/include/distributed_input_kit.h:25) |
| `i_start_d_input_call_back.h` | Start 回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:27](../interfaces/inner_kits/include/distributed_input_kit.h:27) |
| `i_stop_d_input_call_back.h` | Stop 回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:28](../interfaces/inner_kits/include/distributed_input_kit.h:28) |
| `i_register_d_input_call_back.h` | Register 回调接口 | - |
| `i_simulation_event_listener.h` | 仿真事件监听器接口 | [interfaces/inner_kits/include/distributed_input_kit.h:80](../interfaces/inner_kits/include/distributed_input_kit.h:80) |
| `i_input_node_listener.h` | 输入节点监听器接口 | - |
| `i_session_state_callback.h` | 会话状态回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:33](../interfaces/inner_kits/include/distributed_input_kit.h:33) |
| `i_add_white_list_infos_call_back.h` | 白名单添加回调 | - |
| `i_del_white_list_infos_call_back.h` | 白名单删除回调 | - |
| `i_get_sink_screen_infos_call_back.h` | Sink 屏幕信息回调 | - |
| `i_sharing_dhid_listener.h` | 共享 dhId 监听器接口 | - |
| `i_start_stop_d_inputs_call_back.h` | 批量 Start/Stop 回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:29](../interfaces/inner_kits/include/distributed_input_kit.h:29) |
| `i_start_stop_result_call_back.h` | Start/Stop 结果回调接口 | - |
| `i_unprepare_d_input_call_back.h` | Unprepare 回调接口 | [interfaces/inner_kits/include/distributed_input_kit.h:26](../interfaces/inner_kits/include/distributed_input_kit.h:26) |
| `i_unregister_d_input_call_back.h` | Unregister 回调接口 | - |
| `i_dinput_context.h` | 上下文接口定义 | - |

**依赖**: `common/`

---

### 3. interfaces/ - 对外接口模块

#### 3a. inner_kits/ - 内部 SDK

**职责**: 提供 DistributedInputKit API，供多模输入模块调用。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_kit.h` | 91 | 公共 API：PrepareRemoteInput、StartRemoteInput、StopRemoteInput、IsNeedFilterOut 等（静态方法） |
| `distributed_input_kit.cpp` | - | 公共 API 实现 |

**输出库**: `libdinput_sdk.so`

**证据**: [bundle.json:57-58](../bundle.json:57-58) 定义 libdinput_sdk 为 inner_kits

#### 3b. ipc/ - IPC 接口实现

**职责**: 实现 Stub 和 Proxy，用于客户端和服务之间的 IPC 通信。

**关键文件**:

| 文件 | 说明 |
|------|------|
| `distributed_input_source_stub.h/cpp` | Source 服务 Stub（服务端 IPC），继承自 `IRemoteStub<IDistributedSourceInput>` |
| `distributed_input_source_proxy.h/cpp` | Source 服务 Proxy（客户端 IPC），用于调用 Source 服务 |
| `distributed_input_sink_stub.h/cpp` | Sink 服务 Stub（服务端 IPC），继承自 `IRemoteStub<IDistributedSinkInput>` |
| `distributed_input_sink_proxy.h/cpp` | Sink 服务 Proxy（客户端 IPC），用于调用 Sink 服务 |
| `distributed_input_client.h/cpp` | 客户端类，同时支持 Source 和 Sink 操作（228 行） |
| `dinput_sa_manager.h/cpp` | System Ability 管理器，用于加载和查找 SA |
| 各种回调 stub/proxy | Start/Stop/Prepare/Unprepare 等回调的 Stub 和 Proxy 实现（共 18 个回调文件） |

**依赖**: `frameworks/`、`common/`

**证据**: [interfaces/ipc/include](../interfaces/ipc/include) 目录包含 36 个 IPC 相关头文件

---

### 4. services/ - SA 具体实现

#### 4a. transportbase/ - 基础传输层

**职责**: 基于 SoftBus 的跨设备通信基础，提供会话管理和消息发送/接收。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_transport_base.h/cpp` | 105/ - 基础传输类：会话管理、消息发送/接收、SoftBus 集成 |
| `softbus_permission_check.h/cpp` | - | SoftBus 权限检查：账号验证、访问控制、同账号检查 |
| `distributed_input_transport_base.cpp` | - | 基础传输实现 |

**输出库**: `libdinput_trans_base.so`

**依赖**: `common/`、`services/common/`

**证据**: [bundle.json:67](../bundle.json:67) 列出 libdinput_trans_base

#### 4b. services/common/ - 服务公共定义

**职责**: 定义服务层共享的常量、回调和 SoftBus 消息类型。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `dinput_softbus_define.h` | 114 | SoftBus 消息类型常量（TRANS_SINK_MSG_ONPREPARE、TRANS_SOURCE_MSG_PREPARE 等） |
| `dinput_source_manager_callback.h` | - | Source Manager 回调接口 |
| `dinput_sink_manager_callback.h` | - | Sink Manager 回调接口 |
| `dinput_source_trans_callback.h` | - | Source 传输回调接口 |
| `dinput_sink_trans_callback.h` | - | Sink 传输回调接口 |
| `dinput_transbase_source_callback.h` | - | 传输基类 Source 回调 |
| `dinput_transbase_sink_callback.h` | - | 传输基类 Sink 回调 |

**依赖**: `common/`、`frameworks/`

---

#### 4c. services/source/ - Source 侧服务

##### 4c-i. sourcemanager/ - Source 业务管理

**职责**: Source 侧业务管理和外部接口实现，实现 System Ability (SA 4809)。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_source_manager.h` | 500 | 主要 Source Manager：SystemAbility、Register/Unregister、Prepare/Start/Stop 操作、回调处理 |
| `distributed_input_source_manager.cpp` | - | Source Manager 实现（含 SA 注册和发布） |
| `dinput_source_listener.h/cpp` | - | Source 监听器，用于传输层事件 |
| `distributed_input_source_event_handler.h/cpp` | - | Source 操作事件处理器 |
| `dinput_source_manager_event_handler.h/cpp` | - | Manager 事件处理器 |
| `distributed_input_source_sa_cli_mgr.h/cpp` | - | SA 客户端管理器 |

**输出库**: `libdinput_source.so`

**SA ID**: 4809

**依赖**: `services/common/`、`services/transportbase/`、`services/source/transport/`、`services/source/inputinject/`、`frameworks/`、`interfaces/ipc/`

**证据**: [distributed_input_source_manager.h:110](../services/source/sourcemanager/include/distributed_input_source_manager.h:110) - DECLARE_SYSTEM_ABILITY

##### 4c-ii. inputinject/ - 输入注入

**职责**: 将接收的原始输入事件注入到虚拟输入驱动。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_inject.h` | 64 | 事件注入类：RegisterDistributedHardware、RegisterDistributedEvent、虚拟设备管理 |
| `distributed_input_inject.cpp` | - | 事件注入实现 |
| `distributed_input_node_manager.h/cpp` | - | 虚拟输入节点管理器 |
| `virtual_device.h/cpp` | - | 虚拟设备创建和管理 |

**输出库**: `libdinput_inject.so`

**依赖**: `common/`、`inputdevicehandler/`

**证据**: [bundle.json:63](../bundle.json:63) 列出 libdinput_inject

##### 4c-iii. transport/ - Source 事件接收

**职责**: Source 侧传输层，接收来自 Sink 的事件。

**关键文件**:
| 文件 | 说明 |
|------|------|
| `distributed_input_source_transport.h/cpp` | Source 传输实现 |
| `distributed_input_source_sa_cli_mgr.h/cpp` | Source SA 客户端管理器 |

**输出库**: `libdinput_source_trans.so`

**依赖**: `services/transportbase/`、`services/common/`

**证据**: [bundle.json:62](../bundle.json:62) 列出 libdinput_source_trans

---

#### 4d. services/sink/ - Sink 侧服务

##### 4d-i. sinkmanager/ - Sink 业务管理

**职责**: Sink 侧业务管理，响应 Source 调用，实现 System Ability (SA 4810)。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_sink_manager.h` | 206 | 主要 Sink Manager：SystemAbility、Init/Release、屏幕信息回调、共享 dhId 管理 |
| `distributed_input_sink_manager.cpp` | - | Sink Manager 实现（含 SA 注册和发布） |
| `distributed_input_sink_event_handler.h/cpp` | - | Sink 操作事件处理器 |

**输出库**: `libdinput_sink.so`

**SA ID**: 4810

**依赖**: `services/common/`、`services/transportbase/`、`services/sink/transport/`、`services/sink/inputcollector/`、`services/state/`

**证据**: [distributed_input_sink_manager.h:50](../services/sink/sinkmanager/include/distributed_input_sink_manager.h:50) - DECLARE_SYSTEM_ABILITY

##### 4d-ii. inputcollector/ - 输入采集

**职责**: 从本地输入驱动采集输入外设原始事件。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_collector.h` | 84 | 事件采集器：StartCollectionThread、SetSharingTypes、GetSharingDhIds |
| `distributed_input_collector.cpp` | - | 事件采集实现 |

**输出库**: `libdinput_collector.so`

**依赖**: `common/`、`inputdevicehandler/`

**证据**: [bundle.json:66](../bundle.json:66) 列出 libdinput_collector

##### 4d-iii. transport/ - Sink 事件发送

**职责**: Sink 侧传输层，将事件发送到 Source。

**关键文件**:
| 文件 | 说明 |
|------|------|
| `distributed_input_sink_transport.h/cpp` | Sink 传输实现 |
| `distributed_input_sink_switch.h/cpp` | Sink 传输开关 |

**输出库**: `libdinput_sink_trans.so`

**依赖**: `services/transportbase/`、`services/common/`

**证据**: [bundle.json:65](../bundle.json:65) 列出 libdinput_sink_trans

---

#### 4e. services/state/ - 状态管理

**职责**: 管理设备状态（THROUGH_IN/THROUGH_OUT）和按键状态跟踪。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `dinput_sink_state.h` | 103 | 状态管理器：DhIdState 枚举、按键按下/释放状态跟踪、触摸板事件片段管理 |
| `dinput_sink_state.cpp` | - | 状态管理实现 |
| `touchpad_event_fragment_mgr.h/cpp` | - | 触摸板事件片段管理器 |
| `touchpad_event_fragment.h/cpp` | - | 触摸板事件片段定义 |

**输出库**: `libdinput_sink_state.so`

**依赖**: `common/`、`services/sink/inputcollector/`、`services/sink/transport/`、`dfx_utils/`、`utils/`

**证据**: [bundle.json:68](../bundle.json:68) 列出 libdinput_sink_state

---

### 5. sourcehandler/ - DH Framework Source 集成

**职责**: 实现分布式硬件管理框架定义的 Source 侧部件接入接口。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_source_handler.h` | 105 | Source Handler：InitSource、RegisterDistributedHardware、ConfigDistributedHardware |
| `distributed_input_source_handler.cpp` | - | Source Handler 实现 |
| `load_d_input_source_callback.h/cpp` | - | SA 加载回调 |

**输出库**: `libdinput_source_handler.so`

**依赖**: `interfaces/ipc/`、`services/source/sourcemanager/`

**证据**: [bundle.json:69](../bundle.json:69) 列出 libdinput_source_handler

---

### 6. sinkhandler/ - DH Framework Sink 集成

**职责**: 实现分布式硬件管理框架定义的 Sink 侧部件接入接口。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_sink_handler.h` | 98 | Sink Handler：InitSink、SubscribeLocalHardware、Pause/Resume/StopDistributedHardware |
| `distributed_input_sink_handler.cpp` | - | Sink Handler 实现（含隐私资源注册） |
| `load_d_input_sink_callback.h/cpp` | - | SA 加载回调 |

**输出库**: `libdinput_sink_handler.so`

**依赖**: `interfaces/ipc/`、`services/sink/sinkmanager/`

**证据**: [bundle.json:70](../bundle.json:70) 列出 libdinput_sink_handler

---

### 7. inputdevicehandler/ - 硬件能力查询

**职责**: 实现硬件能力查询接口，由分布式硬件管理框架定义。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `distributed_input_handler.h` | 87 | 硬件处理器：Initialize、QueryMeta、Query、FindDevicesInfoByType |
| `distributed_input_handler.cpp` | - | 硬件处理器实现 |

**输出库**: `libdinput_handler.so`

**依赖**: `common/`

**证据**: [bundle.json:71](../bundle.json:71) 列出 libdinput_handler

---

### 8. dfx_utils/ - DFX 工具

**职责**: 提供 DFX（Debug、Diagnosis、Trace）相关工具和 HiDumper 实现。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `hidumper.h` | 84 | HiDumper 类：节点信息、会话信息、dump 命令 |
| `hidumper.cpp` | - | HiDumper 实现 |
| `hisysevent_util.h/cpp` | - | HiSysEvent 工具类 |

**输出库**: `libdinput_dfx_utils.so`

**依赖**: `common/`、`utils/`

**证据**: [bundle.json:72](../bundle.json:72) 列出 libdinput_dfx_utils

---

### 9. utils/ - 通用工具

**职责**: 提供通用的工具函数和辅助类。

**关键文件**:
| 文件 | 行数 | 说明 |
|------|------|------|
| `dinput_utils_tool.h` | 74 | 工具函数：GetLocalDeviceInfo、GetUUIDBySoftBus、SetAnonyId、JSON 校验、设备路径扫描 |
| `dinput_utils_tool.cpp` | - | 工具函数实现 |
| `dinput_log.h` | - | 日志宏定义 |
| `dinput_context.h/cpp` | - | 上下文管理 |

**输出库**: `libdinput_utils.so`

**依赖**: `common/`

**证据**: [bundle.json:73](../bundle.json:73) 列出 libdinput_utils

---

### 10. sa_profile/ - SA 配置

**职责**: System Ability 配置文件和服务初始化配置。

**关键文件**:
| 文件 | 说明 |
|------|------|
| `4809.json` | Source SA 配置（SA ID: 4809, lib: libdinput_source.z.so） |
| `4810.json` | Sink SA 配置（SA ID: 4810, lib: libdinput_sink.z.so） |
| `dinput.cfg` | Init 配置文件（进程: dinput） |
| `BUILD.gn` | SA profile 构建配置（35 行） |

**权限**: 根据 [dinput.cfg](../sa_profile/dinput.cfg:9-12)，服务拥有以下权限：
- `ohos.permission.DISTRIBUTED_DATASYNC`
- `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE`

**证据**: [sa_profile/BUILD.gn](../sa_profile/BUILD.gn:17-34) 定义 SA profile 目标

---

## 模块依赖关系

### 层次依赖

```
基础层 (Base Layer)
├── common/ - 常量、数据结构
└── utils/ - 工具函数

框架层 (Framework Layer)
└── frameworks/ - 回调接口定义

传输层 (Transport Layer)
└── services/transportbase/ - SoftBus 传输基础

状态层 (State Layer)
└── services/state/ - 设备状态管理

服务层 (Service Layer)
├── services/source/
│   ├── sourcemanager/ - Source Manager (SA 4809)
│   ├── inputinject/ - 事件注入
│   └── transport/ - Source 事件接收
└── services/sink/
    ├── sinkmanager/ - Sink Manager (SA 4810)
    ├── inputcollector/ - 事件采集
    └── transport/ - Sink 事件发送

接口层 (Interface Layer)
├── interfaces/inner_kits/ - Inner SDK (libdinput_sdk)
└── interfaces/ipc/ - IPC Stub/Proxy 实现

框架集成层 (Framework Integration)
├── sourcehandler/ - DH Framework Source 接入
└── sinkhandler/ - DH Framework Sink 接入

工具层 (Utility Layer)
├── inputdevicehandler/ - 硬件能力查询
└── dfx_utils/ - DFX 工具
```

### 依赖方向

**无环依赖**：模块依赖方向清晰，无循环依赖。

| 模块 | 依赖 |
|------|------|
| services/source/sourcemanager | services/common/, services/transportbase/, services/source/transport/, services/source/inputinject/, frameworks/, interfaces/ipc/, common/, utils/ |
| services/sink/sinkmanager | services/common/, services/transportbase/, services/sink/transport/, services/sink/inputcollector/, services/state/, frameworks/, interfaces/ipc/, common/, utils/ |
| interfaces/inner_kits | frameworks/, interfaces/ipc/, common/, utils/ |
| sourcehandler | interfaces/ipc/, services/source/sourcemanager/ |
| sinkhandler | interfaces/ipc/, services/sink/sinkmanager/ |
| inputdevicehandler | common/ |
| dfx_utils | common/, utils/ |
| services/transportbase | common/, services/common/ |

## 相关跳转

- [架构设计](03_Architecture.md) - 详细的组件图和数据流
- [公共 API](04_Public_API.md) - Inner SDK 接口详解
- [GN 目标](06_GN_Targets.md) - 编译目标和产物映射

---

*更新时间: 2026-02-06 15:08:55*
