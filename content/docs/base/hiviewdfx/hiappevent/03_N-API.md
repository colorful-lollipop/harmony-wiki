# N-API 接口文档

## 3.1 接口注册点与模块结构

HiAppEvent 组件对外提供两套 N-API 接口集，分别是面向 API 7 及更早版本的 `hiappevent` 模块，以及面向 API 9 及以后版本的 `hiappevent_v9` 模块。两套接口在功能上保持一致，但在接口组织和扩展性上有所不同。

### 3.1.1 模块注册点

**API 7 模块（hiappevent）**：

| 注册项 | 值 |
|-------|-----|
| 模块名称 | `hiAppEvent` |
| 入口文件 | `frameworks/js/napi/src/napi_hiappevent_js.cpp` |
| 注册函数 | `Init()` |
| 构建目标 | `//base/hiviewdfx/hiappevent/frameworks/js/napi:hiappevent` |
| 输出产物 | `libhiappevent.z.so` |

**API 9+ 模块（hiappevent_v9）**：

| 注册项 | 值 |
|-------|-----|
| 模块名称 | `hiAppEvent` |
| 入口文件 | `frameworks/js/napi/src/napi_hiappevent_js_v9.cpp` |
| 注册函数 | `InitV9()` |
| 构建目标 | `//base/hiviewdfx/hiappevent/frameworks/js/napi:hiappevent_v9` |
| 输出产物 | `hiappevent_napi.so` |

### 3.1.2 N-API 注册源码

模块注册通过 N-API 的 `napi_module_register()` 函数完成，源码位于 `napi_hiappevent_js.cpp:109-112`：

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "hiAppEvent",
    .nm_priv = ((void *)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&_module);
}
```

## 3.2 JS API 清单表

### 3.2.1 事件写入接口

#### hiAppEvent.write()

**功能描述**：应用事件异步打点方法，用于将应用运行过程中的关键事件记录到事件日志系统中。

**C/C++ 绑定位置**：`frameworks/js/napi/src/napi_hiappevent_js.cpp:33-69`

**函数签名**：
```typescript
// Callback 模式
write(eventName: string, type: EventType, keyValues: object, callback: AsyncCallback<void>): void

// Promise 模式
write(eventName: string, type: EventType, keyValues: object): Promise<void>
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|:----:|------|
| eventName | string | 是 | 事件名称，字符串类型，长度限制 1-128 字符 |
| type | EventType | 是 | 事件类型枚举值 |
| keyValues | object | 否 | 事件参数字典，JSON 对象类型，键值对形式 |
| callback | AsyncCallback | 否 | 异步回调函数，不传则使用 Promise 模式 |

**返回值说明**：

| 返回值 | 类型 | 说明 |
|-------|------|------|
| callback 第二个参数 | void | 异步操作完成后的空返回值 |
| Promise resolved | void | 异步操作成功完成 |
| Promise rejected | Error | 操作失败时的错误对象，包含 code 和 message 属性 |

**回调函数错误码**：

| 错误码值 | 含义 | 处理建议 |
|---------|------|---------|
| 0 | 事件参数校验成功，事件正常写入 | 无需处理 |
| 正整数 | 事件存在异常参数，异常参数被忽略后正常写入 | 检查 keyValues 中的参数是否符合规范 |
| 负整数 | 事件校验失败，不执行写入操作 | 检查 eventName 是否合法、type 是否正确 |

**使用示例**：

```javascript
// Callback 模式
import hiAppEvent from '@ohos.hiAppEvent'

hiAppEvent.write("user_login", hiAppEvent.EventType.BEHAVIOR, 
    {"user_id": "U10001", "login_type": "password"}, 
    (err, value) => {
        if (err) {
            console.error(`事件写入失败: ${err.code}`)
            return
        }
        console.log(`事件写入成功: ${value}`)
    })

// Promise 模式
hiAppEvent.write("app_crash", hiAppEvent.EventType.FAULT, 
    {"crash_reason": "NULL_POINTER", "stack_trace": "..."})
    .then((value) => {
        console.log(`事件写入成功: ${value}`)
    })
    .catch((err) => {
        console.error(`事件写入失败: ${err.code}`)
    })
```

