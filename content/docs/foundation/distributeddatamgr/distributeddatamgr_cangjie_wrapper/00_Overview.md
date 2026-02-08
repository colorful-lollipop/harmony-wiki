# 项目概览

## 文档目的

本文档旨在提供 distributeddatamgr_cangjie_wrapper 项目的整体概览，帮助新人快速理解项目的定位、架构和核心概念。

## 适用范围

本文档覆盖：
- 项目背景与定位
- 核心概念与术语
- 技术选型与架构模式

## 项目定位

distributeddatamgr_cangjie_wrapper 是 OpenHarmony 系统的**分布式数据管理仓颉语言封装层**，为使用仓颉（Cangjie）语言开发应用的开发者提供以下能力：

1. **结构化数据持久化**
2. **跨设备数据同步**
3. **数据共享与查询**

### 目标设备

- **当前支持**: Standard 设备
- **未支持**: Small、Mini 等其他设备类型

证据: bundle.json:17-19 - `"adapted_system_type": ["standard"]`

### 技术选型

| 维度 | 选择 | 说明 |
|--------|------|------|
| 语言 | Cangjie（仓颉） | 华为自研编程语言 |
| 绑定方式 | FFI (Foreign Function Interface) | 直接调用底层 C++ API |
| 构建系统 | GN (Generate Ninja) | OpenHarmony 官方构建系统 |
| API Level | 22 | 对应 OpenHarmony API Level 22 |

## 核心能力

### 1. 分布式键值数据库（Distributed KV Store）

**定位**: 跨设备分布式数据协同能力，支持多设备间数据无缝同步与共享。

**核心特性**:
- 数据库管理（创建、关闭、删除）
- 数据操作（增删改查、批量操作）
- 事务支持（启用、提交、回滚）
- 数据订阅（订阅/取消订阅数据变更）
- 同步功能（端到端同步）
- 备份恢复（数据库备份、恢复、删除备份）

**证据**:
- ohos/data/distributed_kv_store/distributed_kv_store.cj:40-79 - DistributedKVStore 类定义
- ohos/data/distributed_kv_store/single_kvstore.cj:51-449 - SingleKVStore 类定义

### 2. 关系型数据库（Relational Store）

**定位**: 基于 SQLite 的完整本地关系型数据库管理机制。

**核心特性**:
- 数据库管理（创建、删除）
- 数据操作（增删改查）
- SQL 执行（直接执行自定义 SQL，支持表的创建和删除）
- 分布式表配置（设置分布式表和类型）
- 云同步（数据库云同步）
- 事务、备份、恢复

**约束**:
- **不支持事务**（与 ArkTS API 的区别）

**证据**:
- ohos/data/relational_store/rdb_store.cj:35 - RdbStore 类定义
- README_zh.md:143 - "RDB Store does not currently support the following functions: - Transactions"

### 3. 用户首选项（Preferences）

**定位**: 轻量级 Key-Value 数据处理能力，用于应用配置信息和用户偏好持久化。

**核心特性**:
- 数据库管理（获取、删除、缓存移除）
- 数据操作（写入、查找、删除）
- 数据订阅（基于 Key/Value 订阅数据变更）
- 批量操作（清空所有、获取所有、持久化）

**存储类型**:
- XML 存储类型
- GSKV 存储类型

**证据**:
- ohos/data/preferences/preferences.cj:80 - Preferences 类定义
- ohos/data/preferences/preferences_options.cj:32-51 - StorageType 枚举

### 4. 数据共享谓词（DataShare Predicates）

**定位**: 数据查询筛选能力，支持构建复杂查询条件。

**核心特性**:
- 比较谓词（等于）
- 逻辑谓词（与、或）
- 范围谓词（IN 条件）
- 排序功能（升序、降序）
- 分页查询（Limit + Offset）

**使用场景**:
- RDB 查询
- KVDB (schema) 查询
- 跨模块数据检索（如相册图片和视频检索）

**证据**:
- ohos/data/data_share_predicates/data_share_predicates.cj:46 - DataSharePredicates 类定义
- README_zh.md:18-22 - DataShare Predicates 功能描述

### 5. 数据集（Values Bucket）

**定位**: 标准化数据字段类型枚举类。

**支持类型**:
- Integer (Int64)
- Double (Float64)
- StringValue (String)
- Boolean (Bool)

**证据**:
- ohos/data/values_bucket/value_type.cj:30-65 - VBValueType 枚举定义

## 架构模式

### FFI 绑定模式

本项目采用 **FFI (Foreign Function Interface)** 模式与底层 C++ 系统服务交互：

