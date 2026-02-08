# Thermal Manager Wiki 站点导航

## 概述

本文档提供 thermal_manager 项目的完整导航，帮助新人按正确顺序阅读文档，快速理解项目。

## 更新记录

| 版本 | 日期 | 更新内容 |
|---|---|---|
| v1.0 | 2026-02-06 | 初版完成，覆盖所有核心文档 |

---

## 推荐阅读顺序

### 新人路径（按顺序阅读）

推荐新开发者按以下顺序阅读，系统学习项目：

1. **[00_Overview.md](00_Overview.md)** ⭐ 必读
   - 项目概览、核心能力
   - 运行环境
   - 系统能力

2. **[01_Module_Boundaries.md](01_Module_Boundaries.md)**
   - 模块定位与边界
   - 核心能力清单
   - 权限模型
   - 运行环境要求

3. **[02_Directory_Structure.md](02_Directory_Structure.md)**
   - 完整目录树
   - 各模块职责说明
   - 代码统计

4. **[03_Architecture.md](03_Architecture.md)** ⭐ 必读
   - 组件架构图
   - 数据流向
   - 线程模型
   - 关键时序

5. **[04_NAPI_Interface.md](04_NAPI_Interface.md)** ⭐ 必读
   - 完整 N-API 清单
   - API 参数和返回值
   - 调用链说明
   - 错误处理

6. **[05_Inner_API.md](05_Inner_API.md)**
   - 内部 API 接口
   - 模块依赖关系
   - 可替换点
   - 稳定性标注

7. **[06_GN_Targets.md](06_GN_Targets.md)**
   - GN Targets 详解
   - 编译产物说明
   - 特性开关
   - 依赖系统组件

8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)**
   - 编译产物清单
   - 安装路径
   - 运行时加载关系
   - HDI 依赖

9. **[08_Security_Review.md](08_Security_Review.md)** ⭐ 必读
   - 威胁模型
   - 攻击面分析
   - 可利用点清单
   - 安全加固建议

10. **[09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md)**
   - 常见构建问题
   - 运行时问题
   - 调试技巧
   - 定位路径

---

## 按主题阅读

### 想快速了解 API？

→ 阅读：**[04_NAPI_Interface.md](04_NAPI_Interface.md)**

### 想理解架构设计？

→ 阅读：**[03_Architecture.md](03_Architecture.md)**

### 想进行安全开发或审计？

→ 阅读：**[08_Security_Review.md](08_Security_Review.md)**

### 遇到构建或运行问题？

→ 阅读：**[09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md)**

### 想修改构建配置？

→ 阅读：**[06_GN_Targets.md](06_GN_Targets.md)**

### 想了解模块职责？

→ 阅读：**[02_Directory_Structure.md](02_Directory_Structure.md)**

### 想了解内部接口？

→ 阅读：**[05_Inner_API.md](05_Inner_API.md)**

---

## 进阶阅读

### 附录文档

以下文档提供详细的补充信息，适合深入阅读：

11. **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
    - 温度监控流程
    - 回调通知流程
    - 策略执行流程

12. **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 配置宏说明
    - Feature flags 列表
    - 条件编译宏
    - 默认配置值

---

## 快速参考

### 按任务查找文档

| 任务 | 文档 |
|---|---|
| 我要调用 JS API | [04_NAPI_Interface.md](04_NAPI_Interface.md) |
| 我要编写 Native 代码 | [05_Inner_API.md](05_Inner_API.md) + [01_Module_Boundaries.md](01_Module_Boundaries.md) |
| 我要修改热策略 | [08_Security_Review.md](08_Security_Review.md) + [03_Architecture.md](03_Architecture.md) |
| 我要编译项目 | [06_GN_Targets.md](06_GN_Targets.md) |
| 我要调试问题 | [09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md) |
| 我要审计安全性 | [08_Security_Review.md](08_Security_Review.md) |

### 按角色查找文档

| 角色 | 推荐文档 |
|---|---|
| 新人开发者 | [00_Overview.md](00_Overview.md) → [02_Directory_Structure.md](02_Directory_Structure.md) → [04_NAPI_Interface.md](04_NAPI_Interface.md) |
| 架构师 | [00_Overview.md](00_Overview.md) → [03_Architecture.md](03_Architecture.md) → [05_Inner_API.md](05_Inner_API.md) |
| 安全工程师 | [00_Overview.md](00_Overview.md) → [08_Security_Review.md](08_Security_Review.md) |
| 构建工程师 | [00_Overview.md](00_Overview.md) → [06_GN_Targets.md](06_GN_Targets.md) |

---

## 文档状态

