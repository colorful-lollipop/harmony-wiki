# 目录

## 推荐阅读路径

### 新人学习路线

快速理解项目定位和架构：

1. [Wiki 首页](README.md) - 了解 Wiki 使用指南
2. [项目概览](00_Overview.md) - 5 分钟了解项目定位和核心能力
3. [目录结构](01_Directory_Structure.md) - 15 分钟熟悉代码组织
4. [架构说明](02_Architecture.md) - 30 分钟理解系统架构和数据流
5. [对外 API](03_External_APIs.md) - 掌握接口使用方法

**预计时间**: 1-2 小时

### 安全研究路线

快速识别攻击面和安全风险：

1. [Wiki 首页](README.md) - 了解 Wiki 使用指南
2. [项目概览](00_Overview.md) - 了解项目定位和核心能力
3. [安全风险分析](06_Security_Analysis.md) - 识别 6 大可利用点
4. [对外 API](03_External_APIs.md) - 了解所有外部输入接口
5. [内部 API](04_Internal_APIs.md) - 了解内部敏感操作
6. [附录 - 关键调用链](appendix/Callgraphs.md) - 深入关键调用链

**预计时间**: 2-3 小时

## 文档目录

### 核心文档

- [Wiki 首页](README.md) - 使用指南和阅读建议
- [项目概览](00_Overview.md) - 定位、边界、核心能力、运行环境
- [目录结构](01_Directory_Structure.md) - 模块职责与文件组织
- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型、关键时序

### API 文档

- [对外 API](03_External_APIs.md) - C++ Inner API 与 IPC 接口
- [内部 API](04_Internal_APIs.md) - 模块内部接口和依赖关系

### 构建与产物

- [GN Targets](05_GN_Targets.md) - 构建目标与依赖
- [编译产物](07_Build_Products.md) - 输出文件与安装路径

### 安全与调试

- [安全风险分析](06_Security_Analysis.md) - 威胁模型、攻击面、可利用点分析
- [常见问题排查](08_Troubleshooting.md) - 调试与定位

### 附录

- [关键调用链](appendix/Callgraphs.md)
- [配置参数](appendix/Config_Flags.md)
