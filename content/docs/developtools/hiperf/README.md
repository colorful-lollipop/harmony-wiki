# hiperf Wiki

## 简介

本文档是 OpenHarmony hiperf 组件的工程 Wiki，旨在帮助开发者快速理解和使用 hiperf 性能分析工具。

## 覆盖范围

本文档覆盖以下内容：

- **项目概览**: 定位、边界、核心能力、运行环境
- **目录结构**: 模块职责、文件组织
- **架构说明**: 组件关系、数据流、线程模型、关键时序
- **对外 API**: C++ Inner API（hiperf_client、hiperf_local）
- **内部 API**: 模块接口、依赖方向
- **GN 构建**: Targets、依赖、产物
- **编译产物**: 文件清单、安装路径、加载关系
- **安全风险**: 攻击面、信任边界、可被利用点
- **常见问题**: 构建、运行、调试问题定位

## 未覆盖内容

- **测试代码**: 本文档不包含 test/ 目录相关内容
- **JS API**: hiperf 不提供 N-API 接口
- **详细实现**: 具体算法实现细节（请参考源代码）

## 更新方式

本文档基于代码生成，建议随代码更新同步更新：

1. 当接口变更时，更新 `04_Public_API.md` 和 `05_Internal_API.md`
2. 当构建系统变更时，更新 `06_GN_Targets.md` 和 `07_Build_Artifacts.md`
3. 当发现新的安全风险时，更新 `08_Security.md`

## 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: 4.0
- **基于提交**: developtools/hiperf (HEAD)

## 阅读指南

### 新人阅读顺序

1. [首页/概览](index.md) - 了解项目基本情况
2. [项目定位与核心能力](01_Overview.md) - 理解 hiperf 能做什么
3. [目录结构](02_Directory_Structure.md) - 了解代码组织
4. [架构说明](03_Architecture.md) - 理解系统架构
5. [对外 API](04_Public_API.md) - 学习如何使用 API
6. [安全风险](08_Security.md) - 了解安全注意事项

### 快速参考

- [GN Targets 速查](06_GN_Targets.md)
- [编译产物清单](07_Build_Artifacts.md)
- [配置 Flags](appendix/Config_Flags.md)
- [调用链参考](appendix/Callgraphs.md)

## 贡献指南

如需更新本文档：

1. 修改对应 `.md` 文件
2. 更新 `SUMMARY.md` 中的链接（如有新增页面）
3. 更新本文档的生成信息
4. 提交代码审查

## 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [hiperf 源码](../../) - 本文档对应的源代码
- [问题反馈](../../issues) - 提交问题或建议

## 许可证

本文档与 hiperf 项目使用相同的许可证：Apache License 2.0
