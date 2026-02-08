# 项目定位与边界

## 目的

本文档定义 relational_store 组件的定位、边界、核心能力、运行环境和关键概念。

## 适用范围

- OpenHarmony 关系型数据库组件
- 定位在 distributeddatamgr 子系统
- 提供本地持久化数据管理能力

## 关键结论

### 1. 组件定位

**relational_store** 是 OpenHarmony 的**本地关系型数据库解决方案**，提供：

| 职责 | 说明 | 证据 |
|------|------|------|
| **本地数据持久化** | 基于 SQLite 的关系型数据库存储 | `README_zh.md:5` |
| **关系型数据模型** | 支持表、列、索引、外键、视图、触发器 | `rdb_store.h` |
| **分布式数据管理** | 支持设备间和云端数据同步 | `rdb_store.h:693-734` |
| **ACID 事务** | 完整的事务支持（原子性、一致性、隔离性、持久性） | `rdb_store.h:593-614` |
| **并发控制** | 连接池管理（最大 4 个连接） | `README_zh.md:46` |

### 2. 边界定义

#### 2.1 功能边界

| 包含功能 | 不包含功能 | 证据 |
|----------|----------|------|
| 本地 CRUD 操作 | 分布式事务（跨设备） | `rdb_store.h:166-218` |
| 单表事务 | 多表联合事务（需应用层实现） | `rdb_store.h:593-614` |
| 关系型查询（SQL, Predicates） | 非关系型文档存储（使用 kv_store） | `rdb_predicates.h` |
| 结果集迭代 | 流式数据处理（大文件） | `result_set.h` |
| 数据加密 | 行级加密（使用 file_api 加密文件） | `rdb_security_manager.cpp` |
| 数据变更订阅 | 实时数据推送（需应用层实现） | `rdb_store.h:714-729` |

#### 2.2 模块边界

| 模块 | 职责 | 依赖模块 | 证据 |
|------|------|----------|------|
| **Core RDB** | 核心 CRUD、事务、查询 | 无 | `interfaces/inner_api/rdb/` |
| **Cloud Data** | 云同步管理 | Core RDB, HUKS, DeviceManager | `interfaces/inner_api/cloud_data/` |
| **DataAbility Adapter** | DataAbility 适配层 | Core RDB, DataAbility | `interfaces/inner_api/rdb_data_ability_adapter/` |
| **DataShare Adapter** | DataShare 适配层 | Core RDB, DataShare | `interfaces/inner_api/rdb_data_share_adapter/` |
| **AppDataFwk** | 共享块、序列化 | 无 | `interfaces/inner_api/appdatafwk/` |

#### 2.3 API 边界

**NDK C API 边界**（`interfaces/ndk/`）：
- 提供 C 语言接口
- 不暴露内部实现细节（RdbStoreImpl、RdbHelper 等）
- 错误码统一在 `relational_store_error_code.h`

**Inner API 边界**（`interfaces/inner_api/`）：
- 供其他子系统使用（如 data_share, datamgr_service）
- 部分接口标注可见性（如 cloud_data_inner 限制 visibility）
- 包含实现类头文件（非纯接口）

**N-API 边界**（`frameworks/js/napi/`）：
- 只暴露 JS/TypeScript 需要的 API
- 使用 `napi_define_class`、`napi_define_property` 定义绑定
- 内部工具函数不导出到 JS

### 3. 核心能力

#### 3.1 数据操作

| 能力 | API | 说明 |
|------|-----|------|
| **插入** | `Insert()` | 单行插入，支持冲突解决策略 |
| **批量插入** | `BatchInsert()` | 批量插入，支持 RETURNING 子句 |
| **更新** | `Update()` | 基于条件或 Predicates 更新 |
| **删除** | `Delete()` | 基于条件或 Predicates 删除 |
| **查询** | `Query()`, `QuerySql()`, `QueryByStep()` | 多种查询方式 |
| **执行 SQL** | `Execute()`, `ExecuteExt()` | 执行任意 SQL 语句 |

#### 3.2 事务管理

