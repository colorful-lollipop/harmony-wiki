# 导航

本文档是 applications_launcher 项目的全站导航，帮助新人快速定位所需信息。

## 新人阅读路线

### 第一天：快速上手

```
1. 阅读 [概览](00_Overview.md)           → 了解项目定位与技术栈
2. 阅读 [目录结构](01_Directory_Structure.md) → 熟悉代码组织方式
3. 阅读 [构建配置](05_Build.md)          → 了解如何编译运行
```

### 第二周：深入开发

```
4. 阅读 [架构说明](02_Architecture.md)    → 理解组件关系
5. 阅读 [对外 API](03_APIs.md)           → 了解模块接口
6. 阅读 [内部 API](04_Inner_API.md)      → 了解内部模块
7. 阅读 [调用链图谱](appendix/Callgraphs.md) → 追踪关键流程
```

## 完整目录

### 核心文档

| 章节 | 文档 | 说明 |
|------|------|------|
| 📖 | [README](README.md) | 本 Wiki 说明 |
| 🗺️ | [SUMMARY](SUMMARY.md) | 本导航文档 |
| 🏠 | [概览](00_Overview.md) | 项目定位与技术栈 |
| 📁 | [目录结构](01_Directory_Structure.md) | 模块划分与职责 |
| 🏗️ | [架构说明](02_Architecture.md) | 组件图与数据流 |

### 接口文档

| 章节 | 文档 | 说明 |
|------|------|------|
| 🔌 | [对外 API](03_APIs.md) | ArkTS 模块导出 |
| 🔧 | [内部 API](04_Inner_API.md) | 内部模块接口 |

### 工程文档

| 章节 | 文档 | 说明 |
|------|------|------|
| 🔨 | [构建配置](05_Build.md) | hvigor Targets |
| 🛡️ | [安全评审](06_Security.md) | 风险与修复 |

### 附录

| 章节 | 文档 | 说明 |
|------|------|------|
| 📊 | [调用链图谱](appendix/Callgraphs.md) | 关键调用链 |

## 快速索引

### 按功能查找

| 功能 | 文档位置 |
|------|---------|
| 如何编译 | [构建配置](05_Build.md) |
| 如何运行 | [概览 → 运行环境](00_Overview.md) |
| 模块依赖 | [内部 API → 模块依赖](04_Inner_API.md) |
| 常见问题 | [安全评审 → 风险修复](06_Security.md) |

### 按文件类型查找

| 类型 | 位置 |
|------|------|
| ArkTS 组件 | `feature/*/index.ts` |
| 常量定义 | `common/src/main/ets/default/constants/` |
| 视图模型 | `*/viewmodel/*.ts` |
| 配置文件 | `build-profile.json5` |
