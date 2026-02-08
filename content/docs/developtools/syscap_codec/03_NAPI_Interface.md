# 03_NAPI_Interface - N-API 接口说明

## 目的

本文档详细介绍 `syscap_codec` 提供的 N-API 和 ANI 接口，包括 API 清单、调用链、参数校验和错误码。

## 适用范围

- 需要调用系统能力查询接口的 JS 开发者
- 需要理解 JS 接口实现的 Native 开发者

## 接口概览

本项目提供两套 JS 接口：

| 接口类型 | 位置 | 技术 | 状态 |
|----------|------|------|------|
| N-API | `napi/` | 传统 N-API | 稳定 |
| ANI/Taihe | `taihe/` | ArkCompiler Native Interface | 新引入 |

## N-API 接口

### 模块信息

**模块名**: `systemCapability`
**JS入口**: `napi/query_syscap.js`
**Native实现**: `napi/napi_query_syscap.cpp`

### API 清单

| JS API | 同步/异步 | 参数 | 返回值 | 对应Native函数 |
|--------|-----------|------|--------|----------------|
| `querySystemCapabilities(callback?)` | 异步 | callback?: Function | Promise<string> \| void | `QuerySystemCapability()` |

### 详细说明

#### querySystemCapabilities

**功能**: 查询设备支持的所有系统能力

**调用位置**: `napi/query_syscap.js:21`

**JS签名**:
```javascript
function querySystemCapabilities(callback?: (err: Error, syscap: string) => void): Promise<string> | void
```

**参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| callback | Function | 否 | 回调函数，如果不传则返回Promise |

**回调参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| err | Error | 错误对象，成功时为null |
| syscap | string | 系统能力字符串 |

**返回值格式**:
```
"header1,header2,u32_1,u32_2,...,u32_30,private1,private2,..."
```

示例:
```
"0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,SystemCapability.Customization.ConfigPolicy"
```

**使用示例**:

```javascript
// Promise 方式
import systemCapability from '@ohos.systemCapability';

try {
    const syscap = await systemCapability.querySystemCapabilities();
    console.log('System capabilities:', syscap);
} catch (err) {
    console.error('Failed to query:', err);
}

// Callback 方式
systemCapability.querySystemCapabilities((err, syscap) => {
    if (err) {
        console.error('Failed to query:', err);
        return;
    }
    console.log('System capabilities:', syscap);
});
```

### 注册点

**模块注册**: `napi/napi_query_syscap.cpp:244-260`

```cpp
static napi_module g_systemCapabilityModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = QuerryExport,
    .nm_modname = "systemCapability",
    .nm_priv = nullptr,
    .reserved = {nullptr},
};

extern "C" __attribute__((constructor)) void SystemCapabilityRegisterModule(void)
{
    napi_module_register(&g_systemCapabilityModule);
}
```

**导出函数**: `napi/napi_query_syscap.cpp:230-238`

```cpp
napi_value QuerryExport(napi_env env, napi_value exports)
{
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("querySystemCapabilities", QuerySystemCapability),
    };
    NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc));
    return exports;
}
```

### 调用链

```
JS: querySystemCapabilities()
    │
    ▼
napi/napi_query_syscap.cpp: QuerySystemCapability()
    │
    ├──▶ PreHandleSystemCapability() [napi_query_syscap.cpp:152]
    │       ├── 解析参数 (napi_get_cb_info)
    │       ├── 判断是否有callback
    │       └── 创建 Promise 或保存 callback
    │
    ├──▶ napi_create_async_work() [napi_query_syscap.cpp:184]
    │       ├── 执行函数: GetSystemCapability() (工作线程)
    │       └── 完成回调: 返回结果 (主线程)
    │
    └──▶ napi_queue_async_work() [napi_query_syscap.cpp:225]
            │
            ▼ (工作线程)
            GetSystemCapability() [napi_query_syscap.cpp:110]
                │
                ├──▶ EncodeOsSyscap() [syscap_interface.c:81]
                │       └── 读取 /system/etc/pcid.sc
                │
                ├──▶ EncodePrivateSyscap() [syscap_interface.c:110]
                │       └── 读取 /system/etc/pcid.sc
                │
                ├──▶ DecodePrivateSyscap() [syscap_interface.c:221]
                │       └── 解码私有syscap
                │
                └──▶ CalculateAllStringLength() [napi_query_syscap.cpp:53]
                        └── 组装最终字符串
            │
            ▼ (主线程回调)
            完成回调函数 [napi_query_syscap.cpp:196]
                ├── 创建JS字符串返回值
                ├── 如果有deferred: napi_resolve_deferred()
                └── 如果有callback: napi_call_function()
```

