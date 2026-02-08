# 资源调度服务 Wiki 导航

## 新人阅读路线

### 第一步：了解项目（10 分钟）
1. [README](README.md) - 本文档说明
2. [00_Overview](00_Overview.md) - 项目概览与核心概念
3. [01_Architecture](01_Architecture.md) - 架构设计与数据流

### 第二步：理解代码结构（15 分钟）
4. [02_Directory_Structure](02_Directory_Structure.md) - 目录组织与模块职责

### 第三步：掌握接口（20 分钟）
5. [03_NAPI_Reference](03_NAPI_Reference.md) - JS API 文档
6. [04_Inner_API](04_Inner_API.md) - C++ 内部 API

### 第四步：了解构建（15 分钟）
7. [05_GN_Targets](05_GN_Targets.md) - GN 构建目标
8. [06_Build_Artifacts](06_Build_Artifacts.md) - 编译产物清单

### 第五步：安全与问题排查（按需）
9. [07_Security_Analysis](07_Security_Analysis.md) - 安全风险分析
10. [08_Troubleshooting](08_Troubleshooting.md) - 常见问题与调试

## 快速索引

### 按主题

| 主题 | 相关文档 |
|------|----------|
| **架构设计** | [Overview](00_Overview.md), [Architecture](01_Architecture.md) |
| **N-API 接口** | [NAPI Reference](03_NAPI_Reference.md) |
| **IPC 通信** | [Architecture](01_Architecture.md) > IPC/SA 架构 |
| **插件开发** | [Inner API](04_Inner_API.md) > 插件框架 |
| **构建系统** | [GN Targets](05_GN_Targets.md) |
| **安全问题** | [Security Analysis](07_Security_Analysis.md) |

### 按角色

| 角色 | 推荐阅读 |
|------|----------|
| **应用开发者** | [NAPI Reference](03_NAPI_Reference.md), [Troubleshooting](08_Troubleshooting.md) |
| **系统开发者** | [Architecture](01_Architecture.md), [Inner API](04_Inner_API.md) |
| **安全工程师** | [Security Analysis](07_Security_Analysis.md) |
| **构建工程师** | [GN Targets](05_GN_Targets.md), [Build Artifacts](06_Build_Artifacts.md) |

## 附录

- [调用链](appendix/Callgraphs.md) - 关键调用流程
- [配置标志](appendix/Config_Flags.md) - Feature Flags 与编译选项

---

**导航提示**: 点击文档标题可跳转到对应页面。每个文档末尾都有"相关链接"章节，方便在文档间跳转。
