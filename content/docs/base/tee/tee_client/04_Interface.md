# TEE Client 对外接口文档

## 1. TEEC API 清单表

### 1.1 核心 API（GlobalPlatform TEE Client API v1.0）

| JS API | 参数/返回 | 同步/异步 | C++ 入口 | 权限/错误码 |
|--------|-----------|-----------|----------|-------------|
| **TEEC_InitializeContext** | `name` (const char*), `context` (TEEC_Context*) / TEEC_Result | 同步 | `frameworks/libteec_vendor/tee_client_api.c:932` | 无需特殊权限，`TEEC_ERROR_BAD_PARAMETERS`, `TEEC_ERROR_GENERIC` |
| **TEEC_FinalizeContext** | `context` (TEEC_Context*) / void | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1052` | 无需特殊权限，清理资源 |
| **TEEC_OpenSession** | `context`, `session`, `destination`, `connectionMethod`, `connectionData`, `operation`, `returnOrigin` / TEEC_Result | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1205` | 需要 CA 身份认证，`TEEC_ERROR_ACCESS_DENIED`, `TEEC_ERROR_OUT_OF_MEMORY`, `TEEC_ERROR_TRUSTED_APP_LOAD_ERROR` |
| **TEEC_CloseSession** | `session` (TEEC_Session*) / void | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1296` | 无需特殊权限，释放会话资源 |
| **TEEC_InvokeCommand** | `session`, `commandID`, `operation`, `returnOrigin` / TEEC_Result | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1398` | 需要有效会话，`TEEC_ERROR_BAD_PARAMETERS`, `TEEC_ERROR_ACCESS_DENIED` |
| **TEEC_RegisterSharedMemory** | `context`, `sharedMem` / TEEC_Result | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1467` | 无需特殊权限，`TEEC_ERROR_BAD_PARAMETERS` |
| **TEEC_AllocateSharedMemory** | `context`, `sharedMem` / TEEC_Result | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1625` | 无需特殊权限，`TEEC_ERROR_OUT_OF_MEMORY` |
| **TEEC_ReleaseSharedMemory** | `sharedMem` / void | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1731` | 无需特殊权限，释放内存 |
| **TEEC_RequestCancellation** | `operation` (TEEC_Operation*) / void | 同步 | `frameworks/libteec_vendor/tee_client_api.c:1800` | 发送取消请求（当前无效） |

### 1.2 扩展 API

| API | 参数/返回 | 说明 | 位置 |
|-----|-----------|------|------|
| **TEEC_EXT_RegisterProcessDeath** | `notify` (sptr<IRemoteObject>) / int32_t | 注册 CA 进程死亡通知 | `cadaemon_service.cpp:1194` |
| **TEEC_EXT_SendSecfile** | `path`, `fd`, `fp` / TEEC_Result | 发送安全文件到 TEE | `cadaemon_service.cpp:1308` |
| **TEEC_EXT_GetTeeVersion** | 无 / uint32_t | 获取 TEE 版本 | `cadaemon_service.cpp:1319` |

## 2. IPC 接口

### 2.1 System Ability 信息

| 属性 | 值 |
|------|-----|
| **服务名** | `CaDaemonService` |
| **SA ID** | `8001` |
| **接口令牌** | `u"ohos.tee_client.accessToken"` |
| **进程名** | `cadaemon` |

### 2.2 IPC 操作码（CadaemonOperationInterfaceCode）

```cpp
// services/cadaemon/src/ca_daemon/cadaemon_stub.cpp
enum class CadaemonOperationInterfaceCode : uint32_t {
    INIT_CONTEXT    = 0,   // 初始化 TEE 上下文
    FINAL_CONTEXT   = 1,   // 结束 TEE 上下文
    OPEN_SESSION    = 2,   // 打开 TA 会话
    CLOSE_SESSION   = 3,   // 关闭 TA 会话
    INVOKE_COMMND   = 4,   // 调用 TA 命令
    REGISTER_MEM    = 5,   // 注册共享内存
    ALLOC_MEM       = 6,   // 分配共享内存
    RELEASE_MEM     = 7,   // 释放共享内存
    SET_CALL_BACK   = 8,   // 设置死亡回调
    SEND_SECFILE    = 9,   // 发送安全文件
    GET_TEE_VERSION = 10,  // 获取 TEE 版本
};
```

### 2.3 IPC 请求处理流程

```
┌─────────────────────────────────────────────────────────────┐
│                     IPC 请求处理流程                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CA 应用                                                      │
│     │                                                         │
│     ├──► TeeClient::OpenSession()                            │
│     │        @ frameworks/libteec_client/tee_client.cpp       │
│     │                                                         │
│     ├──► MessageParcel::WriteInterfaceToken()                │
│     │        写入接口令牌                                     │
│     │                                                         │
│     ├──► MessageParcel::WriteBuffer()                        │
│     │        写入 Context, Session, Operation 等              │
│     │                                                         │
│     └──► IPC::SendRequest() ──► CaDaemonStub::OnRemoteRequest()
│                                      @ cadaemon_stub.cpp:30   │
│                                        │                      │
│                                        ├──► CheckPermission() │
│                                        │      @ line 44       │
│                                        │      验证 INTERFACE_TOKEN │
│                                        │                      │
│                                        └──► OpenSessionRecvProc()
│                                               @ line 131     │
│                                                 │            │
│                                                 └──► CaDaemonService::OpenSession()
│                                                        @ cadaemon_service.cpp:827
│                                                          │   │
│                                                          ├──► IsValidContext()    │
│                                                          ├──► TEEC_OpenSessionInner() │
│                                                          └──► reply.WriteBuffer() │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 3. 配置文件

