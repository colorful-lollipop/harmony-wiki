# HiHope Vendor Wiki 导航

本文档目录提供新人快速理解 OpenHarmony vendor_hihope 仓库的阅读路径。

## 新人推荐阅读顺序

### 第一阶段：快速入门（30 分钟）

1. **[首页概览](00_Overview.md)** ⭐ 从这里开始
   - 了解仓库整体定位和架构
   - 理解 vendor 仓库的作用
   - 查看产品线概览

2. **[产品定位与边界](01_Product_Positioning.md)**
   - 理解各产品线的适用场景
   - 了解产品类型差异（standard/mini）
   - 查看核心能力列表

3. **[目录结构与模块职责](02_Directory_Structure.md)**
   - 快速浏览目录组织
   - 理解标准功能区域
   - 查看各模块职责

### 第二阶段：核心架构理解（60 分钟）

4. **[HAL 实现说明](03_HAL_Implementation.md)** ⭐ 重点关注
   - 了解硬件抽象层实现
   - 查看可用的 HAL 接口
   - 理解 HAL 适配模式

5. **[HDF 配置详解](04_HDF_Configuration.md)** ⭐ 重点关注
   - 理解硬件驱动框架
   - 查看 UHDF 和 KHDF 服务配置
   - 学习 HCS 配置语法

6. **[内部 API 与模块接口](05_Internal_API.md)**
   - 了解内部模块依赖关系
   - 查看模块间接口
   - 理解数据流和线程模型

### 第三阶段：构建与部署（45 分钟）

7. **[GN Targets 梳理](06_GN_Targets.md)** ⭐ 实用指南
   - 理解构建系统结构
   - 查看主要 targets 和产物
   - 学习 GN 配置语法

8. **[编译产物说明](07_Build_Artifacts.md)**
   - 了解最终产物（.so/.a/.hap 等）
   - 查看安装路径
   - 理解运行时加载关系

### 第四阶段：安全与权限（45 分钟）

9. **[安全风险评审](08_Security_Review.md)** ⭐ 必读
   - 了解攻击面和信任边界
   - 查看安全机制
   - 学习权限模型和签名验证

### 第五阶段：实践与排查（按需）

10. **[常见问题与定位](09_Common_Issues.md)**
    - 查看常见构建问题
    - 学习运行时调试技巧
    - 了解问题排查路径

11. **[附录：调用链](appendix/Callgraphs.md)**（可选）
    - 查看关键 API 调用链
    - 理解完整执行流程

12. **[附录：配置标志](appendix/Config_Flags.md)**（可选）
    - 查看关键宏和 feature flags
    - 了解编译开关说明

## 快速索引

### 按主题查找

| 主题 | 相关文档 |
|------|---------|
| **概览与入门** | [首页概览](00_Overview.md), [产品定位](01_Product_Positioning.md) |
| **架构与接口** | [目录结构](02_Directory_Structure.md), [HAL 实现](03_HAL_Implementation.md), [HDF 配置](04_HDF_Configuration.md), [内部 API](05_Internal_API.md) |
| **构建与部署** | [GN Targets](06_GN_Targets.md), [编译产物](07_Build_Artifacts.md) |
| **安全与权限** | [安全评审](08_Security_Review.md) |
| **问题排查** | [常见问题](09_Common_Issues.md) |
| **附录参考** | [调用链](appendix/Callgraphs.md), [配置标志](appendix/Config_Flags.md) |

### 按产品查找

| 产品线 | 相关文档 |
|---------|---------|
| **RK3568（完整）** | [RK3568 专项文档](./products/rk3568/) |
| **NearLink DK-3863** | [NearLink 专项文档](./products/nearlink_dk_3863/) |
| **Mini 系统** | [Mini 系统说明](./products/mini_system/) |
| **其他产品** | 各产品的 config.json 说明 |

## 文档说明

### 文档标记说明

- ⭐ **重点关注**：核心必读文档
- ⏸ **已跳过**：本仓库不涉及的内容
- 🔧 **实用指南**：实践性强，可直接应用的文档

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

*提示：本 Wiki 专为 OpenHarmony vendor 开发和适配工作设计。*
