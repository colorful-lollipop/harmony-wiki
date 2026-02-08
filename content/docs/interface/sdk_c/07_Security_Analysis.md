# 安全风险评审

> **目的**: 基于代码证据分析安全风险，识别攻击面和可被利用点  
> **适用范围**: 安全工程师、代码审计人员  
> **生成时间**: 2025-02-06

---

## 1. 轻量威胁模型

### 1.1 系统边界与信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         外部输入                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ 用户输入  │  │ 网络数据  │  │ 文件数据  │  │ IPC 调用  │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
└───────┼─────────────┼─────────────┼─────────────┼───────────────┘
        │             │             │             │
        ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界（应用沙箱）                        │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    N-API 接口层                          │   │
│   │   参数解析 → 类型检查 → 范围校验 → 权限检查             │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    系统服务层                            │   │
│   │   IPC 调用 → 身份校验 → 权限校验 → 业务逻辑             │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    内核/驱动层                           │   │
│   │   系统调用 → 硬件访问 → 敏感操作                         │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面清单

| 攻击面 | 风险等级 | 涉及模块 | 防护措施 |
|--------|----------|----------|----------|
| **N-API 参数解析** | 高 | napi, ace_engine | 类型检查、范围校验 |
| **IPC 通信** | 高 | IPCKit | Token ID、接口校验 |
| **文件操作** | 中 | filemanagement | 沙箱路径、权限检查 |
| **网络通信** | 中 | network | HTTPS、证书校验 |
| **密钥管理** | 高 | security/huks | 硬件安全存储 |
| **内存管理** | 中 | 所有模块 | 边界检查、空指针检查 |

---

## 2. 可被利用点分析（基于代码证据）

### 2.1 风险 #1: N-API 参数解析缺失

**证据**:
- 文件: `arkui/napi/native_api.h`
- 观察: 部分 API 未在头文件中显示参数校验逻辑

**风险描述**:
- 从 JS 传入的参数类型不匹配可能导致崩溃
- 字符串长度未检查可能导致缓冲区溢出

**影响**:
- 应用崩溃
- 潜在的内存损坏

**修复建议**:
```c
// 在 NAPI 函数入口进行严格校验
static napi_value MyApi(napi_env env, napi_callback_info info) {
    size_t argc = 1;
    napi_value args[1] = {nullptr};
    napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);
    
    // 1. 检查参数数量
    if (argc < 1) {
        napi_throw_error(env, nullptr, "Expected 1 argument");
        return nullptr;
    }
    
    // 2. 检查参数类型
    napi_valuetype type;
    napi_typeof(env, args[0], &type);
    if (type != napi_string) {
        napi_throw_type_error(env, nullptr, "Expected string");
        return nullptr;
    }
    
    // 3. 检查字符串长度
    size_t str_len;
    napi_get_value_string_utf8(env, args[0], nullptr, 0, &str_len);
    if (str_len > MAX_STRING_LEN) {
        napi_throw_range_error(env, nullptr, "String too long");
        return nullptr;
    }
    
    // ... 业务逻辑
}
```

### 2.2 风险 #2: 路径遍历攻击

**证据**:
- 文件: `filemanagement/fileio/include/oh_fileio.h`
- 观察: 文件路径操作接口

**风险描述**:
- 如果应用直接使用用户输入作为文件路径，可能导致路径遍历
- 示例: `../../../system/etc/passwd`

**影响**:
- 越权访问沙箱外文件
- 敏感信息泄露

**修复建议**:
```c
// 使用沙箱路径检查
bool IsValidPath(const char* path) {
    // 1. 检查路径是否在沙箱内
    if (!IsPathInSandbox(path)) {
        return false;
    }
    
    // 2. 检查路径规范化后是否包含 .. 跳转
    char resolved[PATH_MAX];
    if (realpath(path, resolved) == nullptr) {
        return false;
    }
    
    return IsPathInSandbox(resolved);
}
```

### 2.3 风险 #3: IPC 接口混淆攻击

**证据**:
- 文件: `IPCKit/ipc_cparcel.h:46-53`
- 代码:
```c
/**
 * @brief Writes an interface token to an OHIPCParcel object for interface identity verification.
 */
int OH_IPCParcel_WriteInterfaceToken(OHIPCParcel *parcel, const char *token);

/**
 * @brief Reads an interface token from an OHIPCParcel object for interface identity verification.
 */
int OH_IPCParcel_ReadInterfaceToken(const OHIPCParcel *parcel, char **token, 
    int32_t *len, OH_IPC_MemAllocator allocator);
```

**风险描述**:
- 如果服务未验证 Interface Token，攻击者可能调用非预期接口

**影响**:
- 未授权操作
- 权限提升

**修复建议**:
```c
// 服务端必须验证 Interface Token
void OnRemoteRequest(uint32_t code, OHIPCParcel *data, OHIPCParcel *reply) {
    // 1. 读取 Token
    char *token = nullptr;
    int32_t len = 0;
    OH_IPCParcel_ReadInterfaceToken(data, &token, &len, allocator);
    
    // 2. 验证 Token
    if (strcmp(token, EXPECTED_INTERFACE_TOKEN) != 0) {
        // Token 不匹配，拒绝请求
        return ERR_INVALID_TOKEN;
    }
    
    // 3. 处理请求
    // ...
}
```

### 2.4 风险 #4: 密钥别名长度未检查

