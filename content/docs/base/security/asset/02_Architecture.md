# 架构说明

## 目的

本文档描述 ASSET 服务的整体架构，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- 涵盖内容：组件关系、数据流向、线程模型、关键时序
- 包含：Mermaid 图表

---

## 整体架构

### 分层架构

```mermaid
graph TB
    subgraph "应用层"
        AppJS[应用 JS/TS 代码]
        AppC[应用 C 代码]
    end

    subgraph "N-API 层"
        NAPI[asset_napi.so<br/>JS→Native 桥接]
    end

    subgraph "NDK 层"
        NDK[asset_ndk.so<br/>公共 C API]
        SysAPI[asset_sdk.so<br/>内部 C API]
    end

    subgraph "Rust SDK 层"
        RustSDK[asset_sdk.so<br/>Rust SDK 客户端]
    end

    subgraph "IPC 层"
        Proxy[IPC Proxy<br/>SystemAbilityManager]
        Stub[IPC Stub<br/>RemoteStub]
    end

    subgraph "服务层"
        Core[Asset Service<br/>SA ID: 8100]
        Ops[Operations<br/>CRUD 逻辑]
    end

    subgraph "数据层"
        Crypto[Crypto Manager<br/>HUKS 集成]
        DB[DB Operator<br/>SQLite + AES-256-GCM]
    end

    subgraph "外部依赖"
        HUKS[HUKS<br/>硬件密钥库]
        UserIAM[UserIAM<br/>统一用户认证]
        AccessToken[AccessToken<br/>权限管理]
        BMS[Bundle Manager<br/>应用信息]
        SQLite[SQLite3]
    end

    AppJS --> NAPI
    AppC --> NDK

    NAPI --> SysAPI
    NDK --> SysAPI

    SysAPI --> RustSDK
    RustSDK --> Proxy

    Proxy -->|加载 SA<br/>SA_ID=8100| Core
    Proxy -->|IPC 请求| Stub

    Core --> Ops
    Ops --> Crypto
    Ops --> DB

    Crypto --> HUKS
    Ops --> UserIAM
    Ops --> AccessToken
    Ops --> BMS

    DB --> SQLite
```

**证据**：
- N-API 层：`frameworks/js/napi/BUILD.gn:16`（asset_napi）
- NDK 层：`interfaces/kits/c/BUILD.gn:20-23`（asset_ndk）
- Rust SDK：`interfaces/inner_kits/rs/BUILD.gn:30-34`（asset_sdk_rust）
- IPC 层：`frameworks/ipc/src/lib.rs:25-87`（SA_ID=8100）
- 服务层：`services/core_service/src/lib.rs:16-100`（AssetAbility）
- Crypto 层：`services/crypto_manager/src/huks_wrapper.c:1-100`（HUKS）

### 层次说明

| 层次 | 组件 | 职责 |
|------|------|------|
| **应用层** | JS/TS 应用、C 应用 | 使用 ASSET API 存储敏感数据 |
| **N-API 层** | asset_napi.so | JS 到 Native 的桥接层 |
| **NDK 层** | asset_ndk.so, asset_sdk.so | 公共/内部 C API |
| **Rust SDK 层** | asset_sdk.so | Rust 客户端 SDK |
| **IPC 层** | IPC Proxy/Stub | 进程间通信 |
| **服务层** | Asset Service (SA:8100) | 核心业务逻辑 |
| **数据层** | Crypto Manager, DB Operator | 加密和持久化 |
| **外部依赖** | HUKS, UserIAM, AccessToken | 系统服务依赖 |

---

## 数据流

### 1. 添加资产流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant SDK as Rust SDK
    participant Service as Asset Service
    participant HUKS as HUKS
    participant DB as Database

    App->>NAPI: 1. 调用 add(attributes)
    NAPI->>NAPI: 2. 验证参数<br/>(tag, type, length)
    NAPI->>SDK: 3. 调用 AssetAdd()
    SDK->>Service: 4. IPC 请求 (AddAsset)

    Service->>Service: 5. 权限检查<br/>AccessToken::VerifyPermission
    Service->>Service: 6. 获取调用者信息<br/>BMS::GetProcessInfo
    Service->>Service: 7. 生成应用密钥<br/>HUKS::GenerateKey
    HUKS->>HUKS: 8. 创建 AES-256 密钥<br/>(TEE 中)
    HUKS-->>Service: 9. 返回密钥 ID

    Service->>HUKS: 10. 加密敏感数据<br/>AES-256-GCM
    HUKS->>HUKS: 11. TEE 中加密<br/>(密钥不暴露)
    HUKS-->>Service: 12. 返回密文

    Service->>DB: 13. 存储加密数据<br/>(SQLite 事务)
    DB-->>Service: 14. 返回成功

    Service-->>SDK: 15. 返回 SEC_ASSET_SUCCESS
    SDK-->>NAPI: 16. 返回结果
    NAPI-->>App: 17. Promise resolve
