# OpenHarmony Sensor 子系统 Wiki 导航

> **最后更新**: 2026-02-06

---

## 新人阅读路线（推荐顺序）

如果你刚接触 OpenHarmony Sensor 子系统，建议按以下顺序阅读：

### 第一阶段：快速入门

1. **[README](README.md)** - 文档范围和更新说明
2. **[00_Overview](00_Overview.md)** - 项目定位、边界、核心能力、运行环境、关键概念
3. **[01_Directory_Structure](01_Directory_Structure.md)** - 目录结构与模块职责

### 第二阶段：理解架构

4. **[02_Architecture](02_Architecture.md)** - 组件图、数据流、线程模型、关键时序
5. **[04_Internal_API](04_Internal_API.md)** - 内部模块接口、依赖方向、稳定性、可替换点

### 第三阶段：使用 API

6. **[03_NAPI_Reference](03_NAPI_Reference.md)** - 对外 N-API 完整参考（JS API）

### 第四阶段：构建和部署

7. **[05_GN_Targets](05_GN_Targets.md)** - GN 目标梳理、targets 列表、类型、依赖、产物、开关
8. **[06_Build_Artifacts](06_Build_Artifacts.md)** - 编译产物（.so/.a/.hap/可执行文件）、安装路径、运行时加载关系
9. **[08_Troubleshooting](08_Troubleshooting.md)** - 常见构建/运行/调试问题与定位路径

### 第五阶段：安全审计

10. **[07_Security_Audit](07_Security_Audit.md)** - 攻击面、信任边界、可被利用点、修复建议

---

## 按角色查找文档

### 应用开发者

- **[03_NAPI_Reference](03_NAPI_Reference.md)** - 学习如何使用传感器 JS API
- **[08_Troubleshooting](08_Troubleshooting.md)** - 解决常见问题

### 系统开发者

- **[00_Overview](00_Overview.md)** - 了解系统整体设计
- **[02_Architecture](02_Architecture.md)** - 理解组件交互和数据流
- **[04_Internal_API](04_Internal_API.md)** - 查看内部 API 定义
- **[05_GN_Targets](05_GN_Targets.md)** - 了解构建配置
- **[06_Build_Artifacts](06_Build_Artifacts.md)** - 了解产物和依赖

### 安全审计人员

- **[07_Security_Audit](07_Security_Audit.md)** - 安全风险分析和漏洞清单
- **[02_Architecture](02_Architecture.md)** - 理解信任边界和数据流
- **[04_Internal_API](04_Internal_API.md)** - 分析内部接口安全性

### 平台集成者

- **[00_Overview](00_Overview.md)** - 了解系统定位和边界
- **[01_Directory_Structure](01_Directory_Structure.md)** - 熟悉代码组织
- **[02_Architecture](02_Architecture.md)** - 理解系统架构
- **[05_GN_Targets](05_GN_Targets.md)** - 了解构建系统
- **[06_Build_Artifacts](06_Build_Artifacts.md)** - 了解产物和依赖关系

---

## 附录

### 附录文档（可选）

- **[appendix/Callgraphs](appendix/Callgraphs.md)** - 关键调用链（入口→核心逻辑）
- **[appendix/Config_Flags](appendix/Config_Flags.md)** - 关键宏/feature flags

---

## 文档状态

| 文档 | 状态 | 完成度 |
|------|--------|---------|
| README.md | ✅ 完成 | 100% |
| SUMMARY.md | ✅ 完成 | 100% |
| 00_Overview.md | ✅ 完成 | 90% |
| 01_Directory_Structure.md | ✅ 完成 | 80% |
| 02_Architecture.md | ✅ 完成 | 85% |
| 03_NAPI_Reference.md | ✅ 完成 | 80% |
| 04_Internal_API.md | ✅ 完成 | 75% |
| 05_GN_Targets.md | ✅ 完成 | 90% |
| 06_Build_Artifacts.md | ✅ 完成 | 85% |
| 07_Security_Audit.md | ✅ 完成 | 90% |
| 08_Troubleshooting.md | ✅ 完成 | 70% |

---

## 图例说明

- ✅ 完成 - 文档已生成且内容完整
- 📝 草稿 - 文档已创建，但内容不完整
- 🚧 进行中 - 正在编写中
- 📋 计划中 - 已规划但未开始

---

## 版本历史

- **v1.0** (2026-02-06) - 初始版本，创建 Wiki 骨架
