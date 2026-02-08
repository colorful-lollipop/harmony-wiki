# 目录结构与模块职责

## 目的

本文档描述 ASSET 服务的目录结构、各目录/模块的职责和边界（不含测试）。

## 适用范围

- 涵盖内容：顶层目录树、各模块职责、模块边界
- 排除内容：`test/` 目录

---

## 完整目录树（不含测试）

```
/Volumes/lexar/code/d/work/oh/base/security/asset/
├── BUILD.gn                          # 根构建配置
├── Cargo.toml                        # Rust 工作空间配置
├── bundle.json                       # OpenHarmony 组件清单
├── config.gni                        # 构建配置变量
├── hisysevent.yaml                   # 系统事件日志配置
├── rustfmt.toml                      # Rust 格式化配置
├── README.md / README_zh.md          # 项目文档
├── LICENSE                           # Apache 2.0 许可证
├── OAT.xml                           # OSS 归属文件
├── figures/                          # 架构图
├── etc/                              # 系统配置
│   └── init/
│       ├── BUILD.gn
│       └── asset_service.cfg         # 服务启动配置
├── sa_profile/                        # 系统能力配置
│   ├── BUILD.gn
│   └── 8100.json                     # SA 配置（SA_ID=8100）
├── frameworks/                         # 框架层（客户端库）
│   ├── definition/                   # 核心数据类型定义
│   │   └── src/
│   │       ├── lib.rs                # Tag, Value, AssetMap, ErrCode 等
│   │       ├── macros.rs             # 过程宏
│   │       ├── macros_lib.rs         # 宏工具
│   │       └── extension.rs          # Trait 扩展
│   ├── ipc/                          # 进程间通信
│   │   └── src/
│   │       └── lib.rs                # SA_ID=8100, IpcCode, 序列化
│   ├── utils/                        # 工具函数
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── time.rs
│   │       └── hasher.rs
│   ├── os_dependency/                # OS 抽象层
│   │   ├── memory/                   # 内存管理
│   │   │   ├── inc/asset_mem.h
│   │   │   └── src/asset_mem.c
│   │   ├── file/                     # 文件操作（DE/CE 目录）
│   │   │   └── src/
│   │   │       ├── lib.rs
│   │   │       ├── de_operator.rs
│   │   │       ├── ce_operator.rs
│   │   │       └── common.rs
│   │   ├── log/                      # 日志基础设施
│   │   │   ├── inc/asset_log.h
│   │   │   └── src/lib.rs
│   │   └── openssl/                  # OpenSSL 包装器
│   │       ├── inc/openssl_wrapper.h
│   │       └── src/openssl_wrapper.c
│   ├── js/napi/                      # JavaScript N-API 绑定
│   │   ├── inc/
│   │   │   ├── asset_napi_common.h
│   │   │   ├── asset_napi_add.h
│   │   │   ├── asset_napi_query.h
│   │   │   ├── asset_napi_remove.h
│   │   │   ├── asset_napi_update.h
│   │   │   ├── asset_napi_pre_query.h
│   │   │   ├── asset_napi_post_query.h
│   │   │   ├── asset_napi_query_sync_result.h
│   │   │   ├── asset_napi_error_code.h
│   │   │   ├── asset_napi_context.h
│   │   │   └── asset_napi_check.h
│   │   └── src/
│   │       ├── asset_napi.cpp        # N-API 注册入口
│   │       ├── asset_napi_add.cpp
│   │       ├── asset_napi_query.cpp
│   │       ├── asset_napi_remove.cpp
│   │       ├── asset_napi_update.cpp
│   │       ├── asset_napi_pre_query.cpp
│   │       ├── asset_napi_post_query.cpp
│   │       ├── asset_napi_query_sync_result.cpp
│   │       ├── asset_napi_common.cpp    # 工具函数
│   │       ├── asset_napi_context.cpp   # 异步上下文
│   │       └── asset_napi_check.cpp      # 参数验证
│   └── c/system_api/                 # C 系统 API 实现
│       └── src/
│           └── asset_system_api.c    # AssetAdd/Remove/Update/Query 等
├── interfaces/                         # 公共 API 边界
│   ├── inner_kits/                   # 内部 API（系统组件用）
│   │   ├── rs/                       # Rust SDK
│   │   │   └── src/
│   │   │       ├── lib.rs            # AssetManager, IPC 代理
│   │   │       └── timeout_feature.c  # 超时特性
│   │   ├── c/                        # C 内部 API
│   │   │   ├── inc/
│   │   │   │   ├── asset_system_api.h  # Asset* 函数
│   │   │   │   └── asset_system_type.h  # SEC_ASSET_* 类型
│   │   │   └── src/
│   │   │       └── lib.rs            # C FFI 实现
│   │   └── plugin_interface/         # 插件接口
│   │       └── src/
│   │           ├── lib.rs
│   │           └── plugin_interface.rs  # IAssetPlugin trait
│   └── kits/c/                       # 公共 NDK（第三方应用用）
│       ├── inc/
│       │   ├── asset_api.h           # OH_Asset_* 函数
│       │   └── asset_type.h          # ASSET_* 类型和标签
│       └── src/
│           └── asset_api.c           # 包装器，调用 system_api
└── services/                           # 服务层代码（服务端）
    ├── core_service/                 # 主 Asset 服务实现
    │   └── src/
    │       ├── lib.rs                # AssetAbility, SA 注册
    │       ├── stub.rs               # IPC stub 实现
    │       ├── sys_event.rs          # 系统事件上报
    │       ├── trace_scope.rs        # 性能追踪
    │       ├── data_size_mod.rs      # 数据大小监控
    │       ├── upgrade_ce.rs         # CE 升级
    │       ├── upgrade_operator.rs   # 数据库升级操作
    │       ├── common_event.rs       # 常用事件处理
    │       │   ├── listener.rs
    │       │   └── start_event.rs
    │       └── operations/           # CRUD 操作实现
    │           ├── operation_add.rs
    │           ├── operation_remove.rs
    │           ├── operation_update.rs
    │           ├── operation_query.rs
    │           ├── operation_pre_query.rs
    │           ├── operation_post_query.rs
    │           ├── operation_query_sync_result.rs
    │           └── common.rs
    ├── crypto_manager/               # 加密/解密管理
    │   └── src/
    │       ├── lib.rs
    │       ├── crypto.rs             # 加密操作
    │       ├── crypto_manager.rs     # 密钥生命周期管理
    │       ├── secret_key.rs         # 密钥处理
    │       ├── db_key_operator.rs    # 数据库密钥操作
    │       ├── huks_wrapper.c        # HUKS 包装器（C）
    │       └── huks_wrapper.h
    ├── db_operator/                  # 数据库操作
    │   └── src/
    │       ├── lib.rs
    │       ├── database.rs           # SQLite 数据库管理
    │       ├── table.rs              # 表结构操作
    │       ├── statement.rs          # SQL 语句处理
    │       ├── transaction.rs        # 事务管理
    │       ├── database_file_upgrade.rs  # DB 文件迁移
    │       ├── database_util.rs      # 数据库工具
    │       ├── process_batch_data.rs # 批量处理
    │       ├── types.rs              # 数据库类型
    │       ├── sqlite3_wrapper.c     # SQLite3 C 绑定
    │       └── common/               # 通用 DB 操作
    │           ├── argument_check.rs
    │           ├── permission_check.rs
    │           └── operation_add_common.rs
    ├── common/                       # 共享服务工具
    │   └── src/
    │       ├── lib.rs
    │       ├── calling_info.rs       # 调用者身份管理
    │       ├── counter.rs            # 操作计数
    │       ├── task_manager.rs       # 异步任务管理
    │       └── process_info.rs       # 进程信息
    ├── os_dependency/                # 服务 OS 抽象
    │   ├── inc/
    │   │   ├── data_share_wrapper.h
    │   │   ├── memory_manager_wrapper.h
    │   │   ├── os_account_wrapper.h
    │   │   ├── access_token_wrapper.h
    │   │   ├── file_operator_wrapper.h
    │   │   ├── system_event_wrapper.h
    │   │   ├── system_ability_wrapper.h
    │   │   └── bms_wrapper.h         # Bundle Manager 包装器
    │   └── src/
    │       ├── data_share_wrapper.cpp
    │       ├── memory_manager_wrapper.cpp
    │       ├── os_account_wrapper.cpp
    │       ├── access_token_wrapper.cpp
    │       ├── file_operator_wrapper.cpp
    │       ├── system_event_wrapper.cpp
    │       ├── system_ability_wrapper.cpp
    │       └── bms_wrapper.cpp
    └── plugin/                       # 插件系统
        └── src/
            ├── lib.rs
            └── asset_plugin.rs       # AssetPlugin, 加载器
```

