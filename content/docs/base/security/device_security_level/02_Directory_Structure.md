# 目录结构与模块职责

## 目的

本文档详细说明 DSLM 模块的目录结构，各目录的职责、包含的文件和模块边界。

## 适用范围

- ✅ 完整目录树（排除 test/ 目录）
- ✅ 各目录的职责说明
- ✅ 核心文件与类说明

## 完整目录树（排除测试）

```
base/security/device_security_level/
├── baselib/                           # 基础库
│   ├── msglib/                           # 消息通信库
│   │   ├── include/
│   │   │   └── messenger.h              # 消息库对外接口
│   │   └── src/
│   │       ├── common/
│   │       │   ├── messenger.c              # 消息核心实现
│   │       │   ├── messenger_impl.h          # 消息实现基类
│   │       │   └── messenger_utils.c        # 消息工具函数
│   │       ├── lite/
│   │       │   └── messenger_device_session_manager.c  # Lite 系统会话管理
│   │       ├── standard/
│   │       │   └── messenger_device_socket_manager.cpp  # Standard 系统 Socket 管理
│   │       └── utils/
│   │           └── messenger_utils.h       # 消息工具声明
│   │
│   └── utils/                            # 工具库
│       ├── include/
│       │   ├── utils_base64.h            # Base64 编解码
│       │   ├── utils_datetime.h           # 日期时间处理
│       │   ├── utils_hexstring.h          # 十六进制字符串转换
│       │   ├── utils_json.h              # JSON 处理
│       │   ├── utils_mem.h               # 内存管理
│       │   ├── utils_state_machine.h     # 状态机
│       │   ├── utils_tlv.h              # TLV 编解码
│       │   ├── utils_work_queue.h        # 工作队列
│       │   ├── utils_mutex.h             # 互斥锁
│       │   ├── utils_timer.h             # 定时器
│       │   ├── utils_dslm_list.h         # DSLM 链表
│       │   └── utils_log.h              # 日志
│       └── src/
│           ├── utils_base64.c
│           ├── utils_datetime.c
│           ├── utils_hexstring.c
│           ├── utils_json.c
│           ├── utils_mem.c
│           ├── utils_state_machine.c
│           ├── utils_tlv.c
│           ├── utils_work_queue.c
│           ├── utils_mutex.c
│           ├── utils_timer.c
│           ├── utils_dslm_list.c
│           └── utils_log.c
│
├── common/                               # 公共定义与配置
│   ├── include/
│   │   ├── idevice_security_level.h              # IPC 接口定义（IDeviceSecurityLevel）
│   │   ├── dslm_service_ipc_interface_code.h # IPC 命令码定义
│   │   └── dslm_additions.h               # 扩展定义
│   └── BUILD.gn                                 # 公共编译配置
│
├── interfaces/                           # 对外接口层
│   └── inner_api/                      # Inner API（C 接口 + SDK）
│       ├── include/
│       │   ├── device_security_info.h           # 对外 C API 头文件
│       │   └── device_security_defines.h       # 公共数据结构定义
│       └── src/
│           ├── standard/                       # Standard 系统实现
│           │   ├── device_security_info.cpp         # API 实现（同步/异步）
│           │   ├── device_security_level_proxy.h     # IPC Proxy 声明
│           │   ├── device_security_level_proxy.cpp   # IPC Proxy 实现
│           │   ├── device_security_level_loader.h    # SA 加载器
│           │   ├── device_security_level_loader.cpp  # SA 加载实现
│           │   ├── device_security_level_callback_stub.h    # 回调 Stub 声明
│           │   ├── device_security_level_callback_stub.cpp  # 回调 Stub 实现
│           │   ├── device_security_level_callback_helper.h   # 回调辅助
│           │   └── device_security_level_callback_helper.cpp # 回调辅助实现
│           └── lite/                           # Lite 系统实现
│               ├── device_security_info.c               # API 实现
│               ├── small/
│               │   ├── device_security_level_proxy.h # IPC Proxy 声明
│               │   └── device_security_level_proxy.c # IPC Proxy 实现
│               └── mini/
│                   ├── device_security_level_inner.h  # 内部 API 声明
│                   └── device_security_level_inner.c  # 内部 API 实现
│
├── oem_property/                         # OEM 适配层
│   ├── include/
│   │   ├── dslm_credential.h              # 凭据接口定义
│   │   ├── dslm_cred.h                  # 凭据信息定义
│   │   └── dslm_additions.h            # 扩展定义
│   ├── common/                              # 公共代码
│   │   ├── dslm_credential.c           # 默认凭据实现
│   │   ├── dslm_credential_utils.c     # 凭据工具（签名验证）
│   │   └── dslm_additions.h
│   └── ohos/                               # OHOS 实现
│       ├── common/
│       │   ├── dslm_ohos_request.c       # 凭据请求
│       │   ├── dslm_ohos_verify.c         # 凭据验证（核心）
│       │   ├── external_interface_adapter.c # 外部接口适配
│       │   └── impl/
│       │       └── dslm_ohos_init.c        # OHOS 初始化
│       ├── standard/                            # Standard 系统
│       │   ├── dslm_ohos_credential.c         # 凭据实现
│       │   ├── impl/
│       │   │   ├── dslm_ohos_request.c      # 凭据请求实现
│       │   │   ├── dslm_ohos_verify.c        # 凭据验证实现
│       │   │   └── hks_adapter.c            # HUKS 适配器
│       │   └── impl/
│       │       └── dslm_ohos_cred_obj.c       # 凭据对象
│       └── lite/                                # Lite 系统
│           ├── dslm_ohos_credential.c                 # 简化凭据实现
│           └── impl/
│               └── dslm_ohos_cred_obj.c             # 凭据对象
│
├── profile/                              # 组件配置
│   ├── dslm_service.xml                   # SA 配置（XML）
│   ├── dslm_service.json                  # SA 配置（JSON）
│   ├── dslm_service.cfg                   # 权限配置
│   ├── dslm_service.rc                    # 启动脚本
│   └── BUILD.gn                                # 配置编译
│
├── services/                             # 服务框架代码
│   ├── include/                             # 服务内部头文件
│   │   ├── dslm_ipc_process.h              # IPC 处理接口
│   │   ├── dslm_additions.h              # 扩展定义
│   │   └── dslm_hidumper.h               # Dump 工具
│   │
│   ├── sa/                                  # SA 服务实现
│   │   ├── common/
│   │   │   ├── dslm_rpc_process.c          # RPC 处理（Lite/Standard 公共）
│   │   │   └── dslm_rpc_process.h
│   │   ├── standard/
│   │   │   ├── dslm_service.h             # DslmService 类声明
│   │   │   ├── dslm_service.cpp           # DslmService 实现
│   │   │   ├── dslm_ipc_process.h         # Standard IPC 处理
│   │   │   ├── dslm_ipc_process.cpp       # Standard IPC 处理实现
│   │   │   ├── dslm_callback_proxy.h       # 回调 Proxy
│   │   │   └── dslm_callback_proxy.cpp   # 回调 Proxy 实现
│   │   └── lite/
│   │       ├── small/
│   │       │   ├── dslm_service.h         # Lite 服务声明
│   │       │   ├── dslm_ipc_process.h     # Lite IPC 处理
│   │       │   ├── dslm_ipc_process.c   # Lite IPC 处理实现
│   │       │   ├── dslm_service.c         # Lite 服务实现
│   │       │   └── dslm_service_main.c  # Lite 服务入口
│   │       └── mini/
│   │           ├── dslm_service.h         # Mini 服务声明
│   │           ├── dslm_inner_process.h    # Mini 内部处理
│   │           ├── dslm_inner_process.c   # Mini 内部处理实现
│   │           ├── dslm_service.c         # Mini 服务实现
│   │           └── dslm_service_feature.c # Mini 特性
│   │
│   ├── dslm/                                # DSLM 核心逻辑
│   │   ├── dslm_core_defines.h              # 核心定义
│   │   ├── dslm_core_process.c              # 核心处理
│   │   ├── dslm_device_list.h              # 设备列表
│   │   ├── dslm_device_list.c              # 设备列表实现
│   │   ├── dslm_fsm_process.h              # 状态机
│   │   ├── dslm_fsm_process.c              # 状态机实现
│   │   ├── dslm_hievent.h                 # HiSysEvent
│   │   ├── dslm_hievent.c                 # HiSysEvent 实现
│   │   ├── dslm_inner_process.h            # 内部处理
│   │   ├── dslm_inner_process.c            # 内部处理实现
│   │   ├── dslm_msg_utils.h               # 消息工具
│   │   └── dslm_msg_utils.c               # 消息工具实现
│   │
│   ├── msg/                                 # 消息处理
│   │   ├── dslm_messenger_wrapper.c          # 消息封装
│   │   └── BUILD.gn
│   │
│   ├── dfx/                                 # DFX 调测
│   │   ├── dslm_bigdata.cpp               # 大数据统计
│   │   ├── dslm_hidumper.cpp               # HiDumper
│   │   ├── dslm_hitrace.cpp               # HiTrace
│   │   ├── dslm_dfx_default.c             # 默认 DFX
│   │   └── BUILD.gn
│   │
│   └── common/                              # 公共服务代码
│       ├── dslm_crypto.c                     # 加密工具
│       └── dslm_msg_serialize.c             # 消息序列化
│
├── figures/                              # 文档图片
│   ├── ohos_system_security_architecture.png
│   └── ohos_device_security_level.png
│
└── test/                                # 测试代码（本文档不涵盖）
```

