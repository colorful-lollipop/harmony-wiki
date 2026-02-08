# 内部架构

## 整体架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                        应用层 (Application)                          │
├─────────────────────────────────────────────────────────────────────┤
│  JS/TS 接口层 (N-API)           │        ArkTS 接口层 (ANI)         │
│  @ohos/hichecker                │        @ohos/hichecker.ani       │
└──────────────┬──────────────────┴────────────────────┬──────────────┘
               │                                       │
               ▼                                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      接口适配层 (Interface)                          │
│  ┌─────────────────────┐           ┌─────────────────────────────┐  │
│  │ napi_hichecker.cpp  │           │   ani_hichecker.cpp        │  │
│  │ (N-API 实现)        │           │   (ANI 实现)               │  │
│  └──────────┬──────────┘           └──────────────┬──────────────┘  │
└─────────────┼────────────────────────────────────┼─────────────────┘
              │                                    │
              ▼                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     核心框架层 (Framework)                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    HiChecker.cpp                             │   │
│  │  - 规则管理 (AddRule/RemoveRule/Contains)                   │   │
│  │  - 告警处理 (HandleCaution/OnThreadCautionFound)           │   │
│  │  - 通知入口 (NotifySlowProcess/NotifySlowEvent)             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Caution.cpp                               │   │
│  │  - 告警数据结构                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────┬──────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
┌─────────────────────────┐   ┌───────────────────────────────────────┐
│  系统依赖层              │   │           js_leak_watcher           │
│  - hilog (日志)         │   │  ┌───────────────────────────────┐  │
│  - faultloggerd (回栈)  │   │  │ LeakWatcherEventHandler       │  │
│  - init (系统参数)      │   │  │ WindowLifeCycleListener      │  │
│  - ipc (进程间通信)     │   │  │ Heap Dump / GC Tasks         │  │
└─────────────────────────┘   └───────────────────────────────────────┘
```

## 核心模块

### HiChecker 类

**文件**: `frameworks/native/hichecker.cpp`

#### 关键成员

```cpp
// 静态成员变量
static std::mutex mutexLock_;              // 互斥锁
static volatile bool checkMode_;            // 检查模式
static volatile uint64_t processRules_;     // 进程级规则
static thread_local uint64_t threadLocalRules_; // 线程级规则
```

#### 职责说明

| 方法 | 职责 | 线程安全 |
|------|------|----------|
| `AddRule(rule)` | 添加检测/告警规则 | ✅ 互斥锁保护 |
| `RemoveRule(rule)` | 删除规则 | ✅ 互斥锁保护 |
| `GetRule()` | 获取当前规则 | ✅ 互斥锁保护 |
| `Contains(rule)` | 检查规则是否存在 | ✅ 互斥锁保护 |
| `NotifySlowProcess(tag)` | 通知线程耗时调用 | ⚠️ 读取 threadLocalRules_ |
| `NotifySlowEvent(tag)` | 通知进程耗时事件 | ⚠️ 读取 processRules_ |
| `NotifyAbilityConnectionLeak(caution)` | 通知 Ability 泄露 | ⚠️ 读取 processRules_ |
| `NotifyCaution(rule, tag, caution)` | 通用告警通知 | ⚠️ 读取规则 |

### 规则分类

```
                    ┌────────────────────┐
                    │     ALL_RULES      │
                    └─────────┬──────────┘
          ┌───────────────────┴───────────────────┐
          ▼                                       ▼
┌─────────────────────┐               ┌─────────────────────┐
│   ALL_THREAD_RULES │               │  ALL_PROCESS_RULES  │
│   (线程级规则)       │               │   (进程级规则)       │
│   - SLOW_PROCESS   │               │   - SLOW_EVENT     │
└─────────────────────┘               │   - ABILITY_LEAK   │
                                      │   - ARKUI_PERF      │
                                      └─────────────────────┘
                    ┌─────────────────┐
                    │ ALL_CAUTION_RULES│
                    │  (告警规则)       │
                    │  - PRINT_LOG    │
                    │  - TRIGGER_CRASH│
                    └─────────────────┘
```

### 告警处理流程

```
┌─────────────────────────────────────────────────────────────────┐
│                      告警处理流程                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NotifyXxx() 入口                                                 │
│       │                                                          │
│       ▼                                                          │
│  检查规则是否启用 (threadLocalRules_ / processRules_)            │
│       │                                                          │
│       ▼                                                          │
│  生成 Caution 对象                                                │
│   - 设置 triggerRule                                             │
│   - 设置 cautionMsg                                              │
│   - 调用 DumpStackTrace() 获取堆栈                               │
│       │                                                          │
│       ▼                                                          │
│  HandleCaution()                                                 │
│       │                                                          │
│       ▼                                                          │
│  判断是线程级还是进程级告警                                        │
│       │                                                          │
│       ▼                                                          │
│  OnThreadCautionFound() / OnProcessCautionFound()               │
│       │                                                          │
│       ▼                                                          │
│  根据告警规则执行响应                                             │
│   - RULE_CAUTION_PRINT_LOG → HILOG_INFO 打印日志               │
│   - RULE_CAUTION_TRIGGER_CRASH → kill(pid, SIGABRT) 触发崩溃  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 堆栈回溯实现

