# FFRT Wiki 文档导航

> OpenHarmony FFRT (Function Flow Runtime) 项目Wiki
> 版本：4.0 | 更新时间：2026-02-07

---

## 文档简介

本文档为 OpenHarmony **FFRT**（Function Flow Runtime）项目的工程Wiki，面向：

- **新人学习者**：快速理解项目架构、API使用
- **安全研究员**：识别攻击面、评估安全风险
- **开发者**：深入了解内部实现和构建系统

---

## 推荐阅读路线

### 🔰 新人学习路线（建议阅读顺序）

1. **[项目概览](01_Overview.md)** - 了解FFRT是什么、解决什么问题
2. **[目录结构与代码地图](03_CodeMap.md)** - 快速定位代码位置
3. **[对外接口文档](04_Interface.md)** - 学习C/C++ API使用（原03_API_Reference.md）
4. **[架构与数据流](02_Architecture.md)** - 理解内部工作机制
5. **[构建与产物](04_Build.md)** - 了解如何编译

### 🔐 安全研究路线

1. **[攻击面分析](05_AttackSurface.md)** - 识别所有外部输入入口
2. **[安全风险评估](06_SecurityReview.md)** - 深度安全分析
3. **[架构与数据流](02_Architecture.md)** - 理解信任边界
4. **[内部实现细节](08_Internals.md)** - 检查敏感操作

### 🔧 开发者深入路线

1. **[项目概览](01_Overview.md)** - 建立基础认知
2. **[架构与数据流](02_Architecture.md)** - 理解设计思想
3. **[内部实现细节](08_Internals.md)** - 深入源码细节
4. **[构建与产物](04_Build.md)** - 掌握构建系统

---

## 文档目录

### 核心内容

| 文档 | 说明 | 受众 |
|------|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览、核心概念、编程模型 | 所有读者 |
| [02_Architecture.md](02_Architecture.md) | 组件架构、数据流、线程模型、时序 | 开发者、安全研究员 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构、核心文件定位、代码导航 | 新人、开发者 |
| [04_Interface.md](04_Interface.md) | C/C++ API完整清单、调用示例 | 开发者 |
| [05_AttackSurface.md](05_AttackSurface.md) | 外部输入、敏感操作、信任边界 | 安全研究员 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估、漏洞分析、修复建议 | 安全研究员 |
| [04_Build.md](04_Build.md) | GN构建、编译产物、配置选项 | 开发者 |
| [08_Internals.md](08_Internals.md) | 核心类、资源生命周期、内部API | 开发者 |

## 新人阅读顺序

### 路径 1: 快速上手

```
1. 阅读 [概览](01_Overview.md)    --> 理解 FFRT 是什么
2. 阅读 [API 参考](03_API_Reference.md)  --> 了解如何调用
3. 查看 examples/ 目录         --> 学习使用示例
```

### 路径 2: 深入理解

```
1. 阅读 [概览](01_Overview.md)    --> 理解 FFRT 定位
2. 阅读 [架构设计](02_Architecture.md)  --> 理解内部机制
3. 阅读 [编译构建](04_Build.md)   --> 熟悉构建流程
4. 阅读 [API 参考](03_API_Reference.md)  --> 掌握接口使用
```

### 路径 3: 安全审计

```
1. 阅读 [概览](01_Overview.md)    --> 理解基本概念
2. 阅读 [安全风险](05_Security.md)  --> 了解攻击面
3. 追溯代码证据             --> 验证风险点
```

## 模块速查

### 按功能分类

| 功能 | 相关文档 | 关键文件 |
|------|----------|----------|
| 任务提交 | [API 参考](03_API_Reference.md) | `interfaces/kits/c/task.h` |
| 同步原语 | [API 参考](03_API_Reference.md) | `interfaces/kits/c/mutex.h`, `condition_variable.h` |
| 队列管理 | [API 参考](03_API_Reference.md) | `interfaces/kits/c/queue.h` |
| 定时器 | [API 参考](03_API_Reference.md) | `interfaces/kits/c/timer.h` |
| 调度器 | [架构设计](02_Architecture.md) | `src/sched/` |
| 协程 | [架构设计](02_Architecture.md) | `src/eu/co_routine.cpp` |
| 构建配置 | [编译构建](04_Build.md) | `BUILD.gn`, `ffrt.gni` |

## 外部资源

| 资源 | 说明 |
|------|------|
| [主仓库](https://gitee.com/openharmony/foundation_resourceschedule_ffrt) | FFRT 官方代码仓库 |
| [用户指南](docs/README.md) | 官方用户指南索引 |
| [API 指南 - C](docs/ffrt-api-guideline-c.md) | C API 详细使用说明 |
| [API 指南 - C++](docs/ffrt-api-guideline-cpp.md) | C++ API 详细使用说明 |
| [OpenHarmony 文档](https://gitee.com/openharmony/docs) | OpenHarmony 官方文档 |

## 版本兼容性

| Wiki 版本 | FFRT 版本 | 更新日期 |
|-----------|-----------|----------|
| 1.0 | 4.0 | 2025-02-06 |
