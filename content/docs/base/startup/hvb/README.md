# HVB Wiki

## 文档覆盖范围

本文档覆盖 OpenHarmony Verified Boot (HVB) 组件的技术架构、API 接口、构建系统和安全分析。

### 已覆盖内容
- ✅ 项目概览与定位
- ✅ 目录结构与模块职责
- ✅ 架构设计与数据流
- ✅ 公共 C API 接口（无 N-API）
- ✅ 内部架构与模块依赖
- ✅ GN 构建目标与编译产物
- ✅ 安全风险分析与攻击面
- ✅ 常见问题与调试指南

### 未覆盖内容
- ❌ N-API 文档（项目为纯 C 库，无 JS 绑定）
- ❌ IPC 机制（项目无 IPC，Bootloader 直接调用）
- ❌ 权限验证（权限检查在 Bootloader 层面）
- ❌ 测试用例（按需求排除测试内容）

### 更新方式

本 Wiki 生成于 2026-02-06，基于代码库 `/base/startup/hvb` 的分析。

**随代码更新文档**：
1. 修改源码后，更新对应的 API 清单和架构图
2. 修改 GN 构建配置后，更新 `wiki/06_GN_Targets.md` 和 `wiki/07_Build_Artifacts.md`
3. 修复安全问题后，更新 `wiki/08_Security_Analysis.md`
4. 运行 `wiki/_work/PLAN.md` 检查完整性

**文档版本控制**：
- 本文档通过 Git 管理版本
- 建议使用 Git 提交时一并提交 Wiki 更新

---

## 快速导航

- [全站导航与阅读顺序](wiki/SUMMARY.md) - 推荐新人阅读路线
- [项目概览](wiki/00_Overview.md) - HVB 是什么，解决什么问题
- [项目定位与边界](wiki/01_Project_Scope.md) - 核心能力、运行环境、关键概念
- [目录结构](wiki/02_Directory_Structure.md) - 模块划分与职责
- [架构说明](wiki/03_Architecture.md) - 组件图、数据流、线程模型
- [对外 API](wiki/04_Public_API.md) - C 接口清单、参数、错误码
- [内部 API](wiki/05_Internal_API.md) - 模块接口、依赖方向
- [GN Targets](wiki/06_GN_Targets.md) - 构建目标、依赖、产物
- [编译产物](wiki/07_Build_Artifacts.md) - .a/.hap 文件、安装路径
- [安全分析](wiki/08_Security_Analysis.md) - 攻击面、信任边界、风险评估
- [常见问题](wiki/09_FAQ.md) - 构建、运行、调试问题

### 附录
- [调用链分析](wiki/appendix/Callgraphs.md) - 关键调用链（待补充）
- [配置选项](wiki/appendix/Config_Flags.md) - 关键宏/feature flags（待补充）

---

## 贡献指南

如需更新或修正文档：

1. 确保所有关键结论都有代码证据（文件路径、行号、符号名）
2. 保持术语统一（verity/hashtree/RVT 等）
3. 中文为主，关键技术术语保留英文
4. 代码证据格式：`path:line` 或 `libhvb/include/hvb.h:41-53`
5. 不得引用测试目录内容作为业务证据

---

## 联系方式

- 项目路径: `base/startup/hvb`
- 子系统: `startup`
- 部件名: `hvb`
- 许可证: Apache License 2.0

---

*最后更新: 2026-02-06*