**文件**: `hichecker.cpp:211-216`

```cpp
void HiChecker::DumpStackTrace(std::string& msg)
{
    if (!GetBacktrace(msg)) {  // 依赖 faultloggerd:backtrace_local
        HILOG_INFO(LOG_CORE, "HiChecker DumpStackTrace fail.");
    }
}
```

## 线程模型

### 线程级规则 vs 进程级规则

| 特性 | 线程级规则 | 进程级规则 |
|------|-----------|-----------|
| 存储位置 | `thread_local` | 全局变量 |
| 作用域 | 单线程 | 进程内所有线程 |
| 适用场景 | UI 线程耗时检测 | Ability 泄露检测 |
| 规则 | `RULE_THREAD_CHECK_SLOW_PROCESS` | `RULE_CHECK_SLOW_EVENT` 等 |

### 线程安全

**证据**: `hichecker.cpp:44-47`

```cpp
std::mutex HiChecker::mutexLock_;
volatile bool HiChecker::checkMode_;
volatile uint64_t HiChecker::processRules_;
thread_local uint64_t HiChecker::threadLocalRules_;
```

- **写操作**: 使用 `std::mutex` 互斥锁保护
- **读操作**: 在单次检查中读取多个变量是安全的
- **线程局部存储**: `thread_local` 变量天然线程安全

## 系统参数配置

HiChecker 支持通过系统参数动态配置规则：

**文件**: `hichecker.cpp:227-254`

```cpp
void HiChecker::InitHicheckerParam(const char *processName)
{
    // 参数名格式: "hiviewdfx.hichecker.{processName}"
    char checkerName[QUERYNAME_LEN] = "hiviewdfx.hichecker.";
    strcat_s(checkerName, sizeof(checkerName), processName);
    
    // 读取参数值
    GetParameter(checkerName, defStrValue, paramOutBuf, PARAM_BUF_LEN);
    
    // 只允许 RULE_CHECK_ARKUI_PERFORMANCE
    if (!(rule & ALLOWED_RULE)) {
        HILOG_ERROR(LOG_CORE, "not allowed param.");
        return;
    }
    AddRule(rule & ALLOWED_RULE);
}
```

**参数格式**: `hiviewdfx.hichecker.{进程名}`
**允许的规则**: `RULE_CHECK_ARKUI_PERFORMANCE` (1ULL << 34)

## JsLeakWatcher 架构

### 组件构成

| 组件 | 职责 |
|------|------|
| `LeakWatcherEventHandler` | 事件处理循环，定期执行 Dump/GC |
| `WindowLifeCycleListener` | 窗口生命周期监听 |
| `registerArkUIObjectLifeCycleCallback` | ArkUI 对象生命周期回调 |

### 工作流程

```
用户调用 registerArkUIObjectLifeCycleCallback(cb)
                    │
                    ▼
┌─────────────────────────────────────────┐
│  LeakWatcherEventHandler                 │
│  - 保存回调引用                           │
│  - 启动定时 Dump 任务 (30s 间隔)          │
│  - 启动定时 GC 任务 (27s 间隔)           │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│  事件触发时                              │
│  - 执行 JS 回调函数                       │
│  - 传递相关对象/窗口信息                  │
└─────────────────────────────────────────┘
```

### 堆文件导出

**文件**: `js_leak_watcher_napi.cpp:346-371`

```cpp
static napi_value DumpRawHeap(napi_env env, napi_callback_info info)
{
    // 1. 创建文件
    CreateFile(filePath);
    
    // 2. 调用引擎 Dump 接口
    engine->DumpHeapSnapshot(filePath, true, DumpFormat::BINARY, false, true, true);
    
    // 3. 追加元数据
    AppendMetaData(filePath);
}
```

## 依赖关系

### 内部依赖

```
interfaces/native/innerkits
    │
    ▼
frameworks/native
    ├── hiccheckER.cpp ──► hilog (日志)
    │                   ──► faultloggerd (backtrace)
    │                   ──► init (GetParameter)
    │
    └── caution.cpp ─────► (无外部依赖)
```

### 外部依赖 (bundle.json)

- `hilog` - 日志系统
- `faultloggerd` - 崩溃日志
- `ipc` - 进程间通信
- `init` - 系统参数
- `eventhandler` - 事件处理
- `window_manager` - 窗口管理
- `ace_engine` - Ace 框架