### 3.1 SA 配置文件

**文件**: `services/cadaemon/build/standard/sa_profile/8001.json`

```json
{
    "process": "cadaemon",
    "systemability": [
        {
            "name": 8001,
            "libpath": "libcadaemon.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "start-on-demand": {},
            "stop-on-demand": {}
        }
    ]
}
```

### 3.2 Init 配置文件

**文件**: `services/cadaemon/build/standard/init/cadaemon.cfg`

```json
{
    "services": [
        {
            "name": "cadaemon",
            "path": "/system/lib/libcadaemon.so",
            "uid": "system",
            "gid": ["system"],
            "secon": "u:r:cadaemon:s0",
            "permission": ["ohos.permission.ACCESS_TEE"]
        }
    ]
}
```

**文件**: `services/teecd/build/standard/init/teecd.cfg`

```json
{
    "services": [
        {
            "name": "teecd",
            "path": "/vendor/bin/teecd",
            "uid": "root",
            "gid": ["root"],
            "secon": "u:r:teecd:s0"
        }
    ]
}
```

**文件**: `services/tlogcat/build/standard/init/tlogcat.cfg`

```json
{
    "services": [
        {
            "name": "tlogcat",
            "path": "/system/bin/tlogcat",
            "uid": "system",
            "gid": ["system", "log"],
            "secon": "u:r:tlogcat:s0"
        }
    ]
}
```

### 3.3 配置文件字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 服务名称 |
| `path` | string | 可执行文件或库路径 |
| `uid` | string | 运行用户 ID |
| `gid` | array | 运行组 ID 列表 |
| `secon` | string | SELinux 安全上下文 |
| `permission` | array | 所需权限列表 |
| `run-on-create` | bool | 系统启动时是否自动运行 |

## 4. 数据类型定义

### 4.1 基本类型

```c
// interfaces/inner_api/tee_client_type.h

typedef uint32_t TEEC_Result;

typedef struct {
    uint32_t timeLow;
    uint16_t timeMid;
    uint16_t timeHiAndVersion;
    uint8_t clockSeqAndNode[8];
} TEEC_UUID;
```

### 4.2 上下文与会话

