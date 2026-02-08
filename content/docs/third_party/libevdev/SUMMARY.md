# 阅读路线建议

本文档为 libevdev 库在 OpenHarmony 中的集成提供全面的技术参考。根据您的角色和需求，选择合适的阅读路径。

## 文档结构

```
libevdev Wiki
├── README.md              # 项目概览与快速导航
├── SUMMARY.md            # 本文档 - 阅读路线建议
├── _work/
│   ├── ASSESSMENT.md     # 项目评估报告（开发者参考）
│   ├── NOTES.md          # 分析过程记录
│   └── PLAN.md           # 任务进度跟踪
├── 01_Overview.md        # 库功能概述
├── 02_Patches.md         # Patch 详细分析（核心文档）
├── 03_Build_Integration.md # 构建适配说明
└── 04_Usage_in_OH.md     # OH 使用场景与依赖关系
```

## 读者指南

### 场景 1：库维护者

如果您负责 libevdev 库在 OpenHarmony 中的维护工作，建议按以下顺序阅读：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | [02_Patches.md](./02_Patches.md) | 了解所有 OH 适配的详细技术细节 |
| 2 | [03_Build_Integration.md](./03_Build_Integration.md) | 构建系统适配与编译选项 |
| 3 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | Patch 维护风险与升级建议 |

**维护者关注要点：**

- Patch 与上游版本的兼容性
- 构建配置的正确性
- 安全相关的适配内容
- 版本升级时的回归风险

### 场景 2：输入系统开发者

如果您需要在 OpenHarmony 中开发输入相关的功能，建议按以下顺序阅读：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 了解 libevdev 的使用场景和 API |
| 2 | [01_Overview.md](./01_Overview.md) | 理解库的基本功能 |
| 3 | [02_Patches.md](./02_Patches.md) | 了解 OH 特有的行为变更 |

**开发者关注要点：**

- 如何正确引入 libevdev 依赖
- OH 特有的 API 扩展
- 输入事件处理的最佳实践
- 调试和问题排查

### 场景 3：安全审计人员

如果您需要评估 libevdev 库的安全性，建议按以下顺序阅读：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | [02_Patches.md](./02_Patches.md) | 分析安全相关的 Patch |
| 2 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | 查看安全评估部分 |
| 3 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 了解攻击面和使用场景 |

**安全审计关注要点：**

- FDSAN 注解的有效性
- 输入事件处理的边界条件
- 潜在的攻击向量
- 与系统安全机制的交互

### 场景 4：版本升级参与者

如果您负责将 libevdev 升级到上游新版本，建议按以下顺序阅读：

| 顺序 | 文档 | 重点内容 |
|------|------|---------|
| 1 | [ASSESSMENT.md](./_work/ASSESSMENT.md) | 查看 Patch 维护建议 |
| 2 | [02_Patches.md](./02_Patches.md) | 逐个分析 Patch 的升级影响 |
| 3 | [03_Build_Integration.md](./03_Build_Integration.md) | 构建配置的兼容性 |

**升级关注要点：**

- 哪些 Patch 可以推向上游
- 哪些 Patch 需要重新适配
- 构建系统的兼容性
- 回归测试的范围

## 关键概念速查

### libevdev 核心概念

| 概念 | 说明 | OH 适配 |
|------|------|-------|
| **libevdev** | evdev 设备包装库 | 无特殊适配 |
| **libevdev-uinput** | uinput 虚拟设备支持 | 添加 FDSAN 安全注解 |
| **libevdev-util** | 工具函数 | 修复位操作 Bug |

### OpenHarmony 特有概念

| 概念 | 说明 | 相关 Patch |
|------|------|----------|
| **FDSAN** | File Descriptor Sanitizer，用于检测文件描述符 Use-After-Free | libevdev-uinput.c |
| **BUS_SDW** | SoundWire 总线类型 | input.h |
| **多点触控同步** | 触摸事件同步机制 | libevdev.c |

## 常见问题

| 问题 | 解答 |
|------|-----|
| 为什么需要 Patch？ | libevdev 需要适配 OpenHarmony 的安全机制（FDSAN）和添加 OH 特有的键值定义 |
| Patch 可以推向上游吗？ | 部分 Bug 修复可以，部分 OH 特有功能不能 |
| 如何调试 libevdev 相关问题？ | 参考 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 中的调试建议 |
| 版本升级要注意什么？ | 参考 [ASSESSMENT.md](./_work/ASSESSMENT.md) 中的升级建议 |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 3.1 | 2026-02-07 | 当前 OH 版本，1 个 Patch |
| - | - | 初始 Wiki 创建 |

---

> 如有疑问，请联系组件负责人：gaoshangqi1@huawei.com
