# 全站导航

## 快速入口

- [首页](/README.md)
- [项目概览](/01_Overview.md)

## 核心文档

| 章节 | 标题 | 说明 |
|------|------|------|
| 01 | [项目概览](01_Overview.md) | 项目定位、目标、关键特性 |
| 02 | [芯片型号说明](02_Chips.md) | 各 SoC 型号特性对比 |
| 03 | [目录结构](03_Directory_Structure.md) | 代码组织结构 |
| 04 | [HAL 模块说明](04_HAL_Modules.md) | 硬件抽象层模块 |
| 05 | [平台驱动说明](05_Platform_Drivers.md) | 外设驱动详解 |
| 06 | [SDK 架构说明](06_SDK_Architecture.md) | SDK 设计理念 |
| 07 | [构建系统](07_Build_System.md) | GN 构建配置 |
| 08 | [编译产物说明](08_Products.md) | 产物清单与加载 |
| 09 | [安全风险评审](09_Security_Review.md) | 安全分析与建议 |

## 附录

| 文档 | 说明 |
|------|------|
| [关键调用链](appendix/Callgraphs.md) | 入口到核心逻辑的调用路径 |
| [配置选项](appendix/Config_Flags.md) | 关键宏与 Feature Flags |

## 阅读路线图

### 新人入门
```
README.md → 01_Overview.md → 02_Chips.md → 03_Directory_Structure.md
```

### HAL 开发
```
01_Overview.md → 04_HAL_Modules.md → 05_Platform_Drivers.md
```

### 构建与编译
```
07_Build_System.md → 08_Products.md
```

### 安全相关
```
01_Overview.md → 09_Security_Review.md → 附录/Config_Flags.md
```
