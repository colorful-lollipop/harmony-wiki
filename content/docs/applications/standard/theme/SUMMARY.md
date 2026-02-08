# Wiki 导航目录

本文档提供 applications_theme Wiki 的完整导航，按新人学习路径组织。

## 快速开始

推荐阅读顺序：

```
1. README.md (本文档说明)
2. index.md (首页概览)
3. 01_Project_Overview.md (项目定位)
4. 02_Directory_Structure.md (目录结构)
5. 03_Architecture.md (架构理解)
6. 04_API.md (接口文档)
7. 06_Build.md (构建配置)
8. 08_Security.md (安全考量)
```

## 完整文档列表

### 入门指南

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README.md](README.md) | Wiki 使用说明、更新方式、覆盖范围 | 必读 |
| [index.md](index.md) | 首页概览，快速了解项目 | 必读 |

### 项目理解

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [01_Project_Overview.md](01_Project_Overview.md) | 项目定位、核心能力、运行环境、关键概念 | 必读 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构、模块职责、文件组织 | 必读 |

### 深入理解

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、线程模型、关键时序 | 推荐 |
| [04_API.md](04_API.md) | 对外 API（N-API）清单与说明 | 推荐 |
| [05_Inner_API.md](05_Inner_API.md) | 内部 API、模块接口、依赖方向 | 推荐 |

### 工程实践

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [06_Build.md](06_Build.md) | GN Targets、构建配置、编译选项 | 推荐 |
| [07_Artifacts.md](07_Artifacts.md) | 编译产物、运行时加载、部署路径 | 推荐 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题、调试技巧、定位路径 | 推荐 |

### 安全与审计

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [08_Security.md](08_Security.md) | 安全风险评审、攻击面分析、修复建议 | 推荐 |

## 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图示（待补充） |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键配置参数（待补充） |

## 文档贡献

如需修改或补充 Wiki 内容，请：

1. 遵循本文档的文档规范
2. 引用代码证据（文件路径 + 符号名）
3. 保持术语一致
4. 更新 SUMMARY.md 导航（如有新增）
