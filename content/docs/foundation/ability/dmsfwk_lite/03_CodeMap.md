# 03_目录结构与代码地图

> dmsfwk_lite 项目的目录结构、核心文件定位及代码导航指南，帮助开发者快速找到目标代码位置。

## 一、目录结构总览

```
dmsfwk_lite/
├── interfaces/
│   └── innerkits/
│       └── dmsfwk_interface.h     # 对外接口定义
├── include/
│   ├── dmslite.h                   # 服务入口定义
│   ├── dmslite_famgr.h             # FA 管理模块接口
│   ├── dmslite_feature.h           # Feature 接口定义
│   ├── dmslite_permission.h        # 权限校验接口
│   ├── dmslite_session.h           # 会话管理接口
│   ├── dmslite_parser.h            # TLV 解析接口
│   ├── dmslite_packet.h            # 消息封装接口
│   ├── dmslite_tlv_common.h        # TLV 公共定义
│   ├── dmslite_msg_handler.h       # 消息处理接口
│   ├── dmslite_devmgr.h            # 设备管理接口
│   ├── dmslite_utils.h             # 工具宏与内联函数
│   └── dmslite_log.h               # 日志宏定义
├── source/
│   ├── dmslite.c                   # 服务入口实现
│   ├── dmslite_famgr.c             # FA 管理模块
│   ├── dmslite_feature.c           # Feature 实现
│   ├── dmslite_permission.c         # 权限校验模块
│   ├── dmslite_session.c           # 会话管理模块
│   ├── dmslite_parser.c            # TLV 解析模块
│   ├── dmslite_packet.c             # 消息封装模块
│   ├── dmslite_tlv_common.c        # TLV 公共实现
│   ├── dmslite_msg_handler.c        # 消息处理模块
│   └── dmslite_devmgr.c            # 设备管理模块
├── BUILD.gn                        # 构建配置
├── bundle.json                     # 组件配置
└── figures/                        # 架构图资源
```

---

## 二、目录职责说明

### 2.1 接口目录（interfaces/）

