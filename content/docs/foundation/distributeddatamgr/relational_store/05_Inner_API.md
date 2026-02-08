# 内部 API（Inner API）

## 目的

本文档描述 relational_store 组件的内部接口，包括模块接口、依赖方向、稳定性和可替换点。

## 适用范围

- interfaces/inner_api/ 中的所有头文件
- 模块间的依赖关系
- 接口稳定性与可见性
- 可替换的组件边界

## 关键结论

### 1. Inner API 模块清单

| 模块 | 路径 | 职责 | 主要头文件 |
|------|------|------|----------|
| **rdb** | `interfaces/inner_api/rdb/` | 核心 RDB 接口 | rdb_store.h<br/>rdb_helper.h<br/>rdb_predicates.h<br/>value_object.h<br/>values_bucket.h |
| **cloud_data** | `interfaces/inner_api/cloud_data/` | 云同步接口 | cloud_manager.h<br/>cloud_service.h<br/>cloud_types.h |
| **dataability** | `interfaces/inner_api/dataability/` | DataAbility 适配 | ishared_result_set.h<br/>data_ability_predicates.h |
| **appdatafwk** | `interfaces/inner_api/appdatafwk/` | 应用数据框架 | shared_block.h<br/>serializable.h |
| **rdb_data_share_adapter** | `interfaces/inner_api/rdb_data_share_adapter/` | DataShare 适配 | rdb_utils.h |
| **rdb_data_ability_adapter** | `interfaces/inner_api/rdb_data_ability_adapter/` | DataAbility 适配 | rdb_data_ability_utils.h |
| **common_type** | `interfaces/inner_api/common_type/` | 公共类型 | common_types.h |

### 2. 核心 RDB 接口

#### 2.1 RdbStore 类

**头文件**：`interfaces/inner_api/rdb/include/rdb_store.h`

| 方法分类 | 方法签名 | 稳定性 | 证据 |
|----------|-----------|--------|------|
| **CRUD** | `Insert()`, `Update()`, `Delete()`, `Query()` | 稳定（虚函数） | `rdb_store.h:39-849` |
| **事务** | `CreateTransaction()`, `BeginTrans()`, `Commit()`, `RollBack()` | 稳定 | 同上 |
| **数据库管理** | `GetPath()`, `GetVersion()`, `SetVersion()`, `Backup()`, `Restore()` | 稳定 | 同上 |
| **分布式** | `SetDistributedTables()`, `Sync()`, `ObtainDistributedTableName()` | 稳定 | 同上 |
| **观察者** | `Subscribe()`, `UnSubscribe()`, `Notify()` | 稳定 | 同上 |
| **属性访问** | `IsOpen()`, `IsReadOnly()`, `IsInTransaction()` | 稳定 | 同上 |
| **加密** | `Rekey()`, `RekeyEx()` | 稳定 | 同上 |

**稳定级别**：✅ **核心稳定接口**（所有虚函数，实现由子类提供）

#### 2.2 RdbHelper 类

**头文件**：`interfaces/inner_api/rdb/include/rdb_helper.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetRdbStore()` | ✅ 稳定 | `rdb_helper.h:40-85` |
| `DeleteRdbStore()` | ✅ 稳定 | 同上 |
| `ExecuteSql()` | ✅ 稳定 | 同上 |

#### 2.3 AbsRdbPredicates 类

**头文件**：`interfaces/inner_api/rdb/include/abs_rdb_predicates.h`

| 方法分类 | 方法签名 | 稳定性 |
|----------|-----------|--------|
| **条件** | `EqualTo()`, `NotEqualTo()`, `Contains()`, `BeginsWith()` | ✅ 稳定（纯虚函数） |
| **逻辑** | `And()`, `Or()` | ✅ 稳定 | 同上 |
| **排序** | `OrderByAsc()`, `OrderByDesc()` | ✅ 稳定 | 同上 |
| **限制** | `LimitAs()`, `OffsetAs()`, `Distinct()` | ✅ 稳定 | 同上 |
| **分组** | `GroupBy()`, `IndexedBy()` | ✅ 稳定 | 同上 |

