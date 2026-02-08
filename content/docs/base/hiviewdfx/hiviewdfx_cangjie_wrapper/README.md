# DFX Cangjie封装 Wiki

## 项目概述

本 Wiki 文档为 `hiviewdfx_cangjie_wrapper` 项目提供完整的技术文档。该项目是 OpenHarmony DFX (Design for X) 能力的 Cangjie 语言封装，为 Cangjie 开发者提供 DF（可靠性设计）和 DT（可测试性设计）能力。

**代码仓库路径**: `base/hiviewdfx/hiviewdfx_cangjie_wrapper`

**项目状态**: Beta

**支持的设备类型**: standard

---

## 覆盖范围

### 已文档化的内容

| 文档 | 描述 | 状态 |
|------|------|------|
| [README.md](README.md) | 本文档，说明与导航 | ✅ 完成 |
| [SUMMARY.md](SUMMARY.md) | 全站导航与新人阅读路线 | ✅ 完成 |
| [index.md](index.md) | 项目首页 | ✅ 完成 |
| [01_Overview.md](01_Overview.md) | 项目概览 | ✅ 完成 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构 | ✅ 完成 |
| [03_Architecture.md](03_Architecture.md) | 架构说明 | ✅ 完成 |
| [04_External_API.md](04_External_API.md) | 对外API（N-API） | ✅ 完成 |
| [05_Internal_API.md](05_Internal_API.md) | 内部API | ✅ 完成 |
| [06_GN_Build.md](06_GN_Build.md) | GN构建系统 | ✅ 完成 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物 | ✅ 完成 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审 | ✅ 完成 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题 | ✅ 完成 |

### 核心能力

1. **HiLog** - 流水日志系统，支持多级别日志输出
2. **HiAppEvent** - 应用事件打点与订阅框架
3. **HiTraceMeter** - 性能追踪与打点

### 暂未覆盖的内容

- 详细单元测试用例说明
- 性能基准测试数据
- 完整的 FFI 实现细节

---

## 使用说明

### 新人阅读路线

建议阅读顺序：
1. [index.md](index.md) - 项目首页，了解项目定位
2. [01_Overview.md](01_Overview.md) - 项目概览，理解核心能力
3. [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构，熟悉代码组织
4. [03_Architecture.md](03_Architecture.md) - 架构说明，理解模块关系
5. [04_External_API.md](04_External_API.md) - API文档，查找接口使用
6. 根据需要查阅其他文档

### 查找特定信息

- **API使用**: 查阅 [04_External_API.md](04_External_API.md)
- **构建配置**: 查阅 [06_GN_Build.md](06_GN_Build.md)
- **编译产物**: 查阅 [07_Build_Artifacts.md](07_Build_Artifacts.md)
- **安全问题**: 查阅 [08_Security_Review.md](08_Security_Review.md)

---

## 文档更新

### 更新方式

当代码发生变化时，需同步更新相关 Wiki 文档：

1. **API变更**: 更新 [04_External_API.md](04_External_API.md) 中的 API 清单表
2. **构建配置变更**: 更新 [06_GN_Build.md](06_GN_Build.md) 中的 targets 列表
3. **新增模块**: 在 [02_Directory_Structure.md](02_Directory_Structure.md) 添加模块说明
4. **架构调整**: 更新 [03_Architecture.md](03_Architecture.md) 和相关架构图
5. **安全问题**: 在 [08_Security_Review.md](08_Security_Review.md) 添加/更新风险项

### 代码证据要求

所有关键结论必须可在代码仓库中找到直接证据：
- 文件路径（必要时包含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

---

## 相关链接

### 内部文档
- 项目 README: ../../README.md
- 项目 README (中文): ../../README_zh.md
- bundle.json: ../../bundle.json
- BUILD.gn: ../../BUILD.gn

### 外部参考
- [OpenHarmony DFX 设计指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/design/debug/)
- [Cangjie API Reference](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [Performance Analysis Kit 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/dfx/cj-performance-analysis-kit-overview.md)

---

## 文档信息

- **最后更新**: 2026-02-06
- **文档版本**: 1.0.0
- **负责团队**: hiviewdfx_cangjie_wrapper Maintainers
