# SUMMARY.md - 阅读路线建议

本文档提供 tzdata Wiki 的阅读路线建议，帮助不同需求的读者快速找到所需信息。

---

## 快速概览（5 分钟）

适合想快速了解本库在 OH 中作用的读者：

1. **[README.md](./README.md)** - 库概览和文档导航
2. **[01_Overview.md](./01_Overview.md)** - 原始库简介和 OH 定位

---

## 深度阅读（30 分钟）

适合需要全面了解本库 OH 适配细节的读者：

### 第一阶段：背景了解
1. **[01_Overview.md](./01_Overview.md)** - 了解 tzdata 是什么，在 OH 中的作用
2. **[02_Patches.md](./02_Patches.md)** - Patch 分析（本库无 Patch，但需了解原因）

### 第二阶段：技术实现
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 理解 BUILD.gn 的适配方式
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解谁在使用，如何使用

### 第三阶段：维护和安全
5. **[06_Security.md](./06_Security.md)** - 安全风险和维护策略

---

## 按场景阅读

### 场景 1：我是系统开发者，需要了解如何使用 tzdata

阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解基本概念
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖关系和 API
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建集成（如需修改）

### 场景 2：我需要升级 tzdata 到新版本

阅读顺序：
1. [02_Patches.md](./02_Patches.md) - 确认无 Patch 需要迁移
2. [03_Build_Integration.md](./03_Build_Integration.md) - 检查 BUILD.gn 是否需要调整
3. [06_Security.md](./06_Security.md) - 查看版本更新建议

### 场景 3：我在调查与时区相关的 Bug

阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 理解 tzdata 在 OH 中的定位
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 确认依赖关系，定位问题来源
3. [02_Patches.md](./02_Patches.md) - 确认无 OH 特有修改导致问题

### 场景 4：我需要评估安全风险

阅读顺序：
1. [06_Security.md](./06_Security.md) - 完整的安全分析
2. [02_Patches.md](./02_Patches.md) - 确认无额外攻击面

---

## 文档速查表

| 问题 | 参考文档 |
|-----|---------|
| tzdata 是什么？ | [01_Overview.md](./01_Overview.md) |
| OH 有哪些 Patch？ | [02_Patches.md](./02_Patches.md) |
| BUILD.gn 如何配置？ | [03_Build_Integration.md](./03_Build_Integration.md) |
| 谁在使用 tzdata？ | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| 安全风险如何？ | [06_Security.md](./06_Security.md) |
| 版本信息在哪里？ | [README.md](./README.md), [01_Overview.md](./01_Overview.md) |

---

## 附加资源

- **[ASSESSMENT.md](./_work/ASSESSMENT.md)** - 项目评估原始数据和分析过程