#### 2.4 AbsSharedResultSet 类

**头文件**：`interfaces/inner_api/rdb/include/abs_shared_result_set.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetRowCount()` | ✅ 稳定 | `abs_shared_result_set.h` |
| `GetAllRows()` | ✅ 稳定 | 同上 |
| `GetColumnNames()` | ✅ 稳定 | 同上 |
| `GoToRow()` | ✅ 稳定 | 同上 |
| `GoToFirstRow()` | ✅ 稳定 | 同上 |
| `GoToNextRow()` | ✅ 稳定 | 同上 |
| `IsClosed()` | ✅ 稳定 | 同上 |
| `Close()` | ✅ 稳定 | 同上 |

### 3. 云同步接口

#### 3.1 ICloudService 类

**头文件**：`interfaces/inner_api/cloud_data/include/icloud_service.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `UploadData()` | ⚠️ 不稳定（IPC 接口） | `icloud_service.h` |
| `DownloadData()` | ⚠️ 不稳定 | 同上 |
| `CleanDirtyData()` | ⚠️ 不稳定 | 同上 |
| `GetConfig()` | ⚠️ 不稳定 | 同上 |
| `ChangeCloudSwitch()` | ⚠️ 不稳定 | 同上 |

**稳定级别**：⚠️ **不稳定接口**（通过 IPC 访问，可能随服务端变化）

#### 3.2 CloudManager 类

**头文件**：`interfaces/inner_api/cloud_data/include/cloud_manager.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `Init()` | ✅ 稳定 | `cloud_manager.h` |
| `Release()` | ✅ 稳定 | 同上 |
| `DoSync()` | ✅ 稳定 | 同上 |

### 4. DataAbility 接口

#### 4.1 ISharedResultSet 类

**头文件**：`interfaces/inner_api/dataability/include/ishared_result_set.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetShareBlock()` | ✅ 稳定 | `ishared_result_set.h` |
| `GetBlock()` | ✅ 稳定 | 同上 |
| `GetRow()` | ✅ 稳定 | 同上 |
| `GetColumnNames()` | ✅ 稳定 | 同上 |
| `IsClosed()` | ✅ 稳定 | 同上 |
| `Close()` | ✅ 稳定 | 同上 |

#### 4.2 DataAbilityPredicates 类

**头文件**：`interfaces/inner_api/dataability/include/data_ability_predicates.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetRawSelection()` | ✅ 稳定 | `data_ability_predicates.h` |
| `GetSelectionArgs()` | ✅ 稳定 | 同上 |
| `GetOrder()` | ✅ 稳定 | 同上 |
| `GetGroup()` | ✅ 稳定 | 同上 |
| `GetLimit()` | ✅ 稳定 | 同上 |
| `GetDistinct()` | ✅ 稳定 | 同上 |

### 5. 公共类型接口

#### 5.1 ValueObject 类

**头文件**：`interfaces/inner_api/rdb/include/value_object.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetType()` | ✅ 稳定 | `value_object.h` |
| `GetInt()`, `GetLong()`, `GetDouble()` | ✅ 稳定 | 同上 |
| `GetString()`, `GetBlob()` | ✅ 稳定 | 同上 |
| `operator<<` (流输出) | ✅ 稳定 | 同上 |

#### 5.2 ValuesBucket 类

**头文件**：`interfaces/inner_api/rdb/include/values_bucket.h`

| 方法 | 稳定性 | 证据 |
|------|--------|------|
| `GetSize()` | ✅ 稳定 | `values_bucket.h` |
| `GetAllKeys()` | ✅ 稳定 | 同上 |
| `Get()` | ✅ 稳定 | 同上 |
| `Put()` | ✅ 稳定 | 同上 |
| `Delete()` | ✅ 稳定 | 同上 |
| `Clear()` | ✅ 稳定 | 同上 |

