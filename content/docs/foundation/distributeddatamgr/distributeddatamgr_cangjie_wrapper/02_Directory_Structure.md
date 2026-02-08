# 目录结构与模块职责

## 文档目的

本文档详细说明 distributeddatamgr_cangjie_wrapper 的代码组织结构和各模块职责，帮助开发者快速定位代码。

## 适用范围

本文档覆盖：
- 完整目录树（不含测试）
- 各模块职责说明
- 模块间依赖关系

## 顶层目录结构

```
distributeddatamgr_cangjie_wrapper/
├── figures/                           # 架构图
│   └── distributeddatamgr_cangjie_wrapper_architecture.png
├── kit/                              # Kit 化代码（聚合入口）
│   └── ArkData/
│       ├── BUILD.gn
│       └── index.cj               # Kit 入口，导出所有模块
├── ohos/                             # 仓颉分布式数据管理接口实现
│   ├── data/
│   │   ├── BUILD.gn
│   │   ├── data.cj                # 空包，命名空间定义
│   │   ├── data_share_predicates/    # 数据共享谓词模块
│   │   │   ├── BUILD.gn
│   │   │   ├── data_share_predicates.cj
│   │   │   ├── data_share_predicates_common.cj
│   │   │   └── data_share_predicates_ffi.cj
│   │   ├── distributed_kv_store/     # 分布式键值数据库模块
│   │   │   ├── BUILD.gn
│   │   │   ├── distributed_kv_store.cj
│   │   │   ├── distributed_kv_store_common.cj
│   │   │   ├── distributed_kv_store_ffi.cj
│   │   │   ├── device_kvstore.cj
│   │   │   ├── kvstore_result_set.cj
│   │   │   ├── query.cj
│   │   │   └── single_kvstore.cj
│   │   ├── preferences/            # 用户首选项模块
│   │   │   ├── BUILD.gn
│   │   │   ├── preferences.cj
│   │   │   ├── preferences_options.cj
│   │   │   └── preferences_ffi.cj
│   │   ├── relational_store/       # 关系型数据库模块
│   │   │   ├── BUILD.gn
│   │   │   ├── relational_store.cj
│   │   │   ├── relational_store_common.cj
│   │   │   ├── relational_store_ffi.cj
│   │   │   ├── rdb_predicates.cj
│   │   │   └── result_set.cj
│   │   └── values_bucket/        # 数据集模块
│   │       ├── BUILD.gn
│   │       └── value_type.cj
├── mock/                             # Mock 实现（用于测试）
│   ├── ohos.data.distributed_kv_store.cj
│   ├── ohos.data.relational_store.cj
│   ├── ohos.data.data_share_predicates.cj
│   └── ohos.data.values_bucket.cj
├── test/                             # 测试用例（不含在本文档中）
├── bundle.json                        # 组件配置
├── BUILD.gn                           # 根构建配置
├── README.md
├── README_zh.md
├── LICENSE
└── OAT.xml
```

## 模块详细说明

### 1. kit.ArkData 聚合模块

**路径**: `kit/ArkData/`

**职责**:
- 作为统一的 Kit 入口，聚合导出所有数据管理模块
- 提供便捷的包导入方式

**文件结构**:
```
kit/ArkData/
├── BUILD.gn                   # 定义 kit.ArkData target
└── index.cj                   # 导出所有公共 API
```

**关键代码**: kit/ArkData/index.cj:20-24
```cangjie
public import ohos.data.*
public import ohos.data.data_share_predicates.*
public import ohos.data.distributed_kv_store.*
public import ohos.data.preferences.*
public import ohos.data.relational_store.*
public import ohos.data.values_bucket.*
```

**依赖**: 依赖所有 `ohos.data.*` 子模块

### 2. ohos.data 基础模块

**路径**: `ohos/data/`

**职责**:
- 提供基础命名空间 `ohos.data`
- 作为所有数据管理模块的父包

**文件结构**:
```
ohos/data/
├── BUILD.gn
└── data.cj                    # 空包，仅 package 声明
```

**关键代码**: ohos/data/data.cj:18
```cangjie
package ohos.data
```

**依赖**: 无外部依赖

### 3. data_share_predicates 数据共享谓词模块

**路径**: `ohos/data/data_share_predicates/`

**职责**:
- 提供数据查询筛选能力
- 支持构建复杂查询条件（比较、逻辑、范围、排序、分页）

**文件结构**:
```
data_share_predicates/
├── BUILD.gn
├── data_share_predicates.cj           # DataSharePredicates 类定义
├── data_share_predicates_common.cj     # 公共常量和日志
└── data_share_predicates_ffi.cj         # FFI 外部函数声明
```

**类结构**:

