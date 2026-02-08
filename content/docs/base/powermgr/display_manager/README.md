# display_manager Wiki 文档

> 本 Wiki 为 OpenHarmony display_manager 模块的完整技术文档，覆盖架构、API、构建、安全等各方面。

## 文档说明

### 文档范围

本 Wiki 文档涵盖 `base/powermgr/display_manager` 模块的以下内容：
- 项目定位与核心能力
- 目录结构与模块职责
- 架构设计（组件图、数据流、线程模型）
- 对外 N-API（JS API 接口）
- 内部 API（模块接口、依赖关系）
- GN 构建系统（Targets、编译产物）
- 安全风险评审（攻击面、可被利用点、修复建议）
- 常见问题与定位

### 排除范围

- **测试代码**：所有 `test/` 目录内容均不作为业务证据引用
- **第三方依赖**：不涉及第三方库的源码分析

### 更新方式

本 Wiki 基于代码证据自动生成。更新代码时应同步更新相关文档：
1. 修改 N-API 时更新 `04_NAPI_Interface.md`
2. 修改 IPC 接口时更新 `05_Internal_API.md`
3. 修改 GN 配置时更新 `06_GN_Targets.md`
4. 发现新的安全风险时更新 `08_Security_Analysis.md`

### 生成时间

- **生成日期**：2026-02-06
- **代码版本**：基于当前仓库状态
- **文档版本**：v1.0

---

## 文档导航

本 Wiki 的完整导航与阅读顺序请参见 **[SUMMARY.md](SUMMARY.md)**。

### 新人推荐阅读顺序

1. [00_Overview.md](00_Overview.md) - 项目概览
2. [01_Project_Scope.md](01_Project_Scope.md) - 项目定位与边界
3. [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
4. [03_Architecture.md](03_Architecture.md) - 架构设计
5. [04_NAPI_Interface.md](04_NAPI_Interface.md) - 对外 N-API
6. [05_Internal_API.md](05_Internal_API.md) - 内部 API
7. [06_GN_Targets.md](06_GN_Targets.md) - GN 构建系统
8. [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物
9. [08_Security_Analysis.md](08_Security_Analysis.md) - 安全风险评审
10. [09_Troubleshooting.md](09_Troubleshooting.md) - 常见问题

### 附录文档

- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置标志

---

## 文档约定

### 证据引用

所有关键技术结论均需在代码中找到直接证据：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

若无法确认必须标注 `TODO(需确认)` 并说明缺少的证据。

### 术语统一

- **state_manager** - 显示状态管理模块
- **brightness_manager** - 亮度管理模块
- **SA** - System Ability（系统能力）
- **IPC** - 进程间通信
- **N-API** - Native API（传统 JS 绑定）
- **ANI** - Ark Native Interface（新版本 ETS 绑定）
- **ZIDL** - Zero-copy IDL（OpenHarmony 接口定义语言）

### 代码引用格式

- **头文件**：`path/to/header.h`
- **源文件**：`path/to/source.cpp`
- **特定行**：`path/to/source.cpp:123`
- **BUILD.gn**：`path/to/BUILD.gn`

---

## 维护者

本文档由代码分析自动生成，维护时请遵循以下原则：
1. 基于代码事实，不凭空猜测
2. 保持证据链完整可追溯
3. 统一术语和命名
4. 及时更新以保持与代码同步
