# 全站导航 - graphic_cangjie_wrapper Wiki

## 新人阅读路线

```
推荐阅读顺序:
1. 01_Overview.md  → 项目定位与核心能力
2. 02_Architecture.md → 架构设计与组件关系
3. 03_N-API.md      → API 接口详解
4. 05_Build.md      → 构建配置与产物
```

## 文档索引

### 基础文档

| 文档 | 描述 | 关键内容 |
|-----|-----|---------|
| [README](README.md) | Wiki 说明 | 更新方式、覆盖范围 |
| [01_Overview](01_Overview.md) | 项目概览 | 定位、边界、能力、约束 |
| [02_Architecture](02_Architecture.md) | 系统架构 | 组件图、数据流、线程模型 |

### API 文档

| 文档 | 描述 | 关键内容 |
|-----|-----|---------|
| [03_N-API](03_N-API.md) | Cangjie API | ColorSpace、ColorSpaceManager |
| [04_Inner_API](04_Inner_API.md) | 内部模块 | 模块接口、依赖方向 |

### 工程文档

| 文档 | 描述 | 关键内容 |
|-----|-----|---------|
| [05_Build](05_Build.md) | 构建配置 | GN Targets、产物清单 |
| [06_Security](06_Security.md) | 安全评审 | 攻击面、风险点、修复建议 |

### 附录

| 文档 | 描述 | 关键内容 |
|-----|-----|---------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 调用链图 | JS → Native 调用路径 |

## 快速跳转

### API 快速查找

- **ColorSpace 枚举**: [03_N-API.md#colorspace-枚举](./03_N-API.md#colorspace-枚举)
- **ColorSpacePrimaries 类**: [03_N-API.md#colorspaceprimaries-类](./03_N-API.md#colorspaceprimaries-类)
- **ColorSpaceManager 类**: [03_N-API.md#colorspacemanager-类](./03_N-API.md#colorspacemanager-类)
- **create() 工厂方法**: [03_N-API.md#create-工厂方法](./03_N-API.md#create-工厂方法)

### 构建快速查找

- **Kit Target**: [05_Build.md#kitarkgraphics2d](./05_Build.md#kitarkgraphics2d)
- **ColorSpaceManager Target**: [05_Build.md#ohosgraphicscolor_space_manager](./05_Build.md#ohosgraphicscolor_space_manager)
- **产物路径**: [05_Build.md#产物清单](./05_Build.md#产物清单)

### 安全快速查找

- **攻击面**: [06_Security.md#攻击面清单](./06_Security.md#攻击面清单)
- **风险点**: [06_Security.md#风险分析与修复建议](./06_Security.md#风险分析与修复建议)