| 文件 | 类/类型 | 职责 |
|------|----------|--------|
| data_share_predicates.cj | DataSharePredicates | 查询谓词主类，提供 equalTo, and, orderByAsc 等方法 |
| data_share_predicates_common.cj | LOG, ERR_PARAMETER_ERROR | 公共常量和错误定义 |
| data_share_predicates_ffi.cj | FfiOHOSDataSharePredicates* | FFI 声明，约 8 个外部函数 |

**依赖关系**:
- 外部依赖: `values_bucket:ohos.data.values_bucket`
- C++ 依赖: `data_share:cj_data_share_predicates_ffi`

**证据**: data_share_predicates.cj:20 - `import ohos.data.values_bucket.{VBValueType, CValueType}`

### 4. distributed_kv_store 分布式键值数据库模块

**路径**: `ohos/data/distributed_kv_store/`

**职责**:
- 提供跨设备分布式数据协同能力
- 支持多设备间数据无缝同步与共享

**文件结构**:
```
distributed_kv_store/
├── BUILD.gn
├── distributed_kv_store.cj               # DistributedKVStore 类定义
├── distributed_kv_store_common.cj         # 常量、枚举、配置类
├── distributed_kv_store_ffi.cj           # FFI 外部函数声明
├── device_kvstore.cj                    # DeviceKVStore 类定义
├── kvstore_result_set.cj               # KVStoreResultSet 类定义
├── query.cj                            # Query 类定义
└── single_kvstore.cj                    # SingleKVStore 类定义
```

**类结构**:

| 文件 | 类/类型 | 职责 | 行号范围 |
|------|----------|--------|---------|
| distributed_kv_store.cj | DistributedKVStore, KVManager | 分布式 KV 管理器和 KV 管理器 | 40-79 |
| distributed_kv_store_common.cj | Constants, KVValueType, KVManagerConfig, KVSecurityLevel, FieldNode, Schema, KVOptions, Entry | 公共类型定义 | 33-536 |
| distributed_kv_store_ffi.cj | foreign 函数, @C struct | FFI 声明和 C 结构体映射 | 24-470 |
| device_kvstore.cj | DeviceKVStore | 设备 KV 存储类 | 38-263 |
| kvstore_result_set.cj | KVStoreResultSet | KV 结果集类 | 33-277 |
| query.cj | Query | 查询构造器类 | 33-525 |
| single_kvstore.cj | SingleKVStore, KVStoreEvent, SyncMode, SubscribeType, ChangeNotification | 单设备 KV 存储类和相关枚举 | 29-495 |

**依赖关系**:
- Cangjie 依赖: `data_share_predicates:ohos.data.data_share_predicates`
- C++ 依赖: `kv_store:cj_distributed_kv_store_ffi`
- 外部依赖: ability_cangjie_wrapper, hiviewdfx_cangjie_wrapper, cangjie_ark_interop

### 5. preferences 用户首选项模块

**路径**: `ohos/data/preferences/`

**职责**:
- 提供轻量级 Key-Value 数据处理能力
- 用于应用配置信息和用户偏好的持久化存储

**文件结构**:
```
preferences/
├── BUILD.gn
├── preferences.cj                      # Preferences 类定义
├── preferences_options.cj                # PreferencesOptions 类和枚举
└── preferences_ffi.cj                    # FFI 外部函数声明
```

**类结构**:

| 文件 | 类/类型 | 职责 | 行号范围 |
|------|----------|--------|---------|
| preferences.cj | Preferences, PreferencesEvent | Preferences 主类和事件枚举 | 40-396 |
| preferences_options.cj | PreferencesOptions, StorageType, PreferencesValueType | 选项类和类型枚举 | 32-272 |
| preferences_ffi.cj | FfiOHOSPreferences* | FFI 声明，约 12 个外部函数 | 24-213 |

**依赖关系**:
- C++ 依赖: `preferences:cj_preferences_ffi`
- 外部依赖: ability_cangjie_wrapper, hiviewdfx_cangjie_wrapper, cangjie_ark_interop

### 6. relational_store 关系型数据库模块

**路径**: `ohos/data/relational_store/`

**职责**:
- 提供基于 SQLite 的完整本地数据库管理机制
- 支持标准关系型数据模型

**文件结构**:
```
relational_store/
├── BUILD.gn
├── relational_store.cj                 # RdbStore 类定义
├── relational_store_common.cj           # 常量、枚举、配置类
├── relational_store_ffi.cj               # FFI 外部函数声明
├── rdb_predicates.cj                    # RdbPredicates 类定义
└── result_set.cj                        # ResultSet 类定义
```

**类结构**:

| 文件 | 类/类型 | 职责 | 行号范围 |
|------|----------|--------|---------|
| relational_store.cj | RdbStore | RDB 存储主类 | 35-538 |
| relational_store_common.cj | CryptoParam, StoreConfig, Asset, RelationalStoreValueType, 等 | 大量公共类型定义（>1500 行） | 30-1490 |
| relational_store_ffi.cj | FfiOHOSRelationalStore* | FFI 声明，约 70+ 个外部函数 | 26-686 |
| rdb_predicates.cj | RdbPredicates | RDB 查询谓词类 | 31-485 |
| result_set.cj | ResultSet | 结果集类 | 31-1496 |

