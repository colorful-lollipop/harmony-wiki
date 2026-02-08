# N-API 参考 (JavaScript/ArkTS)

## 模块概述

HiTrace 提供三个 N-API 模块供 JavaScript/ArkTS 调用：

| 模块名 | JS 命名空间 | 导出文件 | 说明 |
|--------|------------|----------|------|
| hiTraceChain | `@ohos.hiTraceChain` | `libhitracechain_napi.so` | 调用链追踪核心 |
| hiTraceMeter | `@ohos.hiTraceMeter` | `libhitracemeter_napi.so` | 性能追踪测量 |
| bytrace | `@ohos.bytrace` | `libbytrace_napi.so` | 遗留兼容接口 |

**证据来源**:
- `interfaces/js/kits/napi/src/napi_hitrace_js.cpp:316-328` - 模块注册
- `interfaces/js/kits/napi/hitracemeter/napi_hitrace_meter.cpp:522-538` - Meter 模块注册
- `BUILD.gn:17-69` - N-API 编译配置

## hiTraceChain 模块

### 模块注册

**注册位置**: `interfaces/js/kits/napi/src/napi_hitrace_js.cpp:316-328`

```cpp
static napi_module hitrace_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = TraceNapiInit,
    .nm_modname = "hiTraceChain",
    .nm_priv = (reinterpret_cast<void *>(0)),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&hitrace_module);
}
```

### JS API 清单

#### 1. begin - 开始追踪

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `begin(name: string, flags?: number): HiTraceId` |
| **Native 函数** | `Begin()` (line 73) |
| **同步/异步** | 同步 |
| **参数** | `name: string` (必填), `flags: number` (可选, 默认 HITRACE_FLAG_DEFAULT) |
| **返回值** | `HiTraceId` 对象 |

**参数校验** (`napi_hitrace_js.cpp:88-92`):
```cpp
if (!ParseStringParam(env, params[ParamIndex::PARAM_FIRST], name)) {
    HILOG_ERROR(LOG_CORE, "name type must be string.");
    return val;  // 返回无效 HiTraceId
}
```

**Native 调用** (`napi_hitrace_js.cpp:100`):
```cpp
traceId = HiTraceChain::Begin(name, flag, LOG_DOMAIN);
```

**使用示例**:
```typescript
import hiTraceChain from '@ohos.hiTraceChain';

const traceId = hiTraceChain.begin("myOperation");
hiTraceChain.end(traceId);
```

#### 2. end - 结束追踪

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `end(id: HiTraceId): void` |
| **Native 函数** | `End()` (line 105) |
| **同步/异步** | 同步 |
| **参数** | `id: HiTraceId` (必填) |
| **返回值** | `void` |

**参数校验** (`napi_hitrace_js.cpp:117-121`):
```cpp
if (!ParseTraceIdObject(env, params[ParamIndex::PARAM_FIRST], traceId)) {
    HILOG_ERROR(LOG_CORE, "hitarce id type must be object.");
    return nullptr;
}
```

**Native 调用** (`napi_hitrace_js.cpp:122`):
```cpp
HiTraceChain::End(traceId, LOG_DOMAIN);
```

#### 3. getId - 获取当前 TraceId

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `getId(): HiTraceId` |
| **Native 函数** | `GetId()` (line 126) |
| **同步/异步** | 同步 |
| **参数** | 无 |
| **返回值** | `HiTraceId` |

**Native 调用** (`napi_hitrace_js.cpp:128`):
```cpp
HiTraceId traceId = HiTraceChain::GetId();
```

#### 4. setId - 设置 TraceId

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `setId(id: HiTraceId): void` |
| **Native 函数** | `SetId()` (line 134) |
| **同步/异步** | 同步 |
| **参数** | `id: HiTraceId` (必填) |
| **返回值** | `void` |

#### 5. clearId - 清除 TraceId

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `clearId(): void` |
| **Native 函数** | `ClearId()` (line 155) |
| **同步/异步** | 同步 |
| **参数** | 无 |
| **返回值** | `void` |

**Native 调用** (`napi_hitrace_js.cpp:157`):
```cpp
HiTraceChain::ClearId();
```

#### 6. createSpan - 创建 Span

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `createSpan(): HiTraceId` |
| **Native 函数** | `CreateSpan()` (line 161) |
| **同步/异步** | 同步 |
| **参数** | 无 |
| **返回值** | `HiTraceId` (新的子 Span) |

**Native 调用** (`napi_hitrace_js.cpp:163`):
```cpp
HiTraceId traceId = HiTraceChain::CreateSpan();
```

