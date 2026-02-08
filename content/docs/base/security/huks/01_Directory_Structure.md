# 目录结构与模块职责

> HUKS 项目的代码组织和各模块职责说明

**目的**: 了解代码目录结构、各模块职责和关键文件
**适用范围**: 所有人（新人、应用开发者、系统开发者）
**相关文档**: [项目概览](./00_Overview.md) | [架构说明](./02_Architecture.md)

---

## 1. 顶层目录结构

```
/Volumes/lexar/code/d/work/oh/base/security/huks/
├── build/                              # 编译配置文件
├── etc/                                # 运行时配置文件
├── figures/                            # 文档图片
├── frameworks/                         # 框架代码
├── interfaces/                         # 接口 API 代码
├── services/                           # 系统服务代码
├── utils/                              # 工具代码
└── wiki/                               # Wiki 文档（本目录）
```

**证据**: `README_zh.md:31-45`

---

## 2. 目录职责说明

### 2.1 build/ - 编译配置

**职责**: GN 构建系统配置和模板

| 文件/目录 | 说明 |
|-----------|------|
| `config.gni` | 主要 feature flags 和构建参数 |
| `BUILD.gn` | 构建配置模板 |

**证据**: `build/config.gni`

### 2.2 etc/ - 配置文件

**职责**: 运行时参数和配置

| 文件/目录 | 说明 |
|-----------|------|
| `huks.para` | 启动事件参数 |
| `huks.para.dac` | DAC 权限参数 |

**证据**: `etc/BUILD.gn`

### 2.3 figures/ - 文档图片

**职责**: 存放架构图等文档图片

| 文件/目录 | 说明 |
|-----------|------|
| `huks_architecture.png` | HUKS 架构图 |

**证据**: `README_zh.md:25`

### 2.4 frameworks/ - 框架代码

**职责**: 核心框架实现，作为基础功能被 interfaces 和 services 使用

```
frameworks/
├── huks_standard/                    # 标准系统框架
│   └── main/
│       ├── common/                  # 通用工具和类型
│       ├── core/                    # 核心框架逻辑
│       ├── crypto_engine/           # 加密引擎（mbedtls/openssl）
│       └── os_dependency/          # OS 抽象层
├── huks_lite/                      # 轻量系统框架
└── crypto_lite/                     # 轻量加密实现
    ├── cipher/                      # AES/RSA 密码实现
    └── js/                          # JS API（轻量系统）
```

**证据**: `README_zh.md:33-36`

#### frameworks/huks_standard/main/

**职责**: 标准系统框架核心实现

| 子目录 | 职责 | 关键文件 |
|--------|------|---------|
| `common/` | 通用类型定义、工具函数 | `hks_config.h`, `hks_log.h` |
| `core/` | 核心逻辑（本地引擎、验证器） | `hks_local_engine.c`, `hks_keyblob.c` |
| `crypto_engine/` | 加密引擎封装 | `hks_crypto_adapter.c` |
| `os_dependency/` | OS 抽象层 | `hks_mem.c`, `hks_mutex.c` |

### 2.5 interfaces/ - 接口代码

**职责**: 提供 API 给应用和其他模块使用

```
interfaces/
├── inner_api/                       # 内部 API（C/C++）
│   ├── huks_lite/                   # 轻量系统内部 API
│   └── huks_standard/               # 标准系统内部 API
│       └── main/include/            # 核心头文件
│           ├── hks_api.h           # 主 API 定义
│           ├── hks_type.h         # 类型定义
│           └── hks_param.h        # 参数处理
├── kits/                            # 对外 API
│   ├── c/                          # Native C API (NDK)
│   ├── cj/                         # Cangjie FFI
│   ├── liteapi/                    # 轻量系统 API
│   └── napi/                       # JS/TS Node-API
└── js/                              # JavaScript 模块
    └── crypto_extension_module/       # 加密扩展模块
```

**证据**: `README_zh.md:36-38`

#### interfaces/kits/napi/

**职责**: JS/TS Node-API 实现