**依赖关系**:
- C++ 依赖: `relational_store:cj_relational_store_ffi`
- 外部依赖: ability_cangjie_wrapper, hiviewdfx_cangjie_wrapper, cangjie_ark_interop

### 7. values_bucket 数据集模块

**路径**: `ohos/data/values_bucket/`

**职责**:
- 提供标准化的数据字段类型枚举类
- 作为其他模块的公共类型依赖

**文件结构**:
```
values_bucket/
├── BUILD.gn
└── value_type.cj                      # VBValueType 枚举和 CValueType 结构体
```

**类结构**:

| 文件 | 类/类型 | 职责 |
|------|----------|--------|
| value_type.cj | VBValueType, CValueType | 值类型枚举和 C 结构体映射 |

**依赖关系**:
- 无 Cangjie 依赖（被其他模块依赖）
- 外部依赖: cangjie_ark_interop

**证据**: value_type.cj:20 - 仅 import `ohos.ffi.CTypeResource` 和 `ohos.labels.APILevel`

### 8. mock 测试桩模块

**路径**: `mock/`

**职责**:
- 为非 OpenHarmony 平台（Windows/macOS）提供 Mock 实现
- 支持开发阶段的测试和调试

**文件结构**:
```
mock/
├── ohos.data.distributed_kv_store.cj
├── ohos.data.relational_store.cj
├── ohos.data.data_share_predicates.cj
└── ohos.data.values_bucket.cj
```

**Mock 实现模式**:
- 提供与真实 API 兼容的接口
- 使用空实现或内存数据存储
- 通过编译条件 `is_mingw || is_mac` 启用

**证据**: 各模块的 BUILD.gn 中的条件编译（如 relational_store/BUILD.gn）

## 模块依赖关系图

```
kit.ArkData (聚合层)
    ├── ohos.data (基础命名空间)
    ├── ohos.data.data_share_predicates
    │       └── ohos.data.values_bucket
    ├── ohos.data.distributed_kv_store
    │       └── ohos.data.data_share_predicates
    │               └── ohos.data.values_bucket
    ├── ohos.data.preferences
    └── ohos.data.relational_store

外部 C++ 依赖:
    ├── data_share (cj_data_share_predicates_ffi)
    ├── kv_store (cj_distributed_kv_store_ffi)
    ├── preferences (cj_preferences_ffi)
    └── relational_store (cj_relational_store_ffi)

通用依赖:
    ├── ability_cangjie_wrapper (应用上下文)
    ├── cangjie_ark_interop (仓颉互操作)
    └── hiviewdfx_cangjie_wrapper (HiLog 日志)
```

## 文件大小与复杂度

| 模块 | 源文件数 | 总行数（约） | 复杂度 |
|--------|-----------|-------------|--------|
| data_share_predicates | 3 | ~340 | 低 |
| values_bucket | 1 | ~120 | 低 |
| preferences | 3 | ~420 | 中 |
| distributed_kv_store | 7 | ~2300 | 高 |
| relational_store | 5 | ~4700 | 高 |
| kit.ArkData | 1 | ~25 | 低 |
| ohos.data | 1 | ~18 | 低 |
| **总计** | **21** | **~8023** | - |

**证据**: 使用 `wc -l` 和 `ls` 命令统计

## 关键常量定义位置

| 常量类型 | 定义位置 | 说明 |
|----------|----------|------|
| KV Store 常量 | distributed_kv_store_common.cj:33-89 | MAX_KEY_LENGTH, MAX_VALUE_LENGTH 等 |
| Preferences 常量 | preferences_options.cj:141-150 | MAX_KEY_LENGTH, MAX_VALUE_LENGTH |
| RDB 错误码 | relational_store_common.cj:30-59 | E_BASE, E_SQLITE_* 等 |
| Preferences 错误码 | preferences_options.cj:155-199 | ERROR_BASE, E_INNER_ERROR 等 |

## 模块职责总结

| 模块 | 核心职责 | 主要类 | FFI 函数数 |
|--------|----------|--------|------------|
| DataSharePredicates | 数据查询筛选 | DataSharePredicates | ~8 |
| DistributedKVStore | 分布式 KV 存储 | KVManager, SingleKVStore, DeviceKVStore, Query | ~50 |
| Preferences | 用户首选项 | Preferences | ~12 |
| RelationalStore | 关系型数据库 | RdbStore, RdbPredicates, ResultSet | ~70 |
| ValuesBucket | 数据类型枚举 | VBValueType | 0 (无 FFI) |

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构设计和数据流
- [05_Internal_API.md](05_Internal_API.md) - FFI 层详细实现
- [06_GN_Targets.md](06_GN_Targets.md) - 构建配置和依赖
