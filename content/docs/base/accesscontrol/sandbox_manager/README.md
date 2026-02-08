# Sandbox Manager Wiki

> OpenHarmony 应用沙箱策略管理系统文档

## 项目简介

**Sandbox Manager** 是 OpenHarmony `accesscontrol` 子系统中的核心系统服务，负责管理应用沙箱间的文件共享策略。它提供持久化和临时策略管理能力，确保应用间的文件访问安全可控。

**核心功能**：
- **持久化策略**：策略存储在关系型数据库，有效期至应用卸载
- **临时策略**：策略仅在当前应用生命周期内有效（存储在 MAC 内核层）
- **策略检查**：验证应用对文件的访问权限
- **批量管理**：支持按路径、用户 ID、Token ID 清理策略

**技术特点**：
- SystemAbility (SA) 服务架构
- 三层架构设计（Interface/Framework/Service）
- 依赖 MAC (Mandatory Access Control) 内核层实施访问控制
- 使用 RDB (关系型数据库) 持久化存储

## 文档结构

### 快速导航

| 文档 | 描述 | 受众 |
|-----|------|-----|
| [01_Overview](01_Overview.md) | 项目定位、能力边界、快速开始 | 新人 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、时序图 | 全部 |
| [03_CodeMap](03_CodeMap.md) | 目录结构、代码导航 | 开发者 |
| [04_Interface](04_Interface.md) | Inner Kit API 完整清单 | 开发者 |
| [05_AttackSurface](05_AttackSurface.md) | 外部输入清单、信任边界 | 安全研究员 |
| [06_SecurityReview](06_SecurityReview.md) | 安全风险深度分析 | 安全研究员 |
| [07_Build](07_Build.md) | GN targets、feature 开关 | 开发者 |
| [08_Internals](08_Internals.md) | 核心类实现细节 | 高级开发者 |

### 推荐阅读路径

**新人学习路线**（30 分钟理解项目）:
```
README → 01_Overview → 02_Architecture → 04_Interface → 03_CodeMap
```

**安全研究路线**（快速定位风险）:
```
README → 05_AttackSurface → 06_SecurityReview → 02_Architecture
```

**开发者路线**（API 使用参考）:
```
README → 04_Interface → 08_Internals → 07_Build
```

## 关键概念

### 策略类型 (PolicyType)

| 类型 | 说明 | 权限要求 |
|-----|------|---------|
| `SELF_PATH` | 应用自有文件路径 | 需验证 bundle name |
| `AUTHORIZATION_PATH` | 授权访问的路径 | 需授权验证 |
| `OTHERS_PATH` | 其他应用的路径 | 高权限验证 |

### 操作模式 (OperateMode)

| 模式 | 值 | 描述 |
|-----|---|------|
| `READ_MODE` | 0x01 | 读权限 |
| `WRITE_MODE` | 0x02 | 写权限 |
| `CREATE_MODE` | 0x04 | 创建权限 |
| `DELETE_MODE` | 0x08 | 删除权限 |
| `DENY_READ_MODE` | 0x20 | 拒绝读 |
| `DENY_WRITE_MODE` | 0x40 | 拒绝写 |

### 权限常量

| 权限名 | 用途 |
|-------|------|
| `ohos.permission.SET_SANDBOX_POLICY` | 设置策略 |
| `ohos.permission.CHECK_SANDBOX_POLICY` | 检查策略 |
| `ohos.permission.FILE_ACCESS_PERSIST` | 持久化访问 |
| `ohos.permission.FILE_ACCESS_MANAGER` | 文件访问管理 |

## 构建信息

- **构建系统**: GN (Generate Ninja)
- **子系统**: accesscontrol
- **组件**: sandbox_manager
- **主要依赖**: samgr, relational_store, access_token, ipc

## 更新日志

| 版本 | 日期 | 变更 |
|-----|------|-----|
| 1.0.0 | 2024-XX-XX | 初始版本 |

## 相关资源

- OpenHarmony 主仓库: https://gitee.com/openharmony
- accesscontrol 子系统文档
- MAC 内核模块文档

---

*文档最后更新: 2025-02-07*
