# Patch 详细分析

## 2.1 Patch 清单概览

rustix 库在 OpenHarmony 中包含 **3 个 Patch 文件**，全部位于 `ci/` 目录下。这些 Patch **不直接修改 rustix 源代码**，而是修改 CI 测试环境中的 QEMU 模拟器。

| # | Patch 文件 | 修改对象 | 修改范围 | Patch 类型 | 状态 |
|---|-----------|---------|---------|-----------|------|
| 1 | `ci/getsockopt-timeouts.patch` | QEMU linux-user | 4 文件 | Bugfix | 可移除 |
| 2 | `ci/s390x-stat-have-nsec.patch` | QEMU s390x | 1 文件 | Bugfix | 可移除 |
| 3 | `ci/translate-errno.patch` | QEMU errno | 1 文件 | Bugfix | 可移除 |

### Patch 统计

| 维度 | 统计 |
|------|------|
| **Patch 总数** | 3 |
| **直接修改 rustix 源码** | 0 |
| **修改 QEMU 模拟器** | 3 |
| **Bugfix 类型** | 3 |
| **Feature 类型** | 0 |
| **OH 特有适配** | 0 |

## 2.2 Patch 1：getsockopt-timeouts.patch

### 基本信息

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `ci/getsockopt-timeouts.patch` |
| **修改文件数** | 4 |
| **修改对象** | QEMU linux-user 模式的 socket 处理代码 |
| **Patch 类型** | Bugfix |
| **关联 Issue** | QEMU issue #885 |

### 修改文件列表

| 文件 | 修改内容 |
|------|---------|
| `linux-user/generic/sockbits.h` | 添加 `TARGET_SO_RCVTIMEO_NEW` 和 `TARGET_SO_SNDTIMEO_NEW` 宏定义 |
| `linux-user/mips/sockbits.h` | 添加 MIPS 架构的 socket timeout 宏定义 |
| `linux-user/sparc/sockbits.h` | 添加 SPARC 架构的 socket timeout 宏定义 |
| `linux-user/syscall.c` | 添加 setsockopt/getsockopt 处理逻辑，返回 `-TARGET_ENOPROTOOPT` |

### 修改摘要

```c
// 新增宏定义 (各架构 sockbits.h)
#define TARGET_SO_RCVTIMEO_NEW 66
#define TARGET_SO_SNDTIMEO_NEW 67

// syscall.c 中的处理逻辑
case TARGET_SO_RCVTIMEO_NEW:
case TARGET_SO_SNDTIMEO_NEW:
    return -TARGET_ENOPROTOOPT;
```

### 原始问题

QEMU 的 linux-user 模式在处理新的 socket timeout 选项（`SO_RCVTIMEO_NEW`、`SO_SNDTIMEO_NEW`）时没有正确的处理逻辑，可能导致存储意外的值或行为异常。

### 修改目的

1. **定义目标架构的宏**：为 QEMU 模拟的各目标架构添加新的 socket timeout 选项宏定义
2. **返回明确错误**：对于这些新选项，返回 `-TARGET_ENOPROTOOPT`（不支持的协议选项）错误码
3. **避免未定义行为**：确保 QEMU 在遇到这些选项时不会存储意外值

### OH 价值

| 方面 | 说明 |
|------|------|
| **对 rustix 的影响** | 间接影响，确保 rustix 的 socket 相关测试在 QEMU 模拟环境中正确执行 |
| **对 OH 的价值** | 确保 CI 测试环境的正确性，验证网络相关代码的跨架构行为 |
| **解决的问题** | 修复 socket timeout 测试用例在 QEMU 模拟环境中的异常行为 |

### 关键代码变更

**setsockopt 处理** (`linux-user/syscall.c`):
```c
case TARGET_SO_SNDTIMEO:
    optname = SO_SNDTIMEO;
    goto set_timeout;
case TARGET_SO_RCVTIMEO_NEW:
case TARGET_SO_SNDTIMEO_NEW:
    return -TARGET_ENOPROTOOPT;  // 新增：返回明确错误
```