#### 7. tracepoint - 输出追踪点

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `tracepoint(mode: HiTraceCommunicationMode, type: HiTraceTracePointType, id: HiTraceId, description?: string): void` |
| **Native 函数** | `Tracepoint()` (line 169) |
| **同步/异步** | 同步 |
| **参数** | `mode: HiTraceCommunicationMode` (必填), `type: HiTraceTracePointType` (必填), `id: HiTraceId` (必填), `description: string` (可选) |
| **返回值** | `void` |

**通信模式常量**:
```typescript
HiTraceCommunicationMode.DEFAULT  = 0  // 默认
HiTraceCommunicationMode.THREAD  = 1  // 线程间
HiTraceCommunicationMode.PROCESS = 2  // 进程间 (IPC)
HiTraceCommunicationMode.DEVICE = 3  // 设备间
```

**追踪点类型常量**:
```typescript
HiTraceTracePointType.CS      = 0  // 客户端发送
HiTraceTracePointType.CR      = 1  // 客户端接收
HiTraceTracePointType.SS      = 2  // 服务端发送
HiTraceTracePointType.SR      = 3  // 服务端接收
HiTraceTracePointType.GENERAL = 4  // 通用信息
```

#### 8. isValid - 检查 TraceId 有效性

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `isValid(id: HiTraceId): boolean` |
| **Native 函数** | `IsValid()` (line 209) |
| **同步/异步** | 同步 |
| **参数** | `id: HiTraceId` (必填) |
| **返回值** | `boolean` |

**Native 调用** (`napi_hitrace_js.cpp:229`):
```cpp
isValid = traceId.IsValid();
```

#### 9. isFlagEnabled - 检查标志位

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `isFlagEnabled(id: HiTraceId, flag: HiTraceFlag): boolean` |
| **Native 函数** | `IsFlagEnabled()` (line 234) |
| **同步/异步** | 同步 |
| **参数** | `id: HiTraceId` (必填), `flag: HiTraceFlag` (必填) |
| **返回值** | `boolean` |

#### 10. enableFlag - 启用标志位

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `enableFlag(id: HiTraceId, flag: HiTraceFlag): void` |
| **Native 函数** | `EnableFlag()` (line 265) |
| **同步/异步** | 同步 |
| **参数** | `id: HiTraceId` (必填), `flag: HiTraceFlag` (必填) |
| **返回值** | `void` |

### HiTraceId 对象结构

JS 层 HiTraceId 对象包含以下属性：

| 属性名 | 类型 | 说明 |
|--------|------|------|
| `chainId` | number (BigInt) | 调用链 ID |
| `spanId` | number | Span ID |
| `parentSpanId` | number | 父 Span ID |
| `flags` | number | 追踪标志位 |

**证据来源**: `interfaces/js/kits/napi/src/napi_hitrace_util.cpp` - 对象转换函数

### HiTraceFlag 枚举

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `DEFAULT` | 0 | 默认行为 |
| `INCLUDE_ASYNC` | 1 | 追踪异步调用 |
| `DONOT_CREATE_SPAN` | 2 | 不创建 Span |
| `TP_INFO` | 4 | 输出追踪点信息 |
| `NO_BE_INFO` | 8 | 不输出开始/结束信息 |
| `DISABLE_LOG` | 16 | 不关联日志 |
| `FAULT_TRIGGER` | 32 | 故障触发 |
| `D2D_TP_INFO` | 64 | 跨设备追踪点信息 |

**证据来源**: `interfaces/js/kits/napi/src/napi_hitrace_init.cpp:1-200` - Enum 类初始化

---

## hiTraceMeter 模块

### 模块注册

**注册位置**: `interfaces/js/kits/napi/hitracemeter/napi_hitrace_meter.cpp:522-538`

```cpp
static napi_module hitracemeter_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = HiTraceMeterInit,
    .nm_modname = "hiTraceMeter",
    .nm_priv = (reinterpret_cast<void *>(0)),
    .reserved = {0}
};
```

### JS API 清单

#### 1. startTrace - 开始追踪

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `startTrace(name: string, taskId: number): void` |
| **Native 函数** | `JSTraceStart()` |
| **同步/异步** | 同步 |
| **参数** | `name: string` (必填), `taskId: number` (必填) |
| **返回值** | `void` |

#### 2. finishTrace - 结束追踪

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `finishTrace(taskId: number): void` |
| **Native 函数** | `JSTraceFinish()` |
| **同步/异步** | 同步 |
| **参数** | `taskId: number` (必填) |
| **返回值** | `void` |

#### 3. traceByValue - 计数追踪

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `traceByValue(name: string, count: number): void` |
| **Native 函数** | `JSTraceCount()` |
| **同步/异步** | 同步 |
| **参数** | `name: string` (必填), `count: number` (必填) |
| **返回值** | `void` |

