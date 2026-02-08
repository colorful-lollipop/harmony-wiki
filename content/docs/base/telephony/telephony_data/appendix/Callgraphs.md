# 关键调用链图

## A.1 SIM 卡查询调用链

### A.1.1 数据查询完整调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as DataShareHelper
    participant SAMgr as SystemAbilityManager
    participant Stub as TelephonyDataShareStub
    participant SimAbility as SimAbility
    participant Perm as PermissionUtil
    participant RDB as RdbStore

    App->>Helper: Query(simUri, predicates, columns)
    Helper->>SAMgr: GetSystemAbility(TELEPHONY_CORE_SA_ID)
    SAMgr-->>Helper: IRemoteObject
    
    Note over Helper,Stub: IPC 连接建立
    
    Helper->>Stub: Query(uri, predicates, columns)
    Stub->>Stub: GetOwner(uri)
    Stub->>SimAbility: Query(uri, predicates, columns)
    
    Note over SimAbility,Perm: 权限检查
    
    SimAbility->>Perm: CheckPermission(GET_TELEPHONY_STATE)
    Perm->>IPCSkeleton: GetCallingTokenID()
    IPCSkeleton-->>Perm: TokenID
    Perm->>AccessTokenKit: VerifyAccessToken(TokenID, permission)
    AccessTokenKit-->>Perm: GRANTED/DENIED
    Perm-->>SimAbility: true/false
    
    alt Permission denied
        SimAbility-->>Stub: ERR_PERMISSION
        Stub-->>Helper: ERR_PERMISSION
        Helper-->>App: throw Exception
    else Permission granted
        SimAbility->>RDB: ExecuteQuery(predicates)
        RDB-->>SimAbility: ResultSet
        SimAbility-->>Stub: DataShareResultSet
        Stub-->>Helper: DataShareResultSet
        Helper-->>App: DataShareResultSet
    end
```

### A.1.2 SIM 卡插入调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as DataShareHelper
    participant Stub as TelephonyDataShareStub
    participant SimAbility as SimAbility
    participant Perm as PermissionUtil
    participant RDB as RdbStore

    App->>Helper: Insert(simUri, value)
    Helper->>Stub: Insert(uri, value)
    Stub->>SimAbility: Insert(uri, value)
    
    SimAbility->>Perm: CheckPermission(SET_TELEPHONY_STATE)
    Perm-->>SimAbility: true/false
    
    alt Permission denied
        SimAbility-->>Stub: ERR_PERMISSION
    else Permission granted
        SimAbility->>RDB: ExecuteInsert(predicates, value)
        RDB-->>SimAbility: rowId
        SimAbility-->>Stub: rowId
    end
    
    Stub-->>Helper: rowId/ERR
    Helper-->>App: rowId/throw Exception
```

---

## A.2 SMS/MMS 查询调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as DataShareHelper
    participant SAMgr as SystemAbilityManager
    participant Stub as TelephonyDataShareStub
    participant SmsAbility as SmsMmsAbility
    participant Perm as PermissionUtil
    participant RDB as RdbStore

    Note over App: SMS/MMS 需要 READ_MESSAGES 权限
    
    App->>Helper: Query(smsUri, predicates, columns)
    Helper->>SAMgr: GetSystemAbility(TELEPHONY_SMS_MMS_SA_ID)
    SAMgr-->>Helper: IRemoteObject
    
    Helper->>Stub: Query(uri, predicates, columns)
    Stub->>SmsAbility: GetOwner(uri)
    SmsAbility-->>Stub: this
    
    Stub->>SmsAbility: Query(uri, predicates, columns)
    SmsAbility->>Perm: CheckPermission(READ_MESSAGES)
    
    Note over Perm: 注意: SMS/MMS 使用不同权限
    
    Perm-->>SmsAbility: true/false
    
    alt Permission granted
        SmsAbility->>RDB: ExecuteQuery(predicates)
        RDB-->>SmsAbility: ResultSet
        SmsAbility->>SmsAbility: Convert to DataShareResultSet
        SmsAbility-->>Stub: ResultSet
        Stub-->>Helper: ResultSet
        Helper-->>App: DataShareResultSet
    end
