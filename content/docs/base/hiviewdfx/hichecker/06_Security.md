# 安全风险评审

## 评审范围

本文档基于以下代码范围进行安全评审：

- `frameworks/native/` - Native 核心实现
- `interfaces/native/innerkits/` - Native 接口
- `interfaces/js/kits/napi/` - N-API 接口
- `interfaces/ets/ani/` - ANI 接口
- `bundle.json` - 组件配置

**不在评审范围内**:
- `test/` - 测试代码
- 第三方依赖（hilog, faultloggerd, ipc 等）

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              受信任区域 (HiChecker 组件内部)             │   │
│  │  - frameworks/native/                                   │   │
│  │  - interfaces/native/innerkits/                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ 边界                              │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              半信任区域 (系统框架)                        │   │
│  │  - hilog (日志系统)                                     │   │
│  │  - faultloggerd (崩溃日志)                              │   │
│  │  - ipc (进程间通信)                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ 边界                              │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              非信任区域 (应用层)                         │   │
│  │  - N-API 调用方                                         │   │
│  │  - 第三方应用                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流

```
应用调用 (N-API/ANI)
        │
        ▼
┌─────────────────┐
│  参数校验        │  ◄─── 攻击面 #1: 输入验证
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  规则管理        │  ◄─── 攻击面 #2: 竞态条件
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  告警处理        │
│  - HILOG_INFO   │  ◄─── 攻击面 #3: 日志注入
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│ kill()│ │ crash │  ◄─── 攻击面 #4: DoS
└───────┘ └───────┘
```

## 攻击面清单

| 攻击面 | 类型 | 位置 |
|--------|------|------|
| N-API 参数输入 | 输入验证 | `napi_hichecker.cpp` |
| 系统参数读取 | 配置注入 | `hichecker.cpp:InitHicheckerParam` |
| 回调函数执行 | 代码执行 | `js_leak_watcher_napi.cpp` |
| 堆栈信息泄露 | 信息泄露 | `hichecker.cpp:DumpStackTrace` |
| 规则状态竞态 | 竞态条件 | `hichecker.cpp` 规则操作 |

## 风险分析与修复建议

### 风险 #1: N-API 参数校验不完整

**严重程度**: 中

**证据**: `napi_hichecker.cpp:161-187`

```cpp
uint64_t GetRuleParam(napi_env env, napi_callback_info info)
{
    // ... 参数提取 ...
    if (!MatchValueType(env, argv[ARRAY_INDEX_FIRST], napi_bigint)) {
        return GET_RULE_PARAM_FAIL;  // 只返回失败，无异常抛出
    }
    // ...
    napi_get_value_bigint_uint64(env, argv[ARRAY_INDEX_FIRST], &rule, &lossless);
    if (!lossless) {
        return GET_RULE_PARAM_FAIL;
    }
    // 无白名单校验，可能传入任意位模式
    return rule;
}
```

**问题**:
- 允许传入任意 uint64 值
- 虽然 `HiChecker::AddRule` 有 `CheckRule` 校验，但错误处理静默
- 没有抛异常或返回明确错误码给 JS

**触发**:
```typescript
// 传入任意大整数（即使不是有效规则）
HiChecker.addRule(0xFFFFFFFFFFFFFFFFn)  // 静默失败
```

**影响**:
- JS 层无法区分参数错误和规则不存在
- 开发者可能误以为规则已添加
- 调试困难

**修复建议**:
```cpp
// 方案1: ThrowError 返回明确错误码
uint64_t GetRuleParam(napi_env env, napi_callback_info info)
{
    // ... 提取参数 ...
    if (!MatchValueType(env, argv[0], napi_bigint)) {
        ThrowError(env, ERR_PARAM_INVALID_TYPE);
        return GET_RULE_PARAM_FAIL;
    }
    // 校验是否在允许的规则范围内
    if (rule != GET_RULE_PARAM_FAIL && rule != 0 && 
        (rule & ~Rule::ALL_RULES)) {
        ThrowError(env, ERR_PARAM_UNKNOWN_RULE);
        return GET_RULE_PARAM_FAIL;
    }
    return rule;
}
```

