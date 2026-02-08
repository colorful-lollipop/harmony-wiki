# 安全风险评审

## 概述

本文档对 KV Store 进行安全风险分析，涵盖攻击面、信任边界和潜在安全风险。

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              OpenHarmony 应用进程                   │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │          KV Store N-API 层                     │  │   │
│  │  │     (frameworks/jskitsimpl/)                  │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  │                        │                             │   │
│  │                        ▼                             │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │          KV Store Inner API 层                 │  │   │
│  │  │     (interfaces/innerkits/)                   │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  │                        │                             │   │
│  └─────────────────────────────────────────────────────┘   │
│                        │                                   │
│  ┌─────────────────────┴─────────────────────┐             │
│  │              跨进程通信 (IPC)              │             │
│  └─────────────────────────────────────────────┘             │
│                        │                                   │
│  ┌─────────────────────┴─────────────────────┐             │
│  │              KV Store 服务进程              │             │
│  │  ┌───────────────────────────────────────┐ │             │
│  │  │      核心实现 (distributeddb)           │ │             │
│  │  └───────────────────────────────────────┘ │             │
│  │                        │                     │             │
│  │                        ▼                     │             │
│  │  ┌───────────────────────────────────────┐ │             │
│  │  │            SQLite 存储                 │ │             │
│  │  └───────────────────────────────────────┘ │             │
│  └─────────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### 边界说明

| 区域 | 信任级别 | 说明 |
|-----|---------|------|
| 应用进程 | 高 | 同一设备上的可信应用 |
| IPC 通信 | 中 | 同设备进程间通信，受系统框架保护 |
| 服务进程 | 高 | 系统级服务，受权限控制 |

## 攻击面清单

### 1. N-API 接口层

**攻击向量**：应用通过 JS API 调用 KV Store

**风险点**：
- 输入参数校验不严 → 注入攻击
- 资源释放顺序问题 → UAF
- 异步回调处理不当 → 竞态条件

**证据**：`frameworks/jskitsimpl/distributedkvstore/src/js_util.cpp`

```cpp
// js_util.cpp:130-131
napi_valuetype type = napi_undefined;
napi_status status = napi_typeof(env, in, &type);
ASSERT((status == napi_ok) && (type == napi_string), "invalid type", ...);
```

### 2. 文件系统

**攻击向量**：数据库文件存储路径

**风险点**：
- 路径遍历 → 文件越权访问
- 符号链接攻击 → 替换数据库文件
- 权限配置不当 → 非授权读取

**证据**：`kv_store.gni:14`

```gn
kv_store_base_path = "//foundation/distributeddatamgr/kv_store"
```

### 3. 跨设备同步

**攻击向量**：分布式数据同步

**风险点**：
- 中间人攻击 → 数据篡改
- 设备伪造 → 身份冒充
- 同步冲突处理不当 → 数据覆盖

**证据**：`bundle.json:44`

```json
"features": [
  "kv_store_cloud",
  "kv_store_device"
]
```

### 4. 加密/解密

**攻击向量**：数据库加密

**风险点**：
- 密钥管理不当 → 密码泄露
- 弱加密算法 → 暴力破解
- 内存中的密钥 → 内存读取

**证据**：`BUILD.gn:52`

```gn
"SQLITE_HAS_CODEC",
```

## 安全机制

### 1. 权限控制

**机制**：基于 Bundle Name 的访问控制

**证据**：`interfaces/jskits/distributedkvstore/distributed_kvstore.js:17-18`

```javascript
createKVManager: distributedDataSo.createKVManager,
```

**说明**：通过 `bundleName` 限制应用访问权限

### 2. 安全级别

**机制**：SecurityLevel 枚举定义

**证据**：`interfaces/jskits/distributedkvstore/distributed_kvstore.js:56-63`

```javascript
SecurityLevel: {
  NO_LEVEL: 0,
  S0: 1,
  S1: 2,
  S2: 3,
  S3: 5,
  S4: 6,
},
```

### 3. 加密支持

**机制**：SQLite Codec 集成

**证据**：`BUILD.gn:52`

```gn
defines = [
  "SQLITE_HAS_CODEC",
  // ...
]
```

## 潜在风险与修复建议

### 风险 1：输入验证不完整

| 属性 | 值 |
|-----|---|
| 严重程度 | 中 |
| 可利用性 | 高 |
| 影响范围 | 所有使用 N-API 的应用 |

**证据**：`js_single_kv_store.cpp` 中的参数校验

```cpp
// js_single_kv_store.cpp:175-190
napi_value JsSingleKVStore::Put(napi_env env, napi_callback_info info)
{
    // 存在 ASSERT_BUSINESS_ERR 检查，但不完整
}
```

**触发条件**：
1. 构造超长字符串作为 key 或 value
2. 构造特殊字符序列