**证据**:
- 文件: `security/huks/include/native_huks_type.h:91`
- 代码:
```c
#define OH_HUKS_MAX_KEY_ALIAS_LEN 64
```

**风险描述**:
- 虽然定义了最大长度，但需要在所有使用处检查
- 超长别名可能导致缓冲区溢出

**影响**:
- 内存损坏
- 潜在的代码执行

**修复建议**:
```c
struct OH_Huks_Result OH_Huks_GenerateKeyItem(
    const struct OH_Huks_Blob *keyAlias,
    const struct OH_Huks_ParamSet *paramSetIn,
    struct OH_Huks_ParamSet *paramSetOut) {
    
    // 1. 检查别名长度
    if (keyAlias == nullptr || keyAlias->size > OH_HUKS_MAX_KEY_ALIAS_LEN) {
        return OH_Huks_Result{OH_HUKS_ERR_CODE_ILLEGAL_ARGUMENT, ...};
    }
    
    // 2. 检查别名内容（只允许合法字符）
    if (!IsValidKeyAlias(keyAlias->data, keyAlias->size)) {
        return OH_Huks_Result{OH_HUKS_ERR_CODE_ILLEGAL_ARGUMENT, ...};
    }
    
    // ... 业务逻辑
}
```

### 2.5 风险 #5: 反序列化数据未验证

**证据**:
- 文件: `arkui/napi/native_api.h:644-662`
- 代码:
```c
/**
 * @brief Restore serialization data to an ArkTS object.
 */
NAPI_EXTERN napi_status napi_deserialize(napi_env env,
                                         void* buffer,
                                         napi_value* object);
```

**风险描述**:
- 反序列化操作如果未验证数据来源，可能导致对象注入攻击

**影响**:
- 恶意对象注入
- 潜在的远程代码执行

**修复建议**:
```c
// 1. 仅接受来自可信源的序列化数据
// 2. 添加数据完整性校验（HMAC）
// 3. 限制反序列化数据大小

napi_status SafeDeserialize(napi_env env, void* buffer, size_t buffer_size, 
                            const uint8_t* hmac, napi_value* object) {
    // 1. 检查缓冲区大小
    if (buffer_size > MAX_DESERIALIZE_SIZE) {
        return napi_generic_failure;
    }
    
    // 2. 验证 HMAC
    if (!VerifyHMAC(buffer, buffer_size, hmac)) {
        return napi_generic_failure;
    }
    
    // 3. 反序列化
    return napi_deserialize(env, buffer, object);
}
```

---

## 3. 信任边界与数据流

### 3.1 数据流图

```
外部输入
    │
    ├─> N-API 层（参数校验、类型检查）
    │       │
    │       ├─> 校验失败 → 抛出异常
    │       │
    │       └─> 校验通过 → IPC 调用
    │                   │
    │                   ├─> IPC 层（Token 校验、身份认证）
    │                   │       │
    │                   │       ├─> 认证失败 → 返回错误
    │                   │       │
    │                   │       └─> 认证通过 → 系统服务
    │                   │                   │
    │                   │                   ├─> 权限检查
    │                   │                   │       │
    │                   │                   │       ├─> 无权限 → 返回拒绝
    │                   │                   │       │
    │                   │                   │       └─> 有权限 → 业务逻辑
    │                   │                   │                   │
    │                   │                   │                   └─> 敏感操作
    │                   │                   │                               │
    │                   │                   │                               ├─> 加密/密钥
    │                   │                   │                               ├─> 文件操作
    │                   │                   │                               └─> 网络通信
```

---

## 4. 检查范围与局限性

### 4.1 已覆盖范围

| 检查项 | 覆盖范围 | 状态 |
|--------|----------|------|
| N-API 参数校验 | 头文件声明检查 | ✅ 已覆盖 |
| IPC 安全机制 | IPCKit 代码分析 | ✅ 已覆盖 |
| 密钥管理安全 | HUKS 接口分析 | ✅ 已覆盖 |
| 文件操作安全 | FileIO 头文件检查 | ✅ 已覆盖 |
| 输入长度检查 | 边界常量定义检查 | ✅ 已覆盖 |

### 4.2 局限性

1. **实现代码未分析**: 本仓库仅包含接口声明，具体实现在其他仓库
2. **测试代码未分析**: 按规范未分析 test/ 目录
3. **动态行为未知**: 静态代码分析无法覆盖运行时行为
4. **业务逻辑未深入**: 各模块业务逻辑安全风险需单独分析

---

## 5. 代码证据

| 结论 | 证据文件 | 关键内容 |
|------|----------|----------|
| IPC Token 机制 | `IPCKit/ipc_cparcel.h` | Interface Token 验证 |
| 密钥长度限制 | `security/huks/include/native_huks_type.h:91` | OH_HUKS_MAX_KEY_ALIAS_LEN |
| 反序列化 API | `arkui/napi/native_api.h:644-662` | napi_deserialize |
| 权限错误码 | `AbilityKit/ability_runtime/ability_runtime_common.h` | PERMISSION_DENIED = 201 |

---

## 6. 相关跳转

- **上一章**: [编译产物](./06_Build_Artifacts.md)
- **下一章**: [常见问题](./08_Common_Issues.md)
- **IPC 安全**: `IPCKit/ipc_cskeleton.h`
- **密钥管理**: `security/huks/include/native_huks_api.h`
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**安全风险评审文档 - 基于代码生成**
