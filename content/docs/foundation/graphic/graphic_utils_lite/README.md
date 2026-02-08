# graphic_utils_lite Wiki 使用指南

## 文档概述

本文档为 `graphic_utils_lite` 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、核心能力、编译配置以及安全注意事项。

## 覆盖范围

### 已覆盖内容

| 模块 | 说明 | 证据来源 |
|------|------|----------|
| 项目定位 | 图形子系统公共工具库 | `README.md` |
| 目录结构 | 框架代码、接口分类 | `interfaces/`, `frameworks/` |
| 核心模块 | Utils、Diagram、HAL | 代码分析 |
| 构建配置 | GN targets、feature flags | `BUILD.gn`, `bundle.json` |
| 内部 API | Inner API 头文件清单 | `interfaces/innerkits/` |
| 外部 API | Kits 头文件清单 | `interfaces/kits/gfx_utils/` |
| 依赖关系 | 外部依赖、被依赖模块 | `bundle.json` |
| 数据类型 | 核心结构体、枚举 | `graphic_types.h` |

### 未覆盖内容

| 模块 | 原因 |
|------|------|
| 测试代码 | 根据规范，测试相关内容不纳入 Wiki |
| 平台特定实现（linux/liteos/windows） | 源代码中未发现独立目录，相关适配在 framework/hals 中 |
| 运行时加载细节 | 需查看产品配置，此处未包含 |

## 阅读顺序建议

对于新加入的开发者，建议按以下顺序阅读：

1. **[概览](00_Overview.md)** - 了解项目定位、核心能力、运行环境
2. **[架构说明](01_Architecture.md)** - 理解模块划分、数据流、线程模型
3. **[内部 API](03_InnerAPI.md)** - 掌握模块间接口调用方式
4. **[GN 构建配置](04_Build.md)** - 理解编译流程、产物生成
5. **[安全评估](05_Security.md)** - 了解安全风险与最佳实践

## 快速导航

```
根目录
├── README.md              ← 本文，Wiki 使用指南
├── SUMMARY.md             ← 全站导航
├── 00_Overview.md         ← 项目概览
├── 01_Architecture.md      ← 架构说明
├── 02_NAPI.md             ← N-API 接口（本项目无 N-API）
├── 03_InnerAPI.md         ← 内部 API
├── 04_Build.md            ← GN 构建配置
├── 05_Security.md         ← 安全风险评估
└── 06_FAQ.md              ← 常见问题
```

## 术语表

| 术语 | 定义 |
|------|------|
| Utils | 图形子系统公共工具模块，提供数据结构、OS 适配 |
| Diagram | 2D 图形引擎，包含光栅化、顶点生成、图元处理 |
| HAL | Hardware Abstraction Layer，硬件抽象层 |
| NDK | Native Development Kit，本项目的 C++ 库分发形式 |
| GN | Generate Ninja，构建系统 |
| Feature Flag | 编译开关，控制功能模块是否启用 |

## 版本信息

| 项目 | 值 |
|------|-----|
| 项目版本 | 3.1 |
| ROM 占用 | ~450KB |
| RAM 占用 | ~50KB |
| 适配系统 | mini, small |
| 许可证 | Apache-2.0 |

## 如何更新本文档

当项目代码发生变更时，请同步更新 Wiki：

1. **新增模块**：在 `01_Architecture.md` 添加模块说明，在 `03_InnerAPI.md` 添加 API 清单
2. **修改 GN 配置**：在 `04_Build.md` 更新 targets 列表和 feature flags
3. **发现安全问题**：在 `05_Security.md` 添加风险项和修复建议
4. **FAQ 补充**：在 `06_FAQ.md` 添加新问题和解答

**更新原则**：
- 关键结论必须附带代码证据（文件路径 + 符号名）
- 禁止引用测试代码作为业务证据
- 保持术语统一，与代码注释一致

## 反馈与贡献

如发现文档错误或遗漏，请通过以下方式反馈：

- 在 Gitee 仓库提交 Issue
- 直接修改文档后提交 Pull Request

---

*最后更新时间：2026-02-06*
*基于代码版本：3.1*
