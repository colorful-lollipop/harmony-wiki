# 架构设计

## 整体架构图

```mermaid
graph TB
    subgraph "客户端应用"
        A["其他应用<br/>DataShare API"]
    end

    subgraph "SettingsData 应用"
        B["DataExtAbility<br/>DataShareExtensionAbility"]
        
        C["权限校验层<br/>verifyPermission"]
        
        D["业务逻辑层<br/>DoSystemSetting"]
        
        E["数据持久化层<br/>SettingsDBHelper"]
        
        F["用户管理<br/>UserChangeStaticSubscriber"]
    end

    subgraph "OpenHarmony 框架"
        G["relationalStore<br/>RDB 数据库"]
        H["Audio Manager<br/>系统音量设置"]
        I["abilityAccessCtrl<br/>权限管理"]
        J["commonEventManager<br/>公共事件"]
    end

    subgraph "数据存储"
        K["settingsdata.db<br/>SQLite 数据库"]
        L["default_settings.json<br/>默认配置"]
    end

    A --> B
    B --> C
    C --> D
    C --> E
    E --> K
    D --> H
    F --> J
    F --> E
    K --> L
    B --> I
```

---

## 组件职责

| 组件 | 职责 | 证据位置 |
|------|------|----------|
| **DataExtAbility** | DataShare 数据提供者，处理 CRUD 请求 | `DataExtAbility.ets:55` |
| **权限校验层** | 验证调用者是否有权限修改设置 | `DataExtAbility.ets:271-298` |
| **业务逻辑层** | 执行系统设置操作（如音量调节） | `DataExtAbility.ets:233-269` |
| **SettingsDBHelper** | RDB 数据库初始化与管理 | `SettingsDBHelper.ets:53` |
| **UserChangeStaticSubscriber** | 监听用户变更事件，管理用户数据表 | `UserChangeStaticSubscriber.ets:27` |

---

## 数据流分析

### 写入数据流 (Insert/Update)

```mermaid
sequenceDiagram
    participant Client as 客户端应用
    participant DataExt as DataExtAbility
    participant Perm as 权限校验
    participant System as 系统设置
    participant RDB as SettingsDBHelper
    participant DB as SQLite DB

    Client->>DataExt: insert(uri, value)
    DataExt->>Perm: verifyPermission(value)
    
    alt 受信任列表中的 key
        Perm-->>DataExt: GrantStatus=true
    else 调用者 UID 等于自身
        Perm-->>DataExt: GrantStatus=true
    else 拥有 MANAGE_SECURE_SETTINGS 权限
        Perm-->>DataExt: GrantStatus=true
    else
        Perm-->>DataExt: GrantStatus=false
        Perm-->>Client: 返回错误
    end
    
    DataExt->>System: DoSystemSetting(key, value)
    System->>Audio: setVolume(type, value)
    
    DataExt->>RDB: insert(TABLE_NAME, value)
    RDB->>DB: 写入数据库
    DB-->>RDB: 返回行 ID
    RDB-->>DataExt: 返回结果
    DataExt-->>Client: 异步回调结果
```

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:128-164`

### 查询数据流 (Query)

```mermaid
sequenceDiagram
    participant Client as 客户端应用
    participant DataExt as DataExtAbility
    participant RDB as SettingsDBHelper
    participant DB as SQLite DB

    Client->>DataExt: query(uri, predicates, columns)
    DataExt->>RDB: query(TABLE_NAME, predicates, columns)
    RDB->>DB: 执行查询
    DB-->>RDB: 返回 ResultSet
    RDB-->>DataExt: 返回 ResultSet
    DataExt-->>Client: 异步回调 ResultSet
```

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:209-231`

---

## 线程模型

### 主线程职责

| 操作 | 线程 | 说明 |
|------|------|------|
| DataAbility 生命周期 | 主线程 | `onCreate()`, `onInitialized()` |
| 权限校验 | 主线程 | `verifyPermission()` |
| 系统设置操作 | 主线程（异步） | `DoSystemSetting()` 调用 Audio API |
| 结果回调 | 主线程 | 异步回调返回主线程执行 |

### 异步操作

