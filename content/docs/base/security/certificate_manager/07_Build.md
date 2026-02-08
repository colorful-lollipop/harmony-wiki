# 构建与产物 (Build & Artifacts)

## 文档目的
本文档说明 certificate_manager 模块的 GN 构建系统、编译产物和运行时依赖。

---

## GN 目标清单

### 根构建配置
**文件**: `BUILD.gn`

### 基础组 (base_group)

| 目标 | 描述 | 产物 |
|------|------|------|
| cert_manager_type_base | 基础组件（N-API、C API、配置、证书）| 多个 .so 文件 |
| cert_manager_type_fwk | 框架组件（IPC 客户端）| libcert_manager_ipc_client.z.so |
| cert_manager_typer_services | 服务组件（SA）| libcert_manager_service.z.so |

### 详细 Targets

#### N-API 模块
**配置**: `interfaces/kits/napi/BUILD.gn`

| 目标 | 产物 | 说明 |
|------|------|------|
| certmanager | libcert_manager_napi.z.so | 主 N-API 模块 |
| certmanagerdialog | libcert_manager_dialog_napi.z.so | Dialog N-API 模块 |

#### C 接口
**配置**: `interfaces/innerkits/cert_manager_standard/main/BUILD.gn`

| 目标 | 产物 | 说明 |
|------|------|------|
| cert_manager_sdk | libcert_manager_sdk.z.so | 内部 C API 库 |

#### 服务层
**配置**: `services/cert_manager_standard/BUILD.gn`

| 目标 | 产物 | 说明 |
|------|------|------|
| cert_manager_service | libcert_manager_service.z.so | SystemAbility 主服务 |
| cert_manager_sa_profile | cert_manager_service.json | SA 配置文件 |
| hisysevent_wrapper | libcm_hisysevent_wrapper.z.so | HiSysEvent 报告封装 |
| security_guard_report | libcm_security_guard_report.z.so | 安全事件上报 |

#### 引擎层
**配置**: `services/cert_manager_standard/cert_manager_engine/BUILD.gn`

| 目标 | 产物 | 说明 |
|------|------|------|
| cert_manager_engine | libcert_manager_engine.z.so | 核心业务逻辑库 |
| cm_rdb | libcm_rdb_data_manager.z.so | 关系型数据库支持 |

#### 配置文件
**配置**: `config/BUILD.gn`

| 目标 | 产物 | 说明 |
|------|------|------|
| trusted_system_certificate0-118 | 118 个系统 CA 证书文件 | /etc/security/certificates/cert0.pem 到 cert117.pem |
| integrate_cacert | 集成 CA 证书脚本 | - |

---

## 编译产物

### 主要库文件

| 产物文件 | 路径 | 大小 | 说明 |
|----------|------|------|------|
| libcert_manager_napi.z.so | /usr/lib/ | N-API 主模块 |
| libcert_manager_dialog_napi.z.so | /usr/lib/ | N-API Dialog 模块 |
| libcert_manager_sdk.z.so | /vendor/lib64/ | 内部 C API 库 |
| libcert_manager_ipc_client.z.so | /vendor/lib64/ | IPC 客户端库 |
| libcert_manager_service.z.so | /vendor/lib64/ | SystemAbility 服务库 |
| libcert_manager_engine.z.so | /vendor/lib64/ | 核心引擎库 |
| libcm_rdb_data_manager.z.so | /vendor/lib64/ | RDB 数据库库 |
| libcm_hisysevent_wrapper.z.so | /vendor/lib64/ | HiSysEvent 封装 |
| libcm_security_guard_report.z.so | /vendor/lib64/ | 安全事件上报 |

**ROM 占用**: 5000KB
**RAM 占用**: 500KB

### 配置文件

| 文件 | 安装路径 | 说明 |
|------|---------|------|
| cert_manager_service.json | /system/profile/ | SA 配置（按需启动）|
| cert_manager_service.cfg | /system/etc/init/ | 服务初始化配置 |
| cert_manager_service.rc | /system/etc/init/ | 服务启动脚本 |

---

## 特性开关

### 功能特性
**文件**: `bundle.json`

| 特性 | 默认值 | 说明 |
|------|--------|------|
| certificate_manager_deps_huks_enabled | true | 启用 HUKS 依赖 |
| certificate_manager_feature_ca_enabled | true | 启用 CA 证书功能 |
| certificate_manager_feature_credential_enabled | true | 启用证书凭证功能 |
| certificate_manager_feature_dialog_enabled | true | 启用对话框功能 |

### 系统支持
**文件**: `bundle.json:29-32`

| 系统类型 | 说明 |
|---------|------|
| mini | 轻量级系统（精简功能）|
| small | 小型系统 |
| standard | 标准系统（完整功能）|

---

## 依赖组件

### 核心依赖

