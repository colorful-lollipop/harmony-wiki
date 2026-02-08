# 目录结构与模块职责

## 目的

本文档提供 relational_store 项目的完整目录结构（不含测试），并说明各模块的职责、边界和依赖关系。

## 适用范围

- relational_store 源代码目录结构
- 模块职责划分
- 文件组织方式
- 依赖关系（排除测试目录）

## 关键结论

### 1. 顶层目录树

```
relational_store/
├── frameworks/                    # 框架层实现
│   ├── native/                  # 原生 C++ 实现
│   ├── js/                      # JavaScript/TypeScript NAPI 绑定
│   ├── ets/                     # TypeScript ETS 绑定
│   ├── cj/                      # 仓颉语言 FFI 绑定
│   └── common/                  # 公共头文件
├── interfaces/                   # 接口层
│   ├── inner_api/               # 内部 API（供其他模块使用）
│   ├── ndk/                    # NDK 公共 C API
│   └── rdb_ndk_utils/          # NDK 工具库
└── conf/                        # 配置文件
    ├── trusts_config.json       # 信任列表配置
    └── silentproxy_config.json # 静默代理配置
```

### 2. Frameworks 模块详解

#### 2.1 frameworks/native/ - 原生实现层

| 子目录 | 职责 | 关键文件 | 依赖 |
|--------|------|----------|------|
| **rdb/** | 核心 RDB 实现 | `rdb_store_impl.cpp`<br/>`rdb_helper.cpp`<br/>`connection_pool.cpp` | sqlite, ipc, huks, icu |
| **cloud_data/** | 云同步管理 | `cloud_manager.cpp`<br/>`cloud_service_proxy.cpp` | kv_store, device_manager, ipc |
| **dataability/** | DataAbility 适配 | `ishared_result_set.cpp`<br/>`data_ability_predicates.cpp` | data_ability |
| **rdb_crypt/** | 加密模块 | 加密密钥管理<br/>HUKS 集成 | huks |
| **icu/** | ICU 支持 | Unicode 排序<br/>文本处理 | icu (系统) |
| **obs_mgr_adapter/** | 观察者适配器 | 观察者管理<br/>生命周期 | ability_runtime |
| **appdatafwk/** | 应用数据框架 | `shared_block.cpp`<br/>`serializable.h` | 无 |
| **dfx/** | 诊断与追踪 | 日志、性能统计、错误追踪 | hilog, hitrace |

**核心 RDB 实现文件列表**（`frameworks/native/rdb/src/`）：

| 文件 | 职责 |
|------|------|
| `rdb_store_impl.cpp` | RdbStore 核心实现，CRUD 操作 |
| `rdb_store_manager.cpp` | 存储管理器，实例管理 |
| `connection_pool.cpp` | 连接池，管理最多 4 个 SQLite 连接 |
| `sqlite_connection.cpp` | SQLite 连接封装，数据库打开/关闭 |
| `rdb_helper.cpp` | RdbHelper 实现，数据库初始化入口 |
| `rdb_predicates.cpp` | 查询谓词实现 |
| `sqlite_sql_builder.cpp` | SQL 语句构建器 |
| `step_result_set.cpp` | 分步结果集实现（大型结果集） |
| `transaction_impl.cpp` | 事务实现 |
| `value_object.cpp` | 值对象封装 |
| `values_bucket.cpp` | 值桶封装（键值对） |
| `rdb_security_manager.cpp` | 加密密钥管理 |
| `security_policy.cpp` | 安全策略实现（S1-S4 级别） |
| `rdb_service_proxy.cpp` | RDB 服务 IPC 代理 |
| `rdb_manager_impl.cpp` | 系统能力管理实现 |
| `rdb_notifier_stub.cpp` | 数据变更通知存根 |
| `result_set_proxy.cpp` | 结果集 IPC 代理 |

#### 2.2 frameworks/js/ - JavaScript NAPI 绑定层

| 子目录 | 导出模块名 | 入口文件 | 职责 |
|--------|-----------|----------|------|
| **rdb/** | `data.rdb` | `entry_point.cpp` | 旧版 RDB NAPI 绑定 |
| **relationalstore/** | `data.relationalStore` | `entry_point.cpp` | 新版 RDB NAPI 绑定 |
| **cloud_data/** | `data.cloudData` | `entry_point.cpp` | 云数据 NAPI 绑定 |
| **dataability/** | `data.dataability` | `entry_point.cpp` | DataAbility NAPI 绑定 |
| **cloud_extension/** | `data.cloudExtension` | `cloud_extension.cpp` | 云扩展 NAPI 绑定 |
| **sendablerelationalstore/** | (待确认) | `entry_point.cpp` | 可发送关系型存储 NAPI |
| **common/** | 公共工具 | N/A | JS 工具函数、类型转换 |
| **ani/relationalstore/** | N/A | `BUILD.gn` | ANI（Ark Native Interface）绑定 |

#### 2.3 frameworks/ets/ - TypeScript ETS 绑定层

| 子目录 | 职责 | 说明 |
|--------|------|------|
| **taihe/relationalstore/** | RDB Taihe 绑定 | ArkTS 原生接口 |
| **taihe/cloud_data/** | Cloud Data Taihe 绑定 | 云数据 ArkTS 接口 |
| **cloud_extension/** | Cloud Extension 绑定 | 云扩展 ArkTS 接口 |

#### 2.4 frameworks/cj/ - 仓颉 FFI 绑定层

| 文件 | 职责 |
|------|------|
| `relational_store_ffi.h` | FFI 接口定义 |
| `relational_store_ffi.cpp` | 仓颉语言 FFI 实现 |

### 3. Interfaces 模块详解

#### 3.1 interfaces/inner_api/ - 内部 API 层

**供其他子系统使用的接口**（如 data_share, datamgr_service）：

| 子目录 | 职责 | 关键头文件 | 使用者 |
|--------|------|-----------|--------|
| **rdb/** | 核心 RDB 接口 | `rdb_store.h`<br/>`rdb_helper.h`<br/>`rdb_predicates.h`<br/>`value_object.h`<br/>`values_bucket.h` | JS NAPI, Cloud Data, DataAbility |
| **cloud_data/** | 云数据接口 | `cloud_manager.h`<br/>`cloud_service.h`<br/>`cloud_types.h` | 云同步服务 |
| **dataability/** | DataAbility 接口 | `ishared_result_set.h`<br/>`data_ability_predicates.h` | DataAbility 框架 |
| **appdatafwk/** | 应用数据框架 | `shared_block.h`<br/>`serializable.h` | 各数据子系统 |
| **common_type/** | 公共类型 | `common_types.h` | 所有模块 |
| **rdb_data_share_adapter/** | DataShare 适配 | `rdb_utils.h` | DataShare 子系统 |
| **rdb_data_ability_adapter/** | DataAbility 适配 | `rdb_data_ability_utils.h` | DataAbility 框架 |

**接口可见性**：
- 大部分为 `ohos_shared_library`，默认对所有模块可见
- `cloud_data_inner` 标注 `visibility = ["datamgr_service"]`，限制可见性

#### 3.2 interfaces/ndk/ - NDK 公共 C API

**供 NDK 开发者使用的 C 接口**：

| 文件 | 说明 | API 类别 |
|------|------|---------|
| `relational_store.h` | 主要 NDK API（71KB，完整） | 数据库操作、事务、同步 |
| `oh_predicates.h` | 查询谓词接口 | Predicates 类定义 |
| `oh_cursor.h` | 结果游标接口 | Cursor 类定义 |
| `oh_values_bucket.h` | 值桶接口 | ValuesBucket 类定义 |
| `oh_rdb_transaction.h` | 事务接口 | Transaction 类定义 |
| `relational_store_error_code.h` | 错误码定义 | RDB_ERR_* 常量 |
| `oh_data_values.h` | 数据值接口 | DataValues 类定义 |
| `oh_data_value.h` | 数据值接口（单个） | DataValue 类定义 |
| `oh_value_object.h` | 值对象接口 | ValueObject 类定义 |
| `oh_rdb_crypto_param.h` | 加密参数接口 | CryptoParam 类定义 |
| `data_asset.h` | 数据资产接口 | Asset 类定义 |

#### 3.3 interfaces/rdb_ndk_utils/ - NDK 工具库

| 文件 | 说明 |
|------|------|
| `rdb_ndk_utils.h` | NDK 工具函数声明 |
| `rdb_ndk_utils.cpp` | 工具函数实现（Bundle 验证等） |

### 4. Conf 模块详解

#### 4.1 配置文件

| 文件 | 安装路径 | 内容 | 用途 |
|------|----------|------|------|
| `trusts_config.json` | `/system/etc/trusts/conf/trusts_config.json` | 信任 Bundle 列表（8 个系统应用） |
| `silentproxy_config.json` | `/system/etc/silent/conf/silentproxy_config.json` | 静默代理配置 |

**信任列表内容**（`trusts_config.json`）：
```json
[
  {"bundleName": "com.ohos.calendardata", "storeNames": ["calendardata"]},
  {"bundleName": "com.ohos.camera", "storeNames": ["HmosCamera"]},
  {"bundleName": "com.ohos.photos", "storeNames": ["El2PhotosData"]},
  {"bundleName": "com.ohos.security.privacycenter", "storeNames": ["access"]},
  {"bundleName": "com.ohos.contactsdataability", "storeNames": ["contacts", "calls", "callsEl1", "contacts"]},
  {"bundleName": "com.ohos.ringtonelibrary.ringtonelibrarydata", "storeNames": ["ringtone_library"]},
  {"bundleName": "com.ohos.settingsdata", "storeNames": ["settingsdata"]},
  {"bundleName": "com.ohos.telephonydataability", "storeNames": ["sms_mms"]}
]
```

### 5. 依赖方向图

```
                    ┌────────────────────────────┐
                    │   JS/TS/ETS 应用层       │
                    └───────────┬────────────┘
                                │
                ┌───────────────────┼───────────────────┐
                │                   │                   │
                ▼                   ▼                   ▼
    ┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
    │  NAPI 绑定层      │   │  Inner API 接口层   │   │   NDK 公共 API     │
    │  (frameworks/js/) │   │  (interfaces/inner_api/)│   │  (interfaces/ndk/)  │
    └───────────┬───────┘   └───────────┬───────┘   └───────────────────┘
                │                       │
                │                       │
                ▼                       ▼
    ┌─────────────────────────────────────────────┐
    │        Core RDB 实现               │
    │    (frameworks/native/rdb/)          │
    └───────────┬───────────────────────────┘
                  │
      ┌───────────┼───────────────────┐
      │           │                   │
      ▼           ▼                   ▼
┌──────────────┐  ┌──────────────┐   ┌──────────────┐
│  SQLite      │  │  IPC        │   │  HUKS        │
│  引擎        │  │  通信        │   │  加密         │
└──────────────┘  └──────────────┘   └──────────────┘
```

**依赖说明**：
- **JS/TS/ETS 层**：通过 NAPI 调用
- **NAPI 层**：调用 Inner API 接口
- **Inner API 层**：调用 Core RDB 实现
- **Core RDB**：调用 SQLite、IPC、HUKS
- **NDK 层**：直接调用 Inner API 接口

### 6. 模块职责总结

| 层级 | 模块 | 核心职责 | 稳定性 |
|------|------|----------|--------|
| **应用层** | JS/TS/ETS 应用 | 数据消费 | 稳定（N-API 接口） |
| **绑定层** | NAPI/NDK | 语言桥接 | 稳定（导出接口） |
| **接口层** | Inner API | 抽象与契约 | 中等（部分接口可见性受限） |
| **实现层** | Native RDB | 核心逻辑 | 稳定（内部接口） |
| **存储层** | SQLite | 数据持久化 | 稳定（外部依赖） |

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 架构图与数据流
- [05_Inner_API.md](./05_Inner_API.md) - 内部接口详细说明
- [06_GN_Build.md](./06_GN_Build.md) - 构建目标与依赖

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: 目录结构扫描、README_zh.md、bundle.json、各模块头文件
