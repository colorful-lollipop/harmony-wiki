# pac - 指针认证码

## 1. 概述

### 1.1 模块定位

PAC 模块利用 ARMv8.3-a 架构提供的 PAC（Pointer Authentication Code）特性，为 Linux 内核提供指针认证和完整性保护能力，防护 JOP/ROP/DOP 攻击。

**证据来源**: `pac/README_zh.md:1-8`

```
pac/README_zh.md:1-8
现阶段，内存安全漏洞是对计算机系统安全最严重的威胁。
PAC机制能有效防护JOP/ROP/DOP攻击。
PAC模块基于ARMv8.3-a架构，提供密钥管理、数据和指针签名验签，
以及任务切换上下文、异常中断上下文的PAC保护机制。
```

### 1.2 PAC 原理

ARM 硬件基于 QARMA 密码算法，将被保护的指针或数据、用于签名的密钥和盐值作为输入，计算输出一个 MAC 值。对于指针，将有效的 MAC 值存放在指针的未使用高位。

**证据来源**: `pac/README_zh.md:12-14`

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/pac/
├── Makefile                    # 构建配置
├── README_zh.md               # 文档
├── apply_pac.sh               # 集成脚本
├── config/
│   └── config.txt             # 受保护数据结构配置 (lines 1-57)
├── figures/                   # 文档图例
│   ├── pac.png               # PAC 原理图
│   ├── key.png               # 密钥管理图
│   └── pac_context.png       # 上下文保护图
├── include/                   # 头文件
│   ├── pointer_auth_common.h        # 核心 PAC 宏 (lines 1-89)
│   ├── pointer_auth_context.h       # 上下文保护 API (lines 1-120)
│   ├── asm_pointer_auth_key.h      # 密钥安装汇编宏 (lines 1-65)
│   └── asm_pointer_auth_context.h  # 上下文签名汇编宏 (lines 1-173)
└── src/                       # 实现
    ├── pointer_auth_key.c          # 密钥初始化 (lines 1-11)
    ├── pointer_auth_context.c      # 上下文保护 C 函数 (lines 1-134)
    ├── asm_pointer_auth_key.S      # 密钥初始化汇编 (lines 1-57)
    ├── asm_pointer_auth_context.S  # 上下文签名汇编 (lines 1-247)
    └── asm_pointer_auth_constructors.S  # 构造函数初始化 (lines 1-27)
```

**证据来源**: `pac/README_zh.md:26-37`

---

## 3. 密钥管理

### 3.1 密钥类型 (ARMv8.3-a)

**文件**: `include/asm_pointer_auth_key.h`

| 密钥 | 用途 | 系统寄存器 |
|------|------|------------|
| APIA | 指令指针（前向 CFI） | APIAKEYLO/HI_EL1 |
| APIB | 指令指针（后向 CFI） | APIBKEYLO/HI_EL1 |
| APDA | 数据指针认证 | APDAKEYLO/HI_EL1 |
| APDB | 数据指针认证 | APDBKEYLO/HI_EL1 |
| APGA | 通用认证（哈希） | APGAKEYLO/HI_EL1 |

### 3.2 核心 PAC 指令宏

**文件**: `include/pointer_auth_common.h`

| 宏 | 行号 | 功能 |
|-----|------|------|
| `pauth_sign()` | 11-12 | 使用指定密钥签名指针 |
| `pauth_validate()` | 14-15 | 验证指针签名 |
| `pauth_strip()` | 17-23 | 从指针移除 PAC 位 |
| `pauth_hash()` | 25 | 使用 PACGA 生成哈希 |
| `pauth_pacda/pacdb` | 57-59 | 使用密钥 A/B 签名数据指针 |
| `pauth_autda/autdb` | 76-78 | 使用密钥 A/B 验证数据指针 |
| `pauth_xpacd/xpaci` | 84-86 | 剥离数据/指令 PAC |

### 3.3 密钥安装

**文件**: `src/asm_pointer_auth_key.S:51-56`

```asm
SYM_CODE_START(ptrauth_kernel_keys_init)
    ptrauth_back_key_init      // 初始化 APIB 密钥用于后向 CFI
    ptrauth_common_keys_init   // 初始化 APIA, APDA, APDB, APGA
    isb
    ret
