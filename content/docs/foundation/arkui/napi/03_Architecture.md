# 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Application Layer                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   JS App    │  │   Worker    │  │  Task Pool  │  │  Native Modules │ │
│  │   (Main)    │  │  (Thread)   │  │  (Threads)  │  │   (.so files)   │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └─────────────────┘ │
└─────────┼────────────────┼────────────────┼─────────────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         NAPI Layer (this repo)                           │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                      NativeEngine (Abstract)                         │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │ │
│  │  │ AsyncWork   │  │  Reference  │  │  Deferred   │  │SafeAsync│ │ │
│  │  │ Manager     │  │  Manager    │  │  (Promise)  │  │Work(TSFN)│ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │              ArkNativeEngine (Ark Implementation)                    │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │ │
│  │  │  EcmaVM*    │  │ Global Refs │  │ Module Cache│  │Contexts │ │ │
│  │  │  (JS Heap)  │  │ (Handles)   │  │ (loaded_)   │  │(Multi-env)│ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         Ark Runtime (ecmascript)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   JSI API   │  │   GC        │  │  Compiler   │  │  Bytecode Exec │ │
│  │  (JSNApi)   │  │  (Heap)     │  │  (AOT/JIT)  │  │  (Interpreter) │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         System Services                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │  libuv      │  │  HiTrace    │  │  EventHandler│  │  ContainerScope│ │
│  │  (async I/O)│  │  (tracing)  │  │  (UI thread) │  │  (UI context) │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## 核心组件

### 1. NativeEngine 抽象层

**职责**：JS 引擎抽象层，统一 JS 引擎在 N-API 层的接口行为

**证据**：`native_engine.h:166`

```cpp
class NAPI_EXPORT NativeEngine {
public:
    explicit NativeEngine(void* jsEngine);
    explicit NativeEngine(NativeEngine* parent);
    virtual ~NativeEngine();
    
    virtual NativeModuleManager* GetModuleManager();
    virtual NativeReferenceManager* GetReferenceManager();
    virtual NativeCallbackScopeManager* GetCallbackScopeManager();
    virtual uv_loop_t* GetUVLoop() const;
    // ... 更多接口
};
```

### 2. ArkNativeEngine 实现

**职责**：基于 Ark（panda ecmascript）引擎的具体实现

**证据**：`ark_native_engine.h:16-150`

```cpp
class ArkNativeEngine : public NativeEngine {
private:
    EcmaVM* vm_;                          // Ark JS VM
    Global<JSValueRef> context_;           // JS 全局上下文
    ArkNativeEngine* parentEngine_;        // 父引擎（用于 context）
    ArkNativeEngineState engineState_;      // 引擎状态
    // ...
};
```

### 3. ModuleManager

**职责**：管理模块加载、模块信息缓存

**证据**：`native_module_manager.h:82-150`

```cpp
class NAPI_EXPORT NativeModuleManager {
public:
    static NativeModuleManager* GetInstance();
    void Register(NativeModule* nativeModule);
    NativeModule* LoadNativeModule(const char* moduleName, const char* path, 
                                  bool isAppModule, std::string& errInfo, 
                                  bool internal = false, const char* relativePath = "");
};
```

### 4. ScopeManager

**职责**：管理 NativeValue 的生命周期

**注意**：当前代码标记为废弃

### 5. ReferenceManager

**职责**：管理 NativeReference 的生命周期

**证据**：`reference_manager.h:22-32`

```cpp
class NAPI_EXPORT NativeReferenceManager {
public:
    void CreateHandler(NativeReference* reference);
    void ReleaseHandler(NativeReference* reference);
};
```

## 线程模型

### 线程类型

**证据**：`native_engine.h:320-326`

```cpp
enum JSThreadType {
    MAIN_THREAD,           // 主应用线程
    WORKER_THREAD,         // 标准 Web Worker
    RESTRICTEDWORKER_THREAD, // 受限能力 Worker
    TASKPOOL_THREAD,      // Task pool 线程
    FORM_THREAD,           // 组件/卡片线程
    NATIVE_THREAD         // 纯 native 线程
};
```

### 线程架构

```
┌─────────────────────────────────────────────────────────┐
│                    Main Thread                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │  EcmaVM     │  │  UV Loop   │  │  NativeEngine   │ │
│  │  (JS Heap)  │  │  (async)   │  │  (napi_env)     │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Worker Thread │   │  Task Pool    │   │  Background   │
│ (Owns VM)     │   │  (Shared VM)  │   │  UV Workers   │
└───────────────┘   └───────────────┘   └───────────────┘
```

