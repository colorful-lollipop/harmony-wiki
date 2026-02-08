# 阅读路线建议

本文档提供针对不同读者的文档阅读路线，帮助您快速找到所需信息。

---

## 按角色分类

### OpenHarmony 系统开发者

**目标**：了解 cangjie_tools 在 OpenHarmony 中的使用方式、依赖关系和集成点

**阅读顺序**：
1. [README.md](./README.md) - 快速了解库概览和 OH 适配概述
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解依赖关系和典型使用场景
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解 OH 特定的构建配置
4. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看完整的项目评估结果

**重点关注**：
- 该工具链是否有运行时依赖（答案：无）
- 如何集成到 OH 构建系统（答案：当前使用独立构建系统）
- OH 特定的编译选项和配置

---

### Cangjie 语言用户 / 开发者

**目标**：了解工具链的功能、使用方法和最佳实践

**阅读顺序**：
1. [README.md](./README.md) - 了解工具链组成和 OH 适配概述
2. [01_Overview.md](./01_Overview.md) - 了解原始库的功能和版本信息
3. 各工具的官方文档（在 `doc/` 目录下）

**重点关注**：
- 每个工具的功能和用途
- 如何安装和使用这些工具
- 工具之间的依赖关系

---

### 构建系统维护者

**目标**：了解 OH 特定的构建配置，以便集成或修改构建系统

**阅读顺序**：
1. [README.md](./README.md) - 了解构建系统适配概述
2. [03_Build_Integration.md](./03_Build_Integration.md) - 深入了解构建系统结构和 OH 特定配置
3. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看构建系统的详细评估

**重点关注**：
- 为什么不使用 GN 构建系统（答案：采用自有 Python + CMake 架构）
- 如何集成到 OH 主构建系统
- OH 特定的编译器、链接器选项
- 第三方依赖的管理方式

---

### 版本升级维护者

**目标**：了解上游版本同步机制和升级注意事项

**阅读顺序**：
1. [README.md](./README.md) - 了解维护策略
2. [02_Patches.md](./02_Patches.md) - 了解是否有 Patch 文件（答案：无）
3. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看 Git 维护记录

**重点关注**：
- 上游版本同步机制（自动同步脚本）
- 是否需要重新应用 patch（答案：不需要）
- OH 特定修改如何处理（直接修改源文件）

---

### DevEco Studio 集成开发者

**目标**：了解 LSP 和 ArkTS 互操作的集成细节

**阅读顺序**：
1. [README.md](./README.md) - 了解 OH 适配概述中的 DevEco 集成部分
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解 LSP 的 OH 特定配置
3. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看 OH 特有功能适配

**重点关注**：
- DevEco 环境检测机制
- OHOS 特定的 CJD 索引
- Interop 自动导入
- ArkTS 互操作代码生成

---

## 按主题分类

### 主题 1：工具链功能

**相关文档**：
- [01_Overview.md](./01_Overview.md) - 原始库功能介绍
- [README.md](./README.md) - 工具链组成表

---

### 主题 2：OpenHarmony 适配

**相关文档**：
- [README.md](./README.md) - OH 适配概述
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统适配
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - OH 特有功能适配

---

### 主题 3：构建系统

**相关文档**：
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统详解
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 构建文件清单和配置分析

---

### 主题 4：依赖关系

**相关文档**：
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - OH 依赖关系和使用情况
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 依赖者搜索结果

---

### 主题 5：Patch 管理

**相关文档**：
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 仓库维护方式

---

### 主题 6：DevEco Studio 集成

**相关文档**：
- [README.md](./README.md) - DevEco 集成概述
- [03_Build_Integration.md](./03_Build_Integration.md) - LSP OH 特定配置
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - DevEco 集成机制

---

### 主题 7：ArkTS 互操作

**相关文档**：
- [README.md](./README.md) - ArkTS 互操作概述
- [03_Build_Integration.md](./03_Build_Integration.md) - HLE 工具配置
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - HLE 工具 OH 特定功能

---

## 快速参考

### 常见问题

**Q: 该库有 Patch 文件吗？**
A: 没有。采用直接同步上游代码的方式。

**Q: 该库使用 GN 构建系统吗？**
A: 不使用。采用 Python + CMake 混合架构。

**Q: 该库被其他 OH 模块依赖吗？**
A: 没有。作为独立开发工具使用。

**Q: 支持 OHOS 目标平台吗？**
A: 支持。包括 `ohos-x86_64` 和 `ohos-aarch64`（cjfmt 工具）。

**Q: 如何集成到 OH 主构建系统？**
A: 当前未集成。如需集成，需要创建对应的 BUILD.gn 文件。

---

### 关键文件位置

| 文件 | 路径 |
|------|------|
| bundle.json | `./bundle.json` |
| cjpm 构建脚本 | `./cjpm/build/build.py` |
| cjfmt 构建脚本 | `./cjfmt/build/build.py` |
| LSP 构建脚本 | `./cangjie-language-server/build/build.py` |
| OH 特定测试 | `./cangjie-language-server/test/testChr/crossLanguageDefinition/ohos/` |

---

### 核心概念

| 概念 | 说明 |
|--------|------|
| **直接同步** | 不使用 Patch，直接从上游同步完整代码 |
| **独立构建系统** | 不使用 GN，自有 Python + CMake 架构 |
| **OHOS 目标** | `ohos-x86_64`, `ohos-aarch64` 编译目标 |
| **DevEco 集成** | LSP 深度集成 DevEco Studio |
| **ArkTS 互操作** | HLE 工具生成跨语言桥接代码 |

---

## 延伸阅读

**相关项目**：
- [cangjie_compiler](https://gitcode.com/Cangjie/cangjie_compiler) - Cangjie 编译器
- [cangjie_stdx](https://gitcode.com/Cangjie/cangjie_stdx) - Cangjie 标准库
- [cangjie_build](https://gitcode.com/Cangjie/cangjie_build) - Cangjie 构建系统
- [cangjie_docs](https://gitcode.com/Cangjie/cangjie_docs) - Cangjie 文档

**OpenHarmony 相关**：
- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [DevEco Studio 文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-overview-V5)
