# memory_security - 内存安全

## 1. 概述

### 1.1 模块定位

memory_security 模块提供内存安全定制化功能，增强内核对内存安全漏洞的防护能力。

**证据来源**: `memory_security/README_zh.md:1-4`

```
memory_security/README_zh.md:1-4
当前linux内核在内存安全方面还有需要加固的空间，
memory_security模块为内存安全定制相应的功能来增强安全能力。
```

### 1.2 子模块

| 子模块 | 功能 | 防护目标 |
|--------|------|----------|
| hideaddr | 隐藏 /proc/[pid]/maps 中的可执行内存地址 | KASLR 绕过防护 |
| jit_memory | JIT 内存访问控制 | SMEP/SMAP 绕过防护 |

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/memory_security/
├── module.c                    # 模块初始化入口 (lines 10-22)
├── Makefile                    # 构建配置 (lines 1-38)
├── Kconfig                     # 内核配置选项 (lines 1-30)
├── apply_hideaddr.sh          # 安装脚本 (lines 1-31)
├── README_zh.md               # 文档
├── include/
│   ├── hideaddr.h             # hideaddr 接口 (lines 1-20)
│   ├── jit_memory.h           # JIT 内存接口 (lines 1-21)
│   ├── jit_memory_module.h    # 模块注册接口 (lines 1-20)
│   ├── jit_memory_log.h       # 日志宏 (lines 1-22)
│   ├── jit_process.h          # 进程跟踪 (lines 1-20)
│   ├── jit_space_list.h       # JIT 空间数据结构 (lines 1-38)
│   └── hideaddr.h             # 地址隐藏接口 (lines 1-20)
└── src/
    ├── hideaddr.c             # 地址隐藏实现 (lines 1-77)
    ├── jit_memory.c           # JIT 内存控制 (lines 1-100)
    ├── jit_memory_module.c    # LiteHCK hooks 注册 (lines 1-21)
    ├── jit_process.c          # 进程 JIT 空间跟踪 (lines 1-92)
    └── jit_space_list.c       # 内存区域列表管理 (lines 1-91)
```

**证据来源**: `memory_security/README_zh.md:32-59`

---

## 3. HIDEADDR 模块

### 3.1 功能说明

通过检查渲染进程映射的匿名内存是否具有可执行权限，将映射后的内存地址的 start 和 end 值设置为 NULL，达到隐藏内存地址的目的。

**证据来源**: `memory_security/README_zh.md:9-12`

### 3.2 进程类型检查

通过进程的 SELinux 安全上下文判定是否为渲染进程。

**证据来源**: `memory_security/README_zh.md:13-15`

### 3.3 匿名内存区域权限检查

检查 vm_flags_t 的 flags 成员是否具有 `-x-` 权限。

**证据来源**: `memory_security/README_zh.md:17-19`

### 3.4 关键实现

**文件**: `src/hideaddr.c`

| 函数 | 行号 | 功能 |
|------|------|------|
| `is_anon_exec()` | 22-38 | 检查 VMA 是否为匿名可执行内存 |
| `hideaddr_avc_has_perm()` | 40-55 | SELinux AVC 权限检查 |
| `hideaddr_header_prefix()` | 57-72 | Hook /proc/[pid]/maps 输出 |
| `hideaddr_header_prefix_lhck_register()` | 74-77 | 注册 LiteHCK hook |

### 3.5 安全类定义

| 安全类 | 权限 |
|--------|------|
| `SECCLASS_HIDEADDR` | 自定义 SELinux 类 |
| `HIDEADDR__HIDE_EXEC_ANON_MEM` | 隐藏可执行匿名内存 |
| `HIDEADDR__HIDE_EXEC_ANON_MEM_DEBUG` | 调试权限（可查看地址） |

---

## 4. JIT_MEMORY 模块

### 4.1 功能说明

禁止渲染进程直接申请匿名可执行内存，限制将已申请的内存变更为可执行内存。

**证据来源**: `memory_security/README_zh.md:21-23`

### 4.2 预申请机制

渲染进程在申请匿名可执行内存前需要在 `mmap` 时携带 `MAP_JIT` flag，之后才可通过 `mprotect` 变更为可执行内存。

**证据来源**: `memory_security/README_zh.md:29-30`

### 4.3 关键实现

**文件**: `src/jit_memory.c`

| 函数 | 行号 | 功能 |
|------|------|------|
| `jit_avc_has_perm()` | 18-36 | SELinux 权限检查（绕过 init 进程 PID 1） |
| `check_jit_memory()` | 52-73 | 验证 mmap() 的 MAP_JIT flag |
| `find_jit_memory()` | 38-50 | 验证 mprotect() 对注册的 JIT 区域 |
| `delete_jit_memory()` | 75-86 | 处理 munmap() 的 JIT 区域 |
| `exit_jit_memory()` | 88-100 | 进程退出时清理 |

### 4.4 数据结构

**文件**: `include/jit_space_list.h`

```c
// 进程跟踪节点 (rbtree)
struct jit_process {
    int pid;                    // 键
    unsigned long cookie;       // 验证令牌
    struct rb_node node;        // rbtree 链接
    struct list_head head;      // 内存区域列表
};

