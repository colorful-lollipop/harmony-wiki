# N-API 参考

> JS/TS API 详细说明

---

## 模块概览

| 模块名 | 命名空间 | 描述 |
|--------|----------|------|
| systemTime | `@ohos.systemTime` | 系统时间获取/设置 |
| systemTimer | `@ohos.systemTimer` | 定时器管理 |
| systemDateTime | `@ohos.systemDateTime` | 日期时间处理 |

---

## @ohos.systemTime

### 模块注册

**注册点**: `framework/js/napi/system_time/src/js_systemtime.cpp:446-457`

```cpp
static napi_module system_time_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = SystemTimeExport,
    .nm_modname = "systemTime",  // JS 导入名
    .nm_priv = ((void *)0),
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void SystemTimeRegister() {
    napi_module_register(&system_time_module);
}
```

### 导出方法

| JS 方法名 | C++ 实现函数 | 位置 | 同步/异步 |
|-----------|-------------|------|-----------|
| `setTime` | `JSSystemTimeSetTime` | `js_systemtime.cpp:68` | 异步 |
| `setDate` | `JSSystemTimeSetTime` | `js_systemtime.cpp:68` | 异步 |
| `setTimezone` | `JSSystemTimeSetTimeZone` | `js_systemtime.cpp:120` | 异步 |
| `getCurrentTime` | `JSSystemTimeGetCurrentTime` | `js_systemtime.cpp:172` | 异步 |
| `getRealActiveTime` | `JSSystemTimeGetRealActiveTime` | `js_systemtime.cpp:225` | 异步 |
| `getRealTime` | `JSSystemTimeGetRealTime` | `js_systemtime.cpp:278` | 异步 |
| `getDate` | `JSSystemTimeGetDate` | `js_systemtime.cpp:331` | 异步 |
| `getTimezone` | `JSSystemTimeGetTimeZone` | `js_systemtime.cpp:379` | 异步 |

### API 详情

#### setTime

```typescript
function setTime(time: number): Promise<boolean>
function setTime(time: number, callback: AsyncCallback<boolean>): void
```

**参数**: 
- `time`: Unix 时间戳（毫秒，1970-01-01 至今）

**权限**: `ohos.permission.SET_TIME`

**错误码**:
- `201`: 权限校验失败
- `401`: 参数错误
- `1000`: 系统错误

**C++ 实现**:
```cpp
// js_systemtime.cpp:68
napi_value JSSystemTimeSetTime(napi_env env, napi_callback_info info) {
    // 1. 解析参数（时间戳 + 可选回调）
    // 2. 创建 AsyncContext
    // 3. 创建异步任务
    // 4. 调用 TimeServiceClient::GetInstance()->SetTime()
    // 5. 返回 Promise 或执行回调
}
```

#### getCurrentTime

```typescript
function getCurrentTime(isNano?: boolean): Promise<number>
function getCurrentTime(callback: AsyncCallback<number>): void
function getCurrentTime(isNano: boolean, callback: AsyncCallback<number>): void
```

**参数**:
- `isNano`: 是否返回纳秒（默认毫秒）

**权限**: 无需权限

**返回值**: Wall Time（UTC 时间）

**调用链**:
```
JS getCurrentTime()
  → JSSystemTimeGetCurrentTime()
    → TimeServiceClient::GetWallTimeMs() / GetWallTimeNs()
      → clock_gettime(CLOCK_REALTIME)
```

---

## @ohos.systemTimer

### 模块注册

**注册点**: `framework/js/napi/system_timer/src/timer_init.cpp:50-54`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_filename = nullptr,
    .nm_register_func = RegisterModule,
    .nm_modname = "systemTimer",
    .nm_priv = nullptr,
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void RegisterSystemTimerModule() {
    napi_module_register(&_module);
}
```

### 导出方法

| JS 方法名 | C++ 实现函数 | 位置 | 说明 |
|-----------|-------------|------|------|
| `createTimer` | `CreateTimer` | `napi_system_timer.cpp:249` | 创建定时器 |
| `startTimer` | `StartTimer` | `napi_system_timer.cpp:282` | 启动定时器 |
| `stopTimer` | `StopTimer` | `napi_system_timer.cpp:317` | 停止定时器 |
| `destroyTimer` | `DestroyTimer` | `napi_system_timer.cpp:345` | 销毁定时器 |

### 导出常量

| JS 常量名 | 值 | 说明 |
|-----------|-----|------|
| `TIMER_TYPE_REALTIME` | 1 | 基于实时时间 |
| `TIMER_TYPE_WAKEUP` | 2 | 唤醒模式 |
| `TIMER_TYPE_EXACT` | 4 | 精确触发 |
| `TIMER_TYPE_IDLE` | 8 | 空闲模式 |

### API 详情

#### createTimer

```typescript
interface TimerOptions {
    type: number;           // 定时器类型（位掩码）
    repeat: boolean;        // 是否重复
    interval?: number;      // 重复间隔（毫秒，repeat=true 时需 >= 5000）
    autoRestore?: boolean;  // 重启后是否恢复
    wantAgent?: WantAgent;  // 触发时发送通知
    callback?: () => void;  // 触发回调
    name?: string;          // 定时器名称（<= 64字符）
}