```

**证据**：
- 步骤 7：`services/db_operator/src/common/permission_check.rs:23-41`（check_system_permission）
- 步骤 8：`services/os_dependency/src/bms_wrapper.cpp:27-60`（GetHapProcessInfo）
- 步骤 9-11：`services/crypto_manager/src/huks_wrapper.c:5-45`（GenerateKey, EncryptData）

### 2. 查询资产流程（带认证）

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API
    participant Service as Asset Service
    participant UserIAM as UserIAM
    participant HUKS as HUKS
    participant DB as Database

    App->>NAPI: 1. preQuery(query)
    NAPI->>Service: 2. IPC 请求 (PreQuery)
    Service->>HUKS: 3. 生成挑战值<br/>GenerateRandom
    HUKS-->>Service: 4. 返回 challenge
    Service-->>NAPI: 5. 返回 challenge

    NAPI-->>App: 6. Promise resolve(challenge)

    Note over App: 用户完成身份认证<br/>(PIN/指纹/人脸)

    App->>UserIAM: 7. 调用认证 API
    UserIAM->>UserIAM: 8. 拉起认证界面
    UserIAM-->>App: 9. 返回认证结果
    Note over App: 包含 auth_token

    App->>NAPI: 10. postQuery(handle, auth_token)
    NAPI->>Service: 11. IPC 请求 (PostQuery)
    Service->>HUKS: 12. 验证认证令牌<br/>VerifyAuthToken
    HUKS->>HUKS: 13. TEE 中验证<br/>(防重放攻击)
    HUKS-->>Service: 14. 验证通过

    Service->>DB: 15. 查询加密数据
    Service->>HUKS: 16. 解密数据<br/>AES-256-GCM
    HUKS-->>Service: 17. 返回明文
    Service-->>NAPI: 18. 返回资产数据
    NAPI-->>App: 19. Promise resolve(data)
```

**证据**：
- 步骤 3-4：`services/core_service/src/operations/operation_pre_query.rs`（生成 challenge）
- 步骤 12-14：`services/crypto_manager/src/huks_wrapper.c:77-100`（ExecCrypt 带认证）

### 3. 跨用户查询流程

```mermaid
sequenceDiagram
    participant App as 应用 (user_id: 101)
    participant NAPI as N-API
    participant Service as Asset Service
    participant BMS as Bundle Manager

    App->>NAPI: 1. queryAsUser(101, query)
    NAPI->>Service: 2. IPC 请求 (带 user_id)

    Service->>Service: 3. 检查系统应用权限<br/>CheckSystemHapPermission
    alt 非系统应用
        Service->>Service: 3a. 拒绝访问
        Service-->>App: 4a. 错误: NOT_SYSTEM_APPLICATION
    else 系统应用
        Service->>BMS: 3b. 检查 INTERACT_ACROSS<br/>_LOCAL_ACCOUNTS 权限
        BMS-->>Service: 4b. 权限存在

        Service->>DB: 5. 查询用户 101 数据
        Service->>HUKS: 6. 使用用户 101 的密钥解密
        Service-->>App: 7. 返回用户 101 的资产
    end
```

**证据**：
- 步骤 3a：`services/db_operator/src/common/permission_check.rs:32-34`（系统应用检查）
- 步骤 3b：`services/os_dependency/src/access_token_wrapper.cpp:14-20`（CheckPermission）

---

## 线程模型

### 服务端线程模型

**Asset Service 使用 OpenHarmony 的 Rust SA 框架，线程模型如下**：

```rust
// services/core_service/src/lib.rs
impl Ability for AssetAbility {
    fn on_start_with_reason(&self, reason: OnDemandReason) {
        // 主线程：启动事件处理
        // 异步任务：通过 ylong_runtime 执行
    }

    fn on_idle(&self) -> i32 {
        // 返回延迟卸载时间
        // 服务空闲 20 秒后卸载
        return DELAYED_UNLOAD_TIME_IN_SEC * SEC_TO_MILLISEC;
    }
}
```

**关键点**：
1. **主线程**：处理 SA 生命周期（on_start, on_stop, on_idle）
2. **异步任务**：通过 `ylong_runtime::RuntimeBuilder` 执行异步操作
3. **延迟卸载**：空闲 20 秒后自动卸载（`services/core_service/src/lib.rs:88`）

