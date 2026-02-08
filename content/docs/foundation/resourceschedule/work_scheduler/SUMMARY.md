# SUMMARY - Work Scheduler Wiki

**文档导航与推荐阅读路径**

---

## 快速导航

| 文档 | 阅读时长 | 目标受众 | 主要内容 |
|------|----------|----------|----------|
| [README.md](README.md) | 2分钟 | 所有人 | 项目概览、文档导航 |
| [01_Overview.md](01_Overview.md) | 10分钟 | 新人 | 项目定位、快速开始、使用约束 |
| [02_Architecture.md](02_Architecture.md) | 15分钟 | 开发者 | 架构图、数据流、组件关系 |
| [03_CodeMap.md](03_CodeMap.md) | 10分钟 | 开发者 | 目录结构、代码导航、关键文件 |
| [04_Interface.md](04_Interface.md) | 15分钟 | 开发者 | N-API、IPC、Extension API |
| [05_AttackSurface.md](05_AttackSurface.md) | 15分钟 | 安全研究员 | 攻击面、输入入口、信任边界 |
| [06_SecurityReview.md](06_SecurityReview.md) | 20分钟 | 安全研究员 | 风险评估、漏洞分析、修复建议 |
| [07_Build.md](07_Build.md) | 10分钟 | 开发者 | GN目标、编译产物、Feature开关 |
| [08_Internals.md](08_Internals.md) | 15分钟 | 高级开发者 | 核心类、内部API、生命周期 |

---

## 推荐阅读路线

### 🔰 新人学习路线 (30分钟 → 2小时)

**目标**: 快速理解项目并上手开发

```mermaid
graph LR
    A[README.md] --> B[01_Overview.md]
    B --> C[02_Architecture.md]
    C --> D[04_Interface.md]
    D --> E[03_CodeMap.md]
    E --> F[实践开发]
```

**第1步：了解项目 (5分钟)**
- 阅读 [README.md](README.md)
- 重点：项目定位、核心功能、约束条件

**第2步：深入理解 (10分钟)**
- 阅读 [01_Overview.md](01_Overview.md)
- 重点：能力边界、快速开始示例、使用约束

**第3步：理解架构 (15分钟)**
- 阅读 [02_Architecture.md](02_Architecture.md)
- 重点：组件图、数据流、线程模型

**第4步：学习API (15分钟)**
- 阅读 [04_Interface.md](04_Interface.md)
- 重点：N-API使用、参数说明、错误处理

**第5步：代码导航 (15分钟)**
- 阅读 [03_CodeMap.md](03_CodeMap.md)
- 重点：目录结构、关键文件位置

