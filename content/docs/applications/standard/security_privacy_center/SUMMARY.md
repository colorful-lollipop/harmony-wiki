# 文档导航

本文档是 OpenHarmony 安全隐私中心模块的完整 Wiki 索引。

## 快速开始

如果你是第一次接触本项目，建议按照以下顺序阅读：

1. [01_Overview.md](./01_Overview.md) → 项目概览
2. [02_Directory_Structure.md](./02_Directory_Structure.md) → 目录结构
3. [03_Architecture.md](./03_Architecture.md) → 架构设计

## 核心文档

| 章节 | 标题 | 说明 | 优先级 |
|-----|------|------|--------|
| 01 | [项目概览](./01_Overview.md) | 项目定位、核心能力、运行环境 | ⭐⭐⭐ |
| 02 | [目录结构](./02_Directory_Structure.md) | 模块划分、文件组织 | ⭐⭐⭐ |
| 03 | [架构设计](./03_Architecture.md) | MVI 架构、组件关系、数据流 | ⭐⭐⭐ |
| 04 | [N-API 接口](./04_NAPI.md) | Native API 接口（本项目不涉及） | ⭐ |
| 05 | [内部 API](./05_Inner_API.md) | 模块接口、依赖方向 | ⭐⭐ |
| 06 | [构建配置](./06_Build_Config.md) | hvigor 构建、编译参数 | ⭐⭐ |
| 07 | [编译产物](./07_Build_Outputs.md) | .hap 文件、安装路径 | ⭐⭐ |
| 08 | [安全评审](./08_Security_Review.md) | 威胁模型、风险分析 | ⭐⭐⭐ |
| 09 | [常见问题](./09_FAQ.md) | 构建、运行、调试问题 | ⭐⭐ |

## 附录

| 文档 | 说明 |
|-----|------|
| [关键调用链](./appendix/Callgraphs.md) | 入口→核心逻辑调用链 |
| [配置开关](./appendix/Config_Flags.md) | 关键宏与 feature flags |

## 文档更新日志

| 日期 | 版本 | 更新内容 |
|-----|------|---------|
| 2026-02-06 | 1.0.0 | 初始版本，完成核心文档框架 |

## 阅读路线图

### 业务开发者

```
01_Overview.md → 02_Directory_Structure.md → 03_Architecture.md → 05_Inner_API.md
```

### 安全审计人员

```
01_Overview.md → 08_Security_Review.md → 05_Inner_API.md
```

### 构建工程师

```
01_Overview.md → 06_Build_Config.md → 07_Build_Outputs.md
```

## 返回首页

- [README.md](./README.md) → 文档说明与更新规则
