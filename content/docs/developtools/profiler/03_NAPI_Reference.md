# 03_NAPI_Reference - N-API 接口参考

> **模块**: `@ohos.hidebug`  
> **源码位置**: `hidebug/interfaces/js/kits/napi/`  
> **注册方式**: `napi_module_register` (行 1145)

---

## 1. API 清单

### 1.1 CPU 性能分析

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `startProfiling(fileName)` | `StartProfiling` | `fileName: string` | `undefined` | 同步 |
| `stopProfiling()` | `StopProfiling` | - | `undefined` | 同步 |
| `startJsCpuProfiling(fileName)` | `StartJsCpuProfiling` | `fileName: string` | `undefined` | 同步 |
| `stopJsCpuProfiling()` | `StopJsCpuProfiling` | - | `undefined` | 同步 |

### 1.2 堆内存分析

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `dumpHeapData(fileName)` | `DumpHeapData` | `fileName: string` | `undefined` | 同步 |
| `dumpJsHeapData(fileName)` | `DumpJsHeapData` | `fileName: string` | `undefined` | 同步 |
| `dumpJsRawHeapData(isGc)` | `DumpJsRawHeapData` | `isGc: boolean` | `Promise<string>` | 异步 |
| `getAppVMObjectUsedSize()` | `GetAppVMObjectUsedSize` | - | `bigint` | 同步 |
| `getAppVMMemoryInfo()` | `GetAppVMMemoryInfo` | - | `object` | 同步 |

### 1.3 原生内存分析

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `getPss()` | `GetPss` | - | `bigint` | 同步 |
| `getSharedDirty()` | `GetSharedDirty` | - | `bigint` | 同步 |
| `getPrivateDirty()` | `GetPrivateDirty` | - | `bigint` | 同步 |
| `getNativeHeapSize()` | `GetNativeHeapSize` | - | `bigint` | 同步 |
| `getNativeHeapAllocatedSize()` | `GetNativeHeapAllocatedSize` | - | `bigint` | 同步 |
| `getNativeHeapFreeSize()` | `GetNativeHeapFreeSize` | - | `bigint` | 同步 |
| `getVss()` | `GetVss` | - | `bigint` | 同步 |
| `getAppNativeMemInfo()` | `GetAppNativeMemInfo` | - | `object` | 同步 |
| `getAppNativeMemInfoWithCache(forceRefresh)` | `GetAppNativeMemInfoWithCache` | `forceRefresh: boolean` | `object` | 同步 |
| `getAppNativeMemInfoAsync()` | `GetAppNativeMemInfoAsync` | - | `Promise<object>` | 异步 |

### 1.4 CPU 使用率

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `getCpuUsage()` | `GetCpuUsage` | - | `number` | 同步 |
| `getSystemCpuUsage()` | `GetSystemCpuUsage` | - | `number` | 同步 |
| `getAppThreadCpuUsage()` | `GetAppThreadCpuUsage` | - | `Array<{threadId, cpuUsage}>` | 同步 |

### 1.5 系统内存

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `getSystemMemInfo()` | `GetSystemMemInfo` | - | `object{totalMem, freeMem, availableMem}` | 同步 |
| `getAppMemoryLimit()` | `GetAppMemoryLimit` | - | `object{rssLimit, vssLimit, vmHeapLimit, vmTotalHeapSize}` | 同步 |

### 1.6 Trace 追踪

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `startAppTraceCapture(tags, traceFlag, limitSize)` | `StartAppTraceCapture` | `tags: bigint[]`, `traceFlag: number`, `limitSize: number` | `string` | 同步 |
| `stopAppTraceCapture()` | `StopAppTraceCapture` | - | `undefined` | 同步 |
| `getVMRuntimeStats()` | `GetVMRuntimeStats` | - | `object` | 同步 |
| `getVMRuntimeStat(property)` | `GetVMRuntimeStat` | `property: string` | `bigint` | 同步 |

### 1.7 系统服务

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `getServiceDump(serviceId, fd, args)` | `GetServiceDump` | `serviceId: number`, `fd: number`, `args: string[]` | `undefined` | 同步 |

### 1.8 资源限制

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `setAppResourceLimit(type, value, enabledDebugLog)` | `SetAppResourceLimit` | `type: string`, `value: number`, `enabledDebugLog: boolean` | `undefined` | 同步 |

### 1.9 调试状态

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `isDebugState()` | `IsDebugState` | - | `boolean` | 同步 |
| `removeNapiWrap(jsObj, needRemoveProperty)` | `RemoveNapiWrap` | `jsObj: object`, `needRemoveProperty: boolean` | `undefined` | 同步 |

