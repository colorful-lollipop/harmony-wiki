# 00_Overview - 项目概览

## 项目定位

**EcologicalRuleManagerService（生态规则管控服务）** 是 OpenHarmony BundleManager 子系统中的一个系统扩展能力（System Ability, SA）。

**核心价值**: 允许设备厂商在定制设备上（2B 合作项目等），对应用的行为进行管控，包括：

- **跳转控制**: 控制元服务（Atomic Service）的拉起
- **加桌管控**: 控制卡片（Form）添加到桌面的行为
- **免安装管理**: 控制元服务的免安装（Free Install）策略

## 运行环境

| 条件 | 要求 |
|------|------|
| **系统类型** | OpenHarmony Standard System |
| **子系统** | bundlemanager |
| **System Ability ID** | 6105 |
| **运行进程** | foundation |
| **启动方式** | run-on-create: true（随进程启动） |

**依赖组件**:
- ability_base / ability_runtime
- bundle_framework
- ipc / samgr (IPC 通信)
- safwk (System Ability Framework)
- access_token (权限管理)
- hilog (日志)

## 核心能力

### 1. 体验规则查询 (Experience Rule Query)

系统服务（AbilityManagerService, BundleManagerService）在执行特定操作前，调用本服务查询是否允许该操作，以及允许时的具体体验规则。

**支持的调用方**:
- `AbilityManagerService` - 控制元服务跳转
- `BundleManagerService` - 控制免安装
- `FormManagerService` - 控制卡片加桌

### 2. 过滤规则评估 (Resolve Info Evaluation)

支持过滤掉不允许出现的服务提供者，实现白名单/黑名单机制。

## 版本信息

| 字段 | 值 |
|------|-----|
| **bundle.json version** | 3.1 |
| **License** | Apache License 2.0 |
| **ROM 预估** | 300KB |
| **RAM 预估** | 1024KB |

## 关键概念

| 概念 | 描述 |
|------|------|
| **元服务 (Atomic Service)** | 免安装、即用即走的轻量应用 |
| **体验 Want (Experience Want)** | 替代原始 Want 的自定义跳转意图 |
| **CallerInfo** | 调用方信息（包名、UID、PID、应用类型等） |
| **ExperienceRule** | 体验规则（是否允许、返回码、替代 Want） |

## 相关文档

- 目录结构: [01_Directory_Structure.md](01_Directory_Structure.md)
- 架构设计: [02_Architecture.md](02_Architecture.md)
- API 接口: [03_Inner_API.md](03_Inner_API.md)
