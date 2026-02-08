# 模块详解

## 1. Framework 层

### 1.1 gtask - TA 生命周期管理

**代码路径**: `framework/gtask/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| TA 进程管理 | 创建、销毁 TA 进程 | `src/app_load/tee_app_load_srv.c` |
| CA-TA 通信 | 接收 SMC 命令，分发到 TA | `src/framework/tee_ns_cmd_dispatch.c` |
| 会话管理 | 打开/关闭会话，管理会话上下文 | `src/manager/session_manager.c` |
| Agent 管理 | 锁定/解锁共享资源 Agent | `src/manager/agent_manager.c` |
| 服务注册 | TA 服务注册和查找 | `src/manager/service_manager.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/init/main.c` | 初始化 IPC 通道、定时器、文件 I/O |
| `src/framework/global_task.c` | gtask 主事件循环 |
| `src/framework/tee_ns_cmd_dispatch.c` | CA 命令分发 |
| `src/framework/tee_s_cmd_dispatch.c` | TA 命令分发 |
| `src/manager/session_manager.c` | 会话生命周期 |
| `src/manager/agent_manager.c` | Agent 锁定机制 |
| `src/manager/service_manager.c` | 服务注册表 |

#### API 接口

| 函数 | 描述 |
|------|------|
| `open_session()` | 打开 TA 会话 |
| `close_session()` | 关闭 TA 会话 |
| `start_ta_task()` | 启动 TA 任务处理命令 |
| `tee_lock_agent()` | 锁定 Agent |
| `tee_unlock_agent()` | 解锁 Agent |
| `load_secure_file_image()` | 加载 TA 镜像 |

#### 依赖关系

```
gtask 依赖:
├── libtee_shared (IPC、内存管理)
├── libpermission_service (权限校验)
├── libteemem (内存分配)
├── libdrv (驱动接口)
└── libteeconfig (配置管理)
```

---

### 1.2 teesmcmgr - SMC 命令分发

**代码路径**: `framework/teesmcmgr/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| SMC 接收 | 等待并接收 REE 的 SMC 调用 | `src/teesmc.c:tee_smc_thread()` |
| 命令分发 | 将 SMC 命令转发给 gtask | `src/teesmc.c` |
| Idle 管理 | 系统休眠/唤醒状态管理 | `src/idle.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/main.c` | 入口点，创建 SMC 和 Idle 线程 |
| `src/teesmc.c` | SMC 线程实现 |
| `src/idle.c` | Idle 线程实现 |
| `src/teesmcmgr.h` | 公共接口定义 |

#### 关键函数

| 函数 | 描述 |
|------|------|
| `tee_smc_thread()` | SMC 处理主线程 |
| `tee_idle_thread()` | Idle 状态管理线程 |
| `smc_wait_switch_req()` | 等待 REE 的 SMC 请求 |
| `smc_switch_req()` | 请求状态切换 |

---

### 1.3 drvmgr - 驱动管理

**代码路径**: `framework/drvmgr/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| 驱动生命周期 | 驱动进程创建、销毁 | `src/drv_process_mgr.c` |
| 访问控制 | MAC 列表、权限检查 | `src/drv_auth.c` |
| IPC 通信 | 与驱动进程通信 | `src/drv_ipc_mgr.c` |
| 动态配置 | 驱动配置管理 | `src/drv_dyn_conf_mgr.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/main.c` | 入口点，服务器循环 |
| `src/drv_dispatch.c` | 驱动操作分发 |
| `src/drv_process_mgr.c` | 驱动进程管理 |
| `src/drv_auth.c` | 权限认证 |
| `src/drv_ipc_mgr.c` | IPC 通信 |
| `src/drv_dyn_conf_mgr.c` | 动态配置 |

#### 关键函数

| 函数 | 描述 |
|------|------|
| `driver_open_func()` | 打开驱动 |
| `driver_close_func()` | 关闭驱动 |
| `spawn_driver_handle()` | 创建驱动进程 |
| `caller_open_auth_check()` | 调用者权限检查 |
| `drv_mac_open_auth_check()` | MAC 访问控制 |

---

### 1.4 tarunner - ELF 加载器

**代码路径**: `framework/tarunner/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| ELF 解析 | 加载、解析、重定位 ELF 文件 | `src/main.c` |
| 多线程支持 | TA 多线程执行 | `src/ta_mt.c` |
| 库加载 | 动态库加载和初始化 | `src/load_init.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/main.c` | ELF 加载入口 |
| `src/ta_mc.c` | 多线程支持 |
| `src/load_init.c` | 库初始化 |
| `src/get_spawn_env.c` | 环境参数解析 |

#### TA 入口点

