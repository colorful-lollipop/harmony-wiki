# cangjie_tools 在 OpenHarmony 中的集成文档

本文档说明 OpenHarmony 对 Cangjie 语言工具链的集成与适配。

---

## 库概览

**库名称**：cangjie_tools
**上游版本**：1.1.0-alpha.69
**上游地址**：https://cangjie-lang.cn/
**许可证**：Apache-2.0 with Runtime Library Exceptions
**OH 组件名称**：`@ohos/cangjie_tools`
**所属子系统**：thirdparty

---

## Cangjie 工具链简介

cangjie_tools 是 Cangjie（仓颉）语言的命令行工具链，包含以下工具：

| 工具 | 名称 | 功能 |
|------|------|------|
| cjpm | Cangjie Project Manager | 项目管理工具，支持项目初始化、依赖管理、编译、打包 |
| cjfmt | Cangjie Formatter | 代码格式化工具，基于 Cangjie 编码规范自动格式化代码 |
| lsp | Language Server Protocol | 语言服务器，为 DevEco Studio 提供智能提示、导航、重构等 IDE 功能 |
| cjlint | Cangjie Lint Tool | 静态代码检查工具，发现违反编码规范的问题和安全漏洞 |
| cjcov | Cangjie Coverage Tool | 代码覆盖率工具，帮助开发者提高测试完整性 |
| cjtrace-recover | Exception Stack Trace Recovery | 异常堆栈恢复工具，恢复混淆后的堆栈信息，用于问题定位 |
| hle | HyperLang Extension | ArkTS 互操作代码生成工具，生成 Cangjie 调用 ArkTS 的桥接代码 |

---

## OpenHarmony 适配概述

### 1. 适配策略

**直接同步上游代码**：OpenHarmony 采用直接同步上游代码的方式维护 cangjie_tools，不使用 patch 文件管理修改。每次更新通过自动同步脚本从上游仓颉语言官方仓库拉取完整代码。

**Git 维护记录**：
```
4c7b843d !103 merge auto-sync-cangjie_tools-20260205160047 into master
bca17419 feat(cangjie_tools): sync from upstream v1.1.0-alpha.69
```

### 2. 构建系统适配

**独立构建系统**：该工具链不使用 OpenHarmony 主构建系统的 GN/Ninja，而是采用自有的 **Python 构建脚本 + CMake** 混合架构。

**构建文件结构**：
- 7 个 Python 构建脚本（各子目录 `build/build.py`）
- 5 个 CMakeLists.txt 文件（C++ 项目）
- 1 个 bundle.json（OpenHarmony 组件定义）

### 3. OpenHarmony 特定支持

#### 编译目标支持

| 目标平台 | 编译器 |
|---------|--------|
| ohos-x86_64 | `x86_64-unknown-linux-ohos-clang/clang++` |
| ohos-aarch64 | `aarch64-unknown-linux-ohos-clang/clang++` |

**支持 OHOS 目标的工具**：cjfmt

#### ArkTS 互操作支持

**HLE 工具**：自动生成 ArkTS 与 Cangjie 的互操作代码，包括：
- `ohos.ark_interop.*` 包自动导入
- `ohosGlobalApiCall` 跨语言调用函数生成
- OHOS 构建（BUILD.gn）配置生成

#### DevEco Studio 深度集成

**LSP 集成**：
- 检测 DevEco 环境（`GetIsDeveco()`）
- 自动设置 OHOS 默认目标（`aarch64-linux-ohos`）
- 支持 OHOS 特定的 CJD 索引
- Interop 自动导入补全

#### 条件编译支持

**Cangjie 特性**：
```cangjie
@When[os == "ohos"] {
    // OpenHarmony 特定代码
}
```

**C++ 条件编译**：
```cpp
#ifdef __OHOS__
// OpenHarmony 特定代码
#endif
```

### 4. 第三方依赖适配

所有第三方依赖均从 OpenHarmony 官方仓库下载，使用 `OpenHarmony-v6.0-Release` 分支：

| 依赖 | 用途 | OpenHarmony 仓库 |
|------|------|----------------|
| flatbuffers | 序列化/反序列化 | `third_party_flatbuffers` |
| JSON for Modern C++ | JSON 解析 | `third_party_json` |
| SQLite | 索引存储 | `third_party_sqlite` |

---

## 文档导航

### 核心文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介，包括库名称、版本、许可证、功能描述 |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（该库无 Patch 文件） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配，包括构建系统结构、编译选项、OH 特定配置 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用情况，包括谁在使用、使用方式、典型场景 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果，包含基础信息、Patch 分析、依赖分析、特殊适配识别 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度 |

### 参考文档

| 文档 | 说明 |
|------|------|
| [../README.md](../README.md) | 项目主文档 |
| [../README_zh.md](../README_zh.md) | 项目主文档（中文） |
| [../third_party/README.md](../third_party/README.md) | 第三方依赖说明 |

---

## 快速开始

### 阅读建议

**如果您是**：
- **OpenHarmony 系统开发者**：从 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 开始，了解该工具链在 OH 中的使用方式
- **Cangjie 语言用户**：从 [01_Overview.md](./01_Overview.md) 开始，了解工具链功能
- **构建系统维护者**：重点阅读 [03_Build_Integration.md](./03_Build_Integration.md)，了解 OH 特定的构建配置
- **版本升级维护者**：阅读 [02_Patches.md](./02_Patches.md)，了解该库的维护策略

### 核心要点

1. **无 Patch 文件**：该仓库采用直接同步上游代码的方式，不使用 patch 文件
2. **独立构建系统**：不使用 GN，而是 Python + CMake 混合架构
3. **全面 OHOS 支持**：支持 `ohos-x86_64` 和 `ohos-aarch64` 目标，深度集成 DevEco Studio
4. **ArkTS 互操作**：HLE 工具提供完整的跨语言互操作支持
5. **无运行时依赖**：该工具链作为独立开发工具使用，不被其他 OH 模块依赖

---

## 联系与反馈

如有问题或建议，请参考：
- 上游仓库：https://cangjie-lang.cn/
- OpenHarmony issue：对应子系统的 issue 跟踪
