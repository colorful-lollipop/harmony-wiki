# 附录 A：关键调用链

## 1. 模块加载调用链

```
JS Import
    │
    ▼
ArkNativeEngine::RequireNapi()              // ark_native_engine.cpp
    │
    ├── 参数验证
    ├── 检查模块白名单 (受限 Worker)
    └── 返回: 模块 exports
    │
    ▼
NativeModuleManager::LoadNativeModule()     // native_module_manager.cpp
    │
    ├── 检查 loadedModules_ 缓存
    │   └── 找到 → 返回缓存模块
    │
    ├── GetNativeModulePath()               // 查找模块路径
    │
    ├── dlopen() 加载 .so                   // 平台相关
    │
    ├── dlsym("NMregister_API")            // 获取注册函数
    │
    ├── 调用 registerCallback()              // 模块初始化
    │   └── AppExport()                    // 用户代码
    │       └── napi_define_properties()   // 定义导出
    │
    └── 缓存到 loadedModules_
    │
    ▼
返回 exports 给 JS
```

## 2. JS 调用 Native 函数调用链

```
JS: nativeModule.method(arg1, arg2)
    │
    ▼
ArkNativeEngine::CallFunction()             // ark_native_engine.cpp
    │
    ▼
ArkNativeFunctionCallBack()                 // ark_native_engine.cpp (模板)
    │
    ├── 提取 NapiFunctionInfo
    ├── 启动 HiTrace 追踪
    ├── 进入 JsiNativeScope
    │
    ▼
用户定义的 NapiNativeCallback               // 用户代码
    │
    ├── 从 JsiRuntimeCallInfo 获取参数
    ├──执行业务逻辑
    │
    ▼
返回 napi_value
    │
    ├── 转换结果为 Local<JSValueRef>
    └── 返回给 JS VM
```

## 3. Async Work 调用链

```
NativeAsyncWork::CreateAsyncWork()         // native_async_work.cpp
    │
    ├── 保存 execute callback
    ├── 保存 complete callback
    └── 返回 NativeAsyncWork*
    │
    ▼
NativeAsyncWork::Queue(engine)             // native_async_work.cpp
    │
    ├── 初始化 uv_work_t
    ├── uv_queue_work()                    // libuv
    │   └── 提交到线程池
    │
    ▼
[线程池执行]
    │
    ▼
NativeAsyncWork::AsyncWorkCallback()       // native_async_work.cpp
    │
    ├── 提取 NativeAsyncWork*
    ├── 调用 ExecuteCallback()              // 用户代码
    │   └── 执行耗时操作
    │
    └── 提交 CompleteCallback 到主线程
    │
    ▼
[主线程事件循环]
    │
    ▼
NativeAsyncWork::AsyncAfterWorkCallback()  // native_async_work.cpp
    │
    ├── 验证引擎状态
    ├── 调用 CompleteCallback()             // 用户代码
    │   └── napi_resolve_deferred()       // 或 reject
    │
    └── uv_close() 清理 uv_work_t
    │
    ▼
NativeAsyncWork::~NativeAsyncWork()        // 析构
    └── napi_delete_async_work()
```

## 4. Promise 调用链

```
napi_create_promise()                       // 标准 N-API
    │
    ├── 创建 Promise 对象
    ├── 创建 Deferred 对象
    └── 返回 promise, deferred
    │
    ▼
返回 promise 给 JS
    │
    ▼
[JS 端等待]
    │
    ▼
deferred->Resolve(data)                   // ArkNativeDeferred::Resolve()
    或
deferred->Reject(reason)                   // ArkNativeDeferred::Reject()
    │
    ├── 验证引擎状态
    ├── 调用 JSNApi::ResolvePromise()
    └── 触发 JS 端回调
```

## 5. 引用管理调用链

```
napi_create_reference()                     // 标准 N-API
    │
    ├── 创建 NativeReference
    ├── 设置初始引用计数
    └── 返回 napi_ref
    │
    ▼
[引用计数管理]
    │
    ├── napi_reference_ref()
    │   └── 引用计数++
    │
    └── napi_reference_unref()
        └── 引用计数--
        └── 如果计数为 0 → 设置为弱引用
    │
    ▼
[GC 触发]
    │
    ├── NativeReference::FinalizeCallback()
    │   └── 如果是 RUNTIME 拥有 → 释放
    │
    └── NativeReferenceManager::ReleaseHandler()
```

## 6. Native 调用 JS 回调调用链

```
Native 代码
    │
    ▼
napi_call_function()                        // 标准 N-API
    │
    ├── 参数验证
    ├── 转换参数到 Local<JSValueRef>
    │
    ▼
ArkNativeEngine::CallFunction()             // ark_native_engine.cpp
    │
    ├── JSNApi::CallFunction()
    │   └── VM 层调用
    │
    └── 返回结果
    │
    ▼
转换结果到 napi_value
```

## 7. 模块卸载调用链

```
NativeModuleManager::UnloadNativeModule()   // native_module_manager.cpp
    │
    ├── 检查引用计数 (refCount)
    │   └── 如果 > 0 → 延迟卸载
    │
    ├── 设置 moduleLoaded = false
    │
    ├── 调用模块卸载回调 (如果有)
    │
    └── 从 loadedModules_ 移除
    │
    ▼
如果 refCount == 0:
    │
    ├── dlclose() 卸载 .so
    └── 释放 NativeModule 结构
```