```c
// TA 必须实现的入口点
TA_CreateEntryPoint();           // TA 初始化
TA_OpenSessionEntryPoint();      // 会话打开
TA_InvokeCommandEntryPoint();    // 命令调用
TA_CloseSessionEntryPoint();    // 会话关闭
TA_DestroyEntryPoint();         // TA 销毁
```

---

## 2. Service 层

### 2.1 permission_service - 权限服务

**代码路径**: `services/permission_service/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| SEC 验签 | TA 二进制签名验证 | `src/perm_srv_elf_verify/perm_srv_elf_verify_cmd.c` |
| 证书管理 | X.509 证书验证 | `src/perm_srv_ta_cert/perm_srv_ta_cert.c` |
| CRL 管理 | 证书吊销列表 | `src/perm_srv_ta_crl/perm_srv_ta_crl.c` |
| TA 控制 | TA 去激活控制 | `src/perm_srv_ta_ctrl/perm_srv_ta_ctrl.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/main.c:538` | 服务入口 `tee_task_entry()` |
| `src/perm_srv_elf_verify/perm_srv_elf_verify_cmd.c` | ELF 验证 |
| `src/perm_srv_ta_cert/perm_srv_ta_cert.c` | 证书验证 |
| `src/perm_srv_ta_crl/perm_srv_ta_crl.c` | CRL 管理 |
| `src/perm_srv_set_config.c` | 配置签名验证 |

#### 安全机制

```
权限校验流程:
1. 接收 gtask 的 ELF 验证请求
2. secure_elf_verify() 验证签名
3. perm_srv_ta_run_authorization_check() 检查:
   - TA 是否被去激活
   - Manifest 属性匹配
   - 堆/栈大小限制
   - 设备有效性
4. anti_version_rollback() 防回滚
```

---

### 2.2 ssa - 安全存储服务

**代码路径**: `services/ssa/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| 加密存储 | AES-256-XTS 数据加密 | `src/secure_storage_agent/ssa_crypto.c` |
| 完整性保护 | HMAC-SHA256 完整性验证 | `src/secure_storage_agent/sfs.c` |
| 密钥派生 | HUK → TA Root Key → File Key | `src/secure_storage_agent/sfs_internal.c` |
| 文件操作 | 安全文件 CRUD | `src/secure_storage_agent/ssa_fs.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/secure_storage_agent/secure_storage_agent.c:1117` | 服务入口 |
| `src/secure_storage_agent/ssa_crypto.c` | 加解密实现 |
| `src/secure_storage_agent/sfs.c` | 安全文件系统 |
| `src/secure_storage_agent/sfs_internal.c` | 密钥派生 |
| `src/secure_storage_agent/ssa_fs.c` | 文件操作 |

#### 密钥层次

```
HUK (硬件根密钥)
    │
    └── CMAC 派生 ──→ TA Root Key (绑定 TA UUID)
                            │
            ┌───────────────┼───────────────┐
            ▼               ▼               ▼
    File Encryption    HMAC Key       File Name Key
    Key (XTS)                          (文件名哈希)
```

#### 路径防护

```c
// 证据: ssa_fs.c:141-160
TEE_Result check_file_name(const char *name)
{
    // 拒绝包含 "../" 的路径
    if (strstr(name, "../")) {
        tloge("Invalid file name: %s\n", name);
        return TEE_ERROR_BAD_PARAMETERS;
    }
}
```

---

### 2.3 huk_service - 硬件根密钥服务

**代码路径**: `services/huk_service/`

#### 模块职责

| 功能 | 描述 | 证据 |
|------|------|------|
| 密钥派生 | TA 特定密钥派生 | `src/huk_derive_takey.c` |
| 设备 ID | 设备标识生成 | `src/huk_get_deviceid.c` |

#### 关键文件

| 文件 | 职责 |
|------|------|
| `src/main.c:89` | 服务入口 |
| `src/huk_derive_takey.c` | TA 密钥派生 |
| `src/huk_get_deviceid.c` | 设备 ID |
| `include/huk_derive_takey.h` | 接口定义 |

#### 密钥派生流程

```
输入: Salt + TA UUID
处理:
1. huk_srv_map_from_task() 映射发送方共享内存
2. huk_derive_takey() 组合 salt + UUID
3. do_derive_takey() 调用 tee_crypto_derive_root_key()
4. memset_s() 安全清理临时缓冲区
输出: TA 特定密钥
```

---

## 3. Library 层

### 3.1 teelib - TA 库

**代码路径**: `lib/teelib/`

#### 子模块

| 子模块 | 职责 | 输出 |
|--------|------|------|
| `libteeos` | GP TEE API 实现 | `libteeos.a` |
| `libcrypto` | 加密 API | `libcrypto.a` |
| `libssa` | 安全存储 API | `libssa.a` |
| `libhuk` | HUK API | `libhuk.a` |
| `libtee_stub` | 桩实现 | `libtee_stub.a` |
| `libtaentry` | TA 入口点 | `libtaentry.a` |
| `libtee_shared` | 共享库 | `libtee_shared.so` |