**证据**：完整目录树从实际文件系统扫描得出，源文件总计约 114 个（Rust ~50，C/C++ ~64）。

---

## 各目录/模块职责

### 框架层 (frameworks/)

**职责**：提供客户端库，供应用和系统组件与 Asset 服务交互。

| 子目录 | 职责 | 主要文件 |
|-------|------|---------|
| `definition/` | 核心数据类型定义，共享给所有层 | lib.rs, macros.rs |
| `ipc/` | IPC 通信协议（序列化/反序列化、SA_ID 定义） | lib.rs |
| `utils/` | 工具函数（时间、哈希） | time.rs, hasher.rs |
| `os_dependency/` | OS 抽象层（内存、文件、日志、OpenSSL） | memory/, file/, log/, openssl/ |
| `js/napi/` | JavaScript N-API 绑定（JS 到 native 桥接） | asset_napi.cpp, add/update/query 等 |
| `c/system_api/` | C 系统 API 实现 | asset_system_api.c |

**依赖方向**：`interfaces/` → `frameworks/` → `services/`

---

### 接口层 (interfaces/)

**职责**：定义公共 API 边界。

| 子目录 | 职责 | 主要文件 |
|-------|------|---------|
| `kits/c/` | 公共 NDK API（第三方应用用） | asset_api.h, asset_type.h, asset_api.c |
| `inner_kits/c/` | 内部 C API（系统组件用） | asset_system_api.h, asset_system_type.h |
| `inner_kits/rs/` | Rust SDK（Rust 系统组件用） | lib.rs |
| `inner_kits/plugin_interface/` | 插件扩展接口 | plugin_interface.rs |