## 各目录职责详解

### 1. baselib/ - 基础库

#### 1.1 msglib/ - 消息通信库

**职责**：封装跨设备通信能力，提供统一的消息发送/接收接口。

**核心接口**（`baselib/msglib/include/messenger.h`）：
- `CreateMessenger()` - 创建消息对象
- `DestroyMessenger()` - 销毁消息对象
- `SendMsgTo()` - 发送消息到指定设备
- `IsMessengerReady()` - 检查消息对象是否就绪
- `GetDeviceOnlineStatus()` - 查询设备在线状态
- `GetSelfDeviceIdentify()` - 获取本设备标识
- `ForEachDeviceProcess()` - 遍历所有设备

**适配的系统**：
- **Standard**: 使用 DSoftBus Socket 管理（`messenger_device_socket_manager.cpp`）
- **Small**: 使用 Lite 会话管理（`messenger_device_session_manager.c`）

#### 1.2 utils/ - 工具库

**职责**：提供通用的工具函数和基础数据结构。

**工具列表**：
- `utils_base64` - Base64 编解码
- `utils_datetime` - 日期时间处理
- `utils_hexstring` - 十六进制字符串转换
- `utils_json` - JSON 处理（使用 cJSON）
- `utils_mem` - 内存管理（内存分配、释放）
- `utils_state_machine` - 状态机实现（事件驱动状态转换）
- `utils_tlv` - TLV（Type-Length-Value）编解码
- `utils_work_queue` - 工作队列（异步任务管理）
- `utils_mutex` - 互斥锁（线程同步）
- `utils_timer` - 定时器（定时任务管理）
- `utils_dslm_list` - DSLM 链表（设备列表管理）
- `utils_log` - 日志封装

