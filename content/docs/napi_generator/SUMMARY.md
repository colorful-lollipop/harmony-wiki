# napi-generator Wiki 导航

> 本文档提供全站导航，助您快速定位所需内容。

## 📖 新人阅读路线

**推荐顺序**: 建议新人按以下顺序阅读，快速建立整体认知。

```
第 1 天: 概览与结构
├── 📗 index.md              (5 分钟) 项目首页
├── 📗 01_Overview.md        (10 分钟) 项目定位与核心能力
├── 📗 02_Directory_Structure.md (15 分钟) 目录结构
└── 📗 03_Architecture.md    (15 分钟) 整体架构

第 2 天: 深入模块
├── 📘 04_NAPI_Reference.md (30 分钟) N-API 参考 (按需选择)
│   ├── dts2cpp 工具详解
│   ├── h2sa 工具详解
│   └── h2dtscpp 工具详解
├── 📘 05_Internal_API.md    (20 分钟) 内部模块接口
└── 📘 06_Build_System.md   (15 分钟) 构建系统

第 3 天: 安全与实践
├── 📕 07_Security_Review.md (20 分钟) 安全风险评审
├── 📕 08_Troubleshooting.md (15 分钟) 常见问题
└── 📕 examples/napitutorials/ (实践) N-API 教程示例
```

---

## 📚 完整章节列表

### 快速入门

| 章节 | 标题 | 摘要 |
|------|------|------|
| [index.md](index.md) | 项目首页 | 快速了解 napi-generator |
| [README.md](README.md) | Wiki 使用指南 | 覆盖范围、更新方式 |
| [01_Overview.md](01_Overview.md) | 项目概览 | 定位、边界、核心能力、运行环境 |

### 架构与结构

| 章节 | 标题 | 摘要 |
|------|------|------|
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构 | src/cli/*, examples/* 模块职责 |
| [03_Architecture.md](03_Architecture.md) | 架构说明 | 组件图、数据流、线程模型、时序图 |
| [03_CodeMap.md](03_CodeMap.md) | 代码导航图 | 功能到文件路径的快速索引 |

### API 参考

| 章节 | 标题 | 摘要 |
|------|------|------|
| [04_NAPI_Reference.md](04_NAPI_Reference.md) | N-API 参考 | 6 个工具的 API 清单、注册点、调用链 |
| [05_Internal_API.md](05_Internal_API.md) | 内部 API | 模块接口、依赖方向、稳定性标注 |

### 构建与部署

| 章节 | 标题 | 摘要 |
|------|------|------|
| [06_Build_System.md](06_Build_System.md) | 构建系统 | GN Targets、编译产物、依赖关系 |

### 安全与运维

| 章节 | 标题 | 摘要 |
|------|------|------|
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 | 输入/文件系统/命令执行攻击面 |
| [07_Security_Review.md](07_Security_Review.md) | 安全评审 | 威胁模型、风险点、修复建议 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见问题 | 构建/运行/调试问题与定位路径 |

---

## 🔗 工具速查表

| 工具 | 入口文件 | 主要功能 | 章节链接 |
|------|----------|----------|----------|
| dts2cpp | `src/cli/dts2cpp/src/gen/cmd_gen.js` | TS → N-API 代码生成 | [04_NAPI_Reference.md](04_NAPI_Reference.md#1-dts2cpp-工具) |
| h2sa | `src/cli/h2sa/src/gen/main.js` | SA 服务框架生成 | [04_NAPI_Reference.md](04_NAPI_Reference.md#2-h2sa-工具) |
| h2dtscpp | `src/cli/h2dtscpp/src/src/main.js` | C++ → TS+NAPI+测试 | [04_NAPI_Reference.md](04_NAPI_Reference.md#3-h2dtscpp-工具) |
| h2dts | `src/cli/h2dts/src/` | C++ → TypeScript | [04_NAPI_Reference.md](04_NAPI_Reference.md#4-h2dts-工具) |
| cmake2gn | `src/cli/cmake2gn/src/` | CMake → GN | [04_NAPI_Reference.md](04_NAPI_Reference.md#5-cmake2gn-工具) |
| scan | `src/tool/api/src/` | API 依赖扫描 | [04_NAPI_Reference.md](04_NAPI_Reference.md#6-scan-工具) |

---

## 🎯 常见场景导航

| 场景 | 需要了解 | 推荐章节 |
|------|----------|----------|
| **我想生成一个 N-API 模块** | dts2cpp 使用方法 | 04_NAPI_Reference.md §1 |
| **我想创建一个 SA 服务** | h2sa 使用方法 | 04_NAPI_Reference.md §2 |
| **我要添加新函数类型** | N-API 生成逻辑 | 03_Architecture.md + 05_Internal_API.md |
| **我要修改构建脚本** | GN Targets 配置 | 06_Build_System.md |
| **我发现安全问题** | 安全风险与修复 | 05_AttackSurface.md → 07_Security_Review.md |
| **构建失败了** | 问题排查 | 08_Troubleshooting.md |

---

## 📁 示例项目索引

| 示例 | 路径 | 用途 |
|------|------|------|
| N-API 教程 | `examples/napitutorials/` | 75+ N-API 示例 |
| AKI 框架 | `examples/akitutorials/` | AKI 简化开发 |
| SA 服务 | `examples/serviceCode/` | h2sa 输出示例 |
| 完整应用 | `examples/p7zipTest/` | 生产级架构 |

---

## 🔧 配置与标志

- 详见 [appendix/Config_Flags.md](appendix/Config_Flags.md) (待完善)
- 详见 [appendix/Callgraphs.md](appendix/Callgraphs.md) (待完善)

---

> 最后更新: 2026-02-07
