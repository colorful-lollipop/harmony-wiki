# 08_内部实现细节

> dmsfwk_lite 项目的核心数据结构、模块职责、接口契约及资源生命周期管理详解。

## 一、核心数据结构

### 1.1 服务与 Feature 结构

#### DistributedService

| 字段 | 类型 | 说明 |
|-----|------|------|
| `INHERIT_SERVICE` | 宏扩展 | 继承 Service 基类 |
| `identity` | `Identity` | SAMgr 身份标识 |

**证据来源**：`include/dmslite.h:27-30`

```cpp
typedef struct {
    INHERIT_SERVICE;
    Identity identity;
} DistributedService;
```

#### DmsLite

| 字段 | 类型 | 说明 |
|-----|------|------|
| `GetName` | `const char *(*)(Feature *)` | Feature 名称获取 |
| `OnInitialize` | `void (*)(Feature *, Service *, Identity)` | 初始化回调 |
| `OnStop` | `void (*)(Feature *, Identity)` | 停止回调 |
| `OnMessage` | `BOOL (*)(Feature *, Request *)` | 消息处理 |
| `identity` | `Identity` | 身份标识 |
| `StartRemoteAbility` | `int32_t (*)(...)` | 远程启动接口 |

**证据来源**：`include/dmslite_feature.h` Feature 结构定义。

### 1.2 请求与回调结构

#### RequestData

| 字段 | 类型 | 说明 | 生命周期 |
|-----|------|------|---------|
| `want` | `Want *` | 启动意图 | 单次请求 |
| `callerInfo` | `CallerInfo *` | 调用者信息 | 单次请求 |
| `callback` | `IDmsListener *` | 结果回调 | 单次请求 |

**证据来源**：`include/dmslite_famgr.h:32-36`

```cpp
typedef struct {
    Want *want;
    CallerInfo *callerInfo;
    IDmsListener *callback;
} RequestData;
```

#### IDmsListener

| 字段 | 类型 | 说明 |
|-----|------|------|
| `OnResultCallback` | `void (*)(const void *data, int32_t ret)` | 结果回调函数 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:57-59`

### 1.3 TLV 相关结构

#### TlvNode

| 字段 | 类型 | 说明 |
|-----|------|------|
| `type` | `uint8_t` | 字段类型 |
| `length` | `uint16_t` | 值长度 |
| `value` | `const uint8_t *` | 值指针 |
| `next` | `struct TlvNode *` | 下一个节点 |

**证据来源**：`include/dmslite_tlv_common.h:32-37`

```cpp
typedef struct TlvNode {
    uint8_t type;
    uint16_t length;
    const uint8_t *value;
    struct TlvNode *next;
} TlvNode;
```

#### CommuMessage

| 字段 | 类型 | 说明 |
|-----|------|------|
| `payloadLength` | `uint16_t` | 负载长度 |
| `payload` | `const uint8_t *` | 负载数据 |

**证据来源**：`include/dmslite_tlv_common.h:60-63`

---

## 二、模块职责详解

### 2.1 dmslite.c - 服务入口模块

**职责**：
- 定义 SAMgr Service 结构
- 实现服务生命周期回调
- 配置任务参数

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `GetName()` | 40-46 | 返回服务名 |
| `Initialize()` | 48-56 | 初始化，保存 identity |
| `MessageHandle()` | 58-67 | 消息处理（未使用） |
| `GetTaskConfig()` | 69-73 | 获取任务配置 |
| `Init()` | 75-80 | 模块初始化入口 |

**证据来源**：`source/dmslite.c:40-80`

### 2.2 dmslite_feature.c - Feature 接口模块

**职责**：
- 实现 SAMgr Feature 接口
- 注册 DmsLite 服务能力
- 路由 SAMgr 消息到各处理模块

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `GetName()` | 51-57 | 返回 Feature 名 |
| `OnInitialize()` | 59-66 | 初始化，保存 identity |
| `OnStop()` | 68-71 | 停止回调 |
| `OnMessage()` | 73-110 | 消息路由 |
| `Init()` | 112-125 | Feature 注册 |

**消息路由**：

```cpp
// source/dmslite_feature.c:80-108
switch (request->msgId) {
    case START_REMOTE_ABILITY:
        // 启动远程 FA
    case SESSION_OPEN:
        // 会话建立
    case SESSION_CLOSE:
        // 会话关闭
    case BYTES_RECEIVED:
        // 收到数据
}
```

### 2.3 dmslite_famgr.c - FA 管理模块

**职责**：
- 接收远程启动请求
- 封装 TLV 消息
- 转发到会话管理模块

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `StartRemoteAbility()` | 85-110 | 远程启动入口 |
| `StartRemoteAbilityInner()` | 45-83 | 内部启动接口 |
| `MarshallDmsMessage()` | 112-136 | 消息封装 |
| `FillRequestData()` | 171-218 | 填充请求数据 |
| `FreeRequestData()` | 220-231 | 释放请求资源 |

**资源分配**：

```cpp
// 分配流程
RequestData *reqdata = DMS_ALLOC(sizeof(RequestData));    // 分配
Want *wantData = DMS_ALLOC(sizeof(Want));               // 分配
char *data = DMS_ALLOC(size);                           // 分配
CallerInfo *callerData = DMS_ALLOC(sizeof(CallerInfo)); // 分配

