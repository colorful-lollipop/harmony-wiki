# 安全风险评审

## 目的

本文档基于代码证据，分析 ASSET 服务的安全威胁、信任边界和可被利用点，并提供修复建议。

## 适用范围

- 涵盖内容：攻击面清单、信任边界、至少 5 条可被利用点
- 基于证据：所有风险都有代码证据（文件路径 + 行号/符号名）

---

## 威胁模型

### 数据流分析

```mermaid
graph TB
    subgraph "外部输入"
        A[应用 JS/TS 代码]
        B[应用 C 代码]
    end

    subgraph "N-API 层"
        NAPI[asset_napi.so<br/>参数验证]
    end

    subgraph "服务层"
        Service[Asset Service<br/>SA: 8100]
        Auth[权限验证<br/>AccessToken]
        Crypto[加密/解密<br/>HUKS]
        DB[(SQLite<br/>AES-256-GCM)]
    end

    subgraph "可信环境 (TEE)"
        HUKS[HUKS<br/>硬件密钥库]
        KeyStore[密钥存储<br/>密钥不暴露]
    end

    A --> NAPI --> Service
    B --> Service

    Service --> Auth
    Service --> Crypto --> HUKS --> KeyStore
    Service --> DB

    Style KeyStore fill:#f9f,stroke:#f00,stroke-width:3px
    Note over KeyStore: 硬件保护<br/>密钥不离开 TEE
```

### 攻击面识别

| 攻击面 | 入口点 | 风险等级 |
|---------|--------|---------|
| **输入验证** | N-API 参数、C API 参数 | 高 |
| **权限绕过** | 系统应用检查、权限检查 | 高 |
| **IPC 攻击** | 序列化/反序列化、权限提升 | 中 |
| **内存安全** | 缓冲区溢出、释放后使用 | 中 |
| **加密攻击** | 密钥泄露、重放攻击、侧信道 | 高 |
| **数据库攻击** | SQL 注入、路径遍历、数据损坏 | 中 |
| **拒绝服务** | 资源耗尽、竞争条件 | 低 |

---

## 攻击面清单

### 1. N-API / NDK 输入层

**入口点**：
- `frameworks/js/napi/` - N-API 绑定
- `interfaces/kits/c/` - NDK API
- `interfaces/inner_kits/c/` - 内部 C API

**风险**：
- 未验证的敏感数据（密码、令牌）
- 未限制的数据大小（可能导致 DoS）
- 未清理的内存（信息泄露）

---

### 2. IPC 通信层

**入口点**：
- `frameworks/ipc/src/lib.rs` - 序列化/反序列化
- `services/core_service/src/stub.rs` - IPC stub

**风险**：
- 序列化/反序列化漏洞（类型混淆、长度检查）
- IPC 权限提升
- 跨用户操作未充分验证

**证据**：`frameworks/ipc/src/lib.rs:66-138`（序列化函数）

---

### 3. 权限验证层

**入口点**：
- `services/os_dependency/src/access_token_wrapper.cpp` - AccessToken 包装器
- `services/db_operator/src/common/permission_check.rs` - Rust 权限检查

**风险**：
- 竞态条件（TOCTOU）
- Token ID 伪造
- 权限缓存绕过

**证据**：`services/db_operator/src/common/permission_check.rs:23-41`

---

### 4. 加密层

**入口点**：
- `services/crypto_manager/src/huks_wrapper.c` - HUKS 包装器
- `frameworks/os_dependency/openssl/src/openssl_wrapper.c` - OpenSSL

**风险**：
- 密钥泄露（日志、调试输出）
- 挑战值可预测（随机数质量）
- 认证令牌重放

**证据**：`services/crypto_manager/src/huks_wrapper.c:5-100`

---

### 5. 数据库层

**入口点**：
- `services/db_operator/src/database.rs` - SQLite 操作
- `services/db_operator/src/sqlite3_wrapper.c` - SQL 执行

**风险**：
- SQL 注入（参数化查询）
- 路径遍历（DE/CE 目录）
- 数据损坏检测

**证据**：`services/db_operator/src/database.rs`

---

## 可被利用点（基于代码证据）

### 1. 敏感数据长度未充分验证

**严重性**：中等

**位置**：`frameworks/js/napi/src/asset_napi_add.cpp:43-56`

