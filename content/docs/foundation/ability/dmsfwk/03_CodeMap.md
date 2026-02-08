# 03_CodeMap - 目录结构与代码地图

## 顶层目录结构

```
foundation/ability/dmsfwk/
├── bundle.json              # 组件元数据
├── dmsfwk.gni              # GN 编译配置
├── hisysevent.yaml         # 系统事件配置
├── README.md               # 项目说明
│
├── common/                 # 通用工具库
│   ├── include/           # 公共头文件
│   └── src/               # 公共实现
│
├── etc/                    # 配置文件
│   ├── init/              # 服务启动配置
│   └── profile/           # 信任配置文件
│
├── sa_profile/            # SystemAbility 配置
│   ├── 1401.json          # DistributedSched SA
│   └── 1404.json          # DistributedAbilityManager SA
│
├── interfaces/            # 对外接口
│   ├── innerkits/         # C++ SDK
│   ├── kits/napi/         # JS/TS N-API
│   └── taihe/             # Taihe/ANI 接口
│
├── frameworks/            # 框架实现
│   ├── js/                # JS/ArkTS 扩展
│   └── native/            # Native 扩展
│
├── services/              # 系统服务实现
│   ├── dtbschedmgr/       # 分布式调度服务 (SA 1401)
│   ├── dtbabilitymgr/     # 能力管理服务 (SA 1404)
│   └── dtbcollabmgr/      # 协作管理服务
│
└── test/                  # 测试代码 (fuzz/unit)
```

---

## 核心代码导航

### 1. 系统服务入口

| 功能 | 文件路径 | 关键行 |
|-----|---------|-------|
| **主服务注册** | `services/dtbschedmgr/src/distributed_sched_service.cpp` | 178-180 |
| **服务启动** | `services/dtbschedmgr/src/distributed_sched_service.cpp` | 419-494 |
| **IPC 分发** | `services/dtbschedmgr/src/distributed_sched_stub.cpp` | 95-266 |
| **能力管理服务** | `services/dtbabilitymgr/src/distributed_ability_manager_service.cpp` | 57-59 |

### 2. N-API 接口层

| 模块 | 入口文件 | 注册函数 |
|-----|---------|---------|
| **ContinuationManager** | `interfaces/kits/napi/continuation_manager/continuation_manager_module.cpp` | `JsContinuationManagerInit` |
| **AbilityConnectionManager** | `interfaces/kits/napi/ability_connection_manager/ability_connection_manager_module.cpp` | `JsAbilityConnectionManagerInit` |
| **ContinuationStateManager** | `interfaces/kits/napi/continuation_state_manager/js_continuation_state_manager.cpp` | `ContinuationStateManagerModuleRegister` |

### 3. IPC 接口定义

| 接口 | 头文件 | 描述符 |
|-----|-------|-------|
| **IDistributedSched** | `services/dtbschedmgr/include/distributed_sched_interface.h` | `OHOS.DistributedSchedule.IDistributedSched` |
| **IAbilityConnectionManager** | `services/dtbcollabmgr/include/ability_connection_manager/ability_connection_manager_interface.h` | `OHOS.DistributedCollab.IAbilityConnectionManager` |
| **IDExtension** | `frameworks/native/distributed_extension/include/ipc/i_distributed_extension.h` | `OHOS.AppManagement.Distributed.IExtension` |
| **IDeviceSelectionNotifier** | `interfaces/innerkits/continuation_manager/include/idevice_selection_notifier.h` | `OHOS.DistributedSchedule.IDeviceSelectionNotifier` |

### 4. SoftBus 网络层

| 组件 | 文件路径 | 职责 |
|-----|---------|------|
| **Transport Adapter** | `services/dtbschedmgr/src/softbus_adapter/transport/dsched_transport_softbus_adapter.cpp` | 网络连接管理 |
| **Session 管理** | `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp` | 数据传输与会话 |
| **广播监听** | `services/dtbschedmgr/src/softbus_adapter/softbus_adapter.cpp` | 设备事件监听 |
| **通道管理** | `services/dtbcollabmgr/src/channel_manager/channel_manager.cpp` | 协作通道管理 |

