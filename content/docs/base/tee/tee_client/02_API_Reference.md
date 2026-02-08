# TEE Client API 参考

## 1. API 概述

TEE Client API 实现 GlobalPlatform TEE Client API Specification v1.0 (GPD_SPE_007)，为 CA 提供访问 TEE 的标准化接口。

**API 分类**:
- **上下文管理**: 初始化/销毁 TEE 连接
- **会话管理**: 打开/关闭与 TA 的会话
- **命令操作**: 向 TA 发送命令
- **内存管理**: 注册/分配/释放共享内存
- **操作控制**: 取消正在运行的操作

## 2. API 清单

### 2.1 上下文管理 API

#### TEEC_InitializeContext

| 属性 | 值 |
|------|-----|
| **函数签名** | `TEEC_Result TEEC_InitializeContext(const char *name, TEEC_Context *context)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:78` |
| **同步/异步** | 同步 |
| **返回值类型** | `TEEC_Result` |

**功能描述**: 初始化 TEE 上下文，建立 CA 与 TEE 之间的逻辑连接。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| name | `const char *` | IN | TEE 设备路径，通常为 NULL（使用默认设备） |
| context | `TEEC_Context *` | IN/OUT | TEE 上下文结构体指针 |

**TEEC_Context 结构体** (`tee_client_type.h:75`):
```c
typedef struct {
    int32_t fd;                      // TEE 设备文件描述符
    uint8_t *ta_path;               // TA 文件路径（可选）
    struct ListNode session_list;    // 会话链表
    struct ListNode shrd_mem_list;   // 共享内存链表
    union {
        struct {
            void *buffer;           // 共享缓冲区
            sem_t buffer_barrier;   // 屏障信号量
        } share_buffer;
        uint64_t imp;              // 实现扩展
    };
} TEEC_Context;
```

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `TEEC_SUCCESS` | 初始化成功 |
| `TEEC_ERROR_BAD_PARAMETERS` | name 或 context 参数错误 |
| `TEEC_ERROR_GENERIC` | 系统资源不足 |

**调用示例**:
```c
TEEC_Context context;
TEEC_Result ret;

// 使用默认 TEE 设备初始化
ret = TEEC_InitializeContext(NULL, &context);
if (ret != TEEC_SUCCESS) {
    printf("InitializeContext failed: 0x%x\n", ret);
    return ret;
}

// 使用指定 TA 路径
context.ta_path = (uint8_t *)"/data/myta.sec";
```

---

#### TEEC_FinalizeContext

| 属性 | 值 |
|------|-----|
| **函数签名** | `void TEEC_FinalizeContext(TEEC_Context *context)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:89` |
| **同步/异步** | 同步 |
| **返回值类型** | void |

**功能描述**: 关闭 TEE 上下文，释放相关资源。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| context | `TEEC_Context *` | IN | 已初始化的 TEE 上下文指针 |

**调用示例**:
```c
TEEC_FinalizeContext(&context);
```

---

### 2.2 会话管理 API

#### TEEC_OpenSession

| 属性 | 值 |
|------|-----|
| **函数签名** | `TEEC_Result TEEC_OpenSession(TEEC_Context *context, TEEC_Session *session, const TEEC_UUID *destination, uint32_t connectionMethod, const void *connectionData, TEEC_Operation *operation, uint32_t *returnOrigin)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:120` |
| **同步/异步** | 同步 |
| **返回值类型** | `TEEC_Result` |

**功能描述**: 打开与目标 TA 的会话。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| context | `TEEC_Context *` | IN/OUT | 已初始化的 TEE 上下文 |
| session | `TEEC_Session *` | OUT | 会话句柄（输出） |
| destination | `const TEEC_UUID *` | IN | 目标 TA 的 UUID |
| connectionMethod | `uint32_t` | IN | 连接方法（只支持 `TEEC_LOGIN_IDENTIFY`） |
| connectionData | `const void *` | IN | 连接数据（必须为 NULL） |
| operation | `TEEC_Operation *` | IN/OUT | 操作参数（可选） |
| returnOrigin | `uint32_t *` | OUT | 错误来源（可选） |