```c
// interfaces/inner_api/tee_client_type.h

typedef struct {
    int32_t fd;                      // TEE 设备文件描述符
    uint8_t *ta_path;                // TA 文件路径（可选）
    struct ListNode session_list;    // 会话列表
    struct ListNode shrd_mem_list;   // 共享内存列表
    union {
        struct {
            void *buffer;
            sem_t buffer_barrier;
        } share_buffer;
        uint64_t imp;
    };
} TEEC_Context;

typedef struct {
    uint32_t session_id;             // 会话 ID
    TEEC_UUID service_id;            // TA UUID
    uint32_t ops_cnt;                // 操作计数
    union {
        struct ListNode head;
        uint64_t imp;
    };
    TEEC_Context *context;           // 所属上下文
} TEEC_Session;
```

### 4.3 共享内存

```c
// interfaces/inner_api/tee_client_type.h

typedef struct {
    void *buffer;                    // 缓冲区地址
    uint32_t size;                   // 缓冲区大小
    uint32_t flags;                  // 标志位
    uint32_t ops_cnt;                // 操作计数
    bool is_allocated;               // 是否分配式
    union {
        struct ListNode head;
        void* imp;
    };
    TEEC_Context *context;           // 所属上下文
} TEEC_SharedMemory;
```

### 4.4 参数类型

```c
// interfaces/inner_api/tee_client_type.h

typedef struct {
    void *buffer;
    uint32_t size;
} TEEC_TempMemoryReference;

typedef struct {
    TEEC_SharedMemory *parent;
    uint32_t size;
    uint32_t offset;
} TEEC_RegisteredMemoryReference;

typedef struct {
    uint32_t a;
    uint32_t b;
} TEEC_Value;

typedef struct {
    int ion_share_fd;
    uint32_t ion_size;
} TEEC_IonReference;

typedef union {
    TEEC_TempMemoryReference tmpref;
    TEEC_RegisteredMemoryReference memref;
    TEEC_Value value;
    TEEC_IonReference ionref;
} TEEC_Parameter;

typedef struct {
    uint32_t started;                // 0=取消，非0=执行
    uint32_t paramTypes;             // 参数类型组合
    TEEC_Parameter params[TEEC_PARAM_NUM];  // 参数数组（最多4个）
    TEEC_Session *session;
    bool cancel_flag;
} TEEC_Operation;
```

### 4.5 参数类型宏

```c
// interfaces/inner_api/tee_client_constants.h

#define TEEC_PARAM_NUM          4

// 参数类型值
#define TEEC_NONE               0x00000000
#define TEEC_VALUE_INPUT        0x00000001
#define TEEC_VALUE_OUTPUT       0x00000002
#define TEEC_VALUE_INOUT        0x00000003
#define TEEC_MEMREF_TEMP_INPUT  0x00000005
#define TEEC_MEMREF_TEMP_OUTPUT 0x00000006
#define TEEC_MEMREF_TEMP_INOUT  0x00000007
#define TEEC_MEMREF_WHOLE       0x0000000C
#define TEEC_MEMREF_PARTIAL_INPUT   0x0000000D
#define TEEC_MEMREF_PARTIAL_OUTPUT  0x0000000E
#define TEEC_MEMREF_PARTIAL_INOUT   0x0000000F

// 构造参数类型组合
#define TEEC_PARAM_TYPES(p0, p1, p2, p3) \
    ((p3) << 12 | (p2) << 8 | (p1) << 4 | (p0))

// 获取指定索引的参数类型
#define TEEC_PARAM_TYPE_GET(paramTypes, index) \
    (((paramTypes) >> (4*(index))) & 0x0F)
```

## 5. 错误码定义

