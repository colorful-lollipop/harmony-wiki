# tzdriver - TrustZone 驱动

## 1. 概述

### 1.1 模块定位

tzdriver 是部署在 REE（Rich Execution Environment）侧的内核驱动，支持 REE 和 TEE（Trusted Execution Environment）之间的安全通信。

**证据来源**: `tzdriver/README_zh.md:5-6`

```
tzdriver/README_zh.md:5-6
Tzdriver是部署在REE侧的内核驱动，支持REE和TEE之间通信。
Tzdriver处理来自于Tee Client的命令，发送指令从REE切换到TEE。
```

### 1.2 核心功能

| 功能 | 说明 |
|------|------|
| SMC 通信 | 发送 SMC 指令，将 CPU 从 REE 切换到 TEE |
| 会话管理 | 管理 REE 与 TEE 之间的通信会话 |
| 共享内存 | 通过 mailbox 在 REE 和 TEE 之间共享数据 |
| 命令监控 | 监控 SMC 指令运行，提供超时检测 |
| 调试支持 | 创建 debugfs 调试节点 |
| TEE 日志 | 支持 TEE 日志记录和打印 |

**证据来源**: `tzdriver/README_zh.md:9-19`

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/tzdriver/
├── README.md / README_zh.md       # 文档
├── Kconfig                          # 内核配置选项
├── Makefile                         # 构建配置
├── apply_tzdriver.sh               # 安装脚本
├── tc_ns_client.h                  # 客户端接口头文件 (198行)
├── teek_ns_client.h                # 核心数据结构与SMC命令 (241行)
├── tc_ns_client.h
├── teek_ns_client.h
├── teek_client_constants.h         # GlobalPlatform常量
├── teek_client_type.h              # 类型定义
├── tui/                            # 可信UI
├── tlogger/                        # TEE日志子系统
├── ion/                            # ION内存管理
├── whitelist/                      # CA白名单
│   └── agent_allowed_ca.c          # 允许的Agent CA列表 (74行)
├── auth/                           # 认证子系统
│   ├── client_hash_auth.c          # 客户端哈希认证 (595行)
│   ├── auth_base_impl.c/h          # 基础认证操作
│   └── Kconfig                     # 认证配置
├── agent_rpmb/                     # RPMB子系统
│   └── core/agent_rpmb.c          # RPMB agent实现 (331行)
└── core/                           # 核心驱动实现
    ├── tc_client_driver.c          # 主驱动 (1505行)
    ├── session_manager.c           # 会话管理 (1460行)
    ├── smc_smp.c                   # SMC命令处理 (2000+行)
    ├── agent.c                     # Agent管理 (1369行)
    ├── mailbox_mempool.c           # 共享内存管理
    ├── gp_ops.c                    # GlobalPlatform操作
    ├── cmdmonitor.c                # 命令监控
    └── tzdebug.c                   # 调试支持