// 释放流程
FreeRequestData(reqdata->want, reqdata->callerInfo);
DMS_FREE(reqdata);
```

### 2.4 dmslite_permission.c - 权限校验模块

**职责**：
- 获取调用者签名
- 获取被调用者签名
- 比对签名一致性

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `CheckRemotePermission()` | 63-116 | 权限校验入口 |
| `GetCallerBundleInfo()` | 202-222 | 获取调用者 BundleInfo |
| `GetBundleInfoFromFile()` | 118-161 | 从文件读取 native appId |
| `GetBundleInfoFromBms()` | 163-200 | 从 BMS 获取 BundleInfo |

**签名校验流程**：

```cpp
// source/dmslite_permission.c:103-115
const char *calleeSignature = bundleInfo.appId + strlen(permissionCheckInfo->calleeBundleName)
    + DELIMITER_LENGTH;
if (strcmp(permissionCheckInfo->callerSignature, calleeSignature) != 0) {
    HILOGE("[Signature unmatched]");
    return DMS_EC_CHECK_PERMISSION_FAILURE;
}
return DMS_EC_SUCCESS;
```

### 2.5 dmslite_session.c - 会话管理模块

**职责**：
- 创建/销毁会话服务器
- 打开/关闭会话
- 发送/接收消息
- 处理会话回调

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `SendDmsMessage()` | 194-222 | 发送消息入口 |
| `CreateDMSSessionServer()` | 184-187 | 创建会话服务器 |
| `CloseDMSSessionServer()` | 189-192 | 关闭会话服务器 |
| `OnBytesReceived()` | 72-102 | 字节数据回调 |
| `OnSessionOpened()` | 137-158 | 会话建立回调 |
| `OnSessionClosed()` | 113-126 | 会话关闭回调 |
| `IsDmsBusy()` | 251-258 | 检查忙状态 |
| `InvokeCallback()` | 232-242 | 调用结果回调 |

**全局状态**：

```cpp
// source/dmslite_session.c:40-43
static int32_t g_curSessionId = INVALID_SESSION_ID;
static bool g_curBusy = false;
static time_t g_begin;
static IDmsListener *g_listener = NULL;
```

### 2.6 dmslite_parser.c - TLV 解析模块

**职责**：
- 解析 TLV 字节流
- 验证 TLV 结构有效性
- 分发到消息处理器

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `ProcessCommuMsg()` | 256-294 | 消息处理入口 |
| `TlvBytesToNode()` | 165-227 | 字节转 TLV 节点 |
| `TlvFillNode()` | 80-114 | 填充单个 TLV 节点 |
| `TlvBytesToLength()` | 49-78 | 解析变长长度 |
| `CheckNodeSequence()` | 139-151 | 检查节点顺序 |

### 2.7 dmslite_packet.c - 消息封装模块

**职责**：
- 封装各类数据为 TLV 格式
- 提供全局消息缓冲区
- 管理缓冲区偏移

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `PreprareBuild()` | 51-59 | 准备构建消息 |
| `CleanBuild()` | 61-67 | 清理缓冲区 |
| `MarshallUint16()` | 74-77 | 封装 uint16 |
| `MarshallString()` | 119-136 | 封装字符串 |
| `MarshallRawData()` | 138-156 | 封装原始数据 |
| `GetPacketSize()` | 109-112 | 获取消息大小 |
| `GetPacketBufPtr()` | 114-117 | 获取缓冲区指针 |

### 2.8 dmslite_msg_handler.c - 消息处理模块

**职责**：
- 处理 DMS 命令
- 调用权限校验
- 启动 FA

**关键函数**：

| 函数 | 行号 | 职责 |
|-----|------|------|
| `StartAbilityFromRemoteHandler()` | 25-41 | 启动 FA 处理 |
| `ReplyMsgHandler()` | 43-50 | 回复消息处理 |

---

## 三、内部 API 契约

### 3.1 稳定接口（内部使用）

| 接口 | 所在模块 | 调用者 | 稳定性 |
|-----|---------|-------|-------|
| `StartRemoteAbility()` | famgr | feature | 稳定 |
| `StartRemoteAbilityInner()` | famgr | feature | 稳定 |
| `SendDmsMessage()` | session | famgr | 稳定 |
| `ProcessCommuMsg()` | parser | session | 稳定 |
| `CheckRemotePermission()` | permission | msg_handler | 稳定 |

### 3.2 宏定义契约

#### 消息封装宏

```cpp
// source/dmslite_famgr.c:114
PACKET_MARSHALL_HELPER(Uint16, COMMAND_ID, DMS_MSG_CMD_START_FA);
// 展开为：
do {
    bool ret = MarshallUint16(DMS_MSG_CMD_START_FA, COMMAND_ID);
    if (!ret) {
        CleanBuild();
        return -1;
    }
} while (0)
```

#### 内存分配宏

```cpp
// source/dmslite_utils.h:64-71
#define DMS_ALLOC(size) malloc(size)
#define DMS_FREE(a) \
    do { \
        if ((a) != NULL) { \
            free((a)); \
            (a) = NULL; \
        } \
    } while (0)