// 内存区域节点 (链表)
struct jit_space_node {
    unsigned long begin, end;   // 内存范围
    struct list_head head;      // 链表链接
};

static struct rb_root root_tree = RB_ROOT;  // 全局 rbtree
```

---

## 5. 配置指导

### 5.1 Kconfig 选项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_MEMORY_SECURITY` | y/n | 主开关（必须启用后才能使用子功能） |
| `CONFIG_HIDE_MEM_ADDRESS` | y/n | HIDEADDR 模块 |
| `CONFIG_JIT_MEM_CTRL` | y/n | JIT_MEMORY 模块 |

**证据来源**: `memory_security/README_zh.md:62-70`

```
memory_security/README_zh.md:62-70
1. MEMORY_SECURITY使能：`CONFIG_MEMORY_SECURTIY=y`
2. MEMORY_SECURITY禁用：`CONFIG_MEMORY_SECURTIY=n`
3. MEMORY_SECURITY/JIT_MEM_CONTROL使能: `CONFIG_JIT_MEM_CTRL=y`
4. MEMORY_SECURITY/JIT_MEM_CONTROL禁用: `CONFIG_JIT_MEM_CTRL=n`
5. MEMORY_SECURITY/HIDEADDR使能: `CONFIG_HIDE_MEM_ADDRESS=y`
6. MEMORY_SECURITY/HIDEADDR禁用: `CONFIG_HIDE_MEM_ADDRESS=n`
```

### 5.2 初始化流程

**文件**: `module.c`

```c
int __init mem_security_hooks_init(void) {
    hideaddr_header_prefix_lhck_register();  // [L12]
    jit_memory_register_hooks();              // [L13]
    return 0;
}
module_init(mem_security_hooks_init);         // [L21]
```

---

## 6. 安全考量

### 6.1 攻击面

| 向量 | 风险 | 位置 |
|------|------|------|
| /proc/maps TOCTOU | 低 | `hideaddr_header_prefix()` 获取 task 存在竞态 |
| 内存耗尽 | 中 | 每个进程的 JIT 区域数无限制 |
| Cookie 预测 | 低 | Cookie 为 unsigned long，用户空间可预测 |
| 锁竞争 | 低 | 全局锁可能导致 DoS |

### 6.2 代码质量问题

| 问题 | 文件 | 行号 | 说明 |
|------|------|------|------|
| 赋值错误 | jit_space_list.c | 72 | `node->end == begin` 应该是 `=` |
| 死代码 | jit_space_list.c | 47-48 | now 变量未使用 |
| NULL 检查缺失 | jit_memory.c | - | `security_cred_getsecid()` 返回值未检查 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |
| [SELinux Adapter](https://gitee.com/openharmony/security_selinux_adapter) | SELinux 适配器 |
| [kernel_linux_config](https://gitee.com/openharmony/kernel_linux_config) | 内核配置 |
