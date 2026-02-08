# 调用链示例

## 目的

本文档提供 ASSET 服务关键 API 的调用链示例，帮助理解从入口到核心逻辑的完整数据流。

## 适用范围

- 涵盖内容：关键调用链（入口→核心逻辑）
- 格式：文字说明 + Mermaid 图表

---

## 1. 添加资产调用链

### 完整调用链示意图

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API<br/>asset_napi_add.cpp
    participant SysAPI as C System API<br/>asset_system_api.c
    participant RustSDK as Rust SDK<br/>lib.rs
    participant Service as Asset Service<br/>lib.rs
    participant Ops as Operations<br/>operation_add.rs
    participant Check as Permission Check<br/>permission_check.rs
    participant BMS as Bundle Manager<br/>bms_wrapper.cpp
    participant HUKS as HUKS<br/>huks_wrapper.c
    participant DB as Database<br/>database.rs

    App->>NAPI: 1. add(attributes)
    Note over NAPI: Map 包含<br/>Tag.SECRET, Tag.ALIAS

    NAPI->>NAPI: 2. CheckAddArgs(attrs)
    NAPI->>NAPI: 3. CheckAssetRequiredTag()
    Note over NAPI: 验证必填 Tag<br/>SECRET, ALIAS

    NAPI->>NAPI: 4. CheckAssetTagValidity()
    Note over NAPI: 验证 Tag 类型<br/>和有效性

    NAPI->>NAPI: 5. CheckAssetValueValidity()
    Note over NAPI: 验证 SECRET 长度<br/>(< 1024 bytes)

    NAPI->>NAPI: 6. CreateAsyncWork(context)
    Note over NAPI: 创建异步任务<br/>在线程池执行

    NAPI->>SysAPI: 7. OH_Asset_Add(attrs)
    Note over SysAPI: NDK 层<br/>调用 C API

    SysAPI->>RustSDK: 8. AssetAdd(attrs)
    Note over RustSDK: C FFI 层<br/>转换参数类型

    RustSDK->>Service: 9. IPC 请求 (AddAsset)
    Note over RustSDK: 通过 SA Proxy<br/>序列化参数

    Service->>Service: 10. on_remote_request(Add)
    Note over Service: IPC Stub 分发<br/>到 operation_add

    Service->>Ops: 11. add(context, attrs)
    Note over Ops: 业务逻辑入口

    Ops->>Check: 12. check_system_permission(attrs)
    Note over Check: 权限检查<br/>(跨用户需要权限)

    Check->>BMS: 13. GetCallingProcessInfo()
    Note over BMS: 获取调用者信息<br/>(bundle_name, app_index)

    BMS-->>Check: 14. 返回 ProcessInfo

    Ops->>Ops: 15. validate_and_normalize(attrs)
    Note over Ops: 验证参数<br/>和归一化

    Ops->>HUKS: 16. generate_key_for_user()
    Note over HUKS: 生成应用密钥<br/>(AES-256)

    HUKS->>HUKS: 17. GenerateKey(keyId)
    Note over HUKS: 在 TEE 中生成<br/>硬件保护

    HUKS-->>Ops: 18. 返回密钥 ID

    Ops->>HUKS: 19. encrypt_data(keyId, secret)
    Note over HUKS: 使用应用密钥<br/>加密敏感数据

    HUKS->>HUKS: 20. EncryptData()
    Note over HUKS: AES-256-GCM<br/>在 TEE 中加密

    HUKS-->>Ops: 21. 返回密文

    Ops->>DB: 22. insert(data_info)
    Note over DB: 插入加密数据<br/>到 SQLite

    DB-->>Ops: 23. 返回成功

    Ops-->>Service: 24. 返回 SEC_ASSET_SUCCESS

    Service-->>RustSDK: 25. 返回结果
    RustSDK-->>SysAPI: 26. 返回 AssetResultSet

    SysAPI-->>NAPI: 27. 返回 Asset_ResultSet

    NAPI-->>App: 28. Promise resolve(void)
    Note over NAPI: 操作成功