**TEEC_UUID 结构体** (`tee_client_type.h:63`):
```c
typedef struct {
    uint32_t timeLow;
    uint16_t timeMid;
    uint16_t timeHiAndVersion;
    uint8_t clockSeqAndNode[8];
} TEEC_UUID;
```

**TEEC_Session 结构体** (`tee_client_type.h:94`):
```c
typedef struct {
    uint32_t session_id;        // 会话 ID
    TEEC_UUID service_id;      // TA UUID
    uint32_t ops_cnt;          // 操作计数
    union {
        struct ListNode head;  // 链表头
        uint64_t imp;         // 实现扩展
    };
    TEEC_Context *context;     // 关联上下文
} TEEC_Session;
```

**参数限制**:
- `operation->params[2]` 和 `operation->params[3]` 预留给系统，CA 不可使用
- CA 只能使用 `params[0]` 和 `params[1]`

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `TEEC_SUCCESS` | 会话打开成功 |
| `TEEC_ERROR_BAD_PARAMETERS` | 参数错误 |
| `TEEC_ERROR_ACCESS_DENIED` | 访问被拒绝 |
| `TEEC_ERROR_OUT_OF_MEMORY` | 内存不足 |
| `TEEC_ERROR_TRUSTED_APP_LOAD_ERROR` | TA 加载失败 |

**调用示例**:
```c
TEEC_Context context;
TEEC_Session session;
TEEC_UUID uuid = {
    0x12345678, 0x1234, 0x5678,
    {0x12, 0x34, 0x56, 0x78, 0x9A, 0xBC, 0xDE, 0xF0}
};
TEEC_Operation operation = {0};
uint32_t origin;

operation.started = 1;
operation.paramTypes = TEEC_PARAM_TYPES(TEEC_VALUE_INPUT, TEEC_VALUE_OUTPUT, 
                                         TEEC_NONE, TEEC_NONE);
operation.params[0].value.a = 100;
operation.params[1].value.b = 0;

TEEC_Result ret = TEEC_OpenSession(&context, &session, &uuid,
    TEEC_LOGIN_IDENTIFY, NULL, &operation, &origin);
```

---

#### TEEC_CloseSession

| 属性 | 值 |
|------|-----|
| **函数签名** | `void TEEC_CloseSession(TEEC_Session *session)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:132` |
| **同步/异步** | 同步 |
| **返回值类型** | void |

**功能描述**: 关闭与 TA 的会话。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| session | `TEEC_Session *` | IN | 要关闭的会话句柄 |

---

### 2.3 命令操作 API

#### TEEC_InvokeCommand

| 属性 | 值 |
|------|-----|
| **函数签名** | `TEEC_Result TEEC_InvokeCommand(TEEC_Session *session, uint32_t commandID, TEEC_Operation *operation, uint32_t *returnOrigin)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:152` |
| **同步/异步** | 同步 |
| **返回值类型** | `TEEC_Result` |

**功能描述**: 向 TA 发送命令。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| session | `TEEC_Session *` | IN/OUT | 已打开的会话句柄 |
| commandID | `uint32_t` | IN | 命令 ID（由 TA 定义） |
| operation | `TEEC_Operation *` | IN/OUT | 操作参数 |
| returnOrigin | `uint32_t *` | OUT | 错误来源（可选） |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `TEEC_SUCCESS` | 命令执行成功 |
| `TEEC_ERROR_BAD_PARAMETERS` | 参数错误 |
| `TEEC_ERROR_ACCESS_DENIED` | 访问被拒绝 |
| `TEEC_ERROR_TARGET_DEAD` | TA 已崩溃 |

**调用示例**:
```c
TEEC_Operation operation = {0};
operation.started = 1;
operation.paramTypes = TEEC_PARAM_TYPES(TEEC_MEMREF_TEMP_INPUT, 
                                         TEEC_MEMREF_TEMP_OUTPUT,
                                         TEEC_NONE, TEEC_NONE);

// 准备输入数据
uint8_t input_data[] = {...};
operation.params[0].tmpref.buffer = input_data;
operation.params[0].tmpref.size = sizeof(input_data);

// 准备输出缓冲区
uint8_t output_buffer[256];
operation.params[1].tmpref.buffer = output_buffer;
operation.params[1].tmpref.size = sizeof(output_buffer);

TEEC_Result ret = TEEC_InvokeCommand(&session, 0x100, &operation, NULL);
```

