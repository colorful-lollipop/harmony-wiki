# 01_Directory_Structure - 目录结构与模块职责

## 1. 顶层目录结构

```
dmsfwk_lite/
├── BUILD.gn                    # GN 构建入口
├── bundle.json                 # 组件配置（OpenHarmony 标准）
├── LICENSE                     # Apache License 2.0
├── OAT.xml                     # 开源合规检查配置
├── README.md                   # 英文 README
├── README_zh.md                # 中文 README
├── figures/                    # 文档图片
│   ├── en-us_image_0000001081284974.png
│   └── zh-cn_image_0000001081284974.png
├── include/                    # 内部头文件（15个）
├── interfaces/                 # 对外接口
│   └── innerkits/
├── source/                     # 源码实现（10个）
└── moduletest/                 # 模块测试（本文档忽略）
    └── dtbschedmgr_lite/
```

## 2. 头文件目录 (include/)

| 文件 | 职责 | 关键定义 |
|------|------|----------|
| `dmslite.h` | 服务定义 | `DistributedService` 结构体 |
| `dmslite_devmgr.h` | 设备管理接口 | `AddDevMgrListener()`, `GetPeerId()` |
| `dmslite_famgr.h` | FA 管理接口 | `StartRemoteAbility()`, `RequestData` |
| `dmslite_feature.h` | Feature 定义 | `DmsLite` 结构体, `DmsMsgType` 枚举 |
| `dmslite_inner_common.h` | 内部通用定义 | `DmsAllowedUids`, 回调类型定义 |
| `dmslite_log.h` | 日志宏 | `HILOGD/I/W/E/F` 宏 |
| `dmslite_msg_handler.h` | 消息处理器 | `StartAbilityFromRemoteHandler()` |
| `dmslite_packet.h` | 报文打包 | `Marshall*()`, `GetPacketBufPtr()` |
| `dmslite_parser.h` | TLV 解析入口 | `ProcessCommuMsg()` |
| `dmslite_permission.h` | 权限检查 | `CheckRemotePermission()` |
| `dmslite_session.h` | 会话管理 | `SendDmsMessage()`, `IsDmsBusy()` |
| `dmslite_tlv_common.h` | TLV 通用定义 | `TlvNode`, `FieldType`, `CommuMessage` |
| `dmslite_utils.h` | 工具宏 | `DMS_ALLOC/FREE`, 字节序转换 |

### 2.1 头文件依赖关系

```
dmsfwk_interface.h (对外)
    └── 被所有内部模块引用

dmslite_inner_common.h
    ├── dmslite_feature.h
    ├── dmslite_famgr.h
    ├── dmslite_parser.h
    └── dmslite_permission.h

dmslite_tlv_common.h
    ├── dmslite_packet.h
    ├── dmslite_parser.h
    └── dmslite_msg_handler.h
```

## 3. 源码目录 (source/)

| 文件 | 模块 | 职责 | 代码行数 |
|------|------|------|----------|
| `dmslite.c` | Service | 服务注册、初始化 | ~81 |
| `dmslite_feature.c` | Feature | Feature 注册、消息分发 | ~126 |
| `dmslite_famgr.c` | FA Manager | 远程启动请求处理、消息打包 | ~231 |
| `dmslite_session.c` | Session | SoftBus 会话管理、回调处理 | ~258 |
| `dmslite_parser.c` | Parser | TLV 解析、命令分发 | ~295 |
| `dmslite_permission.c` | Permission | 权限检查、签名验证 | ~223 |
| `dmslite_msg_handler.c` | Msg Handler | 消息处理器实现 | ~50 |
| `dmslite_packet.c` | Packet | 报文打包实现 | ~225 |
| `dmslite_tlv_common.c` | TLV Utils | TLV 反序列化 | ~151 |
| `dmslite_devmgr.c` | DevMgr | 设备状态监听 | ~74 |

### 3.1 模块职责详解

#### 3.1.1 Service 模块 (dmslite.c)

**职责**: 作为 SAMGR 服务注册入口

**关键函数**:
- `GetName()` - 返回服务名 "dtbschedsrv"
- `Initialize()` - 服务初始化
- `MessageHandle()` - 服务级消息处理（当前仅打印日志）
- `GetTaskConfig()` - 任务配置（高优先级、普通优先级、4KB 栈、20 消息队列）
- `Init()` - 注册服务（`SYS_SERVICE_INIT` 宏标记）

