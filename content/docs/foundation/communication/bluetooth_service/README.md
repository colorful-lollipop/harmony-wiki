# OpenHarmony Bluetooth Service 工程文档

## 文档说明

本文档集合为 OpenHarmony `bluetooth_service` 组件（子系统：communication）的工程 Wiki，旨在帮助新人快速理解项目架构、构建系统、安全机制等核心内容。

## 文档覆盖范围

### 已覆盖内容

✅ **项目概览** - [00_Overview.md](00_Overview.md)
- 组件定位与边界
- 核心能力与支持的 Profile
- 运行环境与依赖

✅ **目录结构** - [01_Directory_Structure.md](01_Directory_Structure.md)
- 模块职责划分
- 目录组织说明

✅ **架构设计** - [02_Architecture.md](02_Architecture.md)
- 分层架构（Server/Service/Stack/Hardware）
- 组件交互与数据流
- 线程模型与状态机
- 关键时序图

✅ **内部 API** - [03_Internal_API.md](03_Internal_API.md)
- 模块接口定义
- 接口稳定性说明
- 依赖关系

✅ **GN 构建系统** - [04_GN_Targets.md](04_GN_Targets.md)
- 关键 targets 分析
- Feature flags 说明
- 依赖关系图

✅ **编译产物** - [05_Build_Artifacts.md](05_Build_Artifacts.md)
- 输出文件清单
- 安装路径
- 运行时加载关系

✅ **安全风险评审** - [06_Security_Review.md](06_Security_Review.md)
- 攻击面分析
- 权限控制机制
- 已知风险点与修复建议

✅ **常见问题** - [07_Common_Issues.md](07_Common_Issues.md)
- 构建问题
- 运行时问题
- 调试技巧

### 附录

- [08_Config_Flags.md](08_Config_Flags.md) - Feature flags 与配置宏
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链

## 新人阅读顺序

建议按以下顺序阅读：

1. **[00_Overview.md](00_Overview.md)** - 了解项目整体定位
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 熟悉目录组织
3. **[02_Architecture.md](02_Architecture.md)** - 理解架构设计
4. **[03_Internal_API.md](03_Internal_API.md)** - 掌握模块接口
5. **[04_GN_Targets.md](04_GN_Targets.md)** - 了解构建系统
6. **[06_Security_Review.md](06_Security_Review.md)** - 了解安全机制

根据实际需求选择性阅读：
- 进行构建/调试：[07_Common_Issues.md](07_Common_Issues.md)
- 添加新功能：[04_GN_Targets.md](04_GN_Targets.md), [02_Architecture.md](02_Architecture.md)
- 安全审计：[06_Security_Review.md](06_Security_Review.md)

## 重要说明

### N-API JS API 绑定

⚠️ **重要**：本仓库（`bluetooth_service`）不包含 N-API JS API 绑定代码。

JS API 绑定位于其他仓库（推测为 `foundation/communication/bluetooth`）。本仓库仅包含：
- C++ 服务层实现
- System Ability（SA）服务
- IPC Server/Stub
- 蓝牙协议栈

### 测试代码引用

本文档**不引用**任何测试相关代码，包括：
- `test/` 目录
- `unittest/`、`fuzztest/`、`moduletest/`
- `*_test.*`、`*_fuzzer.*` 文件

所有结论均基于业务代码证据。

## 文档生成信息

- **生成时间**: 2026-02-06
- **代码版本**: commit 待确认
- **项目路径**: `/Volumes/lexar/code/d/work/oh/foundation/communication/bluetooth_service`
- **工具版本**: OpenHarmony GN 构建系统、C++17

## 文档更新方式

本文档基于代码静态分析生成，**随代码变更需手动更新**。

更新建议：
1. 定期运行代码分析工具（如 clang-tidy、grep 搜索）
2. 每个主要版本更新后重新扫描关键接口
3. 修改目录结构后同步更新 [01_Directory_Structure.md](01_Directory_Structure.md)
4. 新增/删除 Feature flags 后更新 [08_Config_Flags.md](08_Config_Flags.md)

## 贡献指南

如发现文档错误或需要补充：
1. 确认代码证据（提供文件路径和行号）
2. 更新对应的 `.md` 文件
3. 更新本文档的"已覆盖内容"部分

## 免责声明

本文档基于代码静态分析生成，可能存在以下局限性：
- 运行时行为未完全验证
- 部分 HDI 接口细节依赖外部实现
- 部分跨仓库依赖未深入分析

请以实际代码和官方文档为准。
