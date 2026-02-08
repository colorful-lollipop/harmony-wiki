# Data Classification Wiki - 数据传输管控模块文档

## 文档覆盖范围

本文档描述 OpenHarmony 安全子系统下的 **dataclassification** 模块，涵盖：

- 项目定位与核心能力
- 目录结构与模块职责
- 内部 API 接口说明
- GN 构建配置与编译产物
- 安全风险分析与修复建议
- 常见问题与调试指南

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── index.md              # 项目概览
├── Architecture.md        # 架构说明
├── Inner_API.md          # 内部 API 文档
├── Build.md              # GN 构建与产物
├── Security.md           # 安全风险评审
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md   # 配置开关
```

## 阅读建议

**新人入门路径**：
1. `index.md` - 快速了解模块定位
2. `Architecture.md` - 理解整体架构
3. `Inner_API.md` - 掌握核心 API 使用

**开发者进阶**：
1. `Build.md` - 了解编译配置
2. `Security.md` - 关注安全风险

## 证据来源

本文档所有关键结论均基于代码证据：

- 源码文件路径：`frameworks/datatransmitmgr/`、`interfaces/inner_api/datatransmitmgr/`
- 构建文件：`BUILD.gn`、`interfaces/inner_api/datatransmitmgr/BUILD.gn`
- API 定义：`interfaces/inner_api/datatransmitmgr/include/dev_slinfo_mgr.h`

## 文档维护

- **最后更新**：2024-02-06
- **更新方式**：手动更新，随代码变更同步维护
- **反馈渠道**：提交 Issue 至 base/security/dataclassification

## 排除范围

- 测试代码（`test/` 目录）不计入本文档
- N-API：本模块不提供对外 JS API，仅 Inner API