**边界**：
- `kits/c/` → 第三方应用（公开 API）
- `inner_kits/` → 系统组件（内部 API）

---

### 服务层 (services/)

**职责**：核心服务实现，作为 System Ability SA_ID=8100 运行。

| 子目录 | 职责 | 主要文件 |
|-------|------|---------|
| `core_service/` | 主服务入口、IPC stub、操作分发 | lib.rs, stub.rs, operations/ |
| `crypto_manager/` | 加密/解密、HUKS 集成 | crypto.rs, huks_wrapper.c |
| `db_operator/` | SQLite 数据库操作、事务、Schema | database.rs, table.rs, sqlite3_wrapper.c |
| `common/` | 共享工具（调用信息、计数、任务管理） | calling_info.rs, counter.rs, task_manager.rs |
| `os_dependency/` | 服务 OS 包装器（BMS、AccessToken 等） | access_token_wrapper.cpp, bms_wrapper.cpp 等 |
| `plugin/` | 插件系统（扩展加载器） | asset_plugin.rs |

---

## 模块边界与依赖方向

### 边界定义

```
+---------------------+
|   interfaces/     |  ← API 边界（公开/内部）
+--------+------------+
         |
         ↓
+---------------------+
|    frameworks/     |  ← 客户端库
+--------+------------+
         |
         ↓ (IPC)
+---------------------+
|    services/       |  ← 服务端实现
+---------------------+
         ↓
+---------------------+
|   外部依赖       |  ← HUKS, UserIAM, AccessToken 等
+---------------------+
```

### 依赖方向

**避免环依赖**：依赖方向清晰，从 interfaces → frameworks → services

| 依赖链 | 说明 |
|-------|------|
| interfaces/kits/c → frameworks/c/system_api | NDK 调用 C system API |
| frameworks/c/system_api → interfaces/inner_kits/rs | C API 通过 Rust 实现 |
| interfaces/inner_kits/rs → frameworks/ipc → frameworks/definition | SDK 通过 IPC 调用服务 |
| frameworks/ipc → frameworks/definition → frameworks/os_dependency/log | IPC 使用定义和日志 |
| services/core_service → interfaces/inner_kits/rs | 服务通过 SDK 加载自身（插件机制） |
| services/core_service → all services/ modules | 服务依赖所有服务内部模块 |

---

## 语言分布

| 目录 | 主要语言 | 文件数 | 说明 |
|------|---------|--------|------|
| `frameworks/definition/` | Rust | 4 | 核心类型定义 |
| `frameworks/ipc/` | Rust | 1 | IPC 协议 |
| `frameworks/js/napi/` | C++ | 20 | JS 绑定 |
| `frameworks/c/system_api/` | C | 1 | C API 实现 |
| `services/core_service/` | Rust | 16 | 主服务逻辑 |
| `services/crypto_manager/` | Rust/C | 6 | 加密（C 用于 HUKS） |
| `services/db_operator/` | Rust/C | 11 | 数据库（C 用于 SQLite） |
| `services/common/` | Rust | 5 | 工具 |
| `services/os_dependency/` | C++ | 16 | OS 包装器 |
| `services/plugin/` | Rust | 2 | 插件 |
| `interfaces/kits/c/` | C | 2 | NDK |
| `interfaces/inner_kits/c/` | Rust | 1 | C FFI |
| `interfaces/inner_kits/rs/` | Rust/C | 2 | SDK |

**总计**：~114 源文件（Rust ~50，C/C++ ~64）

---

## 关键结论

1. **清晰的层次结构**：interfaces（API）→ frameworks（客户端）→ services（服务端）
2. **语言分工**：Rust 用于核心业务逻辑，C/C++ 用于系统集成和 N-API
3. **模块化设计**：每个子目录职责单一，依赖方向清晰
4. **测试隔离**：测试代码完全隔离在 `test/` 目录，不影响主干代码

---

## 相关跳转

- [项目概述](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 深入了解数据流和线程模型
- [对外 N-API](03_NAPI_API.md) - 了解 JS API 调用