**getsockopt 处理** (`linux-user/syscall.c`):
```c
case TARGET_SO_SNDTIMEO:
    optname = SO_SNDTIMEO;
    goto get_timeout;
case TARGET_SO_RCVTIMEO_NEW:
case TARGET_SO_SNDTIMEO_NEW:
    return -TARGET_ENOPROTOOPT;  // 新增：返回明确错误
```

### 升级建议

| 建议项 | 内容 |
|--------|------|
| **上游状态** | 此修复应提交到上游 QEMU 项目 |
| **移除条件** | 当上游 QEMU 版本包含此修复后，可移除此 Patch |
| **回归风险** | 低，此修改仅影响 CI 测试环境 |
| **验证方式** | 检查 rustix 的 socket timeout 相关测试是否通过 |

---

## 2.3 Patch 2：s390x-stat-have-nsec.patch

### 基本信息

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `ci/s390x-stat-have-nsec.patch` |
| **修改文件数** | 1 |
| **修改对象** | QEMU s390x 架构的 syscall_defs.h |
| **Patch 类型** | Bugfix |
| **关联 Issue** | QEMU s390x stat 实现问题 |

### 修改文件列表

| 文件 | 修改内容 |
|------|---------|
| `linux-user/syscall_defs.h` | 为 s390x 架构添加 `TARGET_STAT_HAVE_NSEC` 宏定义 |

### 修改摘要

```c
// 在 target_stat 结构体定义之前
#elif defined(TARGET_S390X)
#define TARGET_STAT_HAVE_NSEC  // 新增：声明 s390x 支持纳秒时间戳
struct target_stat {
    abi_ulong  st_dev;
    abi_ulong  st_ino;
    // ...
};
```

### 原始问题

在 s390x 架构上使用 QEMU 模拟运行 rustix 时，`fstat` 系统调用返回的 `st_mtime_nsec`、`st_atime_nsec`、`st_ctime_nsec` 等纳秒时间戳字段被错误设置为 0。

### 修改目的

1. **启用纳秒时间戳**：为 s390x 架构定义 `TARGET_STAT_HAVE_NSEC` 宏
2. **修复时间戳问题**：确保 `fstat` 调用正确返回纳秒级别的时间戳
3. **通过测试断言**：使 `tests/fs/futimens.rs` 中的纳秒断言能够通过

### OH 价值

| 方面 | 说明 |
|------|------|
| **对 rustix 的影响** | 修复 s390x 架构的 filesystem 时间戳测试 |
| **对 OH 的价值** | 确保 CI 中 s390x 架构测试的正确性 |
| **解决的问题** | s390x stat 结构体纳秒时间戳字段为 0 的问题 |

### 问题背景

- **影响范围**：Ubuntu 20.04 GitHub Actions 环境上的 libc fstat 实现
- **测试用例**：`tests/fs/futimens.rs` 中的纳秒时间戳断言
- **根本原因**：QEMU s390x 模拟缺少 `TARGET_STAT_HAVE_NSEC` 定义

### 升级建议

| 建议项 | 内容 |
|--------|------|
| **上游状态** | 建议提交到上游 QEMU |
| **移除条件** | 当上游 QEMU 或 s390x libc 修复后移除 |
| **回归风险** | 低，仅影响 s390x 架构测试 |
| **验证方式** | 运行 `tests/fs/futimens.rs` 测试 |

---

## 2.4 Patch 3：translate-errno.patch

### 基本信息

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `ci/translate-errno.patch` |
| **修改文件数** | 1 |
| **修改对象** | QEMU linux-user 的 getsockopt errno 处理 |
| **Patch 类型** | Bugfix |
| **关联 Issue** | QEMU issue #872 |

### 修改文件列表

| 文件 | 修改内容 |
|------|---------|
| `linux-user/syscall.c` | 在 getsockopt 处理 `SO_ERROR` 时添加 errno 转换 |

