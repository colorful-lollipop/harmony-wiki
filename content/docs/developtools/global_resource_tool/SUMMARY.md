# global_resource_tool Wiki 导航

> **项目名称**：restool（OpenHarmony 资源编译工具）
> **最后更新**：2026-02-07
> **阅读时长**：新人路线约 30 分钟，安全路线约 45 分钟

---

## 📚 推荐阅读路线

### 🔰 新人学习路线（30 分钟）

| 阶段 | 文档 | 预期收获 | 时长 |
|------|------|---------|------|
| **Step 1** | [README](README.md) | 了解 Wiki 覆盖范围和导航方式 | 1 分钟 |
| **Step 2** | [00_Overview](00_Overview.md) | 理解项目定位：一句话定义 + 能力边界 | 3 分钟 |
| **Step 3** | [01_Directory_Structure](01_Directory_Structure.md) | 熟悉代码组织方式，定位核心文件 | 5 分钟 |
| **Step 4** | [02_Architecture](02_Architecture.md) | 理解系统架构、数据流、线程模型 | 10 分钟 |
| **Step 5** | [03_Public_API](03_Public_API.md) | 掌握命令行使用方法 | 8 分钟 |
| **Step 6** | [08_FAQ](08_FAQ.md) | 快速解决常见问题 | 3 分钟 |

> **新人目标**：完成路线后能在 15 分钟内找到任意核心代码位置，30 分钟内理解基本架构。

---

### 🔒 安全研究路线（45 分钟）

| 阶段 | 文档 | 预期收获 | 时长 |
|------|------|---------|------|
| **Step 1** | [README](README.md) | 了解 Wiki 覆盖范围 | 1 分钟 |
| **Step 2** | [00_Overview](00_Overview.md) | 理解项目攻击面范围 | 3 分钟 |
| **Step 3** | [05_AttackSurface](05_AttackSurface.md) | **⭐ 核心**：识别所有外部输入入口和信任边界 | 10 分钟 |
| **Step 4** | [06_SecurityReview](06_SecurityReview.md) | **⭐ 核心**：理解风险点、利用路径、修复建议 | 15 分钟 |
| **Step 5** | [02_Architecture](02_Architecture.md) | 对照架构理解数据流中的安全检查点 | 8 分钟 |
| **Step 6** | [03_Public_API](03_Public_API.md) | 理解命令行参数的安全影响 | 5 分钟 |
| **Step 7** | [08_Build_Artifacts](08_Build_Artifacts.md) | 了解构建产物的安全属性 | 3 分钟 |

> **安全研究员目标**：完成路线后能快速识别所有攻击面，定位任意风险的代码位置和利用路径。

---

## 📋 全站导航

### 核心文档

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [README](README.md) | Wiki 首页 | 覆盖范围、更新方式、快速导航 |
| [00_Overview.md](00_Overview.md) | 项目概览 | 定位、边界、核心能力、运行环境 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构 | 模块职责、文件组织 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 | 组件图、数据流、线程模型、时序图 |
| [03_Public_API.md](03_Public_API.md) | 对外接口 | 命令行参数、子命令、错误码 |
| [04_Internal_API.md](04_Internal_API.md) | 内部接口 | 模块接口、依赖方向、稳定性 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 | 输入清单、信任边界、敏感操作 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评审 | 风险详情、利用路径、修复建议 |
| [07_GN_Targets.md](07_GN_Targets.md) | GN 构建目标 | targets 列表、依赖、产物 |
| [08_Build_Artifacts.md](08_Build_Artifacts.md) | 编译产物 | 产物清单、安装路径、加载关系 |
| [08_FAQ.md](08_FAQ.md) | 常见问题 | 构建/运行/调试问题 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏和配置项 |

## 按角色导航

### 如果你是...

**新加入的开发者**
→ 阅读路线：概览 → 目录结构 → 架构说明 → 对外接口

**需要集成 restool 的构建工程师**
→ 重点阅读：GN 构建目标、编译产物、对外接口

**进行安全审计的人员**
→ 重点阅读：安全风险评审、架构说明、内部接口

**解决使用问题的开发者**
→ 重点阅读：常见问题、对外接口、错误码参考

**需要扩展功能的开发者**
→ 重点阅读：架构说明、内部接口、调用链附录

## 术语速查

| 术语 | 说明 |
|------|------|
| restool | Resource Tool 的缩写，资源编译工具 |
| HAP | HarmonyOS Ability Package，应用包 |
| HSP | Harmony Shared Package，共享包 |
| HAR | Harmony Archive，静态共享包 |
| Resource Index | 资源索引文件（resources.index） |
| Resource Table | 资源表头文件（ResourceTable.h/js/txt） |
| ID Defined | 资源 ID 定义文件（id_defined.json） |
| 限定词 | 资源目录中的配置标识（如 zh_CN、phone） |