| 操作 | 异步机制 | 说明 |
|------|----------|------|
| RDB 初始化 | Promise | `initRdbStore()` 返回 Promise |
| 数据库操作 | Callback | insert/update/query 使用 AsyncCallback |
| 权限验证 | Promise | `verifyAccessToken()` 返回 Promise |
| 音频设置 | Promise | `setVolume()` 返回 Promise |

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.et:238-263`

```typescript
Audio.getAudioManager().setVolume(volumeType, Number(settingsValue)).then(() => {
  Log.info('settings Promise returned to indicate a successful RINGTONE setting.')
});
```

---

## 数据库架构

### 表结构

所有数据表使用相同的 schema：

```sql
CREATE TABLE IF NOT EXISTS <TABLE_NAME> (
  ID INTEGER PRIMARY KEY AUTOINCREMENT,
  KEYWORD TEXT,
  VALUE TEXT CHECK (LENGTH(VALUE) <= 1000)
)
```

**证据**: `entry/src/main/ets/Utils/SettingsDBHelper.ets:54-63`

### 数据表分类

| 表名 | 用途 | 证据 |
|------|------|------|
| SETTINGSDATA | 公共设置数据 | `SettingsDataConfig.ets:27` |
| USER_SETTINGSDATA_<userId> | 用户设置数据 | `SettingsDataConfig.ets:28` |
| USER_SETTINGSDATA_SECURE_<userId> | 用户安全设置数据 | `SettingsDataConfig.ets:29` |

### 初始数据加载流程

```mermaid
flowchart TD
    A[首次启动] --> B{检查 isFirstStartup}
    B -->|true| C[读取 default_settings.json]
    B -->|false| D[跳过加载]
    
    C --> E[加载 SETTINGSDATA 表数据]
    C --> F[加载 USER_SETTINGSDATA_<100> 数据]
    C --> G[加载 USER_SETTINGSDATA_SECURE_<100> 数据]
    C --> H[初始化设备名称]
    C --> I[初始化亮度值]
    C --> J[初始化克隆标识]
    C --> K[写入 isFirstStartup=false]
    
    E --> L[初始化完成]
    F --> L
    G --> L
    H --> L
    I --> L
    J --> L
    K --> L
```

**证据**: `entry/src/main/ets/Utils/SettingsDBHelper.ets:133-160`

---

## 权限模型

### 权限校验流程

```mermaid
flowchart TD
    A[收到 insert/update 请求] --> B{key 是否在受信任列表?}
    B -->|yes| C[允许操作]
    B -->|no| D{调用者 UID == 自身?}
    D -->|yes| C
    D -->|no| E{检查 MANAGE_SECURE_SETTINGS 权限}
    E -->|granted| C
    E -->|denied| F[拒绝操作]
```

### 受信任列表

以下设置项无需权限即可修改：

| 设置项 | 说明 |
|--------|------|
| `settings.display.SCREEN_BRIGHTNESS_STATUS` | 屏幕亮度状态 |
| `settings.display.AUTO_SCREEN_BRIGHTNESS` | 自动亮度开关 |
| `settings.display.SCREEN_OFF_TIMEOUT` | 屏幕超时时间 |

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:47-51`

```typescript
let trustList: String[] = [
  settings.display.SCREEN_BRIGHTNESS_STATUS,
  settings.display.AUTO_SCREEN_BRIGHTNESS,
  settings.display.SCREEN_OFF_TIMEOUT
];
```

---

## 用户管理机制

### 用户事件监听

```mermaid
sequenceDiagram
    participant System as 系统
    participant Subscriber as UserChangeStaticSubscriber
    participant RDB as SettingsDBHelper
    participant DB as SQLite DB

    System->>Subscriber: COMMON_EVENT_USER_ADDED
    Subscriber->>RDB: 创建 USER_SETTINGSDATA_<userId> 表
    Subscriber->>RDB: 创建 USER_SETTINGSDATA_SECURE_<userId> 表
    Subscriber->>RDB: 加载默认数据
    RDB->>DB: 执行建表和数据插入
    
    System->>Subscriber: COMMON_EVENT_USER_REMOVED
    Subscriber->>RDB: 删除 USER_SETTINGSDATA_<userId> 表
    Subscriber->>RDB: 删除 USER_SETTINGSDATA_SECURE_<userId> 表
```

**证据**: `entry/src/main/ets/StaticSubscriber/UserChangeStaticSubscriber.ets:33-79`

---

## 关键时序图

### 应用启动时序

```mermaid
sequenceDiagram
    participant Sys as 系统
    participant Stage as DataAbilityStage
    participant Context as GlobalContext
    participant Helper as SettingsDBHelper
    participant DataExt as DataExtAbility

    Sys->>Stage: onCreate()
    Stage->>Context: setObject('abilityContext', context)
    Stage->>DataExt: onCreate(want)
    DataExt->>Context: setObject('abilityContext', context)
    DataExt->>Context: setObject('settingsDBHelper', Helper)
    DataExt->>Helper: getArea()
    DataExt->>Helper: getRdbStore()
    Helper->>Helper: initRdbStore()
    Helper->>Helper: firstStartupConfig()
    Helper-->>DataExt: 返回 rdbStore
    DataExt->>DataExt: onInitialized()
    DataExt->>Helper: 执行待处理的请求队列
```

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:56-126`

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [01_Project_Overview.md](./01_Project_Overview.md) |
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |
