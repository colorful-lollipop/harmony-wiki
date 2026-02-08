# 文档导航

> 新人阅读顺序推荐：按顺序阅读 00 → 07，可按需跳转到特定章节。

## 快速入门

- [README](README.md) - 文档说明与使用指南
- [00_Overview.md](00_Overview.md) - 项目定位与核心能力

## 架构与设计

- [01_Architecture.md](01_Architecture.md) - 系统架构、数据流、MVP 分层
- [02_Module_Structure.md](02_Module_Structure.md) - 模块职责与目录结构

## 接口规范

- [03_External_API.md](03_External_API.md) - **外部调用接口**（重点）
- [04_Internal_API.md](04_Internal_API.md) - HAR 模块内部 API

## 构建与部署

- [05_Build_System.md](05_Build_System.md) - hvigor 构建配置
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物与安装路径

## 安全与运维

- [07_Security_Assessment.md](07_Security_Assessment.md) - 安全风险评审
- [08_Troubleshooting.md](08_Troubleshooting.md) - 问题排查指南

## 附录

- [附录：调用链图谱](appendix/Callgraphs.md) - 关键调用链路
- [附录：配置参数表](appendix/Config_Flags.md) - 关键配置项

---

## 新人阅读路线图

```
┌─────────────────────────────────────────────────────────────┐
│                    新人入门路线图                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ① 了解项目                                              │
│   README.md → 00_Overview.md                                │
│           ↓                                                │
│   ② 理解架构                                              │
│   01_Architecture.md (MVP 分层、数据流)                    │
│           ↓                                                │
│   ③ 掌握模块                                              │
│   02_Module_Structure.md (7 个模块职责)                    │
│           ↓                                                │
│   ④ 学习接口 ← 最重要！                                     │
│   03_External_API.md (外部调用入口)                         │
│           ↓                                                │
│   ⑤ 构建部署                                              │
│   05_Build_System.md → 06_Build_Artifacts.md               │
│           ↓                                                │
│   ⑥ 安全运维                                              │
│   07_Security_Assessment.md → 08_Troubleshooting.md       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 按角色导航

| 角色 | 推荐阅读章节 |
|------|--------------|
| **UI 开发者** | 01_Architecture, 02_Module_Structure, 04_Internal_API |
| **系统集成** | 03_External_API, 01_Architecture |
| **构建/运维** | 05_Build_System, 06_Build_Artifacts, 08_Troubleshooting |
| **安全审计** | 07_Security_Assessment, 03_External_API, 权限部分 |

## 版本信息

| 项目 | 版本 |
|------|------|
| Photos | 3.0 |
| 文档版本 | 1.0 |
| 最后更新 | 2026-02-05 |