### 3.2.2 事件配置接口

#### hiAppEvent.configure()

**功能描述**：应用事件打点配置方法，用于对打点功能进行自定义配置，包括打点开关、存储配额等。

**C/C++ 绑定位置**：`frameworks/js/napi/src/napi_hiappevent_js.cpp:71-82`

**函数签名**：
```typescript
configure(config: ConfigOption): boolean
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|:----:|------|
| config | ConfigOption | 是 | 应用事件打点配置项对象 |

**ConfigOption 类型定义**：

| 字段名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| disable | boolean | false | 应用打点功能开关，true 表示关闭打点功能 |
| maxStorage | string | "10M" | 打点数据本地存储目录的配额大小，支持单位：K、M、G |

**返回值**：

| 类型 | 说明 |
|-----|------|
| boolean | true 表示配置成功，false 表示配置失败 |

**使用示例**：

```javascript
import hiAppEvent from '@ohos.hiAppEvent'

// 关闭应用打点功能
hiAppEvent.configure({ disable: true })

// 配置事件存储目录配额为 100M
hiAppEvent.configure({ maxStorage: '100M' })

// 同时配置多个选项
hiAppEvent.configure({ 
    disable: false,
    maxStorage: '50M'
})
```

### 3.2.3 事件类型枚举（EventType）

**C/C++ 定义位置**：`frameworks/js/napi/src/napi_hiappevent_init.cpp:26-29`

**枚举值**：

| 枚举值 | 整数值 | 说明 |
|-------|:------:|------|
| FAULT | 1 | 故障类型事件，用于记录应用运行过程中发生的异常和错误 |
| STATISTIC | 2 | 统计类型事件，用于记录应用的性能指标和统计数据 |
| SECURITY | 3 | 安全类型事件，用于记录与安全相关的操作和事件 |
| BEHAVIOR | 4 | 行为类型事件，用于记录用户的操作行为和业务流程 |

**使用示例**：
```javascript
import hiAppEvent from '@ohos.hiAppEvent'

// 使用预定义的事件类型常量
hiAppEvent.EventType.FAULT     // 1
hiAppEvent.EventType.STATISTIC // 2
hiAppEvent.EventType.SECURITY  // 3
hiAppEvent.EventType.BEHAVIOR  // 4
```

### 3.2.4 预定义事件名称常量（Event）

**C/C++ 定义位置**：`frameworks/js/napi/src/napi_hiappevent_init.cpp:60-78`

| 常量名 | 事件名称字符串 | 说明 |
|-------|---------------|------|
| USER_LOGIN | "hiappevent.user_login" | 用户登录事件 |
| USER_LOGOUT | "hiappevent.user_logout" | 用户登出事件 |
| DISTRIBUTED_SERVICE_START | "hiappevent.distributed_service_start" | 分布式服务启动事件 |
| APP_CRASH | "APP_CRASH" | 应用崩溃事件 |
| APP_FREEZE | "APP_FREEZE" | 应用卡死事件 |
| APP_LAUNCH | "APP_LAUNCH" | 应用启动事件 |
| SCROLL_JANK | "SCROLL_JANK" | 滑动卡顿事件 |
| CPU_USAGE_HIGH | "CPU_USAGE_HIGH" | CPU 使用率过高事件 |
| BATTERY_USAGE | "BATTERY_USAGE" | 电池使用事件 |
| RESOURCE_OVERLIMIT | "RESOURCE_OVERLIMIT" | 资源超限事件 |
| ADDRESS_SANITIZER | "ADDRESS_SANITIZER" | 地址 sanitizer 事件 |
| MAIN_THREAD_JANK | "MAIN_THREAD_JANK" | 主线程卡顿事件 |
| APP_KILLED | "APP_KILLED" | 应用被终止事件 |
| AUDIO_JANK_FRAME | "AUDIO_JANK_FRAME" | 音频卡顿帧事件 |
| APP_HICOLLIE | "APP_HICOLLIE" | 应用 Hicollie 事件 |
| SCROLL_ARKWEB_FLING_JANK | "SCROLL_ARKWEB_FLING_JANK" | ArkWeb 滑动惯性卡顿事件 |

### 3.2.5 预定义参数名称常量（Param）

**C/C++ 定义位置**：`frameworks/js/napi/src/napi_hiappevent_init.cpp:80-85`

| 常量名 | 参数名称字符串 | 说明 |
|-------|---------------|------|
| USER_ID | "user_id" | 用户自定义 ID |
| DISTRIBUTED_SERVICE_NAME | "ds_name" | 分布式服务名称 |
| DISTRIBUTED_SERVICE_INSTANCE_ID | "ds_instance_id" | 分布式服务实例 ID |

### 3.2.6 领域常量（Domain）

**C/C++ 定义位置**：`frameworks/js/napi/src/napi_hiappevent_init.cpp:87-90`（仅 v9 版本）

| 常量名 | 领域名称字符串 | 说明 |
|-------|---------------|------|
| OS | "OS" | 系统内置领域 |

## 3.3 Native API（C/C++）清单

### 3.3.1 打点核心接口

#### OH_HiAppEvent_Write()

**头文件**：`interfaces/native/kits/include/hiappevent/hiappevent.h:477`

**功能描述**：实现应用事件的写入，参数以链表形式组织。

**函数签名**：
```c
int OH_HiAppEvent_Write(const char* domain, const char* name, 
                        enum EventType type, const ParamList list);
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|:----:|------|
| domain | const char* | 是 | 事件领域字符串，用于标识事件的来源域 |
| name | const char* | 是 | 事件名称字符串 |
| type | enum EventType | 是 | 事件类型枚举值 |
| list | const ParamList | 是 | 事件参数列表，指向链表头节点的指针 |

