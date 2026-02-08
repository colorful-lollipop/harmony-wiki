# 常见问题

> 构建、运行、调试问题与定位路径

## 目的与适用范围

### 目的
本文档提供 JSVM 的常见问题与解决方案，帮助开发者快速定位和解决问题。

### 适用范围
- 遇到构建问题的开发者
- 遇到运行时问题的开发者
- 需要调试的开发者

---

## 构建问题

### 1. V8 库未找到

**症状**:
```
error: cannot find -lv8_shared
```

**原因**: V8 库未正确复制到构建目录。

**证据位置**: `BUILD.gn:28-43` - copy_v8 action

**解决方案**:
1. 检查 V8 预编译库路径：`//vendor/${dependency_tag}/binary/artifacts/js_engine_url/`
2. 确保库文件存在：`libv8_shared.so`
3. 检查 copy_v8.sh 脚本是否正确执行
4. 清理构建目录并重新构建

**相关文件**:
- `BUILD.gn:28-43`
- `copy_v8.sh`

### 2. llhttp 编译失败

**症状**:
```
error: llhttp compilation failed
```

**原因**: llhttp 源码未正确复制或编译错误。

**证据位置**: `BUILD.gn:45-60` - copy_llhttp action

**解决方案**:
1. 检查 llhttp 源码是否复制到 `$target_gen_dir/llhttp/`
2. 检查 llhttp.c, http.c, api.c 是否存在
3. 检查编译器是否支持 C99 标准
4. 清理构建目录并重新构建

**相关文件**:
- `BUILD.gn:45-95`
- `copy_llhttp.sh`

### 3. libjsvm.so 链接失败

**症状**:
```
error: undefined reference to 'xxx'
```

**原因**: 缺少外部依赖或链接顺序错误。

**证据位置**: `BUILD.gn:122-135` - build_libjsvm external_deps

**解决方案**:
1. 检查所有外部依赖是否正确安装
2. 检查依赖顺序是否正确
3. 检查依赖库版本是否兼容
4. 使用 `ldd` 或 `readelf` 检查依赖关系

**相关文件**:
- `BUILD.gn:121-181`
- `build_jsvm.sh`

### 4. ASAN/HWASAN 构建失败

**症状**:
```
error: ASAN/HWASAN not supported
```

**原因**: 编译器不支持 ASAN/HWASAN 或配置错误。

**证据位置**: `BUILD.gn:108-112` - ASAN/HWASAN 配置

**解决方案**:
1. 检查编译器是否支持 ASAN/HWASAN
2. 检查 `support_hwasan` 是否为 true
3. 检查 `is_asan` 和 `use_hwasan` 配置
4. 禁用 ASAN/HWASAN 重新构建

**相关文件**:
- `BUILD.gn:108-112`
- `jsvm.gni:34` - support_hwasan

---

## 运行时问题

### 1. VM 初始化失败

**症状**:
```
OH_JSVM_Init failed: JSVM_GENERIC_FAILURE
```

**原因**: JSVM 已经初始化或初始化失败。

**证据位置**: `interface/kits/jsvm.h:323` - `OH_JSVM_Init()`
`src/js_native_api_v8.cpp:1059` - 实现

**解决方案**:
1. 检查是否已经初始化过 JSVM
2. 检查初始化选项是否正确
3. 检查 V8 引擎是否正确初始化
4. 使用 `OH_JSVM_GetLastErrorInfo()` 获取详细错误信息

**相关 API**:
- `OH_JSVM_Init()` - 初始化
- `OH_JSVM_GetLastErrorInfo()` - 获取错误信息

**证据位置**:
- `interface/kits/jsvm.h:323` - API 声明
- `interface/kits/jsvm.h:618` - 错误信息 API

### 2. Environment 创建失败

**症状**:
```
OH_JSVM_CreateEnv failed: JSVM_INVALID_ARG
```

**原因**: 参数无效或 VM 未正确创建。

**证据位置**: `interface/kits/jsvm.h:433` - `OH_JSVM_CreateEnv()`
`src/js_native_api_v8.cpp:1174` - 实现

**解决方案**:
1. 检查 VM 是否正确创建
2. 检查 propertyCount 和 properties 参数
3. 检查 properties 数组中的元素是否有效
4. 使用 `OH_JSVM_GetLastErrorInfo()` 获取详细错误信息

