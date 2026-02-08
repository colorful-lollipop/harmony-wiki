# hievent_lite Wiki 导航

> 全站内容索引与阅读路线图

## 📚 文档目录

### 入门指南

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [README](README.md) | Wiki 使用说明 | 所有读者 |
| [00_Overview](00_Overview.md) | 项目定位与核心能力 | 新人 |
| [05_Troubleshooting](05_Troubleshooting.md) | 常见问题速查 | 开发者 |

### 核心开发

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [01_Architecture](01_Architecture.md) | 架构设计、组件图、数据流 | 架构师 |
| [02_API_Reference](02_API_Reference.md) | C API 接口清单与用法 | 开发者 |
| [03_Build_System](03_Build_System.md) | GN 构建配置与 targets | 开发者 |

### 安全与质量

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [04_Security_Review](04_Security_Review.md) | 安全风险分析与修复建议 | 安全工程师 |

## 🔗 快速跳转

### API 快速查找

- [事件创建](02_API_Reference.md#21-事件创建函数)
- [事件上报](02_API_Reference.md#22-事件上报函数)
- [文件操作](02_API_Reference.md#24-文件操作函数)
- [宏定义接口](02_API_Reference.md#3-宏定义接口)

### 构建相关

- [Targets 清单](03_Build_System.md#2-targets-清单)
- [依赖关系](03_Build_System.md#3-依赖关系)
- [编译产物](03_Build_System.md#4-编译产物)

### 安全相关

- [攻击面清单](04_Security_Review.md#2-攻击面清单)
- [风险项列表](04_Security_Review.md#3-安全风险项)
- [修复建议](04_Security_Review.md#4-修复建议)

## 📖 新人阅读路线

```
第1步: 项目概览
    ↓
    ├─ 00_Overview.md
    └─ 理解项目定位、事件类型、核心能力
           ↓
第2步: 架构理解
    ↓
    ├─ 01_Architecture.md
    └─ 掌握数据流、模块职责、生命周期
           ↓
第3步: API 学习
    ↓
    ├─ 02_API_Reference.md
    └─ 熟悉接口使用、参数规范
           ↓
第4步: 实践入门
    ↓
    ├─ 05_Troubleshooting.md
    └─ 常见问题与解决方案
```

## 🔧 开发者快速参考

### 我需要...

| 需求 | 跳转 |
|------|------|
| 了解事件如何上报 | [数据流](01_Architecture.md#4-核心数据流) |
| 查找 API 接口 | [API 清单](02_API_Reference.md) |
| 配置构建参数 | [BUILD.gn](03_Build_System.md) |
| 理解文件存储格式 | [文件管理模块](01_Architecture.md#22-文件管理模块) |
| 安全编码注意事项 | [安全规范](04_Security_Review.md) |

### 代码证据索引

| 证据类型 | 位置 |
|----------|------|
| 事件类型定义 | `interfaces/native/innerkits/hiview_event.h:27-38` |
| 核心实现 | `frameworks/hiview_event.c` |
| 输出实现 | `frameworks/hiview_output_event.c` |
| 命令处理 | `command/hievent_lite_command.c` |
| 构建配置 | `BUILD.gn` |

## 📝 文档版本

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |

---

*导航更新时间: 2026-02-06*