### 参数校验

**参数数量检查**: `napi/napi_query_syscap.cpp:156`
```cpp
NAPI_ASSERT(env, argc <= 1, "too many parameters");
```

**参数类型检查**: `napi/napi_query_syscap.cpp:161-164`
```cpp
napi_valuetype valueType = napi_undefined;
if (argc == 1) {
    napi_typeof(env, argv[0], &valueType);
}
if (valueType == napi_function) {
    napi_create_reference(env, argv[0], 1, &asyncContext->callbackRef);
}
```

**校验规则**:
- 最多接受1个参数
- 如果提供参数，必须是函数类型（callback）
- 不传参数则返回 Promise

### 错误码

| 错误场景 | 错误信息 | 返回方式 |
|----------|----------|----------|
| 参数过多 | "too many parameters" | 抛出异常 |
| 获取系统能力失败 | "key does not exist" | Promise reject / callback(err) |

**错误处理代码**: `napi/napi_query_syscap.cpp:201-206`
```cpp
if (!asyncContext->status) {
    napi_get_undefined(env, &result[0]);
    napi_create_string_utf8(env, asyncContext->value, strlen(asyncContext->value), &result[1]);
} else {
    napi_value message = nullptr;
    napi_create_string_utf8(env, "key does not exist", NAPI_AUTO_LENGTH, &message);
    napi_create_error(env, nullptr, message, &result[0]);
    napi_get_undefined(env, &result[1]);
}
```

### 权限/前置条件

**运行时依赖**:
- 文件 `/system/etc/pcid.sc` 必须存在且可读
- 需要系统权限才能读取该文件

**无显式权限声明**: 本接口在 `bundle.json` 中没有声明权限要求，但依赖文件系统权限。

## ANI/Taihe 接口

### 模块信息

**命名空间**: `@ohos.systemCapability`
**IDL定义**: `taihe/syscap/idl/ohos.systemCapability.taihe`
**Native实现**: `taihe/syscap/src/ohos.systemCapability.impl.cpp`

### IDL 定义

**文件**: `taihe/syscap/idl/ohos.systemCapability.taihe`

```
@!namespace("@ohos.systemCapability", "systemCapability")

@!sts_inject("""
static { loadLibraryWithPermissionCheck("systemCapability_taihe_native.z", "@ohos.systemCapability") }
""")

@gen_async("querySystemCapabilities")
@gen_promise("querySystemCapabilities")
function querySystemCapabilitie(): String;
```

**注解说明**:
- `@!namespace`: 定义命名空间
- `@!sts_inject`: 注入静态加载代码，带权限检查
- `@gen_async`: 生成异步版本
- `@gen_promise`: 生成Promise版本

### API 清单

| JS API | 同步/异步 | 参数 | 返回值 | 对应Native函数 |
|--------|-----------|------|--------|----------------|
| `querySystemCapabilitie()` | 同步 | 无 | string | `querySystemCapabilitie()` |
| `querySystemCapabilitiesAsync()` | 异步 | 无 | Promise<string> | 自动生成 |
| `querySystemCapabilitiesPromise()` | Promise | 无 | Promise<string> | 自动生成 |