### 5.1 标准错误码（TEEC_ReturnCode）

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `TEEC_SUCCESS` | 0x00000000 | 成功 |
| `TEEC_ERROR_GENERIC` | 0xFFFF0000 | 通用错误 |
| `TEEC_ERROR_ACCESS_DENIED` | 0xFFFF0001 | 访问被拒绝 |
| `TEEC_ERROR_CANCEL` | 0xFFFF0002 | 操作被取消 |
| `TEEC_ERROR_ACCESS_CONFLICT` | 0xFFFF0003 | 访问冲突 |
| `TEEC_ERROR_EXCESS_DATA` | 0xFFFF0004 | 数据过多 |
| `TEEC_ERROR_BAD_FORMAT` | 0xFFFF0005 | 格式错误 |
| `TEEC_ERROR_BAD_PARAMETERS` | 0xFFFF0006 | 参数错误 |
| `TEEC_ERROR_BAD_STATE` | 0xFFFF0007 | 状态错误 |
| `TEEC_ERROR_ITEM_NOT_FOUND` | 0xFFFF0008 | 项目未找到 |
| `TEEC_ERROR_NOT_IMPLEMENTED` | 0xFFFF0009 | 未实现 |
| `TEEC_ERROR_NOT_SUPPORTED` | 0xFFFF000A | 不支持 |
| `TEEC_ERROR_NO_DATA` | 0xFFFF000B | 无数据 |
| `TEEC_ERROR_OUT_OF_MEMORY` | 0xFFFF000C | 内存不足 |
| `TEEC_ERROR_BUSY` | 0xFFFF000D | 忙 |
| `TEEC_ERROR_COMMUNICATION` | 0xFFFF000E | 通信错误 |
| `TEEC_ERROR_SECURITY` | 0xFFFF000F | 安全错误 |
| `TEEC_ERROR_SHORT_BUFFER` | 0xFFFF0010 | 缓冲区过短 |
| `TEEC_ERROR_EXTERNAL_CANCEL` | 0xFFFF0011 | 外部取消 |
| `TEEC_ERROR_TARGET_DEAD` | 0xFFFF3024 | 目标已死亡 |
| `TEEC_ERROR_TRUSTED_APP_LOAD_ERROR` | 0xFFFF3025 | TA 加载错误 |

### 5.2 错误来源（TEEC_ReturnCodeOrigin）

| 来源 | 值 | 说明 |
|------|-----|------|
| `TEEC_ORIGIN_API` | 0x00000001 | 来自 TEE Client API |
| `TEEC_ORIGIN_COMMS` | 0x00000002 | 来自通信层 |
| `TEEC_ORIGIN_TEE` | 0x00000003 | 来自 TEE |
| `TEEC_ORIGIN_TRUSTED_APP` | 0x00000004 | 来自 TA |

## 6. 使用示例

### 6.1 基本流程

```c
#include "tee_client_api.h"
#include <stdio.h>
#include <string.h>

// TA UUID 示例
static const TEEC_UUID TA_UUID = {
    0x58dbb3b9, 0x4a0c, 0x42d2,
    { 0xa8, 0x4d, 0x7c, 0x7a, 0xb1, 0x75, 0x39, 0xfc }
};

int main() {
    TEEC_Context context;
    TEEC_Session session;
    TEEC_Operation operation;
    TEEC_Result result;
    uint32_t returnOrigin;
    
    // 1. 初始化 TEE 上下文
    result = TEEC_InitializeContext(NULL, &context);
    if (result != TEEC_SUCCESS) {
        printf("InitializeContext failed: 0x%x\n", result);
        return -1;
    }
    
    // 2. 打开 TA 会话
    result = TEEC_OpenSession(&context, &session, &TA_UUID,
                              TEEC_LOGIN_PUBLIC, NULL, NULL, &returnOrigin);
    if (result != TEEC_SUCCESS) {
        printf("OpenSession failed: 0x%x (origin: 0x%x)\n", 
               result, returnOrigin);
        TEEC_FinalizeContext(&context);
        return -1;
    }
    
    // 3. 准备操作参数（示例：调用命令 0x01）
    memset(&operation, 0, sizeof(operation));
    operation.paramTypes = TEEC_PARAM_TYPES(
        TEEC_VALUE_INPUT,    // param[0]: 输入值
        TEEC_VALUE_OUTPUT,   // param[1]: 输出值
        TEEC_NONE,           // param[2]: 未使用
        TEEC_NONE            // param[3]: 未使用
    );
    operation.params[0].value.a = 100;  // 输入参数
    
    // 4. 调用 TA 命令
    result = TEEC_InvokeCommand(&session, 0x01, &operation, &returnOrigin);
    if (result != TEEC_SUCCESS) {
        printf("InvokeCommand failed: 0x%x (origin: 0x%x)\n", 
               result, returnOrigin);
    } else {
        printf("Result: %d\n", operation.params[1].value.a);
    }
    
    // 5. 关闭会话
    TEEC_CloseSession(&session);
    
    // 6. 销毁上下文
    TEEC_FinalizeContext(&context);
    
    return 0;
}
```

