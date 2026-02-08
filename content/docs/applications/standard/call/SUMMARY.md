# 文档导航

本文档为 **applications_call** (OpenHarmony 通话应用) 的完整技术文档。

## 快速导航

| 章节 | 描述 | 优先级 |
|-----|------|-------|
| [README](README.md) | 项目入门和快速开始 | ⭐⭐⭐ |
| [概览](01_Overview.md) | 项目定位、功能特性 | ⭐⭐⭐ |
| [架构](02_Architecture.md) | 系统架构、组件关系 | ⭐⭐⭐ |
| [API 参考](03_API_Reference.md) | 系统 API 使用说明 | ⭐⭐ |
| [构建指南](04_Build_Guide.md) | 构建配置和产物说明 | ⭐⭐ |
| [安全评审](05_Security_Review.md) | 安全风险分析 | ⭐⭐ |
| [模块详解](06_Module_Details.md) | 各模块详细说明 | ⭐ |

## 文档地图

```
📁 wiki/
├── 📄 README.md                    ←从这里开始
├── 📄 SUMMARY.md                   ←本文档，全站导航
├── 📄 01_Overview.md              ←项目概述
├── 📄 02_Architecture.md           ←架构设计
├── 📄 03_API_Reference.md         ←API 使用
├── 📄 04_Build_Guide.md           ←构建配置
├── 📄 05_Security_Review.md       ←安全分析
├── 📄 06_Module_Details.md        ←模块说明
└── 📁 appendix/
    ├── 📄 Callgraphs.md           ←调用链图谱
    └── 📄 Config_Flags.md          ←配置项
```

## 阅读路线推荐

### 🟢 新人入门 (30 分钟)
1. `README.md` → 快速了解项目
2. `01_Overview.md` → 理解项目定位
3. `02_Architecture.md` → 掌握整体架构

### 🟡 应用开发 (1-2 小时)
1. `03_API_Reference.md` → 学习系统 API 使用
2. `06_Module_Details.md` → 了解模块结构
3. `04_Build_Guide.md` → 掌握构建流程

### 🔴 安全审计 (2-3 小时)
1. `05_Security_Review.md` → 安全风险清单
2. `03_API_Reference.md` → API 权限验证
3. `02_Architecture.md` → 信任边界分析

## 关键链接

- **项目源码**: `/Volumes/lexar/code/d/work/oh/applications/standard/call`
- **构建入口**: `BUILD.gn`
- **主模块**: `entry/src/main/`
- **配置目录**: `AppScope/`
