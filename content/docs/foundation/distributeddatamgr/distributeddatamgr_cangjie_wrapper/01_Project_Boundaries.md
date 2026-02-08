# 项目定位与边界

## 文档目的

本文档明确 distributeddatamgr_cangjie_wrapper 的项目定位、边界范围和核心能力，帮助开发者理解项目的适用场景和限制。

## 适用范围

本文档适用于：
- 决定是否使用此封装层的开发者
- 理解项目功能边界的架构师
- 进行依赖分析的构建工程师

## 项目定位

### 核心定位

distributeddatamgr_cangjie_wrapper 是 **OpenHarmony 分布式数据管理系统的仓颉语言封装层**，其定位为：

> 为使用仓颉（Cangjie）语言进行应用开发的开发者提供数据持久化和跨设备数据协同能力

### 架构定位

在 OpenHarmony 系统架构中的位置：

```
┌─────────────────────────────────────────────────────────────────┐
│           应用层（Cangjie Apps）                           │
├─────────────────────────────────────────────────────────────────┤
│        本项目（Cangjie API Wrapper）                       │
│   - 提供仓颉语言友好的 API                        │
│   - 通过 FFI 调用底层系统服务                     │
├─────────────────────────────────────────────────────────────────┤
│      分布式数据管理子系统（C++ System Services）              │
│   - kv_store                                       │
│   - relational_store                                  │
│   - preferences                                      │
│   - data_share                                      │
├─────────────────────────────────────────────────────────────────┤
│         OpenHarmony 系统层                                 │
└─────────────────────────────────────────────────────────────────┘
```

**证据**: bundle.json:14 - `"subsystem": "distributeddatamgr"`

## 核心能力边界

### 1. 分布式键值数据库（DistributedKVStore）

#### 支持的功能

| 能力 | 说明 | 证据 |
|--------|------|------|
| 单设备 KV 操作 | put, get, delete, getEntries | distributed_kv_store.cj:51-449 |
| 批量操作 | putBatch, deleteBatch | single_kvstore.cj:67-113 |
| 事务支持 | startTransaction, commit, rollback | single_kvstore.cj:346-417 |
| 分布式同步 | sync, syncByQuery, enableSync | single_kvstore.cj:225-285 |
| 数据订阅 | onDataChange, onSyncComplete | single_kvstore.cj:299-326 |
| 备份恢复 | backup, restore, deleteBackup | single_kvstore.cj:189-224 |
| 查询能力 | Query 构造器（equalTo, like, orderBy 等） | query.cj:33-525 |

#### 不支持的功能

- 设备数据冲突的高级处理策略（需要依赖底层提供）
- 复杂的分布式事务协议（依赖底层）

### 2. 关系型数据库（RelationalStore）

#### 支持的功能

| 能力 | 说明 | 证据 |
|--------|------|------|
| 基本 CRUD | insert, update, delete, query | relational_store.cj:82-262 |
| SQL 执行 | executeSql, querySql | relational_store.cj:295-361 |
| 分布式表配置 | setDistributedTables, setDistributedTablesType | relational_store.cj:411-474 |
| 云同步 | cloudSync | relational_store.cj:482-518 |
| 结果集操作 | ResultSet 遍历、定位 | result_set.cj:31-819 |
| 备份恢复 | backUp, reStore | relational_store.cj:266-294 |

#### 不支持的功能（关键约束）

| 功能 | 约束 | 证据 |
|--------|------|------|
| **事务** | 与 ArkTS API 不同，不支持事务 API | README_zh.md:124-126 |
| DataAbility 集成 | 不支持 DataAbility Predicates | README_zh.md:112 |

#### 支持 SQL 操作

- ✅ 支持：**仅支持通过 SQL 语句创建和删除表**
- ❌ 不支持：复杂的 DDL 操作（如 ALTER TABLE、DROP INDEX 等）

**证据**: README_zh.md:43 - "Currently, only the creation and deletion of tables via SQL statements are supported."

### 3. 用户首选项（Preferences）

#### 支持的功能

| 能力 | 说明 | 证据 |
|--------|------|------|
| KV 操作 | put, get, delete, has | preferences.cj:80-358 |
| 批量操作 | getAll, clear, flush | preferences.cj:360-439 |
| 数据订阅 | on, off (change/multiProcessChange) | preferences.cj:238-330 |
| 多存储类型 | Xml, Gskv | preferences_options.cj:32-51 |

#### 存储类型边界

| 存储类型 | 特点 | 用途 |
|----------|------|------|
| Xml | XML 文件存储 | 通用场景 |
| Gskv | GSKV 格式存储 | 性能优化场景 |

**证据**: preferences_options.cj:32-51 - StorageType 枚举

### 4. 数据共享谓词（DataSharePredicates）

#### 支持的功能

| 能力 | 说明 | 证据 |
|--------|------|------|
| 比较操作 | equalTo | data_share_predicates.cj:79-95 |
| 逻辑操作 | and, or | data_share_predicates.cj:110-275 |
| 排序 | orderByAsc, orderByDesc | data_share_predicates.cj:132-173 |
| 分页 | limit(total, offset) | data_share_predicates.cj:189-200 |
| 范围查询 | inValues | data_share_predicates.cj:217-248 |
| 分组 | beginWrap, endWrap | data_share_predicates.cj:259-276 |

#### 不支持的功能

