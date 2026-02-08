# 目录结构与模块职责

> 证书管理模块的代码组织结构和各模块职责说明

## 文档目的

帮助开发者理解证书管理模块的代码组织结构、各模块职责和代码组织原则。

## 适用范围

- 证书管理模块根目录：`base/security/certificate_manager/`
- 排除测试目录：`test/`

## 顶层目录树

```
base/security/certificate_manager/
├── config/                          # 系统根证书配置
│   ├── systemCertificates/            # 系统可信证书源文件
│   └── integrate_cacert/            # CA 证书集成脚本
├── frameworks/                      # 框架层代码
│   └── cert_manager_standard/        # 标准框架实现
│       └── main/
│           ├── common/               # 公共组件
│           └── os_dependency/        # OS 相关抽象层
├── interfaces/                      # 接口层代码
│   ├── innerkits/                   # 内部 C 接口
│   └── kits/                       # 对外接口
│       ├── napi/                   # Node-API (JS/TS)
│       ├── c/                      # C API
│       ├── cj/                     # CJ API
│       └── ani/                    # Ark Native Interface
├── services/                       # 服务层代码
│   └── cert_manager_standard/       # 标准服务实现
│       └── cert_manager_service/    # 证书管理服务
│       └── cert_manager_engine/     # 核心引擎
├── figures/                        # 文档图片
├── test/                          # 测试目录（本文档忽略）
├── BUILD.gn                       # 根构建文件
├── bundle.json                     # 组件配置
├── cert_manager.gni                # 全局配置
└── README.md                      # 项目说明
```

**证据来源**：`bundle.json`、`BUILD.gn`、目录扫描

## 目录职责详解

### config/ - 系统证书配置

**职责**：管理系统可信根证书的配置和集成。

**内容**：
- `systemCertificates/` - 系统预置证书源文件
- `integrate_cacert/` - CA 证书集成脚本

**关键文件**：
- `config/BUILD.gn` - 证书构建配置
- `config/integrate_cacert/BUILD.gn` - CA 集成构建

**用途**：
- 定义系统信任的根证书
- 构建时将证书打包到系统镜像

**依赖**：无（基础配置层）

### frameworks/ - 框架层

**职责**：提供基础功能组件，被 interfaces 和 services 复用。

**子目录**：
```
frameworks/cert_manager_standard/main/
├── common/                        # 公共组件
│   ├── include/                   # 公共头文件
│   └── src/                      # 公共实现
│       ├── cm_data_parcel_processor.cpp    # 数据序列化
│       ├── cm_ipc_response_type.cpp        # IPC 响应类型
│       └── cm_ukey_data_parcel_strategy.cpp  # UKey 数据策略
└── os_dependency/               # OS 抽象层
    └── cm_ipc/                  # IPC 抽象
        ├── include/
        │   ├── cm_ipc_client.h              # IPC 客户端
        │   ├── cm_ipc_client_serialization.h   # 客户端序列化
        │   ├── cm_ipc_service_serialization.h   # 服务端序列化
        │   └── cm_request.h                # 请求定义
        └── src/
            └── cm_request.cpp               # 请求处理
```

**关键组件**：
- **数据序列化**：`cm_data_parcel_processor`
  - IPC 数据编解码
  - 证书数据封装
  - 参数验证

- **IPC 抽象**：`cm_ipc_client`
  - IPC 客户端接口
  - 服务发现
  - 跨进程通信

- **响应类型**：`cm_ipc_response_type`
  - 错误码映射
  - 响应数据结构

**依赖**：OpenHarmony 基础库（ipc、utils 等）

**被依赖**：interfaces、services

### interfaces/innerkits/ - 内部 C 接口

**职责**：提供 C 语言的内部 API，供系统内部服务调用。

**结构**：
```
interfaces/innerkits/cert_manager_standard/main/
├── include/
│   ├── cert_manager_api.h         # 主 API 头文件
│   └── cm_type.h               # 类型定义
└── BUILD.gn
```

**关键 API**（`cert_manager_api.h`）：
- `CmGetCertList` - 获取证书列表
- `CmGetCertInfo` - 获取证书信息
- `CmInstallAppCert` - 安装应用证书
- `CmUninstallAppCert` - 卸载应用证书
- `CmGrantAppCertificate` - 授权证书
- `CmInit/Update/Finish/Abort` - 签名操作
- ...（详见 [04_Inner_API.md](04_Inner_API.md)）

**关键类型**（`cm_type.h`）：
- `CmBlob` - 二进制数据块
- `CmContext` - 上下文信息
- `CertInfo` - 证书信息
- `Credential` - 凭证信息
- 错误码枚举
- 存储类型枚举

**依赖**：frameworks、services（通过 IPC）

**被依赖**：interfaces/kits、其他系统服务

### interfaces/kits/napi/ - N-API（JavaScript 接口）

**职责**：提供 Node-API 绑定，供 JS/TS 应用调用。