**证据**：
- 异步任务：`services/common/src/task_manager.rs`（TaskManager）
- 延迟卸载：`services/core_service/src/lib.rs:88-89`（DELAYED_UNLOAD_TIME_IN_SEC）

### 客户端线程模型

**N-API 线程**：
- **JS 主线程**：N-API 回调在 JS 主线程执行
- **工作线程**：`CreateAsyncWork` 创建的工作线程执行实际操作

**证据**：`frameworks/js/napi/src/asset_napi_common.cpp:82-85`（CreateAsyncWork）

```cpp
napi_value CreateAsyncWork(const napi_env env, napi_callback_info info,
    std::unique_ptr<BaseContext> context,
    const char *resourceName)
{
    // 创建异步工作，在线程池执行
    napi_value resource = nullptr;
    napi_create_async_work(env, context.get(), executeCallback, completeCallback,
        &resource, &result);
}
```

---

## 关键时序图

### 1. 服务启动时序

```mermaid
sequenceDiagram
    participant SAMgr as SAMgr
    participant Service as Asset Service
    participant HUKS as HUKS
    participant DB as Database
    participant EventSys as Common Event Service

    SAMgr->>Service: 1. 加载 SA<br/>(按需触发事件)
    Service->>Service: 2. on_start_with_reason()
    Service->>HUKS: 3. 初始化 HUKS 连接
    HUKS-->>Service: 4. 连接成功

    Service->>DB: 5. 打开数据库<br/>(用户空间)
    Service->>DB: 6. 检查并升级数据库
    Service->>Service: 7. 订阅系统事件<br/>(PACKAGE_REMOVED, USER_REMOVED 等)
    Service->>EventSys: 8. 注册事件监听器
    EventSys-->>Service: 9. 注册成功

    Service->>Service: 10. on_active()<br/>设置为活动状态
    Service-->>SAMgr: 11. 服务就绪
```

**证据**：
- 事件订阅：`services/core_service/src/common_event/start_event.rs`（事件处理）
- 升级检查：`services/core_service/src/upgrade_ce.rs`（数据库升级）

### 2. 应用卸载清理时序

```mermaid
sequenceDiagram
    participant BMS as Bundle Manager
    participant EventSys as Common Event Service
    participant Service as Asset Service
    participant DB as Database
    participant HUKS as HUKS

    BMS->>BMS: 1. 应用卸载
    BMS->>EventSys: 2. 发送 PACKAGE_REMOVED 事件
    EventSys->>Service: 3. 事件回调

    Service->>Service: 4. 查询应用的所有资产
    Service->>DB: 5. 根据 bundle_name 删除
    Service->>HUKS: 6. 删除应用密钥
    HUKS-->>Service: 7. 密钥删除成功
    DB-->>Service: 8. 资产删除成功
    Service->>EventSys: 9. 上报 SECRET_STORE_INFO_COLLECTION
```

**证据**：
- 事件处理：`services/core_service/src/common_event/listener.rs`（事件监听）
- 密钥删除：`services/crypto_manager/src/huks_wrapper.c:46-54`（DeleteKey）

---

## 组件交互关系

### 服务间依赖

```mermaid
graph LR
    subgraph "Asset Service"
        Core[Asset Service<br/>SA: 8100]
        Plugin[Asset Plugin<br/>扩展系统]
    end

    subgraph "依赖服务"
        HUKS[HUKS<br/>SA: 3501]
        UserIAM[UserIAM<br/>SA: 9901]
        AccessToken[AccessToken<br/>SA: 6501]
        BMS[Bundle Manager<br/>SA: 3011]
        MemoryMgr[Memory Manager<br/>SA: 1909]
        DataShare[DataShare<br/>SA: 8001]
    end

    Core -->|查询密钥| HUKS
    Core -->|加密/解密| HUKS
    Core -->|验证令牌| UserIAM

    Core -->|检查权限| AccessToken
    Core -->|获取应用信息| BMS

    Core -->|通知内存使用| MemoryMgr
    Plugin -->|数据共享| DataShare
```

**证据**：
- SA ID 8100：`frameworks/ipc/src/lib.rs:25`
- SA ID 3501（HUKS）：OpenHarmony 标准
- SA ID 9901（UserIAM）：OpenHarmony 标准
- SA ID 6501（AccessToken）：OpenHarmony 标准
- SA ID 3011（BMS）：OpenHarmony 标准
- SA ID 1909（Memory Manager）：`services/core_service/src/lib.rs:94`

