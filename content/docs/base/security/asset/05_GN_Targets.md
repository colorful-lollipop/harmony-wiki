# GN Targets 与编译产物

## 目的

本文档梳理 ASSET 服务的 GN 构建目标、编译产物和依赖关系。

## 适用范围

- 涵盖内容：BUILD.gn 文件、targets 列表、类型、依赖、产物
- 目标读者：构建工程师

---

## 根 BUILD.gn

**文件**：`BUILD.gn:17-25`

### 主要 Targets

| Target | 类型 | 目的 |
|--------|------|------|
| `asset_component` | group | 主生产构建目标 |
| `asset_bin_test` | group | 测试构建目标（testonly） |

### asset_component 依赖

```gn
group("asset_component") {
  deps = [
    "interfaces/inner_kits/rs:asset_sdk_rust",      # Rust SDK
    "interfaces/kits/c:asset_ndk",                 # NDK C API
    "sa_profile:asset_sa_profiles",                 # SA 配置
    "services/core_service:asset_service",             # 主服务
    "services/plugin:asset_plugin",                 # 插件系统
  ]

  if (support_jsapi) {
    deps += [ "frameworks/js/napi:asset_napi" ]      # JS API（条件）
  }
}
```

**证据**：`BUILD.gn:17-29`

---

## 关键 Targets

### Services 层

#### asset_service (主服务)

| 属性 | 值 |
|------|-----|
| **文件** | services/core_service/BUILD.gn:17-59 |
| **类型** | ohos_rust_shared_library |
| **输出名** | libasset_service.so (dylib) |
| **源文件** | src/lib.rs |
| **主要依赖** | 12 个内部模块 + 9 个外部依赖 |

**依赖列表**：
```
内部依赖：
- asset_definition, asset_ipc, asset_file_operator, asset_log, asset_utils
- asset_sdk_rust, asset_plugin_interface_rust
- asset_common, asset_crypto_manager, asset_db_operator, asset_os_dependency
- asset_plugin

外部依赖：
- hilog_rust, libhilog, hisysevent_rust, hitrace_meter_rust
- ipc_rust, system_ability_fwk_rust, samgr_rust
- ylong_runtime, ylong_json
- lazy-static.rs, serde
```

**证据**：`services/core_service/BUILD.gn:18-35`

#### asset_crypto_manager (加密模块)

| 属性 | 值 |
|------|-----|
| **文件** | services/crypto_manager/BUILD.gn |
| **类型** | ohos_rust_static_library |
| **输出名** | librlib |
| **依赖** | asset_definition, asset_file_operator, asset_log, asset_openssl_wrapper, asset_huks_wrapper |

**证据**：`services/crypto_manager/BUILD.gn`

#### asset_db_operator (数据库模块)

| 属性 | 值 |
|------|-----|
| **文件** | services/db_operator/BUILD.gn |
| **类型** | ohos_rust_static_library (rlib) + ohos_static_library |
| **输出** | librlib + lib.a |
| **依赖** | asset_definition, asset_file_operator, asset_sqlite3_wrapper |

**证据**：`services/db_operator/BUILD.gn`

### Interfaces 层

#### asset_ndk (公共 NDK)

