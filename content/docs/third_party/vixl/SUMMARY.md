# 阅读路线指南

本文档帮助您快速定位所需信息，根据您的角色和需求选择合适的阅读路径。

## 快速定位

### 我想了解...

| 需求 | 推荐文档 |
|-----|---------|
| VIXL 是什么？有什么功能？ | **[01_Overview.md](01_Overview.md)** |
| OH 如何编译 VIXL？ | **[03_Build_Integration.md](03_Build_OH_Integration.md)** |
| 哪个模块在使用 VIXL？ | **[04_Usage_in_OH.md](04_Usage_in_OH.md)** |
| VIXL 与 OH 代码如何交互？ | **[05_API_Differences.md](05_API_Differences.md)** |

## 按角色阅读

### 方舟编译器开发者

**推荐阅读顺序**:
1. **[01_Overview.md](01_Overview.md)** - 了解 VIXL 核心功能
2. **[05_API_Differences.md](05_API_Differences.md)** - 重点关注 API 使用方式
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看集成示例

**关键信息**: VIXL 主要用于 `arkcompiler/runtime_core/static_core/compiler/optimizer/code_generator/` 目录下的代码生成器。

### 构建系统维护者

**推荐阅读顺序**:
1. **[03_Build_Integration.md](03_Build_Integration.md)** - 完整构建配置说明
2. **[01_Overview.md](01_Overview.md)** - 了解库的结构

**关键信息**: BUILD.gn 中的 `vixl_public_config` 和 `libvixl` 目标定义了完整的构建规则。

### 系统集成工程师

**推荐阅读顺序**:
1. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解依赖关系
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 验证构建配置

## 文档结构

```
wiki/
├── README.md              # 库概览和快速导航
├── SUMMARY.md             # 本文档，阅读路线指南
├── _work/
│   ├── ASSESSMENT.md     # 项目评估报告
│   ├── NOTES.md          # 分析过程记录
│   └── PLAN.md           # 任务进度跟踪
├── 01_Overview.md        # 原始库功能简介
├── 02_Patches.md         # Patch 分析（本库无 Patch）
├── 03_Build_Integration.md  # OH 构建适配
├── 04_Usage_in_OH.md     # OH 使用情况
└── 05_API_Differences.md # API 使用说明
```

## 关键信息速查

| 问题 | 答案 |
|-----|------|
| 需要修改源代码吗？ | **否**，VIXL 无需 Patch 即可使用 |
| 主要使用者是谁？ | **方舟编译器 (arkcompiler)** |
| 使用了哪些 OH 特定宏？ | `PANDA_BUILD`, `VIXL_CODE_BUFFER_MMAP` |
| 链接方式是什么？ | **静态链接** (`ohos_static_library`) |
| C++ 标准是什么？ | **C++17** |
| RTTI/异常支持？ | **均禁用** (`-fno-rtti`, `-fno-exceptions`) |

## 贡献指南

如果您需要：

- **报告 Bug**: 请通过 OH Issue 跟踪系统提交
- **提出功能需求**: 请联系库负责人 huanghuijin@huawei.com
- **贡献上游**: 请直接向上游仓库 https://github.com/Linaro/vixl 提交 PR
