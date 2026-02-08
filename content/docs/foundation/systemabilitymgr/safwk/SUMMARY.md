# SAFWK Wiki 导航

## 文档索引

### 快速入门

| 章节 | 标题 | 描述 |
|------|------|------|
| [README](README.md) | 首页 | 文档覆盖范围、更新方式、版本信息 |
| [00_Overview](00_Overview.md) | 项目概览 | 定位、能力、运行环境、关键概念 |

### 核心文档

| 章节 | 标题 | 描述 |
|------|------|------|
| [01_Architecture](01_Architecture.md) | 架构设计 | 组件图、数据流、线程模型、生命周期 |
| [02_APIs](02_APIs.md) | API 接口 | C++ SDK、Rust 绑定、注册宏（无 N-API） |
| [03_Build](03_Build.md) | 构建系统 | GN Targets、依赖关系、产物清单 |

### 专项文档

| 章节 | 标题 | 描述 |
|------|------|------|
| [04_Security](04_Security.md) | 安全风险评审 | 攻击面、信任边界、风险点与修复建议 |
| [05_FAQ](05_FAQ.md) | 常见问题 | 构建、运行、调试问题与解决方案 |

## 新人阅读路线图

```
┌─────────────────────────────────────────────────────────────┐
│                    新人阅读推荐顺序                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. README.md                                               │
│     └── 了解文档覆盖范围、更新方式                           │
│                                                             │
│     ↓                                                       │
│                                                             │
│  2. 00_Overview.md                                         │
│     └── 项目定位、核心能力、运行环境                          │
│                                                             │
│     ↓                                                       │
│                                                             │
│  3. 01_Architecture.md                                     │
│     └── SystemAbility、IPC 框架、生命周期                    │
│                                                             │
│     ↓                                                       │
│                                                             │
│  4. 02_APIs.md                                             │
│     └── SDK 接口、开发步骤、注册宏                           │
│                                                             │
│     ↓                                                       │
│                                                             │
│  5. 03_Build.md                                            │
│     └── 构建配置、产物、安装路径                             │
│                                                             │
│     ↓ (按需阅读)                                            │
│                                                             │
│  6. 04_Security.md                                         │
│     └── 安全风险、攻击面、防护措施                           │
│                                                             │
│     ↓ (按需阅读)                                            │
│                                                             │
│  7. 05_FAQ.md                                              │
│     └── 常见问题、调试技巧、问题定位                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 模块速查

### 代码模块

| 路径 | 职责 | 关键文件 |
|------|------|----------|
| `services/safwk/` | 核心服务实现 | system_ability.cpp, local_ability_manager.cpp |
| `interfaces/innerkits/safwk/` | C++ SDK | system_ability.h, api_cache_manager.h |
| `interfaces/innerkits/safwk/rust/` | Rust 绑定 | ability.rs, wrapper.rs |
| `svc/` | 服务控制工具 | svc_control.cpp |
| `etc/profile/` | 配置文件 | foundation.cfg, foundation_trust.json |

### 关键类/接口

| 名称 | 类型 | 描述 |
|------|------|------|
| `SystemAbility` | 基类 | 所有 SA 的基类 |
| `LocalAbilityManager` | 管理器 | SA 生命周期管理 |
| `IRemoteBroker` | 接口 | IPC Broker 基类 |
| `IRemoteProxy` | 代理 | IPC 客户端代理 |
| `IRemoteStub` | 存根 | IPC 服务端存根 |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 3.1 | 2026-02-06 | 初始 Wiki 生成 |
