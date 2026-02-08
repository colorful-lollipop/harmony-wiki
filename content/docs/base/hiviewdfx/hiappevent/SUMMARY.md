# 文档导航

> 本文档提供 HiAppEvent Wiki 全站导航，列出所有文档页面及其阅读顺序建议。

## 推荐阅读顺序

对于初次接触 HiAppEvent 组件的开发者，建议按照以下顺序阅读：

**第一步：项目概览** —— 首先阅读 [01_Overview.md](./01_Overview.md)，了解组件的定位、功能特性、目录结构和核心概念，建立对整个组件的宏观认知。

**第二步：架构设计** —— 在理解项目整体后，阅读 [02_Architecture.md](./02_Architecture.md)，深入理解组件的内部架构设计，包括模块划分、数据流向、线程模型等。

**第三步：API 接口** —— 如需使用或集成 HiAppEvent，阅读 [03_N-API.md](./03_N-API.md)，获取完整的 API 接口说明，包括 JS API 和 Native API 的详细规格。

**第四步：构建配置** —— 如需编译或定制 HiAppEvent，阅读 [04_Build.md](./04_Build.md)，了解 GN 构建系统的配置细节和编译产物说明。

**第五步：安全评估** —— 如需进行安全审计或了解组件的安全设计，阅读 [05_Security.md](./05_Security.md)，查看安全风险评审结果。

**第六步：问题排查** —— 在使用过程中遇到问题时，可查阅 [06_FAQ.md](./06_FAQ.md]，寻找常见问题的解决方案。

## 文档索引

### 核心文档

| 文档名称 | 文件路径 | 说明 |
|---------|---------|------|
| 首页与导航 | `README.md` | 文档概述、覆盖范围、更新方式 |
| 全站导航 | `SUMMARY.md` | 本文档，提供完整索引和阅读建议 |
| 项目概览 | `01_Overview.md` | 组件定位、功能特性、目录结构、关键概念 |
| 架构设计 | `02_Architecture.md` | 组件图、数据流、线程模型、时序图 |
| API 接口 | `03_N-API.md` | JS API、Native API、参数说明、错误码 |
| 构建配置 | `04_Build.md` | GN Targets、编译产物、安装路径 |
| 安全评审 | `05_Security.md` | 攻击面、信任边界、风险点、修复建议 |
| 常见问题 | `06_FAQ.md` | 构建、运行、调试问题及解决方案 |

### 附录文档

| 文档名称 | 文件路径 | 说明 |
|---------|---------|------|
| 调用链图谱 | `appendix/Callgraphs.md` | 关键调用链（入口→核心逻辑） |
| 配置参数 | `appendix/Config_Flags.md` | 关键宏、Feature Flags 说明 |

## 模块索引

以下按模块分类列出相关文档章节，方便按需查找：

### N-API 相关

- JS API 注册与初始化：`03_N-API.md` → 「JS API 注册点」
- JS API 完整清单：`03_N-API.md` → 「JS API 清单表」
- 参数校验机制：`03_N-API.md` → 「参数校验流程」
- 错误码定义：`03_N-API.md` → 「错误码说明」

### 构建相关

- GN 构建入口：`04_Build.md` → 「根构建配置」
- 核心 Targets 列表：`04_Build.md` → 「核心 Targets 清单」
- 产物映射表：`04_Build.md` → 「产物清单与安装路径」

### 安全相关

- 攻击面清单：`05_Security.md` → 「攻击面识别」
- 信任边界：`05_Security.md` → 「信任边界与数据流」
- 安全风险点：`05_Security.md` → 「安全风险分析」

## 版本历史

| 版本 | 日期 | 变更说明 |
|-----|------|---------|
| 1.0 | 2026-02-06 | 初始版本，完成核心文档框架 |

## 相关链接

- **OpenHarmony 官网**：https://www.openharmony.cn/
- **DFX 子系统文档**：https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md
- **代码仓库**：https://gitee.com/openharmony/hiviewdfx_hiappevent
- **Issue 反馈**：请通过代码仓库的 Issue 系统提交问题报告
