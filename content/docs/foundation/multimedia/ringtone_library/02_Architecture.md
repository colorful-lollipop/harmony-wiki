# 架构与数据流

> **适用对象**: 新人学习者、安全研究员
> **阅读时间**: 20 分钟
> **前置知识**: DataShare 框架、RDB 数据库

---

## 目的与适用范围

本文档详细说明 RingtoneLibrary 的架构设计，包括：
- 组件图和模块依赖
- 数据流和信任边界
- 线程模型和时序
- 关键流程说明

**适用场景**:
- 开发者：理解系统架构，定位问题
- 安全研究员：识别潜在攻击点和数据流

---

## 系统架构图

### 整体架构

```mermaid
graph TB
    subgraph "应用层"
        A1[系统应用<br/>SystemSoundManager]
        A2[音乐开放能力<br/>RingtoneKit]
        A3[N-API 模块<br/>ringtonerestore]
    end

    subgraph "IPC 层"
        B1[DataShare Framework]
        B2[Binder IPC]
    end

    subgraph "RingtoneLibrary 服务层"
        C1[DataShareExtension<br/>RingtoneDataShareExtension]
        C2[DataManager<br/>RingtoneDataManager]
        C3[扫描器<br/>RingtoneScannerManager]
        C4[设置管理<br/>RingtoneSettingManager]
        C5[恢复模块<br/>RingtoneRestore]
        C6[DFX<br/>DfxManager]
    end

    subgraph "存储层"
        D1[RDB 数据库<br/>ToneFiles/VibrateFiles/SimCardSetting]
        D2[文件系统<br/>/data/storage/.../Ringtone/]
        D3[配置文件<br/>/etc/param/]
    end

    A1 --> B1
    A2 --> B1
    A3 --> B2

    B1 --> C1
    B2 --> C5

    C1 --> C2
    C1 --> C3
    C1 --> C4
    C1 --> C5
    C1 --> C6

    C2 --> D1
    C3 --> D1
    C4 --> D1
    C5 --> D1

    C1 --> D2
    C3 --> D2

    C3 --> D3
    C4 --> D3

    style A1 fill:#51cf66
    style A2 fill:#51cf66
    style A3 fill:#51cf66
    style C1 fill:#feca57
    style C2 fill:#feca57
    style C3 fill:#feca57
    style C4 fill:#feca57
    style C5 fill:#feca57
    style C6 fill:#feca57
    style D1 fill:#ff6b6b
    style D2 fill:#ff6b6b
    style D3 fill:#ff6b6b
```

**证据**: `README_zh.md:10-11`, `services/ringtone_data_extension/src/ringtone_datashare_extension.cpp`

---

## 模块职责

### 核心模块

| 模块 | 主要职责 | 关键类 | 证据 |
|------|---------|---------|------|
| **DataShareExtension** | DataShare 接口实现，IPC 入口 | `RingtoneDataShareExtension` | `ringtone_datashare_extension.cpp` |
| **DataManager** | 数据操作管理，RDB CRUD | `RingtoneDataManager` | `ringtone_data_manager.cpp` |
| **Scanner** | 扫描系统铃音，元数据提取 | `RingtoneScannerObj` | `ringtone_scanner.cpp` |
| **SettingManager** | 铃音设置管理（来电/闹钟/通知） | `RingtoneSettingManager` | `ringtone_setting_manager.cpp` |
| **Restore** | 备份恢复，跨版本迁移 | `RingtoneRestore` | `ringtone_restore.cpp` |
| **DFX** | 诊断和故障报告 | `DfxManager` | `dfx_manager.cpp` |

**证据**: `services/` 各模块源文件

---

## 数据流