### 2. common/ - 公共定义

**职责**：提供公共的头文件和编译配置，所有模块共享。

**核心定义**：
- `idevice_security_level.h` - IPC 接口定义（IDeviceSecurityLevel、IDeviceSecurityLevelCallback）
- `dslm_service_ipc_interface_code.h` - IPC 命令码（CMD_GET_DEVICE_SECURITY_LEVEL = 1）

### 3. interfaces/inner_api/ - 对外接口

**职责**：提供对外 C API（Native 接口）和 IPC Proxy。

**C API 实现**（`interfaces/inner_api/src/standard/device_security_info.cpp`）：
- `RequestDeviceSecurityInfo()` - 同步查询
- `RequestDeviceSecurityInfoAsync()` - 异步查询
- `FreeDeviceSecurityInfo()` - 释放
- `GetDeviceSecurityLevelValue()` - 提取等级

**IPC Proxy**（`device_security_level_proxy.cpp`）：
- `DeviceSecurityLevelProxy` - 客户端代理，通过 IPC 调用 SA
- `DeviceSecurityLevelCallbackStub` - 客户端接收回调的 Stub

**SA 加载器**（`device_security_level_loader.cpp`）：
- `DeviceSecurityLevelLoader` - SA 加载器，通过 SystemAbilityManagerClient 加载 SA 3511