```

**证据来源**: `tzdriver/README_zh.md:27-37`

---

## 3. 架构设计

### 3.1 核心组件

#### A. 主驱动接口

**文件**: `core/tc_client_driver.c`

| 组件 | 行号 | 说明 |
|------|------|------|
| `tc_init()` | 1392 | 模块初始化入口 |
| `tc_ns_client` 设备 | 94 | 公共客户端接口设备 |
| `tc_private` 设备 | 95 | 私有守护进程接口设备 |
| `g_tc_ns_client_fops` | 1073 | 公共文件操作 |
| `g_teecd_fops` | 1084 | 守护进程文件操作 |

#### B. SMC 子系统

**文件**: `core/smc_smp.c`

| 组件 | 行号 | 说明 |
|------|------|------|
| `struct tc_ns_smc_queue *g_cmd_data` | 145 | 命令队列 |
| `tc_ns_smc()` | 1859 | 主 SMC 发送函数 |
| `smp_smc_send()` | 754 | 低级 SMC 调用 |
| `send_asm_smc_cmd()` | 710 | 汇编 SMC 包装器 |

#### C. 会话管理

**文件**: `core/session_manager.c`

| 组件 | 行号 | 说明 |
|------|------|------|
| `tc_ns_open_session()` | 1016 | 打开 TA 会话 |
| `tc_ns_close_session()` | 1221 | 关闭 TA 会话 |
| `tc_ns_send_cmd()` | 1277 | 向 TA 发送命令 |
| `find_service()` | 597 | 查找/创建服务 |
| `tc_ns_load_image()` | 876 | 加载 TA 镜像 |

#### D. Agent 系统

**文件**: `core/agent.c`

| 组件 | 行号 | 说明 |
|------|------|------|
| `tc_ns_register_agent()` | 964 | 注册 Agent |
| `tc_ns_unregister_agent()` | 1023 | 注销 Agent |
| `agent_process_work()` | 534 | 处理 Agent 请求 |
| `tc_ns_wait_event()` | 586 | 等待 Agent 唤醒 |

---

## 4. IOCTL 接口

### 4.1 主 IOCTL 命令

**文件**: `tc_ns_client.h:153-196`

| IOCTL 命令 | 值 | 描述 | 处理器 |
|------------|-------|------|--------|
| `TC_NS_CLIENT_IOCTL_SES_OPEN_REQ` | _IOW('t', 1) | 打开会话 | `tc_ns_open_session()` |
| `TC_NS_CLIENT_IOCTL_SES_CLOSE_REQ` | _IOWR('t', 2) | 关闭会话 | `tc_ns_close_session()` |
| `TC_NS_CLIENT_IOCTL_SEND_CMD_REQ` | _IOWR('t', 3) | 发送命令 | `tc_ns_send_cmd()` |
| `TC_NS_CLIENT_IOCTL_SHRD_MEM_RELEASE` | _IOWR('t', 4) | 释放共享内存 | - |
| `TC_NS_CLIENT_IOCTL_WAIT_EVENT` | _IOWR('t', 5) | 等待事件 | `tc_ns_wait_event()` |
| `TC_NS_CLIENT_IOCTL_REGISTER_AGENT` | _IOWR('t', 7) | 注册 Agent | `tc_ns_register_agent()` |
| `TC_NS_CLIENT_IOCTL_UNREGISTER_AGENT` | _IOWR('t', 8) | 注销 Agent | `tc_ns_unregister_agent()` |
| `TC_NS_CLIENT_IOCTL_LOAD_APP_REQ` | _IOWR('t', 9) | 加载 TA | `tc_ns_load_secfile()` |
| `TC_NS_CLIENT_IOCTL_LOGIN` | _IOWR('t', 14) | 设置登录信息 | `tc_ns_client_login_func()` |

### 4.2 IOCTL 处理器

| 处理器 | 文件 | 行号 | 说明 |
|--------|------|------|------|
| `public_ioctl()` | tc_client_driver.c | 753 | 公共 IOCTL |
| `tc_client_ioctl()` | tc_client_driver.c | 911 | 客户端 IOCTL |
| `tc_private_ioctl()` | tc_client_driver.c | 879 | 私有 IOCTL |
| `tc_client_agent_ioctl()` | tc_client_driver.c | 838 | Agent IOCTL |

---

## 5. 安全机制

### 5.1 认证子系统

**文件**: `auth/client_hash_auth.c`

| 功能 | 行号 | 说明 |
|------|------|------|
| `calc_task_so_hash()` | 207 | 计算 libteec 库的 SHA256 哈希 |
| `calc_task_hash()` | auth_base_impl.c:198 | 计算调用进程的哈希 |
| `calc_client_auth_hash()` | 301 | 计算客户端认证哈希 |

### 5.2 白名单机制

**文件**: `whitelist/agent_allowed_ca.c`

```c
static struct ca_info g_allowed_ext_agent_ca[] = {
    {"/vendor/bin/hiaiserver", 3094, TEE_SECE_AGENT_ID},
    {"/vendor/bin/hw/hdf_devhost", 1114, TEE_FACE_AGENT1_ID},
    // 仅开发版本:
    {"/vendor/bin/tee_test_agent", 0, TEE_SECE_AGENT_ID},
};
```

**访问检查**: `is_allowed_ca()` (line 42)

### 5.3 RPMB Agent

**文件**: `agent_rpmb/core/agent_rpmb.c`

| 属性 | 值 |
|------|-----|
| Agent ID | `TEE_RPMB_AGENT_ID` (0x4abe6198) |
| 命令 | SEC_GET_DEVINFO, SEC_SEND_IOCMD, SEC_RPMB_LOCK |
| 锁管理 | 基于计数器的锁 |
| 超时监控 | 800ms 超时检测 |

### 5.4 登录认证

**文件**: `core/session_manager.c:505`

| 方法 | 说明 |
|------|------|
| `TEEC_LOGIN_IDENTIFY` | 基于包名 + 公钥的认证 |
| UID 回退 | 如无公钥，使用进程 UID |

---

## 6. 构建配置

### 6.1 Kconfig 选项

| 选项 | 说明 |
|------|------|
| `CONFIG_TZDRIVER` | 主驱动开关 |
| `CONFIG_CPU_AFF_NR` | CPU 亲和性限制（默认: 1） |
| `CONFIG_KERNEL_CLIENT` | 支持内核模式 CA |
| `CONFIG_TEELOG` | TEE 日志开关 |
| `CONFIG_PAGES_MEM` | 使用页面管理日志内存 |
| `CONFIG_THIRDPARTY_COMPATIBLE` | 第三方 TEE 兼容 |
| `CONFIG_CLIENT_AUTH` | 客户端认证 |
| `CONFIG_AUTH_HASH` | 基于哈希的认证 |
| `CONFIG_RPMB_AGENT` | RPMB Agent |

**证据来源**: `tzdriver/README_zh.md:45-53`

```
tzdriver/README_zh.md:45-53
CONFIG_TZDRIVER=y
CONFIG_CPU_AFF_NR=1
CONFIG_KERNEL_CLIENT=y
CONFIG_TEELOG=y
CONFIG_PAGES_MEM=y
CONFIG_THIRDPARTY_COMPATIBLE=y
```

### 6.2 编译命令

```bash
./build.sh --product-name rk3568 --ccache --build-target kernel \
  --gn-args linux_kernel_version="linux-5.10"
```

**证据来源**: `tzdriver/README_zh.md:73-74`

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [TEE Client 仓库](https://gitee.com/openharmony/tee_tee_client) | 用户态 TEE Client 库 |
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |
| [02_Architecture.md](../02_Architecture.md) | 整体架构 |