---

## 功能到文件映射

### 续接功能 (Continuation)

```
启动续接
  └── interfaces/kits/napi/continuation_manager/js_continuation_manager.cpp:120
      └── services/dtbabilitymgr/src/distributed_ability_manager_service.cpp:237
          └── services/dtbschedmgr/src/continue/dsched_continue.cpp:155

设备选择
  └── services/dtbabilitymgr/src/continuation_manager/device_selection_notifier_*.cpp

状态保存/恢复
  └── services/dtbschedmgr/src/continue/dsched_continue.cpp:530
      └── services/dtbschedmgr/src/distributedWant/distributed_want.cpp:1070

SoftBus 传输
  └── services/dtbschedmgr/src/softbus_adapter/transport/dsched_transport_softbus_adapter.cpp:530
```

### 协作功能 (Collaboration)

```
创建会话
  └── interfaces/kits/napi/ability_connection_manager/js_ability_connection_manager.cpp:2209
      └── services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:85

连接管理
  └── services/dtbcollabmgr/src/ability_connection_manager/ability_connection_session.cpp:368

数据传输
  └── services/dtbcollabmgr/src/channel_manager/data_sender_receiver.cpp:148

音视频流
  └── services/dtbcollabmgr/src/av_trans_stream_provider/
```

### 远程启动

```
StartRemoteAbility
  └── services/dtbschedmgr/src/distributed_sched_stub.cpp:271
      └── services/dtbschedmgr/src/distributed_sched_service.cpp:880
          └── 权限检查: services/dtbschedmgr/src/distributed_sched_permission.cpp:129
```

---

## 关键数据结构

| 结构体 | 文件 | 用途 |
|-------|------|------|
| **DistributedWant** | `services/dtbschedmgr/src/distributedWant/distributed_want.h` | 跨设备意图 |
| **CallerInfo** | `services/dtbschedmgr/include/caller_info.h` | 调用者信息 |
| **ContinuationResult** | `services/dtbabilitymgr/include/continuation_manager/continuation_result.h` | 续接结果 |
| **SessionDataHeader** | `services/dtbschedmgr/include/softbus_adapter/transport/dsched_softbus_session.h` | 软总线会话头 |

---

## 配置文件位置

| 配置 | 文件路径 | 安装位置 |
|-----|---------|---------|
| **SA 配置** | `sa_profile/1401.json` | `/system/profile/` |
| **启动配置** | `etc/init/distributedsched.cfg` | `/system/etc/init/` |
| **信任配置** | `etc/profile/distributedsched_trust.json` | `/system/profile/` |

---

## 构建产物地图

| 产物 | 类型 | 输出路径 |
|-----|------|---------|
| `libdistributedschedsvr.z.so` | shared_library | `system/lib/platformsdk/` |
| `libdistributed_ability_manager_svr.z.so` | shared_library | `system/lib/` |
| `libcontinuationmanager_napi.z.so` | shared_library | `system/lib/module/continuation/` |
| `libabilityconnectionmanager_napi.z.so` | shared_library | `system/lib/module/distributedsched/` |
| `libdtbcollab_channel_manager.z.so` | shared_library | `system/lib/platformsdk/` |

---

## 调试与日志

| 日志标签 | 文件 | 级别 |
|---------|------|------|
| `DistributedSchedStub` | `services/dtbschedmgr/src/distributed_sched_stub.cpp` | HILOG |
| `DSchedContinue` | `services/dtbschedmgr/src/continue/dsched_continue.cpp` | HILOG |
| `AbilityConnectionManager` | `services/dtbcollabmgr/src/ability_connection_manager/` | DTBCOLLABMGR_HILOG |

---

## 相关链接

- 上一章: [02_Architecture.md](02_Architecture.md) - 系统架构
- 下一章: [04_Interface.md](04_Interface.md) - 接口文档