与完整的 DataShare Predicates 相比，当前仅支持：
- ✅ 等于条件
- ✅ AND 条件
- ✅ IN 条件
- ✅ 排序（升序/降序）
- ✅ 分页（Limit + Offset）

不支持（TODO 需确认）：
- ❌ 不等于条件
- ❌ 大于/小于/大于等于/小于等于
- ❌ 模糊查询（like/unlike）
- ❌ 空值判断

**注意**: 完整的谓词能力在底层的 RdbPredicates 中支持，但 DataSharePredicates 当前仅提供基础功能。

### 5. 数据集（Values Bucket）

#### 支持的类型

| 类型 | 仓颉类型 | 说明 |
|------|----------|------|
| Integer | Int64 | 整型值 |
| Double | Float64 | 双精度浮点 |
| StringValue | String | 字符串值 |
| Boolean | Bool | 布尔值 |

**证据**: values_bucket/value_type.cj:30-65 - VBValueType 枚举

#### 不支持的类型

与 ArkTS API 的 ValueBucket 相比，不支持：
- ❌ Blob（二进制大对象）
- ❌ Asset（资源对象）
- ❌ Assets（资源数组）

**证据**: values_bucket/value_type.cj:30-65 - 仅定义 4 种类型

## 运行环境边界

### 目标设备

| 设备类型 | 支持状态 | 证据 |
|----------|----------|------|
| Standard | ✅ 支持 | bundle.json:17-19 |
| Small | ❌ 不支持 | bundle.json:17-19 |
| Mini | ❌ 不支持 | bundle.json:17-19 |

### API Level

| API Level | 状态 | 证据 |
|-----------|--------|------|
| 22 | ✅ 当前支持 | 所有 @APILevel 注解 |

### 系统依赖边界

#### 必需依赖

| 依赖组件 | 用途 | 是否可选 |
|----------|------|---------|
| ability_cangjie_wrapper | 应用上下文 | ❌ 必需 |
| cangjie_ark_interop | 仓颉互操作 | ❌ 必需 |
| kv_store | KV 数据库服务 | ❌ 必需（DistributedKVStore） |
| relational_store | 关系型数据库服务 | ❌ 必需（RdbStore） |
| preferences | 首选项服务 | ❌ 必需（Preferences） |
| data_share | 数据共享服务 | ❌ 必需（DataSharePredicates） |
| hiviewdfx_cangjie_wrapper | HiLog 日志 | ❌ 必需 |

**证据**: bundle.json:22-31 - deps.components

#### 跨平台支持

| 平台 | 支持状态 | Mock 实现 |
|------|----------|-----------|
| Linux/Android | ✅ 支持 | - |
| Windows | ✅ 支持（Mock） | mock/*.cj |
| macOS | ✅ 支持（Mock） | mock/*.cj |

**证据**: 各模块的 BUILD.gn 中的条件编译（如 relational_store/BUILD.gn）

## 功能限制总结

### 硬约束

1. **目标设备限制**: 仅支持 Standard 设备
2. **RDB 事务限制**: 不支持事务 API（与 ArkTS 差异）
3. **SQL 限制**: 仅支持表的创建和删除

### 软约束

1. **数据大小限制**:
   - KV Store: MAX_KEY_LENGTH=1024, MAX_VALUE_LENGTH=4194303
   - Preferences: MAX_KEY_LENGTH=1024, MAX_VALUE_LENGTH=16MB

2. **查询限制**:
   - KV Store: MAX_QUERY_LENGTH=512000
   - 批量操作: MAX_BATCH_SIZE=128

**证据**:
- distributed_kv_store/distributed_kv_store_common.cj:41-86 - 常量定义
- preferences/preferences_options.cj:141-150 - 常量定义

## 与其他模块的关系

### 上游依赖

```
distributeddatamgr_cangjie_wrapper
    ├── ability_cangjie_wrapper (Ability 框架)
    ├── cangjie_ark_interop (仓颉运行时)
    ├── distributeddatamgr_kv_store (KV Store C++ 服务)
    ├── distributeddatamgr_relational_store (RDB C++ 服务)
    ├── distributeddatamgr_preferences (Preferences C++ 服务)
    ├── distributeddatamgr_data_share (DataShare C++ 服务)
    └── hiviewdfx_cangjie_wrapper (HiLog)
```

### 下游消费者

```
仓颉应用
    └── kit.ArkData
            └── distributeddatamgr_cangjie_wrapper
```

**证据**: kit/ArkData/index.cj:18-25 - public import 导出

## 适用场景

### 推荐使用场景

| 场景 | 推荐模块 | 说明 |
|--------|----------|------|
| 应用配置存储 | Preferences | 轻量级 Key-Value，简单快速 |
| 用户数据持久化 | RdbStore | 结构化数据，支持复杂查询 |
| 跨设备数据同步 | DistributedKVStore | 分布式 KV，自动同步 |
| 通用查询条件 | DataSharePredicates | 跨模块数据筛选 |
| 媒体文件元数据 | Values Bucket + DataSharePredicates | 相册、文件管理等 |

### 不推荐使用场景

| 场景 | 不推荐原因 | 替代方案 |
|--------|----------|----------|
| 大文件存储 | 不支持 Blob | 使用文件系统 API |
| 复杂事务处理 | RDB 不支持事务 | 使用 ArkTS API 或底层 C++ API |
| 实时数据流 | 不支持流式操作 | 使用其他 IPC 机制 |

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 模块组织
- [04_Public_API.md](04_Public_API.md) - 详细 API 参考