**返回值说明**：

| 返回值类型 | 返回值范围 | 含义 |
|-----------|-----------|------|
| int | 0 | 事件参数校验成功，事件正常写入 |
| int | 正整数 | 事件存在异常参数，异常参数被忽略后正常写入 |
| int | 负整数 | 事件校验失败，不执行写入操作 |

### 3.3.2 参数列表构造接口

#### OH_HiAppEvent_CreateParamList()

**头文件**：`interfaces/native/kits/include/hiappevent/hiappevent.h:248`

**功能描述**：创建一个空的 ParamList 节点。

**函数签名**：
```c
ParamList OH_HiAppEvent_CreateParamList(void);
```

**返回值**：指向新创建 ParamList 节点的指针，若创建失败返回 nullptr。

#### OH_HiAppEvent_DestroyParamList()

**头文件**：`interfaces/native/kits/include/hiappevent/hiappevent.h:257`

**功能描述**：销毁 ParamList，释放其占用的内存。

**函数签名**：
```c
void OH_HiAppEvent_DestroyParamList(ParamList list);
```

#### 参数添加接口族

HiAppEvent 提供了一系列参数添加接口，支持不同类型的数据：

| 函数名 | 头文件位置 | 参数类型 |
|-------|-----------|---------|
| OH_HiAppEvent_AddBoolParam | :269 | bool |
| OH_HiAppEvent_AddBoolArrayParam | :282 | bool[] |
| OH_HiAppEvent_AddInt8Param | :294 | int8_t |
| OH_HiAppEvent_AddInt8ArrayParam | :307 | int8_t[] |
| OH_HiAppEvent_AddInt16Param | :319 | int16_t |
| OH_HiAppEvent_AddInt16ArrayParam | :332 | int16_t[] |
| OH_HiAppEvent_AddInt32Param | :344 | int32_t |
| OH_HiAppEvent_AddInt32ArrayParam | :357 | int32_t[] |
| OH_HiAppEvent_AddInt64Param | :369 | int64_t |
| OH_HiAppEvent_AddInt64ArrayParam | :382 | int64_t[] |
| OH_HiAppEvent_AddFloatParam | :394 | float |
| OH_HiAppEvent_AddFloatArrayParam | :407 | float[] |
| OH_HiAppEvent_AddDoubleParam | :419 | double |
| OH_HiAppEvent_AddDoubleArrayParam | :432 | double[] |
| OH_HiAppEvent_AddStringParam | :444 | const char* |
| OH_HiAppEvent_AddStringArrayParam | :457 | const char*[] |