| 目录/文件 | 职责 | 说明 |
|----------|------|------|
| `interfaces/innerkits/` | 对外 Inner Kit 接口 | 供其他子系统调用 |
| `dmsfwk_interface.h` | 核心接口定义 | 服务名、错误码、回调定义 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:1-79`

### 2.2 头文件目录（include/）

| 文件 | 职责 | 关键内容 |
|-----|------|---------|
| `dmslite.h` | 服务入口 | `DistributedService` 结构定义 |
| `dmslite_famgr.h` | FA 管理 | `StartRemoteAbility()` 声明 |
| `dmslite_feature.h` | Feature 接口 | `DmsLite` 结构、`StartRemoteAbilityInner()` |
| `dmslite_permission.h` | 权限校验 | `CheckRemotePermission()` 声明 |
| `dmslite_session.h` | 会话管理 | `SendDmsMessage()` 声明 |
| `dmslite_parser.h` | TLV 解析 | `ProcessCommuMsg()` 声明 |
| `dmslite_packet.h` | 消息封装 | `Marshall*()` 系列函数 |
| `dmslite_tlv_common.h` | TLV 公共定义 | TLV 节点结构、字段类型枚举 |
| `dmslite_msg_handler.h` | 消息处理 | `StartAbilityFromRemoteHandler()` |
| `dmslite_devmgr.h` | 设备管理 | 设备上下线回调 |
| `dmslite_utils.h` | 工具函数 | 字节序转换、内存分配宏 |
| `dmslite_log.h` | 日志 | `HILOGE`、`HILOGI` 等宏 |

### 2.3 源文件目录（source/）

| 文件 | 职责 | 代码行数 |
|-----|------|---------|
| `dmslite.c` | 服务入口 | 81 |
| `dmslite_famgr.c` | FA 管理 | 231 |
| `dmslite_feature.c` | Feature 实现 | 126 |
| `dmslite_permission.c` | 权限校验 | 223 |
| `dmslite_session.c` | 会话管理 | 258 |
| `dmslite_parser.c` | TLV 解析 | 295 |
| `dmslite_packet.c` | 消息封装 | 225 |
| `dmslite_tlv_common.c` | TLV 公共 | 134 |
| `dmslite_msg_handler.c` | 消息处理 | 50 |
| `dmslite_devmgr.c` | 设备管理 | ~50 |

---

## 三、核心文件定位

### 3.1 远程启动入口

| 功能 | 文件 | 行号 | 函数 |
|-----|------|-----|------|
| 对外接口 | `dmslite_feature.c` | 42 | `StartRemoteAbilityInner()` |
| 内部入口 | `dmslite_famgr.c` | 85 | `StartRemoteAbility()` |
| 参数校验 | `dmslite_famgr.c` | 88-90 | 参数 NULL 检查 |
| 消息封装 | `dmslite_famgr.c` | 112 | `MarshallDmsMessage()` |
| 消息发送 | `dmslite_session.c` | 194 | `SendDmsMessage()` |

**证据来源**：`source/dmslite_famgr.c:85-110` 完整流程。

### 3.2 消息解析入口

| 功能 | 文件 | 行号 | 函数 |
|-----|------|-----|------|
| 字节接收 | `dmslite_session.c` | 72 | `OnBytesReceived()` |
| 消息处理 | `dmslite_parser.c` | 256 | `ProcessCommuMsg()` |
| TLV 解析 | `dmslite_parser.c` | 165 | `TlvBytesToNode()` |
| 命令分发 | `dmslite_parser.c` | 278-292 | `switch(commandId)` |

**证据来源**：`source/dmslite_parser.c:256-294` 消息处理。

### 3.3 权限校验入口

| 功能 | 文件 | 行号 | 函数 |
|-----|------|-----|------|
| 远程校验 | `dmslite_permission.c` | 63 | `CheckRemotePermission()` |
| 签名比对 | `dmslite_permission.c` | 111 | `strcmp()` |
| 包信息获取 | `dmslite_permission.c` | 202 | `GetCallerBundleInfo()` |
| 文件读取 | `dmslite_permission.c` | 118 | `GetBundleInfoFromFile()` |

**证据来源**：`source/dmslite_permission.c:63-116` 权限校验。

### 3.4 服务注册入口

| 功能 | 文件 | 行号 | 函数 |
|-----|------|-----|------|
| 服务注册 | `dmslite.c` | 77 | `RegisterService()` |
| Feature 注册 | `dmslite_feature.c` | 114 | `RegisterFeature()` |
| API 注册 | `dmslite_feature.c` | 119 | `RegisterFeatureApi()` |

**证据来源**：`source/dmslite_feature.c:112-125` Feature 初始化。

---

## 四、代码导航图

### 4.1 按功能导航

| 功能需求 | 目标文件 | 关键函数 | 行号 |
|---------|---------|---------|------|
| **远程启动** | `dmslite_famgr.c` | `StartRemoteAbility()` | 85 |
| **权限校验** | `dmslite_permission.c` | `CheckRemotePermission()` | 63 |
| **会话管理** | `dmslite_session.c` | `SendDmsMessage()` | 194 |
| **TLV 解析** | `dmslite_parser.c` | `ProcessCommuMsg()` | 256 |
| **消息封装** | `dmslite_packet.c` | `MarshallString()` | 119 |
| **服务注册** | `dmslite_feature.c` | `Init()` | 112 |
| **错误码** | `interfaces/innerkits/dmsfwk_interface.h` | 枚举定义 | 31-55 |

### 4.2 按模块导航

#### FA 管理模块

```
入口函数：StartRemoteAbility()
    │
    ├── 参数校验 (dmslite_famgr.c:88)
    ├── FillRequestData() (dmslite_famgr.c:171)
    │   ├── SetElementBundleName()
    │   ├── SetElementAbilityName()
    │   └── SetElementDeviceID()
    ├── MarshallDmsMessage() (dmslite_famgr.c:112)
    │   ├── PACKET_MARSHALL_HELPER()
    │   └── GetCallerBundleInfo()
    └── SendDmsMessage() → dmslite_session.c:194
```

#### 会话管理模块

```
入口函数：SendDmsMessage()
    │
    ├── CreateDMSSessionServer() (dmslite_session.c:184)
    │   └── CreateSessionServer() [DSoftBus]
    ├── OpenSession() (dmslite_session.c:214)
    │   └── OpenSession() [DSoftBus]
    ├── OnBytesReceived() (dmslite_session.c:72) [回调]
    ├── OnSessionOpened() (dmslite_session.c:137) [回调]
    └── OnSessionClosed() (dmslite_session.c:113) [回调]
```

#### 权限校验模块

```
入口函数：CheckRemotePermission()
    │
    ├── GetBundleInfo() (dmslite_permission.c:86)
    │   └── BMS 接口调用
    └── strcmp() 签名比对 (dmslite_permission.c:111)
        │
        ├── 获取调用方签名
        └── 获取被调用方签名
```

#### TLV 解析模块

```
入口函数：ProcessCommuMsg()
    │
    ├── CanCall() (dmslite_parser.c:243)
    │   └── getuid() 检查
    ├── TlvBytesToNode() (dmslite_parser.c:165)
    │   ├── TlvFillNode() (dmslite_parser.c:80)
    │   ├── TlvBytesToLength() (dmslite_parser.c:49)
    │   └── CheckNodeSequence() (dmslite_parser.c:139)
    └── switch(commandId) (dmslite_parser.c:278)
        ├── DMS_MSG_CMD_START_FA
        │   └── StartAbilityFromRemoteHandler()
        └── DMS_MSG_CMD_REPLY
            └── ReplyMsgHandler()
