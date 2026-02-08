# elfutils 概览

---

## 库基本信息

| 属性 | 值 |
|-----|---|
| **库名称** | elfutils |
| **上游版本** | 0.193 |
| **上游地址** | https://sourceware.org/elfutils/ |
| **许可证** | LGPL V3.0, GPL V2.0, GPL V3.0 |
| **OH 版本** | 4.0 |
| **子系统** | thirdparty |
| **维护者** | zhanghaibo0@huawei.com |

---

## 原始功能描述

elfutils 是一个用于读取、创建和修改 **ELF 二进制文件**的工具和库集合，主要用于：

- ELF 文件格式解析和操作
- DWARF 调试信息读取和处理
- 符号表分析
- 堆栈跟踪和 unwind
- 进程和核心文件分析

### 核心库

- **libelf**: ELF 文件操作核心库
- **libdw**: DWARF 调试数据解析库
- **libdwfl**: 高级 DWARF/ELF 处理库
- **libebl**: 架构特定后端库
- **libdwfl_stacktrace**: 堆栈跟踪库（0.193 新增，实验性）

---

## 在 OpenHarmony 中的作用和定位

### 核心定位

elfutils 在 OpenHarmony 中**不是独立使用的工具库**，而是作为 **libabigail 的依赖库**提供服务。

### 主要用途

**ABI 兼容性检查支持**

```
OpenHarmony ABI 分析工具链
    ↓
libabigail（ABI 分析库）
    ↓
elfutils: libdw_static (DWARF 解析)
    ↓
ELF 二进制文件 + DWARF 调试信息
```

libabigail 在生成 ABI 特征文件时，需要：
1. 读取 ELF 二进制文件的结构信息
2. 解析 DWARF 调试数据（类型信息、函数签名等）
3. 提取符号表和位置信息

这些功能通过 elfutils 的 `libdw` 库提供。

### 使用特点

| 特性 | 说明 |
|------|-----|
| **编译目标** | host_only（仅用于宿主机编译） |
| **链接方式** | 静态链接 |
| **暴露接口** | 仅 `libdw_static` 作为 inner_kits |
| **使用范围** | 限于 libabigail 工具链，不用于系统运行时 |

---

## OH 适配策略

### 主要适配点

1. **构建系统迁移**：从 autotools (autoconf/make) 迁移到 GN
2. **精简编译**：仅编译库文件，不编译命令行工具
3. **预生成配置**：config.h 预生成而非 configure 生成
4. **少量 Bugfix**：4 个源文件的小幅修改

### 关键差异

| 方面 | 上游 | OpenHarmony |
|------|------|------------|
| 构建系统 | autotools | GN |
| 输出产物 | 共享库 + 静态库 + 工具 | 仅静态库 |
| 守护进程 | debuginfod | 移除 |
| 配置方式 | configure 生成 | 预生成 config.h |
| 使用场景 | 独立工具链 | libabigail 依赖 |

---

## 实验性功能：libdwfl_stacktrace

### 来源

elfutils 0.193 版本新增的实验性库。

### 功能

- 解析堆栈采样数据到调用链
- 跨多个 libdwfl 会话跟踪和缓存 Elf 结构
- 初步支持 perf_events 数据格式

### 状态

- **API 稳定性**: 实验性，可能在未来版本中变化
- **OH 编译**: 已集成到 `libdw_static`
- **使用场景**: 暂无 OH 模块使用此功能

---

## 架构支持

### OH 关注的架构

OpenHarmony 主要使用以下架构的 elfutils 后端：

| 架构 | 后端文件 | 状态 |
|------|---------|------|
| aarch64 | `backends/aarch64_*.c` | 主要支持 |
| arm | `backends/arm_*.c` | 支持 |
| riscv64 | `backends/riscv*_*.c` | 支持 |
| x86_64 | `backends/x86_64_*.c` | 支持 |

### 其他架构

上游还支持：alpha, bpf, csky, ia64, m68k, ppc, ppc64, s390, s390x, sh, sparc, sparc64, tilegx, x32, hexagon, mips

---

## 依赖关系

### 外部依赖

- **zlib** (`libz`): ELF section 压缩支持

### OH 内部依赖

- 被 `libabigail` 依赖（唯一主要使用者）

---

## 版本信息

### 当前版本

- **上游版本**: 0.193
- **OH 版本**: 4.0
- **配置文件版本**: config.h 显示 0.188（预生成文件滞后）

### 版本特性

0.193 主要更新：
- debuginfod: CORS 支持，HTTP 头缓存
- libdw: 新增 `dwarf_language()` 函数
- libdwfl_stacktrace: 新实验性库
- libelf: 改进 >64K section 处理

---

## 相关文档

- [构建集成](./03_Build_Integration.md) - GN 构建系统详解
- [代码修改](./02_Patches.md) - 源代码 Patch 分析
- [使用情况](./04_Usage_in_OH.md) - 依赖关系详情
- [完整评估](_work/ASSESSMENT.md) - Phase 0 技术评估
