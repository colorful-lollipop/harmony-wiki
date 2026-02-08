# Native API 参考

## 重要说明

**本仓库不包含 N-API 层**，所有 API 均为 Native C/C++ 接口。

JavaScript 应用通过以下路径访问 TEE 功能：

```
OpenHarmony App (JS)
    ↓ (OpenHarmony Framework N-API 层 - 独立仓库)
CA (Client Application - Native C)
    ↓ SMC
TEE OS Framework (本仓库 - Native)
    ↓ GP API
TA (Trusted Application)
```

---

## 1. GP TEE API 概述

### 1.1 API 分类

| 分类 | 头文件 | 描述 |
|------|--------|------|
| **核心 API** | `tee_internal_api.h` | TA 生命周期、会话管理 |
| **可信存储 API** | `tee_trusted_storage_api.h` | 安全文件操作 |
| **加密 API** | `tee_crypto_api.h` | 加解密、哈希、签名 |
| **时间 API** | `tee_time_api.h` | 安全时间获取 |
| **扩展 API** | `tee_hw_ext_api.h` | 硬件扩展（HUK、RPMB、TUI） |

### 1.2 API 头文件位置

```
lib/teelib/
├── libteeos/include/tee/
│   ├── tee_internal_api.h      # 核心 API
│   ├── tee_core_api.h          # 核心类型和函数
│   ├── tee_defines.h           # 常量定义
│   └── tee_common.h             # 公共定义
├── libssa/include/
│   └── tee_trusted_storage_api.h  # 可信存储
├── libcrypto/include/
│   └── tee_crypto_api.h        # 加密 API
└── libtee_stub/include/
    ├── tee_hw_ext_api.h        # 硬件扩展
    ├── rpmb_fcntl.h            # RPMB 存储
    └── tee_rtc_time_api.h      # RTC 时间
```

---

## 2. TA 入口点

### 2.1 必需入口点

所有 TA 必须实现以下入口点函数：

```c
// 证据: lib/teelib/libtaentry/src/elf_main_entry.c:126-159

/**
 * TA 创建时调用
 * @param session_context 输出会话上下文
 * @param param_types 参数类型掩码
 * @param params 参数数组
 * @return TEE_Result
 */
TEE_Result TA_CreateEntryPoint(void);

/**
 * 打开会话
 * @param session_context 会话上下文
 * @param param_types 参数类型掩码
 * @param params 参数数组
 * @return TEE_Result
 */
TEE_Result TA_OpenSessionEntryPoint(
    uint32_t* session_context,
    uint32_t param_types,
    TEE_Param params[4]
);

/**
 * 调用命令
 * @param session_context 会话上下文
 * @param cmd_id 命令 ID
 * @param param_types 参数类型掩码
 * @param params 参数数组
 * @return TEE_Result
 */
TEE_Result TA_InvokeCommandEntryPoint(
    uint32_t session_context,
    uint32_t cmd_id,
    uint32_t param_types,
    TEE_Param params[4]
);

/**
 * 关闭会话
 * @param session_context 会话上下文
 * @param param_types 参数类型掩码
 * @param params 参数数组
 */
void TA_CloseSessionEntryPoint(
    uint32_t session_context,
    uint32_t param_types,
    TEE_Param params[4]
);

/**
 * 销毁 TA
 */
void TA_DestroyEntryPoint(void);
```

### 2.2 调用顺序

```
1. TA_CreateEntryPoint()          - TA 加载时调用一次
2. TA_OpenSessionEntryPoint()     - 每次打开会话
   └── TA_InvokeCommandEntryPoint() - 会话期间多次调用
3. TA_CloseSessionEntryPoint()    - 关闭会话
4. TA_DestroyEntryPoint()         - TA 卸载时调用
```

---

## 3. 核心 API

### 3.1 类型定义

