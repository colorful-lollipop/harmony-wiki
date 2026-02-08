# GN 构建系统详解

## 目的

本文档详细说明 relational_store 组件的 GN（Ninja）构建系统，包括构建目标列表、类型、依赖关系和配置参数。

## 适用范围

- 所有 BUILD.gn 文件
- relational_store.gni 配置文件
- bundle.json 构建配置
- 构建目标类型与输出

## 关键结论

### 1. 根配置文件

#### 1.1 relational_store.gni

**文件路径**：`relational_store.gni`

**定义的路径变量**：

| 变量名 | 默认值 | 说明 | 证据 |
|----------|--------|------|------|
| `relational_store_rdb_support_icu` | true | 是否支持 ICU（Unicode 排序） | `relational_store.gni:15` |
| `arkdata_db_core_is_exists` | 动态检测 | arkdata 数据库核心是否存在 | `relational_store.gni:16-21` |
| `relational_store_config` | true | 是否启用配置模块 | `relational_store.gni:22` |
| `relational_store_dm_part_is_enabled` | 动态检测 | device_manager 部件是否启用 | `relational_store.gni:24-28` |
| `common_tool_path` | `//foundation/distributeddatamgr/kv_store/frameworks/common` | 公共工具路径 | `relational_store.gni:31` |
| `relational_store_base_path` | `//foundation/distributeddatamgr/relational_store` | 项目基础路径 | `relational_store.gni:34` |
| `relational_store_js_common_path` | `${relational_store_base_path}/frameworks/js/napi/common` | JS 公共路径 | `relational_store.gni:41-42` |

### 2. 核心 Build Targets

#### 2.1 Inner API 库

| 目标名 | 类型 | 输出 | 依赖 | 证据 |
|--------|------|------|--------|------|
| **native_rdb** | ohos_shared_library | libnative_rdb.z.so | 无 | `interfaces/inner_api/rdb/BUILD.gn` |
| **native_rdb_static** | ohos_static_library | libnative_rdb.a | 无 | 同上 |
| **native_appdatafwk** | ohos_shared_library | libnative_appdatafwk.z.so | 无 | `interfaces/inner_api/appdatafwk/BUILD.gn` |
| **native_dataability** | ohos_shared_library | libnative_dataability.z.so | native_appdatafwk | `interfaces/inner_api/dataability/BUILD.gn` |
| **cloud_data_native** | ohos_shared_library | libcloud_data_native.z.so | native_rdb, native_appdatafwk | `interfaces/inner_api/cloud_data/BUILD.gn` |
| **cloud_data_inner** | ohos_static_library | libcloud_data_inner.a | cloud_data_native | `interfaces/inner_api/cloud_data/BUILD.gn:189` |
| **rdb_data_share_adapter** | ohos_shared_library | librdb_data_share_adapter.z.so | native_rdb | `interfaces/inner_api/rdb_data_share_adapter/BUILD.gn` |
| **rdb_data_ability_adapter** | ohos_shared_library | librdb_data_ability_adapter.z.so | native_rdb, native_dataability | `interfaces/inner_api/rdb_data_ability_adapter/BUILD.gn` |

#### 2.2 NDK 库

| 目标名 | 类型 | 输出 | 依赖 | 证据 |
|--------|------|------|--------|------|
| **native_rdb_ndk** | ohos_shared_library | libnative_rdb_ndk.z.so | native_rdb | `interfaces/ndk/BUILD.gn` |
| **native_rdb_ndk_utils** | ohos_shared_library | libnative_rdb_ndk_utils.z.so | native_rdb_ndk | `interfaces/rdb_ndk_utils/BUILD.gn` |

#### 2.3 NAPI 库

| 目标名 | 类型 | 导出模块名 | 依赖 | 证据 |
|--------|------|-----------|--------|------|
| **rdb** | ohos_shared_library | data.rdb | native_rdb, native_appdatafwk | `frameworks/js/napi/rdb/BUILD.gn` |
| **relationalstore** | ohos_shared_library | data.relationalStore | native_rdb, native_appdatafwk, rdb_data_share_adapter | `frameworks/js/napi/relationalstore/BUILD.gn` |
| **clouddata** | ohos_shared_library | data.cloudData | cloud_data_native, native_rdb | `frameworks/js/napi/cloud_data/BUILD.gn` |
| **dataability** | ohos_shared_library | data.dataability | native_dataability, native_appdatafwk | `frameworks/js/napi/dataability/BUILD.gn` |
| **cloudextension** | ohos_shared_library | data.cloudExtension | cloud_data_native | `frameworks/js/napi/cloud_extension/BUILD.gn` |
| **sendablerelationalstore** | ohos_shared_library | data.sendableRelationalStore | native_rdb | `frameworks/js/napi/sendablerelationalstore/BUILD.gn` |
| **commontype_napi** | ohos_shared_library | data.commonType | 无 | `frameworks/js/napi/common/BUILD.gn` |

#### 2.4 Native 框架库

| 目标名 | 类型 | 输出 | 依赖 | 证据 |
|--------|------|------|--------|------|
| **relational_store_crypt** | ohos_shared_library | librelational_store_crypt.z.so | huks:libhukssdk | `frameworks/native/rdb_crypt/BUILD.gn` |
| **relational_store_icu** | ohos_shared_library | librelational_store_icu.z.so | icu | `frameworks/native/icu/BUILD.gn` |
| **rdb_obs_mgr_adapter** | ohos_shared_library | librdb_obs_mgr_adapter.z.so | ability_runtime | `frameworks/native/obs_mgr_adapter/BUILD.gn` |

