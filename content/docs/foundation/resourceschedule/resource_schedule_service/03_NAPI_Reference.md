# N-API 接口参考

## 目的

本文档详细描述 Resource Schedule Service 提供的 JS API，包括接口清单、参数说明、错误码和调用链。

## 适用范围

- 应用开发者
- 需要调用资源调度能力的系统服务开发者

---

## 模块一：@ohos.resourceschedule.systemload

### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `resourceschedule.systemload` |
| **完整路径** | `@ohos.resourceschedule.systemload` |
| **SystemCapability** | `SystemCapability.ResourceSchedule.SystemLoad` |
| **源代码** | `ressched/interfaces/kits/js/napi/systemload/` |
| **输出产物** | `systemload.so` (安装至 module/resourceschedule/) |

### API 清单

| JS 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|---------|------|--------|-----------|------|
| `on(type, callback)` | `type: string`<br>`callback: function` | `void` | 同步注册<br>异步回调 | 订阅系统负载变化 |
| `off(type, callback)` | `type: string`<br>`callback: function` | `void` | 同步 | 取消订阅 |
| `getLevel()` | 无 | `Promise<SystemLoadLevel>` | 异步 | 获取当前负载等级 |

### SystemLoadLevel 枚举

```typescript
enum SystemLoadLevel {
    LOW = 0,        // 系统负载低
    NORMAL = 1,     // 系统负载正常
    MEDIUM = 2,     // 系统负载中等
    HIGH = 3,       // 系统负载高
    OVERHEATED = 4, // 系统过热
    WARNING = 5,    // 系统警告
    EMERGENCY = 6,  // 系统紧急
    ESCAPE = 7      // 系统逃逸
}
```

**C++ 定义位置**: `ressched/interfaces/kits/js/napi/systemload/src/js_systemload_napi_init.cpp:43-89`

### 使用示例

```typescript
import systemload from '@ohos.resourceschedule.systemload';

// 订阅系统负载变化
systemload.on('systemLoadChange', (level: systemload.SystemLoadLevel) => {
    console.log(`System load level: ${level}`);
    if (level >= systemload.SystemLoadLevel.HIGH) {
        // 降低应用资源消耗
    }
});

// 获取当前负载等级
systemload.getLevel().then((level) => {
    console.log(`Current level: ${level}`);
});

// 取消订阅
systemload.off('systemLoadChange', callback);
```

### 参数校验

#### `on(type, callback)` 参数检查

| 检查项 | 规则 | 错误码 |
|--------|------|--------|
| 参数数量 | 必须为 2 | 401 |
| type 类型 | 必须为 string | 401 |
| type 取值 | 必须为 `"systemLoadChange"` | 401 |
| callback 类型 | 必须为 function | 401 |

**代码位置**: `ressched/interfaces/kits/js/napi/systemload/src/js_systemload.cpp:280-315`

```cpp
bool Systemload::CheckCallbackParam(napi_env env, napi_callback_info info,
                                    std::string &cbType, napi_value *jsCallback,
                                    int32_t status) {
    // 参数数量检查
    if (status == ON && argc != ARG_COUNT_TWO) {
        return false;
    }
    
    // 类型检查
    if (!ConvertFromJsValue(env, argv[0], cbType)) {
        RESSCHED_LOGE("Parameter error. The type of \"type\" must be string");
        return false;
    }
    
    // 枚举值检查
    if (cbType != SYSTEMLOAD_LEVEL) {  // "systemLoadChange"
        RESSCHED_LOGE("Parameter error. The type of \"type\" must be systemLoadChange");
        return false;
    }
    
    // 回调函数检查
    bool isCallable = false;
    napi_is_callable(env, *jsCallback, &isCallable);
    if (status == ON && !isCallable) {
        return false;
    }
    return true;
}
```

### 错误码

| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误。输入参数有误 |

### 调用链

```
JS: systemload.on()
  └─► NAPI: Systemload::SystemloadOn()
       └─► Systemload::RegisterSystemloadCallback()
            ├─► CheckCallbackParam()           // 参数校验
            ├─► napi_create_reference()        // 创建回调引用
            ├─► new SystemloadListener()       // 创建监听器
            └─► ResSchedClient::RegisterSystemloadNotifier()
                 └─► IPC: RegisterSystemloadNotifier() (SA 1901)
                      └─► NotifierMgr::RegisterNotifier()

当系统负载变化时:
NotifierMgr::OnDeviceLevelChanged()
  └─► SystemloadListener::OnSystemloadLevel() (IPC callback)
       └─► napi_call_threadsafe_function()    // 线程安全回调
            └─► ThreadSafeCallBack()          // 主线程执行
                 └─► JS callback              // 执行 JS 回调
```

