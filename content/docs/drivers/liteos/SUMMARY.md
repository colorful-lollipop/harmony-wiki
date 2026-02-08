# 文档导航

本文档提供 `drivers_liteos` Wiki 的完整导航结构。

## 快速入门

| 章节 | 内容 | 适合人群 |
|-----|------|---------|
| [README](README.md) | Wiki 使用说明 | 所有开发者 |
| [首页](index.md) | 项目概览 | 新人 |
| [常见问题](07_FAQ.md) | 快速答疑 | 遇到问题的开发者 |

## 核心文档

### 1. 项目概览
- [首页](index.md) - 项目定位、核心能力、运行环境
- [目录结构](02_Directory_Structure.md) - 源码目录与模块职责

### 2. 架构设计
- [架构说明](03_Architecture.md) - 组件图、数据流、线程模型

### 3. 接口文档
- [hievent API](04_Hievent_API.md) - 驱动接口、事件处理

### 4. 构建配置
- [构建配置](05_Build_Configuration.md) - GN targets、Kconfig

### 5. 安全分析
- [安全分析](06_Security_Analysis.md) - 威胁模型、攻击面、风险清单

### 6. 运维支持
- [常见问题](07_FAQ.md) - 构建、运行、调试问题

## 阅读路线图

### 场景 A：新人入门

```
1. README.md (Wiki 使用说明)
2. index.md (项目概览)
3. 03_Architecture.md (整体架构)
4. 04_Hievent_API.md (接口使用)
```

### 场景 B：API 集成

```
1. 04_Hievent_API.md (接口清单)
2. 03_Architecture.md (调用流程)
3. 05_Build_Configuration.md (依赖配置)
```

### 场景 C：安全审计

```
1. 06_Security_Analysis.md (安全概览)
2. 03_Architecture.md (攻击面)
3. 04_Hievent_API.md (输入验证)
```

### 场景 D：构建开发

```
1. 05_Build_Configuration.md (配置说明)
2. 07_FAQ.md (常见问题)
3. 相关源码 (src/*.c)
```

## 文档变更历史

| 版本 | 日期 | 变更内容 |
|-----|------|---------|
| 1.0 | 2024 | 初始版本 |

## 贡献指南

欢迎完善 Wiki 内容：

1. 发现文档错误？请直接编辑对应 `.md` 文件
2. 新增内容？参考现有文档风格编写
3. 需要帮助？请提交 Issue

---

*本文档自动生成于 Git commit: `HEAD`*
