# 导航与阅读路线

本文档提供 memory_utils Wiki 的完整导航，包括所有页面的链接和新人推荐阅读顺序。

## 文档索引

### 快速入门

| 文档 | 说明 | 推荐人群 |
|------|------|----------|
| [README](README.md) | Wiki 使用指南、覆盖范围、更新说明 | 所有开发者 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、目录结构 | 新人入门 |

### 架构与设计

| 文档 | 说明 | 推荐人群 |
|------|------|----------|
| [01_Architecture](01_Architecture.md) | 组件图、数据流、线程模型、时序图 | 架构师、开发者 |
| [03_Inner_API](03_Inner_API.md) | 模块接口、依赖方向、稳定性标注 | 二次开发者 |

### 接口文档

| 文档 | 说明 | 推荐人群 |
|------|------|----------|
| [02_NDK_API](02_NDK_API.md) | NDK C 接口清单、参数、错误码 | Native 应用开发者 |
| [04_GN_Build](04_GN_Build.md) | GN targets、编译产物、加载关系 | 构建工程师 |

### 安全与运维

| 文档 | 说明 | 推荐人群 |
|------|------|----------|
| [05_Security_Review](05_Security_Review.md) | 攻击面、信任边界、风险清单 | 安全工程师、开发者 |

---

## 新人阅读路线

### 路线 A：Native 应用开发者（使用 NDK）

```
建议时间: 30 分钟

1. [README](README.md)           → 5 分钟
2. [00_Overview](00_Overview.md)  → 5 分钟
3. [02_NDK_API](02_NDK_API.md)   → 15 分钟 (重点)
4. [04_GN_Build](04_GN_Build.md) → 5 分钟
```

**目标**: 掌握如何使用 NDK 接口开发应用

### 路线 B：系统开发者（进行二次开发）

```
建议时间: 60 分钟

1. [README](README.md)           → 5 分钟
2. [00_Overview](00_Overview.md)  → 10 分钟
3. [01_Architecture](01_Architecture.md) → 15 分钟
4. [03_Inner_API](03_Inner_API.md) → 20 分钟 (重点)
5. [04_GN_Build](04_GN_Build.md)  → 10 分钟
```

**目标**: 理解内部架构，进行模块扩展或修改

### 路线 C：安全评审人员

```
建议时间: 45 分钟

1. [README](README.md)           → 5 分钟
2. [05_Security_Review](05_Security_Review.md) → 30 分钟 (重点)
3. [01_Architecture](01_Architecture.md) → 10 分钟
```

**目标**: 了解安全风险和信任边界

---

## 模块速查表

### libdmabufheap - DMA 内存分配

| 主题 | 文档 | 章节 |
|------|------|------|
| 功能概述 | 00_Overview | libdmabufheap 系统库 |
| 架构设计 | 01_Architecture | libdmabufheap 架构 |
| 内部 API | 03_Inner_API | libdmabufheap 内部接口 |
| 构建配置 | 04_GN_Build | libdmabufheap |

### libmeminfo - 内存查询

| 主题 | 文档 | 章节 |
|------|------|------|
| 功能概述 | 00_Overview | libmeminfo 系统库 |
| 架构设计 | 01_Architecture | libmeminfo 架构 |
| 内部 API | 03_Inner_API | libmeminfo 内部接口 |
| 构建配置 | 04_GN_Build | libmeminfo |

### libpurgeablemem - 可回收内存

| 主题 | 文档 | 章节 |
|------|------|------|
| 功能概述 | 00_Overview | libpurgeable 系统库 |
| NDK API | 02_NDK_API | 完整接口清单 |
| 架构设计 | 01_Architecture | libpurgeablemem 架构 |
| 内部 API | 03_Inner_API | libpurgeablemem 内部接口 |
| 构建配置 | 04_GN_Build | libpurgeablemem |
| 安全风险 | 05_Security_Review | 可回收内存风险 |

---

## API 速查

### NDK API (C 接口)

| 函数 | 说明 | 头文件 |
|------|------|--------|
| [OH_PurgeableMemory_Create](02_NDK_API.md#oh_purgeablememory_create) | 创建可清除内存对象 | purgeable_memory.h |
| [OH_PurgeableMemory_Destroy](02_NDK_API.md#oh_purgeablememory_destroy) | 销毁对象 | purgeable_memory.h |
| [OH_PurgeableMemory_BeginRead](02_NDK_API.md#oh_purgeablememory_beginread) | 开始读取 | purgeable_memory.h |
| [OH_PurgeableMemory_EndRead](02_NDK_API.md#oh_purgeablememory_endreak) | 结束读取 | purgeable_memory.h |
| [OH_PurgeableMemory_BeginWrite](02_NDK_API.md#oh_purgeablememory_beginwrite) | 开始写入 | purgeable_memory.h |
| [OH_PurgeableMemory_EndWrite](02_NDK_API.md#oh_purgeablememory_endwrite) | 结束写入 | purgeable_memory.h |
| [OH_PurgeableMemory_GetContent](02_NDK_API.md#oh_purgeablememory_getcontent) | 获取内容指针 | purgeable_memory.h |
| [OH_PurgeableMemory_ContentSize](02_NDK_API.md#oh_purgeablememory_contentsize) | 获取内容大小 | purgeable_memory.h |
| [OH_PurgeableMemory_AppendModify](02_NDK_API.md#oh_purgeablememory_appendmodify) | 追加修改操作 | purgeable_memory.h |

### 内部 API (C/C++ 接口)

| 模块 | 符号 | 类型 | 说明 |
|------|------|------|------|
| libdmabufheap | DmabufHeapBuffer | struct | DMA 缓冲区结构体 |
| libdmabufheap | DmabufHeapOpen | function | 打开 DMA 堆设备 |
| libdmabufheap | DmabufHeapBufferAlloc | function | 分配 DMA 缓冲区 |
| libmeminfo | GetRssByPid | function | 获取进程 RSS |
| libmeminfo | GetPssByPid | function | 获取进程 PSS |
| libmeminfo | GetDmaInfo | function | 获取 DMA 缓冲区信息 |
| libpurgeablemem | PurgeableMem | class | C++ 可回收内存类 |
| libpurgeablemem | PurgeableAshMem | class | Ashmem 方案实现 |
| libpurgeablemem | UxPageTable | class | 用户扩展页表 |

---

## 常见问题

| 问题 | 解答 |
|------|------|
| 如何使用可回收内存？ | 见 [02_NDK_API](02_NDK_API.md) 完整示例 |
| 如何查询进程内存占用？ | 见 [03_Inner_API](03_Inner_API.md#libmeminfo-内部接口) |
| 如何分配 DMA 缓冲区？ | 见 [03_Inner_API](03_Inner_API.md#libdmabufheap-内部接口) |
| 有 N-API 吗？ | **无**，本项目仅提供 NDK C 接口 |
| 如何进行构建？ | 见 [04_GN_Build](04_GN_Build.md) |
| 存在哪些安全风险？ | 见 [05_Security_Review](05_Security_Review.md) |

---

## 外部链接

- [OpenHarmony 官方文档](https://www.openharmony.cn/)
- [memory_utils 源码](https://gitee.com/openharmony/utils)
- [NDK 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/ndk/Readme.md)

---

**最后更新**: 2026-02-06