**结构**：
```
interfaces/kits/napi/
├── include/                      # 头文件
│   ├── cm_napi_common.h         # 通用 N-API
│   ├── cm_napi_install_app_cert.h
│   ├── cm_napi_get_cert_info.h
│   └── ... (其他 API 头文件)
├── src/                         # 实现文件
│   ├── cm_napi.cpp             # 模块注册
│   ├── cm_napi_common.cpp      # 通用实现
│   ├── cm_napi_install_app_cert.cpp
│   └── ... (其他 API 实现)
└── BUILD.gn
```

**模块注册**：
- 模块名：`security.certmanager`
- 注册函数：`CMNapiRegister`
- 入口文件：`cm_napi.cpp:225`

**导出函数**（23 个，详见 [03_N-API.md](03_N-API.md)）：
- 系统证书：`getSystemTrustedCertificateList`、`getSystemTrustedCertificate`、`setCertificateStatus`
- 应用证书：`installPublicCertificate`、`uninstallPublicCertificate`、`getAllPublicCertificates`
- 用户 CA：`installUserTrustedCertificate`、`uninstallUserTrustedCertificate`、`getAllUserTrustedCertificates`
- 私钥凭证：`installPrivateCertificate`、`uninstallPrivateCertificate`、`getAllAppPrivateCertificates`
- 授权：`grantPublicCertificate`、`isAuthorizedApp`、`getAuthorizedAppList`
- 签名：`init`、`update`、`finish`、`abort`
- UKey：`getUkeyCertificateList`、`getUkeyCertificate`
- 其他：`getCertificateStorePath`、`installSystemAppCertificate`

**导出常量**：
- `CMErrorCode` - 错误码
- `CmKeyPurpose` - 密钥用途
- `CmKeyDigest` - 摘要算法
- `CmKeyPadding` - 填充方式
- `CertType` - 证书类型
- `CertScope` - 证书范围
- `CertFileFormat` - 文件格式
- `AuthStorageLevel` - 存储级别
- `CertAlgorithm` - 算法类型
- `CertificatePurpose` - 证书用途

**依赖**：
- 内部：frameworks、innerkits
- 外部：napi、ipc、os_account、samgr

**输出产物**：`libcertmanager.z.so`

### interfaces/kits/c/ - C API

**职责**：提供 C 语言的对外 API。

**依赖**：innerkits

**输出产物**：`libohcert_manager.z.so`

### interfaces/kits/cj/ - CJ API

**职责**：提供 CJ 语言绑定的 FFI 接口。

**输出产物**：`libcj_cert_manager_ffi.z.so`

### interfaces/kits/ani/ - Ark Native Interface

**职责**：提供 Ark Native Interface 绑定，供 ArkUI 应用直接调用。

**结构**：
```
interfaces/kits/ani/
├── certificate_manager_ani/       # 主证书管理 ANI
├── cm_ani_common/              # ANI 公共组件
└── certificate_manager_dialog_ani/  # 对话框 ANI
```

**关键组件**：
- `cm_ani_common/impl/` - ANI 实现基类
- `cm_ani_common/builder/` - 结果构建器
- `cm_ani_common/utils/` - 工具函数

### services/cert_manager_standard/ - 服务层

**职责**：实现证书管理的核心服务和业务逻辑。

**结构**：
```
services/cert_manager_standard/
├── cert_manager_service/         # 服务端
│   └── main/
│       ├── os_dependency/
│       │   ├── sa/              # 系统能力（SA）配置
│       │   │   └── sa_profile/
│       │   │       └── cert_manager_service.json
│       │   ├── idl/            # IDL 定义
│       │   └── ...            # 其他 OS 依赖
│       ├── hisysevent_wrapper/    # HiSysEvent 封装
│       ├── security_guard_report/  # SecurityGuard 上报
│       └── BUILD.gn
└── cert_manager_engine/          # 核心引擎
    └── main/
        ├── core/                # 核心功能
        │   ├── include/
        │   │   ├── cert_manager.h              # 主引擎
        │   │   ├── cert_manager_service.h      # 服务接口
        │   │   ├── cert_manager_permission_check.h  # 权限检查
        │   │   ├── cert_manager_storage.h       # 存储管理
        │   │   ├── cert_manager_key_operation.h  # 密钥操作
        │   │   ├── cert_manager_auth_mgr.h      # 授权管理
        │   │   └── ... (其他核心组件)
        │   └── src/
        │       ├── cert_manager.cpp
        │       ├── cert_manager_permission_check.cpp
        │       └── ... (其他实现)
        └── rdb/                  # 数据库层
            ├── include/
            │   ├── cm_rdb_data_manager.h    # RDB 管理器
            │   ├── cm_rdb_open_callback.h    # 打开回调
            │   └── cm_cert_property_rdb.h    # 证书属性 RDB
            └── src/
                ├── cm_rdb_data_manager.cpp
                ├── cm_rdb_open_callback.cpp
                └── cm_cert_property_rdb.cpp
```