**相关 API**:
- `OH_JSVM_CreateEnv()` - 创建 Environment
- `OH_JSVM_GetLastErrorInfo()` - 获取错误信息

**证据位置**:
- `interface/kits/jsvm.h:433` - API 声明
- `interface/kits/jsvm.h:618` - 错误信息 API

### 3. 脚本编译失败

**症状**:
```
OH_JSVM_CompileScript failed: JSVM_PENDING_EXCEPTION
```

**原因**: JavaScript 代码有语法错误或运行时错误。

**证据位置**: `interface/kits/jsvm.h:501` - `OH_JSVM_CompileScript()`
`src/js_native_api_v8.cpp:1274` - 实现

**解决方案**:
1. 检查 JavaScript 代码语法
2. 使用 `OH_JSVM_GetAndClearLastException()` 获取异常信息
3. 检查 `eagerCompile` 参数是否正确
4. 使用 `OH_JSVM_CreateCodeCache()` 创建代码缓存加速编译

**相关 API**:
- `OH_JSVM_CompileScript()` - 编译脚本
- `OH_JSVM_GetAndClearLastException()` - 获取异常
- `OH_JSVM_CreateCodeCache()` - 创建代码缓存

**证据位置**:
- `interface/kits/jsvm.h:501` - API 声明
- `interface/kits/jsvm.h:742` - 异常 API

### 4. 引用计数错误

**症状**:
```
Segmentation fault / Use-after-free
```

**原因**: 引用计数管理错误，导致悬垂指针。

**证据位置**: `interface/kits/jsvm.h:823-904` - 引用管理 API
`src/jsvm_reference.cpp` - 引用实现

**解决方案**:
1. 使用 ASan/HWASan 检测内存错误
2. 检查 `OH_JSVM_CreateReference()` 和 `OH_JSVM_DeleteReference()` 是否配对
3. 检查 `OH_JSVM_ReferenceRef()` 和 `OH_JSVM_ReferenceUnref()` 是否平衡
4. 使用 Valgrind 检测内存错误

**相关 API**:
- `OH_JSVM_CreateReference()` - 创建引用
- `OH_JSVM_DeleteReference()` - 删除引用
- `OH_JSVM_ReferenceRef()` - 增加引用计数
- `OH_JSVM_ReferenceUnref()` - 减少引用计数

**证据位置**:
- `interface/kits/jsvm.h:823-904` - API 声明
- `src/jsvm_reference.cpp` - 实现

---

## 调试问题

### 1. Inspector 无法连接

**症状**:
```
OH_JSVM_OpenInspector failed / Chrome DevTools cannot connect
```

**原因**: Inspector 未正确打开或端口被占用。

**证据位置**: `interface/kits/jsvm.h:1719` - `OH_JSVM_OpenInspector()`
`src/inspector/js_native_api_v8_inspector.cpp` - Inspector 实现

**解决方案**:
1. 检查 `OH_JSVM_OpenInspector()` 是否成功
2. 检查端口是否被占用（如 9229）
3. 检查防火墙设置
4. 检查 Chrome DevTools 版本是否兼容
5. 使用 `OH_JSVM_WaitForDebugger()` 等待调试器连接

**相关 API**:
- `OH_JSVM_OpenInspector()` - 打开 Inspector
- `OH_JSVM_CloseInspector()` - 关闭 Inspector
- `OH_JSVM_WaitForDebugger()` - 等待调试器

**证据位置**:
- `interface/kits/jsvm.h:1719,1736,1747` - API 声明
- `src/inspector/inspector_socket_server.cpp` - WebSocket 服务器

### 2. 断点不生效

**症状**:
断点设置后但不停止执行。

**原因**: 断点位置错误或代码未加载。

**证据位置**: `src/inspector/js_native_api_v8_inspector.cpp` - Inspector 实现

**解决方案**:
1. 检查断点是否设置在可执行的代码行
2. 检查代码是否被正确加载
3. 检查 Source Map 是否正确配置
4. 使用 `OH_JSVM_CompileScriptWithOrigin()` 设置源信息

**相关 API**:
- `OH_JSVM_CompileScriptWithOrigin()` - 带源信息编译

**证据位置**:
- `interface/kits/jsvm.h:524` - API 声明

---

## 性能问题

### 1. 内存泄漏

**症状**:
进程内存持续增长。

