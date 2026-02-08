# Accessibility 子系统文档导航

## 📚 文档目录

### 新人入门路线

建议按以下顺序阅读文档：

1. **[00_Overview.md](00_Overview.md)** - 项目概览
2. **[01_Project_Scope.md](01_Project_Scope.md)** - 项目定位与边界
3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构与模块职责
4. **[03_Architecture.md](03_Architecture.md)** - 架构设计
5. **[04_N-API.md](04_N-API.md)** - 对外 N-API 接口
6. **[05_Inner_API.md](05_Inner_API.md)** - 内部 API
7. **[06_GN_Targets.md](06_GN_Targets.md)** - GN 构建系统
8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物
9. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险评审
10. **[09_FAQ.md](09_FAQ.md)** - 常见问题

---

## 📖 核心文档

### [00_Overview.md](00_Overview.md)
**目的**: 项目总体介绍

**内容**:
- 项目背景与目标
- 主要功能场景
- 架构概览
- 快速开始

**适合读者**: 所有关注本项目的开发者

---

### [01_Project_Scope.md](01_Project_Scope.md)
**目的**: 定义项目定位、边界和核心能力

**内容**:
- 项目定位
- 核心能力清单
- 不在项目范围内的功能
- 运行环境要求
- 关键概念说明

**适合读者**: 需要了解项目边界的开发者

---

### [02_Directory_Structure.md](02_Directory_Structure.md)
**目的**: 说明代码组织结构和模块职责

**内容**:
- 完整目录树（排除测试）
- 各目录职责说明
- 关键文件清单
- 模块组织关系

**适合读者**: 需要理解代码结构的开发者

---

### [03_Architecture.md](03_Architecture.md)
**目的**: 详细说明系统架构设计

**内容**:
- 架构分层说明
- 组件图与依赖关系
- 数据流与时序图
- 线程模型
- 关键设计决策

**适合读者**: 需要深入理解架构的开发者

---

### [04_N-API.md](04_N-API.md)
**目的**: 对外 JS/TS API 的完整文档

**内容**:
- N-API 模块清单
- API 清单表（JS 名称、参数、返回值、C++ 实现）
- 参数校验机制
- 同步/异步模式
- 错误码说明
- 权限要求
- 调用链示例

**适合读者**: 使用无障碍 API 的应用开发者

---

### [05_Inner_API.md](05_Inner_API.md)
**目的**: 内部 C/C++ API 参考

**内容**:
- Inner Kits 接口清单
- 模块间依赖方向
- 接口稳定性标注
- 可替换点说明
- 内部 API 使用指南

**适合读者**: 内部开发者、贡献者

---

### [06_GN_Targets.md](06_GN_Targets.md)
**目的**: GN 构建系统说明

**内容**:
- 主要 BUILD.gn 文件列表
- 关键 targets 列表
- target 类型分类
- 依赖关系图
- 配置参数说明

**适合读者**: 需要理解构建系统的开发者

---

### [07_Build_Artifacts.md](07_Build_Artifacts.md)
**目的**: 说明编译产物及其安装位置

**内容**:
- 产物清单（.so, .abc, 配置文件等）
- 安装路径说明
- 运行时加载关系
- 产物与 target 的映射

**适合读者**: 需要了解部署的开发者

---

### [08_Security_Review.md](08_Security_Review.md)
**目的**: 安全风险分析与建议

**内容**:
- 攻击面清单
- 信任边界分析
- 可被利用点（含证据、触发、影响、修复建议）
- 安全最佳实践

**适合读者**: 安全审计人员、开发负责人

---

### [09_FAQ.md](09_FAQ.md)
**目的**: 常见构建、运行、调试问题

**内容**:
- 构建相关问题
- 运行时问题
- 调试技巧
- 定位路径

**适合读者**: 遇到问题的开发者

---

## 📎 附录

### [appendix/Callgraphs.md](appendix/Callgraphs.md)
**目的**: 关键调用链图

**内容**:
- API 入口到核心逻辑的调用链
- Mermaid 图表示

---

### [appendix/Config_Flags.md](appendix/Config_Flags.md)
**目的**: 关键宏和 feature flags 说明

**内容**:
- 主要配置参数
- 特性开关说明
- 默认值和可选值

---

## 🔍 快速查找

### 按角色查找

| 角色 | 推荐阅读 |
|------|---------|
| **新人** | Overview → Project_Scope → Directory_Structure → Architecture |
| **应用开发者** | Overview → N-API → FAQ |
| **系统开发者** | Overview → Directory_Structure → Architecture → Inner_API |
| **构建工程师** | Overview → GN_Targets → Build_Artifacts |
| **安全审计员** | Overview → Architecture → Security_Review |

### 按问题查找

| 我想了解... | 查看文档 |
|-------------|---------|
| 项目是什么 | Overview |
| 项目边界 | Project_Scope |
| 代码在哪 | Directory_Structure |
| 系统如何工作 | Architecture |
| 如何调用 API | N-API |
| 如何构建 | GN_Targets |
| 产物是什么 | Build_Artifacts |
| 有哪些安全风险 | Security_Review |
| 遇到问题怎么办 | FAQ |
| 调用流程 | appendix/Callgraphs.md |
| 配置选项 | appendix/Config_Flags.md |

---

## 📝 文档更新

最后更新时间: 2026-02-06

如需更新文档，请参考 [README.md](README.md) 中的"如何更新文档"部分。

---

[返回首页](README.md)
