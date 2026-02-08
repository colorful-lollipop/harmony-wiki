# 示例代码说明

> **阅读时间**: 20 分钟 | **目标**: 参考完整示例，学习 TA/CA 开发

---

## 概述

tee_dev_kit 提供多个 TA/CA 示例，覆盖基本功能和安全操作。每个示例都包含完整的 TA 端代码和对应的 CA 端代码，演示如何实现特定的安全功能。

**示例列表**:

| 名称 | 功能 | TA 路径 | CA 路径 | 复杂度 |
|------|------|---------|---------|--------|
| **helloworld_demo** | 基础入门示例 | `sdk/src/TA/helloworld_demo/` | `sdk/src/CA/helloworld_demo/` | ⭐ |
| **aes_demo** | AES 加解密 | `sdk/src/TA/aes_demo/` | `sdk/src/CA/aes_demo/` | ⭐⭐ |
| **rsa_demo** | RSA 签名/验签 | `sdk/src/TA/rsa_demo/` | `sdk/src/CA/rsa_demo/` | ⭐⭐⭐ |
| **mac_demo** | MAC 计算 | `sdk/src/TA/mac_demo/` | `sdk/src/CA/mac_demo/` | ⭐⭐ |
| **secstorage_demo** | 安全存储 | `sdk/src/TA/secstorage_demo/` | `sdk/src/CA/secstorage_demo/` | ⭐⭐⭐ |

**阅读建议**:
- 新手: 从 helloworld_demo 开始
- 中级: 掌握 aes_demo 和 mac_demo
- 高级: 学习 rsa_demo 和 secstorage_demo

---

## 示例结构说明

### CA ↔ TA 配对关系

每个功能都有对应的 CA 和 TA 代码：

```
                    ┌─────────────────────────────────────┐
                    │         功能实现示例                  │
                    └─────────────────────────────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            ↓                       ↓                       ↓
    ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
    │  CA 端代码     │    │   编译工具     │    │  TA 端代码     │
    │ (REE 侧运行)   │    │   (SDK)       │    │ (TEE 侧运行)   │
    └───────────────┘    └───────────────┘    └───────────────┘
            │                       │                       │
            └───────────────────────┼───────────────────────┘
                                    ↓
                          ┌─────────────────┐
                          │  .sec 格式     │
                          │  TA 安装包     │
                          └─────────────────┘
```

**代码证据**: `sdk/src/CA/helloworld_demo/ca_demo.c` ↔ `sdk/src/TA/helloworld_demo/ta_demo.c`

### 示例文件结构

```
sdk/src/
├── TA/
│   ├── helloworld_demo/
│   │   ├── ta_demo.c           # TA 入口实现
│   │   └── configs.xml          # TA 配置
│   ├── aes_demo/
│   │   ├── aes_demo_ta.c       # AES 加解密实现
│   │   └── configs.xml
│   └── ...
└── CA/
    ├── helloworld_demo/
    │   ├── ca_demo.c           # CA 调用示例
    │   └── Makefile
    ├── aes_demo/
    │   ├── aes_demo_ca.c       # CA 调用端
    │   └── Makefile
    └── ...
```

### 功能说明

最基础的 TA 示例，演示 TA 的基本结构和命令调用。

### TA 源码分析

**证据**: `sdk/src/TA/helloworld_demo/ta_demo.c`

```c
// 命令定义
enum {
    CMD_GET_TA_VERSION = 1,
};

// 获取 TA 版本
static TEE_Result get_ta_version(char* buffer, size_t *buf_len)
{
    const char *version = TA_TEMPLATE_VERSION;  // "demo_20200601"

    if (*buf_len < strlen(version) + 1) {
        tloge("buffer is too short");
        *buf_len = strlen(version) + 1;
        return TEE_ERROR_SHORT_BUFFER;
    }

    errno_t err = strncpy_s(buffer, *buf_len, version, strlen(version) + 1);
    if (err != EOK)
        return TEE_ERROR_SECURITY;

    *buf_len = strlen(version) + 1;
    return TEE_SUCCESS;
}
```

### CA 源码分析

**证据**: `sdk/src/CA/helloworld_demo/ca_demo.c`

```c
int main(void)
{
    TEEC_Context context;
    TEEC_Session session;
    TEEC_Operation operation;
    TEEC_Result result;

    // 1. 初始化 Context
    TEEC_InitializeContext(NULL, &context);

    // 2. 打开会话
    result = TEEC_OpenSession(&context, &session, &UUID,
        TEEC_LOGIN_PUBLIC, NULL, NULL, NULL);

    // 3. 设置操作参数
    operation.paramTypes = TEEC_PARAM_TYPES(
        TEEC_NONE, TEEC_NONE, TEEC_NONE, TEEC_MEMREF_TEMP_OUTPUT);
    operation.params[3].tmpref.buffer = buffer;
    operation.params[3].tmpref.size = sizeof(buffer);

    // 4. 调用命令
    result = TEEC_InvokeCommand(&session, CMD_GET_TA_VERSION,
        &operation, NULL);

    // 5. 清理
    TEEC_CloseSession(&session);
    TEEC_FinalizeContext(&context);

    return 0;
}
```

