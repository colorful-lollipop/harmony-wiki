# 模块清单

## 1. 安全核心模块

### 1.1 tzdriver - TrustZone 驱动

| 属性 | 值 |
|------|-----|
| 功能 | TrustZone 驱动，支持 REE 和 TEE 之间通信 |
| 架构 | SMC 指令通信、会话管理、共享内存 |
| 关键组件 | smc、session_manager、mailbox、cmd_monitor、tzdebug、tlogger |
| 配置项 | CONFIG_TZDRIVER、CONFIG_CPU_AFF_NR、CONFIG_KERNEL_CLIENT |

**证据来源**: `tzdriver/README_zh.md:5-19`

```
tzdriver/README_zh.md:5-19
Tzdriver是部署在REE侧的内核驱动，支持REE和TEE之间通信。
Tzdriver处理来自于Tee Client的命令，发送指令从REE切换到TEE。

主要模块：
- smc：发送smc指令，将CPU从REE侧切换到TEE侧运行
- session_manager：管理REE与TEE之间的通信会话
- mailbox：REE和TEE之间通过mailbox共享数据
- cmd_monitor：监控smc指令的运行，提供超时检测机制
```

### 1.2 memory_security - 内存安全

| 属性 | 值 |
|------|-----|
| 功能 | 内存安全定制化，增强安全能力 |
| 子模块 | hideaddr、jit_memory |
| 配置项 | CONFIG_MEMORY_SECURITY、CONFIG_HIDE_MEM_ADDRESS、CONFIG_JIT_MEM_CTRL |
| 安全机制 | SELinux 权限检查、rbtree 追踪 |

**证据来源**: `memory_security/README_zh.md:1-31`

### 1.3 pac - 指针认证码

| 属性 | 值 |
|------|-----|
| 功能 | 基于 ARMv8.3-a 的指针认证机制 |
| 密钥类型 | APIA、APIB、APDA、APDB、APGA |
| 保护范围 | 任务切换上下文、异常中断上下文 |
| 配置项 | CONFIG_ARM64_PTR_AUTH、CONFIG_ARM64_PTR_AUTH_EXT |

**证据来源**: `pac/README_zh.md:1-68`

### 1.4 container_escape_detection - 容器逃逸检测

| 属性 | 值 |
|------|-----|
| 功能 | 检测和防止容器逃逸尝试 |
| 检测机制 | HCK hooks 监控进程属性、凭证、命名空间变更 |
| 跟踪结构 | RB-tree |
| 事件类型 | CRED_CHANGED、NSPROXY_CHANGED、ATTRIBUTE_CHANGED |

---

## 2. 功能模块

### 2.1 code_sign - 代码签名

| 属性 | 值 |
|------|-----|
| 功能 | ELF 文件签名验证、证书链管理 |
| 数据结构 | red-black tree、sign_block_t、merkle_tree_t |
| 主要文件 | code_sign_ioctl.c、code_sign_ext.c、verify_cert_chain.c |
| 配置项 | CONFIG_SECURITY_CODE_SIGN |

### 2.2 qos_auth - QoS 认证

| 属性 | 值 |
|------|-----|
| 功能 | FFRT 底层调度能力、权限管控 |
| 子模块 | auth_ctrl、qos_ctrl |
| 策略 | 前台/后台/system 等多种 policy |
| 配置项 | CONFIG_AUTHORITY_CTRL、CONFIG_QOS_CTRL |

**证据来源**: `qos_auth/README_zh.md:1-71`

```
qos_auth/README_zh.md:1-21
轻量化权限管控模块：uid粒度的权限管控，根据app前后台状态动态管控
动态多级qos模块：提供多种policy，每个policy包含6个qos等级
```

### 2.3 newip - 新 IP 协议

| 属性 | 值 |
|------|-----|
| 功能 | 可变长多语义地址、可变长定制化报头封装 |
| 效率提升 | 相比 IPv4 节省 25.9%，相比 IPv6 节省 44.9% |
| 主要代码 | net/newip/、drivers/net/bt/ |
| 配置 | kernel_linux_5.10 |

**证据来源**: `newip/README_zh.md:1-98`

### 2.4 xpm - 可执行权限管理器

| 属性 | 值 |
|------|-----|
| 功能 | 应用二进制和 ABC 代码的运行时管控 |
| 特性 | 执行权限检查、验签地址区、代码完整性保护 |
| 标签 | exec_no_sign、execmem_anon |
| 配置项 | CONFIG_SECURITY_XPM |

**证据来源**: `xpm/README_zh.md:1-79`

---

## 3. 基础模块

### 3.1 ucollection - 统一采集

| 属性 | 值 |
|------|-----|
| 功能 | 进程 CPU 维测数据采集 |
| 接口 | ioctl via /dev/ucollection |
| 指标 | 进程/线程 CPU 使用率 |
| 配置项 | CONFIG_UNIFIED_COLLECTION |

**证据来源**: `ucollection/README_zh.md:1-33`

### 3.2 module_sample - 示例模块

| 属性 | 值 |
|------|-----|
| 功能 | BUILD.gn 构建示例 |
| 文件 | ko_sample.c、sample_fun.c、BUILD.gn |
| 用途 | 演示如何构建 ko 模块 |

**证据来源**: `module_sample/BUILD.gn:1-23`

---

## 4. 模块依赖关系

```
基础依赖
└── ucollection/          # 通用数据结构

安全模块
├── code_sign/           # 代码签名
├── container_escape_detection/  # 容器安全
├── memory_security/     # 内存安全
├── pac/                 # 指针认证
├── qos_auth/            # QoS认证
└── tzdriver/            # TEE/TrustZone驱动

网络模块
└── newip/              # 新IP协议

包管理
└── xpm/                # 包管理器
```

---

## 5. 模块文档索引

| 模块 | 文档位置 |
|------|----------|
| tzdriver | [modules/tzdriver.md](modules/tzdriver.md) |
| memory_security | [modules/memory_security.md](modules/memory_security.md) |
| pac | [modules/pac.md](modules/pac.md) |
| container_escape_detection | [modules/container_escape_detection.md](modules/container_escape_detection.md) |
| code_sign | [modules/code_sign.md](modules/code_sign.md) |
| qos_auth | [modules/qos_auth.md](modules/qos_auth.md) |
| newip | [modules/newip.md](modules/newip.md) |
| xpm | [modules/xpm.md](modules/xpm.md) |
| ucollection | [modules/ucollection.md](modules/ucollection.md) |
| module_sample | [modules/module_sample.md](modules/module_sample.md) |