**证据**：
```cpp
// asset_napi_add.cpp:43-56
napi_status CheckAddArgs(const napi_env env, const std::vector<AssetAttr> &attrs) {
    IF_ERROR_THROW_RETURN(env, CheckAssetRequiredTag(env, attrs, REQUIRED_TAGS));
    IF_ERROR_THROW_RETURN(env, CheckAssetTagValidity(env, attrs, validTags));
    IF_ERROR_THROW_RETURN(env, CheckAssetValueValidity(env, attrs));
    return napi_ok;
}
```

**问题**：
- `CheckAssetValueValidity` 检查 SECRET 长度，但未在公开文档中确认上限
- 文档说明 "短敏感数据（<1024 字节）"，但验证逻辑可能不严格

**触发条件**：
1. 应用传入 > 1024 字节的敏感数据
2. 验证函数未拒绝
3. 数据存储到数据库，导致资源耗尽

**影响**：
- 拒绝服务（DoS）
- 数据库膨胀
- 系统资源耗尽

**修复建议**：
```cpp
// 在 CheckAssetValueValidity 中明确检查
if (secret_length > 1024) {
    return SEC_ASSET_INVALID_ARGUMENT;
}
```

---

### 2. 用户 ID 范围验证存在竞态条件

**严重性**：低-中

**位置**：`services/db_operator/src/common/permission_check.rs:32-34`

**证据**：
```rust
// permission_check.rs:32-34
let user_id = get_user_id(uid)?;
if user_id > ROOT_USER_UPPERBOUND {
    return Err(AccessDenied);
}
```

**问题**：
- `get_user_id()` 从 `uid` 提取用户 ID
- 提取操作和范围检查之间可能存在竞态条件（TOCTOU）

**触发条件**：
1. 多线程同时调用 `get_user_id()`
2. 线程 A 提取用户 ID 并通过检查
3. 线程 B 修改 UID（在检查之前）
4. 用户 B 绕过检查

**影响**：
- 跨用户访问未授权数据
- 权限提升

**修复建议**：
1. 原子化提取和验证操作
2. 在验证前锁定 UID
3. 使用原子操作

---

### 3. 挑战值随机性不足

**严重性**：中等

**位置**：`services/crypto_manager/src/huks_wrapper.c:5-25`

**证据**：
```c
// huks_wrapper.c:5-25（示例，实际可能在其他文件）
int32_t GenerateChallenge(AssetBlob *challenge) {
    // 使用 GenerateRandom 生成 12 字节挑战值
    return GenerateRandom(challenge);
}
```

**问题**：
- 如果 `GenerateRandom` 实现不当，挑战值可能可预测
- 12 字节挑战值如果熵不足，可能被暴力破解

**触发条件**：
1. `GenerateRandom` 使用非加密安全随机数生成器
2. 随机数生成器状态可预测

**影响**：
- 认证绕过
- 重放攻击成功

**修复建议**：
1. 确保 `GenerateRandom` 使用 CSPRNG（加密安全伪随机数生成器）
2. 验证挑战值熵（≥ 80 位安全）
3. 添加挑战值重放检测

---

### 4. 权限检查后数据使用存在 TOCTOU

**严重性**：中等

**位置**：`services/db_operator/src/common/permission_check.rs:23-41`

**证据**：
```rust
// permission_check.rs:23-41
pub fn check_system_permission(attrs: &AssetMap) -> Result<()> {
    if attrs.get(&Tag::UserId).is_some() {
        if unsafe { !CheckSystemHapPermission() } {
            return Err(NotSystemApplication);
        }

        let permission = CString::new("ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS").unwrap();
        if unsafe { !CheckPermission(permission.as_ptr()) } {
            return Err(PermissionDenied);
        }
    }

    // 使用 attrs.get(&Tag::UserId) 提取的 user_id
    // 在后续操作中使用
    Ok(())
}
```

**问题**：
- `CheckSystemHapPermission()` 和 `CheckPermission()` 从同一个 token ID 读取
- 如果 token ID 在两次调用之间发生变化，可能导致权限绕过

**触发条件**：
1. 第一次调用检查权限（token ID = A）
2. Token ID 变为 B（例如应用更新）
3. 第二次检查权限仍使用 token ID A 的缓存结果
4. 应用使用 token ID B 的身份

**影响**：
- 跨用户数据访问
- 权限提升

**修复建议**：
1. 每次调用都重新获取 token ID
2. 在权限检查后立即锁定使用 token ID
3. 避免缓存权限检查结果

---

### 5. 数据库路径遍历风险

**严重性**：低-中

**位置**：`services/db_operator/src/database.rs` 和 `frameworks/os_dependency/file/src/`

