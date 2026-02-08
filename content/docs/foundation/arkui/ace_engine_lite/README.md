# Ace Engine Lite Wiki

> **生成时间**: 2026-02-06
> **项目版本**: 3.1
> **覆盖范围**: arkui_ace_engine_lite 子系统源代码分析

---

## 文档导航

本 Wiki 提供以下文档：

| 文档 | 描述 | 适用读者 |
|------|------|----------|
| [00_Overview.md](00_Overview.md) | 项目概览、核心组件、运行环境 | 所有人 |
| [01_Positioning.md](01_Positioning.md) | 项目定位、边界、核心能力 | 所有人 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | 所有人 |
| [03_Architecture.md](03_Architecture.md) | 架构说明：组件图/数据流/线程模型/关键时序 | 所有人 |
| [04_JS_API.md](04_JS_API.md) | 对外 JS API（JerryScript 绑定）模块列表、API 清单 | 应用开发者 |
| [05_Inner_API.md](05_Inner_API.md) | 内部 API：核心类、接口定义、稳定性评估 | 框架开发者 |
| [06_GN_Targets.md](06_GN_Targets.md) | GN 构建目标：targets 列表、类型、依赖、产物 | 构建工程师 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物：.so/.a 文件、安装路径、运行时加载 | 构建工程师 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审：攻击面、信任边界、可利用点、修复建议 | 安全审计人员 |
| [09_FAQ.md](09_FAQ.md) | 常见问题：构建、运行、调试问题及解决方案 | 所有人 |

---

## 新人阅读路线

### 快速了解（15 分钟）

1. **[00_Overview.md](00_Overview.md)** - 了解项目定位、核心组件、运行环境
2. **[01_Positioning.md](01_Positioning.md)** - 理解项目边界、能力范围、应用场景

### 深入架构（30 分钟）

3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 熟悉代码组织结构
4. **[03_Architecture.md](03_Architecture.md)** - 理解分层架构、数据流、时序

### 开发与集成（45 分钟）

5. **[04_JS_API.md](04_JS_API.md)** - 学习 JS 模块 API（应用开发必读）
6. **[05_Inner_API.md](05_Inner_API.md)** - 了解内部 API（框架开发参考）

### 构建与配置（30 分钟）

7. **[06_GN_Targets.md](06_GN_Targets.md)** - 理解构建系统（编译配置）
8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 了解编译产物和运行时加载

### 安全与问题（20 分钟）

9. **[08_Security_Review.md](08_Security_Review.md)** - 了解安全风险和防护机制
10. **[09_FAQ.md](09_FAQ.md)** - 解决常见开发问题

---

## 覆盖范围

### 已覆盖

- JerryScript JS 引擎绑定机制（JSI 封装层）
- JS 模块注册和加载（ModuleManager）
- 核心框架架构（Components、Router、StyleManager、Context）
- GN 构建系统（6 个主要 targets）
- 编译产物和依赖关系（.so/.a 文件）
- 安全风险分析（10 类攻击面、修复建议）

### 未覆盖

- 测试代码（test/ 目录）
- 第三方依赖详细实现（jerryscript、ui_lite 等）
- 产品适配层代码（platform_adapter）
- 示例代码（examples/ 目录）
- Qt 模拟器实现（simulator 目录）

---

**注意**：所有结论均基于源代码静态分析，实际运行行为可能因配置和平台而异。

## 文档生成信息

**生成方式**：基于源代码自动分析
**证据完整性**：所有关键结论包含文件路径和行号引用
**术语统一**：统一使用 "ace_engine_lite" 或 "Ace Engine Lite" 指代项目

---

## 贡献指南

如需更新或补充文档：

1. **新增功能**：更新对应的定位、架构、API 章节
2. **重构代码**：同步更新目录结构、调用链
3. **发现安全问题**：更新安全评审章节
4. **构建变化**：更新 GN Targets 和编译产物章节
5. **发现问题修复**：更新 FAQ 章节

所有修改请保持：
- **证据引用**：关键结论必须标注 `文件路径:行号`
- **链接有效**：确保 SUMMARY.md 中的链接可访问
- **术语一致**：使用项目统一术语

---

**问题反馈**：如发现文档错误或遗漏，请提交 Issue 或 PR。