### 1.10 图形内存

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `getGraphicsMemory()` | `GetGraphicsMemory` | - | `Promise<number>` | 异步 |
| `getGraphicsMemorySync()` | `GetGraphicsMemorySync` | - | `number` | 同步 |
| `getGraphicsMemorySummary(interval)` | `GetGraphicsMemorySummary` | `interval: number` | `Promise<object>` | 异步 |

### 1.11 GWP-ASan

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `enableGwpAsanGrayscale(options, duration)` | `EnableGwpAsanGrayscale` | `options: GwpAsanOptions`, `duration: number` | `undefined` | 同步 |
| `disableGwpAsanGrayscale()` | `DisableGwpAsanGrayscale` | - | `undefined` | 同步 |
| `getGwpAsanGrayscaleState()` | `GetGwpAsanGrayscaleState` | - | `number` | 同步 |

### 1.12 JS 堆整理

| JS API | C++ 函数 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| `setJsRawHeapTrimLevel(level)` | `SetJsRawHeapTrimLevel` | `level: number` | `undefined` | 同步 |
| `setProcDumpInSharedOOM(enable)` | `SetProcDumpInSharedOOM` | `enable: boolean` | `undefined` | 同步 |

---

## 2. 常量定义

### 2.1 TraceFlag

```typescript
enum TraceFlag {
    MAIN_THREAD = 1,    // 仅主线程
    ALL_THREADS = 2,    // 所有线程
}
```

### 2.2 JsRawHeapTrimLevel

```typescript
enum JsRawHeapTrimLevel {
    TRIM_LEVEL_1 = 0,
    TRIM_LEVEL_2 = 1,
}
```

### 2.3 Trace Tags

```typescript
class tags {
    // 能力与框架
    static readonly ABILITY_MANAGER = 1n;
    static readonly ARKUI = 2n;
    static readonly ARK = 4n;
    static readonly BLUETOOTH = 8n;
    static readonly COMMON_LIBRARY = 16n;
    
    // 分布式服务
    static readonly DISTRIBUTED_HARDWARE_DEVICE_MANAGER = 32n;
    static readonly DISTRIBUTED_AUDIO = 64n;
    static readonly DISTRIBUTED_CAMERA = 128n;
    static readonly DISTRIBUTED_DATA = 256n;
    static readonly DISTRIBUTED_HARDWARE_FRAMEWORK = 512n;
    static readonly DISTRIBUTED_INPUT = 1024n;
    static readonly DISTRIBUTED_SCREEN = 2048n;
    static readonly DISTRIBUTED_SCHEDULER = 4096n;
    
    // 系统服务
    static readonly FFRT = 8192n;
    static readonly FILE_MANAGEMENT = 16384n;
    static readonly GLOBAL_RESOURCE_MANAGER = 32768n;
    static readonly GRAPHICS = 65536n;
    static readonly HDF = 131072n;
    static readonly MISC = 262144n;
    static readonly MULTIMODAL_INPUT = 524288n;
    static readonly NET = 1048576n;
    static readonly NOTIFICATION = 2097152n;
    static readonly NWEB = 4194304n;
    static readonly OHOS = 8388608n;
    static readonly POWER_MANAGER = 16777216n;
    static readonly RPC = 33554432n;
    static readonly SAMGR = 67108864n;
    static readonly WINDOW_MANAGER = 134217728n;
    
    // 媒体
    static readonly AUDIO = 268435456n;
    static readonly CAMERA = 536870912n;
    static readonly IMAGE = 1073741824n;
    static readonly MEDIA = 2147483648n;
}
```

---

## 3. 错误码

| 错误码 | 常量名 | 描述 |
|--------|--------|------|
| 201 | `PERMISSION_ERROR` | 权限错误 |
| 401 | `PARAMETER_ERROR` | 参数错误 |
| 801 | `VERSION_ERROR` | 版本错误 |
| 11400101 | `SYSTEM_ABILITY_NOT_FOUND` | 系统能力未找到 |
| 11400102 | `HAVE_ALREADY_TRACE` | 已在追踪中 |
| 11400103 | `WITHOUT_WRITE_PERMISSON` | 无写权限 |
| 11400104 | `SYSTEM_STATUS_ABNORMAL` | 系统状态异常 |
| 11400105 | `NO_CAPTURE_TRACE_RUNNING` | 无运行中的追踪 |
| 11400106 | `QUOTA_EXCEEDED` | 超出配额 |
| 11400107 | `FORK_FAILED` | fork 失败 |
| 11400108 | `FAILED_WAIT_CHILD_PROCESS_FINISHED` | 等待子进程失败 |
| 11400109 | `TIMEOUT_WAIT_CHILD_PROCESS_FINISHED` | 等待子进程超时 |
| 11400110 | `LOW_DISK_SPACE` | 磁盘空间不足 |
| 11400111 | `NAPI_INTERFACE_ERROR` | N-API 接口错误 |
| 11400112 | `REPEAT_DUMPING` | 重复 dump |
| 11400113 | `FAILED_CREATE_FILE` | 创建文件失败 |
| 11400114 | `OVER_ENABLE_LIMIT` | 超过启用限制 |

