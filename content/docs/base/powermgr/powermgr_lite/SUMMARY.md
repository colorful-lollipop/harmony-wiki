# Wiki 目录

> powermgr_lite 项目文档导航与阅读指南

---

## 📚 新人学习路线

**目标**: 在 30 分钟内理解项目定位，在 1 小时内理解基本架构

### 快速入门 (15 分钟)
1. [00_Overview](00_Overview.md) - 项目定位、核心能力、关键概念
2. [01_Project_Boundaries](01_Project_Boundaries.md) - 适用场景、能力边界、运行环境
3. [02_Directory_Structure.md#快速导航](02_Directory_Structure.md#快速导航) - 快速定位核心代码

### 架构理解 (45 分钟)
4. [03_Architecture](03_Architecture.md) - 三层架构、数据流、线程模型、mini/small 差异
5. [05_Internal_API](05_Internal_API.md) - 模块接口、依赖方向、内部 API 契约
6. [06_GN_Targets.md#依赖图](06_GN_Targets.md#依赖图) - 构建依赖关系

### 应用开发 (30 分钟)
7. [04_NAPI_JS_API](04_NAPI_JS_API.md#c-api) - RunningLock C API 使用指南
8. [04_NAPI_JS_API.md#javascript-jsi-绑定](04_NAPI_JS_API.md#javascript-jsi-绑定) - Battery JS API 使用指南

### 构建与部署 (30 分钟)
9. [06_GN_Targets](06_GN_Targets.md) - GN 目标配置、Feature 开关
10. [07_Build_Artifacts](07_Build_Artifacts.md) - 编译产物、安装路径

### 问题定位 (15 分钟)
11. [09_Troubleshooting](09_Troubleshooting.md) - 常见构建/运行问题

---

## 🔒 安全研究路线

**目标**: 快速识别攻击面、信任边界、潜在漏洞点

### 攻击面识别 (15 分钟)
1. [05_AttackSurface](05_AttackSurface.md) - 外部输入清单、敏感操作清单、信任边界图

### 风险评估 (45 分钟)
2. [08_Security_Assessment](08_Security_Assessment.md) - 5 类安全风险分析（含证据、触发路径、修复建议）
3. [04_NAPI_JS_API.md#攻击面](04_NAPI_JS_API.md#攻击面) - API 参数类型、校验机制

### 深度分析 (30 分钟)
4. [03_Architecture.md#信任边界](03_Architecture.md#信任边界) - 安全域跨越点分析
5. [02_Directory_Structure.md#安全相关](02_Directory_Structure.md#安全相关) - 安全相关文件定位

### 构建与测试 (15 分钟)
6. [06_GN_Targets.md](06_GN_Targets.md) - 安全相关 Feature 开关
7. [09_Troubleshooting.md#安全相关问题](09_Troubleshooting.md#安全相关问题) - 安全问题排查

---

## 🎯 按角色导航

### 应用开发者
**入口**: [04_NAPI_JS_API](04_NAPI_JS_API.md)
**核心路径**: [00_Overview](00_Overview.md) → [04_NAPI_JS_API](04_NAPI_JS_API.md) → [09_Troubleshooting](09_Troubleshooting.md#api-调用问题)

### 模块开发者
**入口**: [03_Architecture](03_Architecture.md)
**核心路径**: [00_Overview](00_Overview.md) → [03_Architecture](03_Architecture.md) → [05_Internal_API](05_Internal_API.md) → [02_Directory_Structure](02_Directory_Structure.md)

### 构建工程师
**入口**: [06_GN_Targets](06_GN_Targets.md)
**核心路径**: [06_GN_Targets](06_GN_Targets.md) → [07_Build_Artifacts](07_Build_Artifacts.md) → [01_Project_Boundaries.md#feature-flags](01_Project_Boundaries.md#feature-flags)

### 安全审计员
**入口**: [08_Security_Assessment](08_Security_Assessment.md)
**核心路径**: [08_Security_Assessment](08_Security_Assessment.md) → [05_AttackSurface](05_AttackSurface.md) → [03_Architecture.md#信任边界](03_Architecture.md#信任边界) → [04_NAPI_JS_API.md#攻击面](04_NAPI_JS_API.md#攻击面)

---

## 📌 快速导航

### 新人 5 分钟快速理解
- [00_Overview](00_Overview.md#核心能力) - 一句话定义 + 能力边界
- [00_Overview.md#数据流示例](00_Overview.md#数据流示例) - 运行锁获取流程

### 安全 5 分钟快速上手
- [08_Security_Assessment.md#关键漏洞](08_Security_Assessment.md#关键漏洞) - 5 类风险快速浏览
- [08_Security_Assessment.md#攻击面清单](08_Security_Assessment.md#攻击面清单) - 5 个攻击入口点

### 常见问题
- [09_Troubleshooting](09_Troubleshooting.md) - 构建失败、运行时错误、调试技巧

---

## 文档结构

### 概览层
- [00_Overview](00_Overview.md) - 项目概述、核心能力、关键概念

### 项目层
- [01_Project_Boundaries](01_Project_Boundaries.md) - 项目定位、边界、运行环境
- [02_Directory_Structure](02_Directory_Structure.md) - 目录结构与模块职责

### 架构层
- [03_Architecture](03_Architecture.md) - 架构说明、组件图、数据流、线程模型、关键时序

### 接口层
- [04_NAPI_JS_API](04_NAPI_JS_API.md) - 对外 API: C API + JSI 绑定、导出符号、权限/参数/错误码
- [05_Internal_API](05_Internal_API.md) - 内部 API: 模块接口、依赖方向、稳定性、可替换点

### 构建层
- [06_GN_Targets](06_GN_Targets.md) - GN 目标梳理: targets 列表、类型、依赖、产物、开关
- [07_Build_Artifacts](07_Build_Artifacts.md) - 编译产物: .so/.a/.hap/可执行文件、安装路径、运行时加载关系

### 安全层
- [08_Security_Assessment](08_Security_Assessment.md) - 安全风险评审: 攻击面、信任边界、可被利用点、修复建议

### 支持层
- [09_Troubleshooting](09_Troubleshooting.md) - 常见构建/运行/调试问题与定位路径

---

## 按角色导航

### 应用开发者
- 入口: [04_NAPI_JS_API](04_NAPI_JS_API.md)
- 补充: [00_Overview](00_Overview.md), [01_Project_Boundaries](01_Project_Boundaries.md)
- 问题定位: [09_Troubleshooting](09_Troubleshooting.md#api-调用问题)

### 模块开发者
- 入口: [03_Architecture](03_Architecture.md)
- 核心: [05_Internal_API](05_Internal_API.md), [02_Directory_Structure](02_Directory_Structure.md)
- 接口: [04_NAPI_JS_API](04_NAPI_JS_API.md#内部接口映射)

### 构建工程师
- 入口: [06_GN_Targets](06_GN_Targets.md)
- 补充: [07_Build_Artifacts](07_Build_Artifacts.md)
- 配置: [01_Project_Boundaries.md#feature-flags](01_Project_Boundaries.md#feature-flags)

### 安全审计员
- 入口: [08_Security_Assessment](08_Security_Assessment.md)
- 背景: [03_Architecture.md#信任边界](03_Architecture.md#信任边界)
- 攻击面: [04_NAPI_JS_API.md#攻击面](04_NAPI_JS_API.md#攻击面)

---

## 相关跳转

### 从本页跳转
- [返回主 README](README.md)
- [更新工作笔记](../_work/NOTES.md)
- [查看工作计划](../_work/PLAN.md)

### 跨文档跳转
- [00_Overview](00_Overview.md) ← [01_Project_Boundaries](01_Project_Boundaries.md) → [02_Directory_Structure](02_Directory_Structure.md)
- [03_Architecture](03_Architecture.md) ← [04_NAPI_JS_API](04_NAPI_JS_API.md) → [05_Internal_API](05_Internal_API.md)
- [06_GN_Targets](06_GN_Targets.md) ← [07_Build_Artifacts](07_Build_Artifacts.md) → [08_Security_Assessment](08_Security_Assessment.md)

---

## 附录

### 附录目录
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链 (入口→核心逻辑)
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 关键宏/feature flags

---

## 文档维护

本文档最后更新: 2026-02-06
对应代码版本: powermgr_lite v3.1
维护者: Wiki 生成脚本

如发现文档错误,请参考 [README.md](README.md#贡献指南) 的贡献指南。