#### 2.5 跨平台目标

| 目标名 | 平台 | 类型 | 输出 | 证据 |
|--------|------|------|------|------|
| **relationalstore** | MinGW | ohos_shared_library | librelationalstore.dll | `frameworks/js/napi/relationalstore/BUILD.gn:92` |
| **relationalstore** | Mac | ohos_shared_library | librelationalstore.dylib | `frameworks/js/napi/relationalstore/BUILD.gn:132` |
| **data_relationalstore** | Android | ohos_source_set | - | `frameworks/js/napi/relationalstore/BUILD.gn:176` |
| **data_relationalstore** | iOS | ohos_source_set | - | `frameworks/js/napi/relationalstore/BUILD.gn:217` |

### 3. 配置模块 Targets

| 目标名 | 类型 | 输出 | 安装路径 | 证据 |
|--------|------|------|----------|------|
| **trusts_conf** | ohos_prebuilt_etc | trusts_config.json | /system/etc/trusts/conf/ | `conf/BUILD.gn:26-31` |
| **silent_conf** | ohos_prebuilt_etc | silentproxy_config.json | /system/etc/silent/conf/ | `conf/BUILD.gn:33-38` |
| **build_module** | group | 依赖上述两个配置 | - | `conf/BUILD.gn:17-24` |

### 4. 关键依赖关系

#### 4.1 外部依赖（来自 bundle.json）

| 依赖 | 说明 | 使用位置 |
|------|------|----------|
| **ability_runtime** | 能力运行时 | JS NAPI、观察者适配 |
| **access_token** | 访问令牌 | 权限验证 |
| **common_event_service** | 公共事件服务 | NAPI 绑定 |
| **data_share** | 数据共享 | RDB 适配器 |
| **device_manager** | 设备管理 | 云同步 |
| **eventhandler** | 事件处理 | 异步队列 |
| **hilog** | 日志 | 所有模块 |
| **hitrace** | 性能追踪 | 性能统计 |
| **huks** | 硬件密钥服务 | 数据库加密 |
| **ipc** | 进程间通信 | 分布式同步 |
| **kv_store** | KV 存储 | 云同步 |
| **napi** | N-API 基础库 | JS 绑定 |
| **samgr** | 系统能力管理器 | 服务加载 |
| **hisysevent** | 系统事件 | 诊断 |
| **bounds_checking_function** | 边界检查 | 安全增强 |
| **icu** | 国际化组件 | 文本处理 |
| **sqlite** | SQLite 数据库 | 存储引擎 |
| **file_api** | 文件 API | 数据库文件操作 |
| **json** | JSON 库 | 配置解析 |
| **runtime_core** | 运行时核心 | 应用上下文 |

#### 4.2 内部依赖图

```
                    ┌──────────────────────┐
                    │  bundle.json        │
                    │  (依赖声明）        │
                    └───────────┬──────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │Native RDB    │   │  Native Cloud │   │  Native Etc  │
    │Libraries      │   │  Libraries    │   │  Libraries    │
    └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
           │                    │                   │
           │              ┌─────────┬─────────┐   │
           │              │         │         │   │
           ▼              ▼         ▼         ▼   ▼
    ┌─────────────────────────────────────────────┐
    │          NAPI/NDK Libraries          │
    │(依赖 Native Libraries)                │
    └─────────────────────────────────────────────┘
```

**依赖说明**：
- **Native RDB Libraries**：core RDB、dataability、appdatafwk 等
- **Native Cloud Libraries**：cloud_data、适配器等
- **Native Etc Libraries**：加密、ICU、观察者适配等
- **NAPI/NDK Libraries**：依赖上述所有 Native 库

### 5. 特性开关

| 特性 | 配置 | 影响 | 证据 |
|------|------|------|------|
| **ICU 支持** | `relational_store_rdb_support_icu=true` | 编译 relational_store_icu 库 | `relational_store.gni:15` |
| **配置模块** | `relational_store_config=true` | 安装信任列表和静默代理配置 | `relational_store.gni:22` |
| **设备管理** | `relational_store_dm_part_is_enabled=true` | 启用设备间同步 | `relational_store.gni:24-28` |

### 6. 目标输出清单

| 目标类型 | 输出后缀 | 安装目录 | 说明 |
|----------|-----------|----------|------|
| **ohos_shared_library** | .z.so | /system/lib64/ | 动态库 |
| **ohos_static_library** | .a | - | 静态库（内部链接） |
| **ohos_prebuilt_etc** | - | /system/etc/... | 配置文件 |
| **ohos_source_set** | - | - | 跨平台源码集合 |

### 7. 构建参数

| 参数 | 说明 | 典型值 |
|------|------|--------|
| `cflags_cc` | C++ 编译标志 | `-std=c++17` |
| `defines` | 宏定义 | `SQLITE_DISTRIBUTE_RELATIONAL` |
| `include_dirs` | 头文件搜索路径 | 各模块 include/ |
| `deps` | 内部依赖 | 依赖其他 gn target |
| `external_deps` | 外部依赖 | 来自其他子系统的库 |
| `subsystem_name` | 子系统名称 | distributeddatamgr |
| `part_name` | 部件名称 | relational_store |

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物与安装路径
- [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 配置标志详解

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: relational_store.gni、bundle.json、各模块 BUILD.gn 文件
