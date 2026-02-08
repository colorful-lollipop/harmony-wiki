# 附录 B：配置开关

## 编译时配置

### napi.gni 配置

**证据位置**：`napi.gni`

| 开关 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `napi_enable_container_scope` | bool | false | 启用容器作用域管理 |
| `napi_enable_memleak_debug` | bool | true | 启用内存泄漏调试 |
| `napi_feature_enable_pgo` | bool | false | 启用 PGO 性能分析导向优化 |
| `napi_feature_pgo_path` | string | "" | PGO profile 数据路径 |
| `enabled_data_protector` | bool | auto | 启用数据保护（arm64 OHOS 自动启用） |

### BUILD.gn 平台配置

**证据位置**：`BUILD.gn:22-98`

#### OHOS 标准系统

| 宏 | 描述 |
|----|------|
| `OHOS_PLATFORM` | OHOS 平台标识 |
| `OHOS_STANDARD_PLATFORM` | 标准系统标识 |
| `ENABLE_HITRACE` | 启用性能追踪 |
| `ENABLE_EVENT_HANDLER` | 启用事件处理器 |
| `ENABLE_FFRT` | 启用快慢任务分离 |
| `ENABLE_UCOLLECTION` | 启用集合工具 |
| `RESOURCE_SCHEDULE_SERVICE_ENABLE` | 启用资源调度服务 |
| `HOOK_ENABLE` | 启用内存 hook（musl 且非 asan） |

#### 模拟器

| 宏 | 描述 |
|----|------|
| `SIMULATOR` | 模拟器标识 |

#### Watch/Wearable

| 宏 | 描述 |
|----|------|
| `DISABLE_SHORT_IDLE_CHECK` | 禁用短空闲检查 |

#### CPU 架构

| 宏 | 描述 |
|----|------|
| `NAPI_TARGET_AMD64` / `NAPI_TARGET_64` | x86_64 架构 |
| `NAPI_TARGET_X86` / `NAPI_TARGET_32` | x86 架构 |
| `NAPI_TARGET_ARM64` / `NAPI_TARGET_64` | arm64 架构 |
| `NAPI_TARGET_ARM32` / `NAPI_TARGET_32` | arm 架构 |
| `_ARM64_` | ARM64 编译标识（特定条件） |

### Feature Flags

**证据位置**：`bundle.json`

```json
"features": [
  "napi_enable_container_scope",
  "napi_feature_enable_pgo",
  "napi_feature_pgo_path",
  "napi_enable_data_protector"
]
```

## 运行时配置

### 环境变量

| 环境变量 | 描述 | 示例 |
|----------|------|------|
| `UV_THREADPOOL_SIZE` | libuv 线程池大小 | `UV_THREADPOOL_SIZE=32` |
| `OHOS_ABILITY_CONNECTION` | 能力连接配置 | - |
| `persist.ark.arkbundlename` | Ark bundle 名称 | `persist.ark.arkbundlename="com.example"` |

### 配置回调

**证据位置**：`native_engine.h`

| 回调 | 用途 | 设置方法 |
|------|------|----------|
| `PermissionCheckCallback` | 权限检查 | `RegisterPermissionCheck()` |
| `AppFreezeFilterCallback` | 应用冻结过滤 | `SetAppFreezeFilterCallback()` |
| `NapiUncaughtExceptionCallback` | 未捕获异常 | `RegisterNapiUncaughtExceptionHandler()` |
| `SourceMapCallback` | SourceMap 翻译 | `RegisterTranslateBySourceMap()` |
| `NapiOnMainThreadErrorCallback` | 主线程错误 | - |
| `NapiOnWorkerErrorCallback` | Worker 错误 | - |

## 特性开关

### 容器作用域

**证据位置**：`native_engine.h:77-79`, `BUILD.gn:147-149`

```cpp
#ifdef ENABLE_CONTAINER_SCOPE
    int32_t containerScopeId_;
#endif
```

启用条件：
```gn
if (napi_enable_container_scope) {
    external_deps += [ "ace_engine:ace_container_scope_static" ]
    defines += [ "ENABLE_CONTAINER_SCOPE" ]
}
```

### Sendable 对象

**证据位置**：`native_api.h:159-200`

| API | 描述 |
|-----|------|
| `napi_define_sendable_class()` | 定义 sendable 类 |
| `napi_create_sendable_object_with_properties()` | 创建 sendable 对象 |
| `napi_create_sendable_array()` | 创建 sendable 数组 |
| `napi_wrap_sendable()` | 包装 sendable 引用 |
| `napi_is_sendable()` | 检查是否可发送 |

### 并发函数

**证据位置**：`native_engine.h:212-215`

```cpp
virtual bool InitTaskPoolThread(NativeEngine* engine, NapiConcurrentCallback callback) = 0;
virtual bool InitTaskPoolThread(napi_env env, NapiConcurrentCallback callback) = 0;
virtual bool InitTaskPoolFunc(napi_env env, napi_value func, void* taskInfo) = 0;
```

## 调试配置

### 日志级别

**证据位置**：`utils/log.h`

```cpp
HILOG_DEBUG("message");   // 调试
HILOG_INFO("message");    // 信息
HILOG_WARN("message");    // 警告
HILOG_ERROR("message");   // 错误
HILOG_FATAL("message");   // 致命
```

### 性能分析

**证据位置**：`native_engine.h:266-267`

```cpp
virtual void StartCpuProfiler(const std::string& fileName = "") = 0;
virtual void StopCpuProfiler() = 0;
```

### 堆快照

**证据位置**：`native_engine.h:276-283`

```cpp
virtual void DumpHeapSnapshot(const std::string &path, bool isVmMode = true,
    DumpFormat dumpFormat = DumpFormat::JSON, bool isPrivate = false,
    bool captureNumericValue = false, bool isJSLeakWatcher = false) = 0;
```

### 堆内存追踪

**证据位置**：`native_engine.h:295-296`

```cpp
virtual bool StartHeapTracking(double timeInterval, bool isVmMode = true) = 0;
virtual bool StopHeapTracking(const std::string &filePath) = 0;
```