| 属性 | 值 |
|------|-----|
| **文件** | interfaces/kits/c/BUILD.gn:20-42 |
| **类型** | ohos_shared_library |
| **输出名** | libasset_ndk.so |
| **源文件** | src/asset_api.c |
| **依赖** | asset_sdk (frameworks/c/system_api）, asset_mem |

**安全配置**：PAC-RET, integer_overflow, CFI, boundary_sanitize, UBSAN

**证据**：`interfaces/kits/c/BUILD.gn:21-47`

#### asset_sdk_rust (Rust SDK)

| 属性 | 值 |
|------|-----|
| **文件** | interfaces/inner_kits/rs/BUILD.gn:30-55 |
| **类型** | ohos_rust_shared_library |
| **输出名** | libasset_sdk.so (dylib) |
| **依赖** | asset_definition, asset_ipc, asset_log, asset_timeout_feature |

**证据**：`interfaces/inner_kits/rs/BUILD.gn:31-34`

#### asset_napi (JS API)

| 属性 | 值 |
|------|-----|
| **文件** | frameworks/js/napi/BUILD.gn:16-60 |
| **类型** | ohos_shared_library |
| **输出名** | libasset_napi.so |
| **安装路径** | module/security/ |
| **源文件** | 10 个 C++ 文件 |
| **依赖** | asset_sdk, asset_mem |

**证据**：`frameworks/js/napi/BUILD.gn:16-30`

### Frameworks 层

#### asset_definition (类型定义)

| 属性 | 值 |
|------|-----|
| **文件** | frameworks/definition/BUILD.gn |
| **类型** | ohos_rust_static_library |
| **输出名** | librlib |
| **依赖** | serde, ylong_runtime |

**证据**：`frameworks/definition/BUILD.gn`

#### asset_ipc (IPC)

| 属性 | 值 |
|------|-----|
| **文件** | frameworks/ipc/BUILD.gn |
| **类型** | ohos_rust_static_library |
| **输出名** | librlib |
| **依赖** | asset_definition, asset_log, ipc_rust |

**证据**：`frameworks/ipc/BUILD.gn`

### 配置 Targets

#### asset_sa_profiles (SA 配置)

| 属性 | 值 |
|------|-----|
| **文件** | sa_profile/BUILD.gn:19-36 |
| **类型** | ohos_sa_profile |
| **输出名** | 8100.json |
| **内容** | SA ID 8100 配置，on-demand 启动触发器 |

**证据**：`sa_profile/8100.json`

#### asset_service.rc (启动配置)

| 属性 | 值 |
|------|-----|
| **文件** | etc/init/BUILD.gn |
| **类型** | ohos_prebuilt_etc |
| **输出名** | asset_service.cfg |
| **安装路径** | system/etc/init/ |

**证据**：`etc/init/asset_service.cfg`

---

## 依赖关系图

### 核心服务依赖树

```
asset_service (dylib)
├── asset_definition (rlib)
├── asset_ipc (rlib)
│   └── asset_definition
│   └── asset_log
├── asset_file_operator (rlib)
│   └── asset_definition
│   └── asset_log
├── asset_log (rlib)
├── asset_utils (rlib)
│   └── asset_definition
│   └── asset_log
│   └── asset_openssl_wrapper (static)
├── asset_sdk_rust (dylib)
│   └── asset_definition
│   └── asset_ipc
│   └── asset_log
│   └── asset_timeout_feature (static)
├── asset_plugin_interface_rust (dylib)
│   └── asset_definition
│   └── asset_ipc
│   └── asset_log
│   └── asset_sdk_rust
├── asset_common (rlib)
│   └── asset_definition
│   └── asset_log
│   └── asset_os_dependency
├── asset_crypto_manager (rlib)
│   └── asset_huks_wrapper (static)
│   └── asset_definition
│   └── asset_file_operator
│   └── asset_log
│   └── asset_openssl_wrapper
│   └── asset_utils
│   └── asset_common
├── asset_db_operator (rlib)
│   └── asset_sqlite3_wrapper (static)
│   └── asset_definition
│   └── asset_file_operator
│   └── asset_log
│   └── asset_utils
│   └── asset_sdk_rust
│   └── asset_common
│   └── asset_crypto_manager
├── asset_os_dependency (static)
│   └── asset_mem (static)
└── asset_plugin (rlib)
    └── asset_definition
    └── asset_file_operator
    └── asset_log
    └── asset_utils
    └── asset_sdk_rust
    └── asset_plugin_interface_rust
    └── asset_os_dependency
    └── asset_common
    └── asset_db_operator
    └── asset_crypto_manager
```

**证据**：从各 BUILD.gn 的 `deps` 字段汇总

---

## 编译产物

### 共享库 (.so)

| 产物 | Target | 安装路径 | 目的 |
|------|--------|-----------|------|
| `libasset_service.so` | services/core_service:asset_service | system/lib/ | 主服务 (SA:8100) |
| `libasset_sdk.so` | interfaces/inner_kits/rs:asset_sdk_rust | system/lib/ | Rust SDK |
| `libasset_plugin_interface.so` | interfaces/inner_kits/plugin_interface:asset_plugin_interface_rust | system/lib/ | 插件接口 |
| `libasset_ndk.so` | interfaces/kits/c:asset_ndk | system/lib/ | NDK 公共 API |
| `libasset_sdk.so` | frameworks/c/system_api:asset_sdk | system/lib/ | C 系统 API |
| `libasset_napi.so` | frameworks/js/napi:asset_napi | system/lib/module/security/ | JS/TS NAPI 绑定 |

### 静态库 (.a)

| 产物 | Target | 目的 |
|------|--------|------|
| `libasset_common.a` | services/common:asset_common | 通用工具 |
| `libasset_db_operator.a` | services/db_operator:asset_db_operator | 数据库操作 |
| `libasset_sqlite3_wrapper.a` | services/db_operator:asset_sqlite3_wrapper | SQLite 包装器 |
| `libasset_crypto_manager.a` | services/crypto_manager:asset_crypto_manager | 加密操作 |
| `libasset_huks_wrapper.a` | services/crypto_manager:asset_huks_wrapper | HUKS 包装器 |
| `libasset_os_dependency.a` | services/os_dependency:asset_os_dependency | OS 抽象 |
| `libasset_plugin.a` | services/plugin:asset_plugin | 插件系统 |
| `libasset_ipc.a` | frameworks/ipc:asset_ipc | IPC 序列化 |
| `libasset_definition.a` | frameworks/definition:asset_definition | 类型定义 |
| `libasset_utils.a` | frameworks/utils:asset_utils | 工具函数 |
| `libasset_file_operator.a` | frameworks/os_dependency/file:asset_file_operator | 文件操作 |
| `libasset_log.a` | frameworks/os_dependency/log:asset_log | 日志 |
| `libasset_mem.a` | frameworks/os_dependency/memory:asset_mem | 内存管理 |
| `libasset_openssl_wrapper.a` | frameworks/os_dependency/openssl:asset_openssl_wrapper | OpenSSL 包装器 |

### 配置文件

| 产物 | Target | 安装路径 | 内容 |
|------|--------|-----------|------|
| `8100.json` | sa_profile:asset_sa_profiles | system/profile/ | SA 配置（on-demand 触发） |
| `asset_service.cfg` | etc/init:asset_service.rc | system/etc/init/ | 服务启动配置 |

**证据**：从 BUILD.gn 输出规则推断

---

## 关键 Defines

### config.gni 变量

| 变量 | 默认值 | 用途 | 证据 |
|------|---------|------|------|
| `enable_local_test` | false | 启用本地测试（添加 `--cfg feature="AssetTest"`） | config.gni:15 |
| `asset_access_control_enabled` | false | 访问控制特性 | config.gni:16 |
| `asset_split_hap_list` | "{}" | HAP 拆分配置 | config.gni:17 |
| `asset_extend_timeout` | false | 扩展超时特性 | config.gni:18 |
| `asset_ce_upgrade_list` | "" | CE 升级列表 | config.gni:19 |

### Conditional Defines

**services/os_dependency/BUILD.gn**：
- `ASSET_UPGRADE_HAP_CONFIG="${asset_split_hap_list}"`
- `ASSET_CE_UPGRADE_CONFIG="${asset_ce_upgrade_list}"`

**interfaces/inner_kits/rs/BUILD.gn**：
- `ASSET_ENABLE_EXTEND_LOAD_TIMEOUT`（如果 `asset_extend_timeout == true`）

**证据**：`services/os_dependency/BUILD.gn:47-49`

---

## 安全配置

### Sanitizers

**目标**：asset_ndk, asset_sdk, asset_napi, asset_openssl_wrapper

| Sanitizer | 说明 |
|-----------|------|
| `integer_overflow` | 整数溢出检查 |
| `cfi` | 控制流完整性 |
| `cfi_cross_dso` | 跨 DSO CFI |
| `boundary_sanitize` | 边界检查 |
| `ubsan` | 未定义行为检查 |
| `pac_ret` | 返回地址保护 |

**证据**：`interfaces/kits/c/BUILD.gn:43-50`

---

## 构建命令

### 本模块编译

```bash
# 编译本模块源码
./build.sh --product-name rk3568 --ccache --build-target asset

# 编译本模块测试代码
./build.sh --product-name rk3568 --ccache --build-target asset_bin_test
```

**证据**：`README_zh.md:49-57`

---

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md) - 了解模块组织
- [对外 N-API](03_NAPI_API.md) - 了解 JS API
- [编译产物](06_Build_Artifacts.md) - 了解产物详情