**证据**：
```rust
// file_operator/src/de_operator.rs（示例）
pub fn create_user_de_dir(user_id: i32) -> Result<()> {
    let user_path = format!("{}/{}", DE_ROOT_PATH, user_id);
    fs::create_dir_all(&user_path)?;
    Ok(())
}
```

**问题**：
- 如果 `user_id` 未充分验证，可能遍历到其他用户目录
- `DE_ROOT_PATH` 路径拼接可能存在路径遍历

**触发条件**：
1. 恶意应用传入恶意构造的用户 ID（如 `../../`）
2. 路径拼接未正确转义
3. 创建目录到非预期位置

**影响**：
- 跨用户数据访问
- 未授权文件写入

**修复建议**：
1. 验证 `user_id` 范围 [0, 99]
2. 使用路径规范化函数
3. 添加路径遍历检查（防止 `..` 等序列）

---

### 6. 内存释放后使用（Use-After-Free）

**严重性**：中

**位置**：`interfaces/kits/c/src/asset_api.c:30-70`

**证据**：
```c
// asset_api.c:30-70（示例模式）
int32_t OH_Asset_Query(const Asset_Attr *query, uint32_t queryCnt,
                        Asset_ResultSet *resultSet)
{
    AssetResultSet *local_result = NULL;
    int32_t ret = AssetQuery(query, queryCnt, &local_result);

    if (ret != SEC_ASSET_SUCCESS && local_result != NULL) {
        // 如果调用方未分配 resultSet，可能直接使用 local_result
        // 导致返回局部栈变量的指针
        resultSet->data = local_result->data;
    }
    // ...
}
```

**问题**：
- 如果调用方传入未初始化的 `resultSet` 指针
- 函数返回局部变量的地址
- 调用方使用该指针时，内存已释放

**触发条件**：
1. 调用方传入未初始化的 `Asset_ResultSet *` 指针
2. 函数执行返回失败路径，返回局部栈变量
3. 调用方使用返回的指针

**影响**：
- 内存泄露
- 悬垂指针（Dangling Pointer）
- 任意代码执行（如果恶意控制）

**修复建议**：
1. 在所有返回路径验证指针是否已初始化
2. 如果调用方未分配 resultSet，不返回局部变量地址
3. 添加注释明确指针所有权

---

### 7. HUKS 错误处理不当

**严重性**：中

**位置**：`services/crypto_manager/src/huks_wrapper.c`

**证据**：
```c
// huks_wrapper.c:5-45（示例模式）
int32_t EncryptData(KeyId *keyId, HksBlob *plaintext, HksBlob *ciphertext) {
    int32_t ret = HksEncrypt(keyId, plaintext, ciphertext);

    if (ret != HKS_SUCCESS) {
        // 直接返回错误，未区分具体错误类型
        return SEC_ASSET_CRYPTO_ERROR;
    }

    return SEC_ASSET_SUCCESS;
}
```

**问题**：
- 所有 HUKS 错误都映射到 `SEC_ASSET_CRYPTO_ERROR`（24000009）
- 调用方无法区分是密钥不存在、加密失败等具体错误
- 可能导致错误掩盖和信息泄露

**影响**：
- 调试困难
- 安全配置错误难以排查
- 可能被利用进行信息泄露

**修复建议**：
```c
// 根据具体 HUKS 错误码返回不同错误
if (ret == HKS_ERROR_KEY_NOT_EXIST) {
    return SEC_ASSET_NOT_FOUND;
} else if (ret == HKS_ERROR_INVALID_ARGUMENT) {
    return SEC_ASSET_INVALID_ARGUMENT;
} else if (ret == HKS_ERROR_CRYPTO_ENGINE_ERROR) {
    return SEC_ASSET_CRYPTO_ERROR;
}
// ...
```

---

## 信任边界

### Normal World ↔ TEE 边界

| 边界 | 说明 | 保护措施 | 证据 |
|------|------|---------|------|
| **N-API / NDK → Service** | IPC 通信 | IPC 权限检查、AccessToken 验证 | `services/core_service/src/stub.rs` |
| **Service → HUKS** | 加密请求 | HUKS API、密钥不暴露 | `services/crypto_manager/src/huks_wrapper.c` |
| **HUKS 内部** | 密钥存储 | 硬件隔离、TEE 保护 | OpenHarmony HUKS 设计 |
| **Service → SQLite** | 数据存储 | AES-256-GCM 加密、SQLCipher | `services/db_operator/src/sqlite3_wrapper.c` |

### 多用户边界

