# ETS/ANI 接口

## 概述

HiChecker 为 ArkTS 应用提供了 **ANI (ArkTS Native Interface)** 接口，这是比 N-API 更高效的原生绑定方式。

## 模块信息

- **模块名**: `@ohos.hichecker.ani` (内部标识)
- **实现文件**: `interfaces/ets/ani/hichecker/src/ani_hichecker.cpp`
- **头文件**: `interfaces/ets/ani/hichecker/include/ani_hichecker.h`
- **产物**: `ani_hichecker_package` (bundle.json 中定义)

## API 清单

ANI 接口与 N-API 接口保持一致：

| ArkTS API | 参数类型 | 返回类型 | 对应 N-API |
|-----------|----------|----------|------------|
| `addRule(rule)` | bigint | void | `addRule` |
| `removeRule(rule)` | bigint | void | `removeRule` |
| `getRule()` | - | bigint | `getRule` |
| `contains(rule)` | bigint | boolean | `contains` |
| `addCheckRule(rule)` | bigint | void | `addCheckRule` |
| `removeCheckRule(rule)` | bigint | void | `removeCheckRule` |
| `containsCheckRule(rule)` | bigint | boolean | `containsCheckRule` |

## 规则常量

与 N-API 相同：

| 常量 | 值 | 描述 |
|------|-----|------|
| `HiChecker.RULE_CAUTION_PRINT_LOG` | `1n << 63n` | 日志告警 |
| `HiChecker.RULE_CAUTION_TRIGGER_CRASH` | `1n << 62n` | 崩溃告警 |
| `HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS` | `1n` | 线程耗时检测 |
| `HiChecker.RULE_CHECK_SLOW_EVENT` | `1n << 32n` | 进程耗时检测 |
| `HiChecker.RULE_CHECK_ABILITY_CONNECTION_LEAK` | `1n << 33n` | Ability泄露检测 |
| `HiChecker.RULE_CHECK_ARKUI_PERFORMANCE` | `1n << 34n` | ArkUI性能检测 |

## 使用示例

```typescript
// ArkTS 中使用 ANI 接口
import HiChecker from '@ohos.hichecker.ani'

// 添加检测规则
HiChecker.addCheckRule(HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS)

// 组合规则
let combinedRule = HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS | HiChecker.RULE_CHECK_ABILITY_CONNECTION_LEAK
HiChecker.addCheckRule(combinedRule)

// 检查规则是否存在
if (HiChecker.contains(HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS)) {
    console.log("耗时检测已开启")
}

// 获取所有规则
let currentRules = HiChecker.getRule()
console.log(`当前规则: ${currentRules}`)
```

## 实现原理

### ANI 注册

**文件**: `interfaces/ets/ani/hichecker/src/ani_hichecker.cpp`

```cpp
// ANI 接口使用 ANI 框架注册
// 相比 N-API，有更低的调用开销
static void AddRule(ani_env *env, ani_object rule)
{
    uint64_t ruleValue = GetRuleParam(env, rule);
    if (ruleValue != GET_RULE_PARAM_FAIL) {
        OHOS::HiviewDFX::HiChecker::AddRule(ruleValue);
    }
}
```

### 参数解析

```cpp
static uint64_t GetRuleParam(ani_env *env, ani_object rule)
{
    uint64_t value = 0;
    // ANI 参数解析
    ani::GetBigIntValue(env, rule, &value);
    return value;
}
```

## N-API vs ANI 对比

| 特性 | N-API | ANI |
|------|-------|-----|
| 调用开销 | 较高 | 较低 |
| 类型安全 | 运行时校验 | 编译期校验 |
| 支持语言 | JS/TS | ArkTS |
| 性能 | 普通 | 优化 |
| 使用场景 | 通用 | ArkTS 优先 |

## 注意事项

1. ANI 接口仅在 **ArkTS 运行环境** 下可用
2. 推荐 ArkTS 应用优先使用 ANI 接口以获得更好性能
3. API 行为与 N-API 完全一致
