# 文档导航

本文档为方舟工具链组件的 Wiki 导航页，提供了所有文档的链接和推荐阅读顺序。

## 推荐阅读顺序

对于新加入项目的开发者，建议按以下顺序阅读文档以快速建立整体认知：

首先阅读 [00_Overview.md](./00_Overview.md) 了解项目定位和核心能力，然后阅读 [01_Directory_Structure.md](./01_Directory_Structure.md) 熟悉目录结构和模块职责，接着阅读 [02_Architecture.md](./02_Architecture.md) 理解系统架构和数据流，最后根据实际需要查阅 API 文档、构建指南或安全分析文档。

## 文档清单

### 概览与入门

| 文档 | 描述 |
|------|------|
| [README.md](./README.md) | Wiki 使用说明、覆盖范围、更新方式 |
| [SUMMARY.md](./SUMMARY.md) | 全站导航（本文档） |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责（不含测试） |

### 架构与实现

| 文档 | 描述 |
|------|------|
| [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型、关键时序（Mermaid） |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | N-API 接口清单、参数、返回值、绑定位置 |
| [04_Internal_API.md](./04_Internal_API.md) | 内部模块接口、依赖方向、稳定性标注 |

### 构建与部署

| 文档 | 描述 |
|------|------|
| [05_GN_Build.md](./05_GN_Build.md) | GN Targets 梳理、依赖关系、编译产物映射 |
| [06_Build_Artifacts.md](./06_Build_Artifacts.md) | .so/.a/.hap/可执行文件清单、安装路径、加载关系 |

### 安全与分析

| 文档 | 描述 |
|------|------|
| [07_Security_Review.md](./07_Security_Review.md) | 攻击面分析、信任边界、风险点与修复建议 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见构建/运行/调试问题与定位路径 |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏与 Feature Flags |
