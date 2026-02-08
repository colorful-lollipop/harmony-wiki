# GN Targets 梳理

> HUKS 项目的 GN 构建目标详细说明

**目的**: 了解 HUKS 的构建目标、依赖关系和编译产物
**适用范围**: 构建维护、系统开发者
**相关文档**: [目录结构](./01_Directory_Structure.md) | [编译产物](./06_Build_Artifacts.md)

---

## 1. 构建入口

### 1.1 根构建文件

**文件**: `BUILD.gn`

**主要 Groups**:

| Target | 类型 | 说明 |
|--------|------|------|
| `huks_sdk_test` | group | 测试套件入口 |
| `huks_capi` | group | C API (NDK) 入口 |
| `huks_napi` | group | JS/TS NAPI 入口 |
| `huks_cjapi` | group | Cangjie FFI 入口 |
| `cipher_napi` | group | 加密 NAPI 入口 |
| `huks_crypto_extension` | group | UKey 扩展入口 |
| `fwk_group` | group | 框架层汇总 |
| `service_group` | group | 服务层汇总 |
| `huks_components` | group | 完整组件汇总 |

**证据**: `BUILD.gn`

---

## 2. 关键 Targets

### 2.1 框架层 Targets

**文件**: `frameworks/huks_standard/main/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `huks_standard_frameworks` | group | - | 框架层汇总 |
| `libhuks_common_standard_static` | static_library | - | 通用库 |
| `libhuks_core_standard_static` | static_library | - | 核心库 |
| `libhuks_crypto_engine_standard_static` | static_library | - | 加密引擎 |
| `libhuks_mem_standard_static` | static_library | - | 内存管理 |
| `libhuks_os_dependency_standard_static` | static_library | - | OS 依赖 |

**证据**: `frameworks/huks_standard/main/*/BUILD.gn`

### 2.2 接口层 Targets

#### C API (NDK)

**文件**: `interfaces/kits/c/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `huks_ndk` | ohos_shared_library | `libhuks_ndk.so` | C API 库 |
| `huks_external_crypto` | ohos_shared_library | `libhuks_external_crypto.so` | 外部加密库 |

**证据**: `interfaces/kits/c/BUILD.gn`

#### N-API

**文件**: `interfaces/kits/napi/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `huks` | ohos_shared_library | `libhuks.so` | 主 NAPI 库 |
| `huksexternalcrypto_napi` | ohos_shared_library | `libhuksnapi.so` | UKey NAPI 库 |

**证据**: `interfaces/kits/napi/BUILD.gn`

#### 内部 API

**文件**: `interfaces/inner_api/huks_standard/main/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libhukssdk` | ohos_shared_library | `libhukssdk.so` | 内部 SDK |
| `libhukschipsetsdk` | ohos_shared_library | `libhukschipsetsdk.so` | 芯片 SDK |

**证据**: `interfaces/inner_api/huks_standard/main/BUILD.gn`

### 2.3 服务层 Targets

#### 主服务

**文件**: `services/huks_standard/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `huks_service` | ohos_shared_library | `libhuksservice.so` | 系统服务库 |
| `huks_service.rc` | ohos_prebuilt_etc | `huks_service.cfg` | 服务配置 |

**证据**: `services/huks_standard/BUILD.gn`

#### 服务组件

**文件**: `services/huks_standard/huks_service/main/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libhuks_service_standard_static` | static_library | - | 服务核心 |
| `hks_compatibility_bin` | ohos_executable | `hks_compatibility_bin` | 兼容性二进制 |

**证据**: `services/huks_standard/huks_service/main/BUILD.gn`

#### 扩展

**文件**: `services/huks_standard/huks_service/extension/ukey/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libhuks_external_crypto_ext_core` | ohos_shared_library | `libhuks_external_crypto_ext_core.so` | UKey 扩展核心 |

**证据**: `services/huks_standard/huks_service/extension/ukey/BUILD.gn`

### 2.4 引擎层 Targets

**文件**: `services/huks_standard/huks_engine/main/core/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `huks_engine_core_standard` | ohos_shared_library | `libhuks_engine_core_standard.so` | 引擎核心 |

**证据**: `services/huks_standard/huks_engine/main/core/BUILD.gn`

---

## 3. 依赖关系

### 3.1 依赖图

```
huks_components (group)
├── fwk_group (group)
│   ├── huks_cjapi → cj_huks_ffi (.so)
│   ├── cipher_napi → cipher_napi (.so)
│   ├── huks_capi → huks_ndk (.so) [+ huks_external_crypto]
│   ├── huks_napi → huks (.so) [+ huksexternalcrypto_napi]
│   └── huks_standard_frameworks (group)
│       ├── libhuks_common_standard_static
│       ├── libhuks_core_standard_static
│       ├── libhuks_crypto_engine_standard_static
│       ├── libhuks_mem_standard_static
│       └── libhuks_os_dependency_standard_static
└── service_group (group)
    ├── huks_service (.so)
    │   └── libhuks_service_standard_static
    ├── huks_sa_profile
    └── [可选] libhuks_external_crypto_ext_core (.so)
```

**证据**: `BUILD.gn`, `bundle.json`

### 3.2 主要依赖

| 模块 | 依赖 | 说明 |
|-----|------|------|
| `interfaces/kits/napi/` | `interfaces/inner_api/` | 内部 API |
| `interfaces/inner_api/` | `frameworks/huks_standard/`, `utils/` | 框架和工具 |
| `services/huks_service/` | `interfaces/inner_api/`, `frameworks/huks_standard/` | 框架和内部 API |
| `services/huks_engine/` | `frameworks/huks_standard/` | 框架 |
| `frameworks/huks_standard/` | `openssl` / `mbedtls` | 加密库 |

---

## 4. Feature Flags

### 4.1 主要 Feature Flags

**文件**: `build/config.gni`, `huks.gni`

| Flag | 默认值 | 说明 |
|------|--------|------|
| `huks_enabled` | true | 启用 HUKS |
| `huks_use_mbedtls` | true | 使用 mbedtls 引擎 |
| `use_crypto_lib` | "openssl" | 加密库选择 |
| `huks_use_hardware_root_key` | false | 使用硬件根密钥 |
| `huks_enable_log` | true | 启用日志 |
| `huks_enable_upgrade_key` | true | 启用密钥文件自动升级 |
| `huks_security_level` | "software" | 安全级别 |
| `huks_use_rkc_in_standard` | false | 标准系统使用 RKC |
| `huks_enable_hdi_in_standard` | true | 启用 HDI |
| `huks_enable_ukey_config` | false | 启用 UKey（PC端） |

**证据**: `build/config.gni`, `huks.gni`

### 4.2 配置 Defines

| Define | 说明 |
|--------|------|
| `L2_STANDARD` | 标准系统 |
| `HKS_L1_SMALL` | 小型系统 |
| `_HARDWARE_ROOT_KEY_` | 硬件根密钥 |
| `_HUKS_LOG_ENABLE_` | 日志启用 |
| `HKS_ENABLE_UPGRADE_KEY` | 密钥升级 |

---

## 5. 编译产物映射

### 5.1 动态库 (.so)

| Target | 输出文件 | 安装路径 |
|--------|---------|---------|
| `huks_ndk` | `libhuks_ndk.so` | system/lib |
| `huks_external_crypto` | `libhuks_external_crypto.so` | system/lib |
| `huks` (NAPI) | `libhuks.so` | system/lib/module/security |
| `huksexternalcrypto_napi` | `libhuksnapi.so` | system/lib/module/security |
| `cj_huks_ffi` | `libcj_huks_ffi.so` | system/lib |
| `libhukssdk` | `libhukssdk.so` | system/lib, updater/lib |
| `libhukschipsetsdk` | `libhukschipsetsdk.so` | system/lib, updater/lib |
| `huks_service` | `libhuksservice.so` | system/lib |
| `huks_engine_core_standard` | `libhuks_engine_core_standard.so` | system/lib |
| `libhuks_external_crypto_ext_core` | `libhuks_external_crypto_ext_core.so` | system/lib |

**证据**: `bundle.json`, `BUILD.gn`

### 5.2 可执行文件

| Target | 输出文件 | 说明 |
|--------|---------|------|
| `hks_compatibility_bin` | `hks_compatibility_bin` | 兼容性二进制工具 |
| `huks_server` | `huks_server` | 服务进程（轻量系统）|

**证据**: `services/huks_standard/huks_service/main/BUILD.gn`

### 5.3 配置文件

| Target | 输出文件 | 安装路径 |
|--------|---------|---------|
| `huks_service.rc` | `huks_service.cfg` | system/etc/init |
| `huks.para` | `huks.para` | system/etc/param |
| `huks.para.dac` | `huks.para.dac` | system/etc/param |
| `huks_sa_profile` | `3510.json` | SA 配置 |

**证据**: `etc/BUILD.gn`, `services/huks_standard/huks_service/main/os_dependency/sa/sa_profile/BUILD.gn`

---

## 6. 相关文档

- [目录结构](./01_Directory_Structure.md) - 代码组织和模块职责
- [编译产物](./06_Build_Artifacts.md) - 编译产物和安装路径
