# 文档导航

> 本文档为设备互信认证模块的全站导航，提供新人阅读路线和各文档间的跳转链接。

---

## 新人阅读路线

### 路线 A：快速入门（30 分钟）

1. **[01_Overview.md](./01_Overview.md)** - 项目定位与核心功能
2. **[03_API_Reference.md](./03_API_Reference.md)** - JS API 快速使用
3. **[07_Troubleshooting.md](./07_Troubleshooting.md)** - 常见问题速查

### 路线 B：深入理解（2 小时）

1. **[01_Overview.md](./01_Overview.md)** - 项目概述
2. **[02_Architecture.md](./02_Architecture.md)** - 架构设计与数据流
3. **[03_API_Reference.md](./03_API_Reference.md)** - API 详解
4. **[04_Inner_API.md](./04_Inner_API.md)** - 内部模块接口
5. **[appendix/Callgraphs.md](./appendix/Callgraphs.md)** - 关键调用链

### 路线 C：开发集成（3 小时）

1. **[01_Overview.md](./01_Overview.md)** - 项目概述
2. **[02_Architecture.md](./02_Architecture.md)** - 架构设计
3. **[03_API_Reference.md](./03_API_Reference.md)** - API 使用
4. **[04_Inner_API.md](./04_Inner_API.md)** - Inner API 集成
5. **[05_Build_Config.md](./05_Build_Config.md)** - 构建配置
6. **[07_Troubleshooting.md](./07_Troubleshooting.md)** - 调试指南

### 路线 D：安全评估

1. **[06_Security_Review.md](./06_Security_Review.md)** - 安全风险评审
2. **[02_Architecture.md](./02_Architecture.md)** - 信任边界
3. **[appendix/Config_Flags.md](./appendix/Config_Flags.md)** - 安全配置

---

## 文档索引

### 核心文档

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| **[01_Overview.md](./01_Overview.md)** | 项目概览 | 定位、功能、运行环境 |
| **[02_Architecture.md](./02_Architecture.md)** | 架构设计 | 组件图、数据流、时序 |
| **[03_API_Reference.md](./03_API_Reference.md)** | N-API 参考 | JS API 清单、参数、示例 |
| **[04_Inner_API.md](./04_Inner_API.md)** | Inner API | C/C++ 模块接口 |
| **[05_Build_Config.md](./05_Build_Config.md)** | 构建配置 | GN targets、产物、依赖 |
| **[06_Security_Review.md](./06_Security_Review.md)** | 安全评审 | 攻击面、风险点、修复建议 |
| **[07_Troubleshooting.md](./07_Troubleshooting.md)** | 常见问题 | 构建/运行/调试问题 |

### 附录

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| **[appendix/Callgraphs.md](./appendix/Callgraphs.md)** | 调用链图 | 入口→核心逻辑调用链 |
| **[appendix/Config_Flags.md](./appendix/Config_Flags.md)** | 配置开关 | 宏定义、feature flags |

---

## API 快速索引

### N-API（JS 接口）

| API | 类名 | 用途 |
|-----|------|------|
| `getCredMgrInstance()` | 模块级 | 获取 CredManager 实例 |
| `batchUpdateCredentials()` | CredManager | 批量更新凭证 |

### Inner API（C/C++ 接口）

| 接口 | 模块 | 用途 |
|------|------|------|
| `GetGmInstance()` | GroupManager | 获取群组管理实例 |
| `GetGaInstance()` | GroupAuthManager | 获取群组认证实例 |
| `GetCredMgrInstance()` | IdentityService | 获取凭证管理实例 |
| `StartAuthDevice()` | IdentityService | 启动设备认证 |
| `ProcessAuthDevice()` | IdentityService | 处理认证数据 |

---

## 架构组件索引

| 组件 | 路径 | 职责 |
|------|------|------|
| Group Manager | `services/legacy/group_manager/` | 设备群组管理 |
| Group Auth | `services/legacy/group_auth/` | 设备群组认证 |
| Identity Service | `services/identity_service/` | 凭证管理 |
| Session Manager | `services/session_manager/` | 会话生命周期 |
| Protocol | `services/protocol/` | 加密协议实现 |
| MK Agree | `services/mk_agree/` | 主密钥协商 |

---

## 关键词索引

| 关键词 | 相关文档 |
|--------|----------|
| N-API | 03_API_Reference.md, 07_Troubleshooting.md |
| SA 4701 | 02_Architecture.md, 04_Inner_API.md |
| Binder IPC | 02_Architecture.md, 04_Inner_API.md |
| GN 构建 | 05_Build_Config.md, appendix/Config_Flags.md |
| 安全风险 | 06_Security_Review.md |
| 调试 | 07_Troubleshooting.md, appendix/Callgraphs.md |
| 凭证管理 | 03_API_Reference.md, 04_Inner_API.md |

---

## 版本信息

| 项目 | 值 |
|------|-----|
| 模块版本 | 4.0.2 |
| OpenHarmony | 5.0+ |
| 系统能力 | 见 `bundle.json` |

---

*本文档最后更新：2026-02-06*