#### 4. isTraceEnabled - 检查追踪状态

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `isTraceEnabled(): boolean` |
| **Native 函数** | `JSIsTraceEnabled()` |
| **同步/异步** | 同步 |
| **参数** | 无 |
| **返回值** | `boolean` |

#### 5. registerTraceListener - 注册追踪监听器

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `registerTraceListener(listener: HiTraceOutputListener): void` |
| **Native 函数** | `JSRegisterTraceListener()` |
| **同步/异步** | 同步 |
| **参数** | `listener: HiTraceOutputListener` (必填) |
| **返回值** | `void` |

#### 6. unregisterTraceListener - 取消注册监听器

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `unregisterTraceListener(): void` |
| **Native 函数** | `JSUnregisterTraceListener()` |
| **同步/异步** | 同步 |
| **参数** | 无 |
| **返回值** | `void` |

### HiTraceOutputLevel 枚举

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `DEBUG` | 0 | 调试级别 |
| `INFO` | 1 | 信息级别 |
| `CRITICAL` | 2 | 关键级别 |
| `COMMERCIAL` | 3 | 商用级别 |
| `MAX` | 4 | 最大级别 |

---

## bytrace 模块（遗留）

### 模块注册

**注册位置**: `interfaces/js/kits/napi/bytrace_napi/bytrace_napi_common.cpp:230-249`

```cpp
static napi_module bytrace_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = BytraceInit,
    .nm_modname = "bytrace",
    .nm_priv = (reinterpret_cast<void *>(0)),
    .reserved = {0}
};
```

### JS API 清单

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `startTrace` | `name: string, taskId: number` | 开始异步追踪 |
| `finishTrace` | `taskId: number` | 结束异步追踪 |
| `traceByValue` | `name: string, count: number` | 计数追踪 |

**注意**: 此模块为遗留兼容接口，建议新代码使用 `hiTraceMeter` 模块。

---

## 错误码与异常处理

### 错误处理策略

N-API 层采用静默失败策略：

1. **参数校验失败**: 记录错误日志，返回无效/默认值
2. **Native 调用失败**: 同样返回无效/默认值
3. **不抛出异常**: 与 Node.js N-API 模式一致

**证据来源**: `napi_hitrace_js.cpp:84-99` - 参数校验失败仅记录日志

```cpp
if (!ParseStringParam(env, params[ParamIndex::PARAM_FIRST], name)) {
    HILOG_ERROR(LOG_CORE, "name type must be string.");
    return val;  // 返回无效 HiTraceId，不抛出异常
}
```

### 日志域

| 模块 | LOG_DOMAIN | LOG_TAG |
|------|------------|---------|
| hiTraceChain | 0xD002D33 | "HitraceChainNapi" |
| hiTraceMeter | 0xD002D33 | "HitraceMeterNapi" |
| bytrace | - | "HITRACE_METER_JS" |

**证据来源**:
- `napi_hitrace_js.cpp:27-31`
- `napi_hitrace_meter.cpp:48-53`

---

## 调用链示例

### 基础调用链追踪

```typescript
import hiTraceChain from '@ohos.hiTraceChain';

// 开始追踪
const traceId = hiTraceChain.begin("userLogin", 
    hiTraceChain.HiTraceFlag.DEFAULT);

// 检查 TraceId 是否有效
if (hiTraceChain.isValid(traceId)) {
    // 设置 TraceId 到当前线程
    hiTraceChain.setId(traceId);
    
    // 执行业务逻辑
    // ...
    
    // 创建子 Span
    const spanId = hiTraceChain.createSpan();
    
    // 输出追踪点
    hiTraceChain.tracepoint(
        hiTraceChain.HiTraceCommunicationMode.PROCESS,
        hiTraceChain.HiTraceTracePointType.CS,
        traceId,
        "用户登录请求"
    );
    
    // 结束追踪
    hiTraceChain.end(traceId);
}
```

### 跨进程追踪

```typescript
import hiTraceChain from '@ohos.hiTraceChain';

const traceId = hiTraceChain.begin("rpcCall");

// 将 TraceId 序列化后通过 IPC 发送
// IPC 框架会自动处理 TraceId 的传递

// 在服务端接收并设置
hiTraceChain.setId(traceId);

// 输出追踪点
hiTraceChain.tracepoint(
    hiTraceChain.HiTraceCommunicationMode.PROCESS,
    hiTraceChain.HiTraceTracePointType.SR,
    traceId,
    "收到 RPC 请求"
);
```

---

## 注意事项

1. **参数校验**: 所有 API 都会进行类型校验，类型错误时返回无效值
2. **嵌套追踪**: 在已有追踪中再次调用 `begin()` 会返回无效 HiTraceId
3. **线程安全**: TLS 存储机制保证每个线程独立追踪状态
4. **性能影响**: 开启过多追踪标志位会影响性能
