# Wiki 导航

## 快速导航

- [README](README.md) - 文档说明和阅读指南
- [00. 项目概览](00_Overview.md) - 分布式音频简介
- [01. 项目定位与边界](01_Project_Boundary.md) - 核心能力和运行环境
- [02. 目录结构](02_Directory_Structure.md) - 代码组织方式
- [03. 架构说明](03_Architecture.md) - 系统架构详解
- [04. 对外 API](04_Public_API.md) - IPC 接口定义
- [05. 内部 API](05_Inner_API.md) - 模块间接口
- [06. GN Targets](06_GN_Targets.md) - 构建系统
- [07. 编译产物](07_Build_Artifacts.md) - 输出文件说明
- [08. 安全风险评审](08_Security_Review.md) - 安全分析
- [09. 常见问题](09_Troubleshooting.md) - 调试和故障排除

## 附录

- [A. 调用链](appendix/Callgraphs.md) - 关键调用链
- [B. 配置标志](appendix/Config_Flags.md) - 编译配置选项

## 新人阅读顺序

### 第 1 步：了解项目（10 分钟）

1. [README](README.md) - 了解文档结构
2. [00. 项目概览](00_Overview.md) - 了解基本概念
3. [01. 项目定位与边界](01_Project_Boundary.md) - 了解项目边界

### 第 2 步：理解架构（20 分钟）

4. [02. 目录结构](02_Directory_Structure.md) - 代码在哪里
5. [03. 架构说明](03_Architecture.md) - 系统如何工作

### 第 3 步：接口和构建（20 分钟）

6. [04. 对外 API](04_Public_API.md) - 对外暴露什么
7. [06. GN Targets](06_GN_Targets.md) - 如何编译
8. [07. 编译产物](07_Build_Artifacts.md) - 输出什么

### 第 4 步：深入开发（按需）

9. [05. 内部 API](05_Inner_API.md) - 模块间如何交互
10. [08. 安全风险评审](08_Security_Review.md) - 安全注意事项
11. [09. 常见问题](09_Troubleshooting.md) - 调试技巧

## 按角色导航

### 架构师

- [00. 项目概览](00_Overview.md)
- [01. 项目定位与边界](01_Project_Boundary.md)
- [03. 架构说明](03_Architecture.md)
- [A. 调用链](appendix/Callgraphs.md)

### 开发工程师

- [02. 目录结构](02_Directory_Structure.md)
- [04. 对外 API](04_Public_API.md)
- [05. 内部 API](05_Inner_API.md)
- [06. GN Targets](06_GN_Targets.md)
- [09. 常见问题](09_Troubleshooting.md)

### 安全研究员快速路线

**Step 1: 攻击面识别（5分钟）**
1. [08. 安全风险评审 - 攻击面分析](08_Security_Review.md#攻击面分析) - 了解所有攻击入口
2. [04. 对外 API](04_Public_API.md) - IPC 接口清单（SA 4805/4806）

**Step 2: 关键风险点（10分钟）**
3. [08. 安全风险评审 - 可利用点分析](08_Security_Review.md#可利用点分析) - 深度分析9个风险点
   - **高危**: DAudioNotifyInner 无权限检查 (#8)
   - **高危**: UpdateWorkMode 权限逻辑错误 (#9)
4. [08. 安全风险评审 - 信任边界](08_Security_Review.md#信任边界) - 安全域跨越点

**Step 3: 验证与测试（按需）**
5. [03. 架构说明](03_Architecture.md) - 理解数据流和组件关系
6. 附录 [A. 调用链](appendix/Callgraphs.md) - 关键函数调用路径

### 安全工程师（详细审计）

- [08. 安全风险评审](08_Security_Review.md)（完整阅读）
- [03. 架构说明](03_Architecture.md)（信任边界部分）
- [04. 对外 API](04_Public_API.md)（IPC 接口部分）
- [05. 内部 API](05_Inner_API.md)（模块间接口）

### 构建工程师

- [06. GN Targets](06_GN_Targets.md)
- [07. 编译产物](07_Build_Artifacts.md)
- [B. 配置标志](appendix/Config_Flags.md)

---

*最后更新: 2025-02-06*
