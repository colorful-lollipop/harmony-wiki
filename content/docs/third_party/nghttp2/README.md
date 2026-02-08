# nghttp2 Wiki

## 简介

本文档是 OpenHarmony 第三方库 **nghttp2** 的专项 Wiki，重点记录 nghttp2 在 OpenHarmony 中的集成方式、Patch 分析和特殊适配。

nghttp2 是 HTTP/2 协议及其头部压缩算法 HPACK 的 C 语言实现，在 OpenHarmony 中主要为 curl 提供 HTTP/2 协议支持。

## 文档导航

### 快速入门

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 库概览、OH 定位、版本信息 |
| [02_Patches.md](./02_Patches.md) | **核心文档** - Patch 详细分析 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建系统适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异（如有） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

### 工作文档

- [`_work/ASSESSMENT.md`](./_work/ASSESSMENT.md) - 项目评估报告（信息收集阶段输出）
- [`_work/NOTES.md`](./_work/NOTES.md) - 分析过程记录
- [`_work/PLAN.md`](./_work/PLAN.md) - 任务进度

## 关键信息速览

| 属性 | 值 |
|------|-----|
| **版本** | v1.66.0 |
| **许可证** | MIT |
| **上游** | https://nghttp2.org |
| **Patch 数量** | 1 个（仅示例代码） |
| **主要依赖者** | curl |
| **OH 特有代码** | 无（仅构建配置） |

## 重要说明

- 本文档 **不重复** 上游文档内容，仅关注 OH 相关差异
- 所有结论基于直接证据（代码、Patch、BUILD.gn）
- 不确定之处标注为 `TODO(需确认)`

---

*最后更新: 2025年2月*
