# Resource Management 组件 Wiki - 文档导航

## 新人阅读顺序（推荐）

如果你是初次接触 OpenHarmony 资源管理组件，建议按以下顺序阅读：

1. **[概述](01_Overview.md)** - 了解组件定位、边界、核心能力
2. **[目录结构](02_DirectoryStructure.md)** - 理解代码组织方式和模块职责
3. **[架构设计](03_Architecture.md)** - 掌握组件架构、数据流和线程模型
4. **[N-API 接口](04_NAPI.md)** - 学习 JavaScript API 使用方法
5. **[常见问题](09_FAQ.md)** - 了解构建、运行、调试常见问题

## 开发者参考

### 核心文档

| 文档 | 内容 | 阅读场景 |
|------|------|----------|
| [概述](01_Overview.md) | 项目定位、边界、核心能力、运行环境、关键概念 | 初次接触组件 |
| [目录结构](02_DirectoryStructure.md) | 目录结构与模块职责（不含测试） | 理解代码组织 |
| [架构设计](03_Architecture.md) | 组件图、数据流、线程模型、关键时序 | 理解架构设计 |
| [N-API 接口](04_NAPI.md) | JS API 清单、参数、返回值、错误码 | 开发 JS/ArkUI 应用 |
| [内部 API](05_InnerAPI.md) | 模块接口、依赖方向、稳定性、可替换点 | 内部开发/维护 |

### 构建相关

| 文档 | 内容 | 阅读场景 |
|------|------|----------|
| [GN Targets](06_GNTargets.md) | targets 列表、类型、依赖、产物、开关 | 修改构建系统 |
| [编译产物](07_BuildArtifacts.md) | .so/.a/.hap/可执行文件、安装路径、运行时加载关系 | 理解产物部署 |

### 安全与问题排查

| 文档 | 内容 | 阅读场景 |
|------|------|----------|
| [安全风险评审](08_Security.md) | 攻击面、信任边界、可被利用点、修复建议 | 安全审计 |
| [常见问题](09_FAQ.md) | 构建、运行、调试问题与定位路径 | 问题排查 |

### 附录

| 文档 | 内容 | 阅读场景 |
|------|------|----------|
| [附录 - 调用链](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） | 深入理解执行流程 |
| [附录 - 配置选项](appendix/Config_Flags.md) | 关键宏/feature flags | 配置自定义构建 |

## 按主题浏览

### JavaScript 应用开发
- [N-API 接口](04_NAPI.md) - 65 个导出方法的完整文档
- [常见问题](09_FAQ.md) - JS API 使用常见问题

### C/C++ 原生开发
- [内部 API](05_InnerAPI.md) - C++ 内部接口
- [目录结构](02_DirectoryStructure.md) - 核心框架实现

### 系统开发与维护
- [架构设计](03_Architecture.md) - 组件架构和设计
- [GN Targets](06_GNTargets.md) - 构建系统配置
- [编译产物](07_BuildArtifacts.md) - 产物和部署

### 安全审计
- [安全风险评审](08_Security.md) - 攻击面分析和可被利用点

### 快速查找

**想了解...** → **查看...**

- 组件是什么、做什么 → [概述](01_Overview.md)
- 代码在哪里、如何组织 → [目录结构](02_DirectoryStructure.md)
- 如何使用 JS API → [N-API 接口](04_NAPI.md)
- 构建产物是什么 → [编译产物](07_BuildArtifacts.md)
- 有哪些安全风险 → [安全风险评审](08_Security.md)
- 遇到问题如何定位 → [常见问题](09_FAQ.md)
- 如何修改构建系统 → [GN Targets](06_GNTargets.md)

## 术语表

| 术语 | 说明 |
|------|------|
| N-API | Node-API，OpenHarmony 使用的 JavaScript Native API 框架 |
| ANI | ArkTS Native Interface，ArkTS 与 Native 代码的桥接 |
| FFI | Foreign Function Interface，Cangjie 语言的 FFI 绑定 |
| HAP | Harmony Ability Package，OpenHarmony 应用包格式 |
| ResourceManager | 资源管理器，资源管理组件的核心类 |
| RawFile | 原始文件，未编译的资源文件 |
| Bundle | 应用包，OpenHarmony 应用的基本单位 |
| SystemAbility | 系统能力，OpenHarmony 的跨进程服务机制（本组件未使用） |

## 相关资源

- **官方文档**: [OpenHarmony 资源管理文档](https://docs.openharmony.cn/)
- **代码仓库**: [global_resource_management](https://gitee.com/openharmony/global_resource_management)
- **子系统**: 全球化子系统 (global)
- **相关组件**:
  - global_i18n_standard - 国际化组件
  - bundle_framework - Bundle 框架

## 文档维护

本文档由自动化工具生成和更新。如发现错误或遗漏，请：

1. 检查代码库是否已更新
2. 提交 Issue 或 PR 到代码仓库
3. 确保所有修改都有代码证据支持

---

**最后更新**: 2026-02-06