**影响**：
- 内存溢出
- 拒绝服务

**修复建议**：
```cpp
// 添加长度检查
if (key.size() > MAX_KEY_LENGTH) {
    return STATUS_INVALID_ARGUMENT;
}
if (value.size() > MAX_VALUE_LENGTH) {
    return STATUS_INVALID_ARGUMENT;
}
```

### 风险 2：路径遍历

| 属性 | 值 |
|-----|---|
| 严重程度 | 高 |
| 可利用性 | 中 |
| 影响范围 | 文件系统访问 |

**证据**：Backup/Restore 操作中的文件路径处理

```cpp
// js_single_kv_store.cpp:555-577
napi_value JsSingleKVStore::Backup(napi_env env, napi_callback_info info)
{
    // 需验证路径参数的安全性
}
```

**触发条件**：
1. 使用 "../" 等路径遍历字符
2. 使用绝对路径

**影响**：
- 读取任意文件
- 写入任意位置

**修复建议**：
```cpp
// 验证路径规范性
std::string normalizedPath = NormalizePath(filePath);
if (!IsPathInAllowedDir(normalizedPath)) {
    return STATUS_PERMISSION_DENIED;
}
```

### 风险 3：内存安全

| 属性 | 值 |
|-----|---|
| 严重程度 | 高 |
| 可利用性 | 低 |
| 影响范围 | 进程稳定性 |

**证据**：`js_util.cpp` 中的数组操作

```cpp
// js_util.cpp:410-424
JSUtil::StatusMsg JSUtil::GetValue(napi_env env, napi_value in, std::vector<uint8_t>& out)
{
    // 检查数组边界和类型
    ASSERT(type == napi_uint8_array, "is not Uint8Array!", napi_invalid_arg);
    ASSERT((length > 0) && (data != nullptr), "invalid data!", napi_invalid_arg);
}
```

**触发条件**：
1. 传递空 TypedArray
2. 传递超大 TypedArray

**影响**：
- 内存访问越界
- 进程崩溃

**修复建议**：
```cpp
// 限制最大分配大小
constexpr size_t MAX_BUFFER_SIZE = 64 * 1024 * 1024; // 64MB
if (length > MAX_BUFFER_SIZE) {
    return STATUS_INVALID_ARGUMENT;
}
```

### 风险 4：竞态条件

| 属性 | 值 |
|-----|---|
| 严重程度 | 中 |
| 可利用性 | 低 |
| 影响范围 | 数据一致性 |

**证据**：事务处理

```cpp
// js_single_kv_store.cpp:417-448
napi_value JsSingleKVStore::StartTransaction(napi_env env, napi_callback_info info)
// ...
napi_value JsSingleKVStore::Commit(napi_env env, napi_callback_info info)
```

**触发条件**：
1. 并发事务操作
2. 事务嵌套

**影响**：
- 数据不一致
- 死锁

**修复建议**：
```cpp
// 添加事务状态检查
if (inTransaction_) {
    return STATUS_ALREADY_IN_TRANSACTION;
}
```

### 风险 5：信息泄露

| 属性 | 值 |
|-----|---|
| 严重程度 | 中 |
| 可利用性 | 低 |
| 影响范围 | 敏感数据 |

**证据**：`js_util.cpp` 中的字符串处理

```cpp
// js_util.cpp:130-144
status = napi_get_value_string_utf8(env, in, buf, maxLen + STR_TAIL_LENGTH, &len);
// 注意：缓冲区末尾未做零终止保证
```

**触发条件**：
1. 构造特殊字符串
2. 跨边界读取

**影响**：
- 内存信息泄露
- 栈溢出

**修复建议**：
```cpp
// 确保零终止
buf[maxLen] = '\0';
```

## 已有的安全最佳实践

### 1. FORTIFY_SOURCE

**证据**：`BUILD.gn:118`

```gn
cflags_cc = [
  "-D_FORTIFY_SOURCE=2",
  "-flto",
]
```

### 2. 编译器防护

**证据**：`BUILD.gn:100-106`

```gn
sanitize = {
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
}
```

## 检查范围说明

### 已检查

- N-API 层参数校验（`frameworks/jskitsimpl/`）
- 文件路径处理（Backup/Restore）
- 类型转换安全（`js_util.cpp`）
- 内存分配边界检查
- 编译器安全选项

### 未检查

- IPC 层通信安全（依赖 dmsfwk）
- SQLite 内部实现
- OpenSSL 加密实现
- 系统调用权限校验
- 第三方依赖安全

## 相关文档

- [攻击面分析](07_AttackSurface.md) - 系统攻击面分析
- [代码地图](03_CodeMap.md) - 安全相关代码定位
- [架构设计](02_Architecture.md)
- [故障排查](09_Troubleshooting.md)