### 详细说明

#### querySystemCapabilitie

**功能**: 同步查询设备支持的所有系统能力

**调用位置**: `taihe/syscap/src/ohos.systemCapability.impl.cpp:144`

**JS签名**:
```typescript
function querySystemCapabilitie(): string;
```

**返回值**: 同 N-API 版本

**使用示例**:
```typescript
import { querySystemCapabilitie } from '@ohos.systemCapability';

try {
    const syscap = querySystemCapabilitie();
    console.log('System capabilities:', syscap);
} catch (err) {
    console.error('Failed to query:', err);
}
```

### 注册点

**ANI构造函数**: `taihe/syscap/src/ani_constructor.cpp:17`

```cpp
ANI_EXPORT ani_status ANI_Constructor(ani_vm *vm, uint32_t *result)
{
    ani_env *env;
    if (ANI_OK != vm->GetEnv(ANI_VERSION_1, &env)) {
        return ANI_ERROR;
    }
    if (ANI_OK != ohos::systemCapability::ANIRegister(env)) {
        std::cerr << "Error from ohos::systemCapability::ANIRegister" << std::endl;
        return ANI_ERROR;
    }
    *result = ANI_VERSION_1;
    return ANI_OK;
}
```

**导出宏**: `taihe/syscap/src/ohos.systemCapability.impl.cpp:167`
```cpp
TH_EXPORT_CPP_API_querySystemCapabilitie(querySystemCapabilitie);
```

### 调用链

```
JS: querySystemCapabilitie()
    │
    ▼
taihe/syscap/src/ohos.systemCapability.impl.cpp: querySystemCapabilitie() [line:144]
    │
    ├──▶ new SystemCapabilityAsyncContext()
    │
    ├──▶ GetSystemCapability() [line:147]
    │       │
    │       ├──▶ EncodeOsSyscap() [syscap_interface.c:81]
    │       │       └── 读取 /system/etc/pcid.sc
    │       │
    │       ├──▶ EncodePrivateSyscap() [syscap_interface.c:110]
    │       │       └── 读取 /system/etc/pcid.sc
    │       │
    │       ├──▶ DecodePrivateSyscap() [syscap_interface.c:221]
    │       │       └── 解码私有syscap
    │       │
    │       └──▶ CalculateAllStringLength() [line:43]
    │               └── 组装最终字符串
    │
    ├──▶ 检查结果状态
    │       ├── 成功: 返回字符串
    │       └── 失败: set_business_error(-1, "key does not exist")
    │
    └──▶ delete asyncContext
```

### 错误处理

**业务错误**: `taihe/syscap/src/ohos.systemCapability.impl.cpp:158`
```cpp
if (!asyncContext->status) {
    value = asyncContext->value;
} else {
    taihe::set_business_error(-1, "key does not exist");
}
```

**错误码**: `-1`
**错误信息**: `"key does not exist"`

### 权限检查

**加载时权限检查**: IDL中的 `@!sts_inject` 注解
```
static { loadLibraryWithPermissionCheck("systemCapability_taihe_native.z", "@ohos.systemCapability") }
```

这表明 Taihe 接口在加载时会进行权限检查。

## 接口对比

| 特性 | N-API | ANI/Taihe |
|------|-------|-----------|
| 执行方式 | 异步（线程池） | 同步 |
| 调用方式 | Promise / Callback | 直接调用 |
| 权限检查 | 运行时文件权限 | 加载时权限检查 |
| 错误处理 | Promise reject / callback err | 异常抛出 |
| 代码生成 | 手动编写 | IDL生成部分代码 |
| 维护状态 | 稳定 | 新引入 |

## 相关跳转

- [项目概览](00_Overview.md) - 系统能力概念说明
- [架构说明](02_Architecture.md) - 线程模型和时序
- [内部API](04_Inner_API.md) - 底层C/C++接口
- [安全风险分析](06_Security_Analysis.md) - 接口安全风险