| 子目录 | 职责 |
|--------|------|
| `src/` | N-API 实现代码 |
| `v8/` | V8 API 实现（旧版）|
| `v9/` | V9 API 实现（KeyItem 操作）|
| `v12/` | V12 API 实现（AsUser 操作）|
| `ukey/` | UKey 扩展实现 |

**证据**: `interfaces/kits/napi/BUILD.gn`

### 2.6 services/ - 系统服务代码

**职责**: HUKS 系统服务实现

```
services/huks_standard/
├── huks_engine/                     # HUKS 核心层
│   └── main/
│       ├── core/                    # 引擎核心逻辑
│       └── core_dependency/         # HAL API
└── huks_service/                    # HUKS 服务层
    └── main/
        ├── core/                    # 服务核心（密钥管理、存储）
        ├── os_dependency/          # OS 依赖
        │   ├── idl/               # IPC 接口定义
        │   ├── posix/             # POSIX 兼容
        │   └── sa/                # System Ability 框架
        ├── systemapi_wrap/          # 系统 API 包装
        │   ├── at_wrapper/        # Access Token 包装
        │   ├── bms/               # Bundle 管理包装
        │   ├── dcm/               # 设备证书管理包装
        │   ├── hisysevent_wrapper/ # HiSysEvent 包装
        │   ├── hitrace_meter_wrapper/ # HiTrace 包装
        │   └── useridm/           # 用户 ID 管理
        ├── upgrade/                 # 密钥升级功能
        └── extension/               # 扩展模块
            ├── ukey/               # UKey 支持
            │   ├── common/
            │   ├── app_observer/
            │   ├── session_manger/
            │   ├── handle_manager/
            │   ├── ext_life_cycle_manger/
            │   └── connection/
            ├── ability_native/      # 加密扩展能力
            ├── module_loader/       # 插件模块加载器
            └── ukey_client_service/ # UKey 客户端服务
```

**证据**: `README_zh.md:39-42`

#### services/huks_standard/huks_service/main/core/

**职责**: 服务核心逻辑

| 关键文件 | 职责 |
|-----------|------|
| `hks_client_check.c` | 客户端参数检查、UID 白名单 |
| `hks_client_service.c` | 客户端服务主逻辑 |
| `hks_session_manager.c` | 会话管理 |
| `hks_storage_manager.c` | 存储管理 |
| `huks_access.c` | 访问控制 |

**证据**: `services/huks_standard/huks_service/main/core/BUILD.gn`

### 2.7 utils/ - 工具代码

**职责**: 共享工具库

```
utils/
├── crypto_adapter/                   # 加密适配器层
├── file_operator/                   # 文件操作
├── list/                           # 链表实现
├── mutex/                          # 互斥锁
├── condition/                      # 条件变量
└── file_iterative_reader/           # 文件迭代读取器
```

**证据**: `utils/*/BUILD.gn`

---

## 3. 模块依赖关系

### 3.1 依赖方向

```
应用层 (JS/TS/C)
    ↓
interfaces/kits (N-API/C-API)
    ↓
interfaces/inner_api (libhukssdk)
    ↓
services/huks_service (SA 3510)
    ↓
services/huks_engine / frameworks/crypto_engine
    ↓
openssl / mbedtls
```

**证据**: `BUILD.gn`, `bundle.json:68-95`

### 3.2 主要依赖

| 模块 | 依赖 | 说明 |
|-----|------|------|
| `interfaces/kits/napi/` | `interfaces/inner_api/` | 内部 API |
| `interfaces/inner_api/` | `frameworks/huks_standard/`, `utils/` | 框架和工具 |
| `services/huks_service/` | `interfaces/inner_api/`, `frameworks/huks_standard/` | 框架和内部 API |
| `services/huks_engine/` | `frameworks/huks_standard/` | 框架 |
| `frameworks/huks_standard/` | `openssl` / `mbedtls` | 加密库 |

---

## 4. 关键文件索引

### 4.1 核心头文件

