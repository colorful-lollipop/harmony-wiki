# 编译产物与安装路径

## 目的

本文档说明 relational_store 组件的编译产物清单、安装路径和运行时加载关系。

## 适用范围

- GN 构建输出的动态库、静态库、配置文件
- 安装路径和文件名
- 运行时加载依赖关系

## 关键结论

### 1. 产物清单

#### 1.1 核心动态库（.so）

| 库名 | 目标 | 安装路径 | 大小估计 | 证据 |
|------|------|----------|---------|------|
| **libnative_rdb.z.so** | `native_rdb` | `/system/lib64/` | ~200 KB | `interfaces/inner_api/rdb/BUILD.gn:52` |
| **libnative_appdatafwk.z.so** | `native_appdatafwk` | `/system/lib64/` | ~50 KB | `interfaces/inner_api/appdatafwk/BUILD.gn` |
| **libcloud_data_native.z.so** | `cloud_data_native` | `/system/lib64/` | ~100 KB | `interfaces/inner_api/cloud_data/BUILD.gn:52` |
| **libnative_dataability.z.so** | `native_dataability` | `/system/lib64/` | ~60 KB | `interfaces/inner_api/dataability/BUILD.gn` |
| **librdb_data_share_adapter.z.so** | `rdb_data_share_adapter` | `/system/lib64/` | ~40 KB | `interfaces/inner_api/rdb_data_share_adapter/BUILD.gn` |
| **librdb_data_ability_adapter.z.so** | `rdb_data_ability_adapter` | `/system/lib64/` | ~40 KB | `interfaces/inner_api/rdb_data_ability_adapter/BUILD.gn` |
| **librelational_store_crypt.z.so** | `relational_store_crypt` | `/system/lib64/` | ~30 KB | `frameworks/native/rdb_crypt/BUILD.gn` |
| **librelational_store_icu.z.so** | `relational_store_icu` | `/system/lib64/` | ~50 KB | `frameworks/native/icu/BUILD.gn` |
| **librdb_obs_mgr_adapter.z.so** | `rdb_obs_mgr_adapter` | `/system/lib64/` | ~20 KB | `frameworks/native/obs_mgr_adapter/BUILD.gn` |

**注意**：`librelational_store_icu.z.so` 仅在 `relational_store_rdb_support_icu=true` 时编译

#### 1.2 NDK 动态库（.so）

| 库名 | 目标 | 安装路径 | 大小估计 | 证据 |
|------|------|----------|---------|------|
| **libnative_rdb_ndk.z.so** | `native_rdb_ndk` | `/system/lib64/` | ~150 KB | `interfaces/ndk/BUILD.gn:52` |
| **libnative_rdb_ndk_utils.z.so** | `native_rdb_ndk_utils` | `/system/lib64/` | ~20 KB | `interfaces/rdb_ndk_utils/BUILD.gn:52` |

#### 1.3 NAPI 动态库（.so）

| 库名 | 目标 | 模块名 | 安装路径 | 证据 |
|------|------|---------|----------|------|
| **librelationalstore.z.so** | `relationalstore` | data.relationalStore | `/system/lib64/module/data/` | `frameworks/js/napi/relationalstore/BUILD.gn:89` |
| **librdb.z.so** | `rdb` | data.rdb | `/system/lib64/module/data/` | `frameworks/js/napi/rdb/BUILD.gn:61` |
| **libclouddata.z.so** | `clouddata` | data.cloudData | `/system/lib64/module/data/` | `frameworks/js/napi/cloud_data/BUILD.gn:52` |
| **libdataability.z.so** | `dataability` | data.dataability | `/system/lib64/module/data/` | `frameworks/js/napi/dataability/BUILD.gn:52` |
| **libcloudextension.z.so** | `cloudextension` | data.cloudExtension | `/system/lib64/module/data/` | `frameworks/js/napi/cloud_extension/BUILD.gn:52` |
| **libsendablerelationalstore.z.so** | `sendablerelationalstore` | data.sendableRelationalStore | `/system/lib64/module/data/` | `frameworks/js/napi/sendablerelationalstore/BUILD.gn:52` |
| **libcommontype_napi.z.so** | `commontype_napi` | data.commonType | `/system/lib64/module/data/` | `frameworks/js/napi/common/BUILD.gn:52` |

#### 1.4 仓颉 FFI 库（.so）

| 库名 | 目标 | 安装路径 | 证据 |
|------|------|----------|------|
| **libcj_relational_store_ffi.z.so** | `cj_relational_store_ffi` | `/system/lib64/` | `frameworks/cj/BUILD.gn:89` |

#### 1.5 静态库（.a）

