# SOC 统一调频部件 Wiki

本文档为 OpenHarmony SOC 统一调频部件（soc_perf）的工程 Wiki，旨在帮助新人快速理解项目架构，也为安全研究员提供攻击面分析参考。

## 文档范围

| 分类 | 状态 | 说明 |
|------|------|------|
| **项目概览** | ✅ | 定位、能力边界、运行环境 |
| **架构设计** | ✅ | 组件图、数据流、线程模型 |
| **代码地图** | ✅ | 目录结构、核心文件定位 |
| **接口文档** | ✅ | IPC 接口、N-API 清单 |
| **攻击面分析** | ✅ | 外部输入、敏感操作、信任边界 |
| **安全风险** | ✅ | 常见风险、代码证据、修复建议 |
| **构建说明** | ✅ | GN Targets、编译产物 |
| **内部实现** | ✅ | 核心类、资源生命周期 |

## 快速导航

### 新人学习路线

建议按以下顺序阅读，逐步建立全局认知：

1. **[01_Overview](01_Overview.md)** → 项目定位与核心能力
2. **[02_Architecture](02_Architecture.md)** → 组件交互与数据流
3. **[03_CodeMap](03_CodeMap.md)** → 代码文件定位
4. **[04_Interface](04_Interface.md)** → API 使用方式
5. **[07_Build](07_Build.md)** → 构建与编译

### 安全研究路线

针对安全审计场景，建议按以下优先级查阅：

1. **[05_AttackSurface](05_AttackSurface.md)** → 外部输入入口清单
2. **[06_SecurityReview](06_SecurityReview.md)** → 风险点与利用路径
3. **[02_Architecture](02_Architecture.md)** → 信任边界分析

## 文档维护

### 更新方式

本文档基于代码静态分析生成。如需更新：

1. 修改源码后检查对应文档章节
2. 运行 `./build.sh --product-name {product} --build-target soc_perf` 验证编译
3. 更新 `SUMMARY.md` 导航链接
4. 在 `_work/NOTES.md` 中记录新增证据

### 质量标准

所有文档需满足：

- 每个技术结论有代码证据支撑（文件路径 + 行号）
- 架构图、流程图使用 Mermaid 语法
- 代码片段有语法高亮标注
- 术语统一，内部链接有效

## 相关资源

| 资源 | 链接 |
|------|------|
| 项目 README | [README_ZH](../../README_ZH.md) |
| 组件配置 | [bundle.json](../../bundle.json) |
| OpenHarmony 资源调度服务 | [resource_schedule_service](https://gitee.com/openharmony/resourceschedule_resource_schedule_service) |
| 系统能力框架 | [safwk](https://gitee.com/openharmony/security_safwk) |

## 变更历史

| 日期 | 版本 | 变更内容 |
|------|------|----------|
| 2026-02-06 | v1.0 | 初始版本，生成全部核心文档 |

---

*最后更新：2026-02-07*
*文档版本：v1.0*
