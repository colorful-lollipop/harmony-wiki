# 02 - 目录结构与模块职责

## 目的

本文档描述 Cellular Call 模块的**目录组织结构和各模块职责**，帮助开发者快速定位代码位置，理解模块划分依据。

## 适用范围

- **读者对象**：需要了解代码结构的开发者
- **使用场景**：
  - 查找特定功能代码
  - 理解模块边界
  - 新人代码定位

---

## 2.1 顶层目录结构

```
cellular_call/                                      # 模块根目录
├── 📄 BUILD.gn                                    # 根构建配置
├── 📄 bundle.json                                 # 组件配置文件
├── 📄 cellularcall.gni                            # GN 构建参数
├── 📄 LICENSE                                      # Apache 2.0 许可证
├── 📄 README.md / README_zh.md                    # 项目说明文档
│
├── 📁 figures/                                    # 文档图片资源
│   └── en-us_architecture-of-the-cellular-call-module.png
│
├── 📁 interfaces/                                 # 🔷 对外接口层
│   └── 📁 innerkits/
│       ├── 📁 ims/                               # IMS 通话接口
│       │   ├── ims_call_interface.h             # IMS 通话主接口
│       │   ├── ims_call_types.h                 # 类型定义
│       │   ├── ims_call_client.h                # IMS 客户端
│       │   ├── ims_call_proxy.h                 # IMS 代理
│       │   ├── ims_call_callback_*.h           # 回调接口
│       │   └── BUILD.gn                         # 构建配置
│       │
│       ├── 📁 ims_common/                       # IMS 公共接口
│       │   └── ims_feature.h
│       │
│       └── 📁 satellite/                         # 卫星通话接口（条件编译）
│           ├── satellite_call_interface.h       # 卫星通话主接口
│           ├── satellite_call_types.h           # 类型定义
│           ├── satellite_call_client.h          # 卫星客户端
│           ├── satellite_call_callback_*.h     # 回调接口
│           └── BUILD.gn                         # 构建配置
│
├── 📁 sa_profile/                                 # 🔷 SA 配置文件
│   ├── 4006.json                                 # SA 静态配置
│   ├── 4006_dynamic.json                         # SA 动态配置
│   └── BUILD.gn                                  # 构建配置
│
├── 📁 services/                                   # 🔷 核心服务层
│   ├── 📁 common/                                 # 公共工具模块
│   │   ├── 📁 include/
│   │   │   ├── base_request.h                   # 请求基类
│   │   │   ├── cellular_call_hisysevent.h       # HiSysEvent 埋点
│   │   │   ├── cellular_call_rdb_helper.h     # RDB 数据库
│   │   │   ├── mmi_code_message.h              # MMI 码消息
│   │   │   ├── supplement_request_*.h          # 补充业务请求
│   │   │   └── ...
│   │   │
│   │   └── 📁 src/
│   │       ├── base_request.cpp
│   │       ├── cellular_call_hisysevent.cpp
│   │       ├── cellular_call_rdb_helper.cpp
│   │       ├── mmi_code_message.cpp
│   │       ├── supplement_request_cs.cpp
│   │       └── supplement_request_ims.cpp
│   │
│   ├── 📁 manager/                                # 🔷 管理层
│   │   ├── 📁 include/
│   │   │   ├── cellular_call_service.h         # 主服务入口
│   │   │   ├── cellular_call_stub.h            # IPC 存根
│   │   │   ├── cellular_call_handler.h         # 事件处理器
│   │   │   ├── cellular_call_register.h       # 观察者注册
│   │   │   └── cellular_call_callback.h       # 回调接口
│   │   │
│   │   └── 📁 src/
│   │       ├── cellular_call_service.cpp       # 服务实现
│   │       ├── cellular_call_stub.cpp          # IPC 实现
│   │       ├── cellular_call_handler.cpp      # 事件处理
│   │       ├── cellular_call_register.cpp     # 注册管理
│   │       └── cellular_call_callback.cpp     # 回调处理
│   │
│   ├── 📁 control/                                # 🔷 控制层
│   │   ├── 📁 include/
│   │   │   ├── control_base.h                  # 控制基类
│   │   │   ├── cs_control.h                    # CS 通话控制
│   │   │   ├── ims_control.h                   # IMS 通话控制
│   │   │   ├── ims_video_call_control.h       # 视频通话控制
│   │   │   └── satellite_control.h             # 卫星通话控制
│   │   │
│   │   └── 📁 src/
│   │       ├── control_base.cpp
│   │       ├── cs_control.cpp
│   │       ├── ims_control.cpp
│   │       ├── ims_video_call_control.cpp
│   │       └── satellite_control.cpp
│   │
│   ├── 📁 connection/                              # 🔷 连接层
│   │   ├── 📁 include/
│   │   │   ├── base_connection.h               # 连接基类
│   │   │   ├── cellular_call_connection_cs.h   # CS 连接
│   │   │   ├── cellular_call_connection_ims.h  # IMS 连接
│   │   │   └── cellular_call_connection_satellite.h  # 卫星连接
│   │   │
│   │   └── 📁 src/
│   │       ├── base_connection.cpp
│   │       ├── cellular_call_connection_cs.cpp
│   │       ├── cellular_call_connection_ims.cpp
│   │       └── cellular_call_connection_satellite.cpp
│   │
│   ├── 📁 ims_service_interaction/               # 🔷 IMS 服务交互层
│   │   ├── ims_call_client.cpp                   # IMS 客户端实现
│   │   ├── ims_call_proxy.cpp                    # IMS 代理
│   │   ├── ims_call_callback_stub.cpp           # IMS 回调存根
│   │   └── ...
│   │
│   ├── 📁 satellite_service_interaction/         # 🔷 卫星服务交互层
│   │   ├── satellite_call_client.cpp            # 卫星客户端
│   │   ├── satellite_call_proxy.cpp             # 卫星代理
│   │   ├── satellite_call_callback_stub.cpp     # 卫星回调
│   │   └── ...
│   │
│   ├── 📁 telephony_ext_wrapper/                  # 🔷 电话扩展包装
│   │   ├── telephony_ext_wrapper.h
│   │   └── telephony_ext_wrapper.cpp
│   │
│   └── 📁 utils/                                  # 🔷 工具模块
│       ├── 📁 include/
│       │   ├── cellular_call_config.h            # 配置管理
│       │   ├── cellular_call_dump_helper.cpp   # Dump 工具
│       │   ├── cellular_call_supplement.h       # 补充业务
│       │   ├── emergency_utils.h               # 紧急呼叫
│       │   ├── mmi_code_utils.h                # MMI 码工具
│       │   ├── standardize_utils.h              # 标准化工具
│       │   └── ...
│       │
│       └── 📁 src/
│           ├── cellular_call_config.cpp
│           ├── cellular_call_dump_helper.cpp
│           ├── emergency_utils.cpp
│           ├── mmi_code_utils.cpp
│           └── ...
│
├── 📁 vendor/                                     # 样例/厂商代码
│   └── 📁 ims/
│       ├── 📁 services/
│       │   ├── ims_base/
│       │   ├── ims_call/
│       │   ├── ims_core_service/
│       │   └── ims_sms/
│       └── 📁 sa_profile/
│
└── 📁 test/                                       # ⚠️ 测试代码（不作为业务证据）
    ├── unittest/                                  # 单元测试
    └── fuzztest/                                 # Fuzz 测试
```

