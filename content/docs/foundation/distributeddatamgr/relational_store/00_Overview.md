# 项目概览

## 目的

本文档提供 relational_store（关系型数据库）组件的快速概览，帮助新人理解项目定位、核心能力、运行环境和关键概念。

## 适用范围

- OpenHarmony 关系型数据库子系统
- 版本：3.1.0
- 子系统：distributeddatamgr
- 基于 SQLite 的本地数据管理组件

## 关键结论

### 1. 项目定位

**relational_store** 是 OpenHarmony 提供的关系型数据库持久化方案，基于 SQLite 引擎实现，支持：
- **本地 CRUD 操作**：创建、读取、更新、删除数据
- **ACID 事务**：支持事务、保存点、回滚
- **索引与查询优化**：支持索引、视图、触发器、外键约束
- **参数化查询**：预编译 SQL 语句，防止 SQL 注入
- **云同步**：支持与云端和其他设备的分布式数据同步
- **并发控制**：连接池最大 4 个，同时只支持一个写操作

### 2. 核心能力

| 能力 | 说明 | 证据 |
|------|------|------|
| **关系型存储** | 基于 SQLite 的完整关系型数据库特性 | `README_zh.md:7` |
| **事务支持** | ACID 事务（BEGIN、COMMIT、ROLLBACK） | `rdb_store.h:593-614` |
| **分布式同步** | 支持设备间和云端数据同步 | `rdb_store.h:693-724` |
| **加密存储** | 支持数据库加密（HUKS 集成） | `rdb_security_manager.cpp` |
| **查询谓词** | 类型安全的查询构建 API | `rdb_predicates.h` |
| **观察者模式** | 数据变更订阅与通知 | `rdb_store.h:714-729` |
| **并发控制** | 连接池管理（最大 4 个连接） | `connection_pool.cpp` |

### 3. 运行环境

| 特性 | 说明 | 配置 |
|------|------|------|
| **目标平台** | OpenHarmony 标准系统 | `bundle.json:46-48` |
| **跨平台支持** | MinGW/Windows, Mac, Android, iOS（测试用） | 多个 BUILD.gn 文件 |
| **最小 ROM** | 1000 KB | `bundle.json:49` |
| **最小 RAM** | 350 KB | `bundle.json:50` |
| **SysCap** | SystemCapability.DistributedDataManager.RelationalStore.Core | `bundle.json:39` |

### 4. 关键概念

#### RDB Store（关系型数据库存储）
- 本地数据库文件（.db, .db-wal, .db-shm）
- 表、列、索引、视图的集合
- 支持 BLOB、INTEGER、REAL、TEXT 数据类型

#### ResultSet（结果集）
- 查询返回的数据集合
- 提供迭代访问：goToFirstRow、goToNextRow、goTo
- 支持列索引访问和按列名访问

#### RdbPredicates（查询谓词）
- 类型安全的查询条件构建器
- 支持 equalTo、contains、between、like、in 等操作
- 链式 API：pred.equalTo("name").and().contains("value")

#### Transaction（事务）
- BEGIN/COMMIT/ROLLBACK 操作
- 支持 EXCLUSIVE 事务隔离级别
- 多种创建方式：CreateTransaction、BeginTrans

#### Distributed Tables（分布式表）
- 标记为可同步的表
- 支持设备间和云端同步
- 通过 SetDistributedTables() 配置

#### Asset（数据资产）
- 大文件/二进制数据封装
- 支持云端存储和分享
- 状态跟踪：ASSET_NORMAL、ASSET_INSERT、ASSET_UPDATE 等

### 5. 依赖关系

**外部依赖**（来自 `bundle.json:51-76`）：
- `sqlite` - SQLite 数据库引擎
- `ipc` - 进程间通信
- `hilog` - 日志系统
- `hitrace` - 性能追踪
- `huks` - 硬件密钥服务（加密）
- `kv_store` - KV 存储（分布式支持）
- `data_share` - 数据共享
- `ability_runtime` - 能力运行时
- `access_token` - 访问令牌（权限）
- `c_utils` - C 工具库
- `icu` - ICU Unicode 支持

**内部依赖**：
- `native_rdb` - 核心 RDB 库
- `native_appdatafwk` - 应用数据框架
- `rdb_data_share_adapter` - DataShare 适配器
- `relational_store_crypt` - 加密模块
- `relational_store_icu` - ICU 支持

### 6. 约束与限制

| 约束 | 说明 | 证据 |
|------|------|------|
| **连接池大小** | 最大 4 个数据库连接 | `README_zh.md:46` |
| **写操作并发** | 同一时间只支持一个写操作 | `README_zh.md:47` |
| **SQL 注入防护** | 使用参数化查询，直接 SQL 执行需谨慎 | `sqlite_statement.cpp` |
| **信任列表限制** | 只有信任列表中的 bundle 可使用特定功能 | `trusts_config.json` |
| **权限要求** | 云数据同步需要 CLOUDDATA_CONFIG 权限 | `test/ets/cloud_data_system/entry/src/main/module.json` |

### 7. 数据流概览

```
应用层 (JS/TS/ETS/C)
    ↓
N-API 层 (frameworks/js/napi/)
    ↓
内部 API 层 (interfaces/inner_api/)
    ↓
核心实现层 (frameworks/native/)
    ↓
SQLite 引擎 (third_party_sqlite)
```

## 相关跳转

- [01_Project_Positioning.md](./01_Project_Positioning.md) - 项目定位与边界详解
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 完整目录结构
- [03_Architecture.md](./03_Architecture.md) - 架构图与数据流
- [04_N-API.md](./04_N-API.md) - JavaScript/TypeScript API 清单
- [08_Security.md](./08_Security.md) - 安全风险与修复建议

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: README_zh.md, bundle.json, rdb_store.h, rdb_security_manager.cpp
