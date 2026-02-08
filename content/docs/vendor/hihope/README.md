# HiHope Vendor Wiki

本 Wiki 文档提供对 OpenHarmony vendor_hihope 仓库的全面技术文档。

## 覆盖范围

本文档覆盖以下内容：
- 仓库整体架构和产品线概述
- 目录结构与模块职责
- HAL（硬件抽象层）实现
- HDF（硬件驱动框架）配置
- 安全机制与权限模型
- 构建系统（GN）说明
- 产品配置差异
- 常见问题与排查路径

## 未覆盖范围

以下内容**不在**本文档覆盖范围内：
- N-API（JavaScript API）实现（位于 OpenHarmony 主源码树）
- IPC/ServiceAbility 实际 C++ 实现（位于 OpenHarmony 主源码树）
- 系统内核实现
- 第三方库实现

**原因**：本仓库为 vendor 仓库，仅包含产品配置、HAL 适配层、HDF 配置等，不包含系统框架核心代码。

## 生成信息

- **生成时间**：2025-02-06
- **仓库路径**：`/Volumes/lexar/code/d/work/oh/vendor/hihope`
- **仓库类型**：OpenHarmony Vendor Repository
- **许可证**：Apache License 2.0
- **产品数量**：13 个产品线
- **构建文件**：103 个 BUILD.gn，35 个 .gni
- **文档数量**：12 个核心 Markdown 文档

## 已创建文档列表

### 核心文档（12 个）
1. **[00_Overview.md](00_Overview.md)** - 仓库概览、产品线、核心能力、架构层次
2. **[01_Product_Positioning.md](01_Product_Positioning.md)** - 产品定位、边界、适用场景、选型建议
3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 完整目录树、模块职责、功能区域
4. **[03_HAL_Implementation.md](03_HAL_Implementation.md)** - HAL 接口、实现模式、开发指南
5. **[04_HDF_Configuration.md](04_HDF_Configuration.md)** - HDF 配置、服务定义、语法、加载顺序
6. **[05_Internal_API.md](05_Internal_API.md)** - 内部 API、依赖关系、线程模型、资源生命周期
7. **[06_GN_Targets.md](06_GN_Targets.md)** - GN 模板、Targets 结构、依赖关系、调试技巧
8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物类型、生成位置、运行时加载
9. **[08_Security_Review.md](08_Security_Review.md)** - 攻击面、信任边界、可被利用点

### N-API 文档（1 个）
10. **[06_NAPI_Reference.md](06_NAPI_Reference.md)** - N-API 引用说明、实际实现位置（vendor 仓库不包含实现）

### 附录文档（1 个）
11. **[09_Common_Issues.md](09_Common_Issues.md)** - 常见构建/运行/调试问题、排查路径

## 文档导航

### 新人推荐阅读顺序（按阶段）

**第一阶段：快速入门（30 分钟）**
1. **[首页概览](00_Overview.md)** ⭐ 从这里开始
   - 了解仓库整体定位和架构
   - 理解 vendor 仓库的作用
   - 查看产品线概览
2. **[产品定位与边界](01_Product_Positioning.md)**
   - 理解各产品线的适用场景
   - 了解产品类型差异（standard/mini）
   - 查看核心能力列表

**第二阶段：核心架构理解（60 分钟）**
3. **[目录结构与模块职责](02_Directory_Structure.md)**
   - 快速浏览目录组织
   - 理解标准功能区域
   - 查看各模块职责
4. **[HAL 实现说明](03_HAL_Implementation.md)** ⭐ 重点关注
   - 了解硬件抽象层实现
   - 查看可用的 HAL 接口
   - 理解 HAL 适配模式
5. **[HDF 配置详解](04_HDF_Configuration.md)** ⭐ 重点关注
   - 理解硬件驱动框架
   - 查看 UHDF 和 KHDF 服务配置
   - 学习 HCS 配置语法

**第三阶段：构建与部署（45 分钟）**
6. **[GN Targets 梳理](06_GN_Targets.md)** ⭐ 实用指南
   - 理解构建系统结构
   - 查看主要 targets 和产物
   - 学习 GN 配置语法
7. **[编译产物说明](07_Build_Artifacts.md)**
   - 了解最终产物类型
   - 查看安装路径
   - 理解运行时加载关系

**第四阶段：安全与权限（45 分钟）**
8. **[安全风险评审](08_Security_Review.md)** ⭐ 必读
   - 了解攻击面和信任边界
   - 查看安全机制
   - 学习权限模型和签名验证
   - 识别关键安全控制点

**第五阶段：实践与排查（按需）**
9. **[常见问题与定位](09_Common_Issues.md)**
   - 查看常见构建问题
   - 学习运行时调试技巧
   - 了解问题排查路径

## 快速索引

### 按主题查找

| 主题 | 相关文档 |
|------|---------|
| **概览与入门** | [首页概览](00_Overview.md), [产品定位](01_Product_Positioning.md) |
| **架构与接口** | [目录结构](02_Directory_Structure.md), [HAL 实现](03_HAL_Implementation.md), [HDF 配置](04_HDF_Configuration.md), [内部 API](05_Internal_API.md) |
| **构建与部署** | [GN Targets](06_GN_Targets.md), [编译产物](07_Build_Artifacts.md) |
| **安全与权限** | [安全评审](08_Security_Review.md) |
| **问题排查** | [常见问题](09_Common_Issues.md) |

### 按产品查找

| 产品线 | 相关文档 |
|---------|----------|
| **RK3568（完整）** | [RK3568 专项文档](./products/rk3568/)（待创建） |
| **NearLink DK-3863** | [NearLink 专项文档](./products/nearlink_dk_3863/)（待创建） |
| **Mini 系统** | [Mini 系统说明](./products/mini_system/)（待创建） |
| **其他产品** | 各产品的 config.json 说明 | - |

## 文档说明

### 文档标记说明

- ⭐ **重点关注**：核心必读文档（标记为 ⭐）
- ⏸ **已跳过**：本仓库不涉及的内容（标记为 ⏸）
- 🔧 **实用指南**：实践性强，可直接应用的文档（标记为 🔧）

### 关键结论的证据来源

本文档中的所有关键结论都基于代码证据，包括：

- **文件路径**：精确到文件名和行号（如适用）
- **符号名**：函数、类、宏、target 名称
- **配置示例**：关键配置文件的完整或片段
- **代码片段**：最小必要的实现片段

无法确认的内容会标记为 `TODO(需确认)`。

### 版本与更新

- **当前版本**：v1.0
- **代码快照日期**：2025-02-06
- **最后更新时间**：2025-02-06

如需了解最新变更，请参考 Git 仓库历史或 OpenHarmony 官方文档。

## 反馈与贡献

如发现文档问题或有改进建议，欢迎：

1. 提交 Issue 到代码仓库
2. 参考代码仓库的贡献流程
3. 遵循 Commit Message 规范

---

*本 Wiki 专为 OpenHarmony vendor 开发和适配工作设计。*