**通用函数签名**：
```c
ParamList OH_HiAppEvent_AddXxxParam(ParamList list, const char* name, XXX value);
ParamList OH_HiAppEvent_AddXxxArrayParam(ParamList list, const char* name, 
                                          const XXX* values, int arrSize);
```

### 3.3.3 配置接口

#### OH_HiAppEvent_Configure()

**头文件**：`interfaces/native/kits/include/hiappevent/hiappevent.h:491`

**功能描述**：实现应用事件打点功能的配置。

**函数签名**：
```c
bool OH_HiAppEvent_Configure(const char* name, const char* value);
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|:----:|------|
| name | const char* | 是 | 配置项名称，可传入预定义常量 |
| value | const char* | 是 | 配置项值 |

**预定义配置常量**：

| 常量名 | 默认值 | 说明 |
|-------|--------|------|
| DISABLE | "false" | 打点功能开关，"true" 表示关闭 |
| MAX_STORAGE | "10M" | 事件存储目录配额大小 |

### 3.3.4 事件观察者接口

#### 创建与销毁

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_CreateWatcher | 创建 Watcher 实例 | :502 |
| OH_HiAppEvent_DestroyWatcher | 销毁 Watcher 实例 | :512 |

#### 条件配置

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_SetTriggerCondition | 设置触发条件 | :529 |
| OH_HiAppEvent_SetAppEventFilter | 设置事件过滤条件 | :546 |
| OH_HiAppEvent_SetWatcherOnTrigger | 设置触发回调 | :562 |
| OH_HiAppEvent_SetWatcherOnReceive | 设置接收回调 | :576 |

#### 数据获取

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_TakeWatcherData | 获取观察者缓存的事件数据 | :590 |

#### 管理操作

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_AddWatcher | 添加观察者 | :602 |
| OH_HiAppEvent_RemoveWatcher | 移除观察者 | :614 |
| OH_HiAppEvent_ClearData | 清除本地事件数据 | :623 |

### 3.3.5 事件处理器接口

#### 创建与销毁

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_CreateProcessor | 创建 Processor 实例 | :632 |
| OH_HiAppEvent_DestroyProcessor | 销毁 Processor 实例 | :782 |

#### 配置接口

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_SetReportRoute | 设置上报路由 | :648 |
| OH_HiAppEvent_SetReportPolicy | 设置上报策略 | :665 |
| OH_HiAppEvent_SetReportEvent | 设置上报事件 | :682 |
| OH_HiAppEvent_SetCustomConfig | 设置自定义配置 | :699 |
| OH_HiAppEvent_SetConfigId | 设置配置 ID | :713 |
| OH_HiAppEvent_SetConfigName | 设置配置名称 | :728 |
| OH_HiAppEvent_SetReportUserId | 设置用户 ID | :744 |
| OH_HiAppEvent_SetReportUserProperty | 设置用户属性 | :760 |

#### 管理操作

| 接口名 | 功能描述 | 头文件位置 |
|-------|---------|-----------|
| OH_HiAppEvent_AddProcessor | 添加 Processor | :774 |
| OH_HiAppEvent_RemoveProcessor | 移除 Processor | :795 |

### 3.3.6 错误码定义

**头文件**：`interfaces/native/kits/include/hiappevent/hiappevent.h:95-112`

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 0 | HIAPPEVENT_SUCCESS | 操作成功 |
| 4 | HIAPPEVENT_INVALID_PARAM_VALUE_LENGTH | 无效的参数值长度 |
| -7 | HIAPPEVENT_PROCESSOR_IS_NULL | 处理器为空 |
| -8 | HIAPPEVENT_PROCESSOR_NOT_FOUND | 处理器未找到 |
| -9 | HIAPPEVENT_INVALID_PARAM_VALUE | 无效的参数值 |
| -10 | HIAPPEVENT_EVENT_CONFIG_IS_NULL | 事件配置为空 |
| -100 | HIAPPEVENT_OPERATE_FAILED | 操作失败 |
| -200 | HIAPPEVENT_INVALID_UID | 无效的用户 ID |

## 3.4 参数校验流程

HiAppEvent 在事件写入过程中执行多层次的参数校验，确保数据的完整性和安全性。校验流程如下：

```
┌─────────────┐
│ 参数解析    │ ◀── 从 JS/NDK 接口接收参数
└──────┬──────┘
       ▼
