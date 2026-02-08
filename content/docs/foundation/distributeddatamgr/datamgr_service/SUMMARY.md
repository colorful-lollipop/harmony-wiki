# 目录导航

本文档为 OpenHarmony 分布式数据管理服务 (DDS) 的 Wiki 导航页，面向**新人学习者**和**安全研究员**双受众设计。

---

## 双路线阅读指南

### 📚 新人学习路线

适合初次接触本项目，希望快速理解架构和使用的开发者：

1. **[概览](00_Overview.md)** → 了解项目定位、核心能力、运行环境
2. **[架构设计](01_Architecture.md)** → 理解四层架构、组件关系、数据流
3. **[构建配置](02_Build.md)** → 掌握 GN 构建系统、Feature 开关
4. **[内部实现细节](06_Internals.md)** → 深入核心类、生命周期、管理机制

**预计时间**: 60-90 分钟

### 🔒 安全研究路线

适合进行安全审计、漏洞挖掘的研究员：

1. **[概览](00_Overview.md)** → 快速了解项目范围（10分钟）
2. **[攻击面分析](05_AttackSurface.md)** → 外部输入、敏感操作、信任边界
3. **[IPC 接口清单](04_Interface.md)** → 125 个 RPC 方法详细列表
4. **[安全评审](03_Security.md)** → 风险点详细分析、利用路径
5. **[内部实现细节](06_Internals.md)** → 安全机制实现细节

**预计时间**: 2-3 小时

---

## 核心文档索引

### 📖 概览与入门

| 章节 | 文件 | 内容概要 | 推荐受众 |
|-----|------|---------|---------|
| 首页 | [README](README.md) | 项目介绍、导航、更新说明 | 全部 |
| 概览 | [00_Overview.md](00_Overview.md) | 项目定位、核心能力、关键概念 | 全部 |

### 🏗️ 架构与设计

| 章节 | 文件 | 内容概要 | 推荐受众 |
|-----|------|---------|---------|
| 架构设计 | [01_Architecture.md](01_Architecture.md) | 四层架构、组件图、数据流、线程模型 | 新人 |
| 内部实现 | [06_Internals.md](06_Internals.md) | 核心类图、生命周期、元数据管理 | 进阶 |

### 🔨 构建与工程

| 章节 | 文件 | 内容概要 | 推荐受众 |
|-----|------|---------|---------|
| 构建配置 | [02_Build.md](02_Build.md) | GN 配置、Targets、编译产物 | 新人 |

### 🔒 安全与审计

| 章节 | 文件 | 内容概要 | 推荐受众 |
|-----|------|---------|---------|
| 攻击面分析 | [05_AttackSurface.md](05_AttackSurface.md) | 外部输入、敏感操作、信任边界图 | 安全研究员 |
| IPC 接口清单 | [04_Interface.md](04_Interface.md) | 125 个 RPC 方法、权限矩阵 | 安全研究员 |
| 安全评审 | [03_Security.md](03_Security.md) | 风险点、漏洞模式、修复建议 | 安全研究员 |

---

## 快速参考

### 关键代码位置

| 组件 | 路径 | 说明 |
|-----|------|-----|
| 服务入口 | `services/distributeddataservice/app/src/kvstore_data_service.cpp` | SystemAbility 实现 |
| IPC 分发 | `services/distributeddataservice/app/src/feature_stub_impl.h` | Feature 请求分发 |
| 权限检查 | `services/distributeddataservice/service/permission/` | 权限验证中心 |
| Feature 注册 | `services/distributeddataservice/framework/include/feature/feature_system.h` | 插件系统 |

### 关键配置

| 配置项 | 文件 | 说明 |
|-------|------|-----|
| SA ID | `services/distributeddataservice/sa_profile/1301.json` | SystemAbility ID |
| Feature 开关 | `datamgr_service.gni` | 编译期功能开关 |
| Bundle 配置 | `bundle.json` | 组件定义和依赖 |

### 关键常量

| 常量 | 值 | 说明 |
|-----|-----|-----|
| SA ID | 1301 | SystemAbility ID |
| 进程名 | distributeddata | 服务进程 |
| IPC 描述符 | `OHOS.DistributedData.ServiceProxy` | Feature 接口 |
| 核心权限 | `ohos.permission.DISTRIBUTED_DATASYNC` | 同步权限 |
| 云权限 | `ohos.permission.CLOUDDATA_CONFIG` | 云配置权限 |

---

## 附录

### 参考信息

| 内容 | 文件 | 说明 |
|-----|------|-----|
| 术语表 | [appendix/Glossary.md](appendix/Glossary.md) | 核心术语定义 |
| 错误码 | [appendix/ErrorCodes.md](appendix/ErrorCodes.md) | 错误码与含义 |

### 内部文档

| 内容 | 文件 | 说明 |
|-----|------|-----|
| 项目评估 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | Phase 0 评估结果 |
| 证据汇总 | [_work/NOTES.md](_work/NOTES.md) | 代码证据记录 |
| 任务计划 | [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

## 文档统计

| 分类 | 数量 | 说明 |
|-----|------|-----|
| 核心文档 | 7 | Overview, Architecture, Build, Security, Interface, AttackSurface, Internals |
| 附录文档 | 2 | Glossary, ErrorCodes |
| 内部文档 | 3 | ASSESSMENT, NOTES, PLAN |
| **总计** | **12** | 不含测试相关内容 |

---

## 版本信息

- **当前版本**: 1.1
- **最后更新**: 2026-02-07
- **适用分支**: master
- **更新内容**: 新增 IPC 接口清单、攻击面分析、内部实现细节

---

## 维护说明

### 文档更新触发条件

1. **新增/删除 Feature 模块** → 更新 Architecture, Interface, AttackSurface
2. **修改 GN feature 开关** → 更新 Build
3. **架构重大变更** → 更新 Architecture, Internals
4. **新增安全风险点** → 更新 AttackSurface, Security
5. **新增 RPC 方法** → 更新 Interface, AttackSurface

### 证据追溯原则

所有技术结论必须包含代码证据：
- 文件路径 (含行号): `path/to/file.cpp:123`
- 符号名: `ClassName::methodName()`
- 最小代码片段

---

**导航提示**: 使用 `[Ctrl+点击]` 或 `[Cmd+点击]` 跳转文档链接