---

### 2.4 内存管理 API

#### TEEC_RegisterSharedMemory

| 属性 | 值 |
|------|-----|
| **函数签名** | `TEEC_Result TEEC_RegisterSharedMemory(TEEC_Context *context, TEEC_SharedMemory *sharedMem)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:170` |
| **同步/异步** | 同步 |
| **返回值类型** | `TEEC_Result` |

**功能描述**: 注册已有的内存区域为共享内存。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| context | `TEEC_Context *` | IN | TEE 上下文 |
| sharedMem | `TEEC_SharedMemory *` | IN/OUT | 共享内存描述符 |

**TEEC_SharedMemory 结构体** (`tee_client_type.h:110`):
```c
typedef struct {
    void *buffer;              // 内存缓冲区指针
    uint32_t size;            // 缓冲区大小
    uint32_t flags;           // 标志 (TEEC_MEM_INPUT/OUTPUT/INOUT)
    uint32_t ops_cnt;         // 操作计数
    bool is_allocated;        // 是否分配
    union {
        struct ListNode head; // 链表头
        void* imp;           // 实现扩展
    };
    TEEC_Context *context;    // 关联上下文
} TEEC_SharedMemory;
```

**TEEC_SharedMemCtl 标志** (`tee_client_constants.h:137`):
```c
enum TEEC_SharedMemCtl {
    TEEC_MEM_INPUT = 0x1,     // 数据从 CA 流向 TA
    TEEC_MEM_OUTPUT = 0x2,     // 数据从 TA 流向 CA
    TEEC_MEM_INOUT = 0x3,      // 双向传输
    TEEC_MEM_SHARED_INOUT = 0x4 // 共享内存双向
};
```

**调用示例**:
```c
TEEC_SharedMemory shm = {0};
shm.buffer = malloc(4096);
shm.size = 4096;
shm.flags = TEEC_MEM_INPUT | TEEC_MEM_OUTPUT;

TEEC_Result ret = TEEC_RegisterSharedMemory(&context, &shm);
```

---

#### TEEC_AllocateSharedMemory

| 属性 | 值 |
|------|-----|
| **函数签名** | `TEEC_Result TEEC_AllocateSharedMemory(TEEC_Context *context, TEEC_SharedMemory *sharedMem)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:190` |
| **同步/异步** | 同步 |
| **返回值类型** | `TEEC_Result` |

**功能描述**: 分配新的共享内存区域。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| context | `TEEC_Context *` | IN | TEE 上下文 |
| sharedMem | `TEEC_SharedMemory *` | IN/OUT | 共享内存描述符（需设置 size 和 flags） |

**注意**: 如果 size 设置为 0，虽然返回 `TEEC_SUCCESS`，但该共享内存无法使用。

---

#### TEEC_ReleaseSharedMemory

| 属性 | 值 |
|------|-----|
| **函数签名** | `void TEEC_ReleaseSharedMemory(TEEC_SharedMemory *sharedMem)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:202` |
| **同步/异步** | 同步 |
| **返回值类型** | void |

**功能描述**: 释放共享内存。

**参数说明**:

| 参数 | 类型 | 方向 | 说明 |
|------|------|------|------|
| sharedMem | `TEEC_SharedMemory *` | IN | 要释放的共享内存 |

**注意**:
- RegisterSharedMemory 注册的内存：只释放本地引用
- AllocateSharedMemory 分配的内存：实际回收内存

---

### 2.5 操作控制 API

#### TEEC_RequestCancellation

| 属性 | 值 |
|------|-----|
| **函数签名** | `void TEEC_RequestCancellation(TEEC_Operation *operation)` |
| **文件位置** | `interfaces/inner_api/tee_client_api.h:212` |
| **同步/异步** | 同步 |
| **返回值类型** | void |

