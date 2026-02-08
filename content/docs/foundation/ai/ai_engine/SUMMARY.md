# AI Engine Wiki - 全站导航

## 阅读指南

### 新人入门路线

```
1. 00_Overview.md         → 项目概览，了解 AI Engine 是什么
2. 01_Directory_Structure.md → 目录结构，快速定位代码
3. 02_Architecture.md      → 架构设计，理解整体框架
4. 04_Interface.md         → SDK 接口，快速上手开发
5. 05_Usage.md            → 使用指南，快速开始开发
6. 08_FAQ.md              → 常见问题，解决入门疑惑
```

### 安全研究路线

```
1. 00_Overview.md         → 理解系统边界和能力
2. 06_AttackSurface.md     → 识别所有外部输入和敏感操作
3. 02_Architecture.md      → 追踪数据流和信任边界
4. 07_SecurityReview.md    → 深度风险分析（含证据和利用路径）
5. 03_CodeMap.md           → 快速定位代码位置
6. 09_Artifacts.md         → 构建产物和部署机制
```

### 开发者深入路线

```
1. 04_Interface.md         → 对外接口文档（SDK API）
2. 10_Internals.md        → 内部实现细节
3. 05_Build.md             → 构建系统
4. 09_Artifacts.md         → 编译产物与部署
```

---

## 核心文档

| 章节 | 文档 | 说明 |
|------|------|------|
| 0 | [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| 2 | [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| 3 | [03_CodeMap.md](./03_CodeMap.md) | 目录结构与模块职责、代码导航图 |
| 4 | [04_Interface.md](./04_Interface.md) | 对外接口（SDK）清单、参数说明、错误码 |
| 5 | [05_Usage.md](./05_Usage.md) | 使用指南、SDK 示例、插件开发教程 |
| 6 | [06_AttackSurface.md](./06_AttackSurface.md) | 攻击面分析、信任边界、敏感操作 |
| 7 | [07_SecurityReview.md](./07_SecurityReview.md) | 深度风险评估（12 个风险点、证据、修复建议） |
| 8 | [08_Build.md](./08_Build.md) | GN targets、依赖配置、编译选项 |
| 9 | [09_Artifacts.md](./09_Artifacts.md) | 产物清单、安装路径、加载关系 |
| 10 | [10_Internals.md](./10_Internals.md) | 内部实现细节、模块接口、资源生命周期 |

---

## 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 快速索引

### 按功能分类

| 功能 | 相关文档 |
|------|---------|
| 开发新插件 | 02_Architecture, 04_Interface, 05_Usage |
| 开发新 SDK | 02_Architecture, 04_Interface, 05_Usage |
| 定制构建 | 08_Build, 09_Artifacts |
| 安全审计 | 06_AttackSurface, 07_SecurityReview, 03_CodeMap |
| 问题定位 | 08_FAQ, 02_Architecture (线程模型) |

### 按模块分类

| 模块 | 相关文档 |
|------|---------|
| 客户端 SDK | 04_Interface (SDK 章节), 10_Internals (客户端 API) |
| 服务端引擎 | 02_Architecture, 10_Internals (服务端 API) |
| 插件系统 | 02_Architecture, 04_Interface, 05_Usage (插件接口和示例） |
| 通信适配 | 02_Architecture, 10_Internals (IPC), 03_CodeMap (代码定位） |
| 构建系统 | 08_Build, 09_Artifacts |

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本，完整 Wiki 骨架 |