| 库名 | 目标 | 用途 | 证据 |
|------|------|------|------|
| **libnative_rdb.a** | `native_rdb_static` | 内部静态链接 | `interfaces/inner_api/rdb/BUILD.gn:79` |
| **libcloud_data_inner.a** | `cloud_data_inner` | 内部静态链接（限 datamgr_service） | `interfaces/inner_api/cloud_data/BUILD.gn:189` |

#### 1.6 配置文件

| 文件名 | 安装路径 | 内容 | 证据 |
|--------|----------|------|------|
| **trusts_config.json** | `/system/etc/trusts/conf/trusts_config.json` | 信任 Bundle 列表（8 个系统应用） | `conf/BUILD.gn:26-31` |
| **silentproxy_config.json** | `/system/etc/silent/conf/silentproxy_config.json` | 静默代理配置 | `conf/BUILD.gn:33-38` |

### 2. 安装路径总结

| 产物类型 | 安装路径 | 权限 | 证据 |
|----------|----------|------|------|
| **系统库** | `/system/lib64/` | 644 | bundle.json 依赖 |
| **NAPI 模块** | `/system/lib64/module/data/` | 644 | NAPI 模块默认值 |
| **配置文件** | `/system/etc/trusts/conf/`<br/>`/system/etc/silent/conf/` | 644 | ohos_prebuilt_etc 目标 |
| **数据库文件** | `/data/storage/el2/.../rdb/` | 644 | 应用沙箱内 |

### 3. 运行时加载关系

#### 3.1 NAPI 模块加载链示例

```
应用启动
    ↓
[加载 librelationalstore.z.so]
    ↓
[napi_module_register("data.relationalStore")]
    ↓
[调用 RdbStoreProxy::Init(env, exports)]
    ↓
[应用调用 getRdbStore(config)]
    ↓
[加载 libnative_rdb.z.so]
    ↓
[RdbHelper::getRdbStore(config)]
    ↓
[加载 librelational_store_crypt.z.so]
    ↓
[RdbSecurityManager::Init(config)]
    ↓
[打开数据库文件]
```

#### 3.2 云同步模块加载链示例

```
应用调用 cloudData.cloudSync()
    ↓
[加载 libclouddata.z.so]
    ↓
[napi_module_register("data.cloudData")]
    ↓
[调用 CloudManager::DoSync()]
    ↓
[加载 libcloud_data_native.z.so]
    ↓
[CloudManager::Init()]
    ↓
[通过 SystemAbilityManager::LoadSystemAbility() 加载云服务]
    ↓
[调用 CloudServiceProxy]
    ↓
[通过 IPC 调用 DataMgr Service]
```

#### 3.3 依赖链解析

```
librelationalstore.z.so
    ├─→ libnative_rdb.z.so (核心 RDB）
    ├─→ libnative_appdatafwk.z.so (共享块）
    └─→ librdb_data_share_adapter.z.so (DataShare 适配）

libclouddata.z.so
    ├─→ libcloud_data_native.z.so (云数据接口）
    └─→ libnative_rdb.z.so (核心 RDB）

librdb.z.so (Legacy)
    ├─→ libnative_rdb.z.so (核心 RDB）
    └─→ libnative_appdatafwk.z.so (共享块）

libdataability.z.so
    ├─→ libnative_dataability.z.so (DataAbility）
    └─→ libnative_appdatafwk.z.so (共享块）
```

### 4. 构建配置产物

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `subsystem_name` | distributeddatamgr | 子系统名称 | `bundle.json:43` |
| `part_name` | relational_store | 部件名称 | `bundle.json:44` |
| `rom` | "1000" | 最小 ROM 需求（KB） | `bundle.json:49` |
| `ram` | "350" | 最小 RAM 需求（KB） | `bundle.json:50` |
| `version` | "3.1.0" | 组件版本 | `bundle.json:3` |

### 5. 特性开关影响

| 特性 | 编译时产物影响 | 运行时影响 |
|------|-------------|-------------|
| `relational_store_rdb_support_icu=true` | 生成 librelational_store_icu.z.so | 加载 ICU 库用于文本排序 |
| `relational_store_rdb_support_icu=false` | 不生成 librelational_store_icu.z.so | 使用 SQLite 默认排序 |
| `relational_store_config=true` | 安装信任列表和静默代理配置 | 应用可使用信任列表功能 |

### 6. 文件大小优化

| 产物类型 | 优化策略 | 当前状态 |
|----------|-----------|---------|
| **核心库** | 分离为多个小库 | 已优化（RDB、加密、ICU 独立） |
| **NAPI 模块** | 延迟加载 | 部分模块使用延迟加载 |
| **NDK 库** | 静态链接可选 | native_rdb_static 提供 |

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [06_GN_Build.md](./06_GN_Build.md) - GN 构建详解

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: bundle.json、各模块 BUILD.gn 文件
