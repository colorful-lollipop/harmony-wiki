# 概览

## 项目定位

HiChecker 是 OpenHarmony DFX 子系统中的**应用检测工具组件**，专为应用开发阶段设计，用于检测代码运行过程中容易被忽略的问题。

### 核心功能

| 功能 | 描述 |
|------|------|
| 耗时调用检测 | 检测应用主线程中的耗时函数调用 |
| Ability 泄露检测 | 检测 Ability 连接资源泄露 |
| ArkUI 性能检测 | 检测 ArkUI 框架性能问题 |
| 规则动态管理 | 支持运行时增删检测规则 |
| 多告警方式 | 支持日志记录、应用崩溃两种告警 |
| Native 回栈 | 支持 Native 层堆栈回溯 |

### 依赖关系

```
┌─────────────────────────────────────────────────────────┐
│                     HiChecker                            │
├─────────────────────────────────────────────────────────┤
│  媒体子系统 ← ImagePacker (耗时调用通知)                  │
│  能力子系统 ← FeatureAbility (泄露检测注册)              │
├─────────────────────────────────────────────────────────┤
│  内部依赖: hilog, faultloggerd, ipc, init                │
└─────────────────────────────────────────────────────────┘
```

## 运行环境

| 环境 | 要求 |
|------|------|
| 系统类型 | OpenHarmony Standard |
| 最低系统版本 | 4.0+ |
| SysCap | SystemCapability.HiviewDFX.HiChecker |

## 关键概念

### 检测规则 (Rule)

检测规则定义了需要检测的问题类型：

```cpp
// 文件: interfaces/native/innerkits/include/hichecker.h:27-38

const uint64_t RULE_CAUTION_PRINT_LOG = 1ULL << 63;     // 告警规则：记录日志
const uint64_t RULE_CAUTION_TRIGGER_CRASH = 1ULL << 62; // 告警规则：触发崩溃
const uint64_t RULE_THREAD_CHECK_SLOW_PROCESS = 1ULL;    // 检测规则：线程耗时调用
const uint64_t RULE_CHECK_SLOW_EVENT = 1ULL << 32;      // 检测规则：进程耗时事件
const uint64_t RULE_CHECK_ABILITY_CONNECTION_LEAK = 1ULL << 33; // 检测规则：Ability泄露
const uint64_t RULE_CHECK_ARKUI_PERFORMANCE = 1ULL << 34;       // 检测规则：ArkUI性能
```

### 告警对象 (Caution)

当检测条件满足时，系统会生成 Caution 对象，包含：

- `triggerRule`: 触发的检测规则
- `cautionMsg`: 告警消息
- `stackTrace`: 堆栈信息（Native 层）

## 使用场景

### 场景 1：检测耗时函数

```typescript
import HiChecker from '@ohos.hichecker'

// 添加耗时检测规则
HiChecker.addCheckRule(HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS)

// 在耗时函数中调用通知
HiChecker.NotifySlowProcess("HeavyComputation")
```

### 场景 2：检测 Ability 泄露

```typescript
import HiChecker from '@ohos.hichecker'

HiChecker.addCheckRule(HiChecker.RULE_CHECK_ABILITY_CONNECTION_LEAK)
```

### 场景 3：配置告警方式

```typescript
import HiChecker from '@ohos.hichecker'

// 方式1：记录日志
HiChecker.addRule(HiChecker.RULE_CAUTION_PRINT_LOG)

// 方式2：触发崩溃（强制退出）
HiChecker.addRule(HiChecker.RULE_CAUTION_TRIGGER_CRASH)
```

## 版本信息

| 版本 | 日期 | 变更 |
|------|------|------|
| 4.0 | 2025 | 支持 ArkUI 性能检测、JsLeakWatcher |
| 3.0 | 2024 | 重构 N-API 接口 |
| 2.0 | 2023 | 新增 Ability 泄露检测 |
| 1.0 | 2022 | 初始版本 |

## 相关文档

- [N-API 接口详解](02_NAPI.md)
- [ETS/ANI 接口](03_ETS_ANI.md)
- [内部架构](04_Architecture.md)
- [安全评审](06_Security.md)
