# 阅读路线建议

本文档提供 FatFs OpenHarmony 适配文档的阅读路线建议，根据不同需求选择合适的阅读路径。

---

## 路线一：快速了解（15 分钟）

**目标**：快速了解 FatFs 在 OH 中的定位和适配策略

1. **[README.md](README.md)** - 阅读开头部分（"快速导航"之前）
2. **[01_Overview.md](01_Overview.md)** - 简要了解 FatFs 和 OH 的作用
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 仅阅读"使用场景"和"架构"部分

**预期收获**：
- ✅ 知道 FatFs 在 OH 中做什么
- ✅ 了解 OH 的适配策略（非侵入式）
- ✅ 知道主要的 OH 特有功能

---

## 路线二：开发者入门（1 小时）

**目标**：了解如何在 OH 中使用 FatFs，理解适配层的实现

1. **[README.md](README.md)** - 完整阅读
2. **[01_Overview.md](01_Overview.md)** - 完整阅读
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 完整阅读
   - 重点关注"OH 集成架构"和"VFS 适配层"部分
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 阅读前两节
   - "构建系统概览"
   - "GNI 文件"
5. **[05_API_Differences.md](05_API_Differences.md)** - 快速浏览"新增 API"部分

**预期收获**：
- ✅ 理解 FatFs 在 OH 中的使用方式
- ✅ 了解 VFS 适配层的工作原理
- ✅ 知道如何通过 Kconfig 配置 FatFs
- ✅ 熟悉 OH 新增的主要 API

---

## 路线三：深入适配层（2 小时）

**目标**：深入理解 OH 的 FatFs 适配实现，包括源代码分析

1. **[02_Patches.md](02_Patches.md)** - 完整阅读
   - 重点：ffconf.h 配置详解、新增文件分析
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 完整阅读
   - 重点：VFS 适配层源码分析
3. **[05_API_Differences.md](05_API_Differences.md)** - 完整阅读
   - 重点：虚拟分区 API、磁盘 I/O 扩展
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 完整阅读
   - 重点：BUILD.gn 分析、Kconfig 配置详解
5. **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 参考"文件清单"部分

**预期收获**：
- ✅ 理解 OH 的配置文件修改策略
- ✅ 了解 VFS 适配层的实现细节
- ✅ 掌握 OH 新增 API 的使用方法
- ✅ 理解 FatFs 的编译和链接方式

---

## 路线四：升级与维护（2 小时）

**目标**：了解如何升级 FatFs 上游版本或维护 OH 适配

1. **[README.md](README.md)** - 阅读"OH 适配策略"部分
2. **[02_Patches.md](02_Patches.md)** - 完整阅读
   - 重点：ffconf.h 配置对比表、升级建议
3. **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 阅读"0.6 适配策略总结"
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 完整阅读
   - 重点：GNI 文件、BUILD.gn、Kconfig
5. **[06_Security.md](06_Security.md)** - 阅读"CVE 风险"和"升级建议"

**预期收获**：
- ✅ 知道 OH 的适配策略，便于上游版本升级
- ✅ 了解哪些配置需要重新应用
- ✅ 知道新增接口的兼容性要求
- ✅ 理解安全风险评估

---

## 路线五：全面掌握（4 小时+）

**目标**：全面掌握 FatFs 在 OH 中的所有细节，包括历史和设计决策

**完整阅读顺序**：

1. **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 了解项目背景和评估结果
2. **[README.md](README.md)** - 整体概览
3. **[01_Overview.md](01_Overview.md)** - 原始库简介
4. **[02_Patches.md](02_Patches.md)** - Patch 和配置分析
5. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建适配
6. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用
7. **[05_API_Differences.md](05_API_Differences.md)** - API 差异
8. **[06_Security.md](06_Security.md)** - 安全分析
9. **[NOTES.md](wiki/_work/NOTES.md)** - 分析过程记录（可选）

**预期收获**：
- ✅ 全面了解 FatFs 在 OH 中的集成和使用
- ✅ 理解所有 OH 特有功能的设计和实现
- ✅ 掌握源代码级别的细节
- ✅ 具备升级和维护的能力

---

## 按主题阅读

### 了解 OH 特有功能

