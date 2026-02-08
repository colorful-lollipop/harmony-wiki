# 文档导航

> **新人阅读顺序建议**：
> 1. [01_Overview.md](./01_Overview.md) - 先了解项目定位
> 2. [02_Architecture.md](./02_Architecture.md) - 再理解整体架构
> 3. [03_NAPI.md](./03_NAPI.md) - 查看对外接口
> 4. 根据需要查阅其他文档

## 核心文档

| 文档 | 说明 | 上次更新 |
|------|------|---------|
| [README.md](./README.md) | Wiki 使用指南 | 2026-02-06 |
| [01_Overview.md](./01_Overview.md) | 项目概览：定位、能力、约束 | 2026-02-06 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计：分层、组件、IPC | 2026-02-06 |
| [03_NAPI.md](./03_NAPI.md) | N-API 接口：JS API 清单、调用链 | 2026-02-06 |
| [04_InnerAPI.md](./04_InnerAPI.md) | 内部 API：模块职责、依赖 | 2026-02-06 |
| [05_GN.md](./05_GN.md) | GN 构建：Targets、配置 | 2026-02-06 |
| [06_Artifacts.md](./06_Artifacts.md) | 编译产物：SO/可执行文件 | 2026-02-06 |
| [07_Security.md](./07_Security.md) | 安全评审：攻击面、风险 | 2026-02-06 |

## 快速索引

### 按功能分类

| 功能 | N-API 模块 | 内部模块 |
|------|----------|---------|
| 存储统计 | `file.storageStatistics` | `storage_statistics` |
| 卷管理 | `file.volumeManager` | `volume` |
| 密钥管理 | `file.keyManager` | `crypto` |
| 分布式文件 | (DFS 模块) | `storage_daemon_communication` |

### 按代码位置

| 层级 | 目录 | 说明 |
|------|------|------|
| JS 接口 | `interfaces/kits/js/` | N-API 绑定 |
| Native 接口 | `interfaces/innerkits/` | C++ 对内接口 |
| 管理服务 | `services/storage_manager/` | SA Manager |
| 守护进程 | `services/storage_daemon/` | SA Daemon |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |
