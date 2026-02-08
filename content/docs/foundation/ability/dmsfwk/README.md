# dmsfwk Wiki 文档

## 概述

本文档是 OpenHarmony **dmsfwk (Distributed Ability Manager Service Framework)** 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、N-API 接口、内部模块、构建配置以及安全风险。

## 覆盖范围

### 已覆盖

- [项目概述与核心能力](00_Overview.md)
- [系统架构与模块职责](01_Architecture.md)
- [N-API 对外接口详解](02_NAPI.md)
- [Inner API 内部接口](03_InnerAPI.md)
- [GN 构建配置与产物](04_Build.md)
- [安全风险评审](05_Security.md)

### 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── 00_Overview.md              # 项目概述
├── 01_Architecture.md           # 架构说明
├── 02_NAPI.md                   # N-API 接口
├── 03_InnerAPI.md               # 内部接口
├── 04_Build.md                  # 构建配置
├── 05_Security.md               # 安全评审
└── appendix/
    └── Callgraphs.md            # 关键调用链
```

## 文档更新方式

本文档基于代码分析自动生成。如需更新：

1. **代码变更**：修改代码后，重新运行 Wiki 生成工具
2. **配置变更**：更新 `bundle.json`、`dmsfwk.gni` 后重新生成
3. **新增模块**：在对应模块下添加文档并更新 SUMMARY.md

## 版本信息

- **生成时间**: 2025-02-06
- **代码版本**: dmsfwk v3.1
- **OpenHarmony 版本**: Standard
