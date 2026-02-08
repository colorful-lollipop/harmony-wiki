# 目录导航

## 核心文档

| 章节 | 标题 | 说明 |
|------|------|------|
| [README](README.md) | 首页 | 文档覆盖范围与更新方式 |
| [01_Overview](01_Overview.md) | 项目概述 | 模块定位、核心能力、运行环境 |
| [02_Architecture](02_Architecture.md) | 架构设计 | 组件图、数据流、FFI 绑定机制 |
| [03_API_Reference](03_API_Reference.md) | API 参考 | CommonEventManager 及数据类 API |
| [03_CodeMap](03_CodeMap.md) | 代码地图 | 目录结构、核心文件定位 |
| [04_Build](04_Build.md) | 构建配置 | GN targets、编译产物、加载关系 |
| [05_Security](05_Security.md) | 安全评审 | 威胁模型、攻击面、风险分析 |
| [08_Internals](08_Internals.md) | 内部实现 | FFI 机制、内存管理、序列化 |

## 附录

| 章节 | 标题 | 说明 |
|------|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链 | 入口→核心逻辑调用链路 |
| [ErrorCodes](appendix/ErrorCodes.md) | 错误码速查 | 错误码含义与处理建议 |

## 新人阅读路线

### 路线 A：快速上手 (10 分钟)

1. [README](README.md) → 了解文档结构
2. [01_Overview](01_Overview.md) → 理解项目定位
3. [03_API_Reference](03_API_Reference.md) → 掌握核心 API

### 路线 B：深入开发 (30 分钟)

1. 完成路线 A
2. [02_Architecture](02_Architecture.md) → 理解 FFI 绑定机制
3. [04_Build](04_Build.md) → 掌握构建配置
4. [appendix/Callgraphs](appendix/Callgraphs.md) → 查看调用链路

### 路线 C：安全审计 (20 分钟)

1. [05_Security](05_Security.md) → 完整安全评审报告
2. 相关章节作为参考