```

---

## A.3 PDP/APN 配置调用链

### A.3.1 PDP 查询调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as DataShareHelper
    participant Stub as TelephonyDataShareStub
    participant PdpAbility as PdpProfileAbility
    participant Perm as PermissionUtil
    participant RDB as RdbStore
    participant Encrypt as ApnEncryptionUtil

    App->>Helper: Query(pdpUri, predicates, columns)
    Helper->>Stub: Query(uri, predicates, columns)
    Stub->>PdpAbility: GetOwner(uri)
    PdpAbility-->>Stub: this
    
    Stub->>PdpAbility: Query(uri, predicates, columns)
    PdpAbility->>Perm: CheckPermission(GET_TELEPHONY_STATE)
    Perm-->>PdpAbility: true/false
    
    alt Permission granted
        PdpAbility->>RDB: ExecuteQuery(predicates)
        RDB-->>PdpAbility: ResultSet
        
        Note over PdpAbility: 敏感字段解密
        
        loop For each row
            PdpAbility->>Encrypt: DecryptApnData(USER_NAME)
            Encrypt-->>PdpAbility: decrypted
            PdpAbility->>Encrypt: DecryptApnData(PASSWORD)
            Encrypt-->>PdpAbility: decrypted
        end
        
        PdpAbility-->>Stub: DataShareResultSet
        Stub-->>Helper: ResultSet
        Helper-->>App: DataShareResultSet
    end
```

### A.3.2 PDP 插入调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant Helper as DataShareHelper
    participant Stub as TelephonyDataShareStub
    participant PdpAbility as PdpProfileAbility
    participant Perm as PermissionUtil
    participant Encrypt as ApnEncryptionUtil
    participant RDB as RdbStore

    App->>Helper: Insert(pdpUri, value)
    
    Note over Helper: value 包含敏感字段
    
    Helper->>Stub: Insert(uri, value)
    Stub->>PdpAbility: Insert(uri, value)
    PdpAbility->>Perm: CheckPermission(SET_TELEPHONY_STATE)
    Perm-->>PdpAbility: true/false
    
    alt Permission granted
        Note over PdpAbility: 敏感字段加密
        
        PdpAbility->>Encrypt: EncryptApnData(USER_NAME)
        Encrypt-->>PdpAbility: encrypted
        PdpAbility->>Encrypt: EncryptApnData(PASSWORD)
        Encrypt-->>PdpAbility: encrypted
        
        PdpAbility->>RDB: ExecuteInsert(predicates, value)
        RDB-->>PdpAbility: rowId
        PdpAbility-->>Stub: rowId
        Stub-->>Helper: rowId
        Helper-->>App: rowId
    end
```

---

## A.4 配置文件加载调用链

```mermaid
sequenceDiagram
    participant System as 系统启动
    participant Parser as ParserUtil
    participant Json as JSON 文件
    participant RDB as RdbStore

    System->>Parser: ParseJson("pdp_profile.json")
    Parser->>Json: Open and read file
    
    Note over Parser: 路径遍历检查
    
    Parser->>Json: Parse JSON content
    
    Json-->>Parser: JSON object
    
    Parser->>Parser: Validate schema
    
    alt Validation passed
        Parser-->>System: Valid config
        System->>RDB: BatchInsert(config)
        RDB-->>System: rowIds
    else Validation failed
        Parser-->>System: Error
        System->>System: Use default config
    end
```

---

## A.5 权限检查调用链

```mermaid
flowchart TD
    A[收到 API 请求] --> B{操作类型?}
    B -->|Write| C[需要 SET_TELEPHONY_STATE]
    B -->|Read| D[需要 GET_TELEPHONY_STATE]
    B -->|SMS/MMS| E[需要 READ_MESSAGES]
    
    C --> F[IPCSkeleton.GetCallingTokenID]
    D --> F
    E --> F
    
    F --> G[AccessTokenKit.VerifyAccessToken<br/>token, permission]
    
    G --> H{返回结果?}
    H -->|PERMISSION_GRANTED| I[允许执行]
    H -->|PERMISSION_DENIED| J[返回 ERROR_PERMISSION]
    
    I --> K[执行业务逻辑]
    J --> L[抛出安全异常]
```

---

## A.6 完整启动流程

```mermaid
flowchart TD
    A[SystemAbilityManager<br/>启动 telephony_data SA] --> B[加载 libtel_telephony_data.z.so]
    B --> C[创建 TelephonyDataShareStubImpl]
    C --> D[注册各模块 Ability]
    D --> E{配置文件存在?}
    E -->|是| F[ParserUtil.ParseJson<br/>etc/*.json]
    E -->|否| G[使用默认配置]
    
    F --> H[验证 Schema]
    H --> I{验证通过?}
    I -->|是| J[初始化 RDB]
    I -->|否| K[记录错误日志]
    K --> G
    
    J --> L[RdbHelper.GetRdbStore<br/>创建数据库表]
    L --> M[注册 DataAbility 到 SAMgr]
    M --> N[等待 IPC 请求]
    
    G --> J
```

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |
| 安全评审 | [07_Security](07_Security.md) |

---

*最后更新: 2024-02-06*