**状态**: ⚠️ 建议修复

---

### 风险 #2: 规则状态竞态条件

**严重程度**: 低

**证据**: `hichecker.cpp:44-47`

```cpp
std::mutex HiChecker::mutexLock_;
volatile bool HiChecker::checkMode_;
volatile uint64_t HiChecker::processRules_;
thread_local uint64_t HiChecker::threadLocalRules_;
```

**问题**:
- `AddRule`/`RemoveRule` 使用互斥锁
- 但 `NotifyXxx` 函数只读取规则状态
- 多线程并发场景下可能观察到不一致的状态

**触发**:
```cpp
// 线程 A: RemoveRule(RULE_THREAD_CHECK_SLOW_PROCESS)
// 线程 B: NotifySlowProcess()  // 同时发生
```

**影响**:
- 可能错过告警或产生重复告警
- 不会导致安全问题，但可能影响检测准确性

**修复建议**:
```cpp
void HiChecker::NotifySlowProcess(const std::string& tag)
{
    // 使用原子读取或添加读写锁
    uint64_t localRules = atomic_load(&threadLocalRules_);
    if ((localRules & Rule::RULE_THREAD_CHECK_SLOW_PROCESS) == 0) {
        return;
    }
    // ...
}
```

**状态**: ⚠️ 建议优化

---

### 风险 #3: 日志注入

**严重程度**: 低

**证据**: `hichecker.cpp:186-189`

```cpp
void HiChecker::PrintLog(const CautionDetail& cautionDetail)
{
    HILOG_INFO(LOG_CORE,
        "HiChecker caution with RULE_CAUTION_PRINT_LOG.\nCautionMsg:%{public}s\nStackTrace:\n%{public}s",
        cautionDetail.caution_.GetCautionMsg().c_str(),
        cautionDetail.caution_.GetStackTrace().c_str());
}
```

**问题**:
- `cautionMsg` 和 `stackTrace` 直接拼接到日志字符串
- 虽然 HiChecker 内部控制输入，但仍属不良实践

**触发**:
```cpp
// HiChecker 内部触发
HiChecker::NotifySlowProcess("tag\nSensitiveInfo:secret");
```

**影响**:
- 可能伪造日志条目
- 可能导致日志解析工具解析错误

**修复建议**:
```cpp
// 对日志内容进行转义或过滤
std::string SanitizeLog(const std::string& input) {
    std::string output;
    for (char c : input) {
        if (c == '\n' || c == '\r' || c == '%') {
            output += ' ';
        } else {
            output += c;
        }
    }
    return output;
}
```

**状态**: ⚠️ 建议优化

---

### 风险 #4: 回调函数代码执行

**严重程度**: 中

**证据**: `js_leak_watcher_napi.cpp:79-89`

```cpp
void ExecuteJsFunc(napi_ref callbackRef)
{
    napi_handle_scope scope = nullptr;
    napi_open_handle_scope(env_, &scope);
    // ... 获取全局和回调 ...
    napi_call_function(env_, global, callback, 1, argv, nullptr);  // 调用 JS
    napi_close_handle_scope(env_, scope);
}
```

**问题**:
- 应用提供的 JS 回调可能在任意线程执行
- 回调中可能执行任意 JS 代码

**触发**:
```typescript
// 应用注册恶意回调
registerArkUIObjectLifeCycleCallback(() => {
    // 执行任意操作
});
```

**影响**:
- 如果回调中包含恶意代码，可能影响 HiChecker 进程
- 可能导致拒绝服务（回调无限循环）

**缓解措施**:
- 回调在 JS 引擎中执行，受引擎沙箱保护
- 通过 `EventRunner` 机制控制执行时机

**状态**: ✅ 已缓解（JS 引擎沙箱）

---

### 风险 #5: 系统参数注入

**严重程度**: 低

**证据**: `hichecker.cpp:227-254`

