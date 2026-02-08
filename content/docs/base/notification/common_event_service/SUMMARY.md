# 文档导航

本文档为 OpenHarmony 公共事件服务的完整技术 Wiki，建议按以下顺序阅读。

## 双路线导航

### 路线一：新人学习路线

```
1. 概览 → 2. 架构 → 3. N-API → 4. 内部 API → 5. 构建 → 6. 安全
```

| 阶段 | 文档 | 目标 | 预计时间 |
|------|------|------|---------|
| 入门 | [00_概览](00_Overview.md) | 理解项目定位、核心能力 | 5分钟 |
| 理解 | [01_架构](01_Architecture.md) | 掌握组件结构、数据流 | 15分钟 |
| 使用 | [02_N-API](02_N-API.md) | 学会使用 JS 接口 | 20分钟 |
| 深入 | [03_内部API](03_Inner_API.md) | 理解 Native 接口 | 15分钟 |
| 构建 | [04_构建编译](04_Build.md) | 掌握编译配置 | 10分钟 |
| 安全 | [05_安全评审](05_Security.md) | 了解安全风险 | 15分钟 |
| 实践 | [06_FAQ](06_FAQ.md) | 解决常见问题 | 10分钟 |

### 路线二：安全研究路线

```
1. 概览 → 2. 攻击面 → 3. 输入入口 → 4. 特权操作 → 5. 风险评估
```

| 阶段 | 文档 | 目标 | 关键章节 |
|------|------|------|---------|
| 定位 | [00_概览](00_Overview.md) | 了解项目类型和暴露面 | 对外暴露面识别 |
| 架构 | [01_架构](01_Architecture.md) | 掌握数据流和信任边界 | 信任边界图 |
| 攻击面 | [05_安全评审](05_Security.md) | 识别所有外部输入入口 | 攻击面分析 |
| 接口 | [02_N-API](02_N-API.md) | 分析参数校验逻辑 | 参数解析代码 |
| 内部 | [03_内部API](03_Inner_API.md) | 理解权限检查机制 | 权限管理 |

#### 安全研究速查

| 关注点 | 文档 | 章节 |
|--------|------|------|
| 外部输入入口 | [05_安全评审](05_Security.md) | N-API参数注入、IPC接口 |
| 权限检查点 | [05_安全评审](05_Security.md) | 权限校验绕过 |
| 敏感操作 | [03_内部API](03_Inner_API.md) | 服务间调用 |
| 可利用漏洞 | [05_安全评审](05_Security.md) | 风险评估 |

---

## 文档清单

## 文档清单

| 文件 | 说明 |
|------|------|
| [README](README.md) | Wiki 说明、更新方式 |
| [SUMMARY](SUMMARY.md) | 本导航页 |
| [00_概览](00_Overview.md) | 项目定位与核心能力 |
| [01_架构](01_Architecture.md) | 系统架构与组件图 |
| [02_N-API](02_N-API.md) | JS 接口详细说明 |
| [03_内部API](03_Inner_API.md) | Native 接口说明 |
| [04_构建编译](04_Build.md) | GN 构建与产物 |
| [05_安全评审](05_Security.md) | 安全风险分析 |
| [06_FAQ](06_FAQ.md) | 常见问题解答 |

## 快速索引

### API 快速查找

| 功能 | N-API | Native API |
|------|-------|------------|
| 发布事件 | `CommonEvent.publish()` | `CommonEventManager::PublishCommonEvent()` |
| 订阅事件 | `CommonEvent.subscribe()` | `CommonEventManager::SubscribeCommonEvent()` |
| 取消订阅 | `CommonEvent.unsubscribe()` | `CommonEventManager::UnSubscribeCommonEvent()` |
| 创建订阅者 | `CommonEvent.createSubscriber()` | `CommonEventSubscriber` |
| 有序事件 | `isOrderedCommonEvent()` | `CommonEventData::IsOrdered()` |

### 文件路径速查

| 分类 | 路径 |
|------|------|
| N-API 实现 | `interfaces/kits/napi/` |
| Native API | `frameworks/native/` |
| 服务实现 | `services/src/` |
| IPC 接口 | `frameworks/core/` |
| 构建配置 | `BUILD.gn`, `event.gni` |
