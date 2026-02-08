# OpenHarmony global_resource_tool Wiki

## 简介

本文档是 OpenHarmony `restool`（资源编译工具）的工程 Wiki，旨在帮助开发者快速理解项目架构、接口设计和安全风险。

**生成时间**: 2026-02-06  
**最后更新**: 2026-02-07

## 覆盖范围

本文档涵盖以下内容：
- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构设计与数据流
- ✅ 对外接口（命令行）
- ✅ 攻击面分析（独立章节）
- ✅ 安全风险评审（深度分析）
- ✅ 内部模块接口
- ✅ GN 构建目标与编译产物
- ✅ 常见问题与调试

## 未覆盖范围

- 测试代码细节（`test/` 目录）
- N-API 接口（本项目为纯命令行工具，无 N-API）
- IPC/SA 接口（本项目无 IPC 相关代码）
- 第三方库内部实现（cJSON、libpng、bounds_checking_function）

## 📚 快速导航

### 推荐阅读路线

| 受众 | 推荐路线 | 时长 |
|------|---------|------|
| **新人学习者** | [SUMMARY.md](SUMMARY.md) → 新人学习路线 | 30 分钟 |
| **安全研究员** | [SUMMARY.md](SUMMARY.md) → 安全研究路线 | 45 分钟 |

### 文档索引

| 类别 | 文档 | 说明 |
|------|------|------|
| **概览** | [README](README.md) | 本文档 |
| | [SUMMARY](SUMMARY.md) | 全站导航 + 双路线推荐 |
| **新人必备** | [00_Overview](00_Overview.md) | 项目定位、能力边界 |
| | [01_Directory_Structure](01_Directory_Structure.md) | 目录结构、代码地图 |
| | [02_Architecture](02_Architecture.md) | 架构图、数据流、线程模型 |
| | [03_Public_API](03_Public_API.md) | 命令行参数、子命令 |
| **安全必备** | [05_AttackSurface](05_AttackSurface.md) | 🔴 攻击面清单、信任边界 |
| | [06_SecurityReview](06_SecurityReview.md) | 🔴 风险详情、修复建议 |
| **工程参考** | [07_GN_Targets](07_GN_Targets.md) | GN targets、编译配置 |
| | [08_Build_Artifacts](08_Build_Artifacts.md) | 编译产物、安装路径 |
| | [04_Internal_API](04_Internal_API.md) | 内部实现、模块接口 |
| **附录** | [08_FAQ](08_FAQ.md) | 常见问题 |
| | [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |
| | [appendix/Config_Flags](appendix/Config_Flags.md) | 关键宏和配置项 |

> **注意**: `05_AttackSurface.md` 和 `06_SecurityReview.md` 是安全研究员**必读**的核心章节。

## 📖 新人快速开始

1. **5 分钟理解项目** → 阅读 [00_Overview.md](00_Overview.md)
2. **10 分钟熟悉代码** → 阅读 [01_Directory_Structure.md](01_Directory_Structure.md)
3. **15 分钟理解架构** → 阅读 [02_Architecture.md](02_Architecture.md)
4. **开始使用** → 阅读 [03_Public_API.md](03_Public_API.md)

## 🔒 安全研究快速入口

1. **了解攻击面** → 阅读 [05_AttackSurface.md](05_AttackSurface.md)（15 分钟）
2. **分析风险详情** → 阅读 [06_SecurityReview.md](06_SecurityReview.md)（20 分钟）
3. **对照架构** → 阅读 [02_Architecture.md](02_Architecture.md)（10 分钟）

## 文档更新方式

本文档基于代码分析生成，建议随代码更新同步维护：

1. 代码结构变更时更新架构文档
2. 新增命令行参数时更新接口文档
3. 发现新的安全风险时更新安全文档
4. **新增攻击面时更新 [05_AttackSurface.md](05_AttackSurface.md)**
5. **新增漏洞时更新 [06_SecurityReview.md](06_SecurityReview.md)**

## 参考资料

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [README_zh.md](../README_zh.md) - 项目中文说明
- [BUILD.gn](../BUILD.gn) - GN 构建配置
- [bundle.json](../bundle.json) - 组件配置
