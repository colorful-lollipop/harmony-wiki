# 用户证书管理工程 Wiki - 文档导航

> 本文档提供完整的 Wiki 导航和新人阅读顺序

---

## 快速导航

| 文档 | 描述 | 适用人群 |
|-------|-----|---------|
| [README](wiki/README.md) | Wiki 覆盖范围、更新方式 | 所有人 |
| [00_Overview.md](wiki/00_Overview.md) | 项目概览、定位、核心能力 | 新人、产品经理 |
| [01_Position_and_Boundary.md](wiki/01_Position_and_Boundary.md) | 项目边界、运行环境、关键概念 | 架构师、开发者 |
| [02_Directory_Structure.md](wiki/02_Directory_Structure.md) | 目录结构、模块职责 | 开发者 |
| [03_Architecture.md](wiki/03_Architecture.md) | 架构设计、数据流、时序图 | 架构师、开发者 |
| [04_External_API.md](wiki/04_External_API.md) | 对外 API 清单 | 开发者、测试人员 |
| [05_Internal_API.md](wiki/05_Internal_API.md) | 内部 API、模块依赖 | 开发者 |
| [06_GN_Targets.md](wiki/06_GN_Targets.md) | GN Targets、构建配置 | 构建工程师 |
| [07_Build_Artifacts.md](wiki/07_Build_Artifacts.md) | 编译产物、安装路径 | 运维、测试 |
| [08_Security_Review.md](wiki/08_Security_Review.md) | 安全风险评审、修复建议 | 安全工程师 |
| [09_FAQ.md](wiki/09_FAQ.md) | 常见问题、定位路径 | 所有人 |

---

## 新人阅读顺序

### 1. 快速了解（30 分钟）

1. [00_Overview.md](wiki/00_Overview.md) - 了解项目是什么、做什么
2. [01_Position_and_Boundary.md](wiki/01_Position_and_Boundary.md) - 理解项目边界和关键概念
3. [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 熟悉代码组织

### 2. 深入理解（1 小时）

4. [03_Architecture.md](wiki/03_Architecture.md) - 理解架构设计和数据流
5. [04_External_API.md](wiki/04_External_API.md) - 了解与外部服务的交互

### 3. 开发准备（1 小时）

6. [05_Internal_API.md](wiki/05_Internal_API.md) - 理解内部模块接口
7. [06_GN_Targets.md](wiki/06_GN_Targets.md) - 了解构建系统
8. [07_Build_Artifacts.md](wiki/07_Build_Artifacts.md) - 了解编译产物

### 4. 安全审查（30 分钟）

9. [08_Security_Review.md](wiki/08_Security_Review.md) - 了解安全风险和修复建议

### 5. 问题排查（随时）

10. [09_FAQ.md](wiki/09_FAQ.md) - 常见问题和定位方法

---

## 按角色阅读建议

### 产品经理

1. [00_Overview.md](wiki/00_Overview.md) - 了解产品定位
2. [01_Position_and_Boundary.md](wiki/01_Position_and_Boundary.md) - 理解产品边界
3. [03_Architecture.md](wiki/03_Architecture.md) - 了解架构设计

### 架构师

1. [01_Position_and_Boundary.md](wiki/01_Position_and_Boundary.md) - 理解项目定位
2. [03_Architecture.md](wiki/03_Architecture.md) - 深入架构设计
3. [04_External_API.md](wiki/04_External_API.md) - 了解外部依赖
4. [05_Internal_API.md](wiki/05_Internal_API.md) - 理解内部接口
5. [08_Security_Review.md](wiki/08_Security_Review.md) - 安全架构评审

### 开发者

1. [00_Overview.md](wiki/00_Overview.md) - 项目概览
2. [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 代码组织
3. [03_Architecture.md](wiki/03_Architecture.md) - 架构理解
4. [04_External_API.md](wiki/04_External_API.md) - 外部 API
5. [05_Internal_API.md](wiki/05_Internal_API.md) - 内部 API
6. [06_GN_Targets.md](wiki/06_GN_Targets.md) - 构建系统
7. [09_FAQ.md](wiki/09_FAQ.md) - 常见问题

### 安全工程师

1. [00_Overview.md](wiki/00_Overview.md) - 产品理解
2. [03_Architecture.md](wiki/03_Architecture.md) - 架构分析
3. [04_External_API.md](wiki/04_External_API.md) - 外部接口
4. [08_Security_Review.md](wiki/08_Security_Review.md) - 安全风险分析

### 测试人员

1. [00_Overview.md](wiki/00_Overview.md) - 产品理解
2. [04_External_API.md](wiki/04_External_API.md) - 接口清单
3. [07_Build_Artifacts.md](wiki/07_Build_Artifacts.md) - 构建产物
4. [09_FAQ.md](wiki/09_FAQ.md) - 常见问题

---

## 附录目录

### 附录 1: 关键调用链

- [appendix/Callgraphs.md](wiki/appendix/Callgraphs.md) - 关键函数调用链

### 附录 2: 配置标志

- [appendix/Config_Flags.md](wiki/appendix/Config_Flags.md) - 关键宏和 feature flags

---

**最后更新**: 2026-02-06
**文档版本**: 1.0.0