function createTimer(options: TimerOptions): Promise<number>
function createTimer(options: TimerOptions, callback: AsyncCallback<number>): void
```

**返回值**: timerId（定时器唯一标识）

**权限**: 系统应用权限

**参数校验** (`napi_system_timer.cpp:143-192`):

| 参数 | 校验规则 |
|------|----------|
| `type` | 必填，必须是 `number` |
| `repeat` | 必填，必须是 `boolean` |
| `interval` | 可选，`>= 0`，`number` 类型 |
| `name` | 可选，字符串长度 `<= 64` |
| `wantAgent` | 可选，必须是非空对象 |
| `callback` | 可选，必须是函数 |

**调用链**:
```
JS createTimer(options)
  → CreateTimer()
    → GetTimerOptions()          // 参数解析
      → ParseTimerOptions()      // 字段校验
    → TimeServiceClient::CreateTimerV9()
      → IPC → TimeSystemAbility
        → TimerManager::CreateTimer()
```

#### startTimer

```typescript
function startTimer(timer: number, triggerTime: number): Promise<boolean>
function startTimer(timer: number, triggerTime: number, callback: AsyncCallback<boolean>): void
```

**参数**:
- `timer`: 定时器 ID（createTimer 返回）
- `triggerTime`: 触发时间（Unix 时间戳，毫秒）

**调用链**:
```
JS startTimer(timerId, triggerTime)
  → StartTimer()
    → TimeServiceClient::StartTimerV9()
      → IPC → TimeSystemAbility
        → TimerManager::StartTimer()
          → timerfd_settime()
```

#### stopTimer / destroyTimer

```typescript
function stopTimer(timer: number): Promise<boolean>
function destroyTimer(timer: number): Promise<boolean>
```

**区别**:
- `stopTimer`: 停止定时器，可重新 start
- `destroyTimer`: 销毁定时器，释放资源

---

## @ohos.systemDateTime

### 模块注册

**注册点**: `framework/js/napi/system_date_time/src/date_time_init.cpp:51`

```cpp
napi_module_register(&_module);
```

### 导出方法

| JS 方法名 | C++ 实现函数 | 位置 |
|-----------|-------------|------|
| `setDate` | `JSSystemDateTimeSetDate` | `napi_system_date_time.cpp` |
| `getDate` | `JSSystemDateTimeGetDate` | `napi_system_date_time.cpp` |

### API 详情

#### setDate

```typescript
function setDate(date: Date): Promise<boolean>
function setDate(date: Date, callback: AsyncCallback<boolean>): void
```

**参数**: `date` - JavaScript Date 对象

**说明**: 内部转换为时间戳后调用 `SetTime`

---

## 参数校验详解

### 通用校验模式

```cpp
// 参数数量检查
CHECK_ARGS_RETURN_VOID(TIME_MODULE_JS_NAPI, context, argc >= ARGC_ONE,
    "Mandatory parameters are left unspecified", 
    JsErrorCode::PARAMETER_ERROR);

// 类型检查
napi_valuetype valueType = napi_undefined;
napi_typeof(env, result, &valueType);
CHECK_ARGS_RETURN_VOID(TIME_MODULE_JS_NAPI, context, 
    valueType == PARA_NAPI_TYPE_MAP[paraType],
    paraType + ": incorrect parameter types",
    JsErrorCode::PARAMETER_ERROR);

// 范围检查
CHECK_ARGS_RETURN_VOID(TIME_MODULE_JS_NAPI, context, interval >= 0,
    "interval number must >= 0.", 
    JsErrorCode::PARAMETER_ERROR);

// 长度检查
CHECK_ARGS_RETURN_VOID(TIME_MODULE_JS_NAPI, context, name.size() <= STR_MAX_LENGTH,
    "timer name must <= 64.", 
    JsErrorCode::PARAMETER_ERROR);
```

### 错误码定义

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERROR_OK` | 0 | 成功 |
| `ERROR` | 1000 | 系统错误 |
| `PARAMETER_ERROR` | 401 | 参数错误 |

---

## 异步处理模式

### Promise/Callback 统一封装

```cpp
// js_systemtime.cpp:53-66
void TimePaddingAsyncCallbackInfo(const napi_env &env, AsyncContext *&asynccallbackinfo, 
    const napi_ref &callback, napi_value &promise) {
    if (callback) {
        // Callback 模式
        asynccallbackinfo->callbackRef = callback;
        asynccallbackinfo->isCallback = true;
    } else {
        // Promise 模式
        napi_deferred deferred = nullptr;
        napi_create_promise(env, &deferred, &promise);
        asynccallbackinfo->deferred = deferred;
        asynccallbackinfo->isCallback = false;
    }
}
```

### 异步任务创建

```cpp
// js_systemtime.cpp:88-117
napi_create_async_work(
    env, nullptr, resource,
    // 执行器（工作线程）
    [](napi_env env, void *data) {
        AsyncContext *asyncContext = (AsyncContext *)data;
        asyncContext->isOK = TimeServiceClient::GetInstance()->SetTime(
            asyncContext->time, errorCode);
    },
    // 完成器（主线程）
    [](napi_env env, napi_status status, void *data) {
        AsyncContext *asyncContext = (AsyncContext *)data;
        NapiUtils::ReturnCallbackPromise(env, info, result);
        napi_delete_async_work(env, asyncContext->work);
        delete asyncContext;
    },
    (void *)asyncContext, &asyncContext->work);

// 提交到队列
napi_queue_async_work_with_qos(env, asyncContext->work, napi_qos_user_initiated);
```

---

## 相关链接

- [内部 API](./04_Inner_API.md) - C++ 客户端接口
- [架构说明](./02_Architecture.md) - 数据流与调用链
- [安全分析](./06_Security_Analysis.md) - 权限与校验
