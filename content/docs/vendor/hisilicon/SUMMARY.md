# Hisilicon Vendor Wiki - 全站导航

## 文档索引

本文档提供 Hisilicon Vendor Wiki 的完整导航，帮助开发者快速定位所需内容。

---

## 快速入门

### 新人必读

| 序号 | 文档 | 说明 | 预计时间 |
|-----|------|------|---------|
| 01 | [项目概述](./01_Overview.md) | 项目定位、目标、相关仓库 | 5 分钟 |
| 02 | [目录结构](./02_Directory_Structure.md) | 代码组织方式 | 10 分钟 |
| 03 | [产品系列](./03_Products.md) | 支持的开发板与芯片 | 10 分钟 |

---

## 核心文档

### 配置指南

| 序号 | 文档 | 说明 | 难度 |
|-----|------|------|------|
| 04 | [配置体系](./04_Configuration.md) | HDF、产品、GN 配置详解 | 中级 |
| 05 | [HDF 配置详解](./04a_HDF_Configuration.md) | 驱动配置语法与示例 | 高级 |
| 06 | [产品配置详解](./04b_Product_Configuration.md) | config.json 结构说明 | 中级 |

### 开发指南

| 序号 | 文档 | 说明 | 难度 |
|-----|------|------|------|
| 07 | [Demo 示例](./05_Demos.md) | 各 Demo 功能与使用 | 初级 |
| 08 | [WiFi Demo 详解](./05a_WiFi_Demo.md) | WiFi STA/AP 模式示例 | 中级 |
| 09 | [传感器 Demo](./05b_Sensor_Demo.md) | 环境传感器使用 | 中级 |

### 构建与部署

| 序号 | 文档 | 说明 |
|-----|------|------|
| 10 | [构建指南](./06_Build.md) | 编译流程与产物 |
| 11 | [GN 构建系统](./06a_GN_Build.md) | 构建文件详解 |

### 安全指南

| 序号 | 文档 | 说明 |
|-----|------|------|
| 12 | [安全风险评审](./07_Security.md) | 安全分析与建议 |

---

## 附录

| 文档 | 说明 |
|-----|------|
| [FAQ](./appendix/FAQ.md) | 常见问题解答 |
| [术语表](./appendix/Glossary.md) | 专业术语说明 |
| [变更历史](./appendix/Changelog.md) | Wiki 更新记录 |

---

## 新人阅读路线推荐

### 路线 A：快速上手（30 分钟）

```
1. [项目概述](./01_Overview.md)  →  了解项目
2. [产品系列](./03_Products.md)  →  选择开发板
3. [Demo 示例](./05_Demos.md)    →  运行示例
```

### 路线 B：深入开发（2 小时）

```
1. [项目概述](./01_Overview.md)      →  了解项目
2. [目录结构](./02_Directory_Structure.md)  →  熟悉结构
3. [产品系列](./03_Products.md)      →  选择开发板
4. [配置体系](./04_Configuration.md) →  掌握配置
5. [Demo 示例](./05_Demos.md)        →  学习示例
6. [构建指南](./06_Build.md)        →  构建项目
```

### 路线 C：系统掌握（4 小时）

```
完成路线 B 后，额外阅读：
- [HDF 配置详解](./04a_HDF_Configuration.md)  →  驱动开发
- [GN 构建系统](./06a_GN_Build.md)          →  构建定制
- [安全风险评审](./07_Security.md)          →  安全加固
```

---

## 按角色导航

### 设备开发者

```
[产品系列](./03_Products.md) → [HDF 配置](./04a_HDF_Configuration.md) → [Demo 示例](./05_Demos.md)
```

### 系统集成商

```
[配置体系](./04_Configuration.md) → [构建指南](./06_Build.md) → [安全指南](./07_Security.md)
```

### 安全工程师

```
[配置体系](./04_Configuration.md) → [安全风险评审](./07_Security.md)
```

### 项目新人

```
推荐路线 A 或 B（见上文）
```

---

## 版本兼容性

| 文档版本 | OpenHarmony 版本 | 说明 |
|---------|-----------------|------|
| v1.0 | OpenHarmony 3.0+ | 初始版本 |
| v1.1 | OpenHarmony 4.0+ | 新增安全章节 |

---

## 贡献者

本 Wiki 由 OpenHarmony Hisilicon Team 维护。

---

*最后更新：2026-02-06*