```c
// 证据: lib/teelib/libteeos/include/tee/tee_defines.h

typedef struct {
    uint32_t timeLow;
    uint16_t timeMid;
    uint16_t timeHiAndVersion;
    uint8_t clockSeqAndNode[8];
} TEE_UUID;

typedef struct {
    uint32_t seconds;
    uint32_t millis;
} TEE_Time;

typedef enum {
    TEE_PARAM_TYPE_NONE = 0,
    TEE_PARAM_TYPE_VALUE_INPUT = 1,
    TEE_PARAM_TYPE_VALUE_OUTPUT = 2,
    TEE_PARAM_TYPE_VALUE_INOUT = 3,
    TEE_PARAM_TYPE_MEMREF_INPUT = 5,
    TEE_PARAM_TYPE_MEMREF_OUTPUT = 6,
    TEE_PARAM_TYPE_MEMREF_INOUT = 7,
} TEE_ParamType;

typedef struct {
    union {
        uint32_t value;          // 值参数
        struct {                 // 内存参数
            void* buffer;        // 缓冲区
            uint32_t size;       // 大小
        } memref;
    } a;
    // ... 更多联合成员
} TEE_Param;

typedef struct {
    uint32_t paramTypes;         // 参数类型掩码
    TEE_Param params[4];         // 参数数组
} TEE_Operation;

typedef int32_t TEE_Result;

#define TEE_SUCCESS                      0x00000000
#define TEE_ERROR_GENERIC                0xFFFF0000
#define TEE_ERROR_ACCESS_DENIED          0xFFFF0001
#define TEE_ERROR_CANCEL                 0xFFFF0002
#define TEE_ERROR_OVERWRITE              0xFFFF0003
#define TEE_ERROR_EXPIRED                0xFFFF0004
#define TEE_ERROR_ITEM_NOT_FOUND         0xFFFF0006
// ... 更多错误码
```

### 3.2 内存管理

```c
/**
 * 分配内存
 */
void* TEE_Malloc(uint32_t size, uint32_t flags);

/**
 * 重新分配内存
 */
void* TEE_Realloc(void* buffer, uint32_t newSize);

/**
 * 释放内存
 */
void TEE_Free(void* buffer);

/**
 * 清零内存（安全）
 */
void TEE_MemFill(void* buffer, uint32_t ch, uint32_t size);

/**
 * 内存比较
 */
int32_t TEE_MemCompare(const void* buffer1, const void* buffer2, uint32_t size);

/**
 * 内存复制
 */
void TEE_MemMove(void* dest, const void* src, uint32_t size);
```

---

## 4. 可信存储 API

### 4.1 对象创建

```c
// 证据: lib/teelib/libssa/include/tee_trusted_storage_api.h

/**
 * 创建持久对象
 * @param objectID 对象 ID
 * @param objectIDLen ID 长度
 * @param attributes 对象属性
 * @param flags 创建标志
 * @param initialData 初始数据
 * @param initialDataLen 初始数据长度
 * @param object 输出对象引用
 * @return TEE_Result
 */
TEE_Result TEE_CreatePersistentObject(
    uint32_t objectID,
    uint32_t objectIDLen,
    TEE_Attribute* attributes,
    uint32_t attrCount,
    uint32_t flags,
    void* initialData,
    uint32_t initialDataLen,
    TEE_ObjectHandle* object
);

/**
 * 打开持久对象
 * @param objectID 对象 ID
 * @param objectIDLen ID 长度
 * @param object 输出对象引用
 * @return TEE_Result
 */
TEE_Result TEE_OpenPersistentObject(
    uint32_t objectID,
    uint32_t objectIDLen,
    TEE_ObjectHandle* object,
    uint32_t flags
);
```

### 4.2 数据读写

```c
/**
 * 读取对象数据
 * @param object 对象句柄
 * @param offset 偏移
 * @param buffer 输出缓冲区
 * @param bufferLen 缓冲区长度
 * @return 实际读取长度
 */
uint32_t TEE_ReadObjectData(
    TEE_ObjectHandle object,
    void* buffer,
    uint32_t bufferLen,
    uint32_t offset
);

/**
 * 写入对象数据
 * @param object 对象句柄
 * @param buffer 输入缓冲区
 * @param bufferLen 数据长度
 * @param offset 偏移
 * @return TEE_Result
 */
TEE_Result TEE_WriteObjectData(
    TEE_ObjectHandle object,
    void* buffer,
    uint32_t bufferLen,
    uint32_t offset
);

/**
 * 关闭并释放对象
 */
void TEE_CloseObject(TEE_ObjectHandle object);

/**
 * 删除持久对象
 */
TEE_Result TEE_DeletePersistentObject(TEE_ObjectHandle object);
```

---

## 5. 加密 API

### 5.1 初始化操作

```c
// 证据: lib/teelib/libcrypto/include/tee_crypto_api.h

/**
 * 分配加密操作
 */
TEE_Result TEE_AllocateOperation(
    TEE_OperationHandle* operation,
    uint32_t algorithm,
    uint32_t mode,
    uint32_t maxKeySize
);

/**
 * 释放加密操作
 */
void TEE_FreeOperation(TEE_OperationHandle operation);

/**
 * 设置密钥
 */
void TEE_SetOperationKey(
    TEE_OperationHandle operation,
    TEE_ObjectHandle key
);

/**
 * 设置密钥参数
 */
void TEE_SetOperationKey2(
    TEE_OperationHandle operation,
    TEE_ObjectHandle key1,
    TEE_ObjectHandle key2
);
```

