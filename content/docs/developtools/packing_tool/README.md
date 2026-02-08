# OpenHarmony Packing Tool Wiki

## 项目概述

本文档是 OpenHarmony `packing_tool` 组件的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 接口和安全风险。

## 文档生成信息

- **生成时间**: 2025-02-06
- **组件版本**: 3.2
- **所属子系统**: developtools
- **源码路径**: `developtools/packing_tool/`

## 文档结构

```
wiki/
├── README.md              # 本文档 - Wiki 总览
├── SUMMARY.md             # 全站导航与阅读路线
├── index.md               # 首页/概览
├── 01_Overview.md         # 项目定位与核心能力
├── 02_Architecture.md     # 架构说明（Mermaid 图）
├── 03_Directory_Structure.md  # 目录结构与模块职责
├── 04_API_Reference.md    # 对外 API 参考
├── 05_Inner_API.md        # 内部 API 说明
├── 06_GN_Targets.md       # GN 构建目标
├── 07_Build_Artifacts.md  # 编译产物说明
├── 08_Security_Review.md  # 安全风险评审
├── 09_Troubleshooting.md  # 常见问题与调试
└── appendix/
    ├── Callgraphs.md      # 关键调用链
    └── Config_Flags.md    # 配置标志说明
```

## 阅读建议

### 新人入门路线

1. **快速了解**: 阅读 [首页](index.md) 和 [项目概述](01_Overview.md)
2. **理解架构**: 查看 [架构说明](02_Architecture.md) 中的组件图和数据流
3. **熟悉结构**: 浏览 [目录结构](03_Directory_Structure.md)
4. **使用工具**: 参考 [对外 API](04_API_Reference.md) 了解如何使用打包拆包功能
5. **排查问题**: 遇到问题时查阅 [常见问题](09_Troubleshooting.md)

### 开发者深入路线

1. **内部实现**: 阅读 [内部 API](05_Inner_API.md) 了解核心类设计
2. **构建系统**: 查看 [GN Targets](06_GN_Targets.md) 理解构建流程
3. **安全审计**: 参考 [安全风险评审](08_Security_Review.md) 了解潜在风险点
4. **调用链分析**: 查看 [附录 - 调用链](appendix/Callgraphs.md)

## 覆盖范围

### 已覆盖内容

- [x] 项目定位与核心能力
- [x] 架构设计与组件关系
- [x] 目录结构与模块职责
- [x] 对外 API 接口（Java）
- [x] 内部核心类设计
- [x] GN 构建系统与目标
- [x] 编译产物清单
- [x] 安全风险评审
- [x] 常见问题与调试方法

### 未覆盖内容（已知限制）

- [ ] C++ 实现（ohos_packing_tool）的详细 API（当前为简化实现）
- [ ] 单元测试详细说明（按需求排除）
- [ ] N-API 接口（本项目为纯 Java/C++ 命令行工具，无 N-API 暴露）

## 更新维护

本文档基于代码分析自动生成，建议随代码更新定期同步：

1. 当新增 Java 类时，更新 [目录结构](03_Directory_Structure.md) 和 [内部 API](05_Inner_API.md)
2. 当修改构建配置时，更新 [GN Targets](06_GN_Targets.md) 和 [编译产物](07_Build_Artifacts.md)
3. 当发现新的安全风险时，更新 [安全风险评审](08_Security_Review.md)

## 参考文档

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [打包工具使用说明](../README_zh.md)
- [LICENSE](../LICENSE) - Apache License 2.0
