# 文档导航

本文档是 OpenHarmony appspawn 应用孵化器的完整技术文档。

## 阅读路线

### 新人入门路线（推荐）

```
1. 先阅读: 00_Overview.md
   → 了解项目定位、核心能力、技术栈

2. 然后阅读: 01_Architecture.md  
   → 理解整体架构、组件关系、数据流

3. 根据需要阅读:
   → 开发者: 02_API.md (接口使用)
   → 构建工程师: 03_Build.md (编译配置)
   → 安全工程师: 04_Security.md (安全评审)
   → 运维/QA: 05_Troubleshooting.md (问题定位)
```

## 文档清单

### 核心文档

| 文档 | 说明 | 目标读者 |
|------|------|---------|
| [README](README.md) | Wiki说明、更新方式 | 所有读者 |
| [00_Overview](00_Overview.md) | 项目概览、定位、能力 | 所有读者 |
| [01_Architecture](01_Architecture.md) | 架构图、数据流、模块职责 | 开发者、架构师 |
| [02_API](02_API.md) | Inner API接口详解 | 开发者 |
| [03_Build](03_Build.md) | GN构建、Targets、产物 | 构建工程师 |
| [04_Security](04_Security.md) | 安全风险评审 | 安全工程师 |
| [05_Troubleshooting](05_Troubleshooting.md) | 常见问题与调试 | 运维、QA |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图示 |

## 快速跳转

### 按任务类型

**我想了解...**
- 项目是什么 → [00_Overview](00_Overview.md)
- 代码怎么运行 → [01_Architecture](01_Architecture.md)
- 如何调用接口 → [02_API](02_API.md)
- 如何编译项目 → [03_Build](03_Build.md)
- 有什么安全风险 → [04_Security](04_Security.md)
- 遇到问题怎么办 → [05_Troubleshooting](05_Troubleshooting.md)

### 按技术领域

**查找特定技术...**
- IPC机制 → 01_Architecture.md (Socket通信)
- 权限管理 → 01_Architecture.md (Security) + 04_Security.md
- 沙箱隔离 → 01_Architecture.md (Sandbox)
- 构建配置 → 03_Build.md (GN Targets)
- 消息格式 → 02_API.md (TLV格式)