**功能描述**: 发送取消操作请求。

**注意**: 此 API 仅发送取消消息，实际是否取消由 TEE 或 TA 决定。当前实现中取消操作不生效。

---

## 3. 参数类型定义

### 3.1 TEEC_ParamType 枚举

| 类型 | 值 | 说明 |
|------|------|------|
| `TEEC_NONE` | 0x0 | 参数未使用 |
| `TEEC_VALUE_INPUT` | 0x01 | 32位值输入 |
| `TEEC_VALUE_OUTPUT` | 0x02 | 32位值输出 |
| `TEEC_VALUE_INOUT` | 0x03 | 32位值双向 |
| `TEEC_MEMREF_TEMP_INPUT` | 0x05 | 临时内存引用输入 |
| `TEEC_MEMREF_TEMP_OUTPUT` | 0x06 | 临时内存引用输出 |
| `TEEC_MEMREF_TEMP_INOUT` | 0x07 | 临时内存引用双向 |
| `TEEC_ION_INPUT` | 0x08 | ION 内存输入 |
| `TEEC_ION_SGLIST_INPUT` | 0x09 | ION SGLIST 输入 |
| `TEEC_MEMREF_SHARED_INOUT` | 0x0a | 共享内存双向 |
| `TEEC_MEMREF_WHOLE` | 0xc | 整个共享内存 |
| `TEEC_MEMREF_PARTIAL_INPUT` | 0xd | 部分共享内存输入 |
| `TEEC_MEMREF_PARTIAL_OUTPUT` | 0xe | 部分共享内存输出 |
| `TEEC_MEMREF_PARTIAL_INOUT` | 0xf | 部分共享内存双向 |

### 3.2 参数类型构建宏

```c
// 构建参数类型值 (4个参数，每个4位)
#define TEEC_PARAM_TYPES(param0Type, param1Type, param2Type, param3Type) \
    ((param3Type) << 12 | (param2Type) << 8 | (param1Type) << 4 | (param0Type))

// 获取指定索引的参数类型
#define TEEC_PARAM_TYPE_GET(paramTypes, index) \
    (((paramTypes) >> (4*(index))) & 0x0F)
```

**使用示例**:
```c
// params[0]: Value Input
// params[1]: Temp Output
// params[2]: 未使用
// params[3]: 未使用
operation.paramTypes = TEEC_PARAM_TYPES(
    TEEC_VALUE_INPUT,
    TEEC_MEMREF_TEMP_OUTPUT,
    TEEC_NONE,
    TEEC_NONE
);
```

---

## 4. 错误码定义

### 4.1 标准错误码

| 错误码 | 值 | 说明 |
|--------|------|------|
| `TEEC_SUCCESS` | 0x0 | 成功 |
| `TEEC_ERROR_INVALID_CMD` | - | 无效命令 |
| `TEEC_ERROR_SERVICE_NOT_EXIST` | - | TA 不存在 |
| `TEEC_ERROR_SESSION_NOT_EXIST` | - | 会话不存在 |
| `TEEC_ERROR_SESSION_MAXIMUM` | - | 会话数达上限 |
| `TEEC_ERROR_TRUSTED_APP_LOAD_ERROR` | - | TA 加载失败 |
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
| `TEEC_ERROR_BUSY` | 0xFFFF000D | 系统繁忙 |
| `TEEC_ERROR_COMMUNICATION` | 0xFFFF000E | 通信错误 |
| `TEEC_ERROR_SECURITY` | 0xFFFF000F | 安全错误 |
| `TEEC_ERROR_SHORT_BUFFER` | 0xFFFF0010 | 缓冲区不足 |
| `TEEC_ERROR_MAC_INVALID` | 0xFFFF3071 | MAC 校验失败 |
| `TEEC_ERROR_TARGET_DEAD` | 0xFFFF3024 | TA 崩溃 |

### 4.2 错误来源 (returnOrigin)