### 6.2 使用共享内存

```c
TEEC_SharedMemory sharedMem;

// 方式1：注册现有内存
char buffer[1024];
sharedMem.buffer = buffer;
sharedMem.size = sizeof(buffer);
sharedMem.flags = TEEC_MEM_INPUT | TEEC_MEM_OUTPUT;
result = TEEC_RegisterSharedMemory(&context, &sharedMem);

// 方式2：分配共享内存
sharedMem.size = 1024;
sharedMem.flags = TEEC_MEM_INPUT | TEEC_MEM_OUTPUT;
result = TEEC_AllocateSharedMemory(&context, &sharedMem);
// 使用 sharedMem.buffer 访问内存

// 在 Operation 中使用
operation.paramTypes = TEEC_PARAM_TYPES(
    TEEC_MEMREF_WHOLE,
    TEEC_NONE,
    TEEC_NONE,
    TEEC_NONE
);
operation.params[0].memref.parent = &sharedMem;

// 释放共享内存
TEEC_ReleaseSharedMemory(&sharedMem);
```

### 6.3 指定 TA 路径

```c
// 方式1：使用默认路径（/system/bin 或 /vendor/bin）
TEEC_Context context;
TEEC_InitializeContext(NULL, &context);

// 方式2：指定自定义 TA 路径（必须是 /data 目录下）
TEEC_Context context;
context.ta_path = (uint8_t *)"/data/ta/58dbb3b9-4a0c-42d2-a84d-7c7ab17539fc.sec";
TEEC_InitializeContext(NULL, &context);
```

## 7. 接口限制与约束

### 7.1 参数限制

| 限制项 | 值 | 说明 |
|--------|-----|------|
| **最大参数数量** | 4 | params[0] - params[3] |
| **params[2], params[3]** | 系统保留 | CA 不能使用 |
| **最大共享内存大小** | 受系统限制 | 通常 4MB+ |
| **TA 路径前缀** | `/data/` | 自定义 TA 路径必须以 /data/ 开头 |
| **最大上下文数** | 8 (MAX_CXTCNT_ONECA) | 每个 CA 最多 8 个上下文 |

### 7.2 Login Method 限制

| Login Method | 支持状态 | 说明 |
|--------------|----------|------|
| `TEEC_LOGIN_PUBLIC` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_USER` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_GROUP` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_APPLICATION` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_USER_APPLICATION` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_GROUP_APPLICATION` | ❌ 不支持 | GP 标准，OpenHarmony 不支持 |
| `TEEC_LOGIN_IDENTIFY` | ✅ 支持 | OpenHarmony 扩展类型 |

### 7.3 线程安全

| API | 线程安全 | 说明 |
|-----|----------|------|
| `TEEC_InitializeContext` | 是 | 可并发调用 |
| `TEEC_OpenSession` | 是 | 可并发调用 |
| `TEEC_InvokeCommand` | 是 | 可并发调用（同一会话） |
| `TEEC_CloseSession` | 是 | 需确保无正在执行的命令 |
| `TEEC_FinalizeContext` | 是 | 需确保所有资源已释放 |

---

**文档版本**: 1.0
**更新时间**: 2026-02-07