**原因**: 引用计数泄漏或 Handle Scope 未正确关闭。

**证据位置**:
- `interface/kits/jsvm.h:762-811` - Handle Scope API
- `interface/kits/jsvm.h:823-904` - 引用管理 API

**解决方案**:
1. 使用 ASan/HWASan 检测内存泄漏
2. 检查 Handle Scope 是否正确配对
3. 检查引用计数是否正确管理
4. 使用 `OH_JSVM_GetHeapStatistics()` 监控内存使用
5. 使用 `OH_JSVM_TakeHeapSnapshot()` 分析堆快照

**相关 API**:
- `OH_JSVM_OpenHandleScope()` / `OH_JSVM_CloseHandleScope()`
- `OH_JSVM_CreateReference()` / `OH_JSVM_DeleteReference()`
- `OH_JSVM_GetHeapStatistics()` - 获取 Heap 统计
- `OH_JSVM_TakeHeapSnapshot()` - 获取堆快照

**证据位置**:
- `interface/kits/jsvm.h:762-904` - API 声明
- `interface/kits/jsvm.h:1661,1708` - 内存分析 API

### 2. 执行速度慢

**症状**: JavaScript 代码执行速度慢。

**原因**: 代码未优化或未使用 JIT。

**证据位置**:
- `interface/kits/jsvm.h:566` - 代码缓存 API
- `interface/kits/jsvm.h:1650` - 内存压力 API

**解决方案**:
1. 使用 `OH_JSVM_CreateCodeCache()` 创建代码缓存
2. 使用 `OH_JSVM_MemoryPressureNotification()` 通知内存压力
3. 使用 `OH_JSVM_StartCpuProfiler()` / `OH_JSVM_StopCpuProfiler()` 分析 CPU 性能
4. 启用 JIT（检查 `jit_enable_list_appid.conf`）
5. 使用 `OH_JSVM_CompileScriptWithOptions()` 设置编译选项

**相关 API**:
- `OH_JSVM_CreateCodeCache()` - 创建代码缓存
- `OH_JSVM_MemoryPressureNotification()` - 内存压力通知
- `OH_JSVM_StartCpuProfiler()` / `OH_JSVM_StopCpuProfiler()` - CPU Profiler
- `OH_JSVM_CompileScriptWithOptions()` - 带选项编译

**证据位置**:
- `interface/kits/jsvm.h:566,1650,1682,1693,550` - API 声明
- `jit_enable_list_appid.conf` - JIT 配置

---

## 定位路径

### 1. 使用错误信息

**方法**: 调用 `OH_JSVM_GetLastErrorInfo()` 获取详细错误信息。

**证据位置**: `interface/kits/jsvm.h:618` - API 声明

**示例**:
```c
JSVM_ExtendedErrorInfo* errorInfo = nullptr;
JSVM_Status status = OH_JSVM_GetLastErrorInfo(env, &errorInfo);
if (status == JSVM_OK && errorInfo) {
    printf("Error: %s\n", errorInfo->errorMessage);
    printf("Error Code: %d\n", errorInfo->errorCode);
}
```

### 2. 使用日志

**方法**: 检查 hilog 日志。

**证据位置**: `bundle.json:26` - hilog 依赖

**日志位置**:
- 系统日志：`/data/log/hilog/`
- 过滤关键字：`JSVM`, `V8`

### 3. 使用调试器

**方法**: 使用 Chrome DevTools 连接到 Inspector。

**证据位置**: `interface/kits/jsvm.h:1719` - `OH_JSVM_OpenInspector()`

**步骤**:
1. 调用 `OH_JSVM_OpenInspector(env, "0.0.0.0", 9229)`
2. 打开 Chrome 浏览器，访问 `chrome://inspect`
3. 点击 "Inspect" 连接到 Inspector

### 4. 使用内存分析工具

**方法**: 使用 ASan/HWASan、Valgrind 等工具。

**证据位置**: `BUILD.gn:108-112` - ASAN/HWASAN 支持

**工具**:
- ASan/HWASan - 检测内存错误
- Valgrind - 检测内存泄漏和错误
- Heap Snapshot - 分析堆内存

---

## 相关链接

- [对外 API 详细文档](./05_Public_API_Details.md) - API 使用
- [目录结构与模块职责](./02_Directory_Structure.md) - 构建配置