> 证据: `util/error_code.h:18-39`

---

## 4. API 注册模式

### 4.1 模块注册

```cpp
// 文件: napi_hidebug.cpp:1133-1146
static napi_module hidebugModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = HiviewDFX::DeclareHiDebugInterface,
    .nm_modname = "hidebug",
    .nm_priv = ((void *)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void HiDebugRegisterModule(void)
{
    napi_module_register(&hidebugModule);
}
```

### 4.2 API 定义

```cpp
// 文件: napi_hidebug.cpp:1086-1128
napi_property_descriptor desc[] = {
    DECLARE_NAPI_FUNCTION("startProfiling", StartProfiling),
    DECLARE_NAPI_FUNCTION("stopProfiling", StopProfiling),
    // ... 更多 API
};

NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc));
```

### 4.3 参数解析模式

```cpp
// 文件: util/napi_util.cpp
// 字符串参数
napi_value GetNapiStringValue(napi_env env, napi_value value, std::string& result);

// 整型参数
napi_value GetNapiInt32Value(napi_env env, napi_value value, int32_t& result);
napi_value GetNapiUint32Value(napi_env env, napi_value value, uint32_t& result);

// 浮点参数
napi_value GetNapiDoubleValue(napi_env env, napi_value value, double& result);

// 布尔参数
napi_value GetNapiBoolValue(napi_env env, napi_value value, bool& result);

// 大整数参数
napi_value GetNapiBigintValue(napi_env env, napi_value value, uint64_t& result);

// 类型检查
napi_value CheckType(napi_env env, napi_value value, napi_valuetype type);
```

---

## 5. 异步任务模式

### 5.1 AsyncTask 基类

```cpp
// 文件: util/napi_util.h:54-86
class AsyncTask {
public:
    virtual ~AsyncTask() = default;
    
    // 创建 Promise
    napi_value CreatePromise(napi_env env);
    
    // 异步执行
    virtual void ExecuteCallBack(napi_env env, void* data) = 0;
    
    // 完成回调
    static void CompletedCallBack(napi_env env, napi_status status, void* data);
    
    // 初始化
    napi_status InitAsyncTask(napi_env env, napi_value asyncWork,
                               napi_deferred deferred, void* data);
    
protected:
    napi_async_work asyncWork_ = nullptr;
    napi_deferred deferred_ = nullptr;
};
```

### 5.2 Promise 使用示例

```cpp
// 文件: napi_hidebug_dump.cpp:343-368
napi_value GetAppNativeMemInfoAsync(napi_env env, napi_callback_info info)
{
    AsyncTask* asyncTask = new AsyncTask();
    napi_value promise = asyncTask->CreatePromise(env);
    
    napi_create_async_work(env, nullptr, asyncTask->GetResourceName(),
        [](napi_env env, void* data) {
            // 后台执行
            asyncTask->Execute();
        },
        [](napi_env env, napi_status status, void* data) {
            // 完成回调
            asyncTask->Complete();
            delete asyncTask;
        },
        &asyncTask->GetData());
    
    napi_queue_async_work(env, asyncTask->GetAsyncWork());
    return promise;
}
```

---

## 6. 调用链示例

### 6.1 startProfiling 调用链

```
JS: hidebug.startProfiling('cpu_trace')
    ↓
N-API: StartProfiling(napi_env, napi_callback_info)
    ↓
插件框架: ProfilerService::StartSession()
    ↓
CPU Plugin: onPluginSessionStart()
    ↓
共享内存: BufferWriter::Write()
```

### 6.2 getCpuUsage 调用链

```
JS: hidebug.getCpuUsage()
    ↓
N-API: GetCpuUsage(napi_env, napi_callback_info)
    ↓
系统调用: /proc/stat 读取
    ↓
计算: CPU 使用率计算
    ↓
N-API: napi_create_double(env, usage, &result)
    ↓
JS: 返回 number 类型
```

---

## 7. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 内部 API | [04_Inner_API.md](./04_Inner_API.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*最后更新: 2026-02-06*
