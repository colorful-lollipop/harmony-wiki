# 文档导航

本文档为 OpenHarmony Lite Battery Manager 工程的完整 Wiki，提供新人可快速完整理解项目的多篇 Markdown 文档。

## 文档目录

### 快速入门

| 文档 | 描述 | 阅读时间 |
|------|------|----------|
| [README](README.md) | Wiki 覆盖范围、更新方式、生成时间 | 2 分钟 |
| [概览](00_Overview.md) | 项目定位、核心能力、运行环境 | 5 分钟 |

### 架构与结构

| 文档 | 描述 | 阅读时间 |
|------|------|----------|
| [目录结构](01_Directory_Structure.md) | 模块职责、代码组织 | 5 分钟 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型 | 10 分钟 |

### 接口文档

| 文档 | 描述 | 阅读时间 |
|------|------|----------|
| [N-API 接口](03_N_API.md) | JS API 清单、绑定、参数校验 | 15 分钟 |
| [内部 API](04_Inner_API.md) | 模块接口、依赖方向、稳定性 | 10 分钟 |

### 构建与部署

| 文档 | 描述 | 阅读时间 |
|------|------|----------|
| [GN 构建](05_GN_Build.md) | Targets 列表、编译产物、安装路径 | 10 分钟 |
| [攻击面分析](05_AttackSurface.md) | 外部输入点、信任边界、敏感操作 | 15 分钟 |
| [安全评审](06_Security_Review.md) | 风险点、证据链、修复建议 | 20 分钟 |

### 附录

| 文档 | 描述 | 阅读时间 |
|------|------|----------|
| [调用链图谱](appendix/Callgraphs.md) | 入口→核心逻辑调用链 | 10 分钟 |
| [配置开关](appendix/Config_Flags.md) | 关键宏、Feature Flags | 5 分钟 |

---

## 新人阅读顺序推荐

### 路径 A：快速上手（30 分钟）

```
README.md → 00_Overview.md → 03_N_API.md → 快速开始使用 API
```

### 路径 B：深入理解（60 分钟）

```
README.md → 00_Overview.md → 01_Directory_Structure.md 
→ 02_Architecture.md → 03_N_API.md → 05_GN_Build.md
```

### 路径 C：安全开发（45 分钟）

```
README.md → 00_Overview.md → 02_Architecture.md 
→ 06_Security_Review.md → 理解风险点
```

---

## 关键术语速查

| 术语 | 说明 | 相关文档 |
|------|------|----------|
| SOC | State of Charge，电池剩余电量百分比 | 03_N_API.md |
| SoH | State of Health，电池健康状态 | 03_N_API.md |
| SAMgr | Service Ability Manager，服务能力管理器 | 02_Architecture.md |
| N-API | Native API，C/C++ 到 JS 的绑定接口 | 03_N_API.md |
| IpcIo | IPC 输入输出参数包 | 02_Architecture.md |

---

## 代码证据索引

所有文档中的关键结论均可在以下代码位置找到证据：

### 服务层证据
- `services/include/battery_device.h:46-61`: 电池设备特征 API 定义
- `services/include/ibattery.h:26-59`: 电池接口结构体
- `services/src/battery_device.c:19-28`: 模拟电池数据

### 框架层证据
- `frameworks/native/include/battery_framework.h:31-39`: 获取 IUnknown 接口
- `frameworks/native/include/batterymgr_intf_define.h:26-33`: 接口宏定义

### JS 绑定证据
- `frameworks/js/builtin/src/battery_module.cpp:44-168`: JS API 实现
- `interfaces/kits/js/@system.battery.d.ts:169-212`: TypeScript 类型定义

---

## 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本，包含全部核心文档 |

---

## 贡献指南

### 如何更新文档

1. **修改代码后同步更新文档**
2. **新增 API 需在 `03_N_API.md` 添加清单表**
3. **架构变更需更新 `02_Architecture.md` 组件图**
4. **发现安全风险需在 `06_Security_Review.md` 添加条目**

### 文档规范

- 关键结论必须包含代码证据（路径 + 符号）
- N-API 文档必须包含完整的 API 清单表
- GN  文档必须包含 Targets列表和产物映射
- 安全文档必须包含攻击面清单和风险点