### 关键线程机制

| 机制 | 用途 | 证据位置 |
|------|------|----------|
| `uv_queue_work` | 线程池执行 CPU 密集任务 | `native_async_work.h:79-80` |
| `uv_async_t` | 线程安全的事件循环通知 | `native_async_work.h:82` |
| `NativeSafeAsyncWork` | 线程安全函数调用（TSFN） | `native_safe_async_work.h` |
| `WorkerThreadState` | 追踪 Worker 空闲/运行状态 | `native_engine.h:72-106` |

## 数据流

### 模块加载流程

```
JS: import xxx from '@ohos/xxx'
        │
        ▼
┌─────────────────┐
│  RequireNapi()  │  (ArkNativeEngine 静态方法)
│  - 参数解析      │
│  - 白名单检查    │
└────────┬────────┘
         ▼
┌─────────────────────────┐
│ NativeModuleManager::LoadNativeModule() │
│  - 检查缓存 (loadedModules_)            │
│  - dlopen 加载 .so                      │
│  - 调用 registerCallback()              │
└────────┬─────────────────────────┘
         ▼
┌─────────────────────────┐
│  Module Init Function  │  (如 AppExport)
│  - 创建 exports 对象   │
│  - 定义属性/函数       │
│  - 返回 exports       │
└────────┬─────────────────────────┘
         ▼
   缓存到 loadedModules_
         │
         ▼
   返回给 JS
```

### JS 到 Native 调用流程

```
JS 调用 native 函数
        │
        ▼
┌─────────────────────────────────┐
│ ArkNativeFunctionCallBack       │  (模板回调)
│  - 提取 NapiFunctionInfo        │
│  - 启动 profiler 追踪           │
│  - 进入 JsiNativeScope          │
└───────────┬─────────────────────┘
            ▼
┌─────────────────────────────────┐
│  User's NapiNativeCallback     │
│  (env, JsiRuntimeCallInfo*)    │
│  - 通过 callInfo 访问参数        │
│  - 返回 napi_value              │
└───────────┬─────────────────────┘
            ▼
   转换结果为 Local<JSValueRef>
            │
            ▼
   返回给 JS VM
```

## 关键时序

### Async Work 时序

```
Native 模块                          JS 线程 (Main)          UV 线程池
    │                                    │                       │
    ├──────────────────────────────────►│                       │
    │  CreateAsyncWork()                 │                       │
    │                                    │                       │
    ├──────────────────────────────────►│                       │
    │  Queue(engine)                    │                       │
    │  (uv_queue_work)                  │                       │
    │                                    │                       │
    │                                    │      ┌───────────────┼───►
    │                                    │      │ ExecuteCallback│   │
    │                                    │      │ (后台执行)     │   │
    │                                    │      │               │   │
    │                                    │      └───────────────┼───►
    │                                    │                       │  完成
    │                                    │◄──────────────────────┘
    │                                    │  AsyncAfterWorkCallback
    │                                    │  (CompleteCallback)
    │                                    │                       │
```

### Promise 流程

```
Native 模块                          JS 线程
    │                                    │
    ├──────────────────────────────────►│
    │  CreatePromise(&deferred)         │
    │  (返回 promise 给 JS)             │
    │                                    │    JS: promise.then()
    │                                    │         │
    │                                    │         ▼
    │                                    │    Pending
    │                                    │         │
    │  deferred->Resolve(value)  ───────┼───────►│
    │  (或 Reject(reason))              │    JS callbacks
```

## 多 Context 支持

**Context 引擎**（受限环境，用于 Worker 等）：

```
父引擎 (Main)              Context 引擎 (Child)
─────────────── ─────────────────────
拥有 UV Loop              无 UV Loop（使用父引擎）
         拥有 EcmaVM              共享 EcmaVM
完整能力                  受限（无 app 模块）
                         需要模块验证回调
```

**证据**：`ark_native_engine.h:64-68`

```cpp
// Context 引擎创建
static ArkNativeEngine* New(ArkNativeEngine* parent, 
                            EcmaVM* vm, 
                            Local<ContextRef> context);
```

---

**相关文档**：
- [N-API 接口参考](./04_NAPI_Reference.md)
- [目录结构](./02_Directory_Structure.md)
- [故障排查](./07_Troubleshooting.md)
