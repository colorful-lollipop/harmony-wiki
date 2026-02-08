# N-API 接口

## 概述

HiChecker 提供了两套 N-API 接口模块：

1. **HiChecker 模块** (`@ohos/hichecker`) - 核心检测功能
2. **JsLeakWatcher 模块** (`jsLeakWatcherNative`) - 内存泄漏检测

## HiChecker 模块

### 模块注册

**文件**: `interfaces/js/kits/napi/src/napi_hichecker.cpp:196-209`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = DeclareHiCheckerInterface,
    .nm_modname = "hichecker",
    .nm_priv = ((void *)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void HiCheckerRegisterModule(void)
{
    napi_module_register(&g_module);
}
```

**导入方式**:
```typescript
import HiChecker from '@ohos.hichecker'
```

### API 清单

#### 规则管理

| JS API | 参数 | 返回值 | C++ 实现 |
|--------|------|--------|----------|
| `addRule(rule)` | BigInt: 规则值 | void | `napi_hichecker.cpp:37` |
| `removeRule(rule)` | BigInt: 规则值 | void | `napi_hichecker.cpp:46` |
| `getRule()` | - | BigInt: 当前规则 | `napi_hichecker.cpp:55` |
| `contains(rule)` | BigInt: 规则值 | boolean | `napi_hichecker.cpp:61` |
| `addCheckRule(rule)` | BigInt: 规则值 | void | `napi_hichecker.cpp:69` |
| `removeCheckRule(rule)` | BigInt: 规则值 | void | `napi_hichecker.cpp:80` |
| `containsCheckRule(rule)` | BigInt: 规则值 | boolean | `napi_hichecker.cpp:91` |

#### 规则常量

| 常量名 | 值 | 描述 |
|--------|-----|------|
| `RULE_CAUTION_PRINT_LOG` | `1n << 63n` | 告警规则：打印日志 |
| `RULE_CAUTION_TRIGGER_CRASH` | `1n << 62n` | 告警规则：触发崩溃 |
| `RULE_THREAD_CHECK_SLOW_PROCESS` | `1n` | 检测规则：线程耗时调用 |
| `RULE_CHECK_SLOW_EVENT` | `1n << 32n` | 检测规则：进程耗时事件 |
| `RULE_CHECK_ABILITY_CONNECTION_LEAK` | `1n << 33n` | 检测规则：Ability连接泄露 |
| `RULE_CHECK_ARKUI_PERFORMANCE` | `1n << 34n` | 检测规则：ArkUI性能 |

### 参数校验

**文件**: `napi_hichecker.cpp:161-187`

```cpp
uint64_t GetRuleParam(napi_env env, napi_callback_info info)
{
    size_t argc = ONE_VALUE_LIMIT;  // 限制1个参数
    napi_value argv[ONE_VALUE_LIMIT] = { nullptr };
    // ...
    if (!MatchValueType(env, argv[ARRAY_INDEX_FIRST], napi_bigint)) {
        // 必须是大整数类型
        return GET_RULE_PARAM_FAIL;
    }
    uint64_t rule = GET_RULE_PARAM_FAIL;
    bool lossless = true;
    napi_get_value_bigint_uint64(env, argv[ARRAY_INDEX_FIRST], &rule, &lossless);
    if (!lossless) {
        // 必须是64位无损转换
        return GET_RULE_PARAM_FAIL;
    }
    return rule;
}
```

**校验规则**:
1. 参数数量必须为 1
2. 参数类型必须是 BigInt
3. BigInt 必须能无损转换为 uint64

### 错误码

| 错误码 | 描述 | 触发条件 |
|--------|------|----------|
| 401 | 参数错误 | 参数数量不对或类型不匹配 |

**错误消息**: `napi_hichecker.cpp:153`

```cpp
std::map<int, std::string> errMap = {
    { ERR_PARAM, "Invalid input parameter! only one bigint type parameter is needed" },
};
```

## JsLeakWatcher 模块

### 模块注册

**文件**: `interfaces/js/kits/napi/js_leak_watcher/js_leak_watcher_napi.cpp:390-399`

```cpp
static napi_module _module = {
    .nm_version = 0,
    .nm_filename = nullptr,
    .nm_register_func = DeclareJsLeakWatcherInterface,
    .nm_modname = "jsLeakWatcherNative",
};

extern "C" __attribute__((constructor)) void NAPI_hiviewdfx_jsLeakWatcher_AutoRegister()
{
    napi_module_register(&_module);
}
```

### API 清单

| JS API | 功能描述 |
|--------|----------|
| `registerArkUIObjectLifeCycleCallback(cb)` | 注册 ArkUI 对象生命周期回调 |
| `unregisterArkUIObjectLifeCycleCallback()` | 注销 ArkUI 对象生命周期回调 |
| `registerWindowLifeCycleCallback(cb)` | 注册窗口生命周期回调 |
| `unregisterWindowLifeCycleCallback()` | 注销窗口生命周期回调 |
| `handleDumpTask(cb)` | 启动 Heap Dump 任务 |
| `handleGCTask(cb)` | 启动 GC 任务 |
| `handleShutdownTask(cb)` | 启动关闭任务 |
| `removeTask()` | 移除所有任务 |
| `dumpRawHeap(filePath)` | 导出 Raw Heap 快照 |

### 调用链示例

```
JS: HiChecker.addCheckRule(HiChecker.RULE_THREAD_CHECK_SLOW_PROCESS)
    ↓
N-API: napi_hichecker.cpp:69 (AddCheckRule)
    ↓
Native: HiChecker::AddRule()
    ↓
    └─→ 规则写入 threadLocalRules_ / processRules_
```

## 线程模型

### N-API 调用线程

- N-API 回调在 **JS 线程** 执行
- 回调中调用 `HiChecker` API 是 **同步阻塞** 的
- 内部实现使用 `std::mutex` 保证线程安全

**证据**: `napi_hichecker.cpp:37-44`

```cpp
napi_value AddRule(napi_env env, napi_callback_info info)
{
    uint64_t rule = GetRuleParam(env, info);
    if (rule != GET_RULE_PARAM_FAIL) {
        HiChecker::AddRule(rule);  // 同步调用
    }
    return CreateUndefined(env);
}
```

## 注意事项

1. 所有 API 都是 **同步** 的，不返回 Promise
2. BigInt 参数必须使用 `n` 后缀
3. 规则值使用 **位掩码**，可以组合使用