**Lite 实现**：
- **Small**: C 实现的 IPC Proxy（`device_security_level_proxy.c`）
- **Mini**: C 实现的内部 API（`device_security_level_inner.c`），无 IPC

### 4. oem_property/ - OEM 适配层

**职责**：实现设备安全等级的评估逻辑，不同系统形态有不同的实现。

**核心接口**（`oem_property/include/dslm_credential.h`）：
```c
typedef int32_t (*GetDeviceCred)(const DeviceIdentify *device, DslmCredBuff *credBuff);
typedef int32_t (*VerifyDslmCred)(const DeviceIdentify *device, uint64_t challenge,
    const DslmCredBuff *credBuff, DslmCredInfo *credInfo);
```

**OHOS 标准实现**（`oem_property/ohos/common/`）：
- `dslm_ohos_request.c` - 从目标设备获取凭据（通过 DSoftBus）
- `dslm_ohos_verify.c` - 验证凭据（证书链 + ECDSA 签名）
- `external_interface_adapter.c` - 外部接口适配（DeviceManager、Huks）

**凭据验证流程**：
1. 验证证书链完整性（Root → Intermediate → Leaf）
2. 验证每个证书的 ECDSA 签名（SHA256/SHA384）
3. 验证 Challenge-Nonce 匹配
4. 提取安全等级

### 5. services/ - 服务框架

#### 5.1 sa/ - SA 服务

**职责**：实现 DslmService (System Ability 3511)，处理 IPC 请求。

**Standard 实现**（`services/sa/standard/dslm_service.cpp`）：
- `DslmService` 类，继承 `SystemAbility` 和 `IRemoteStub<IDeviceSecurityLevel>`
- `OnStart()` - 服务启动（独立线程初始化）
- `OnStop()` - 服务停止
- `OnRemoteRequest()` - 处理 IPC 请求（CMD_GET_DEVICE_SECURITY_LEVEL）
- `RequestDeviceSecurityLevel()` - 请求设备安全等级（调用核心逻辑）
- **插件支持**：支持动态加载插件（`PLUGIN_SO_PATH`）
- **自动卸载**：10 秒无请求后自动卸载 SA

**Lite 实现**（`services/sa/lite/`）：
- **Small**: 使用 Lite SAMGR，支持 Binder IPC
- **Mini**: 静态库，无 IPC（内部调用）

#### 5.2 dslm/ - DSLM 核心逻辑

**职责**：实现设备安全等级管理的核心逻辑。

**核心文件**：
- `dslm_device_list.c` - 设备列表管理（增删查）
- `dslm_fsm_process.c` - 状态机（设备状态转换）
- `dslm_inner_process.c` - 内部处理流程
- `dslm_msg_utils.c` - 消息工具
- `dslm_core_process.c` - 核心处理

