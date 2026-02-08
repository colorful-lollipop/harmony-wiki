# Data Share 工程 Wiki

## 简介

本 Wiki 面向 OpenHarmony Data Share 部件的工程文档，帮助开发者快速理解项目结构、API 接口、架构设计和安全风险。

**生成时间**: 2025-02-06  
**代码版本**: 3.2.0  
**仓库路径**: foundation/distributeddatamgr/data_share

## 适用读者

- **新人开发**: 从 [快速入门](01_Overview.md) 开始
- **API 使用者**: 查阅 [N-API 文档](04_NAPI_Reference.md)
- **架构师**: 阅读 [架构设计](05_Architecture.md)
- **安全工程师**: 参考 [安全分析](08_Security_Analysis.md)
- **构建工程师**: 查看 [构建系统](07_Build_System.md)

## 文档导航

### 入门必读
1. [概览](01_Overview.md) - 项目定位、核心能力、运行环境
2. [目录结构](02_Directory_Structure.md) - 模块职责与文件组织

### 接口文档
3. [对外 N-API](04_NAPI_Reference.md) - JS API 清单、参数、错误码
4. [内部 API](06_Inner_API.md) - C++ 接口定义、模块依赖

### 架构与实现
5. [架构设计](05_Architecture.md) - 组件图、数据流、线程模型
6. [关键调用链](appendix/Callgraphs.md) - 入口→核心逻辑调用链

### 构建与安全
7. [构建系统](07_Build_System.md) - GN Targets、编译产物、依赖
8. [安全分析](08_Security_Analysis.md) - 攻击面、风险点、修复建议

### 附录
- [配置开关](appendix/Config_Flags.md) - 关键宏与 feature flags
- [错误码速查](appendix/Error_Codes.md) - 完整错误码列表

## 更新方式

本 Wiki 基于代码自动生成，建议随代码版本更新同步刷新：

1. 代码变更后，更新相关章节
2. 检查 `SUMMARY.md` 链接有效性
3. 更新本文档的"代码版本"字段

## 注意事项

- 所有结论均基于代码证据（路径+行号）
- 测试代码（test/）内容不纳入文档证据
- 标注 `TODO` 的内容需人工确认后补充

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [分布式数据管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/分布式数据管理子系统.md)
- [Data Share 代码仓库](https://gitee.com/openharmony/distributeddatamgr_data_share)
