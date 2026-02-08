# 原始库简介

本文档简要介绍 Cangjie 语言工具链的原始库信息。

---

## 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | cangjie_tools |
| **许可证** | Apache-2.0 with Runtime Library Exceptions |
| **版本** | 1.1.0-alpha.69 |
| **上游地址** | https://cangjie-lang.cn/ |
| **所有者** | zhangjun283@huawei.com |
| **描述** | The Cangjie language provides a rich set of command line tools and language server tool for developers. |

---

## 原始功能描述

Cangjie（仓颉）是华为推出的新一代编程语言，cangjie_tools 是其配套的命令行工具链，为开发者提供完整的项目管理、开发、调试支持。

**一句话描述**：Cangjie 语言的开发工具链，包含项目管理、代码格式化、静态检查、语言服务器等工具。

---

## 工具组件

### 1. cjpm（Cangjie Project Manager）

**功能**：项目管理工具

**主要特性**：
- 项目初始化
- 依赖管理和更新
- 统一编译入口
- 增量编译和并行编译支持
- 多平台支持（Windows, Linux, macOS, OpenHarmony）

**支持的平台**：
- native（原生平台）
- windows-x86_64（Windows 交叉编译）

---

### 2. cjfmt（Cangjie Formatter）

**功能**：代码自动格式化工具

**主要特性**：
- 基于 Cangjie 编程规范
- 自动代码格式化
- 保持代码风格一致性
- 支持配置文件自定义规则

**支持的平台**：
- native（原生平台）
- windows-x86_64（Windows 交叉编译）
- ohos-x86_64（OpenHarmony x86_64）
- ohos-aarch64（OpenHarmony ARM64）

**技术栈**：C++17, CMake

---

### 3. cangjie-language-server（LSP）

**功能**：语言服务器

**主要特性**：
- 为 IDE（如 DevEco Studio）提供语言服务
- 代码补全和智能提示
- 符号定义和引用跳转
- 重命名和重构
- 语法和语义诊断
- 文档悬停
- 跨语言支持（Cangjie ↔ ArkTS）

**技术栈**：C++17, CMake
**第三方依赖**：
- flatbuffers（序列化/反序列化）
- JSON for Modern C++（JSON 解析）
- SQLite（索引存储）

---

### 4. cjlint（Cangjie Lint Tool）

**功能**：静态代码检查工具

**主要特性**：
- 基于 Cangjie 语言编码规范
- 检测违反编码规范的问题
- 发现潜在的安全漏洞
- 帮助开发者编写合规的 Cangjie 代码
- 支持自定义规则配置

**技术栈**：C++17, CMake
**第三方依赖**：JSON for Modern C++

---

### 5. cjcov（Cangjie Coverage Tool）

**功能**：代码覆盖率工具

**主要特性**：
- 测试覆盖率分析
- 帮助开发者提高测试完整性
- 生成覆盖率报告
- 支持多种覆盖率指标（行覆盖率、分支覆盖率等）

**支持的平台**：
- native（原生平台）
- windows-x86_64（Windows 交叉编译）

**技术栈**：Cangjie

---

### 6. cjtrace-recover（Exception Stack Trace Recovery）

**功能**：异常堆栈恢复工具

**主要特性**：
- 恢复混淆后的异常堆栈信息
- 用于问题定位和根因分析
- 支持符号化和去混淆
- 帮助开发者快速定位错误

**技术栈**：CMake + Cangjie + C++ 混合
**第三方依赖**：demangler 库

---

### 7. hyperlangExtension（HLE）

**功能**：跨语言互操作代码生成工具

**主要特性**：
- 生成 ArkTS 调用 Cangjie 的桥接代码
- 生成 Cangjie 调用 ArkTS 的桥接代码
- 自动处理类型转换和调用约定
- 生成 OHOS 构建（BUILD.gn）配置
- 提升互操作易用性

**输入**：ArkTS 接口声明文件（.d.ts 或 .d.ets）
**输出**：
- 生成的 Cangjie 互操作代码（.cj 文件）
- OHOS 构建（BUILD.gn）配置
- ArkTS 文件信息 JSON

**技术栈**：Cangjie

---

## 在 OpenHarmony 中的作用和定位

### 开发工具链定位

cangjie_tools 在 OpenHarmony 中的定位是 **Cangjie 语言的开发工具链**，为开发者提供完整的开发、调试、测试支持。

### 核心作用

1. **语言支持**：为 OpenHarmony 系统提供 Cangjie 语言开发支持
2. **IDE 集成**：通过 LSP 深度集成 DevEco Studio，提供智能开发体验
3. **跨语言互操作**：通过 HLE 工具实现 Cangjie 与 ArkTS 的无缝互操作
4. **代码质量保障**：通过 cjfmt、cjlint 等工具保障代码质量
5. **测试支持**：通过 cjcov 工具支持测试覆盖率分析

### 使用场景

| 使用场景 | 涉及工具 |
|---------|----------|
| Cangjie 项目开发 | cjpm, cjfmt, cjlint, LSP |
| DevEco Studio 集成 | LSP, HLE |
| Cangjie 与 ArkTS 互操作 | HLE |
| 代码质量检查 | cjfmt, cjlint |
| 测试和调试 | cjcov, cjtrace-recover |

### 与 OpenHarmony 的集成点

1. **DevEco Studio**：LSP 深度集成，提供智能提示、导航、诊断等 IDE 功能
2. **ArkTS 互操作**：HLE 工具生成 Cangjie 调用 ArkTS 的桥接代码
3. **OHOS 目标支持**：cjfmt 支持 `ohos-x86_64` 和 `ohos-aarch64` 编译目标
4. **条件编译**：支持 `@When[os == "ohos"]` 语法
5. **包命名空间**：支持 `ohos.*` 包命名空间

---

## 上游相关仓库

| 仓库 | 说明 |
|------|------|
| [cangjie_compiler](https://gitcode.com/Cangjie/cangjie_compiler) | Cangjie 编译器 |
| [cangjie_stdx](https://gitcode.com/Cangjie/cangjie_stdx) | Cangjie 标准库 |
| [cangjie_build](https://gitcode.com/Cangjie/cangjie_build) | Cangjie 构建系统 |
| [cangjie_test](https://gitcode.com/Cangjie/cangjie_test) | Cangjie 测试框架 |
| [cangjie_docs](https://gitcode.com/Cangjie/cangjie_docs) | Cangjie 文档 |

---

## 参考资源

### 官方资源

- **官方网站**：https://cangjie-lang.cn/
- **文档中心**：https://cangjie-lang.cn/pages/documentation
- **GitHub/GitCode**：https://gitcode.com/Cangjie

### OpenHarmony 资源

- **OpenHarmony 官方文档**：https://docs.openharmony.cn/
- **DevEco Studio 文档**：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/
- **Cangjie 开发指南**：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/cangjie-get-started-V5

---

## 版本信息

### 当前版本

- **上游版本**：1.1.0-alpha.69
- **OpenHarmony 版本**：6.1
- **同步日期**：2026-02-05（基于 git 历史推断）

### 版本说明

该版本为 Alpha 版本，API 可能不稳定，仅供参考和学习使用。