### 5.2 摘要操作

```c
/**
 * 初始化摘要操作
 */
void TEE_DigestInit(TEE_OperationHandle operation);

/**
 * 更新摘要
 */
void TEE_DigestUpdate(
    TEE_OperationHandle operation,
    void* buffer,
    uint32_t bufferLen
);

/**
 * 完成摘要计算
 */
TEE_Result TEE_DigestDoFinal(
    TEE_OperationHandle operation,
    void* buffer,
    uint32_t bufferLen,
    void* hash,
    uint32_t* hashLen
);
```

### 5.3 对称加密

```c
/**
 * 初始化加密
 */
void TEE_CipherInit(
    TEE_OperationHandle operation,
    void* IV,
    uint32_t IVLen
);

/**
 * 更新加密/解密
 */
TEE_Result TEE_CipherUpdate(
    TEE_OperationHandle operation,
    void* srcData,
    uint32_t srcDataLen,
    void* destData,
    uint32_t* destDataLen
);

/**
 * 完成加密/解密
 */
TEE_Result TEE_CipherDoFinal(
    TEE_OperationHandle operation,
    void* srcData,
    uint32_t srcDataLen,
    void* destData,
    uint32_t* destDataLen
);
```

### 5.4 MAC 操作

```c
/**
 * 初始化 MAC
 */
void TEE_MACInit(
    TEE_OperationHandle operation,
    void* IV,
    uint32_t IVLen
);

/**
 * 更新 MAC
 */
void TEE_MACUpdate(
    TEE_OperationHandle operation,
    void* buffer,
    uint32_t bufferLen
);

/**
 * 完成 MAC 计算
 */
TEE_Result TEE_MACComputeFinal(
    TEE_OperationHandle operation,
    void* buffer,
    uint32_t bufferLen,
    void* mac,
    uint32_t* macLen
);
```

---

## 6. TA2TA 通信 API

### 6.1 打开 TA 会话

```c
// 证据: lib/teelib/libteeos/include/tee/tee_core_api.h

/**
 * 打开与另一个 TA 的会话
 * @param ta 目标 TA UUID
 * @param cancellationEvent 取消事件
 * @param paramTypes 参数类型
 * @param params 参数
 * @param session 输出会话引用
 * @param returnOrigin 返回来源
 * @return TEE_Result
 */
TEE_Result TEE_OpenTASession(
    const TEE_UUID* ta,
    uint32_t cancellationEvent,
    uint32_t paramTypes,
    TEE_Param params[4],
    TEE_ObjectHandle* session,
    uint32_t* returnOrigin
);

/**
 * 调用 TA 命令
 * @param session 会话句柄
 * @param cancellationEvent 取消事件
 * @param commandID 命令 ID
 * @param paramTypes 参数类型
 * @param params 参数
 * @param returnOrigin 返回来源
 * @return TEE_Result
 */
TEE_Result TEE_InvokeTACommand(
    TEE_ObjectHandle session,
    uint32_t cancellationEvent,
    uint32_t commandID,
    uint32_t paramTypes,
    TEE_Param params[4],
    uint32_t* returnOrigin
);

/**
 * 关闭 TA 会话
 */
void TEE_CloseTASession(TEE_ObjectHandle session);
```

---

## 7. 错误码参考

| 错误码 | 值 | 描述 |
|--------|-----|------|
| `TEE_SUCCESS` | 0x00000000 | 成功 |
| `TEE_ERROR_GENERIC` | 0xFFFF0000 | 通用错误 |
| `TEE_ERROR_ACCESS_DENIED` | 0xFFFF0001 | 访问拒绝 |
| `TEE_ERROR_CANCEL` | 0xFFFF0002 | 操作取消 |
| `TEE_ERROR_OVERWRITE` | 0xFFFF0003 | 覆盖错误 |
| `TEE_ERROR_EXPIRED` | 0xFFFF0004 | 已过期 |
| `TEE_ERROR_ITEM_NOT_FOUND` | 0xFFFF0006 | 项未找到 |
| `TEE_ERROR_BAD_PARAMETERS` | 0xFFFF0007 | 错误参数 |
| `TEE_ERROR_SHORT_BUFFER` | 0xFFFF000A | 缓冲区不足 |
| `TEE_ERROR_OUT_OF_MEMORY` | 0xFFFF000C | 内存不足 |
| `TEE_ERROR_NOT_IMPLEMENTED` | 0xFFFF000F | 未实现 |
| `TEE_ERROR_BUSY` | 0xFFFF000E | 资源忙 |
| `TEE_ERROR_COMMUNICATION` | 0xFFFF000E | 通信错误 |