| 文件 | 路径 | 说明 |
|-----|------|------|
| `hks_api.h` | `interfaces/inner_api/huks_standard/main/include/` | 主 API 定义 |
| `hks_type.h` | `interfaces/inner_api/huks_standard/main/include/` | 类型定义 |
| `hks_param.h` | `interfaces/inner_api/huks_standard/main/include/` | 参数处理 |
| `hks_error_code.h` | `interfaces/inner_api/huks_standard/main/include/` | 错误码 |
| `hks_tag.h` | `interfaces/inner_api/huks_standard/main/include/` | 参数标签 |

### 4.2 N-API 实现

| 文件 | 路径 | 说明 |
|-----|------|------|
| `huks_napi.cpp` | `interfaces/kits/napi/src/` | 主 N-API 模块注册 |
| `huks_napi_ukey_module.cpp` | `interfaces/kits/napi/src/` | UKey 模块注册 |
| `huks_napi_common_item.h` | `interfaces/kits/napi/include/v9/` | 通用工具函数和宏 |

### 4.3 系统服务

| 文件 | 路径 | 说明 |
|-----|------|------|
| `hks_sa.h` | `services/huks_standard/huks_service/main/os_dependency/sa/` | System Ability 定义 |
| `hks_sa.cpp` | `services/huks_standard/huks_service/main/os_dependency/sa/` | SA 实现 |
| `hks_sa_interface.h` | `services/huks_standard/huks_service/main/os_dependency/sa/` | IPC 接口 |
| `hks_permission_check.cpp` | `services/huks_standard/huks_service/main/os_dependency/idl/ipc/` | 权限检查 |

### 4.4 IPC 接口

| 文件 | 路径 | 说明 |
|-----|------|------|
| `hks_service_ipc_interface_code.h` | `frameworks/huks_standard/main/common/include/` | IPC 接口码定义 |
| `hks_message_handler.h` | `services/huks_standard/huks_service/main/os_dependency/sa/` | 消息处理器 |

---

## 5. 代码组织原则

### 5.1 分层原则

- **interfaces/**: 向外暴露的 API
- **frameworks/**: 框架层实现，被 interfaces 和 services 使用
- **services/**: 系统服务实现
- **utils/**: 工具库，被各层共享

### 5.2 模块化原则

- 按功能模块划分子目录
- 每个模块有独立的 BUILD.gn
- 模块间通过接口通信，尽量减少耦合

### 5.3 系统适配原则

- `huks_standard/`: 标准系统完整功能
- `huks_lite/`: 轻量系统裁剪功能
- `crypto_lite/`: 轻量加密实现

---

## 6. 快速定位

### 6.1 查找 N-API 实现

**路径**: `interfaces/kits/napi/src/`
- 主模块: `huks_napi.cpp`
- UKey 模块: `huks_napi_ukey_module.cpp`
- 具体实现: `v8/`, `v9/`, `v12/`, `ukey/` 子目录

### 6.2 查找权限检查

**路径**: `services/huks_standard/huks_service/main/os_dependency/idl/ipc/`
- 主文件: `hks_permission_check.cpp`
- 关键函数: `SensitivePermissionCheck()`, `SystemApiPermissionCheck()`

### 6.3 查找 IPC 通信

**路径**: `services/huks_standard/huks_service/main/os_dependency/sa/`
- 主文件: `hks_sa.cpp`, `hks_sa_interface.cpp`
- 配置: `sa_profile/3510.json`

### 6.4 查找密钥操作

**路径**: `services/huks_standard/huks_service/main/core/`
- 主文件: `hks_client_service.c`
- 存储管理: `hks_storage_manager.c`
- 会话管理: `hks_session_manager.c`

---

## 7. 相关文档

- [项目概览](./00_Overview.md) - HUKS 定位和核心能力
- [架构说明](./02_Architecture.md) - 架构设计和数据流
- [对外 API](./03_External_API.md) - API 接口详细说明
- [内部 API](./04_Internal_API.md) - 模块接口和依赖关系
