# 项目概览

## 项目定位

TEE OS Framework (`tee_tee_os_framework`) 是 OpenHarmony TEE 解决方案的核心框架组件，提供可信执行环境（Trusted Execution Environment，TEE）的操作系统框架层。

### 在 OpenTrustee 中的位置

```
OpenHarmony (REE - Rich Execution Environment)
    │
    │ OpenHarmony Framework (N-API 层 - 独立仓库)
    │     ↓
    │ CA (Client Application) - Native C/C++
    │     ↓ SMC
    └───────────────────────────────────
       TEE OS Framework (本仓库)
       ├── TA Lifecycle Management
       ├── CA-TA Communication
       ├── Driver Management
       ├── Security Services
       │     ├── Permission Service
       │     ├── Secure Storage (SSA)
       │     └── HUK Service
       └── TEE API (GP Standard)
       
TEE Kernel (独立仓库: tee_tee_os_kernel)
```

### 核心能力

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| TA 生命周期管理 | TA 进程创建、销毁、会话管理 | `framework/gtask/` |
| SMC 命令分发 | CA 命令、系统休眠唤醒处理 | `framework/teesmcmgr/` |
| 驱动管理 | 驱动加载、权限控制、生命周期 | `framework/drvmgr/` |
| ELF 加载 | TA/驱动/服务二进制解析 | `framework/tarunner/` |
| 权限服务 | SEC 文件验签、权限校验 | `services/permission_service/` |
| 安全存储 | 数据加密、完整性保护 | `services/ssa/` |
| 硬件根密钥 | HUK 访问控制 | `services/huk_service/` |
| 加解密框架 | 驱动式加解密接口 | `drivers/crypto_mgr/` |

### 运行环境

| 属性 | 描述 |
|------|------|
| 硬件隔离 | 运行于 ARM TrustZone 安全区域 |
| 同时运行 | 与 OpenHarmony REE 并行但隔离 |
| 更高安全性 | 保护设备上的机密数据 |

## 模块职责总览

### Framework 层

| 模块 | 职责 | 代码路径 |
|------|------|----------|
| gtask | TA 进程生命周期管理、CA-TA 通信、会话管理、Agent 管理 | `framework/gtask/` |
| teesmcmgr | SMC 命令分发（CA 命令、休眠唤醒命令）、idle 状态管理 | `framework/teesmcmgr/` |
| drvmgr | 驱动生命周期管理、驱动接口访问控制、权限管理 | `framework/drvmgr/` |
| tarunner | ELF 文件加载、解析、重定位 | `framework/tarunner/` |

### Library 层

| 模块 | 职责 | 使用者 |
|------|------|--------|
| teelib | 给 TA、服务提供的库（GP API 实现） | TA、服务 |
| drvlib | 给驱动和 drvmgr 提供的库 | 驱动、驱动管理 |
| syslib | 只给 TEE 内部服务使用的库 | TEE 服务 |

### Service 层

| 模块 | 职责 | 证据 |
|------|------|------|
| permission_service | SEC 文件验签、权限控制 | `services/permission_service/src/main.c:538` (`tee_task_entry`) |
| huk_service | 硬件根密钥访问控制 | `services/huk_service/src/main.c:89` (`tee_task_entry`) |
| ssa | 安全存储服务（机密性、完整性、原子性、不可复制性） | `services/ssa/` |

### Driver 层

| 模块 | 职责 |
|------|------|
| tee_misc_driver | 基础驱动，获取 bootloader 传入的共享内存信息 |
| crypto_mgr | 加解密驱动框架 |

## 关键概念

### TA (Trusted Application)

可信应用，运行在 TEE 中的安全应用程序。

**生命周期**：
```
LOAD → OPEN_SESSION → INVOKE_COMMAND → CLOSE_SESSION → UNLOAD
```

**证据**：`lib/teelib/libtaentry/src/elf_main_entry.c`
- `ta_create_entry_point()` (line 126)
- `ta_open_session_entry_point()` (line 137)
- `ta_invoke_command_entry_point()` (line 113)
- `ta_close_session_entry_point()` (line 149)
- `ta_destroy_entry_point()` (line 159)

### CA (Client Application)

客户端应用，运行在 REE（OpenHarmony）中的原生应用，通过 SMC 调用 TEE。

### GP API

GlobalPlatform 标准的 TEE API，提供：

- 可信存储 API (`tee_trusted_storage_api.h`)
- 加解密 API (`tee_crypto_api.h`)
- 内部核心 API (`tee_internal_api.h`)

### SEC 文件

TA 的签名与配置文件，包含证书链和权限声明。

### SMC (Secure Monitor Call)

安全监控调用，是 REE 与 TEE 通信的底层机制。

## 代码目录结构

```
base/tee/tee_os_framework
├── framework/          # 核心框架
│   ├── gtask/         # TA 生命周期
│   ├── teesmcmgr/     # SMC 管理
│   ├── drvmgr/        # 驱动管理
│   └── tarunner/      # ELF 加载
├── lib/               # 库
│   ├── teelib/        # GP API 实现
│   ├── drvlib/        # 驱动库
│   └── syslib/        # 系统库
├── services/          # 服务
│   ├── permission_service/ # 权限
│   ├── huk_service/        # HUK
│   └── ssa/                # 安全存储
├── drivers/           # 驱动
│   ├── tee_misc_driver/
│   └── crypto_mgr/
├── config/            # 配置
│   ├── release_config/
│   └── debug_config/
├── build/             # 构建脚本
└── sample/            # 示例
```

## 相关文档

- 架构设计 → `01_Architecture.md`
- 模块详情 → `02_Module_Detail.md`
- API 参考 → `03_Native_API.md`
- 构建说明 → `04_Build_System.md`
- 安全评审 → `05_Security_Review.md`