### 1. 查询流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant DS as DataShareHelper
    participant Ext as RingtoneDataShareExtension
    participant DM as RingtoneDataManager
    participant RDB as RDB

    App->>DS: Query(uri, predicates, columns)
    DS->>Ext: Query()
    Ext->>Ext: CheckRingtonePerm()
    Ext->>DM: Query(predicates)
    DM->>RDB: QuerySql()
    RDB-->>DM: ResultSet
    DM-->>Ext: ResultSet
    Ext-->>DS: ResultSet
    DS-->>App: ResultSet

    Note over Ext,DM: 信任边界：权限检查
    Note over DM,RDB: 隐信边界：参数化查询
```

**证据**: `ringtone_datashare_extension.cpp:421`, `ringtone_data_manager.cpp`

---

### 2. 新增铃音流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant DS as DataShareHelper
    participant Ext as RingtoneDataShareExtension
    participant DM as RingtoneDataManager
    participant FS as FileUtils
    participant RDB as RDB

    App->>DS: Insert(uri, values)
    DS->>Ext: Insert()
    Ext->>Ext: CheckRingtonePerm()
    Ext->>FS: ValidateAndCopyFile()
    FS-->>Ext: filePath
    Ext->>DM: Insert(values)
    DM->>RDB: InsertSql()
    RDB-->>DM: toneId
    DM-->>Ext: toneId
    Ext-->>DS: toneId
    DS-->>App: toneId

    Note over Ext,FS: 信任边界：文件验证
    Note over DM,RDB: 隐信边界：SQL 参数化
```

**证据**: `ringtone_datashare_extension.cpp:356`, `ringtone_file_utils.cpp`

---

### 3. 扫描流程

```mermaid
sequenceDiagram
    participant Sys as 系统
    participant Ext as RingtoneDataShareExtension
    participant SM as ScannerManager
    participant SC as RingtoneScanner
    participant ME as MetadataExtractor
    participant RDB as RDB

    Sys->>Ext: OnStart()
    Ext->>SM: StartScan()
    SM->>SC: Scan(dir)
    SC->>SC: EnumerateFiles()
    loop 每个文件
        SC->>ME: Extract(file)
        ME-->>SC: metadata
        SC->>RDB: Insert(metadata)
    end
    SC-->>SM: scanCount
    SM-->>Ext: scanResult
    Ext->>Ext: NotifyScanComplete()

    Note over SC,RDB: 信任边界：文件类型检查
```

**证据**: `ringtone_scanner.cpp`, `ringtone_metadata_extractor.cpp`

---

