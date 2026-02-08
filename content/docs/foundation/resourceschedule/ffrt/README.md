# FFRT Wiki 文档说明

## 文档概述

本文档为 OpenHarmony FFRT (Function Flow Runtime) 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、API 使用、编译构建及安全注意事项。

## 覆盖范围

本文档涵盖以下内容：

| 模块 | 说明 |
|------|------|
| [概览](01_Overview.md) | 项目定位、核心能力、基本概念 |
| [架构设计](02_Architecture.md) | 组件架构、数据流、线程模型、关键时序 |
| [API 参考](03_API_Reference.md) | C/C++ API 清单、参数说明、调用示例 |
| [编译构建](04_Build.md) | GN Targets、编译参数、产物说明 |
| [安全风险](05_Security.md) | 攻击面分析、威胁模型、风险点及修复建议 |
| [故障排查](06_Troubleshooting.md) | 常见构建/运行/调试问题及定位 |

## 未覆盖范围

- **测试代码**: 测试用例、mock 代码不在本文档范围内
- **外部依赖**: third_party 依赖的内部实现细节
- **历史演进**: 早期版本的废弃 API 和迁移指南

## 文档维护

### 更新触发条件

本文档应在以下情况发生时更新：

1. **新增 API**: `interfaces/kits/` 新增头文件或函数
2. **架构变更**: `src/` 目录下的模块重构或依赖变化
3. **构建调整**: `BUILD.gn`、`ffrt.gni` 构建参数变更
4. **安全发现**: 新的安全漏洞或攻击面

### 更新方式

1. 克隆仓库
2. 修改 `wiki/` 目录下的相关 Markdown 文件
3. 确保所有关键结论有代码证据支撑（路径 + 符号）
4. 提交 PR 并通过代码审查

### 版本信息

- **文档生成时间**: 2025-02-06
- **FFRT 版本**: 4.0
- **适用系统**: OpenHarmony Standard System
- **系统能力**: SystemCapability.Resourceschedule.Ffrt.Core

## 阅读建议

### 新人入门路线

1. 先阅读 [概览](01_Overview.md) 理解 FFRT 定位
2. 阅读 [API 参考](03_API_Reference.md) 了解接口使用
3. 查看示例代码 `examples/` 目录

### 开发者深入路线

1. 阅读 [架构设计](02_Architecture.md) 理解内部实现
2. 阅读 [编译构建](04_Build.md) 熟悉构建流程
3. 参考 [故障排查](06_Troubleshooting.md) 解决开发问题

### 安全审查路线

1. 阅读 [安全风险](05_Security.md) 了解攻击面
2. 关注信任边界和数据流
3. 参考修复建议进行代码审计

## 相关链接

- **主仓库**: [OpenHarmony/foundation/resourceschedule/ffrt](https://gitee.com/openharmony/foundation_resourceschedule_ffrt)
- **用户指南**: `docs/README.md`
- **API 指南**: `docs/ffrt-api-guideline-c.md` / `docs/ffrt-api-guideline-cpp.md`
- **编译构建**: `BUILD.md`