**第6步：动手实践**
- 参考 [01_Overview.md](01_Overview.md#快速开始) 的示例代码
- 编写自己的 Work Scheduler 应用

---

### 🔒 安全研究路线 (1小时 → 半天)

**目标**: 全面评估安全风险并发现潜在漏洞

```mermaid
graph LR
    A[README.md] --> B[05_AttackSurface.md]
    B --> C[06_SecurityReview.md]
    C --> D[03_CodeMap.md]
    D --> E[02_Architecture.md]
    E --> F[深入审计]
```

**第1步：了解项目 (5分钟)**
- 快速浏览 [README.md](README.md)
- 了解项目功能和边界

**第2步：识别攻击面 (20分钟)**
- 阅读 [05_AttackSurface.md](05_AttackSurface.md)
- 重点：外部输入清单、敏感操作、信任边界

**第3步：风险评估 (30分钟)**
- 阅读 [06_SecurityReview.md](06_SecurityReview.md)
- 重点：已识别的风险、触发路径、影响评估

**第4步：代码定位 (20分钟)**
- 阅读 [03_CodeMap.md](03_CodeMap.md)
- 重点：安全相关文件位置、关键函数

**第5步：理解架构 (20分钟)**
- 阅读 [02_Architecture.md](02_Architecture.md)
- 重点：数据流、组件交互、权限模型

**第6步：深入审计**
- 根据攻击面清单逐条审计
- 重点关注：
  - 输入验证逻辑
  - 权限检查点
  - IPC消息处理
  - 文件操作

---

### 🏗️ 架构师路线 (1小时 → 1天)

**目标**: 深入理解系统设计，进行架构评审或优化

```mermaid
graph LR
    A[README.md] --> B[02_Architecture.md]
    B --> C[03_CodeMap.md]
    C --> D[08_Internals.md]
    D --> E[04_Interface.md]
    E --> F[07_Build.md]
    F --> G[架构评审]
```

**阅读顺序**:
1. [README.md](README.md) - 项目概览
2. [02_Architecture.md](02_Architecture.md) - 完整架构理解
3. [03_CodeMap.md](03_CodeMap.md) - 代码组织结构
4. [08_Internals.md](08_Internals.md) - 实现细节
5. [04_Interface.md](04_Interface.md) - 接口设计
6. [07_Build.md](07_Build.md) - 构建系统

---

## 按主题索引

### 🔍 安全相关

| 主题 | 相关文档 | 关键章节 |
|------|----------|----------|
| 攻击面识别 | [05_AttackSurface.md](05_AttackSurface.md) | 外部输入清单、信任边界 |
| 输入验证 | [06_SecurityReview.md](06_SecurityReview.md) | R1: 输入验证缺陷 |
| 权限控制 | [06_SecurityReview.md](06_SecurityReview.md) | R2: 权限与鉴权 |
| IPC安全 | [05_AttackSurface.md](05_AttackSurface.md) | IPC接口输入 |
| 文件安全 | [06_SecurityReview.md](06_SecurityReview.md) | R1.3: 路径遍历 |

### 🔧 开发相关

| 主题 | 相关文档 | 关键章节 |
|------|----------|----------|
| N-API使用 | [04_Interface.md](04_Interface.md) | N-API接口 |
| Extension开发 | [04_Interface.md](04_Interface.md) | Extension API |
| 错误处理 | [04_Interface.md](04_Interface.md) | 错误码 |
| 代码导航 | [03_CodeMap.md](03_CodeMap.md) | 代码导航图 |
| 构建配置 | [07_Build.md](07_Build.md) | GN目标清单 |

### 📊 架构相关

| 主题 | 相关文档 | 关键章节 |
|------|----------|----------|
| 组件关系 | [02_Architecture.md](02_Architecture.md) | 组件详解 |
| 数据流 | [02_Architecture.md](02_Architecture.md) | 数据流图 |
| 线程模型 | [02_Architecture.md](02_Architecture.md) | 线程模型 |
| 核心类 | [08_Internals.md](08_Internals.md) | 核心类职责 |
| 生命周期 | [08_Internals.md](08_Internals.md) | 资源生命周期 |

---

## 关键词索引

### API 关键词

| 关键词 | 相关文档 | 说明 |
|--------|----------|------|
| `startWork` | [04_Interface.md](04_Interface.md) | 启动延迟任务 |
| `stopWork` | [04_Interface.md](04_Interface.md) | 停止延迟任务 |
| `getWorkStatus` | [04_Interface.md](04_Interface.md) | 获取任务状态 |
| `obtainAllWorks` | [04_Interface.md](04_Interface.md) | 获取所有任务 |
| `stopAndClearWorks` | [04_Interface.md](04_Interface.md) | 清除所有任务 |
| `isLastWorkTimeOut` | [04_Interface.md](04_Interface.md) | 检查上次超时 |
| `WorkSchedulerExtensionAbility` | [04_Interface.md](04_Interface.md) | Extension基类 |
| `onWorkStart` | [04_Interface.md](04_Interface.md) | 任务开始回调 |
| `onWorkStop` | [04_Interface.md](04_Interface.md) | 任务停止回调 |

### 架构关键词

| 关键词 | 相关文档 | 说明 |
|--------|----------|------|
| `SA 1904` | [01_Overview.md](01_Overview.md) | System Ability ID |
| `IWorkSchedService` | [02_Architecture.md](02_Architecture.md) | IPC服务接口 |
| `WorkQueueManager` | [02_Architecture.md](02_Architecture.md) | 队列管理器 |
| `WorkPolicyManager` | [02_Architecture.md](02_Architecture.md) | 策略管理器 |
| `WorkSchedulerService` | [02_Architecture.md](02_Architecture.md) | 主服务类 |
| `WorkInfo` | [02_Architecture.md](02_Architecture.md) | 任务信息 |

### 安全关键词

| 关键词 | 相关文档 | 说明 |
|--------|----------|------|
| `CheckWorkInfo` | [05_AttackSurface.md](05_AttackSurface.md) | 参数验证 |
| `CheckProcessName` | [06_SecurityReview.md](06_SecurityReview.md) | 进程名白名单 |
| `CheckCallingToken` | [06_SecurityReview.md](06_SecurityReview.md) | Token检查 |
| `OnRemoteRequest` | [05_AttackSurface.md](05_AttackSurface.md) | IPC入口 |
| `ConvertFullPath` | [06_SecurityReview.md](06_SecurityReview.md) | 路径验证 |

---

## 文档维护信息

### 版本历史

| 版本 | 日期 | 更新内容 | 作者 |
|------|------|----------|------|
| 1.0 | 2026-02-07 | 初始版本，完成基础架构和安全分析 | AI Agent |

### 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完整 | 定位、能力、约束 |
| 架构设计 | ✅ 完整 | 组件、数据流、线程 |
| 代码地图 | ✅ 完整 | 目录、导航、索引 |
| N-API接口 | ✅ 完整 | 6个API完整说明 |
| IPC接口 | ✅ 完整 | 16个方法定义 |
| 攻击面分析 | ✅ 完整 | 输入、操作、边界 |
| 安全评估 | ✅ 完整 | 5类风险分析 |
| 构建系统 | ✅ 完整 | GN目标、产物 |
| 内部实现 | ✅ 完整 | 核心类、生命周期 |

### 未覆盖范围

- 测试代码（按规范忽略）
- IDL生成代码（自动生成的Proxy/Stub）
- 第三方依赖的内部实现

---

## 反馈与贡献

发现文档问题？
- 代码证据不准确？请提供正确的文件路径和行号
- 安全分析有遗漏？请详细描述攻击场景
- 架构理解有偏差？请提供正确的组件关系

---

**最后更新**: 2026-02-07  
**文档版本**: 1.0  
**维护者**: AI Agent
