# 04_Internal_API - 内部接口文档

## 1. 模块接口总览

| 模块 | 头文件 | 源文件 | 核心接口数 | 稳定性 |
|------|--------|--------|------------|--------|
| FA Manager | `dmslite_famgr.h` | `dmslite_famgr.c` | 4 | 稳定 |
| Session | `dmslite_session.h` | `dmslite_session.c` | 8 | 稳定 |
| Permission | `dmslite_permission.h` | `dmslite_permission.c` | 3 | 稳定 |
| Parser | `dmslite_parser.h` | `dmslite_parser.c` | 1 | 稳定 |
| Packet | `dmslite_packet.h` | `dmslite_packet.c` | 12 | 稳定 |
| TLV Common | `dmslite_tlv_common.h` | `dmslite_tlv_common.c` | 9 | 稳定 |
| Msg Handler | `dmslite_msg_handler.h` | `dmslite_msg_handler.c` | 2 | 稳定 |
| DevMgr | `dmslite_devmgr.h` | `dmslite_devmgr.c` | 3 | 稳定 |
| Feature | `dmslite_feature.h` | `dmslite_feature.c` | 1 | 稳定 |

## 2. FA Manager 模块 (dmslite_famgr)

### 2.1 接口列表

**位置**: `include/dmslite_famgr.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `StartAbilityFromRemote()` | 接收端启动 FA（当前空实现） | 内部 |
| `StartRemoteAbility()` | 实际处理远程启动请求 | 内部 |
| `StartRemoteAbilityInner()` | 对外接口入口（IUnknown） | 稳定 |
| `FreeRequestData()` | 释放请求数据 | 内部 |

### 2.2 StartRemoteAbility

**功能**: 实际处理远程启动请求，构建并发送消息。

**原型**:
```c
int32_t StartRemoteAbility(const Want *want, CallerInfo *callerInfo, IDmsListener *callback);
```

**参数**:
| 参数 | 说明 |
|------|------|
| `want` | 启动参数 |
| `callerInfo` | 调用方信息 |
| `callback` | 结果回调 |

**流程**:
1. 参数检查（want, element, callerInfo）
2. `IsDmsBusy()` 检查并发
3. `PreprareBuild()` 初始化打包缓冲区
4. `MarshallDmsMessage()` 构建 TLV 消息
5. `SendDmsMessage()` 发送消息

**代码位置**: `source/dmslite_famgr.c:85-110`

### 2.3 数据结构

```c
// RequestData - 内部请求数据结构
typedef struct {
    Want *want;              // Want 参数
    CallerInfo *callerInfo;  // 调用方信息
    IDmsListener *callback;  // 回调
} RequestData;
```

**位置**: `include/dmslite_famgr.h:32-36`

## 3. Session 模块 (dmslite_session)

### 3.1 接口列表

**位置**: `include/dmslite_session.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `CreateDMSSessionServer()` | 创建会话服务器 | 内部 |
| `CloseDMSSessionServer()` | 关闭会话服务器 | 内部 |
| `SendDmsMessage()` | 发送消息 | 内部 |
| `OpenDMSSession()` | 打开会话（未使用） | 内部 |
| `CloseDMSSession()` | 关闭当前会话 | 内部 |
| `InvokeCallback()` | 调用结果回调 | 内部 |
| `HandleSessionClosed()` | 处理会话关闭事件 | 内部 |
| `HandleSessionOpened()` | 处理会话建立事件 | 内部 |
| `HandleBytesReceived()` | 处理收到数据事件 | 内部 |
| `IsDmsBusy()` | 检查是否忙 | 内部 |

### 3.2 SendDmsMessage

**功能**: 发送 DMS 消息到远程设备。

**原型**:
```c
int32_t SendDmsMessage(const char *data, int32_t len, const char *deviceId, IDmsListener *callback);
```

**参数**:
| 参数 | 说明 |
|------|------|
| `data` | 消息数据 |
| `len` | 数据长度（最大 1024） |
| `deviceId` | 目标设备 ID |
| `callback` | 结果回调 |

**流程**:
1. 参数检查
2. `CreateDMSSessionServer()` 创建服务器
3. 设置全局状态（g_curBusy, g_listener, g_begin）
4. `OpenSession()` 打开 SoftBus 会话

**代码位置**: `source/dmslite_session.c:194-222`

### 3.3 会话状态管理

**全局变量**:
```c
static int32_t g_curSessionId = INVALID_SESSION_ID;  // 当前会话 ID
static bool g_curBusy = false;                        // 忙标志
static time_t g_begin;                                // 会话开始时间
static IDmsListener *g_listener = NULL;               // 回调监听器
```

**位置**: `source/dmslite_session.c:40-43`

**超时机制**:
```c
// TIMEOUT = 60 秒
bool IsDmsBusy() {
    if (g_curBusy && IsTimeout() && g_curSessionId >= 0) {
        CloseDMSSession();  // 超时自动关闭
    }
    return g_curBusy;
}
```

