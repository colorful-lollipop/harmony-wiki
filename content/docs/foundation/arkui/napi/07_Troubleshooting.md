# 故障排查

## 常见问题

### Q1: 模块加载失败 "module not found"

**问题描述**：
```bash
error: Failed to load native module: @ohos/xxx
```

**可能原因**：

1. **模块未正确注册**
   
   检查模块是否调用了 `napi_module_register`：
   ```cpp
   extern "C" __attribute__((constructor)) void AppRegister()
   {
       napi_module_register(&appModule);
   }
   ```

2. **模块名不匹配**
   
   确保 `nm_modname` 与 JS 中 import 的名称一致：
   ```cpp
   static napi_module appModule = {
       .nm_modname = "app",  // import '@ohos/app'
       // ...
   };
   ```

3. **库未正确安装**
   
   检查 `BUILD.gn` 中的 `relative_install_dir`：
   ```gn
   ohos_shared_library("xxx") {
       relative_install_dir = "module"  // 安装到 /system/lib/module/
   }
   ```

**排查步骤**：
1. 检查模块是否在 `NativeModuleManager::loadedModules_` 缓存中
2. 检查 `dlopen` 是否成功（查看 dlsym 错误）
3. 检查模块路径是否正确

---

### Q2: 参数解析错误 "Invalid argument type"

**问题描述**：
```bash
error: Cannot convert arg to expected type
```

**可能原因**：

1. **参数类型不匹配**
   
   确保 JS 调用时参数类型正确：
   ```js
   // JS
   nativeModule.getValue("string");  // 传递 string
   ```
   ```cpp
   // C++
   napi_value arg;
   napi_get_value_string_utf8(env, argv[0], buffer, &len);  // 预期 string
   ```

2. **参数数量错误**
   
   检查 `napi_get_cb_info` 的 argc 参数：
   ```cpp
   size_t argc = 2;  // 期望 2 个参数
   napi_value argv[2];
   napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
   if (argc < 2) {
       napi_throw_error(env, "Expected 2 arguments");
       return nullptr;
   }
   ```

3. **参数越界**
   
   ```cpp
   size_t argc = 3;
   napi_value argv[3];
   napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
   // argc 实际值可能小于 3，需要检查
   ```

---

### Q3: 内存泄漏

**问题描述**：
```bash
Native module memory leak detected
```

**可能原因**：

1. **未释放的引用**
   
   ```cpp
   // 错误：创建引用后未释放
   napi_create_reference(env, value, 1, &ref);
   // ...
   // 忘记调用 napi_delete_reference(env, ref);
   ```

2. **Async Work 泄漏**
   
   ```cpp
   // 错误：未取消异步工作
   NativeAsyncWork* work = engine->CreateAsyncWork(...);
   work->Queue(engine);
   delete work;  // 必须在 CompleteCallback 中删除
   ```

3. **Wrap 数据泄漏**
   
   ```cpp
   // 错误：wrap 后 finalize 回调中未释放数据
   napi_wrap(env, jsObject, myData, nullptr, nullptr);
   // myData 永远不会被释放
   ```

**解决方案**：
1. 使用 RAII 模式管理资源
2. 在 finalize 回调中释放内存
3. 使用智能指针管理引用

---

### Q4: 异步任务无响应

**问题描述**：
```bash
Async work queued but never executes
```

**可能原因**：

1. **UV Loop 未运行**
   
   确保主线程事件循环在运行：
   ```cpp
   // 在主线程中
   engine->Loop(LOOP_DEFAULT);
   ```

2. **任务队列满**
   
   检查 `NativeAsyncWork` 队列状态

3. **线程池资源耗尽**
   
   libuv 线程池默认大小为 4，可通过环境变量修改：
   ```bash
   UV_THREADPOOL_SIZE=32 ./app
   ```

---

### Q5: Crash - Segmentation Fault

**问题描述**：
```bash
Segmentation fault (core dumped)
```

**可能原因**：

1. **无效的 napi_value**
   
   ```cpp
   // 错误：使用已失效的 napi_value
   napi_value result;
   napi_call_function(env, thisVar, func, 0, nullptr, &result);  // result 未初始化
   ```

2. **错误的引用层级**
   
   ```cpp
   // 错误：在错误的作用域中使用引用
   napi_open_handle_scope(env, &scope);
   napi_create_reference(env, value, 1, &ref);
   napi_close_handle_scope(env, scope);
   // ref 现在可能失效
   ```

3. **Null 检查缺失**
   
   ```cpp
   // 错误：未检查返回值
   napi_value result;
   napi_create_object(env, &result);
   napi_set_named_property(env, result, "key", nullptr);  // 崩溃
   ```

---

### Q6: Promise 无法 resolve

**问题描述**：
```js
// JS
nativeModule.asyncOp().then(() => {
    // 永远不被调用
});
```

**可能原因**：

1. **Deferred 未保存**
   
   ```cpp
   // 错误：deferred 是局部变量，函数返回后失效
   napi_value AsyncOp(napi_env env, napi_callback_info info) {
       napi_deferred deferred;
       napi_value promise;
       napi_create_promise(env, &deferred, &promise);
       
       // 启动异步操作...
       
       // deferred 是局部变量，必须通过其他方式保存
       return promise;
   }
   ```

2. **异步回调中未调用 resolve/reject**
   
   ```cpp
   void Execute(napi_env env, void* data) {
       // 异步执行...
       // 忘记调用 deferred->Resolve()
   }
   ```

3. **引擎状态错误**
   
   确保在正确的线程调用 resolve/reject

---

### Q7: 调试日志输出

**启用详细日志**：

```cpp
#include "utils/log.h"

// 在代码中添加日志
HILOG_INFO("Module loaded: %{public}s", moduleName);
HILOG_ERROR("Failed to load module: %{public}s", errorMsg);
```

**日志级别**：
- `HILOG_DEBUG` - 调试信息
- `HILOG_INFO` - 一般信息
- `HILOG_WARN` - 警告
- `HILOG_ERROR` - 错误
- `HILOG_FATAL` - 致命错误

---

## 调试技巧

### 使用断点

在 `ark_native_engine.cpp` 中设置断点：
- `RequireNapi()` - 模块加载入口
- `ArkNativeFunctionCallBack()` - JS 调用 native 入口

### 内存检查

```bash
# 使用 AddressSanitizer
./build.sh --asan --product-name xxx

# 使用 Valgrind
valgrind --leak-check=full ./app
```

### 线程分析

```bash
# 使用 ThreadSanitizer
./build.sh --tsan --product-name xxx
```

---

## 定位路径

### 模块加载问题定位

```
加载流程：
1. JS: import → arkNativeEngine.RequireNapi()
   位置: ark_native_engine.cpp:RequireNabi()
2. NativeModuleManager::LoadNativeModule()
   位置: native_module_manager.cpp
3. dlopen() 加载 .so
4. 调用 registerCallback()
5. 缓存到 loadedModules_
```

### 函数调用问题定位

```
调用流程：
1. JS 调用 native 函数
2. ArkNativeFunctionCallBack()
   位置: ark_native_engine.cpp
3. 用户回调函数
4. 返回值转换
```

---

## 相关文档

- [N-API 接口参考](./04_NAPI_Reference.md)
- [构建系统](./05_Build_System.md)
- [安全风险评审](./06_Security.md)