### 异步模型

#### `getLevel()` Promise 实现

```cpp
// 文件: ressched/interfaces/kits/js/napi/systemload/src/js_systemload.cpp:202-254

napi_value Systemload::GetSystemloadLevel(napi_env env, napi_callback_info info) {
    // 1. 创建 Promise
    napi_deferred deferred;
    napi_value promise;
    napi_create_promise(env, &deferred, &promise);
    
    // 2. 创建异步工作项
    napi_create_async_work(env, nullptr, resourceName,
        Execute,      // 工作线程执行
        Complete,     // 主线程完成回调
        cbInfo, &asyncWork);
    
    // 3. 加入队列
    napi_queue_async_work(env, asyncWork);
    
    return promise;
}

void Systemload::Execute(napi_env env, void* data) {
    // 工作线程：调用 IPC 获取系统负载级别
    cbInfo->result = ResSchedClient::GetInstance().GetSystemloadLevel();
}

void Systemload::Complete(napi_env env, napi_status status, void* data) {
    // 主线程：解析 Promise
    napi_resolve_deferred(env, cbInfo->deferred, result);
}
```

---

## 模块二：@ohos.resourceschedule.backgroundProcessManager

### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `resourceschedule.backgroundProcessManager` |
| **完整路径** | `@ohos.resourceschedule.backgroundProcessManager` |
| **SystemCapability** | `SystemCapability.Resourceschedule.BackgroundProcessManager` |
| **源代码** | `ressched/interfaces/kits/js/napi/background_process_manager/` |
| **输出产物** | `libbackgroundprocessmanager_napi.z.so` |

### API 清单

| JS 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|---------|------|--------|-----------|------|
| `setProcessPriority(pid, priority)` | `pid: number`<br>`priority: ProcessPriority` | `number` (错误码) | 同步 | 设置进程优先级 |
| `resetProcessPriority(pid)` | `pid: number` | `number` (错误码) | 同步 | 重置进程优先级 |
| `setPowerSaveMode(pid, mode)` | `pid: number`<br>`mode: PowerSaveMode` | `number` (错误码) | 同步 | 设置省电模式 |
| `isPowerSaveMode(pid)` | `pid: number` | `Promise<boolean>` | 异步 | 查询是否省电模式 |
| `getPowerSaveMode(pid)` | `pid: number` | `Promise<PowerSaveMode>` | 异步 | 获取省电模式 |

### ProcessPriority 枚举

```typescript
enum ProcessPriority {
    PROCESS_BACKGROUND = 0,  // 后台进程优先级
    PROCESS_INACTIVE = 1     // 非活动进程优先级
}
```

### PowerSaveMode 枚举

```typescript
enum PowerSaveMode {
    EFFICIENCY_MODE = 0,     // 节能模式
    DEFAULT_MODE = 1         // 默认模式
}
```

### 使用示例

```typescript
import backgroundProcessManager from '@ohos.resourceschedule.backgroundProcessManager';

// 设置进程优先级
let ret = backgroundProcessManager.setProcessPriority(pid, 
    backgroundProcessManager.ProcessPriority.PROCESS_BACKGROUND);
if (ret !== 0) {
    console.error(`Failed to set priority: ${ret}`);
}

// 重置优先级
ret = backgroundProcessManager.resetProcessPriority(pid);

// 设置省电模式
ret = backgroundProcessManager.setPowerSaveMode(pid,
    backgroundProcessManager.PowerSaveMode.EFFICIENCY_MODE);

// 查询省电模式
backgroundProcessManager.isPowerSaveMode(pid).then((isPowerSave) => {
    console.log(`Is power save mode: ${isPowerSave}`);
});
```

### 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 201 | 权限拒绝。需要系统权限 |
| 401 | 参数错误。参数超出范围 |
| 801 | 能力不支持 |
| 31800001 | 远程错误 |
| 31800002 | 参数错误 |
| 31800003 | 设置被任务管理器覆盖 |
| 31800004 | 系统调度原因导致设置失败 |

**错误码定义位置**: `ressched/interfaces/kits/c/background_process_manager/include/background_process_manager.h:88-129`

### 参数校验

#### `setProcessPriority` 参数检查

