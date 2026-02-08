# WuKong 项目文档

## 项目概述

WuKong 是 OpenHarmony 平台的稳定性测试自动化工具，通过模拟用户行为对系统及应用进行压力测试。

## 文档覆盖范围

| 文档 | 说明 |
|------|------|
| [README](README.md) | 项目文档说明 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读顺序 |
| [00_Overview](00_Overview.md) | 项目定位与核心能力 |
| [01_Architecture](01_Architecture.md) | 架构说明与组件图 |
| [02_CommandLine](02_CommandLine.md) | 命令行接口详解 |
| [03_Module_Details](03_Module_Details.md) | 模块职责与内部 API |
| [04_Build_System](04_Build_System.md) | GN 构建配置与产物 |
| [05_Security_Review](05_Security_Review.md) | 安全风险评审 |
| [06_Troubleshooting](06_Troubleshooting.md) | 常见问题与调试 |

## 附录

- [关键调用链](appendix/Callgraphs.md)

## 更新方式

本 Wiki 基于代码自动生成，最后更新时间：**2026-02-06**

如需更新：
1. 修改代码后运行 Wiki 生成脚本
2. 手动更新对应文档的代码证据引用

## 代码证据标注规范

所有关键结论均标注代码来源：
- 文件路径：`path/to/file:line`
- 符号名：函数/类/宏名
- 代码片段：关键逻辑摘要

## 约束与限制

1. **N-API**: 本项目**不涉及** N-API，是原生可执行程序
2. **测试内容**: 文档不引用测试代码作为业务证据
3. **中文**: 默认中文文档
