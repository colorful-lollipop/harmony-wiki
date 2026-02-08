# SAFWK 构建系统

## GN 构建概述

SAFWK 使用 OpenHarmony 的 GN (Generate Ninja) 构建系统。

### 构建入口

| 文件 | 位置 | 描述 |
|------|------|------|
| `config.gni` | 根目录 | 全局配置 (特征开关) |
| `var.gni` | 根目录 | 构建变量定义 |
| `bundle.json` | 根目录 | 组件清单 |
| `BUILD.gn` | 各子目录 | 具体构建目标 |

### 特征开关

**文件**: `config.gni`

| 开关 | 默认值 | 描述 |
|------|--------|------|
| `safwk_feature_coverage` | false | 启用代码覆盖率 (gcov) |
| `safwk_enable_run_on_demand_qos` | false | 按需启动时更新线程优先级 |
| `safwk_feature_support_saspawn` | false | 支持 sa_spawn 模式 |

---

## 核心 Targets

### interfaces/innerkits/safwk/BUILD.gn

#### system_ability_fwk (共享库)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_shared_library |
| **输出** | `libsystem_ability_fwk.z.so` |
| **安装路径** | `system/lib/platformsdk/` |

**源文件** (`services/safwk/src/`):

| 文件 | 描述 |
|------|------|
| ffrt_handler.cpp | FFRT 任务处理 |
| local_ability_manager.cpp | LAM 实现 |
| local_ability_manager_dumper.cpp | Dump 功能 |
| local_ability_manager_stub.cpp | IPC Stub |
| system_ability.cpp | SA 基类 |
| system_ability_ondemand_reason.cpp | 按需原因 |

**外部依赖** (`external_deps`):

| 依赖组件 | 库 | 用途 |
|----------|------|------|
| access_token | libaccesstoken_sdk | 权限验证 |
| c_utils | utils | C 工具库 |
| ffrt | libffrt | 任务调度 |
| hilog | libhilog | 日志 |
| hitrace | hitrace_meter | 跟踪 |
| ipc | ipc_core | IPC 框架 |
| json | nlohmann_json_static | JSON 解析 |
| samgr | samgr_common | 系统管理 |
| samgr | samgr_proxy | 代理 |

**配置**:

| 配置 | 值 |
|------|-----|
| CFI | 启用 |
| PAC-RET | 启用 |
| 版本脚本 | `libsystem_ability_fwk.versionscript` |
| 条件定义 | `SAFWK_ENABLE_RUN_ON_DEMAND_QOS` (条件) |

#### api_cache_manager (共享库)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_shared_library |
| **输出** | `libapi_cache_manager.z.so` |
| **安装路径** | `system/lib/platformsdk/` |

**源文件**:
- api_cache_manager.cpp

**外部依赖**:
- c_utils:utils
- hilog:libhilog
- ipc:ipc_single

#### system_ability_ondemand_reason (静态库)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_static_library |
| **输出** | `.a` 静态库 |
| **源文件** | system_ability_ondemand_reason.cpp |

**外部依赖**:
- c_utils
- json
- samgr_common

---

### services/safwk/BUILD.gn

#### sa_main (可执行文件)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_executable |
| **输出** | `sa_main` |
| **安装** | 启用 |

**源文件**:
- main.cpp

**内部依赖**:
- `:system_ability_fwk` (innerkit)

**外部依赖**:
- c_utils:utils
- hilog:libhilog
- init:libbegetutil
- ipc:ipc_single
- json:nlohmann_json_static
- samgr:samgr_common

**链接参数**:
- `-rdynamic`
- max-page-size=4096
- separate-code

#### sa_start (共享库) [条件构建]

| 属性 | 值 |
|------|-----|
| **类型** | ohos_shared_library |
| **输出** | `libsa_start.z.so` |
| **安装路径** | `system/lib/platformsdk/` |
| **条件** | `safwk_feature_support_saspawn = true` |

**源文件**:
- system_ability_start.cpp

**依赖**:
- `:system_ability_fwk`

#### sa_start_group (组) [条件]

包含 `sa_start` 目标，条件同上。

---

### svc/BUILD.gn

#### svc (可执行文件)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_executable |
| **输出** | `svc` |
| **安装** | 启用 |

**源文件**:
- main.cpp
- svc_control.cpp

**外部依赖**:
- c_utils:utils
- hilog:libhilog
- ipc:ipc_single
- json:nlohmann_json_static
- samgr:samgr_common
- samgr:samgr_proxy

