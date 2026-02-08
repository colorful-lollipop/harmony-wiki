# WuKong Wiki 导航

## 新人学习路线

**目标**: 快速理解项目定位、学会使用、掌握架构

```
推荐阅读顺序 (预计 30 分钟):
1. README.md (本文档说明) - 5 分钟
2. 00_Overview.md (项目概览) - 10 分钟
   ├─ 项目定位与核心能力
   ├─ 运行环境要求
   └─ 快速开始示例
3. 02_CommandLine.md (命令行接口) - 10 分钟
   ├─ 所有命令参数说明
   └─ 实战使用示例
4. 01_Architecture.md (架构理解) - 10 分钟
   ├─ 组件图与数据流
   └─ 线程模型
5. 根据需要查阅:
   - 03_Module_Details.md (模块详情)
   - 04_Build_System.md (构建系统)
   - 06_Troubleshooting.md (常见问题)
```

---

## 安全研究路线

**目标**: 识别攻击面、发现漏洞、评估风险

```
推荐阅读顺序 (预计 45 分钟):
1. README.md (本文档说明) - 5 分钟
2. 00_Overview.md (项目概览) - 10 分钟
   ├─ 理解项目定位
   └─ 识别对外暴露面
3. 05_Security_Review.md (安全风险评估) - 20 分钟
   ├─ 信任边界分析
   ├─ 攻击面清单
   ├─ 7 个已识别风险点
   └─ 利用路径与修复建议
4. 01_Architecture.md (架构理解) - 10 分钟
   ├─ 数据流与信任跨越点
   └─ 系统服务依赖
5. 根据需要查阅:
   - 02_CommandLine.md (命令行输入验证)
   - 03_Module_Details.md (敏感操作定位)
   - 04_Build_System.md (构建配置)
```

---

## 文档清单

### 核心文档

| 章节 | 文档 | 说明 |
|------|------|------|
| - | [README](README.md) | Wiki 说明与更新方式 |
| - | [SUMMARY](SUMMARY.md) | 本导航页面 |
| 00 | [Overview](00_Overview.md) | 项目定位、核心能力、运行环境 |
| 01 | [Architecture](01_Architecture.md) | 组件图、数据流、线程模型 |
| 02 | [CommandLine](02_CommandLine.md) | 命令行选项、使用示例 |
| 03 | [Module_Details](03_Module_Details.md) | 模块职责、依赖方向、内部 API |
| 04 | [Build_System](04_Build_System.md) | GN targets、编译产物 |
| 05 | [Security_Review](05_Security_Review.md) | 攻击面、风险分析 |
| 06 | [Troubleshooting](06_Troubleshooting.md) | 常见问题、调试方法 |

### 附录

| 文档 | 说明 |
|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链图 |

## 快速跳转

### 命令行
- [version 命令](02_CommandLine.md#version-命令)
- [help 命令](02_CommandLine.md#help-命令)
- [appinfo 命令](02_CommandLine.md#appinfo-命令)
- [special 命令](02_CommandLine.md#special-命令)
- [exec 命令](02_CommandLine.md#exec-命令)
- [focus 命令](02_CommandLine.md#focus-命令)

### 模块
- [common 模块](03_Module_Details.md#common-模块)
- [component_event 模块](03_Module_Details.md#component_event-模块)
- [input_factory 模块](03_Module_Details.md#input_factory-模块)
- [report 模块](03_Module_Details.md#report-模块)
- [shell_command 模块](03_Module_Details.md#shell_command-模块)
- [test_flow 模块](03_Module_Details.md#test_flow-模块)

### 安全
- [信任边界](05_Security_Review.md#信任边界)
- [攻击面分析](05_Security_Review.md#攻击面分析)
- [开发者模式检查](05_Security_Review.md#1-开发者模式检查)
- [信号量互斥](05_Security_Review.md#2-信号量互斥)
- [命令行参数注入风险](05_Security_Review.md#3-命令行参数注入风险)
- [输入注入风险](05_Security_Review.md#4-输入注入风险)
- [应用拉起风险](05_Security_Review.md#5-应用拉起风险)
- [报告文件权限](05_Security_Review.md#6-报告文件权限)
- [依赖库安全](05_Security_Review.md#7-依赖库安全)

---
*最后更新: 2026-02-06*
