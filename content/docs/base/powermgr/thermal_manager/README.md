# Thermal Manager Wiki

## 概述

本 Wiki 为 OpenHarmony thermal_manager 项目提供完整的工程文档，帮助开发者快速理解项目架构、API 接口、构建系统和安全风险。

## 文档覆盖范围

### 已覆盖
- [x] 项目定位与边界
- [x] 目录结构与模块职责
- [x] 架构说明（组件、数据流、线程模型）
- [ ] 对外 N-API（JS API 接口）
- [ ] 内部 API（模块接口、依赖方向）
- [ ] GN 目标梳理
- [ ] 编译产物说明
- [ ] 安全风险评审
- [ ] 常见问题与调试

### 未覆盖
- ETS/CJ API 详细说明
- Thermal Protector 的完整实现细节
- 各动作的详细实现逻辑

## 更新方式

本 Wiki 基于代码版本: main (截至 2026-02-06)

**如何更新文档**:
1. 修改代码后，同步更新相关文档章节
2. 保持代码证据路径准确（文件路径:行号）
3. 更新生成时间戳

## 生成时间

- 初版生成: 2026-02-06 04:30 (UTC+8)
- 最后更新: 2026-02-06 04:30 (UTC+8)

---

## 新人阅读顺序

推荐按以下顺序阅读，快速了解项目：

1. **[00_Overview.md](00_Overview.md)** - 项目概览、核心概念
2. **[01_Module_Boundaries.md](01_Module_Boundaries.md)** - 定位、边界、能力、环境
3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构、模块职责
4. **[03_Architecture.md](03_Architecture.md)** - 架构图、数据流、线程模型
5. **[04_NAPI_Interface.md](04_NAPI_Interface.md)** - 对外 JS API 接口
6. **[05_Inner_API.md](05_Inner_API.md)** - 内部 API、模块依赖
7. **[06_GN_Targets.md](06_GN_Targets.md)** - GN 构建系统、编译产物
8. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险分析
9. **[09_FAQ_Troubleshooting.md](09_FAQ_Troubleshooting.md)** - 常见问题与调试

### 进阶阅读
- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
- **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 配置宏说明