| 文档 | 状态 | 描述 |
|---|---|---|
| [00_Overview.md](00_Overview.md) | ✅ 完成 | 项目概览，核心概念 |
| [01_Module_Boundaries.md](01_Module_Boundaries.md) | ✅ 完成 | 模块边界，能力清单 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | ✅ 完成 | 目录结构，模块职责 |
| [03_Architecture.md](03_Architecture.md) | ✅ 完成 | 构构图，数据流，时序 |
| [04_NAPI_Interface.md](04_NAPI_Interface.md) | ✅ 完成 | N-API 清单，API 详情 |
| [05_Inner_API.md](05_Inner_API.md) | ✅ 完成 | 内部 API，依赖关系 |
| [06_GN_Targets.md](06_GN_Targets.md) | ✅ 完成 | GN Targets，编译产物 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | ✅ 完成 | 产物清单，安装路径 |
| [08_Security_Review.md](08_Security_Review.md) | ✅ 完成 | 安全风险，修复建议 |
| [09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md) | ✅ 完成 | FAQ，调试技巧 |

---

## 文档覆盖矩阵

### 核心功能覆盖

| 功能 | 文档 | 详细程度 |
|---|---|---|
| 项目概览 | [00_Overview.md](00_Overview.md) | ⭐⭐⭐ |
| 架构设计 | [03_Architecture.md](03_Architecture.md) | ⭐⭐⭐ |
| JS API | [04_NAPI_Interface.md](04_NAPI_Interface.md) | ⭐⭐⭐ |
| 内部 API | [05_Inner_API.md](05_Inner_API.md) | ⭐⭐ |
| 构建系统 | [06_GN_Targets.md](06_GN_Targets.md) | ⭐⭐ |
| 编译产物 | [07_Build_Artifacts.md](07_Build_Artifacts.md) | ⭐⭐ |
| 安全评审 | [08_Security_Review.md](08_Security_Review.md) | ⭐⭐ |

### 技术覆盖

| 技术领域 | 文档 | 详细程度 |
|---|---|---|
| 模块设计 | [01_Module_Boundaries.md](01_Module_Boundaries.md), [02_Directory_Structure.md](02_Directory_Structure.md), [05_Inner_API.md](05_Inner_API.md) | ⭐⭐⭐ |
| 架构细节 | [03_Architecture.md](03_Architecture.md), [appendix/Callgraphs.md](appendix/Callgraphs.md) | ⭐⭐ |
| 构建配置 | [06_GN_Targets.md](06_GN_Targets.md), [appendix/Config_Flags.md](appendix/Config_Flags.md) | ⭐⭐ |
| 调试与运维 | [09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md) | ⭐⭐ |

---

## 文档维护指南

### 如何更新文档

**规则**:
1. 所有结论必须包含代码证据（文件路径 + 符号）
2. 不得引用测试代码作为业务证据
3. 更新后同步更新版本号和日期
4. 保持术语一致性

**流程**:
1. 修改代码后，识别影响的文档
2. 更新相关章节
3. 更新文档顶部的版本记录
4. 检查 `SUMMARY.md` 中的链接有效性

---

## 快速入门

### 5 分钟快速了解

如果你只有 5 分钟时间，按顺序阅读：

1. **[00_Overview.md](00_Overview.md)** (2 分钟) - 了解项目定位和核心能力
2. **[03_Architecture.md](03_Architecture.md)** (3 分钟) - 理解架构和数据流

### 15 分钟快速了解

如果你有 15 分钟时间：

1. **[00_Overview.md](00_Overview.md)** (2 分钟)
2. **[04_NAPI_Interface.md](04_NAPI_Interface.md)** (5 分钟) - 学习 JS API 使用
3. **[08_Security_Review.md](08_Security_Review.md)** (8 分钟) - 了解安全注意事项

### 30 分钟深入学习

如果你有 30 分钟时间：

按"新人路径"顺序阅读所有核心文档（1-10）

### 深度学习

按"进阶阅读"顺序，阅读附录文档（11-12）

---

## 总结

本 Wiki 提供了一套完整的文档，帮助不同角色的开发者快速了解和使用 thermal_manager 项目。

**文档特点**:
- ✅ 结构清晰：按主题组织，便于查找
- ✅ 证据充分：所有关键结论包含代码证据
- ✅ 导航友好：提供多种阅读路径
- ✅ 可维护：包含更新指南和版本记录

**文档完整性**:
- 12 个文档文件
- 覆盖项目所有关键方面
- 包含快速参考和进阶阅读

**建议**:
- 新开发者从 [00_Overview.md](00_Overview.md) 开始
- 应用开发者重点关注 [04_NAPI_Interface.md](04_NAPI_Interface.md) 和 [08_Security_Review.md](08_Security_Review.md)
- 架构师和构建工程师参考 [03_Architecture.md](03_Architecture.md) 和 [06_GN_Targets.md](06_GN_Targets.md)
- 遇到问题查看 [09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md)