```

---

## 四、资源生命周期

### 4.1 启动阶段

```
系统启动 → SYS_SERVICE_INIT(Init)
    │
    ├── RegisterService() → DistributedService 注册
    └── RegisterFeature() → DmsLite Feature 注册
```

**证据来源**：`source/dmslite.c:75-80` 服务初始化。

### 4.2 请求阶段

```
请求到达 → OnMessage()
    │
    ├── 分配 RequestData (DMS_ALLOC)
    ├── 分配 Want (DMS_ALLOC)
    ├── 分配 CallerInfo (DMS_ALLOC)
    └── 分配 data (DMS_ALLOC)
```

### 4.3 完成阶段

```
处理完成 → FreeRequestData()
    │
    ├── 释放 Want (DMS_FREE)
    ├── 释放 CallerInfo->bundleName (DMS_FREE)
    ├── 释放 CallerInfo (DMS_FREE)
    └── 释放 RequestData (DMS_FREE)
```

**证据来源**：`source/dmslite_famgr.c:220-231`

### 4.4 销毁阶段

```
服务停止 → OnStop()
    │
    ├── 关闭会话服务器
    └── 清理全局状态
```

---

## 五、模块依赖关系

### 5.1 依赖方向

```
     ┌─────────────────────────────────────────┐
     │              dmslite.c                   │
     │         (服务入口，无依赖)               │
     └──────────────────┬────────────────────┘
                        │
                        ▼
     ┌─────────────────────────────────────────┐
     │          dmslite_feature.c              │ 依赖 dmslite.c
     │    (Feature 实现，调用各模块)             │
     └──────────────────┬────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│dmslite_famgr.c│ │dmslite_session│ │dmslite_parser│
│  (依赖 packet │ │   (依赖 parser │ │              │
│    permission)│ │    session)   │ │              │
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│dmslite_packet │ │dmslite_permission│              │
│               │ │               │ │              │
└───────────────┘ └───────────────┘ └───────────────┘
```

### 5.2 数据流向

```
feature.c (消息路由)
    │
    ├──▶ famgr.c (FA 管理)
    │       │
    │       ├──▶ packet.c (消息封装)
    │       │       │
    │       │       └──▶ session.c (会话发送)
    │       │
    │       └──▶ permission.c (权限校验)
    │               │
    │               └──▶ BMS (获取签名)
    │
    └──▶ parser.c (消息解析)
            │
            └──▶ msg_handler.c (命令处理)
```

---

## 六、线程安全说明

### 6.1 当前保护机制

| 资源 | 保护方式 | 说明 |
|-----|---------|------|
| `g_curSessionId` | `IsDmsBusy()` 检查 | 依赖单会话设计 |
| `g_curBusy` | `IsDmsBusy()` 检查 | 忙标志检查 |
| `g_buffer` | 单线程访问 | SAMgr 任务单线程 |
| `g_listener` | InvokeCallback 中复制 | 避免回调覆盖 |

### 6.2 潜在竞态

| 场景 | 风险 | 缓解措施 |
|-----|------|---------|
| 多请求并发 | 状态混乱 | `IsDmsBusy()` 拒绝 |
| 回调覆盖 | 结果错乱 | 复制到本地变量 |
| 超时检测 | 资源泄露 | `IsDmsBusy()` 清理 |

**证据来源**：`source/dmslite_session.c:40-43` 全局状态。

---

## 七、错误处理模式

### 7.1 错误码传播

```
底层模块 → 返回错误码
    │
    ├── 上层模块 → 记录日志
    ├── 转换错误码（如需要）
    └── 返回给调用者
```

### 7.2 资源清理

```cpp
// 标准模式
int32_t Func() {
    Resource *res = DMS_ALLOC(sizeof(Resource));
    if (res == NULL) {
        return DMS_EC_FAILURE;
    }
    
    if (some_operation() != SUCCESS) {
        DMS_FREE(res);  // 失败时清理
        return DMS_EC_FAILURE;
    }
    
    DMS_FREE(res);
    return DMS_EC_SUCCESS;
}
```

---

## 八、相关文档

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |
| [03_CodeMap.md](./03_CodeMap.md) | 代码地图 |
| [04_Interface.md](./04_Interface.md) | 对外接口 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全风险评估 |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*
