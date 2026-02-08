# sys_installer Wiki

## 简介

本文档是 OpenHarmony `sys_installer`（系统安装部件）的工程 Wiki，提供从架构到实现的全面技术参考。

**生成时间**: 2025-02-06  
**代码版本**: 3.2  
**目标系统**: OpenHarmony 标准系统 (standard)

## 覆盖范围

本文档涵盖以下内容：

- **项目概览**: 定位、边界、核心能力、运行环境
- **目录结构**: 模块职责与文件组织
- **架构说明**: 组件图、数据流、线程模型、关键时序
- **对外 API**: C++ Inner API、IPC 接口、权限与错误码
- **内部 API**: 模块接口、依赖方向、稳定性分析
- **GN 构建**: Targets 清单、依赖关系、编译产物
- **安全风险**: 攻击面、信任边界、可利用点分析
- **问题排查**: 常见问题与定位路径

## 未覆盖范围

- 测试代码 (`test/` 目录下的内容)
- JavaScript/ArkTS API (sys_installer 是纯 Native 服务，无 N-API 暴露)
- 详细的性能基准数据

## 阅读建议

### 新人学习路线

**目标**: 5 分钟理解项目定位，30 分钟理解架构，1-2 小时掌握基本使用

1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [目录结构](01_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](02_Architecture.md) - 理解系统架构
4. [对外 API](03_External_APIs.md) - 掌握接口使用

**预计时间**: 1-2 小时

### 安全研究路线

**目标**: 快速识别攻击面，定位敏感操作，理解安全风险

1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [安全风险分析](06_Security_Analysis.md) - 识别 6 大可利用点
3. [对外 API](03_External_APIs.md) - 了解所有外部输入接口
4. [内部 API](04_Internal_APIs.md) - 了解内部敏感操作
5. [架构说明](02_Architecture.md) - 理解信任边界和数据流
6. [附录 - 调用链](appendix/Callgraphs.md) - 深入关键调用链

**预计时间**: 2-3 小时

### 开发人员路线

**目标**: 深入理解内部实现，掌握构建系统，参与开发

1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [目录结构](01_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](02_Architecture.md) - 理解系统架构
4. [内部 API](04_Internal_APIs.md) - 了解内部模块接口
5. [GN 构建](05_GN_Targets.md) - 掌握构建系统
6. [安全风险](06_Security_Analysis.md) - 理解安全机制
7. [附录 - 调用链](appendix/Callgraphs.md) - 深入关键调用链

**预计时间**: 3-4 小时

## 更新方式

本文档基于代码仓库自动生成，建议随代码迭代同步更新：

1. 修改代码后，检查相关 Wiki 页面是否需要更新
2. 新增模块时，在对应章节添加说明
3. 安全相关变更必须同步更新安全分析文档

## 相关资源

- [OpenHarmony 升级子系统](https://gitcode.com/openharmony/update_updater)
- [OpenHarmony 官方文档](https://docs.openharmony.cn)

---

*本文档由工程 Agent 自动生成，如有疑问请参考代码源文件。*