```cpp
void HiChecker::InitHicheckerParam(const char *processName)
{
    char checkerName[QUERYNAME_LEN] = "hiviewdfx.hichecker.";
    strcat_s(checkerName, sizeof(checkerName), processName);  // processName 拼接
    
    // 读取参数
    GetParameter(checkerName, defStrValue, paramOutBuf, PARAM_BUF_LEN);
    
    // 只允许 ARKUI_PERFORMANCE
    if (!(rule & ALLOWED_RULE)) {
        return;  // 非 ARKUI 规则被静默忽略
    }
    AddRule(rule & ALLOWED_RULE);
}
```

**问题**:
- `processName` 直接拼接，无长度检查
- 但 `processName` 通常来自系统传递，可控性有限

**缓解措施**:
- 使用 `strcat_s` 限制最大长度
- 只允许 `RULE_CHECK_ARKUI_PERFORMANCE` (1 << 34)
- 额外的规则被过滤

**状态**: ✅ 已缓解

---

### 风险 #6: 堆栈信息泄露

**严重程度**: 无

**证据**: `hichecker.cpp:95-99`

```cpp
void HiChecker::NotifySlowProcess(const std::string& tag)
{
    if ((threadLocalRules_ & Rule::RULE_THREAD_CHECK_SLOW_PROCESS) == 0) {
        return;  // 提前返回，无堆栈泄露
    }
    std::string stackTrace;
    DumpStackTrace(stackTrace);  // 仅在启用时调用
    Caution caution(...);
}
```

**分析**:
- 堆栈仅在检测规则启用时才会被获取
- 默认情况下不会泄露任何信息
- 适用于所有 `NotifyXxx` 函数

**状态**: ✅ 安全

---

## 未涉及的安全风险

以下风险类型经检查确认**不涉及**：

| 风险类型 | 检查结论 | 证据 |
|----------|----------|------|
| 路径遍历 | ✅ 不涉及 | 无文件路径操作 |
| SQL 注入 | ✅ 不涉及 | 无数据库操作 |
| 内存溢出 | ✅ 不涉及 | 使用智能指针/string |
| 缓冲区溢出 | ✅ 不涉及 | 使用 `strcat_s` 等安全函数 |
| 动态加载 | ✅ 不涉及 | 无 dlopen/dlsym |
| 网络通信 | ✅ 不涉及 | 无 socket 操作 |
| 敏感数据存储 | ✅ 不涉及 | 无文件/DB 存储 |

## 安全最佳实践

### 1. 输入验证

✅ **已实现**: N-API 参数类型检查

```cpp
// napi_hichecker.cpp:172-175
if (!MatchValueType(env, argv[ARRAY_INDEX_FIRST], napi_bigint)) {
    return GET_RULE_PARAM_FAIL;
}
```

### 2. 线程安全

✅ **已实现**: 互斥锁保护

```cpp
// hichecker.cpp:51-52
void HiChecker::AddRule(uint64_t rule)
{
    std::lock_guard<std::mutex> lock(mutexLock_);
    // ...
}
```

### 3. 内存安全

✅ **已实现**: C++ RAII + string

```cpp
// caution.h:19-43
class Caution {
    // 使用 std::string 自动管理内存
    std::string cautionMsg_;
    std::string stackTrace_;
};
```

### 4. 最小权限

✅ **已实现**: 只允许特定规则

```cpp
// hichecker.cpp:42
constexpr uint64_t ALLOWED_RULE = Rule::RULE_CHECK_ARKUI_PERFORMANCE;
```

## 总结

| 风险 | 严重程度 | 状态 |
|------|----------|------|
| N-API 参数校验不完整 | 中 | 建议修复 |
| 规则状态竞态条件 | 低 | 建议优化 |
| 日志注入 | 低 | 建议优化 |
| 回调函数代码执行 | 中 | 已缓解 |
| 系统参数注入 | 低 | 已缓解 |
| 堆栈信息泄露 | 无 | 安全 |

**总体评估**: HiChecker 组件安全状况良好，已实现基本的安全最佳实践。建议优化 N-API 参数校验和竞态条件处理。