| 依赖组件 | 用途 | 最小版本 |
|---------|------|--------|
| huks | 密钥存储和密码学操作 | - |
| access_token | 访问令牌管理（权限检查）| - |
| ipc | IPC 通信框架 | - |
| samgr | 系统能力管理器 | - |
| napi | N-API 框架 | - |
| openssl | 加密库（证书解析）| - |
| relational_store | 关系型数据库 | - |

### 可选依赖

| 依赖组件 | 用途 | 最小版本 |
|---------|------|--------|
| ability_base | 基础能力框架 | - |
| ace_engine | ACE 引擎 | - |
| ability_runtime | 能力运行时 | - |
| bundle_framework | Bundle 管理 | - |
| c_utils | C 工具库 | - |
| hisysevent | HiSysEvent 日志 | - |
| hilog | HiLog 日志 | - |
| runtime_core | 运行时核心 | - |
| security_guard | 安全防护服务 | - |
| selinux_adapter | SELinux 适配 | - |

---

## 构建配置

### cert_manager.gni
**文件**: `cert_manager.gni`

关键配置：
```gni
# 存储路径定义
CM_CERT_PATH = "/data/service/el1/public/cert_manager_service/certificates"

# 最大值定义
MAX_LEN_URI = 256
MAX_LEN_CERT_ALIAS = 129
MAX_LEN_SUBJECT_NAME = 1025
MAX_COUNT_CERTIFICATE = 256
```

### BUILD.gn 关键部分

```gn
# N-API 模块
group("cert_manager_napi") {
  deps = [ "//base/security/certificate_manager/interfaces/kits/napi" ]
  external_deps = [ "libace_napi.z.so", "libhilog_ndk.z.so" ]
}

# 服务目标
group("cert_manager_typer_services") {
  deps = [
    "//base/security/certificate_manager/services/cert_manager_standard:cert_manager_service",
    "//base/security/certificate_manager/services/cert_manager_standard/cert_manager_service:cert_manager_sa_profile"
  ]
}
```

---

## 运行时加载关系

```
应用启动
    ↓
libcert_manager_napi.z.so (N-API 主模块)
    ↓
libcert_manager_dialog_napi.z.so (N-API Dialog)
    ↓
libcert_manager_sdk.z.so (内部 C API，间接依赖)
    ↓
libcert_manager_ipc_client.z.so (IPC 客户端)
    ↓
    ┌──────────────────────┐
    ↓ IPC 调度     │
    ↓                       │
libcert_manager_service.z.so (SA)
    │                       ↓
    ↓ SystemAbility 启动   │
cert_manager_service (SA ID: 3512)
    │                       ↓
    ↓ OnStart()         │
    └──────────────────────┘
libcert_manager_engine.z.so (核心业务)
    ↓ 初始化引擎
cert_manager_service (SA 内部引擎)
```

---

## 安装产物

### 系统镜像

| 目录 | 内容 |
|------|------|
| `/etc/security/certificates/` | 系统预安装 CA 证书文件 |
| `/vendor/lib64/` | 编译的 .so 库文件 |
| `/system/profile/` | SA 配置文件 |
| `/system/etc/init/` | 服务初始化脚本 |

### 用户数据

| 目录 | 内容 |
|------|------|
| `/data/service/el1/public/cert_manager_service/certificates/` | 运行时证书存储目录 |
| `/data/certificates/user_cacerts/` | 用户 CA 证书存储 |

---

## 调试支持

### 日志系统

| 日志类型 | 组件 | 用途 |
|---------|-------|------|
| HiLog | hilog | 应用日志、调试信息 |
| HiSysEvent | hisysevent_wrapper | 系统事件、安全事件上报 |

### 编译选项

| 选项 | 用途 |
|------|------|
| --enable-asan | 启用 AddressSanitizer |
| --enable-ubsan | 启用 UndefinedBehaviorSanitizer |
| --cm-coverage | 启用代码覆盖率统计 |

---

## 常见问题

### Q1: 如何启用/禁用某个特性？
**A**: 在 `bundle.json` 中修改对应的 `features` 配置。

### Q2: 如何调试 IPC 问题？
**A**: 使用 HiLog 日志和 HiSysEvent 事件追踪 IPC 调度。

### Q3: 编译失败怎么办？
**A**: 检查以下内容：
1. 依赖组件是否正常编译
2. GN 配置是否正确
3. OpenSSL 库是否可用

---

## 相关链接

- [接口文档](04_Interface.md) - 了解 API 使用方式
- [内部实现](08_Internals.md) - 深入理解代码逻辑
- [代码地图](03_CodeMap.md) - 定位构建配置

---

**适用范围**：本文档适用于 OpenHarmony 4.0 版本的 certificate_manager 模块构建系统

**最后更新**：2026-02-07