#### GP API 头文件

| 头文件 | 描述 |
|--------|------|
| `include/tee/tee_internal_api.h` | 内部核心 API |
| `include/tee/tee_core_api.h` | 核心 API (TEE_OpenTASession 等) |
| `include/tee/tee_defines.h` | 类型定义、错误码 |
| `include/tee_trusted_storage_api.h` | 可信存储 API |
| `include/tee_crypto_api.h` | 加密 API |

---

### 3.2 drvlib - 驱动库

**代码路径**: `lib/drvlib/`

#### 子模块

| 子模块 | 职责 | 输出 |
|--------|------|------|
| `libdrv_shared` | 驱动共享库 | `libdrv_shared.so` |
| `common/libdrv_frame` | 驱动框架 | `libdrv_frame.a` |

---

### 3.3 syslib - 系统库

**代码路径**: `lib/syslib/`

#### 子模块

| 子模块 | 职责 | 输出 |
|--------|------|------|
| `libspawn_common` | 进程创建 | `libspawn_common.a` |
| `libelf_verify` | ELF 验签 | `libelf_verify.a` |
| `libdynconfmgr` | 动态配置 | `libdynconfmgr.a` |

---

## 4. Driver 层

### 4.1 tee_misc_driver - 基础驱动

**代码路径**: `drivers/tee_misc_driver/`

#### 功能

- 获取 bootloader 传入的共享内存信息
- 提供基础硬件抽象

---

### 4.2 crypto_mgr - 加解密驱动框架

**代码路径**: `drivers/crypto_mgr/`

#### 功能

| 功能 | 描述 | 证据 |
|------|------|------|
| 哈希运算 | SHA 系列 | `src/crypto_ioctl/crypto_syscall_hash.c` |
| 对称加密 | AES 系列 | `src/crypto_ioctl/crypto_syscall_cipher.c` |
| 非对称加密 | RSA、ECC | `src/crypto_ioctl/crypto_syscall_rsa.c` |
| HMAC | 消息认证 | `src/crypto_ioctl/crypto_syscall_hmac.c` |
| 随机数 | RNG | `src/crypto_ioctl/crypto_syscall_random.c` |
| 密钥派生 | 根密钥派生 | `src/crypto_ioctl/crypto_syscall_derive_key.c` |

#### IOCTL 接口

```c
enum crypto_hal {
    IOCTRL_CRYPTO_HASH_INIT = 0xc703,
    IOCTRL_CRYPTO_CIPHER_INIT = 0xc70b,
    IOCTRL_CRYPTO_HMAC_INIT = 0xc707,
    IOCTRL_CRYPTO_RSA_SIGN_DIGEST = 0xc717,
    IOCTRL_CRYPTO_ECC_SIGN_DIGEST = 0xc71c,
    IOCTRL_CRYPTO_GENERATE_RANDOM = 0xc721,
    IOCTRL_CRYPTO_DERIVE_ROOT_KEY = 0xc722,
    // ... 40+ 操作
};
```

---

## 5. 模块依赖关系

```
                    ┌─────────────────────────────────────┐
                    │           teesmcmgr                  │
                    │     (SMC 命令接收分发)               │
                    └─────────────────┬───────────────────┘
                                      │
                                      ▼ IPC
┌─────────────────────────────────────┼─────────────────────────────────────┐
│                                     │                                     │
│                                     ▼                                     │
│  ┌─────────────────────────────┐   │   ┌─────────────────────────────┐   │
│  │           gtask             │   │   │         drvmgr              │   │
│  │     (TA 生命周期管理)        │◄──┼──►│       (驱动管理)            │   │
│  └─────────────┬───────────────┘   │   └─────────────┬───────────────┘   │
│                │                   │                 │                   │
│    ┌───────────┼───────────┐       │     ┌───────────┼───────────┐       │
│    ▼           ▼           ▼       │     ▼           ▼           ▼       │
│ ┌──────┐  ┌──────┐  ┌──────┐    │  ┌──────┐  ┌──────┐  ┌──────┐     │
│ │ tarunner│ │ perm │ │ ssa  │    │  │crypto│ │misc  │ │platform│     │
│ │(ELF) │ │ srv  │ │      │    │  │ mgr  │ │ drv  │ │ drv  │     │
│ └──────┘  └──────┘ └──────┘    │  └──────┘ └──────┘ └──────┘     │
│                                                                 │
│                         TA Processes                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 相关文档

- 架构设计 → `01_Architecture.md`
- Native API → `03_Native_API.md`
- 构建系统 → `04_Build_System.md`
- 安全评审 → `05_Security_Review.md`
