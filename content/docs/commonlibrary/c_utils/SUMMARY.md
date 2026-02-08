# 目录

## 快速导航

- [首页](index.md)
- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [对外 API](04_Public_API.md)
- [内部 API](05_Inner_API.md)
- [GN Targets](06_GN_Targets.md)
- [编译产物](07_Build_Artifacts.md)
- [安全风险评审](08_Security_Review.md)
- [问题排查](09_Troubleshooting.md)

## 安全专题

- [安全风险评审](08_Security_Review.md) - 安全风险总览
- [攻击面分析](05_AttackSurface_Detail.md) - 外部输入与信任边界
- [详细安全评估](Security_Assessment_Detail.md) - 深度风险分析

## 附录

- [关键调用链](appendix/Callgraphs.md)
- [配置与宏](appendix/Config_Flags.md)

## 阅读路线

### 路线一：快速了解（15分钟）
1. [首页](index.md) - 项目简介
2. [项目概览](01_Overview.md) - 核心能力与边界
3. [目录结构](02_Directory_Structure.md) - 代码组织

### 路线二：使用指南（30分钟）
1. [对外 API](04_Public_API.md) - 接口清单
2. 参考具体模块文档（`docs/zh-cn/`）
3. [编译产物](07_Build_Artifacts.md) - 如何依赖

### 路线三：深度理解（60分钟）
1. [架构说明](03_Architecture.md) - 整体设计
2. [内部 API](05_Inner_API.md) - 模块细节
3. [关键调用链](appendix/Callgraphs.md) - 执行流程
4. [安全风险评审](08_Security_Review.md) - 安全考量

### 路线五：安全研究（90分钟）
1. [攻击面分析](05_AttackSurface_Detail.md) - 外部输入与信任边界
2. [详细安全评估](Security_Assessment_Detail.md) - 深度风险分析
3. [代码证据记录](_work/NOTES.md) - 支撑证据

### 路线四：构建集成（20分钟）
1. [GN Targets](06_GN_Targets.md) - 构建配置
2. [编译产物](07_Build_Artifacts.md) - 产物清单
3. [配置与宏](appendix/Config_Flags.md) - 编译选项
