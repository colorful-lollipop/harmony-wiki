# Memory Utils Wiki

## 概述

本文档是 OpenHarmony `memory_utils` 内存基础库的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 接口、构建配置以及安全风险。

**重要声明**: 本 Wiki **不包含 N-API** 相关内容。经过全面搜索确认，`memory_utils` 仅提供 **NDK (Native Development Kit)** 纯 C 接口，**无 JavaScript/TypeScript 绑定**。

## 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| libdmabufheap | ✅ 已覆盖 | DMA 内存分配库，为多媒体服务提供零拷贝共享内存 |
| libmeminfo | ✅ 已覆盖 | 内存信息查询库，提供 RSS/PSS/SwapPss/GPU 内存/DMA 缓冲区查询 |
| libpurgeablemem | ✅ 已覆盖 | 可回收内存管理库，支持匿名内存和 Ashmem 两种方案 |
| libmemleak | ⏳ 规划中 | 内存泄漏检测库（当前代码库中不存在） |
| libspeculative | ⏳ 规划中 | 投机类型内存管理库（当前代码库中不存在） |

## 未覆盖范围

以下内容**不在本 Wiki 范围内**：

- 测试代码（`test/`, `*_test.*` 等）
- 第三方依赖库内部实现
- 内核 DMA-Buf/Ashmem 驱动实现
- OpenHarmony 系统级内存管理框架

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md     # 架构说明
├── 02_NDK_API.md          # NDK 接口文档
├── 03_Inner_API.md        # 内部 API 文档
├── 04_GN_Build.md         # GN 构建配置
├── 05_Security_Review.md  # 安全风险评审
└── _work/                 # 工作笔记（内部使用）
```

## 新人阅读建议

建议阅读顺序：

1. **[README.md](README.md)** - 了解 Wiki 覆盖范围和使用方法
2. **[00_Overview.md](00_Overview.md)** - 理解项目定位和核心能力
3. **[01_Architecture.md](01_Architecture.md)** - 掌握整体架构和数据流
4. **[02_NDK_API.md](02_NDK_API.md)** - 查阅 NDK 接口使用方式（如果使用 purgeable_memory）
5. **[03_Inner_API.md](03_Inner_API.md)** - 了解内部模块接口（如果进行二次开发）
6. **[04_GN_Build.md](04_GN_Build.md)** - 理解构建配置和产物
7. **[05_Security_Review.md](05_Security_Review.md)** - 了解安全风险和最佳实践

## 快速链接

### NDK API

| 接口 | 说明 |
|------|------|
| [OH_PurgeableMemory_Create](02_NDK_API.md#oh_purgeablememory_create) | 创建可清除内存对象 |
| [OH_PurgeableMemory_BeginRead](02_NDK_API.md#oh_purgeablememory_beginread) | 开始读取 |
| [OH_PurgeableMemory_BeginWrite](02_NDK_API.md#oh_purgeablememory_beginwrite) | 开始写入 |

### 内部 API

| 模块 | 说明 |
|------|------|
| [DmabufHeapBuffer](03_Inner_API.md#libdmabufheap) | DMA 缓冲区结构 |
| [GetDmaInfo](03_Inner_API.md#libmeminfo) | 获取 DMA 缓冲区信息 |
| [PurgeableMem](03_Inner_API.md#libpurgeablemem-cpp) | C++ 可回收内存类 |

## 代码证据引用规范

本 Wiki 所有关键结论均基于代码证据，引用格式如下：

| 类型 | 格式 | 示例 |
|------|------|------|
| 文件路径 | `path:line` | `bundle.json:32` |
| 符号名 | 函数/类/宏/target | `DmabufHeapOpen` |
| 代码片段 | 上下文描述 | C 接口实现位于 `c/src/purgeable_memory.c` |

## 更新说明

### 如何随代码更新 Wiki

1. **代码修改后**：检查以下内容是否需要更新
   - 新增/删除的 API 接口
   - 构建配置变更（`BUILD.gn`）
   - 架构设计变化
   - 安全相关修改

2. **更新步骤**：
   ```bash
   # 1. 修改对应模块的源码
   # 2. 更新 wiki/_work/NOTES.md 记录新发现
   # 3. 更新对应的 Wiki 文档
   # 4. 验证 SUMMARY.md 链接正确
   ```

3. **验证方法**：
   - 检查所有跳转链接是否有效
   - 确认 API 签名与源码一致
   - 验证构建配置与 BUILD.gn 匹配

### 版本信息

| 属性 | 值 |
|------|-----|
| **Wiki 版本** | 1.0 |
| **生成时间** | 2026-02-06 |
| **生成工具** | OpenHarmony 工程 Wiki 生成 Agent |
| **memory_utils 版本** | 3.1.0 |
| **OpenHarmony 版本** | 标准版 |

## 贡献指南

欢迎开发者为本 Wiki 贡献内容：

1. **发现错误**：在 `_work/NOTES.md` 中记录问题
2. **补充内容**：编辑对应文档并保持引用规范
3. **提出建议**：通过 Issue 或 PR 反馈

## 许可证

本 Wiki 内容遵循 [Apache License 2.0](../../LICENSE)。

---

**最后更新**: 2026-02-06
**维护者**: OpenHarmony 社区