## 4. Permission 模块 (dmslite_permission)

### 4.1 接口列表

**位置**: `include/dmslite_permission.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `CheckRemotePermission()` | 检查远程调用权限 | 稳定 |
| `GetCallerBundleInfo()` | 获取调用方 Bundle 信息 | 内部 |

### 4.2 CheckRemotePermission

**功能**: 检查远程调用权限，基于签名验证。

**原型**:
```c
int32_t CheckRemotePermission(const PermissionCheckInfo *permissionCheckInfo);
```

**参数**:
| 参数 | 说明 |
|------|------|
| `permissionCheckInfo` | 权限检查信息（包名、Ability 名、签名） |

**流程**:
1. 通过 BMS 获取被调用方 BundleInfo
2. 从 appId 提取 signature（`appId = bundleName + "_" + signature`）
3. 比对 callerSignature 与 calleeSignature

**代码位置**: `source/dmslite_permission.c:63-116`

### 4.3 权限检查信息结构

```c
typedef struct {
    const char* calleeBundleName;   // 被调用方包名
    const char* calleeAbilityName;  // 被调用方 Ability 名
    const char* callerSignature;    // 调用方签名
} PermissionCheckInfo;
```

**位置**: `include/dmslite_inner_common.h:54-58`

### 4.4 调用方信息获取

**GetCallerBundleInfo** 支持两种模式：

1. **Native 服务** (uid <= 99): 从文件 `/system/native_appid/uid_{uid}_appid` 读取
2. **普通应用**: 通过 BMS 查询

**代码位置**: `source/dmslite_permission.c:202-222`

## 5. Parser 模块 (dmslite_parser)

### 5.1 接口列表

**位置**: `include/dmslite_parser.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `ProcessCommuMsg()` | TLV 消息解析和命令分发入口 | 稳定 |

### 5.2 ProcessCommuMsg

**功能**: 处理通信消息，解析 TLV 并分发命令。

**原型**:
```c
int32_t ProcessCommuMsg(const CommuMessage *commuMessage, const IDmsFeatureCallback *dmsFeatureCallback);
```

**参数**:
| 参数 | 说明 |
|------|------|
| `commuMessage` | 通信消息（payload + length） |
| `dmsFeatureCallback` | 回调函数集 |

**安全特性**:
1. `CanCall()` UID 白名单检查（仅 foundation/shell）
2. TLV 节点类型序列检查（必须严格递增）
3. 最小节点数检查（至少 2 个节点）

**命令分发**:
| Command ID | 处理函数 |
|------------|----------|
| `DMS_MSG_CMD_START_FA` (0x01) | `StartAbilityFromRemoteHandler()` |
| `DMS_MSG_CMD_REPLY` (0xFFFF) | `ReplyMsgHandler()` |

**代码位置**: `source/dmslite_parser.c:256-295`

## 6. Packet 模块 (dmslite_packet)

### 6.1 接口列表

**位置**: `include/dmslite_packet.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `PreprareBuild()` | 初始化打包缓冲区 | 内部 |
| `CleanBuild()` | 清理打包缓冲区 | 内部 |
| `MarshallUint8/16/32/64()` | 打包无符号整数 | 内部 |
| `MarshallInt8/16/32/64()` | 打包有符号整数 | 内部 |
| `MarshallString()` | 打包字符串 | 内部 |
| `MarshallRawData()` | 打包原始数据 | 内部 |
| `GetPacketSize()` | 获取包大小 | 内部 |
| `GetPacketBufPtr()` | 获取包缓冲区指针 | 内部 |

### 6.2 打包流程

```c
// 1. 初始化
PreprareBuild();  // g_counter = 0, memset(g_buffer)

// 2. 打包字段
MarshallUint16(DMS_MSG_CMD_START_FA, COMMAND_ID);
MarshallUint16(200, DMS_VERSION);
MarshallString("com.example.app", CALLEE_BUNDLE_NAME);
MarshallString("MainAbility", CALLEE_ABILITY_NAME);
MarshallString("signature", CALLER_SIGNATURE);

// 3. 获取结果
int32_t size = GetPacketSize();
const char *data = GetPacketBufPtr();
```

### 6.3 全局缓冲区

```c
static char g_buffer[PACKET_DATA_SIZE] = {0};  // 1024 字节
static uint16_t g_counter = 0;                  // 当前写入位置
```

**位置**: `source/dmslite_packet.c:42-43`

**注意**: 全局缓冲区非线程安全，依赖单会话设计保证。

## 7. TLV Common 模块 (dmslite_tlv_common)

### 7.1 接口列表

**位置**: `include/dmslite_tlv_common.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `GetNodeByType()` | 按类型查找 TLV 节点 | 内部 |
| `UnMarshallUint8/16/32/64()` | 解包无符号整数 | 内部 |
| `UnMarshallInt8/16/32/64()` | 解包有符号整数 | 内部 |
| `UnMarshallString()` | 解包字符串 | 内部 |

### 7.2 TLV 节点结构

