# 阅读指南

本文档为 rustix 库在 OpenHarmony 中的集成与适配提供完整的技术参考。

## 读者定位

本文档面向以下读者：

- **OpenHarmony 开发者**：了解 rustix 在系统中的角色和使用方式
- **系统集成工程师**：理解依赖关系和构建配置
- **安全工程师**：评估安全风险和 CVE 影响
- **维护者**：掌握 Patch 状态和升级策略

## 推荐阅读顺序

### 场景一：快速概览

1. **README.md** - 5 分钟，了解 rustix 在 OH 中的整体情况
2. **01_Overview.md** - 10 分钟，理解库的核心功能和定位

### 场景二：深入适配细节

1. **README.md** - 快速概览
2. **02_Patches.md** - **核心**，详细分析所有 Patch
3. **03_Build_Integration.md** - 构建系统适配细节
4. **04_Usage_in_OH.md** - 依赖关系和使用场景

### 场景三：安全评估

1. **README.md** - 快速概览
2. **06_Security.md** - 安全风险分析
3. **02_Patches.md** - Patch 引入的安全影响

### 场景四：维护和升级

1. **README.md** - 快速概览
2. **02_Patches.md** - Patch 升级建议
3. **03_Build_Integration.md** - 配置差异
4. **06_Security.md** - CVE 和安全更新

## 文档依赖关系

```
README.md
    │
    ├──► 01_Overview.md (独立，可直接阅读)
    │
    ├──► 02_Patches.md ★ 核心依赖
    │         │
    │         ├──► _work/ASSESSMENT.md (详细评估数据)
    │         └──► 03_Build_Integration.md (构建配置上下文)
    │
    ├──► 03_Build_Integration.md (独立，可直接阅读)
    │         │
    │         └──► 02_Patches.md (理解配置差异的原因)
    │
    ├──► 04_Usage_in_OH.md (独立，可直接阅读)
    │         │
    │         └──► 01_Overview.md (理解功能背景)
    │
    ├──► 05_API_Differences.md (按需阅读)
    │         │
    │         └──► 01_Overview.md (理解 API 背景)
    │
    └──► 06_Security.md (独立，可直接阅读)
              │
              ├──► 02_Patches.md (Patch 安全影响)
              └──► 04_Usage_in_OH.md (使用场景上下文)
```

## 关键信息速查

### Patch 摘要

| Patch 文件 | 修改对象 | 类型 | 状态 |
|-----------|---------|------|------|
| `ci/getsockopt-timeouts.patch` | QEMU socket timeout | Bugfix | 可移除 |
| `ci/s390x-stat-have-nsec.patch` | QEMU s390x stat | Bugfix | 可移除 |
| `ci/translate-errno.patch` | QEMU errno 转换 | Bugfix | 可移除 |

### 依赖链

```
clap ──> is-terminal ──> rustix ──> libc, io-lifetimes
env_logger ──> is-terminal ──> rustix ──> ...
```

### 构建配置关键点

- **目标识别**：`CARGO_CFG_TARGET_OS=linux`
- **后端选择**：强制 `libc` 后端
- **输出类型**：`rlib` 静态库

## 常见问题

### Q: rustix 是否原生支持 OpenHarmony？

A: 不，rustix 源代码中没有 OH 特定的代码。OpenHarmony 通过 Linux 兼容模式适配，将 OH 识别为 Linux 系统。

### Q: 为什么使用 libc 后端而非 linux_raw 后端？

A: libc 后端通过标准 C 库进行系统调用，提供了更好的兼容性。linux_raw 后端直接使用汇编指令进行系统调用，可能存在 OH 特有的细微差异。

### Q: 这些 Patch 会一直存在吗？

A: 不会。这些 Patch 修改的是 CI 环境中的 QEMU 模拟器。当上游 QEMU 合并相关修复后，应移除这些 Patch。

## 术语表

| 术语 | 说明 |
|------|------|
| **linux_raw 后端** | rustix 直接使用 Linux 系统调用的后端实现 |
| **libc 后端** | rustix 通过标准 C 库进行系统调用的后端实现 |
| **vDSO** | Virtual Dynamic Shared Object，Linux 内核提供的虚拟动态链接库 |
| **I/O 安全** | Rust RFC 3128 定义的 I/O 安全特性，使用 OwnedFd/AsFd |