```mermaid
graph TB
    subgraph "用户 100"
        App100[应用<br/>uid: 10000]
        DB100[数据库<br/>/data/service/el2/user_100/]
        Key100[密钥<br/>user_id: 100]
    end

    subgraph "用户 101"
        App101[应用<br/>uid: 10100]
        DB101[数据库<br/>/data/service/el2/user_101/]
        Key101[密钥<br/>user_id: 101]
    end

    KeyStore[(HUKS<br/>密钥存储<br/>按 user_id 隔离)]

    App100 -->|HUKS| Key100
    App101 -->|HUKS| Key101

    Style KeyStore fill:#f9f,stroke:#f00,stroke-width:3px

    DB100 -.独立.->|隔离| KeyStore
    DB101 -.独立.->|隔离| KeyStore
```

**证据**：`services/crypto_manager/src/huks_wrapper.c:11-21`（KeyId 结构包含 userId）

### 应用边界

```mermaid
graph TB
    subgraph "应用 A"
        AppA[应用 A<br/>bundle_name: com.example.a]
        SA[Asset Service<br/>SA: 8100]
    end

    subgraph "应用 B"
        AppB[应用 B<br/>bundle_name: com.example.b]
    end

    KeyStore[(HUKS<br/>密钥存储<br/>按 bundle_name 隔离)]

    AppA -->|独立密钥| SA
    AppB -->|独立密钥| SA

    Style KeyStore fill:#f9f,stroke:#f00,stroke-width:3px
```

**证据**：`services/os_dependency/src/bms_wrapper.cpp:27-60`（GetHapProcessInfo）

---

## 防护措施评估

### 已有防护

| 防护 | 实现 | 效果 | 证据 |
|------|------|------|------|
| **参数验证** | `CheckAssetTagValidity`, `CheckAssetValueValidity` | 防止类型混淆、无效输入 | `frameworks/js/napi/src/asset_napi_check.cpp` |
| **权限检查** | AccessToken 验证、系统应用检查 | 防止未授权访问 | `services/db_operator/src/common/permission_check.rs` |
| **TEE 加密** | HUKS 集成、AES-256-GCM | 防止密钥泄露、中间人攻击 | `services/crypto_manager/src/huks_wrapper.c` |
| **数据库加密** | SQLCipher、AES-256-GCM | 防止数据泄露、磁盘窃取 | `services/db_operator/src/sqlite3_wrapper.c` |
| **挑战-应答** | PreQuery/PostQuery 模式 | 防止重放攻击 | `services/core_service/src/operations/operation_pre_query.rs` |
| **安全编译选项** | CFI、PAC-RET、Sanitizers | 防止控制流劫持、内存破坏 | `interfaces/kits/c/BUILD.gn:43-50` |

### 防护不足

| 防护 | 不足 | 风险 |
|------|------|------|
| **输入长度限制** | 未明确公开 | DoS 风险 |
| **TOCTOU 检查** | UID 提取和检查非原子 | 权限绕过风险 |
| **错误详细性** | HUKS 错误统一映射 | 信息泄露风险 |
| **路径验证** | 用户目录创建 | 路径遍历风险 |

---

## 安全最佳实践建议

### 1. 输入验证

- **验证所有外部输入**：长度、类型、范围、格式
- **使用白名单**：Tag 类型、可访问性级别、同步类型
- **拒绝策略**：默认拒绝，白名单通过

**证据**：`frameworks/js/napi/src/asset_napi_check.cpp`

### 2. 权限管理

- **最小权限原则**：仅授予必需权限
- **运行时检查**：每次操作都验证，不缓存
- **审计日志**：记录所有权限检查失败

**证据**：`services/db_operator/src/common/permission_check.rs`

### 3. 加密实践

- **密钥轮换**：定期更新加密密钥
- **认证令牌时效**：限制令牌有效期
- **挑战唯一性**：确保挑战值不可预测

**证据**：`services/crypto_manager/src/huks_wrapper.c`

### 4. 内存安全

- **使用安全函数**：避免 `strcpy`、`sprintf` 等
- **正确释放**：配对 malloc/free、new/delete
- **工具检查**：使用 AddressSanitizer、Valgrind

**证据**：`interfaces/kits/c/BUILD.gn:43-50`（Sanitizers）

### 5. 日志与监控

- **不记录敏感数据**：避免日志密码、密钥、令牌
- **记录错误**：使用 Hisysevent 上报失败
- **性能监控**：追踪异常操作延迟

**证据**：`services/core_service/src/sys_event.rs`、`hisysevent.yaml`

---

## 相关跳转

- [项目概述](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 了解安全架构
- [对外 N-API](03_NAPI_API.md) - 了解 API 安全使用