```c
typedef struct TlvNode {
    uint8_t type;           // 字段类型
    uint16_t length;        // 值长度
    const uint8_t *value;   // 值指针（指向原始数据）
    struct TlvNode *next;   // 下一个节点
} TlvNode;
```

**位置**: `include/dmslite_tlv_common.h:32-37`

### 7.3 字段类型定义

```c
enum FieldType {
    COMMAND_ID = 1,         // uint16, 命令 ID
    DMS_VERSION = 2,        // uint16, 协议版本
    CALLEE_BUNDLE_NAME = 3, // string, 被调用方包名
    CALLEE_ABILITY_NAME = 4,// string, 被调用方 Ability 名
    CALLER_SIGNATURE = 5,   // string, 调用方签名
    CALLER_PAYLOAD = 6,     // raw, 透传数据
    REPLY_ERR_CODE = 0xFF   // int32, 回复错误码
};
```

**位置**: `include/dmslite_tlv_common.h:50-58`

## 8. Msg Handler 模块 (dmslite_msg_handler)

### 8.1 接口列表

**位置**: `include/dmslite_msg_handler.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `StartAbilityFromRemoteHandler()` | 处理 START_FA 命令 | 内部 |
| `ReplyMsgHandler()` | 处理 REPLY 命令 | 内部 |

### 8.2 StartAbilityFromRemoteHandler

**功能**: 处理远程启动 FA 请求。

**流程**:
1. `UnMarshallString()` 解包 calleeBundleName
2. `UnMarshallString()` 解包 calleeAbilityName
3. `UnMarshallString()` 解包 callerSignature
4. `CheckRemotePermission()` 权限检查
5. `StartAbilityFromRemote()` 启动 FA（当前空实现）

**代码位置**: `source/dmslite_msg_handler.c:25-41`

## 9. DevMgr 模块 (dmslite_devmgr)

### 9.1 接口列表

**位置**: `include/dmslite_devmgr.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `AddDevMgrListener()` | 注册设备状态监听 | 内部 |
| `UnRegisterDevMgrListener()` | 注销设备状态监听 | 内部 |
| `GetPeerId()` | 获取对端设备 ID | 内部 |

### 9.2 设备状态回调

```c
static INodeStateCb g_networkListener = {
    .events = EVENT_NODE_STATE_ONLINE | EVENT_NODE_STATE_OFFLINE,
    .onNodeOnline = onNodeOnline,      // 设备上线
    .onNodeOffline = onNodeOffline,    // 设备离线
    .onNodeBasicInfoChanged = onNodeBasicInfoChanged
};
```

**位置**: `source/dmslite_devmgr.c:30-35`

## 10. Feature 模块 (dmslite_feature)

### 10.1 接口列表

**位置**: `include/dmslite_feature.h`

| 函数 | 功能 | 稳定性 |
|------|------|--------|
| `GetDmsLiteFeature()` | 获取 DmsLite Feature 实例 | 稳定 |

### 10.2 DmsLite 结构

```c
typedef struct {
    INHERIT_FEATURE;                    // 继承 Feature
    INHERIT_IUNKNOWNENTRY(DmsProxy);    // 继承 IUnknownEntry
    Identity identity;                  // SAMGR 身份
} DmsLite;
```

**位置**: `include/dmslite_feature.h:28-34`

### 10.3 消息类型枚举

```c
enum DmsMsgType {
    DEVICE_ONLINE = 0x01,           // 设备上线
    DEVICE_OFFLINE,                 // 设备离线
    SESSION_OPEN,                   // 会话建立
    SESSION_CLOSE,                  // 会话关闭
    BYTES_RECEIVED,                 // 收到数据
    START_REMOTE_ABILITY,           // 启动远程 FA
    START_ABILITY_FROM_REMOTE       // 从远程启动 FA
};
```

**位置**: `include/dmslite_feature.h:36-44`

## 11. 模块依赖关系

```
dmsfwk_interface.h (对外接口)
    ▲
    │
┌───┴─────────────────────────────────────────────────────────┐
│                    内部模块依赖关系                          │
│                                                              │
│  dmslite_feature.h ──────┬──────► dmslite_famgr.h            │
│         │                │              │                    │
│         │                │              ▼                    │
│         │                │       dmslite_session.h           │
│         │                │              │                    │
│         │                ▼              ▼                    │
│         │         dmslite_permission.h  │                    │
│         │                │              │                    │
│         ▼                ▼              ▼                    │
│  dmslite_inner_common.h ◄┘       dmslite_parser.h            │
│         ▲                           │                        │
│         │                           ▼                        │
│         └───────────────── dmslite_tlv_common.h ◄────────────┤
│                                         │                    │
│                                         ▼                    │
│                              dmslite_packet.h                │
│                                                              │
│  dmslite_devmgr.h ───────► dmslite_session.h                 │
│                                                              │
│  dmslite_msg_handler.h ──► dmslite_tlv_common.h              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 12. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构设计
- [03_Public_API](03_Public_API.md) - 对外接口