### 4. 静默访问流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant DS as DataShareHelper
    participant RDB as RDB
    participant Ext as RingtoneLibrary（未启动）

    App->>DS: Query(datashareproxy://...)
    DS->>RDB: Direct Query（不启动 Ext）
    RDB-->>DS: ResultSet
    DS-->>App: ResultSet

    Note over DS,RDB: 隐信边界：直接访问数据库
    Note right of App: 要求 ACCESS_CUSTOM_RINGTONE 权限
```

**证据**: `README_zh.md:184-241`

---

## 线程模型

### 主线程

- **职责**：处理 DataShare 请求（Insert/Update/Delete/Query/OpenFile）
- **实现**：`RingtoneDataShareExtension` 单例
- **证据**: `ringtone_datashare_extension.cpp`

---

### 扫描线程

- **职责**：异步扫描铃音目录，不阻塞主线程
- **实现**：`RingtoneScanExecutor`
- **线程池**：使用工作队列管理扫描任务
- **证据**: `ringtone_scan_executor.cpp`

---

### DFX 线程

- **职责**：异步上报故障和诊断信息
- **实现**：`DfxWorker`
- **线程池**：使用事件队列
- **证据**: `dfx_worker.cpp`

---

### N-API 线程

- **职责**：处理 JavaScript 调用（startRestore）
- **实现**：`RingtoneRestoreNapi`
- **异步**：使用 Promise/Callback 机制
- **证据**: `ringtone_restore_napi.cpp`

---

## 关键时序

### 初始化时序

```mermaid
sequenceDiagram
    participant SA as SA 框架
    participant Ext as RingtoneDataShareExtension
    participant DM as RingtoneDataManager
    participant DFX as DfxManager
    participant SM as ScannerManager
    participant RDB as RDB

    SA->>Ext: OnStart()
    Ext->>Ext: Init()
    Ext->>DM: GetInstance().Init()
    DM->>RDB: Init()
    RDB-->>DM: OK

    Ext->>DFX: Init()
    DFX-->>Ext: OK

    Ext->>SM: StartScan()
    SM->>SM: ExecuteScan()
    SM-->>Ext: ScanResult

    Ext-->>SA: Ready
```

**证据**: `ringtone_datashare_extension.cpp:135-172`

---

### 删除铃音时序

```mermaid
sequenceDiagram
    participant App as 应用
    participant Ext as RingtoneDataShareExtension
    participant DM as RingtoneDataManager
    participant FS as FileUtils
    participant RDB as RDB

    App->>Ext: Delete(predicates)
    Ext->>Ext: CheckRingtonePerm()
    Ext->>DM: Query(predicates)
    DM->>RDB: QuerySql()
    RDB-->>DM: resultSet

    loop 每条记录
        DM->>FS: DeleteFile(path)
        DM->>RDB: Delete(toneId)
    end

    DM-->>Ext: deletedCount
    Ext-->>App: deletedCount
```

**证据**: `ringtone_datashare_extension.cpp:401`, `ringtone_data_manager.cpp`

---

## 数据存储模型

### RDB 数据库

```
/data/service_el2/
└── ringtone_library/
    ├── ringtone.db          # 主数据库
    │   ├── ToneFiles       # 铃音表
    │   ├── VibrateFiles    # 振动表
    │   ├── SimCardSetting  # SIM 卡设置
    │   └── PreloadConfig   # 预加载配置
```

**证据**: `ringtone_rdbstore.cpp:Init()`

---

### 文件系统

```
/data/storage/el2/base/files/Ringtone/
├── ringtones/             # 用户铃音
│   ├── ringtone1.ogg
│   └── custom_ringtone.m4a
├── vibrate/              # 振动文件
│   └── vibrate_pattern.bin
└── backup/               # 备份文件
    └── backup_20240101/
```

**证据**: `ringtone_db_const.h:RINGTONE_DATA_PATH`

---

## 信任边界

### 边界 1：应用 ↔ DataShareExtension

**安全控制**：
- `CheckRingtonePerm()` 权限校验
- `IsSystemApp()` 系统应用检查
- AccessToken 验证

**证据**: `ringtone_datashare_extension.cpp:200-216`

---

### 边界 2：DataShareExtension ↔ RDB

**安全控制**：
- 参数化 SQL 查询（防 SQL 注入）
- 事务隔离
- 数据库文件权限限制

**证据**: `ringtone_rdbstore.cpp`

---

### 边界 3：RingtoneLibrary ↔ 文件系统

**安全控制**：
- 路径规范化（待确认完整实现）
- 符号链接检查（待确认）
- 文件类型验证

**证据**: `ringtone_file_utils.cpp`

---

### 边界 4：扫描器 ↔ 文件系统

**安全控制**：
- 扫描路径白名单
- MIME 类型检查
- 文件大小限制

**证据**: `ringtone_scanner.cpp`, `ringtone_metadata_extractor.cpp`

---

## 关键结论

1. **架构模式**：DataShareExtension 单例模式，RDB 存储模式
2. **数据流**：应用 → DataShare → Extension → DataManager → RDB/FS
3. **线程模型**：主线程 + 扫描线程 + DFX 线程 + N-API 线程
4. **信任边界**：4 个关键边界，需要验证安全控制完整性
5. **存储**：RDB 数据库 + 文件系统（/data/storage/...）

---

## 相关链接

- [项目概览](./01_Overview.md) - 了解项目定位
- [攻击面分析](./05_AttackSurface.md) - 详细攻击面映射
- [内部实现细节](./08_Internals.md) - 深入核心逻辑

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