```

### 关键节点说明

| 节点 | 文件 | 函数 | 职责 |
|------|------|------|------|
| **参数验证** | asset_napi_add.cpp | CheckAddArgs | 必填/可选/类型检查 |
| **权限检查** | permission_check.rs | check_system_permission | INTERACT_ACROSS_LOCAL_ACCOUNTS 权限 |
| **应用信息** | bms_wrapper.cpp | GetCallingProcessInfo | Bundle Name, App Index |
| **密钥生成** | huks_wrapper.c | GenerateKey | 应用唯一 AES-256 密钥 |
| **加密** | huks_wrapper.c | EncryptData | AES-256-GCM 加密 |
| **存储** | database.rs | insert | SQLite 事务插入 |

---

## 2. 查询资产调用链（带认证）

### 完整调用链示意图

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API<br/>asset_napi_pre_query.cpp
    participant Service as Asset Service
    participant UserIAM as UserIAM<br/>外部认证
    participant HUKS as HUKS
    participant DB as Database

    App->>NAPI: 1. preQuery(query)
    Note over NAPI: 传入查询条件

    NAPI->>Service: 2. IPC 请求 (PreQuery)
    Note over NAPI: 通过 Rust SDK

    Service->>Service: 3. pre_query(context, query)
    Note over Service: operation_pre_query.rs

    Service->>HUKS: 4. generate_challenge()
    Note over HUKS: 生成 12 字节<br/>随机挑战值

    HUKS->>HUKS: 5. GenerateRandom(challenge)
    Note over HUKS: CSPRNG<br/>加密安全随机数

    HUKS-->>Service: 6. 返回 challenge

    Service-->>NAPI: 7. 返回 challenge
    NAPI-->>App: 8. Promise resolve(challenge)

    Note over App: 用户完成认证<br/>(PIN/指纹/人脸)

    App->>UserIAM: 9. authenticate(challenge)
    Note over UserIAM: 拉起认证界面

    UserIAM->>UserIAM: 10. 显示认证 UI
    Note over UserIAM: UserIAM 框架<br/>处理认证流程

    UserIAM-->>App: 11. 返回 auth_token
    Note over App: 包含认证结果<br/>和签名

    App->>NAPI: 12. postQuery(handle, auth_token)
    Note over NAPI: handle 包含<br/>challenge 和 auth_token

    NAPI->>Service: 13. IPC 请求 (PostQuery)

    Service->>Service: 14. post_query(context, handle, auth_token)
    Note over Service: operation_post_query.rs

    Service->>HUKS: 15. verify_auth_token(auth_token)
    Note over HUKS: 验证令牌<br/>在 TEE 中

    HUKS->>HUKS: 16. ExecCrypt(keyId, auth_token)
    Note over HUKS: 验证认证<br/>防重放攻击

    HUKS-->>Service: 17. 验证通过/失败

    Service->>DB: 18. query(query)
    Note over DB: 查询加密数据

    Service->>HUKS: 19. decrypt_data(keyId, ciphertext)
    Note over HUKS: 使用应用密钥<br/>解密数据

    HUKS->>HUKS: 20. DecryptData()
    Note over HUKS: AES-256-GCM<br/>在 TEE 中解密

    HUKS-->>Service: 21. 返回明文

    Service-->>NAPI: 22. 返回 AssetResultSet
    NAPI-->>App: 23. Promise resolve(assets)
```

### 关键节点说明

| 节点 | 文件 | 函数 | 职责 |
|------|------|------|------|
| **挑战生成** | huks_wrapper.c | GenerateRandom | CSPRNG 12 字节挑战值 |
| **认证验证** | huks_wrapper.c | ExecCrypt | 在 TEE 中验证令牌 |
| **数据查询** | database.rs | query | 根据 alias 查询 |
| **数据解密** | huks_wrapper.c | DecryptData | AES-256-GCM 解密 |

---

## 3. 删除资产调用链（应用卸载）

### 完整调用链示意图

```mermaid
sequenceDiagram
    participant BMS as Bundle Manager
    participant EventSys as Common Event<br/>service
    participant Service as Asset Service
    participant DB as Database
    participant HUKS as HUKS
    participant Log as HiSysEvent

    BMS->>BMS: 1. 应用卸载
    Note over BMS: 应用从系统卸载

    BMS->>EventSys: 2. 发送 PACKAGE_REMOVED
    Note over EventSys: 广播卸载事件

    EventSys->>Service: 3. 事件回调
    Note over EventSys: 通知订阅者

    Service->>Service: 4. on_package_removed()
    Note over Service: common_event/start_event.rs

    Service->>DB: 5. query_all_by_bundle(bundle_name)
    Note over DB: 根据包名<br/>查询所有资产

    DB-->>Service: 6. 返回资产列表

    Service->>DB: 7. delete_batch(asset_ids)
    Note over DB: 删除所有<br/>相关资产

    DB-->>Service: 8. 返回删除计数

    loop loop for each asset in Service->>DB: 9. delete_key(keyId)
    Note over HUKS: 删除<br/>每个资产的密钥

    HUKS->>HUKS: 10. DeleteKey(keyId)
    Note over HUKS: 从 TEE<br/>删除密钥

    HUKS-->>Service: 11. 返回成功/失败

    Service->>Log: 12. upload_system_event()
    Note over Log: 上报统计<br/>或故障

    Log-->>Service: 13. 返回成功
```