---

## 数据库架构

### SQLite 加密配置

```mermaid
graph TB
    subgraph "数据库层"
        DB[(SQLite Database)]
        Cipher[SQLCipher<br/>AES-256-GCM]
        KDF[Key Derivation<br/>KDF-SHA1<br/>10000 iterations]
    end

    subgraph "密钥管理"
        DBKey[DB Key<br/>derived from<br/>user encryption key]
    end

    DBKey -->|密钥派生| KDF
    KDF -->|派生密钥| Cipher
    Cipher -->|加密| DB

    DB -.加密数据.->|存储| DB
```

**加密参数**：
- 算法：`aes-256-gcm`
- HMAC 算法：`SHA1`
- KDF 函数：`KDF_SHA1`
- 迭代次数：`10000`
- 页面大小：`1024`

**证据**：`services/db_operator/src/sqlite3_wrapper.c:5-13`

### 表结构

```sql
CREATE TABLE assets (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,              -- 多用户支持
    alias BLOB NOT NULL,                 -- 资产别名
    secret BLOB NOT NULL,                -- 加密的敏感数据
    data_label_critical_1 BLOB,           -- 不可变自定义字段
    data_label_critical_2 BLOB,
    data_label_critical_3 BLOB,
    data_label_critical_4 BLOB,
    data_label_normal_1 BLOB,            -- 可变自定义字段
    data_label_normal_2 BLOB,
    data_label_normal_3 BLOB,
    data_label_normal_4 BLOB,
    access_control INTEGER,                -- 访问控制位
    sync_type INTEGER,                    -- 同步类型
    require_attr_encrypted INTEGER,         -- 属性加密标志
    create_time INTEGER NOT NULL,           -- 创建时间
    update_time INTEGER NOT NULL,           -- 更新时间
    UNIQUE(user_id, alias)               -- 防重复
);
```

**证据**：`services/db_operator/src/table.rs`（表结构定义）

---

## 插件架构

### 插件接口

```rust
// interfaces/inner_kits/plugin_interface/src/plugin_interface.rs
pub trait IAssetPlugin {
    fn redirect_request(&self, request: &mut MsgParcel) -> Result<()>;
    fn on_sa_extension(&self, extension: &str) -> Result<()>;
}
```

### 插件加载

```mermaid
sequenceDiagram
    participant Service as Asset Service
    participant Loader as Plugin Loader
    participant Plugin as Asset Plugin

    Service->>Loader: 1. 扫描插件目录
    Loader->>Plugin: 2. 动态加载 .so
    Plugin-->>Loader: 3. 返回 IAssetPlugin
    Loader-->>Service: 4. 注册插件

    Service->>Service: 5. 分发扩展请求<br/>(backup/restore/RssSaExtension)
    Service->>Plugin: 6. 调用 redirect_request()
    Plugin->>Plugin: 7. 处理扩展逻辑
    Plugin-->>Service: 8. 返回结果
```

**证据**：
- 插件接口：`interfaces/inner_kits/plugin_interface/src/plugin_interface.rs`
- 加载器：`services/plugin/src/asset_plugin.rs`（AssetPlugin, 加载器）

---

## 安全边界

### TEE 边界

```mermaid
graph TB
    subgraph "Normal World"
        App[应用]
        NAPI[N-API]
        SDK[SDK]
        Service[Asset Service]
    end

    subgraph "TEE (Trusted Execution Environment)"
        HUKS[HUKS]
        Crypto[Crypto Operations<br/>AES-256-GCM]
        KeyStore[Key Storage<br/>密钥不暴露]
    end

    App --> NAPI --> SDK --> Service
    Service -.加密/解密请求.-> HUKS
    HUKS --> Crypto --> KeyStore

    Style KeyStore fill:#f9f,stroke:#f00,stroke-width:3px
    Note over KeyStore: 硬件保护<br/>密钥不离开 TEE
```

**关键点**：
- 密钥生成：在 HUKS 中进行
- 加密/解密：在 HUKS 中进行
- 密钥存储：在 TEE 中，不暴露给 Normal World
- 认证验证：在 HUKS 中验证，防重放攻击

**证据**：`services/crypto_manager/src/huks_wrapper.c:1-100`（所有 HUKS 操作）

---

## 相关跳转

- [项目概述](00_Overview.md) - 了解项目定位
- [对外 N-API](03_NAPI_API.md) - 详细的 API 清单
- [内部 API](04_Inner_API.md) - 系统间接口
- [安全风险评审](07_Security_Review.md) - 安全威胁分析
