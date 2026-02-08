# 配置标志与特性开关

## 目的

本文档说明 relational_store 组件中的关键配置标志、特性开关和编译参数。

## 适用范围

- GN 构建配置参数
- 运行时特性开关
- 编译时条件编译

## 关键结论

### 1. 构建配置标志（relational_store.gni）

| 标志名 | 默认值 | 说明 | 影响 | 证据 |
|--------|---------|------|------|------|
| `relational_store_rdb_support_icu` | true | 是否支持 ICU（Unicode 排序） | 编译 librelational_store_icu.z.so | `relational_store.gni:15` |
| `relational_store_config` | true | 是否启用配置模块（信任列表、静默代理） | 安装配置文件 | `relational_store.gni:22` |
| `arkdata_db_core_is_exists` | 动态检测 | arkdata 数据库核心是否存在 | 动态加载依赖 | `relational_store.gni:16-21` |
| `relational_store_dm_part_is_enabled` | 动态检测 | device_manager 部件是否启用 | 分布式同步 | `relational_store.gni:24-28` |

### 2. 路径变量

| 变量名 | 默认值 | 说明 | 使用位置 |
|--------|---------|------|----------|
| `common_tool_path` | `//foundation/distributeddatamgr/kv_store/frameworks/common` | KV Store 公共工具路径 | 多个 BUILD.gn |
| `distributeddata_base_path` | `//foundation/distributeddatamgr` | 分布数据基础路径 | 多个 BUILD.gn |
| `relational_store_base_path` | `//foundation/distributeddatamgr/relational_store` | 关系型存储基础路径 | 所有 BUILD.gn |
| `relational_store_mock_path` | `${relational_store_base_path}/rdbmock` | Mock 测试路径 | 跨平台 BUILD.gn |
| `relational_store_js_common_path` | `${relational_store_base_path}/frameworks/js/napi/common` | JS 公共路径 | NAPI 模块 |
| `relational_store_napi_path` | `${relational_store_base_path}/frameworks/js/napi` | NAPI 框架路径 | 所有 NAPI 模块 |
| `relational_store_native_path` | `${relational_store_base_path}/frameworks/native` | 原生实现路径 | 所有 native 模块 |
| `relational_store_innerapi_path` | `${relational_store_base_path}/interfaces/inner_api` | 内部接口路径 | Inner API 模块 |
| `relational_store_common_path` | `${relational_store_base_path}/frameworks/common` | 公共头文件路径 | native 模块 |
| `cloud_data_native_path` | `${relational_store_native_path}/cloud_data` | 云数据实现路径 | cloud_data 模块 |
| `cloud_data_napi_path` | `${relational_store_napi_path}/cloud_data` | 云数据 NAPI 路径 | cloud_data NAPI 模块 |

### 3. 平台特定配置

#### 3.1 OHOS 平台（is_ohos）

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `defines` | `["SQLITE_DISTRIBUTE_RELATIONAL"]` | 启用分布式功能 | `frameworks/js/napi/relationalstore/BUILD.gn:57` |
| `external_deps` | 包含 hilog, ipc, napi 等 | OpenHarmony 系统依赖 | 同上 |

#### 3.2 MinGW/Windows 平台（is_mingw）

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `cflags_cc` | `["-std=c++17", "-stdlib=libc++"]` | C++17 标准 | `frameworks/js/napi/relationalstore/BUILD.gn:103` |
| `defines` | `["WINDOWS_PLATFORM", "CROSS_PLATFORM", "API_EXPORT=__declspec(dllimport)"]` | Windows 平台标识 | 同上 |
| `buildos` | `"windows"` | 构建操作系统标识 | 同上 |

#### 3.3 Mac 平台（is_mac）

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `cflags_cc` | `["-std=c++17", "-stdlib=libc++"]` | C++17 标准 | `frameworks/js/napi/relationalstore/BUILD.gn:144` |
| `defines` | `["MAC_PLATFORM", "CROSS_PLATFORM"]` | Mac 平台标识 | 同上 |
| `buildos` | `"mac"` | 构建操作系统标识 | 同上 |

#### 3.4 Android 平台（is_android）

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `cflags_cc` | `["-std=c++17", "-stdlib=libc++"]` | C++17 标准 | `frameworks/js/napi/relationalstore/BUILD.gn:190` |
| `defines` | `["ANDROID_PLATFORM", "CROSS_PLATFORM"]` | Android 平台标识 | 同上 |

#### 3.5 iOS 平台（is_ios）

| 配置项 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| `cflags_cc` | `["-std=c++17", "-stdlib=libc++"]` | C++17 标准 | `frameworks/js/napi/relationalstore/BUILD.gn:243` |
| `defines` | `["IOS_PLATFORM", "CROSS_PLATFORM"]` | iOS 平台标识 | 同上 |

### 4. 安全级别配置

| 级别 | 含义 | 代码实现 | 证据 |
|------|------|----------|------|
| **S0** | 级别 0 - 不安全 | `security_policy.cpp:25-35` |
| **S1** | 级别 1 - 最低安全 | 同上 |
| **S2** | 级别 2 - 低安全 | 同上 |
| **S3** | 级别 3 - 中等安全 | 同上 |
| **S4** | 级别 4 - 高安全（数据库加密） | 同上 |

**安全级别设置**：
- 通过 `SecurityPolicy::SetSecurityLabel()` 设置
- 影响 ACL（Access Control List）权限
- S4 级别强制数据库加密