SYM_CODE_END(ptrauth_kernel_keys_init)
```

---

## 4. 上下文保护

### 4.1 线程上下文函数

**文件**: `include/pointer_auth_context.h`

| 函数 | 行号 | 功能 |
|------|------|------|
| `sign_thread_context()` | 20-21 | 签名 SP 和 LR |
| `auth_thread_context()` | 20-21 | 验证并认证 |

### 4.2 异常上下文函数

| 函数 | 行号 | 功能 |
|------|------|------|
| `sign_exception_context_asm()` | 23-24 | 汇编签名 |
| `auth_exception_context_asm()` | 23-24 | 汇编验证 |
| `sign_exception_context()` | 108-111 | 安全包装器（ IRQ 管理） |
| `auth_exception_context()` | 108-111 | 安全包装器（IRQ 管理） |

### 4.3 受保护寄存器

```c
enum pac_pt_regs {
    REGS_X16 = 0,   // 临时寄存器
    REGS_X17,       // 临时寄存器
    REGS_LR,        // 链接寄存器
    REGS_SP,        // 栈指针
    REGS_PC,        // 程序计数器
    REGS_PSTATE,    // 处理器状态
};
```

### 4.4 哈希链机制

**文件**: `include/asm_pointer_auth_context.h`

**线程上下文签名**:
```asm
.macro sign_thread_context_common, tmp1=x0, tmp2=x1, tmp3=x2
    pacga  \tmp2, \tmp1, \tmp2    // 用 PC 和 SP 计算哈希
    pacga  \tmp2, \tmp3, \tmp2    // 将额外数据链接到哈希
    str    \tmp2, [\tmp1, CPU_CONTEXT_PAC_HASH]
.endm
```

**验证失败处理**:
```asm
    cmp    \tmp2, \tmp3           // 比较
    b.ne   .Lthread_context_pac_panic\@  // 不匹配则 PANIC
```

---

## 5. 构建配置

### 5.1 Kconfig 选项

| 配置项 | 功能 |
|--------|------|
| `CONFIG_ARM64_PTR_AUTH` | PAC 使能 |
| `CONFIG_ARM64_PTR_AUTH_EXT` | 密钥管理 |
| `CONFIG_ARM64_PTR_AUTH_DATA_PTR` | 数据指针保护 |
| `CONFIG_ARM64_PTR_AUTH_DATA_FIELD` | 数据字段保护 |
| `CONFIG_ARM64_PTR_AUTH_FWD_CFI` | 前向 CFI |

**证据来源**: `pac/README_zh.md:41-57`

```
pac/README_zh.md:41-57
1. PAC使能: CONFIG_ARM64_PTR_AUTH=y
2. 密钥使能: CONFIG_ARM64_PTR_AUTH_EXT=y
3. 关键数据和指针保护使能:
   CONFIG_ARM64_PTR_AUTH_DATA_PTR=y
   CONFIG_ARM64_PTR_AUTH_DATA_FIELD=y
4. 前向CFI使能: CONFIG_ARM64_PTR_AUTH_FWD_CFI=y
```

### 5.2 Makefile 构建

**文件**: `Makefile:8-12`

```makefile
obj-$(CONFIG_ARM64_PTR_AUTH_EXT) += src/pointer_auth_key.o
obj-$(CONFIG_ARM64_PTR_AUTH_EXT) += src/asm_pointer_auth_key.o
obj-$(CONFIG_ARM64_PTR_AUTH_DATA_FIELD) += src/pointer_auth_context.o
obj-$(CONFIG_ARM64_PTR_AUTH_DATA_FIELD) += src/asm_pointer_auth_context.o
obj-$(CONFIG_CONSTRUCTORS) += src/asm_pointer_auth_constructors.o
```

---

## 6. 受保护的数据结构

**文件**: `config/config.txt`

列出 56 个正在被保护的关键内核结构体：

| 结构体 | 保护字段 |
|--------|----------|
| `struct task_struct` | mm, real_parent, cred, security |
| `struct cred` | session_keyring, process_keyring, security |
| `struct super_block` | s_root, s_security |
| `struct inode` | i_op, i_sb, i_security |
| `struct dentry` | d_parent, d_inode, d_sb |
| `struct nsproxy` | 所有命名空间指针 |
| - | SELinux 结构体 |
| - | IPC 结构体 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |
| [02_Architecture.md](../02_Architecture.md) | 整体架构 |
| [kernel_linux_5.10](https://gitee.com/openharmony/kernel_linux_5.10) | 内核代码仓库 |
