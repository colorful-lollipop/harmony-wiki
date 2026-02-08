# ets_frontend Wiki 文档

## 文档覆盖范围

本文档覆盖 OpenHarmony `arkcompiler/ets_frontend` 仓库的工程文档，包括：

### 已覆盖内容
- **项目定位与核心能力**: ETS/JS/TS → ARK 字节码编译
- **目录结构**: es2panda、ets2panda、merge_abc、arkguard 等模块
- **命令行接口**: es2abc 工具选项
- **N-API 绑定**: ets2panda/bindings/native 模块
- **GN 构建系统**: targets、依赖、产物
- **模块职责划分**: 词法分析、语法解析、语义分析、字节码生成

### 未覆盖内容
- 详细编译器内部实现细节 (需要深入 IR/compiler 模块)
- 运行时集成细节 (参考 arkcompiler_ets_runtime)
- 测试用例和测试框架

## 更新方式

本文档基于代码分析自动生成。如需更新：

1. **代码结构变更**: 修改相关模块后，重新运行文档生成脚本
2. **新增模块**: 在 `SUMMARY.md` 中添加导航，在对应文档中增加章节
3. **API 变更**: 更新对应的 API 清单表，注明变更版本

## 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: 当前仓库 HEAD
- **文档版本**: 1.2

## 快速导航

建议阅读顺序：
1. [概览](00_Overview.md) - 了解项目定位
2. [目录结构](01_Directory_Structure.md) - 理解模块划分
3. [命令行接口](02_CLI_Reference.md) - 掌握编译工具使用
4. [N-API 绑定](03_NAPI_Bindings.md) - 了解 Native Interop
5. [GN 构建系统](04_Build_System.md) - 理解编译配置
6. [安全风险评审](05_Security_Review.md) - 了解安全考量
7. [FAQ](appendix/FAQ.md) - 常见问题解答

## 相关链接

- [OpenHarmony ARK Runtime 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/ARK-Runtime-Subsystem.md)
- [arkcompiler_runtime_core](https://gitee.com/openharmony/arkcompiler_runtime_core)
- [arkcompiler_ets_runtime](https://gitee.com/openharmony/arkcompiler_ets_runtime)