**代码位置**: `source/dmslite.c:40-80`

#### 3.1.2 Feature 模块 (dmslite_feature.c)

**职责**: 实现 DmsLite Feature，处理业务消息

**关键函数**:
- `GetName()` - 返回 Feature 名 "dmslite"
- `OnInitialize()` - Feature 初始化
- `OnMessage()` - 消息分发处理
- `Init()` - 注册 Feature 和 API（`SYS_FEATURE_INIT` 宏标记）

**消息类型处理** (`OnMessage`):
```c
START_REMOTE_ABILITY  → 调用 StartRemoteAbility()
SESSION_OPEN          → 调用 HandleSessionOpened()
SESSION_CLOSE         → 调用 HandleSessionClosed()
BYTES_RECEIVED        → 调用 HandleBytesReceived()
```

**代码位置**: `source/dmslite_feature.c:73-109`

#### 3.1.3 FA Manager 模块 (dmslite_famgr.c)

**职责**: 处理远程 FA 启动请求，构建发送消息

**关键函数**:
- `StartRemoteAbilityInner()` - 内部接口入口，异步发送请求
- `StartRemoteAbility()` - 实际处理函数，构建并发送消息
- `StartAbilityFromRemote()` - 接收端处理（当前为空实现）
- `MarshallDmsMessage()` - 构建 TLV 消息
- `FillRequestData()` - 填充请求数据结构
- `FreeRequestData()` - 释放请求数据

**调用链**:
```
StartRemoteAbilityInner()
  └── StartRemoteAbility()
      ├── IsDmsBusy() 检查
      ├── MarshallDmsMessage() 构建消息
      └── SendDmsMessage() 发送
```

**代码位置**: `source/dmslite_famgr.c:45-110`

#### 3.1.4 Session 模块 (dmslite_session.c)

**职责**: 管理 SoftBus 会话，处理网络事件

**关键函数**:
- `CreateDMSSessionServer()` - 创建会话服务器
- `CloseDMSSessionServer()` - 关闭会话服务器
- `SendDmsMessage()` - 发送消息到远程设备
- `OnBytesReceived()` - 收到数据回调（SoftBus）
- `OnSessionOpened()` - 会话建立回调
- `OnSessionClosed()` - 会话关闭回调
- `IsDmsBusy()` - 检查是否忙（含超时处理）

**全局状态**:
```c
g_curSessionId  - 当前会话 ID
g_curBusy       - 是否忙
g_begin         - 会话开始时间（用于超时）
g_listener      - 结果回调监听器
```

**代码位置**: `source/dmslite_session.c:40-43`

#### 3.1.5 Parser 模块 (dmslite_parser.c)

**职责**: TLV 消息解析和命令分发

**关键函数**:
- `ProcessCommuMsg()` - 入口函数，解析并分发消息
- `CanCall()` - UID 白名单检查
- `Parse()` - 解析 TLV 消息
- `TlvBytesToNode()` - 字节流转 TLV 节点链表
- `TlvFillNode()` - 填充单个 TLV 节点

**安全特性**:
- UID 检查（仅 foundation/shell 可调用）
- 节点类型序列检查（必须严格递增）
- 最小节点数检查（至少 2 个节点）

**代码位置**: `source/dmslite_parser.c:243-295`

#### 3.1.6 Permission 模块 (dmslite_permission.c)

**职责**: 远程调用权限检查

**关键函数**:
- `CheckRemotePermission()` - 检查远程权限
- `GetCallerBundleInfo()` - 获取调用方 Bundle 信息
- `GetBundleInfoFromFile()` - 从文件读取 Native 服务信息
- `GetBundleInfoFromBms()` - 从 BMS 查询 Bundle 信息
- `GetBmsInterface()` - 获取 BMS 接口

**权限检查流程**:
1. 获取被调用方 BundleInfo
2. 从 appId 提取 signature
3. 比对 callerSignature 与 calleeSignature

**代码位置**: `source/dmslite_permission.c:63-116`

#### 3.1.7 Message Handler 模块 (dmslite_msg_handler.c)

**职责**: 处理具体业务消息