| 值 | 说明 |
|------|------|
| `TEEC_ORIGIN_API` | 错误来自客户端 API |
| `TEEC_ORIGIN_COMMS` | 错误来自 REE/TEE 通信 |
| `TEEC_ORIGIN_TEE` | 错误来自 TEE 代码 |
| `TEEC_ORIGIN_TRUSTED_APP` | 错误来自 TA 代码 |

---

## 5. 登录方法

### 5.1 TEEC_LoginMethod 枚举

| 登录方法 | 值 | 说明 |
|----------|------|------|
| `TEEC_LOGIN_PUBLIC` | 0x0 | 无登录数据 |
| `TEEC_LOGIN_USER` | - | 用户登录数据 |
| `TEEC_LOGIN_GROUP` | - | 组登录数据 |
| `TEEC_LOGIN_APPLICATION` | 0x4 | 应用登录数据 |
| `TEEC_LOGIN_USER_APPLICATION` | 0x5 | 用户+应用登录数据 |
| `TEEC_LOGIN_GROUP_APPLICATION` | 0x6 | 组+应用登录数据 |
| `TEEC_LOGIN_IDENTIFY` | 0x7 | TEEOS 保留方法（唯一支持） |

**注意**: 当前实现**只支持** `TEEC_LOGIN_IDENTIFY`，其他登录方法会被拒绝。

---

## 6. 调用链与实现位置

### 6.1 libteec.so 调用链

```
TEEC_InitializeContext()
    └── libteec_client/tee_client.cpp
        └── 与 cadaemon IPC 通信

TEEC_OpenSession()
    └── libteec_client/tee_client.cpp
        ├── TEEC_GetApp() - TA 文件加载
        └── IPC -> cadaemon -> TZDriver

TEEC_InvokeCommand()
    └── libteec_client/tee_client.cpp
        └── IPC -> cadaemon -> TZDriver
```

### 6.2 libteec_vendor.so 调用链

```
TEEC_InitializeContext()
    └── libteec_vendor/tee_client_api.c:932
        └── CaDaemonConnectWithoutCaInfo()
            └── tee_client_socket.c: CaDaemonConnectWithCaInfo()
                └── Unix Domain Socket 连接 teecd

TEEC_OpenSession()
    └── libteec_vendor/tee_client_api.c:1205
        ├── TEEC_GetApp() - tee_client_app_load.c
        └── CaDaemonConnectWithCaInfo() + ioctl

TA 加载流程
    └── libteec_vendor/tee_client_app_load.c
        ├── 尝试从 taPath 加载
        └── 尝试从默认路径加载
```

---

## 7. API 使用限制

### 7.1 OpenSession 参数限制

| 限制项 | 说明 |
|--------|------|
| connectionMethod | 只支持 `TEEC_LOGIN_IDENTIFY` |
| connectionData | 必须为 NULL |
| operation->params[2] | 预留给系统，CA 不可用 |
| operation->params[3] | 预留给系统，CA 不可用 |

### 7.2 TA 文件路径限制

| 限制项 | 说明 |
|--------|------|
| 指定路径 | 只支持 `/data/` 目录 |
| 默认路径 | `/system/bin/` 和 `/vendor/bin/` |
| 文件格式 | UUID 命名 `.sec` 文件 |

**示例**:
```c
// 方式1: 使用 context.ta_path 指定
context.ta_path = (uint8_t *)"/data/58dbb3b9-4a0c-42d2-a84d-7c7ab17539fc.sec";

// 方式2: 使用默认路径（系统自动查找）
// 不设置 ta_path，会自动查找 /system/bin/{uuid}.sec 或 /vendor/bin/{uuid}.sec
```

---

## 8. 与 GlobalPlatform 标准的差异

本实现与 GlobalPlatform TEE Client API Specification v1.0 存在以下差异：

| 差异项 | GlobalPlatform 标准 | 本实现 |
|--------|---------------------|--------|
| Login Method | 支持 6 种方法 | 只支持 TEEC_LOGIN_IDENTIFY |
| TA Path | 标准未定义 | 支持通过 context.ta_path 指定 |
| params[2,3] | 未限制 | 预留给系统，CA 不可用 |
| Cancel | 标准支持 | 当前实现不生效 |