### 关键学习点

1. ✅ TA 入口函数实现
2. ✅ 参数类型检查
3. ✅ 缓冲区安全处理
4. ✅ CA 调用流程

---

## aes_demo - 加密示例

### 功能说明

演示 TEE 中的 AES 加解密操作。

### TA 关键代码

**证据**: `sdk/src/TA/aes_demo/aes_demo_ta.c`

```c
// AES 加密命令
case CMD_AES_ENCRYPT:
    // 获取明文
    char *plaintext = params[0].memref.buffer;
    size_t plaintext_len = params[0].memref.size;

    // 生成随机 IV
    uint8_t iv[TEE_AES_BLOCK_SIZE];
    TEE_GenerateRandom(iv, sizeof(iv));

    // 创建加密上下文
    TEE_OperationHandle op = NULL;
    TEE_AllocateOperation(&op, TEE_ALG_AES_CBC_NOPAD,
        TEE_MODE_ENCRYPT, key_size);

    // 设置密钥
    TEE_SetKey(op, &aes_key, sizeof(aes_key));

    // 执行加密
    TEE_CipherDoFinal(op, plaintext, plaintext_len,
        ciphertext, &ciphertext_len);

    // 释放资源
    TEE_FreeOperation(op);
    break;
```

### CA 关键代码

**证据**: `sdk/src/CA/aes_demo/aes_demo_ca.c`

```c
// 1. 生成密钥（如果需要）
TEEC_Operation op;
op.paramTypes = TEEC_PARAM_TYPES(
    TEEC_VALUE_INPUT, TEEC_MEMREF_OUTPUT, TEEC_NONE, TEEC_NONE);

// 2. 调用加密
result = TEEC_InvokeCommand(&session, CMD_AES_ENCRYPT, &op, NULL);
```

---

## rsa_demo - 签名示例

### 功能说明

演示 RSA 密钥生成、签名和验签操作。

### TA 关键代码

**证据**: `sdk/src/TA/rsa_demo/rsa_demo_ta.c`

```c
// RSA 签名命令
case CMD_RSA_SIGN:
    // 1. 生成 RSA 密钥对
    TEE_Attribute attrs[] = {
        TEE_ATTR_RSA_MODULUS, rsa_modulus, rsa_modulus_size,
        TEE_ATTR_RSA_PUBLIC_EXPONENT, rsa_public_exp, 3,
    };
    TEE_GenerateKey(rsa_key, key_size, attrs, 4);

    // 2. 执行签名
    TEE_OperationHandle op = NULL;
    TEE_AllocateOperation(&op, TEE_ALG_RSASSA_PKCS1_V1_5_SHA256,
        TEE_MODE_SIGN, key_size);
    TEE_SetKey(op, &rsa_key, sizeof(rsa_key));

    uint32_t signature_len = signature_size;
    TEE_Sign(op, hash, hash_len, signature, &signature_len);
    break;
```

---

## mac_demo - 消息认证码示例

### 功能说明

演示 HMAC 消息认证码计算。

### TA 关键代码

**证据**: `sdk/src/TA/mac_demo/mac_demo.c`

```c
// HMAC 计算
case CMD_MAC:
    TEE_OperationHandle op = NULL;
    TEE_AllocateOperation(&op, TEE_ALG_HMAC_SHA256,
        TEE_MODE_MAC, mac_size);

    TEE_SetKey(op, &hmac_key, sizeof(hmac_key));
    TEE_MACComputeFinal(op, data, data_len, mac, &mac_len);
    TEE_FreeOperation(op);
    break;
```

---

## secstorage_demo - 安全存储示例

### 功能说明

演示 TEE 安全存储操作，用于持久化保存敏感数据。

### TA 关键代码

**证据**: `sdk/src/TA/secstorage_demo/secstorage_demo.c`

```c
// 安全存储写入
case CMD_SECSTORAGE_WRITE:
    // 1. 打开安全存储对象
    TEE_ObjectHandle object = NULL;
    result = TEE_OpenPersistentObject(TEE_STORAGE_PRIVATE,
        object_id, object_id_len,
        TEE_DATA_FLAG_ACCESS_WRITE, &object);

    // 2. 写入数据
    result = TEE_WriteObjectData(object, data, data_len);

    // 3. 关闭对象
    TEE_CloseObject(object);
    break;
```

---

## 示例结构对比