**关键函数**:
- `StartAbilityFromRemoteHandler()` - 处理 START_FA 命令
- `ReplyMsgHandler()` - 处理 REPLY 命令

**START_FA 处理**:
1. 解包 calleeBundleName/calleeAbilityName/callerSignature
2. 调用 CheckRemotePermission() 权限检查
3. 调用 StartAbilityFromRemote() 启动 FA

**代码位置**: `source/dmslite_msg_handler.c:25-41`

#### 3.1.8 Packet 模块 (dmslite_packet.c)

**职责**: 报文打包（序列化）

**关键函数**:
- `PreprareBuild()` - 初始化打包缓冲区
- `CleanBuild()` - 清理打包缓冲区
- `MarshallUint8/16/32/64()` - 打包整数
- `MarshallInt8/16/32/64()` - 打包有符号整数
- `MarshallString()` - 打包字符串
- `MarshallRawData()` - 打包原始数据
- `GetPacketSize()` - 获取包大小
- `GetPacketBufPtr()` - 获取包缓冲区指针

**全局缓冲区**:
```c
static char g_buffer[PACKET_DATA_SIZE];  // 1024 字节
static uint16_t g_counter;               // 当前写入位置
```

**代码位置**: `source/dmslite_packet.c:42-43`

#### 3.1.9 TLV Common 模块 (dmslite_tlv_common.c)

**职责**: TLV 反序列化工具函数

**关键函数**:
- `GetNodeByType()` - 按类型查找节点
- `UnMarshallUint8/16/32/64()` - 解包整数
- `UnMarshallInt8/16/32/64()` - 解包有符号整数
- `UnMarshallString()` - 解包字符串（检查 '\0' 结尾）

**字节序处理**:
- 自动检测大端/小端
- 提供 Big2Little/Little2Big 转换

**代码位置**: `source/dmslite_tlv_common.c:28-151`

#### 3.1.10 DevMgr 模块 (dmslite_devmgr.c)

**职责**: 监听设备状态变化

**关键函数**:
- `AddDevMgrListener()` - 注册设备状态监听
- `UnRegisterDevMgrListener()` - 注销监听
- `onNodeOnline()` - 设备上线回调
- `onNodeOffline()` - 设备离线回调
- `GetPeerId()` - 获取对端设备 ID

**监听事件**:
- `EVENT_NODE_STATE_ONLINE` - 设备上线
- `EVENT_NODE_STATE_OFFLINE` - 设备离线

**代码位置**: `source/dmslite_devmgr.c:30-35`

## 4. 对外接口目录 (interfaces/innerkits/)

| 文件 | 职责 | 关键定义 |
|------|------|----------|
| `dmsfwk_interface.h` | 对外 C/C++ 接口 | `DmsProxy`, `DmsLiteCommonErrorCode`, `IDmsListener` |

### 4.1 接口内容

```c
// 服务/Feature 名称
#define DISTRIBUTED_SCHEDULE_SERVICE "dtbschedsrv"
#define DMSLITE_FEATURE "dmslite"

// 错误码枚举 DmsLiteCommonErrorCode
// 回调接口 IDmsListener
// 代理接口 DmsProxy (继承 IUnknown)
```

**代码位置**: `interfaces/innerkits/dmsfwk_interface.h`

## 5. 文件组织原则

### 5.1 命名规范

| 类型 | 命名模式 | 示例 |
|------|----------|------|
| 头文件 | `dmslite_{module}.h` | `dmslite_session.h` |
| 源文件 | `dmslite_{module}.c` | `dmslite_session.c` |
| 结构体 | 大驼峰 | `DistributedService`, `DmsLite` |
| 函数 | 大驼峰 | `StartRemoteAbility()` |
| 宏/常量 | 全大写下划线 | `MAX_DATA_SIZE`, `DMS_EC_SUCCESS` |

### 5.2 模块划分原则

1. **单一职责**: 每个模块负责一个明确的功能域
2. **接口隔离**: 对外接口集中在 `interfaces/innerkits/`
3. **依赖单向**: 内部模块可依赖对外接口，反之不可
4. **平台抽象**: 通过宏隔离平台差异（`WEARABLE_PRODUCT`, `__LINUX__`）

## 6. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构设计
- [04_Internal_API](04_Internal_API.md) - 内部接口详情
