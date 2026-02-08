# TypeScript 原始库概述

## 1.1 库基本信息

| 项目 | 信息 |
|------|------|
| **库名称** | TypeScript |
| **当前版本** | 4.9.5 |
| **许可证** | Apache-2.0 |
| **上游仓库** | https://github.com/Microsoft/TypeScript.git |
| **维护者** | Microsoft Corporation |
| **首次发布** | 2012年10月 |
| **语言** | TypeScript（编译为 JavaScript） |

## 1.2 原始功能简介

TypeScript 是由微软开发并维护的开源编程语言，它是 JavaScript 的超集，为 JavaScript 添加了可选的静态类型系统和面向对象编程特性。TypeScript 旨在解决大规模 JavaScript 应用开发中的类型安全问题，在编译时捕获潜在错误，提升代码质量和可维护性。

### 核心特性

TypeScript 的原始设计目标包括以下几个方面：

**类型系统增强**是 TypeScript 最重要的特性。通过添加静态类型，开发者可以在编译阶段发现类型不匹配、属性访问错误等问题。TypeScript 支持接口、泛型、枚举、联合类型、交叉类型等丰富的类型构造，使得类型表达能力强且灵活。类型注解采用可选方式，不强制要求标注每个变量，这使得从 JavaScript 迁移到 TypeScript 非常平滑。

**编译为目标代码**是 TypeScript 的基础功能。TypeScript 编译器（tsc）将 .ts 和 .tsx 文件编译为符合 ECMAScript 标准的 JavaScript 代码。编译产物可以在任何支持 JavaScript 的环境中运行，包括浏览器、Node.js 服务器、移动端等。编译过程支持多种目标版本（ES3 到 ESNext），并可通过配置定制输出代码的风格和模块格式。

**开发工具支持**使得 TypeScript 成为大型项目的首选。类型信息被 IDE 充分利用，提供智能代码补全、跳转定义、重构支持、实时错误检查等功能。TypeScript 编译器本身也作为语言服务运行在后台，为编辑器提供这些能力。

**面向对象特性**为 JavaScript 添加了传统面向对象语言的概念。TypeScript 支持类（class）、模块（module）、命名空间（namespace）、访问修饰符（public、private、protected）等特性，使得代码组织更加清晰，特别适合团队协作开发大规模应用。

## 1.3 上游生态系统

### 社区活跃度

TypeScript 拥有非常活跃的开源社区，是 GitHub 上最受欢迎的语言之一。其生态系统包括：

**官方工具链**包括 TypeScript 编译器（tsc）、语言服务器协议（LSP）实现、TSServer 语言服务、类型定义管理工具等。这些工具构成了完整的 TypeScript 开发环境。

**第三方集成**几乎覆盖所有主流编辑器和 IDE。VS Code 本身就是 TypeScript 开发的，其 TypeScript 支持最为完善。WebStorm、Vim、Emacs、Sublime Text 等都有良好的 TypeScript 插件支持。

**类型定义库**（DefinitelyTyped）提供了超过 8000 个 JavaScript 库的 TypeScript 类型定义，使得使用第三方库时也能获得完整的类型检查和代码补全支持。

**框架支持**方面，主流前端框架（React、Vue、Angular）都优先或原生支持 TypeScript。React 使用 JSX 语法扩展，TypeScript 对 JSX 有原生支持（.tsx 文件）。

## 1.4 在 OpenHarmony 中的定位

### OH 与上游的差异

在 OpenHarmony 生态中，TypeScript 被赋予了新的使命：从通用的 JavaScript 超集语言转变为 **ETS（Extensible TypeScript）语言编译器**。这一转变使其成为 OpenHarmony  ArkUI 声明式 UI 开发范式的核心支撑技术。

### 战略重要性

TypeScript 在 OH 技术栈中处于基础设施层，其重要性体现在以下方面：

**开发范式支撑**：ArkTS/eTS 是一种基于 TypeScript 的声明式 UI 开发范式。开发者使用类似 SwiftUI、Jetpack Compose 的语法编写 UI，TypeScript 编译器负责将这些声明式代码编译为可执行代码。没有 TypeScript，就无法实现声明式 UI 开发。

**工具链基础**：DevEco Studio 的代码编辑、类型检查、自动补全等功能都依赖 TypeScript 语言服务。es2panda、ace_ets2bundle 等编译工具也以 TypeScript 为基础。

**跨平台桥梁**：TypeScript 作为中间表示层，连接了应用层的高级语法和运行时层的执行逻辑，使得 OH 应用能够一次编写，多端运行。

### 适配范围

OH 对 TypeScript 的适配主要集中在以下领域：

| 适配领域 | 说明 |
|---------|------|
| **语法扩展** | Struct 组件语法、装饰器语法 |
| **类型系统** | 组件类型、状态类型 |
| **代码生成** | 面向 Panda VM 的 IR 生成 |
| **IDE 服务** | 组件导航、补全优化 |
| **性能优化** | eTS 编译性能专项优化 |

## 1.5 版本对应关系

### OH 版本与 TypeScript 版本

| OH 版本 | TypeScript 版本 | 适配重点 |
|--------|----------------|---------|
| OpenHarmony 3.x | 4.9.5 | eTS 语言完整支持 |

### 版本说明

当前 OH 使用的 TypeScript 4.9.5 版本于 2022 年 11 月发布。上游 TypeScript 已演进到 5.x 系列，带来了一些新的语言特性和性能改进。OH 是否升级以及何时升级需要综合考虑 eTS 兼容性、工具链依赖、性能收益等因素。

### 升级考虑因素

升级 TypeScript 上游版本时，OH 需要考虑以下因素：

**eTS 兼容性**：OH 对 TypeScript 的修改（特别是类型检查器和代码生成部分）与特定版本强耦合。升级前需要验证所有 eTS 特性在新版本中是否正常工作。

**工具链兼容性**：es2panda、linter 等工具依赖 TypeScript 内部结构，这些依赖可能在版本升级时失效。

**破坏性变更**：TypeScript 5.0 引入了一些破坏性变更，需要评估对 OH 的影响。

## 1.6 许可证信息

### 许可证类型

TypeScript 使用 **Apache License 2.0**，这是一个宽松的 permissive 许可证，允许在商业和开源项目中自由使用、修改和分发。

### 许可证义务

使用 TypeScript 需要遵守以下义务：

**版权声明**：分发时需要保留原始版权声明和许可证文本。OH 通过 `collect_notice` 机制自动收集并分发许可证文件。

**衍生作品**：对 TypeScript 进行修改后分发时，需要在修改文件中注明修改内容和日期。

**无担保**：许可证明确声明软件按「现状」提供，不提供任何明示或暗示的担保。

### OH 许可证管理

OH 在构建过程中自动收集 TypeScript 的许可证信息，生成 `typescript.txt` 文件并分发到 SDK 的相应目录。用户在使用 OH SDK 时会自动获得这些许可证信息。
