# SecurityGuard Wiki 导航

## 文档索引

### 入门

| 文档 | 说明 | 新人必读 |
|------|------|----------|
| [README.md](./README.md) | Wiki 覆盖范围与更新方式 | ✅ |
| [01_Overview.md](./01_Overview.md) | 项目定位、核心能力、架构图 | ✅ |
| [07_QuickStart.md](./07_QuickStart.md) | 5 分钟快速开始指南 | ✅ |

### 核心文档

| 文档 | 说明 | 上次更新 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | 项目定位、核心能力、运行环境、目录结构 | 2026-02-06 |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API 参考（8 个 API 详解） | 2026-02-06 |
| [03_Architecture.md](./03_Architecture.md) | 模块职责、线程模型、IPC 通信、数据流 | 2026-02-06 |
| [04_Build.md](./04_Build.md) | GN 构建配置、targets 列表、编译产物 | 2026-02-06 |
| [05_Services.md](./05_Services.md) | SA 服务架构、IPC 接口、权限管理 | 2026-02-06 |

### 安全分析

| 文档 | 说明 | 状态 |
|------|------|------|
| [06_Security_Review.md](./06_Security_Review.md) | 安全风险评审、攻击面分析、可利用点 | 2026-02-06 |

### 开发指南

| 文档 | 说明 | 状态 |
|------|------|------|
| [07_QuickStart.md](./07_QuickStart.md) | 5 分钟快速开始指南 | 2026-02-06 |
| [08_Debugging.md](./08_Debugging.md) | 调试指南、日志分析、问题排查 | 2026-02-06 |

### 示例代码

| 文档 | 说明 | 状态 |
|------|------|------|
| [samples/01_device_check.md](./samples/01_device_check.md) | 设备安全检查示例 | 2026-02-06 |
| [samples/02_event_monitoring.md](./samples/02_event_monitoring.md) | 事件监控示例 | 2026-02-06 |
| [samples/03_risk_app.md](./samples/03_risk_app.md) | 风险控制集成示例 | 2026-02-06 |

### 附录

| 文档 | 说明 | 状态 |
|------|------|------|
| [appendix/99_Glossary.md](./appendix/99_Glossary.md) | 完整术语表 | 2026-02-06 |
| [appendix/99_ErrorCodeQuickRef.md](./appendix/99_ErrorCodeQuickRef.md) | 错误码速查表 | 2026-02-06 |

---

## 阅读路线图

### 路线 1：应用开发者（使用 JS API）

```
README.md → 01_Overview.md → 02_NAPI_Reference.md
```

### 路线 2：快速上手

```
README.md → 07_QuickStart.md → samples/01_device_check.md
```

### 路线 3：系统集成（构建与部署）

```
01_Overview.md → 04_Build.md → 03_Architecture.md → 05_Services.md
```

### 路线 4：安全评审

```
01_Overview.md → 06_Security_Review.md → 02_NAPI_Reference.md
```

### 路线 5：完整示例学习

```
07_QuickStart.md → samples/01_device_check.md → samples/02_event_monitoring.md → samples/03_risk_app.md
```

### 路线 6：完整技术深入

```
README.md → 01_Overview.md → 02_NAPI_Reference.md → 03_Architecture.md → 
04_Build.md → 05_Services.md → 06_Security_Review.md → 07_QuickStart.md → samples/*.md
```

---

## 文档依赖关系

```
README.md (项目介绍)
    │
    ├──► 01_Overview.md (基础依赖)
    │         │
    │         ├──► 02_NAPI_Reference.md (API 使用)
    │         │         │
    │         │         └──► 06_Security_Review.md (API 安全)
    │         │
    │         ├──► 03_Architecture.md (架构详解)
    │         │         │
    │         │         └──► 05_Services.md (SA 服务详解)
    │         │
    │         ├──► 04_Build.md (构建配置)
    │         │
    │         └──► 07_QuickStart.md (快速开始)
    │                   │
    │                   └──► samples/*.md (示例代码)
    │
    └──► appendix/*.md (参考文档)
```

---

## 术语表

| 术语 | 定义 |
|------|------|
| SG | SecurityGuard，设备风险管理平台 |
| SA | System Ability，系统能力 |
| N-API | Node.js API，OpenHarmony JS 接口层 |
| IDL | Interface Definition Language，接口定义语言 |
| Model ID | 安全模型标识符 |
| FFRT | Fast Function Runtime，异步任务调度框架 |
| Token Bucket | 令牌桶限流算法 |
| IPC | Inter-Process Communication，进程间通信 |
| Inner API | 平台 SDK 接口 |
| Security Event | 安全事件，用于描述安全相关发生的事 |
| Model Result | 模型推理结果，包含风险评估和置信度 |
| Policy | 策略文件，用于配置安全检测规则 |
| Event ID | 事件标识符，用于区分不同类型的安全事件 |
| Collector | 采集器，负责收集设备安全相关数据 |
| Risk Level | 风险等级，评估设备或应用的风险程度 |
| Device Integrity | 设备完整性，验证设备系统是否被篡改 |

---

## 快速导航

### 常用链接

| 场景 | 链接 |
|------|------|
| 首次了解项目 | [01_Overview.md](./01_Overview.md) |
| 查找 API 使用 | [02_NAPI_Reference.md](./02_NAPI_Reference.md) |
| 快速开始开发 | [07_QuickStart.md](./07_QuickStart.md) |
| 查找错误码 | [appendix/99_ErrorCodeQuickRef.md](./appendix/99_ErrorCodeQuickRef.md) |
| 查找术语定义 | [appendix/99_Glossary.md](./appendix/99_Glossary.md) |
| 调试问题 | [08_Debugging.md](./08_Debugging.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

### 代码示例

| 场景 | 示例文档 |
|------|----------|
| 设备安全检查 | [samples/01_device_check.md](./samples/01_device_check.md) |
| 事件监控 | [samples/02_event_monitoring.md](./samples/02_event_monitoring.md) |
| 风险控制集成 | [samples/03_risk_app.md](./samples/03_risk_app.md) |