```
┌─────────────────────────────────────────────────────────────────┐
│         Cangjie 应用层（开发者使用）                      │
├─────────────────────────────────────────────────────────────────┤
│  Public API Layer                                   │
│  - DistributedKVStore                                │
│  - RdbStore                                         │
│  - Preferences                                       │
│  - DataSharePredicates                               │
├─────────────────────────────────────────────────────────────────┤
│  FFI Layer (Foreign Function Interface)                  │
│  - foreign { FfiOHOS*() } 声明                      │
│  - @C struct 类型映射                                 │
├─────────────────────────────────────────────────────────────────┤
│  Native Layer (C++ System Services)                     │
│  - distributeddatamgr_kv_store                         │
│  - distributeddatamgr_relational_store                  │
│  - distributeddatamgr_preferences                        │
│  - distributeddatamgr_data_share                        │
└─────────────────────────────────────────────────────────────────┘
```

**证据**:
- ohos/data/distributed_kv_store/distributed_kv_store_ffi.cj:24-196 - foreign 块声明
- ohos/data/relational_store/relational_store_ffi.cj:26-228 - foreign 块声明

### 不使用 N-API

**重要发现**: 本项目**不使用**传统的 N-API (Node.js API) 绑定机制，而是通过仓颉语言的 FFI 直接调用底层 C++ API。

**原因**:
- 仓颉是独立的编程语言，不是 JavaScript 运行时
- FFI 提供了仓颉与 C/C++ 的原生互操作能力

**证据**:
- 后台搜索结果 "bg_fc9145df" - 未发现任何 napi_ 函数调用
- 所有 *_ffi.cj 文件使用 foreign 块声明外部函数

## 运行环境

### 资源占用

| 资源类型 | 占用量 | 来源 |
|----------|--------|------|
| ROM | 900KB | bundle.json:20 |
| RAM | 864KB | bundle.json:21 |

### 系统能力（Syscap）

所有模块声明了以下系统能力：

- `SystemCapability.DistributedDataManager.KVStore.Core`
- `SystemCapability.DistributedDataManager.KVStore.DistributedKVStore`
- `SystemCapability.DistributedDataManager.RelationalStore.Core`
- `SystemCapability.DistributedDataManager.Preferences.Core`
- `SystemCapability.DistributedDataManager.DataShare.Core`
- `SystemCapability.DistributedDataManager.CloudSync.Client`

**证据**: 各公共 API 类的 @APILevel 注解（如 distributed_kv_store.cj:42-44）

### 外部依赖

| 组件 | 用途 | 来源 |
|--------|------|------|
| ability_cangjie_wrapper | 应用上下文能力 | bundle.json:24 |
| cangjie_ark_interop | 仓颉注解和异常类 | bundle.json:25 |
| data_share | 数据共享组件 | bundle.json:26 |
| hiviewdfx_cangjie_wrapper | HiLog 日志能力 | bundle.json:27 |
| kv_store | KV 数据库组件 | bundle.json:28 |
| preferences | 首选项组件 | bundle.json:29 |
| relational_store | 关系型数据库组件 | bundle.json:30 |

## 与 ArkTS API 的对比

### 不支持的功能

与 ArkTS API 相比，以下功能暂不支持：

1. Common Data Types（通用数据类型）
2. DataAbility Predicates（DataAbility 谓词）
3. DataShare（数据共享）
4. Distributed Data Object（分布式数据对象）
5. Shared User Preferences（共享用户首选项）
6. Shared RDB Store（共享关系型数据库）
7. Unified Data Channel（统一数据通道）
8. Uniform Data Structs（统一数据结构）
9. Uniform Data Definition and Description（统一数据定义与描述）
10. ArkData Intelligence Platform（智慧数据平台）
11. Device-Cloud Service（端云服务）

**证据**: README_zh.md:111-122 - 约束章节

## 关键概念速查

| 概念 | 说明 |
|--------|------|
| RemoteDataLite | 所有数据管理类的基类，提供资源生命周期管理 |
| StageContext | Ability 上下文，用于访问应用资源 |
| BaseContext | 基础上下文类型 |
| Security Level | 数据安全级别（S1-S4），控制数据加密强度 |
| Schema | KV Store 的数据结构定义，指定字段和类型 |

## 相关跳转

- [01_Project_Boundaries.md](01_Project_Boundaries.md) - 详细的项目边界和核心能力说明
- [02_Directory_Structure.md](02_Directory_Structure.md) - 代码组织和模块职责
- [03_Architecture.md](03_Architecture.md) - 详细的架构设计和数据流
- [04_Public_API.md](04_Public_API.md) - 完整的对外 API 清单