### 6. 适配器接口

#### 6.1 RdbDataShareAdapter

**头文件**：`interfaces/inner_api/rdb_data_share_adapter/include/rdb_utils.h`

| 函数 | 稳定性 | 证据 |
|------|--------|------|
| `RdbResultSetToDataShareResultSet()` | ✅ 稳定 | `rdb_utils.h` |
| `RdbPredicatesToDataSharePredicates()` | ✅ 稳定 | 同上 |
| `RdbValuesBucketToDataShareValuesBucket()` | ✅ 稳定 | 同上 |

#### 6.2 RdbDataAbilityAdapter

**头文件**：`interfaces/inner_api/rdb_data_ability_adapter/include/rdb_data_ability_utils.h`

| 函数 | 稳定性 | 证据 |
|------|--------|------|
| `ConvertPredicates()` | ✅ 稳定 | `rdb_data_ability_utils.h` |
| `ConvertResultSet()` | ✅ 稳定 | 同上 |

### 7. 依赖关系图

```
                    ┌──────────────────────┐
                    │  Inner API 消费者    │
                    └───────────┬──────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │Core RDB Impl │   │  Cloud Data  │   │  DataAbility  │
    └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
           │                    │                   │
           │              ┌─────────┬─────────┐   │
           │              │         │         │   │
           ▼              ▼         ▼         ▼   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │Inner RDB API │   │Inner Cloud   │   │Inner Data    │
    │( Interfaces) │   │  API         │   │Ability API    │
    └──────────────┘   └──────────────┘   └──────────────┘

使用者：
┌───────────────────┐   ┌──────────────┐   ┌──────────────┐
│JS NAPI Layer  │   │Data Share    │   │Data Ability  │
└───────────────────┘   └──────────────┘   └──────────────┘
```

**依赖说明**：
- **Core RDB Impl**：实现 Inner RDB API 接口（RdbStore、AbsRdbPredicates 等）
- **Cloud Data**：实现 Inner Cloud API 接口（CloudManager）
- **Inner RDB API**：被 JS NAPI、Cloud Data、DataAbility 使用
- **Inner Cloud API**：被 Cloud NAPI 使用
- **Inner DataAbility API**：被 DataAbility NAPI 使用

### 8. 接口可见性

| 接口 | 可见性 | 说明 | 证据 |
|------|--------|------|------|
| **RdbStore** | 公开（ohos_shared_library） | 所有 Inner API 模块 | `interfaces/inner_api/rdb/BUILD.gn:52` |
| **AbsSharedResultSet** | 公开 | DataAbility、DataShare | 同上 |
| **CloudManager** | 公开 | Cloud NAPI | 同上 |
| **cloud_data_inner** | 受限 | 仅 datamgr_service | `interfaces/inner_api/cloud_data/BUILD.gn:189` |

**可见性控制**：
- 大部分接口为公开，供子系统内使用
- 部分敏感接口（如 `cloud_data_inner`）限制可见性
- 避免暴露内部实现细节（如 `RdbStoreImpl`）

### 9. 可替换点

| 组件 | 可替换性 | 替换接口 | 证据 |
|------|----------|----------|------|
| **SQLite 引擎** | ✅ 可替换 | `sqlite_connection.cpp` 使用 SQLite 接口 | `sqlite_connection.h` |
| **加密模块** | ✅ 可替换 | `rdb_security_manager.cpp` 依赖 HUKS 接口 | `rdb_security_manager.h` |
| **连接池** | ⚠️ 部分可替换 | 连接池策略可调整 | `connection_pool.cpp` |
| **观察者管理** | ✅ 可替换 | 基于 `RdbStoreObserver` 接口 | `rdb_store.h:714-729` |

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构与模块职责
- [03_Architecture.md](./03_Architecture.md) - 架构与数据流
- [04_N-API.md](./04_N-API.md) - 对外 N-API 接口

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: interfaces/inner_api/ 中所有头文件