### 关键节点说明

| 节点 | 文件 | 函数 | 职责 |
|------|------|------|------|
| **事件监听** | core_service | on_package_removed | 处理应用卸载事件 |
| **批量查询** | database.rs | query_all_by_bundle | 查询应用所有资产 |
| **批量删除** | database.rs | delete_batch | 事务删除多个记录 |
| **密钥清理** | huks_wrapper.c | DeleteKey | 从 HUKS 删除密钥 |
| **事件上报** | sys_event.rs | upload_system_event | 上报 Hisysevent |

---

## 4. 跨用户查询调用链（系统应用）

### 完整调用链示意图

```mermaid
sequenceDiagram
    participant App as 系统应用<br/>(userId: 101)
    participant Service as Asset Service
    participant Check as Permission<br/>Check
    participant DB as Database

    App->>Service: 1. queryAsUser(101, query)
    Note over App: 指定目标用户 ID

    Service->>Service: 2. query_as_user(context, userId, query)
    Note over Service: operation_query.rs<br/>带 userId 参数

    Service->>Check: 3. check_system_permission_with_uid(attrs, userId)
    Note over Check: 跨用户检查<br/>UserId + 权限

    Check->>Check: 4. CheckSystemHapPermission()
    Note over Check: 验证调用者<br/>是否为系统应用

    Check->>Check: 5. CheckPermission(INTERACT_ACROSS_LOCAL_ACCOUNTS)
    Note over Check: 检查<br/>跨用户权限

    Check-->>Service: 6. 权限通过

    Service->>DB: 7. query_user_101(query)
    Note over DB: 查询用户 101<br/>的数据

    DB-->>Service: 8. 返回资产

    Service->>Service: 9. decrypt_assets(assets)
    Note over Service: 使用用户 101 的密钥<br/>解密数据

    Service-->>App: 10. 返回用户 101 的资产
```

### 关键节点说明

| 节点 | 文件 | 函数 | 职责 |
|------|------|------|------|
| **系统应用检查** | access_token_wrapper.cpp | CheckSystemHapPermission | 验证调用者身份 |
| **跨用户权限** | access_token_wrapper.cpp | CheckPermission | 验证 INTERACT_ACROSS_LOCAL_ACCOUNTS |
| **用户查询** | database.rs | query_user_* | 指定用户 ID 查询 |

---

## 调用链总结

### 关键数据流模式

1. **应用 → N-API → C System API → Rust SDK → Asset Service**
   - N-API 层：参数验证、异步任务管理
   - C API 层：参数转换、NDK 接口
   - Rust SDK 层：IPC 代理、序列化
   - Service 层：权限检查、业务逻辑、加密、存储

2. **Asset Service 内部流程**
   - 权限验证 → 业务逻辑 → 加密/解密 → 数据库操作

3. **依赖外部服务**
   - HUKS：密钥生成、加密/解密、认证验证
   - UserIAM：用户身份认证（preQuery/postQuery）
   - BMS：应用信息获取
   - AccessToken：权限验证
   - SQLite：数据持久化

### 线程模型

| 层次 | 线程模型 |
|------|---------|
| **应用层** | JS 主线程（N-API 回调） |
| **N-API 层** | JS 主线程 + 工作线程池（异步任务） |
| **服务层** | 主线程（SA 生命周期）+ 异步运行时 |
| **HUKS 层** | TEE 线程（硬件隔离） |
| **数据库层** | SQLite 事务线程 |

---

## 相关跳转

- [项目概述](00_Overview.md) - 了解项目定位
- [对外 N-API](03_NAPI_API.md) - 了解 JS API 详情
- [架构说明](02_Architecture.md) - 了解整体架构
- [内部 API](04_Inner_API.md) - 了解系统 API
