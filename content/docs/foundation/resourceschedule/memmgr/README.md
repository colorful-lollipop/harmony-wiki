# MemMgr 组件 Wiki

> OpenHarmony 内存管理部件完整技术文档

## 文档覆盖范围

本 Wiki 覆盖 OpenHarmony `foundation/resourceschedule/memmgr` 组件的完整技术细节，包括：

### 已覆盖内容

| 模块 | 描述 | 证据来源 |
|------|------|---------|
| 架构设计 | 事件驱动架构、模块职责划分 | `README.md`, `README_zh.md` |
| 进程回收优先级 | -1000~400 优先级体系及更新机制 | `reclaim_priority_manager/` |
| 回收策略 | avail_buffer, kswapd/zswapd 参数配置 | `reclaim_strategy_manager/` |
| 查杀策略 | 低内存查杀器，内存水线联动 | `kill_strategy_manager/` |
| 事件中心 | 6类事件监听器注册与通知 | `event/` |
| Inner API | C++ 对内接口 (无 N-API) | `interface/innerkits/` |
| GN 构建 | BUILD.gn targets 与编译产物 | `BUILD.gn`, `memmgr.gni` |
| SA 配置 | SA ID 1909, 启动配置 | `sa_profile/1909.json` |

### 未覆盖内容

| 范围 | 原因 |
|------|------|
| N-API (JS API) | 本组件**无 N-API**，仅提供 Inner C++ API |
| 测试代码 | 按规范忽略 (`test/` 目录) |
| 运行时动态行为 | 需要运行时环境验证，待补充 |

## 文档更新方式

当代码变更时，需同步更新对应 Wiki 章节：

1. **接口变更** → 更新 `02_Inner_API.md`
2. **模块新增** → 更新 `01_Architecture.md` + `SUMMARY.md`
3. **配置变更** → 更新 `03_Build.md` 配置节
4. **安全风险** → 更新 `04_Security.md`

## 生成信息

- **组件版本**: 3.1.0
- **生成时间**: 2026-02-06
- **代码仓库**: `foundation/resourceschedule/memmgr`
- **文档规范**: OpenHarmony 工程 Wiki 生成标准 v1.0

## 快速导航

```mermaid
graph LR
    A[新人入口] --> B[SUMMARY.md]
    B --> C[概览页]
    C --> D[架构图]
    D --> E[Inner API]
    E --> F[构建配置]
    F --> G[安全评审]
```

---

*文档由 OpenHarmony 工程 Wiki Agent 自动生成*