### services/cert_manager_service/ - 服务端

**职责**：实现 SystemAbility，提供 IPC 服务接口。

**SA 配置**（`cert_manager_service.json`）：
- SA ID：`3512`
- 库名：`libcert_manager_service.z.so`
- 进程名：`cert_manager_service`
- 启动策略：按需启动（commonevent 触发）
- 自动重启：`true`
- 回收策略：`low-memory`

**依赖**：safwk、samgr

### services/cert_manager_engine/ - 核心引擎

**职责**：实现证书管理的核心业务逻辑。

**核心模块**：

#### 权限检查（`cert_manager_permission_check.h/cpp`）
**功能**：
- `CmHasPrivilegedPermission` - 特权检查
- `CmHasCommonPermission` - 通用权限检查
- `CmHasEnterpriseUserTrustedPermission` - 企业权限
- `CmHasUserTrustedPermission` - 用户 CA 权限
- `CmHasSystemAppPermission` - 系统应用权限
- `CmIsSystemApp` - 系统应用判断
- `CmPermissionCheck` - 综合权限检查

**关键机制**：
- 基于 Access Token 鉴权
- 调用 UID 和 UserID 验证
- 存储类型权限映射

**证据来源**：`cert_manager_permission_check.h:25-41`

#### 存储管理（`cert_manager_storage.h/cpp`）
**功能**：
- 证书文件存储
- 目录隔离（UserID + UID）
- 文件读写操作
- 路径管理

**存储路径**：
- 系统证书：`/etc/security/certificates`
- 用户证书：`/data/service/el1/public/cert_manager_service/certificates/`

#### 密钥操作（`cert_manager_key_operation.h/cpp`）
**功能**：
- HUKS 密钥导入/导出
- 密钥删除
- 密钥验证
- 密钥属性查询

**依赖**：HUKS 模块

#### 授权管理（`cert_manager_auth_mgr.h/cpp`）
**功能**：
- 证书授权管理
- 授权列表维护
- 授权状态查询
- 授权撤销

#### 会话管理（`cert_manager_session_mgr.h/cpp`）
**功能**：
- 签名会话管理
- 多步操作（init/update/finish）
- 会话状态维护
- 超时处理

#### 证书查询（`cert_manager_query.h/cpp`）
**功能**：
- 证书列表查询
- 证书信息查询
- 条件过滤
- 结果排序

#### 文件操作（`cert_manager_file_operator.h/cpp`）
**功能**：
- 证书文件读取
- 格式解析（PEM/DER/P7B）
- 证书链处理
- 编码转换

#### 数据库层（rdb/）
**功能**：
- 证书属性存储
- 元数据管理
- 快速查询
- 事务处理

**表结构**：
- 证书属性表
- 授权关系表
- 状态标志表

**依赖**：relational_store（RDB）

## 代码组织原则

### 1. 分层架构

```
Interfaces (对外 API)
    ↓
Innerkits (内部接口)
    ↓
Frameworks (公共组件)
    ↓
Services (服务实现)
    ↓
Engine (核心引擎)
```

### 2. 依赖方向

**原则**：上层依赖下层，避免循环依赖。

**依赖链**：
- kits → innerkits → frameworks → services/engine
- 无反向依赖

### 3. 接口稳定性

**稳定接口**（版本兼容）：
- `interfaces/innerkits/cert_manager_standard/main/include/` - Innerkits
- `interfaces/kits/napi/` - N-API

**内部接口**（可变）：
- `frameworks/` 内部
- `services/` 内部

### 4. 模块化设计

每个模块职责单一：
- `cert_manager_permission_check` - 只负责权限检查
- `cert_manager_storage` - 只负责存储
- `cert_manager_key_operation` - 只负责密钥操作

## 关键文件索引

| 功能模块 | 关键文件 | 说明 |
|---------|----------|------|
| N-API 注册 | `interfaces/kits/napi/src/cm_napi.cpp` | 模块入口 |
| Inner API | `interfaces/innerkits/.../cert_manager_api.h` | 内部接口 |
| IPC 客户端 | `frameworks/.../cm_ipc_client.h` | IPC 抽象 |
| IPC 接口码 | `frameworks/.../cert_manager_service_ipc_interface_code.h` | 消息码定义 |
| SA 配置 | `services/.../sa_profile/cert_manager_service.json` | 服务配置 |
| 权限检查 | `services/.../cert_manager_permission_check.h` | 权限验证 |
| 核心引擎 | `services/.../cert_manager.h` | 主引擎 |
| 数据库 | `services/.../cm_rdb_data_manager.h` | RDB 管理 |
| 类型定义 | `interfaces/innerkits/.../cm_type.h` | 类型定义 |

## 相关跳转

- [项目概述](00_Overview.md)
- [架构说明](02_Architecture.md)
- [N-API 接口文档](03_N-API.md)
- [内部 API 文档](04_Inner_API.md)

---

*更新时间：2026-02-06*