### 5. 订阅模式配置

| 模式 | 值 | 说明 | 证据 |
|------|-----|------|------|
| **LOCAL** | 0 | 本地订阅（仅本地变更） | `rdb_types.h` |
| **REMOTE** | 1 | 远程订阅（远程设备变更） | 同上 |
| **LOCAL_DETAIL** | 2 | 本地详细订阅（所有变更详情） | 同上 |
| **LOCAL_SHARED** | 3 | 本地共享订阅（DataShare） | 同上 |

### 6. 同步模式配置

| 模式 | 值 | 说明 | 证据 |
|------|-----|------|------|
| **PUSH_ONLY** | 0 | 仅推送，不拉取 | `cloud_types.h` |
| **PULL_FIRST** | 1 | 先拉取，后推送 | 同上 |
| **PUSH_FIRST** | 2 | 先推送，后拉取 | 同上 |

### 7. 冲突解决策略

| 策略 | 值 | 说明 | 证据 |
|------|-----|------|------|
| **ON_CONFLICT_NONE** | 0 | 无冲突解决 | `rdb_types.h` |
| **ON_CONFLICT_FAIL** | 1 | 冲突时操作失败 | 同上 |
| **ON_CONFLICT_REPLACE** | 2 | 冲突时替换 | 同上 |
| **ON_CONFLICT_ABORT** | 3 | 冲突时中止 | 同上 |
| **ON_CONFLICT_IGNORE** | 4 | 冲突时忽略 | 同上 |
| **ON_CONFLICT_ROLLBACK** | 5 | 冲突时回滚 | 同上 |

### 8. 运行时配置

#### 8.1 信任列表配置

**文件路径**：`/system/etc/trusts/conf/trusts_config.json`

**信任的 Bundle 列表**：
```json
[
  "com.ohos.calendardata",
  "com.ohos.camera",
  "com.ohos.photos",
  "com.ohos.security.privacycenter",
  "com.ohos.contactsdataability",
  "com.ohos.ringtonelibrary.ringtonelibrarydata",
  "com.ohos.settingsdata",
  "com.ohos.telephonydataability"
]
```

**使用场景**：
- 只有信任列表中的 bundle 可以使用某些高级功能
- 例如：DataAbility 共享、特定查询优化
- 验证代码：`interfaces/ndk/src/oh_data_utils.cpp:85-123`

#### 8.2 静默代理配置

**文件路径**：`/system/etc/silent/conf/silentproxy_config.json`

**配置结构**：
```json
{
  "silentProxys": [
    {
      "bundleName": "com.ohos.calendardata",
      "storeNames": ["calendardata"]
    },
    // ... 其他 bundle 配置
  ]
}
```

**用途**：
- 静默同步时代理网络请求
- 避免应用直接访问网络
- 提供统一的代理服务

### 9. 编译优化标志

| 标志 | 说明 | 平台 | 证据 |
|------|------|------|------|
| `SQLITE_DISTRIBUTE_RELATIONAL` | 启用分布式功能 | OHOS | `frameworks/js/napi/relationalstore/BUILD.gn:57` |
| `CROSS_PLATFORM` | 跨平台编译标志 | 非 OHOS | 所有平台 |
| `NDEBUG` | Release 模式 | 所有平台 | 标准 C++ 定义 |
| `HILOG_ENABLE` | 日志编译标志 | OHOS | `frameworks/native/rdb/src/rdb_sql_log.cpp` |

### 10. SQLite 编译选项

| 选项 | 值 | 说明 | 影响 |
|------|-----|------|------|
| **SQLITE_THREADSAFE=1** | 串行模式 | 禁用 SQLite 线程支持（使用连接池） | `sqlite_connection.cpp` |
| **SQLITE_DEFAULT_MEMSTATUS=0** | 内存管理 | 使用 SQLite 默认 | 同上 |
| **SQLITE_DEFAULT_WAL_SYNCHRONOUS=1** | WAL 同步模式 | 启用 WAL 模式以提高并发 | `rdb_store_config.cpp` |
| **SQLITE_TEMP_STORE=1** | 使用临时存储 | 存储临时表 | `rdb_store_impl.cpp` |

### 11. 连接池配置

| 参数 | 默认值 | 说明 | 证据 |
|------|---------|------|------|
| **最大连接数** | 4 | 连接池最大连接数 | `README_zh.md:46` |
| **连接超时** | 未指定（默认无限制） | 连接获取超时 | `connection_pool.cpp` |
| **空闲超时** | 未指定（默认无限制） | 连接空闲超时 | 同上 |

**连接类型**：
- READ_ONLY：读连接（可并发）
- EXCLUSIVE：写连接（独占）

## 配置影响总结

| 配置项 | 启用后效果 | 禁用后效果 | 切换方式 |
|---------|-----------|-----------|---------|
| **ICU 支持** | 支持中文排序、本地化 | 仅支持 ASCII | 重新编译 |
| **配置模块** | 安装信任列表和代理 | 不安装 | 重新编译 |
| **分布式功能** | 支持设备间同步 | 禁用分布式 | 运行时特性开关 |
| **安全级别 S4** | 强制加密数据库 | 不强制加密 | 运行时配置 |
| **WAL 模式** | 提高并发性能 | 降低并发性 | 修改编译选项 |

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: relational_store.gni、各模块 BUILD.gn、rdb_store_config.cpp、security_policy.cpp
