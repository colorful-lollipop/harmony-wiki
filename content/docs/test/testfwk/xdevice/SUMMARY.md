# XDevice Wiki - 文档导航

本文档提供 XDevice Wiki 的完整导航，帮助新人快速定位所需内容。

---

## 📖 核心文档

### [README.md](README.md)
> 文档概述、覆盖范围、快速导航

- 文档目的与覆盖范围
- 新人阅读顺序推荐
- 按需求查找指南
- 版本信息与更新日志

---

## 🚀 快速入门

### [00_Overview.md](00_Overview.md)
> 项目定位、核心能力、运行环境

- 项目简介与核心定位
- 主要功能特性
- 运行环境要求
- 技术栈概述

### [07_Usage.md](07_Usage.md)
> 安装、配置、执行测试

- 安装步骤
- 配置文件修改
- 命令行使用
- 执行测试用例
- 查看测试结果

---

## 📁 项目结构

### [01_Directory_Structure.md](01_Directory_Structure.md)
> 目录结构、模块职责、文件说明

- 根目录结构
- 核心模块列表（10个模块）
- 插件目录结构（ohos、devicetest）
- 配置文件说明

---

## 🏗️ 架构设计

### [02_Architecture.md](02_Architecture.md)
> 组件图、数据流、线程模型、关键时序

- 系统架构概览
- 核心组件图
- 数据流分析
- 线程/并发模型
- 关键调用时序

### [03_Modules.md](03_Modules.md)
> 模块详细说明、接口定义、实现分析

- command 模块（命令行交互）
- config 模块（配置管理）
- driver 模块（测试驱动）
- environment 模块（设备环境）
- executor 模块（测试执行）
- report 模块（报告生成）
- testkit 模块（测试工具）
- context 模块（上下文管理）
- cluster 模块（分布式测试）
- resource 模块（资源管理）

---

## ⚙️ 构建与配置

### [04_Build.md](04_Build.md)
> GN Targets、编译产物、安装路径

- GN 构建配置
- 主要 Targets 清单
- 编译产物说明
- 安装与部署

### [05_Configuration.md](05_Configuration.md)
> XML/JSON 配置解析、配置项说明

- 配置架构
- user_config.xml 详解
- JSON 配置文件说明
- 配置验证机制

---

## 🔒 安全与合规

### [06_Security.md](06_Security.md)
> 攻击面分析、信任边界、风险评估

- 攻击面清单
- 信任边界
- 安全风险评估（6高危+5中危+2低危）
- 修复建议
- 安全最佳实践

---

## ❓ 故障排查

### [08_Troubleshooting.md](08_Troubleshooting.md)
> 常见问题、解决方案、调试技巧

- 安装问题
- 配置问题
- 执行问题
- 设备连接问题
- 报告生成问题

---

## 📋 附录

### [appendix/Callgraphs.md](appendix/Callgraphs.md)（可选）
> 关键调用链、入口到核心逻辑

### [appendix/Config_Flags.md](appendix/Config_Flags.md)（可选）
> 关键宏、Feature Flags

---

## 新人阅读顺序（推荐）

```
┌─────────────────────────────────────────────────────────┐
│                    新人入门路径                          │
├─────────────────────────────────────────────────────────┤
│  1️⃣  README.md                                        │
│      → 了解文档覆盖范围和导航方式                        │
│                                                         │
│  2️⃣  00_Overview.md                                    │
│      → 理解项目定位和技术栈                              │
│                                                         │
│  3️⃣  01_Directory_Structure.md                         │
│      → 熟悉项目目录结构和模块                           │
│                                                         │
│  4️⃣  02_Architecture.md                                │
│      → 理解系统架构和数据流                             │
│                                                         │
│  5️⃣  05_Configuration.md                               │
│      → 学习如何配置测试环境                             │
│                                                         │
│  6️⃣  07_Usage.md                                      │
│      → 开始执行第一个测试用例                           │
└─────────────────────────────────────────────────────────┘
```

---

## 文档版本

| 项目 | 版本 | 更新日期 |
|------|------|----------|
| XDevice | 5.0.6.100 | 2026-02-06 |
| bundle.json | 2.30.0 | 2026-02-06 |

---

## 相关链接

- [OpenHarmony Testing Subsystem](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E6%B5%8B%E8%AF%95%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [test_developertest 仓库](https://gitee.com/openharmony/test_developertest/blob/master/README_zh.md)
