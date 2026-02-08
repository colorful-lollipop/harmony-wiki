# Wiki 文档说明

本文档为 OpenHarmony 分布式数据管理仓颉封装（distributeddatamgr_cangjie_wrapper）的工程 Wiki。

## 文档概述

本 Wiki 旨在为新人提供完整的项目理解，包括：
- 项目定位与核心能力
- 目录结构与模块职责
- 架构设计与数据流
- 对外 API（仓颉 API）接口清单
- 内部 FFI 层实现
- GN 构建目标与编译产物
- 安全风险分析与评审

## 文档生成信息

- **生成时间**: 2026-02-06
- **覆盖范围**: 整个代码库（不含测试代码）
- **证据溯源**: 所有关键结论均提供代码证据（文件路径 + 符号名 + 行号）
- **语言版本**: API Level 22

## 文档更新方式

当项目代码发生以下变更时，建议更新本文档：

1. **API 变更**（新增/修改/删除公共接口）：
   - 更新 `04_Public_API.md`
   - 如涉及 FFI 层，同步更新 `05_Internal_API.md`

2. **架构重构**（模块调整、依赖变更）：
   - 更新 `03_Architecture.md`
   - 更新 `06_GN_Targets.md`

3. **新增安全机制**：
   - 更新 `08_Security_Review.md`

4. **构建系统变更**（新增 target、修改依赖）：
   - 更新 `06_GN_Targets.md`
   - 更新 `07_Build_Artifacts.md`

## 未覆盖范围

本文档当前未覆盖以下内容：

- **测试代码**: 所有 `test/` 目录下的测试用例和测试桩
- **底层实现**: 不包含底层 C++ 组件的实现细节（如 distributeddatamgr_preferences 的 C++ 实现）
- **性能分析**: 未包含性能测试数据和优化建议
- **使用示例**: 详细的使用示例请参考 [仓颉语言官方文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)

## 文档约定

### 证据标注格式

关键结论均使用以下格式标注证据：
```
证据: <文件路径>:<行号> - <符号/函数名>
```

### 约束标注格式

未实现或有约束的功能使用以下格式标注：
```
TODO(需确认): <说明>
```

### 错误码格式

错误码统一格式：
```
<错误码> - <错误描述>
```

## 目录

- [SUMMARY.md](SUMMARY.md) - 全站导航与阅读顺序
- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Project_Boundaries.md](01_Project_Boundaries.md) - 项目定位与边界
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构与模块职责
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Public_API.md](04_Public_API.md) - 对外 API 清单
- [05_Internal_API.md](05_Internal_API.md) - 内部 FFI API
- [06_GN_Targets.md](06_GN_Targets.md) - GN 目标梳理
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物
- [08_Security_Review.md](08_Security_Review.md) - 安全风险评审
- [09_QA.md](09_QA.md) - 常见问题

## 附录

- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置标志