---

## 2.2 模块职责说明

### 2.2.1 接口层 (interfaces/)

**职责**：定义 Inner API 接口，供其他模块调用

| 模块 | 职责 | 主要文件 |
|------|------|----------|
| **ims/** | IMS 通话接口定义 | `ims_call_interface.h`, `ims_call_types.h`, `ims_call_client.h` |
| **ims_common/** | IMS 公共定义 | `ims_feature.h` |
| **satellite/** | 卫星通话接口（条件编译） | `satellite_call_interface.h`, `satellite_call_types.h` |

### 2.2.2 SA 配置层 (sa_profile/)

**职责**：定义 System Ability 元数据

| 文件 | 用途 |
|------|------|
| **4006.json** | SA 静态配置（默认使用） |
| **4006_dynamic.json** | SA 动态配置（cellular_call_dynamic_start=true 时使用） |

### 2.2.3 服务层 (services/)

#### 管理层 (manager/)

```
┌─────────────────────────────────────────────────────────────┐
│                       Manager 层                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  CellularCallService (SA 4006)                      │   │
│  │  • OnStart/OnStop 生命周期                          │   │
│  │  • IPC 接口实现                                      │   │
│  │  • 依赖 Core Service (4010)                         │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  CellularCallStub                                   │   │
│  │  • OnRemoteRequest 入口                             │   │
│  │  • 权限检查: CONNECT_CELLULAR_CALL_SERVICE         │   │
│  │  • 请求分发                                          │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  CellularCallHandler                                │   │
│  │  • RIL 回调事件处理                                 │   │
│  │  • 状态机管理                                       │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  CellularCallRegister                               │   │
│  │  • 观察者注册/通知                                  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### 控制层 (control/)

| 模块 | 职责 | 证据位置 |
|------|------|----------|
| **CSControl** | 2G/3G 电路交换通话控制 | `services/control/include/cs_control.h` |
| **IMSControl** | 4G/5G IMS 通话控制 | `services/control/include/ims_control.h` |
| **ImsVideoCallControl** | IMS 视频通话特殊控制 | `services/control/include/ims_video_call_control.h` |
| **SatelliteControl** | 卫星通话控制（条件编译） | `services/control/include/satellite_control.h` |

#### 连接层 (connection/)

```
连接层职责：
├── BaseConnection - 连接基类
├── CellularCallConnectionCS - CS 通话连接
├── CellularCallConnectionIMS - IMS 通话连接
└── CellularCallConnectionSatellite - 卫星通话连接（条件编译）
```

#### 服务交互层 (ims_service_interaction/)

```
IMS 交互层职责：
├── ImsCallClient - IMS 服务客户端
├── ImsCallProxy - IMS 代理
├── ImsCallCallbackStub - 回调存根
└── 负责与 IMS Core Service 通信
```

---

## 2.3 文件分类索引

### 2.3.1 核心实现文件 (50+ .cpp)

| 分类 | 文件数 | 用途 |
|------|--------|------|
| Manager | 5 | 服务入口、IPC 存根、事件处理 |
| Control | 5 | CS/IMS/卫星通话控制 |
| Connection | 3 | 通话连接管理 |
| Utils | 10+ | 配置、补充业务、紧急呼叫等 |
| Common | 5 | 公共工具、RDB、HiSysEvent |
| IMS Interaction | 3 | IMS 服务交互 |
| Satellite Interaction | 3 | 卫星服务交互（条件） |

### 2.3.2 接口头文件 (20+ .h)

| 分类 | 文件数 | 用途 |
|------|--------|------|
| IMS 接口 | 10 | IMS 通话主接口、类型、回调 |
| Satellite 接口 | 10 | 卫星通话接口、类型、回调 |
| Manager 接口 | 5 | 主服务、存根、处理器 |
| Control 接口 | 4 | CS/IMS/视频/卫星控制 |
| Utils 接口 | 10+ | 公共类型定义 |

---

## 2.4 快速代码定位指南

| 功能需求 | 查找位置 |
|----------|----------|
| **拨号逻辑** | `services/control/src/cs_control.cpp` 或 `ims_control.cpp` |
| **接听/挂断** | `services/control/src/cs_control.cpp:Answer()/HangUp()` |
| **IMS 交互** | `services/ims_service_interaction/` |
| **IPC 入口** | `services/manager/src/cellular_call_stub.cpp` |
| **权限检查** | `services/manager/src/cellular_call_stub.cpp:45-50` |
| **RIL 回调** | `services/manager/src/cellular_call_handler.cpp` |
| **视频通话** | `services/control/src/ims_video_call_control.cpp` |
| **补充业务** | `services/utils/src/cellular_call_supplement.cpp` |
| **紧急呼叫** | `services/utils/src/emergency_utils.cpp` |
| **卫星通话** | `services/satellite_service_interaction/` |

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 接口规范 | [04_Interfaces.md](./04_Interfaces.md) |
| 构建配置 | [06_GN_Build.md](./06_GN_Build.md) |

---

*最后更新：2026-02-06*