| 能力 | API | 说明 |
|------|-----|------|
| **创建事务** | `CreateTransaction()` | 创建事务对象 |
| **开始事务** | `BeginTrans()` | EXCLUSIVE 模式事务 |
| **提交** | `Commit()` | 提交事务 |
| **回滚** | `RollBack()` | 回滚事务 |
| **检查状态** | `IsInTransaction()` | 检查是否在事务中 |

#### 3.3 分布式能力

| 能力 | API | 说明 |
|------|-----|------|
| **设置分布式表** | `SetDistributedTables()` | 标记表为可同步 |
| **同步到设备** | `Sync(SyncOption, AbsRdbPredicates, AsyncBrief)` | 异步同步到指定设备 |
| **同步到云端** | `Sync(SyncOption, std::vector<std::string>, AsyncDetail)` | 异步同步到云端 |
| **订阅变更** | `Subscribe()`, `SubscribeObserver()` | 订阅数据变更通知 |
| **远程查询** | `RemoteQuery()` | 查询远程设备数据 |
| **获取分布式表名** | `ObtainDistributedTableName()` | 获取设备的分布式表名 |

#### 3.4 安全能力

| 能力 | API | 说明 |
|------|-----|------|
| **数据库加密** | `Rekey()` | 更改加密密钥 |
| **数据库备份** | `Backup()` | 备份数据库（支持加密） |
| **数据库恢复** | `Restore()` | 从备份恢复数据库 |
| **安全级别** | `SecurityPolicy::SetSecurityLabel()` | 设置 S1-S4 安全级别 |
| **附加数据库** | `Attach()` | 附加其他数据库（支持加密） |

### 4. 运行环境

#### 4.1 系统要求

| 要求 | 说明 | 证据 |
|------|------|------|
| **操作系统** | OpenHarmony 标准系统 | `bundle.json:46-48` |
| **最小 ROM** | 1000 KB | `bundle.json:49` |
| **最小 RAM** | 350 KB | `bundle.json:50` |
| **C++ 标准** | C++17 | `frameworks/js/napi/relationalstore/BUILD.gn:103` |

#### 4.2 SysCap

```json
[
  "SystemCapability.DistributedDataManager.CloudSync.Client",
  "SystemCapability.DistributedDataManager.CloudSync.Server",
  "SystemCapability.DistributedDataManager.CloudSync.Config",
  "SystemCapability.DistributedDataManager.RelationalStore.Core",
  "SystemCapability.DistributedDataManager.CommonType"
]
```

证据：`bundle.json:35-41`

#### 4.3 特性开关

| 特性 | 默认值 | 说明 | 证据 |
|------|--------|------|------|
| `relational_store_rdb_support_icu` | true | 支持 ICU（Unicode 排序） | `relational_store.gni:15` |
| `relational_store_config` | true | 启用配置模块 | `relational_store.gni:22` |
| `arkdata_db_core_is_exists` | 动态检测 | arkdata 数据库核心是否存在 | `relational_store.gni:16-21` |
| `relational_store_dm_part_is_enabled` | 动态检测 | device_manager 部件是否启用 | `relational_store.gni:24-28` |

### 5. 与其它子系统的关系

```
                    ┌──────────────────────┐
                    │  distributeddatamgr  │
                    │       子系统         │
                    └──────────┬───────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │relational_store│   │  kv_store    │   │  data_share    │
    │  (RDB)      │   │  (KV)       │   │  (Sharing)   │
    └──────┬───────┘   └──────────────┘   └──────────────┘
           │
           │              ┌─────────┬─────────┐
           │              │         │         │
           │              ▼         ▼         ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │  cloud_data  │   │  sqlite      │   │  ipc         │
    │ (Sync)      │   │  (Storage)   │   │  (Communication)│
    └──────────────┘   └──────────────┘   └──────────────┘
```

**依赖关系**：
- **sqlite**: 提供底层存储引擎
- **ipc**: 用于分布式同步的进程间通信
- **kv_store**: 用于云同步的分布式数据存储
- **data_share**: 用于跨应用数据共享
- **ability_runtime**: 提供系统能力加载和生命周期管理

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 详细架构图与数据流
- [05_Inner_API.md](./05_Inner_API.md) - 内部接口定义
- [08_Security.md](./08_Security.md) - 安全机制与风险

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: README_zh.md, bundle.json, relational_store.gni, rdb_store.h