┌─────────────┐     ┌───────────────────────────┐
│ 必填项校验  │────▶│ eventName/domain 非空检查  │
└──────┬──────┘     └───────────────────────────┘
       │
       ▼
┌─────────────┐     ┌───────────────────────────┐
│ 长度校验    │────▶│ name ≤ 128, key ≤ 256,    │
│             │     │ value ≤ 4096 (字符串)     │
└──────┬──────┘     └───────────────────────────┘
       │
       ▼
┌─────────────┐     ┌───────────────────────────┐
│ 类型校验    │────▶│ 参数值类型匹配检查         │
└──────┬──────┘     └───────────────────────────┘
       │
       ▼
┌─────────────┐     ┌───────────────────────────┐
│ 白名单校验  │────▶│ 若启用白名单，检查事件是否  │
│ (可选)      │     │ 在白名单中                 │
└──────┬──────┘     └───────────────────────────┘
       │
       ▼
   ┌────┴────┐
   │ 校验结果 │
   └────┬────┘
        │
   ┌────┴─────────────────────────┐
   │                             │
   ▼                             ▼
校验通过                   校验失败
返回 0                  返回负数错误码
```

## 3.5 调用链追踪

### 3.5.1 JS API 调用链

```
JavaScript 调用
    │
    ▼
napi_hiappevent_js.cpp::Write()
    │
    ├── 参数解析：napi_hiappevent_js.cpp:38
    │   └── 解析 eventName, type, keyValues, callback
    │
    ├── 构建 AppEventPack：napi_hiappevent_builder.cpp
    │   └── NapiHiAppEventBuilder::Build()
    │
    ├── 事件校验：napi_hiappevent_js.cpp:55
    │   └── VerifyAppEvent()
    │
    └── 异步写入：napi_hiappevent_write.cpp
        └── HiAppEventAsyncContext::Execute()
            │
            ├── FFRT 调度
            │
            ├── 事件分发：observer 模块
            │   └── AppEventObserverMgr::Dispatch()
            │
            ├── 事件缓存：cache 模块
            │   └── AppEventDao::SaveEvent()
            │
            └── 持久化存储：app_event_store.cpp
                └── 写入文件/数据库
```

### 3.5.2 Native API 调用链

```
C/C++ 调用
    │
    ▼
hiappevent.cpp::OH_HiAppEvent_Write()
    │
    ├── 参数校验
    │   └── hiappevent_verify.cpp::VerifyAppEvent()
    │
    ├── 事件分发
    │   └── AppEventObserverMgr::NotifyEvent()
    │
    └── 事件存储
        └── AppEventDao::SaveEvent()
            │
            ├── JSON 序列化：event_json_util.cpp
            │
            ├── 文件写入：file_util.cpp
            │
            └── 数据库写入：app_event_db_cleaner.cpp
```

## 3.6 Inner API（C++）参考

### 3.6.1 HiAppEvent::Event 类

**头文件**：`interfaces/native/inner_api/include/app_event.h`

**C++ 接口示例**：
```cpp
#include "app_event.h"

using namespace OHOS::HiviewDFX::HiAppEvent;

// 创建事件
Event event("hiappevent", "user_login", BEHAVIOR);

// 添加参数
event.AddParam("user_id", "U10001");
event.AddParam("login_type", "password");

// 写入事件
int result = Write(event);
```

### 3.6.2 Inner API 与 N-API 的关系

| Inner API | N-API 对应 | 差异说明 |
|-----------|-----------|---------|
| Event::Write() | hiAppEvent.write() | Inner API 为同步接口，N-API 为异步接口 |
| Event::AddParam() | keyValues 对象 | Inner API 支持链式调用，N-API 使用对象属性 |
