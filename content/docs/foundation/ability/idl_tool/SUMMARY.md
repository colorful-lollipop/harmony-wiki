# OpenHarmony IDL Tool - 文档导航

本文档提供 OpenHarmony IDL Tool 项目的导航索引，帮助新人按合理顺序阅读和理解项目。

---

## 推荐阅读顺序

### 第一步：快速了解（5 分钟）

1. **[00_Overview.md](00_Overview.md)** - 项目概览
   - 项目定位与核心能力
   - HDI vs SA 接口类型
   - 运行环境与依赖
   - 关键概念说明

2. **[README.md](README.md)** - 文档使用指南
   - 覆盖范围说明
   - 文档更新方式
   - 维护指南

---

### 第二步：架构理解（15 分钟）

3. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构
   - 模块职责划分
   - 关键文件清单
   - 模块间依赖关系

4. **[02_Architecture.md](02_Architecture.md)** - 架构设计
   - AST 模块结构
   - Parser/Lexer 流程
   - Codegen 架构
   - Metadata 系统
   - 关键时序图（Mermaid）

---

### 第三步：API 与构建（20 分钟）

5. **[03_External_API.md](03_External_API.md)** - 对外 API
   - **重要说明**：本项目不提供 N-API（JS API）
   - 命令行接口
   - 输入/输出格式

6. **[04_Internal_API.md](04_Internal_API.md)** - 内部 API
   - AST 节点接口
   - Parser/Lexer 接口
   - Codegen Emitter 接口
   - Metadata 接口
   - 稳定性与可替换点

7. **[05_GN_Targets.md](05_GN_Targets.md)** - GN 构建系统
   - 关键 targets 列表
   - 类型与依赖关系
   - 构建产物映射

8. **[06_Build_Artifacts.md](06_Build_Artifacts.md)** - 编译产物
   - idl 可执行文件
   - 安装路径
   - 运行时加载关系

---

### 第四步：安全与问题（10 分钟）

9. **[07_Security_Review.md](07_Security_Review.md)** - 安全风险评审
   - 攻击面分析
   - 信任边界划分
   - 可被利用点清单
   - 修复建议

10. **[08_QA.md](08_QA.md)** - 常见问题
   - 构建问题定位
   - 运行时调试
   - 错误码处理

---

## 按主题导航

### 代码生成相关
- [AST 模块](02_Architecture.md#ast-模块详解) - 37 种类型节点
- [Parser 模块](02_Architecture.md#parser-模块详解) - 递归下降解析器
- [Codegen 模块](02_Architecture.md#codegen-模块详解) - HDI/SA 多后端
- [Metadata 模块](02_Architecture.md#metadata-模块详解) - 运行时类型信息

### 后端语言
- [HDI C](02_Architecture.md#hdl-c) - C 语言代码生成
- [HDI C++](02_Architecture.md#hdl-c) - C++ 语言代码生成
- [HDI Java](02_Architecture.md#hdl-c) - Java 语言代码生成
- [SA C++](02_Architecture.md#hdl-c) - SA C++ 代码生成
- [SA TS](02_Architecture.md#hdl-c) - SA TypeScript 代码生成
- [SA Rust](02_Architecture.md#hdl-c) - SA Rust 代码生成

### 构建相关
- [GN Targets](05_GN_Targets.md) - 构建目标定义
- [编译产物](06_Build_Artifacts.md) - 输出文件与路径

---

## 快速参考

| 想了解... | 查看文档 |
|----------|---------|
| 项目是什么？ | [00_Overview.md](00_Overview.md) |
| 如何编译？ | [05_GN_Targets.md](05_GN_Targets.md) |
| 代码如何生成？ | [02_Architecture.md](02_Architecture.md) |
| 有什么后端？ | [00_Overview.md](00_Overview.md#支持的代码生成后端) |
| 安全风险？ | [07_Security_Review.md](07_Security_Review.md) |
| 遇到问题？ | [08_QA.md](08_QA.md) |

---

## 文档版本信息

**文档生成时间**: 2026-02-06
**覆盖的代码版本**: ability_idl_tool@3.1
**OpenHarmony 版本**: 不特定（支持多版本）