**设备信息结构**（`dslm_core_defines.h:38-61`）：
```c
typedef struct DslmDeviceInfo {
    ListNode linkNode;            // 链表节点
    StateMachine machine;         // 状态机
    DeviceIdentify identity;     // 设备标识
    uint32_t version;            // 版本
    uint32_t onlineStatus;        // 在线状态
    uint64_t nonce;             // 挑战码
    uint64_t nonceTimeStamp;     // 挑战码时间戳
    uint64_t lastOnlineTime;     // 最后在线时间
    uint64_t lastOfflineTime;    // 最后离线时间
    uint64_t lastRequestTime;    // 最后请求时间
    uint64_t lastResponseTime;   // 最后响应时间
    uint64_t lastVerifyTime;     // 最后验证时间
    uint64_t transNum;          // 传输次数
    TimerHandle timeHandle;       // 定时器句柄
    uint32_t queryTimes;         // 查询次数
    uint32_t result;             // 结果（安全等级）
    DslmCredInfo credInfo;     // 凭据信息
    uint32_t notifyListSize;      // 通知列表大小
    ListHead notifyList;         // 通知列表
    uint32_t historyListSize;     // 历史列表大小
    ListHead historyList;         // 历史列表
    uint32_t osType;            // 操作系统类型
} DslmDeviceInfo;
```

#### 5.3 msg/ - 消息处理

**职责**：封装 messenger 库，提供 DSLM 专用的消息接口。

#### 5.4 dfx/ - DFX 调测

**职责**：提供诊断、调测、统计、跟踪能力。

**DFX 组件**：
- `dslm_bigdata.cpp` - 大数据统计（上报性能指标）
- `dslm_hidumper.cpp` - HiDumper 支持（`hidumper -s 3511 -l`）
- `dslm_hitrace.cpp` - HiTrace 支持（性能追踪）
- `dslm_dfx_default.c` - 默认 DFX 实现

#### 5.5 common/ - 公共服务代码

**职责**：提供公共的服务实现代码。

**文件**：
- `dslm_crypto.c` - 加密工具
- `dslm_msg_serialize.c` - 消息序列化

### 6. profile/ - 组件配置

**职责**：提供组件编译和运行时配置。

**配置文件**：
- `dslm_service.xml` - SA 配置（XML 格式）
- `dslm_service.json` - SA 配置（JSON 格式）
- `dslm_service.cfg` - 权限配置（UID/GID/APL/SELinux/Permissions）
- `dslm_service.rc` - 启动脚本（init 进程配置）

## 模块依赖关系

### 层级依赖

```
应用层（Apps）
    ↓ 调用
对外接口层（interfaces/inner_api）
    ↓ 依赖
SA 服务层（services/sa）
    ↓ 调用
核心逻辑层（services/dslm）
    ↓ 调用
OEM 适配层（oem_property）
    ↓ 依赖
基础库层（baselib）
```

### 关键依赖

| 上层 | 依赖的下层 | 说明 |
|------|-----------|------|
| **interfaces** | services (SA) | 通过 IPC Proxy 调用 SA |
| **services/sa** | services/dslm | SA 调用核心逻辑 |
| **services/dslm** | oem_property | 核心逻辑调用 OEM 凭据验证 |
| **oem_property** | baselib | OEM 层使用基础库工具 |
| **所有模块** | common | 公共定义被所有模块共享 |

## 关键结论

1. **baselib/** - 基础库：消息通信（msglib）+ 工具函数（utils）
2. **common/** - 公共定义：IPC 接口、SA ID、错误码
3. **interfaces/** - 对外接口：C API + IPC Proxy（支持 Standard/Small/Mini）
4. **oem_property/** - OEM 适配：设备凭据验证（证书链 + ECDSA 签名）
5. **services/** - 服务框架：
   - sa/ - SA 服务（DslmService, SA ID 3511）
   - dslm/ - 核心逻辑（设备列表、状态机）
   - msg/ - 消息封装
   - dfx/ - 调测支持
6. **profile/** - 组件配置：SA 配置、权限配置、启动脚本

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 系统架构
- [05_Inner_APIs.md](./05_Inner_APIs.md) - 内部接口详解
