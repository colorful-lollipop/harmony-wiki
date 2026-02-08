# Wiki 文档导航

**最后更新**: 2025-02-06

本文档集提供 `accesscontrol_cangjie_wrapper` 项目的完整技术文档，涵盖架构、API、构建、安全等方面。

---

## 📚 快速导航

### 🏠 项目概览
- [00_Overview.md](00_Overview.md) - 项目概述与快速入门
- [01_Positioning_Boundaries.md](01_Positioning_Boundaries.md) - 项目定位、边界与核心能力

### 📁 代码结构
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构与模块职责

### 🏗️ 架构设计
- [03_Architecture.md](03_Architecture.md) - 系统架构、组件图、数据流、线程模型

### 🔌 接口文档
- [04_Public_API.md](04_Public_API.md) - 对外 Cangjie API 清单与使用说明
- [05_Internal_API.md](05_Internal_API.md) - 内部模块接口、依赖关系、稳定性说明

### 🔨 构建系统
- [06_GN_Targets.md](06_GN_Targets.md) - GN 目标梳理、依赖关系、编译配置
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物、安装路径、运行时加载

### 🔒 安全评审
- [08_Security_Review.md](08_Security_Review.md) - 攻击面分析、安全风险、修复建议

### 🛠️ 故障排查
- [09_Troubleshooting.md](09_Troubleshooting.md) - 常见问题与定位路径

### 📎 附录（可选）
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 关键配置项与特性开关

---

## 👨‍💻 新人阅读路线

如果你是第一次接触本项目，建议按以下顺序阅读：

1. **[00_Overview.md](00_Overview.md)** - 快速了解项目全貌（5 分钟）
   - 项目简介
   - 核心功能
   - 依赖组件
   - 快速开始

2. **[01_Positioning_Boundaries.md](01_Positioning_Boundaries.md)** - 理解项目边界（10 分钟）
   - 项目定位
   - 核心能力与限制
   - 运行环境要求
   - 关键概念

3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 熟悉代码组织（10 分钟）
   - 目录树
   - 模块职责划分
   - 文件组织规范

4. **[03_Architecture.md](03_Architecture.md)** - 理解系统架构（20 分钟）
   - 组件依赖图
   - 数据流图
   - 调用时序
   - 线程模型

5. **[04_Public_API.md](04_Public_API.md)** - 学习对外 API（30 分钟）
   - API 清单表
   - 使用示例
   - 错误码说明
   - 调用示例

6. **[08_Security_Review.md](08_Security_Review.md)** - 了解安全注意事项（15 分钟）
   - 攻击面分析
   - 安全风险清单
   - 安全使用建议

**总阅读时间**: 约 90 分钟

---

## 🔧 开发者参考指南

如果你需要深入了解实现细节或进行二次开发，建议按以下顺序阅读：

1. **[06_GN_Targets.md](06_GN_Targets.md)** - 理解构建系统（15 分钟）
   - GN 目标清单
   - 依赖关系
   - 编译配置
   - 如何添加新模块

2. **[05_Internal_API.md](05_Internal_API.md)** - 深入内部实现（20 分钟）
   - 模块接口
   - 依赖方向
   - 稳定性分级
   - 扩展点说明

3. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 了解编译产物（10 分钟）
   - 产物清单
   - 安装路径
   - 运行时加载
   - 调试符号

4. **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 查看关键调用链（15 分钟）
   - checkAccessToken 调用链
   - requestPermissionsFromUser 调用链
   - FFI 调用路径

5. **[09_Troubleshooting.md](09_Troubleshooting.md)** - 排查常见问题（15 分钟）
   - 构建问题
   - 运行时问题
   - 调试技巧

**总阅读时间**: 约 75 分钟

---

## 📌 关键概念速查

| 概念 | 说明 | 文档链接 |
|------|------|----------|
| **TokenID** | 应用的 32 位唯一标识符，用于权限验证 | [01_Positioning_Boundaries.md](01_Positioning_Boundaries.md) |
| **GrantStatus** | 权限授权状态枚举（Granted/Denied） | [04_Public_API.md](04_Public_API.md) |
| **AtManager** | 权限管理主类，提供权限检查和请求接口 | [04_Public_API.md](04_Public_API.md) |
| **FFI** | Cangjie 与 C 语言的互操作接口 | [05_Internal_API.md](05_Internal_API.md) |
| **access_token** | 底层权限子系统，提供 C FFI 接口 | [03_Architecture.md](03_Architecture.md) |
| **UIAbilityContext** | Stage 模型的上下文，用于弹出权限请求对话框 | [04_Public_API.md](04_Public_API.md) |

---

## 🔍 按主题查找

### API 使用
- 如何检查应用是否拥有权限？ → [04_Public_API.md](04_Public_API.md) - checkAccessToken
- 如何向用户请求权限？ → [04_Public_API.md](04_Public_API.md) - requestPermissionsFromUser
- 错误码含义是什么？ → [04_Public_API.md](04_Public_API.md) - 错误码表

### 构建与编译
- 如何编译项目？ → [06_GN_Targets.md](06_GN_Targets.md)
- 编译产物是什么？ → [07_Build_Artifacts.md](07_Build_Artifacts.md)
- 如何添加新的 API？ → [06_GN_Targets.md](06_GN_Targets.md)

### 安全与风险
- 有哪些安全风险？ → [08_Security_Review.md](08_Security_Review.md)
- 如何安全地使用 TokenID？ → [08_Security_Review.md](08_Security_Review.md)
- 权限名称有哪些限制？ → [08_Security_Review.md](08_Security_Review.md)

### 故障排查
- 编译失败怎么办？ → [09_Troubleshooting.md](09_Troubleshooting.md)
- 运行时崩溃如何调试？ → [09_Troubleshooting.md](09_Troubleshooting.md)
- 权限检查返回异常？ → [09_Troubleshooting.md](09_Troubleshooting.md)

---

## 📝 文档更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-06 | 1.0 | 初始版本创建 |

---

## 🔗 外部参考

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [security_access_token 仓库](https://gitcode.com/openharmony/security_access_token)
- [arkcompiler_cangjie_ark_interop 仓库](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [ability_cangjie_wrapper 仓库](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper)
- [hiviewdfx_cangjie_wrapper 仓库](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper)

---

**提示**: 所有文档中的代码结论都包含代码证据（路径:行号或符号名），可通过 `wiki/_work/NOTES.md` 查看完整索引。