```

---

## 五、关键数据结构

### 5.1 服务与 Feature

| 结构体 | 文件 | 说明 |
|-------|------|------|
| `DistributedService` | `dmslite.h:27-30` | SAMgr 服务结构 |
| `DmsLite` | `dmslite_feature.h` | Feature 结构，包含回调 |

**证据来源**：`include/dmslite.h:27-30`

```cpp
typedef struct {
    INHERIT_SERVICE;
    Identity identity;
} DistributedService;
```

### 5.2 消息结构

| 结构体 | 文件 | 说明 |
|-------|------|------|
| `RequestData` | `dmslite_famgr.h:32-36` | 启动请求数据 |
| `CommuMessage` | `dmslite_tlv_common.h:60-63` | 通信消息 |
| `PermissionCheckInfo` | `dmslite_inner_common.h:54-58` | 权限检查信息 |

### 5.3 TLV 结构

| 结构体 | 文件 | 说明 |
|-------|------|------|
| `TlvNode` | `dmslite_tlv_common.h:32-37` | TLV 节点 |
| `FieldType` | `dmslite_tlv_common.h:50-58` | 字段类型枚举 |

**证据来源**：`include/dmslite_tlv_common.h:32-58`

---

## 六、全局变量

### 6.1 服务状态

| 变量 | 文件 | 行号 | 类型 | 说明 |
|-----|------|-----|------|------|
| `g_distributedService` | `dmslite.c` | 33-38 | `DistributedService` | 服务实例 |
| `g_dmslite` | `dmslite_feature.c` | 33-44 | `DmsLite` | Feature 实例 |
| `g_dmsFeatureCallback` | `dmslite_session.c` | 61-65 | `IDmsFeatureCallback` | 解析回调 |

### 6.2 会话状态

| 变量 | 文件 | 行号 | 类型 | 说明 |
|-----|------|-----|------|------|
| `g_curSessionId` | `dmslite_session.c` | 40 | `int32_t` | 当前会话 ID |
| `g_curBusy` | `dmslite_session.c` | 41 | `bool` | 忙标志 |
| `g_begin` | `dmslite_session.c` | 42 | `time_t` | 会话开始时间 |
| `g_listener` | `dmslite_session.c` | 43 | `IDmsListener*` | 结果回调 |

### 6.3 消息缓冲区

| 变量 | 文件 | 行号 | 类型 | 说明 |
|-----|------|-----|------|------|
| `g_buffer` | `dmslite_packet.c` | 42 | `char[1024]` | TLV 消息缓冲区 |
| `g_counter` | `dmslite_packet.c` | 43 | `uint16_t` | 缓冲区偏移 |

**证据来源**：`source/dmslite_packet.c:42-43`

---

## 七、宏定义速查

### 7.1 消息相关

| 宏 | 文件 | 行号 | 值 | 说明 |
|---|------|-----|-----|------|
| `DMS_VERSION_VALUE` | `dmslite_famgr.c` | 32 | 200 | 协议版本 |
| `MAX_DATA_SIZE` | `dmslite_session.c` | 37 | 1024 | 最大数据长度 |
| `TIMEOUT` | `dmslite_session.c` | 38 | 60 | 会话超时（秒） |
| `DMS_MSG_CMD_START_FA` | `dmslite_tlv_common.c` | 66 | 0x01 | 启动 FA 命令 |

### 7.2 路径相关

| 宏 | 文件 | 行号 | 值 | 说明 |
|---|------|-----|-----|------|
| `NATIVE_APPID_DIR` | `dmslite_permission.c` | 37 | `/system/native_appid/` | native appId 目录 |
| `APPID_FILE_PREFIX` | `dmslite_permission.c` | 38 | `uid_` | 文件名前缀 |
| `APPID_FILE_SUFFIX` | `dmslite_permission.c` | 39 | `_appid` | 文件名后缀 |

---

## 八、常见查找任务

| 任务 | 搜索关键词 | 目标文件 |
|-----|-----------|---------|
| 查找错误码定义 | `DMS_EC_` | `interfaces/innerkits/dmsfwk_interface.h` |
| 查找 TLV 字段类型 | `FieldType` | `include/dmslite_tlv_common.h` |
| 查找权限校验 | `CheckRemotePermission` | `source/dmslite_permission.c` |
| 查找会话管理 | `SendDmsMessage` | `source/dmslite_session.c` |
| 查找消息解析 | `ProcessCommuMsg` | `source/dmslite_parser.c` |
| 查找服务注册 | `RegisterService` | `source/dmslite.c` |
| 查找日志输出 | `HILOG` | 多文件 |

---

## 九、相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |
| [04_Interface.md](./04_Interface.md) | 对外接口 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*
