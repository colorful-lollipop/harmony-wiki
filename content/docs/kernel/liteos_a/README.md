# kernel_liteos_a Wiki 使用说明

## 文档覆盖范围

本文档涵盖 OpenHarmony **LiteOS Cortex-A** 内核的完整技术细节，包括：

### 已覆盖内容
- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- 系统调用接口（syscall）
- GN 构建系统与编译产物
- 内核模块详解（进程/内存/调度/IPC）
- 安全风险评审

### 未覆盖内容
- N-API / JS API（本项目为内核，不提供）
- 详细单元测试代码分析
- 具体硬件平台的适配细节

## 文档更新方式

### 随代码更新
当以下内容变更时需要同步更新 Wiki：
1. 新增/删除内核模块
2. 修改系统调用接口
3. 变更 GN 构建配置
4. 新增安全特性

### 更新步骤
```bash
# 1. 修改对应的 wiki/*.md 文件
# 2. 更新 SUMMARY.md 导航链接
# 3. 验证文档链接正确性
```

### 版本对应
- 本文档对应代码版本：见 `CHANGELOG.md`
- 文档生成时间：2026-02-06

## 贡献指南

欢迎贡献 Wiki 文档，请遵循以下规范：
1. 所有关键结论必须包含代码证据（文件路径 + 行号）
2. 术语保持一致
3. 不引用测试代码作为业务证据
4. 接口文档需包含完整的参数说明

## 反馈与问题

如有文档问题，请提交 Issue 到项目仓库。

## 目录导航

- [首页](/README.md)
- [概览](/00_Overview.md)
- [目录结构](/01_Directory_Structure.md)
- [架构说明](/02_Architecture.md)
- [构建系统](/03_Build_System.md)
- [内核模块](/04_Kernel_Modules.md)
- [系统调用接口](/05_Syscall_API.md)
- [安全评审](/06_Security_Review.md)
- [附录：调用链](/appendix/Callgraphs.md)
- [附录：配置项](/appendix/Config_Flags.md)