---

### Rust 绑定构建

**文件**: `interfaces/innerkits/safwk/rust/BUILD.gn`

#### system_ability_fwk_rust_gen (rust_cxx)

- **输出**: 生成的 C++ 代码
- **用途**: 从 Rust 的 `#[cxx::bridge]` 生成 FFI 代码

#### system_ability_fwk_rust_cxx (静态库)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_static_library |
| **输出** | `.a` 静态库 |

**源文件**:
- `system_ability_wrapper.cpp` (C++ FFI)
- 生成的 CXX 代码

#### system_ability_fwk_rust (Rust 共享库)

| 属性 | 值 |
|------|-----|
| **类型** | ohos_rust_shared_library |
| **输出** | `libsystem_ability_fwk.so` |
| **Crate 名称** | system_ability_fwk |

**Rustflags**:
- `-Zstack-protector=all`

**依赖**:
- `:system_ability_fwk_rust_cxx`

---

### 配置构建

**文件**: `etc/profile/BUILD.gn`

#### foundation_cfg

| 属性 | 值 |
|------|-----|
| **类型** | ohos_prebuilt_etc |
| **源** | `foundation.cfg` |
| **安装** | `system/etc/init/` |

#### foundation_trust

| 属性 | 值 |
|------|-----|
| **类型** | ohos_prebuilt_etc |
| **源** | `foundation_trust.json` |
| **安装** | `system/profile/` |

---

## 产物清单

### 运行时产物

| 产物 | 类型 | 源 Target | 安装路径 |
|------|------|----------|----------|
| `libsystem_ability_fwk.z.so` | 共享库 | system_ability_fwk | system/lib/platformsdk/ |
| `libapi_cache_manager.z.so` | 共享库 | api_cache_manager | system/lib/platformsdk/ |
| `libsa_start.z.so` | 共享库 | sa_start (条件) | system/lib/platformsdk/ |
| `libsystem_ability_fwk.so` | 共享库 | system_ability_fwk_rust | system/lib/ |
| `sa_main` | 可执行 | sa_main | system/bin/ |
| `svc` | 可执行 | svc | system/bin/ |

### 配置产物

| 产物 | 安装路径 |
|------|----------|
| `foundation.cfg` | system/etc/init/ |
| `foundation_trust.json` | system/profile/ |

### 静态库 (构建中间产物)

| 产物 | 源 Target |
|------|----------|
| `libsystem_ability_fwk_rust_cxx.a` | system_ability_fwk_rust_cxx |
| `libsystem_ability_ondemand_reason.a` | system_ability_ondemand_reason |

---

## 运行时加载关系

```
system/bin/sa_main
    │
    ├── dlopen() → libsystem_ability_fwk.z.so
    │       │
    │       ├── dlopen() → libsystem_ability_fwk.so (Rust)
    │       │
    │       └── dlopen() → libapi_cache_manager.z.so
    │
    └── dlopen() → libsa_start.z.so (条件)
```

```
system/bin/svc
    │
    └── dlopen() → libsystem_ability_fwk.z.so
            └── dlopen() → libapi_cache_manager.z.so
```

---

## 编译命令

### 全量编译

```bash
# 在 OpenHarmony 根目录
./build.sh --product <product_name>
```

### 单独编译 SAFWK

```bash
# 编译整个 safwk 子系统
hb build -p systemabilitymgr/safwk
```

### 增量编译

```bash
# 编译单个模块
hb build -p systemabilitymgr/safwk -T <target_name>
```

---

## bundle.json 入口

**文件**: `bundle.json`

### Base Group

| Target | 描述 |
|--------|------|
| `foundation_cfg` | Foundation 配置 |
| `foundation_trust` | Trust 配置 |
| `sa_main` | SAFWK 守护进程 |
| `svc` | 服务控制工具 |
| `system_ability_fwk` | 核心框架库 |
| `sa_start_group` | SA 启动组 (条件) |

### Inner Kits

| Target | 头文件 |
|--------|--------|
| `system_ability_fwk` | system_ability.h |
| `system_ability_fwk_rust` | Rust 接口 |
| `system_ability_ondemand_reason` | system_ability_ondemand_reason.h |
| `api_cache_manager` | api_cache_manager.h |

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [01_Architecture.md](01_Architecture.md) |
| API 接口 | [02_APIs.md](02_APIs.md) |
| 安全评审 | [04_Security.md](04_Security.md) |
| 常见问题 | [05_FAQ.md](05_FAQ.md) |