### 修改摘要

```c
// 在 getsockopt 处理 SO_ERROR 时
if (level == SOL_SOCKET && optname == SO_ERROR) {
    val = host_to_target_errno(val);  // 新增：转换 errno
}
```

### 原始问题

在 QEMU linux-user 模式下，通过 `getsockopt` 获取 `SO_ERROR` socket 选项时，返回的 errno 值没有从主机架构正确转换到目标架构。

### 修改目的

1. **errno 转换**：在获取 `SO_ERROR` 值时添加 `host_to_target_errno()` 转换
2. **跨架构正确性**：确保不同架构间 errno 值的一致性
3. **修复测试失败**：使依赖正确 errno 值的测试用例能够通过

### OH 价值

| 方面 | 说明 |
|------|------|
| **对 rustix 的影响** | 修复网络错误处理相关测试的跨架构行为 |
| **对 OH 的价值** | 确保 CI 测试中 errno 相关逻辑的正确性 |
| **解决的问题** | QEMU 跨架构运行时 `SO_ERROR` 返回值错误的问题 |

### 关键代码变更

```c
// linux-user/syscall.c getsockopt 处理
get_timeout:
    // ... 原有代码 ...
    if (level == SOL_SOCKET && optname == SO_TYPE) {
        val = host_to_target_sock_type(val);
    }
    if (level == SOL_SOCKET && optname == SO_ERROR) {  // 新增
        val = host_to_target_errno(val);                 // 新增
    }                                                    // 新增
```

### 升级建议

| 建议项 | 内容 |
|--------|------|
| **上游状态** | 此修复应提交到上游 QEMU |
| **移除条件** | 当上游 QEMU 版本包含此修复后移除 |
| **回归风险** | 低，仅影响 CI 测试环境 |
| **验证方式** | 检查 rustix 网络错误处理测试 |

---

## 2.5 Patch 总结与维护建议

### Patch 管理策略

| 策略 | 说明 |
|------|------|
| **上游优先** | 优先将 Patch 提交到上游 QEMU |
| **定期清理** | 每个版本周期检查上游合并状态，移除已合并的 Patch |
| **测试验证** | 移除 Patch 前确保相关测试仍能通过 |

### QEMU Patch 追踪

| Patch | 上游状态 | 预计移除版本 | 追踪链接 |
|-------|---------|-------------|---------|
| getsockopt-timeouts | 待提交 | TBD | QEMU #885 |
| s390x-stat-have-nsec | 待提交 | TBD | QEMU s390x |
| translate-errno | 待提交 | TBD | QEMU #872 |

### 版本升级检查清单

当升级 rustix 版本时：

- [ ] 确认 Patch 仍适用于新的测试场景
- [ ] 检查 QEMU 版本是否已包含相关修复
- [ ] 验证相关测试用例仍能通过
- [ ] 记录不可移除的 Patch 及其原因

---

## 2.6 注意事项

### 不修改 rustix 源码

这 3 个 Patch **均不直接修改 rustix 源代码**，而是修改 CI 环境中的 QEMU 模拟器。这是因为：

1. rustix 本身没有 OH 特有的代码需要修改
2. Patch 解决的问题是跨架构测试环境的问题
3. rustix 通过 Linux 兼容模式在 OH 上正常工作

### 无 OH 特有 Patch

与 curl、openssl 等库不同，rustix 不需要针对 OpenHarmony 特有的功能修改。原因是：

1. rustix 的设计本身就是跨平台的
2. OH 兼容 POSIX 标准
3. libc 后端提供了足够的抽象层

### 升级时的考虑

当 rustix 升级到新版本时：

1. **Patch 适用性**：检查这些 QEMU Patch 是否仍适用于新版本的测试
2. **新增 Patch**：新版本可能引入新的测试用例，需要新的 QEMU 修复
3. **上游进展**：关注上游 QEMU 的修复合并进度