| 示例 | 命令数 | 输入参数 | 输出参数 | 安全特性 |
|------|--------|----------|----------|----------|
| helloworld_demo | 1 | 无 | 版本号字符串 | 无 |
| aes_demo | 3 | 明文 | 密文 + IV | 加密 |
| rsa_demo | 4 | 哈希值 | 签名 | 签名 |
| mac_demo | 2 | 数据 | MAC | 认证 |
| secstorage_demo | 4 | 数据/对象ID | 状态 | 持久化 |

---

## 运行示例

### 编译所有示例

```bash
# 编译 TA
cd sdk/build/TA_demo/
./build_ta.sh

# 编译 CA
cd sdk/src/CA/helloworld_demo/
make
```

### 执行示例

```bash
# 运行 helloworld_demo
./ca_demo
```

---

## 安全最佳实践

### 常见安全模式

| 实践 | 说明 | 示例位置 |
|------|------|---------|
| **参数校验** | 验证所有输入参数 | helloworld_demo: `check_param_type()` |
| **安全字符串** | 使用 `_s` 后缀安全函数 | helloworld_demo: `strncpy_s()` |
| **内存安全** | 使用 `TEE_Malloc/TEE_Free` | secstorage_demo |
| **错误处理** | 检查所有 API 返回值 | aes_demo |
| **密钥管理** | 密钥安全存储和使用 | rsa_demo |

### 安全编码示例

```c
// ✅ 好的实践：参数校验
static TEE_Result handle_secure_operation(TEE_Param params[])
{
    // 1. 验证参数类型
    if (!check_param_type(param_types,
        TEE_PARAM_TYPE_MEMREF_INPUT,
        TEE_PARAM_TYPE_VALUE_OUTPUT,
        TEE_PARAM_TYPE_NONE,
        TEE_PARAM_TYPE_NONE)) {
        return TEE_ERROR_BAD_PARAMETERS;
    }

    // 2. 验证缓冲区
    if (params[0].memref.buffer == NULL ||
        params[0].memref.size == 0) {
        return TEE_ERROR_BAD_PARAMETERS;
    }

    // 3. 检查缓冲区大小
    if (params[0].memref.size > MAX_BUFFER_SIZE) {
        return TEE_ERROR_OVERFLOW;
    }

    return TEE_SUCCESS;
}

// ❌ 避免：不校验输入
static TEE_Result unsafe_handle(TEE_Param params[])
{
    // 没有参数类型检查
    // 没有空指针检查
    // 没有缓冲区大小检查
    memcpy(dest, params[0].memref.buffer, params[0].memref.size);
    return TEE_SUCCESS;
}
```

### 资源管理

```c
// ✅ 好的实践：资源清理
static TEE_Result safe_operation(TEE_Param params[])
{
    TEE_OperationHandle op = NULL;
    TEE_ObjectHandle key = NULL;
    TEE_Result result = TEE_SUCCESS;

    do {
        // 分配资源
        result = TEE_AllocateOperation(&op, ...);
        if (result != TEE_SUCCESS) break;

        result = TEE_AllocateTransientObject(...);
        if (result != TEE_SUCCESS) break;

        // 使用资源
        // ...

    } while (0);

    // 清理资源（总是执行）
    if (op != NULL) TEE_FreeOperation(op);
    if (key != NULL) TEE_FreeTransientObject(key);

    return result;
}
```

---

## 编译和运行

### 环境准备

```bash
# 1. 设置 TEE_SDK_ROOT
export TEE_SDK_ROOT=/path/to/tee_dev_kit

# 2. 设置 LLVM 工具链
export PATH=$TEE_SDK_ROOT/prebuilts/clang/ohos/linux-x86_64/15.0.4/llvm/bin:$PATH

# 3. 编译 TA
cd $TEE_SDK_ROOT/sdk/build/TA_demo/
./build_ta.sh

# 4. 编译 CA
cd $TEE_SDK_ROOT/sdk/src/CA/helloworld_demo/
make
```

### 运行验证

```bash
# 运行 helloworld_demo
cd $TEE_SDK_ROOT/sdk/src/CA/helloworld_demo/
./ca_demo

# 预期输出
TA version: demo_20200601
```

---

## 继续阅读

### 推荐阅读路径

| 文档 | 阅读时间 | 目标 |
|------|---------|------|
| [02_TA_Development_Guide.md](02_TA_Development_Guide.md) | 30 分钟 | 开发第一个 TA |
| [04_Security_Review.md](04_Security_Review.md) | 30 分钟 | 安全最佳实践 |
| [03_Build_System.md](03_Build_System.md) | 20 分钟 | 构建系统详解 |

### 问题排查

遇到问题？请参考 [06_Troubleshooting.md](06_Troubleshooting.md)

---

## 变更历史

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0.0 | 2026-02-07 | 初始版本 |

---

*最后更新: 2026-02-07*