| 检查项 | 规则 | 错误码 |
|--------|------|--------|
| 参数数量 | 必须为 2 | 31800002 |
| pid 类型 | 必须为 number | 31800002 |
| priority 类型 | 必须为 number | 31800002 |

**代码位置**: `ressched/interfaces/kits/js/napi/background_process_manager/src/background_process_manager_napi_init.cpp:66-105`

```cpp
napi_value SetProcessPriority(napi_env env, napi_callback_info info) {
    // 参数数量检查
    if (argc != SET_PROCESS_PRIORITY_PARAM_NUM) {  // 2
        HandleErrorCode(env, ERR_BACKGROUND_PROCESS_MANAGER_INVALID_PARAM);
        return ret;
    }
    
    // 类型检查
    napi_valuetype pidType, priorityType;
    napi_typeof(env, argv[PID_INDEX], &pidType);
    napi_typeof(env, argv[PRIORITY_INDEX], &priorityType);
    
    if (pidType != napi_number || priorityType != napi_number) {
        HandleErrorCode(env, ERR_BACKGROUND_PROCESS_MANAGER_INVALID_PARAM);
        return ret;
    }
    
    // 提取数值
    int32_t pid, priority;
    napi_get_value_int32(env, argv[PID_INDEX], &pid);
    napi_get_value_int32(env, argv[PRIORITY_INDEX], &priority);
    
    // 调用 C 接口
    int retCode = OH_BackgroundProcessManager_SetProcessPriority(pid, 
        static_cast<BackgroundProcessManager_ProcessPriority>(priority));
    HandleErrorCode(env, retCode);
    return ret;
}
```

### 调用链

#### `setProcessPriority` 调用链

```
JS: setProcessPriority(pid, priority)
  └─► NAPI: SetProcessPriority()
       ├─► 参数校验 (数量/类型)
       ├─► napi_get_value_int32()     // 提取数值
       └─► OH_BackgroundProcessManager_SetProcessPriority()
            └─► ResSchedClient::ReportSyncEvent()  // IPC (SA 1901)
                 └─► ResSchedService::ReportSyncEvent()
                      └─► PluginMgr::DeliverResource()
                           └─► cgroup_sched_plugin::OnDispatchResource()
                                └─► 设置进程优先级
```

#### `isPowerSaveMode` 调用链

```
JS: isPowerSaveMode(pid)
  └─► NAPI: IsPowerSaveMode()
       ├─► 参数校验
       ├─► OH_BackgroundProcessManager_IsPowerSaveMode()
       │    └─► ResSchedClient::ReportSyncEvent()  // IPC
       │
       ├─► napi_create_promise()    // 创建 Promise
       ├─► napi_resolve_deferred()  // 立即解析
       └─► return promise
```

---

## API 对比总结

| 特性 | systemload | backgroundProcessManager |
|------|------------|-------------------------|
| **API 数量** | 3 | 5 |
| **同步 API** | `on`, `off` | `setProcessPriority`, `resetProcessPriority`, `setPowerSaveMode` |
| **异步 API** | `getLevel` (Promise) | `isPowerSaveMode`, `getPowerSaveMode` (Promise) |
| **事件监听** | ✅ 支持 | ❌ 不支持 |
| **线程安全** | ThreadSafeFunction | 无特殊处理 |
| **权限要求** | 无 | `setPowerSaveMode` 等需要系统权限 |
| **底层调用** | ResSchedClient IPC | OH_BackgroundProcessManager C API |

---

## 代码证据

### systemload 模块注册

```cpp
// 文件: ressched/interfaces/kits/js/napi/systemload/include/js_systemload_napi_init.h:35-43

static napi_module systemloadModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitSystemloadApi,
    .nm_modname = "resourceschedule.systemload",
    .nm_priv = ((void*)0),
    .reserved = { 0 },
};
```

### backgroundProcessManager 模块注册

```cpp
// 文件: ressched/interfaces/kits/js/napi/background_process_manager/include/background_process_manager_napi_init.h:36-44

static napi_module backgroundProcessManagerModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "resourceschedule.backgroundProcessManager",
    .nm_priv = ((void*)0),
    .reserved = { 0 },
};
```

---

## 相关链接

- [概览](00_Overview.md) - 项目定位
- [架构设计](01_Architecture.md) - 数据流图
- [内部 API](04_Inner_API.md) - C++ 接口
- [GN 构建](05_GN_Targets.md) - 构建配置