阅读顺序：
1. **[02_Patches.md](02_Patches.md)** - "OH 特有文件分析"
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - "OH 特有功能"
3. **[05_API_Differences.md](05_API_Differences.md)** - "新增 API"

**涵盖功能**：
- 虚拟分区
- 中文文件名支持
- 增强的磁盘 I/O
- LiteOS 线程安全

### 了解构建系统

阅读顺序：
1. **[03_Build_Integration.md](03_Build_Integration.md)** - 完整阅读
2. **[02_Patches.md](02_Patches.md)** - "ffconf.h 配置详解"
3. **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - "0.4 特殊适配识别"

**涵盖内容**：
- GNI 文件结构
- BUILD.gn 配置
- Kconfig 选项
- 条件编译宏

### 了解安全风险

阅读顺序：
1. **[06_Security.md](06_Security.md)** - 完整阅读
2. **[02_Patches.md](02_Patches.md)** - "安全相关修改"
3. **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - "0.6 适配策略总结"

**涵盖内容**：
- 已知 CVE
- OH 适配引入的风险
- 升级建议

---

## 文档关系图

```
ASSESSMENT.md (工作文件)
    ↓
README.md (入口文档)
    ├─→ 01_Overview.md (原始库简介)
    ├─→ 02_Patches.md (Patch 分析)
    ├─→ 03_Build_Integration.md (构建适配)
    ├─→ 04_Usage_in_OH.md (使用方式)
    ├─→ 05_API_Differences.md (API 差异)
    └─→ 06_Security.md (安全分析)

NOTES.md (工作文件)
PLAN.md (工作文件)
```

---

## 常见问题

### Q1: FatFs 和 ext4 的区别？

**A**: FatFs 主要用于可移动存储设备（SD 卡、USB），支持 FAT/FAT32/exFAT 文件系统，兼容性好但功能相对简单。ext4 是 Linux 标准文件系统，功能更丰富但主要用于内置存储。

在 OH 中：
- **FatFs**：用于 SD 卡、USB 等可移动存储
- **ext4**：用于系统内置存储

### Q2: 为什么 OH 不使用传统 Patch 文件？

**A**: OH 采用**非侵入式适配**策略，通过配置文件修改和新增头文件实现适配，不修改 FatFs 核心代码。这样做的好处是：

- 易于上游版本升级
- 职责分离（适配层与 FatFs 核心独立）
- 减少代码冲突

详见 [ASSESSMENT.md](wiki/_work/ASSESSMENT.md) 的"0.6 适配策略总结"。

### Q3: 如何启用中文文件名支持？

**A**: 在 LiteOS Kconfig 中启用 `FS_FAT_CHINESE` 选项：

```
config FS_FAT_CHINESE
    bool "Enable Chinese"
    default y
    depends on FS_FAT
```

这会在 `ffconf.h` 中设置 `FF_CODE_PAGE = 936`（GBK），支持中文文件名。

详见 [03_Build_Integration.md](03_Build_Integration.md)。

### Q4: 虚拟分区是什么？

**A**: 虚拟分区是 OH 独有的功能，允许在单一 FAT 卷上创建多个逻辑分区。每个虚拟分区有独立的目录结构和访问控制。

相关文件：
- `kernel/liteos_a/fs/fat/virpart/` - 虚拟分区实现
- `source/errcode_fat.h` - 虚拟分区错误码

详见 [05_API_Differences.md](05_API_Differences.md)。

### Q5: 如何升级 FatFs 上游版本？

**A**: 由于 OH 采用非侵入式适配，升级相对简单：

1. 替换 FatFs 核心文件（ff.c, ffunicode.c 等）
2. 重新应用 OH 配置到 ffconf.h
3. 验证新增接口（如 disk_raw_read）的兼容性
4. 测试 OH 特有功能（虚拟分区、中文支持）

详见 [ASSESSMENT.md](wiki/_work/ASSESSMENT.md) 的"升级注意事项"。

---

## 反馈与贡献

如果发现文档错误或有改进建议，请：

1. 在 OpenHarmony Gitee 提交 Issue
2. 提交 Pull Request 修复或改进文档
3. 联系组件维护者：yesiyuan2@huawei.com

---

**文档更新**：2026-02-08
**适用版本**：OpenHarmony 3.x