---

## 8. 参数类型

| 类型 | 值 | 描述 |
|------|-----|------|
| `TEE_PARAM_TYPE_NONE` | 0 | 无参数 |
| `TEE_PARAM_TYPE_VALUE_INPUT` | 1 | 值输入 |
| `TEE_PARAM_TYPE_VALUE_OUTPUT` | 2 | 值输出 |
| `TEE_PARAM_TYPE_VALUE_INOUT` | 3 | 值输入输出 |
| `TEE_PARAM_TYPE_MEMREF_INPUT` | 5 | 内存引用输入 |
| `TEE_PARAM_TYPE_MEMREF_OUTPUT` | 6 | 内存引用输出 |
| `TEE_PARAM_TYPE_MEMREF_INOUT` | 7 | 内存引用输入输出 |

---

## 9. 算法标识

### 9.1 对称算法

| 算法 | 标识 |
|------|------|
| `TEE_ALG_AES_ECB_NOPAD` | 0x10000100 |
| `TEE_ALG_AES_CBC_NOPAD` | 0x10000101 |
| `TEE_ALG_AES_CTR` | 0x10000104 |
| `TEE_ALG_AES_XTS` | 0x10000105 |
| `TEE_ALG_DES_ECB_NOPAD` | 0x10000200 |
| `TEE_ALG_DES_CBC_NOPAD` | 0x10000201 |

### 9.2 摘要算法

| 算法 | 标识 |
|------|------|
| `TEE_ALG_SHA1` | 0x00001000 |
| `TEE_ALG_SHA224` | 0x00001001 |
| `TEE_ALG_SHA256` | 0x00001002 |
| `TEE_ALG_SHA384` | 0x00001003 |
| `TEE_ALG_SHA512` | 0x00001004 |
| `TEE_ALG_SM3` | 0x00001005 |

### 9.3 非对称算法

| 算法 | 标识 |
|------|------|
| `TEE_ALG_RSA_NOPAD` | 0x00002000 |
| `TEE_ALG_RSA_PKCS1_V1_5` | 0x00002001 |
| `TEE_ALG_RSA_PKCS1_PSS` | 0x00002002 |
| `TEE_ALG_ECDSA` | 0x00003000 |
| `TEE_ALG_SM2` | 0x00003001 |

### 9.4 MAC 算法

| 算法 | 标识 |
|------|------|
| `TEE_ALG_HMAC_SHA1` | 0x00004000 |
| `TEE_ALG_HMAC_SHA224` | 0x00004001 |
| `TEE_ALG_HMAC_SHA256` | 0x00004002 |
| `TEE_ALG_HMAC_SM3` | 0x00004005 |

---

## 10. 使用示例

### 10.1 简单 TA 示例

```c
#include <tee_internal_api.h>

/* TA UUID - 需替换为实际 UUID */
const TEE_UUID TA_uuid = {
    0x12345678, 0x1234, 0x5678,
    {0x12, 0x34, 0x56, 0x78, 0x9A, 0xBC, 0xDE, 0xF0}
};

TEE_Result TA_CreateEntryPoint(void)
{
    return TEE_SUCCESS;
}

TEE_Result TA_OpenSessionEntryPoint(
    uint32_t* session_context,
    uint32_t param_types,
    TEE_Param params[4])
{
    (void)param_types;
    (void)params;
    *session_context = 0;
    return TEE_SUCCESS;
}

TEE_Result TA_InvokeCommandEntryPoint(
    uint32_t session_context,
    uint32_t cmd_id,
    uint32_t param_types,
    TEE_Param params[4])
{
    (void)session_context;
    (void)param_types;
    (void)params;
    
    switch (cmd_id) {
        case 0x100: /* 自定义命令 */
            /* 处理命令 */
            return TEE_SUCCESS;
        default:
            return TEE_ERROR_BAD_PARAMETERS;
    }
}

void TA_CloseSessionEntryPoint(
    uint32_t session_context,
    uint32_t param_types,
    TEE_Param params[4])
{
    (void)session_context;
    (void)param_types;
    (void)params;
}

void TA_DestroyEntryPoint(void)
{
}
```

---

## 相关文档

- 模块详情 → `02_Module_Detail.md`
- 架构设计 → `01_Architecture.md`
- 构建系统 → `04_Build_System.md`
- 安全评审 → `05_Security_Review.md`
