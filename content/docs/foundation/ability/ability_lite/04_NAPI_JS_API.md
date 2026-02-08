# N-API / JS API

## 目的

本文档描述 ability_lite 提供的 JavaScript API 绑定（N-API），供 JS 应用开发使用。

## 适用范围

- JavaScript 应用开发者
- 需要了解 JS 与 Native 交互机制的开发者

## N-API 模块注册

**模块名**: `aafwk`

**代码位置**: `interfaces/kits/js/napi/js_aafwk.cpp:171-184`

```cpp
static napi_module aafwk_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = AafwkExport,
    .nm_modname = "aafwk",              // JS 模块名
    .nm_priv = ((void*)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void AafwkRegister()
{
    napi_module_register(&aafwk_module);
}
```

## JS API 清单表

### 导出函数

**代码位置**: `interfaces/kits/js/napi/js_aafwk.cpp:159-167`

```cpp
napi_value AafwkExport(napi_env env, napi_value exports)
{
    static napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("startAbility", JSAafwkStartAbility),
        DECLARE_NAPI_FUNCTION("stopAbility", JSAafwkStopAbility),
    };
    NAPI_CALL(env, napi_define_properties(env, exports, 
        sizeof(desc) / sizeof(desc[0]), desc));
    return exports;
}
```

| JS API 名称 | 对应 C++ 函数 | 参数 | 返回值 | 说明 |
|-------------|---------------|------|--------|------|
| `startAbility(want)` | `JSAafwkStartAbility` | want: Want 对象 | number | 启动 Ability |
| `stopAbility(want)` | `JSAafwkStopAbility` | want: Want 对象 | number | 停止 Ability |

## TypeScript 声明

**文件位置**: `interfaces/kits/js/declaration/api/@ohos.aafwk.d.ts`

```typescript
declare namespace aafwk {
    /**
     * 启动 Ability
     * @param want 包含目标 Ability 信息的对象
     * @returns 操作结果码，0 表示成功
     */
    function startAbility(want: Want): number;

    /**
     * 停止 Ability
     * @param want 包含目标 Ability 信息的对象
     * @returns 操作结果码，0 表示成功
     */
    function stopAbility(want: Want): number;
}

export default aafwk;
```

## 参数解析与校验

### Want 对象结构

JS 侧传递的 Want 对象结构：

```javascript
{
    want_param: any,           // 自定义参数（可选）
    elementName: {
        deviceId: string,      // 设备 ID
        bundleName: string,    // 包名
        abilityName: string    // Ability 名
    }
}
```

### 参数校验流程

**代码位置**: `interfaces/kits/js/napi/js_aafwk.cpp:43-65`

```cpp
static int JSAafwkStartAbility(napi_env env, napi_callback_info info)
{
    // 1. 获取参数信息
    size_t argc = 1;
    napi_value args[1] = {0};
    napi_status status = napi_get_cb_info(env, info, &argc, args, NULL, NULL);
    assert(status == napi_ok);
    
    // 2. 类型检查
    napi_valuetype types[1];
    status = napi_typeof(env, args[0], types);
    assert(status == napi_ok);
    assert(argc == 1 && types[0] == napi_object);
    
    // 3. 解析 Want 对象
    Want want;
    if (memset_s(&want, sizeof(Want), 0x00, sizeof(Want)) != 0) {
        return MEMORY_MALLOC_ERROR;
    }
    if (GetWantFromNapiValue(env, args[0], want) == false) {
        return PARAM_CHECK_ERROR;
    }
    
    // 4. 调用 Native API
    StartAbility(&want);
    ClearWant(&want);
}
```

### GetWantFromNapiValue 实现

**代码位置**: `interfaces/kits/js/napi/js_aafwk.cpp:91-133`

```cpp
static bool GetWantFromNapiValue(napi_env env, napi_value args, Want& want)
{
    ElementName element;
    if (memset_s(&element, sizeof(ElementName), 0x00, sizeof(ElementName)) != 0) {
        return MEMORY_MALLOC_ERROR;
    }

    // 获取 want_param
    napi_value data;
    napi_get_named_property(env, args, "want_param", &data);

    // 获取 elementName
    napi_value elementName;
    if (napi_get_named_property(env, args, "elementName", &elementName) != napi_ok) {
        return COMMAND_ERROR;
    }

    // 解析 deviceId
    napi_value napi_deviceId;
    napi_get_named_property(env, elementName, "deviceId", &napi_deviceId);
    char *deviceId = nullptr;
    GetCharPointerArgument(env, napi_deviceId, deviceId);
    SetElementDeviceID(&element, deviceId);
    free(deviceId);

    // 解析 bundleName
    napi_value napi_bundleName;
    napi_get_named_property(env, elementName, "bundleName", &napi_bundleName);
    char *bundleName = nullptr;
    GetCharPointerArgument(env, napi_bundleName, bundleName);
    SetElementBundleName(&element, bundleName);
    free(bundleName);

    // 解析 abilityName
    napi_value napi_abilityName;
    napi_get_named_property(env, elementName, "abilityName", &napi_abilityName);
    char *abilityName = nullptr;
    GetCharPointerArgument(env, napi_abilityName, abilityName);
    SetElementAbilityName(&element, abilityName);
    free(abilityName);

    // 设置 Want
    SetWantData(&want, (void *)data, sizeof(data));
    SetWantElement(&want, element);
    ClearElement(&element);
}
```

### 字符串参数提取

**代码位置**: `interfaces/kits/js/napi/js_aafwk.cpp:136-155`

```cpp
static bool GetCharPointerArgument(napi_env env, napi_value value, char* result)
{
    napi_status status;
    size_t bufLength = 0;
    result = nullptr;
    bool ret = false;
    
    // 1. 先获取字符串长度
    status = napi_get_value_string_utf8(env, value, nullptr, 0, &bufLength);
    if (status == napi_ok && bufLength > 0) {
        // 2. 分配缓冲区
        result = (char *) malloc((bufLength + 1) * sizeof(char));
        if (result != nullptr) {
            // 3. 提取字符串
            status = napi_get_value_string_utf8(env, value, result, 
                bufLength + 1, &bufLength);
            if (status == napi_ok) {
                ret = true;
            }
        }
    }
    return ret;
}
```

## 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERR_OK` | 0 | 成功 |
| `MEMORY_MALLOC_ERROR` | 2 | 内存分配失败 |
| `PARAM_CHECK_ERROR` | 9 | 参数校验失败 |
| `COMMAND_ERROR` | 0x7fff | 通用命令错误 |

## 同步/异步模式

当前实现为**同步模式**：
- `JSAafwkStartAbility` 直接调用 `StartAbility()` 并返回
- `JSAafwkStopAbility` 直接调用 `StopAbility()` 并返回
- 无 Promise/Callback 异步处理

**注意**: 代码中定义了 `SetTimeAsyncContext` 结构但未使用，异步功能待实现。

## 调用链分析

### startAbility 调用链

```
JS 层:
  aafwk.startAbility(want)
    │
    ▼
N-API 层:
  js_aafwk.cpp:JSAafwkStartAbility()
    ├── napi_get_cb_info()          # 获取 JS 参数
    ├── napi_typeof()               # 类型检查
    ├── GetWantFromNapiValue()      # 解析 Want 对象
    │   ├── napi_get_named_property("elementName")
    │   ├── GetCharPointerArgument("deviceId")
    │   ├── GetCharPointerArgument("bundleName")
    │   └── GetCharPointerArgument("abilityName")
    │
    ▼
Native Framework:
  ability_manager.h:StartAbility()
    │
    ▼
IPC Client:
  abilityms_client.cpp:SendRequest(START_ABILITY)
    │
    ▼
AMS Service:
  ability_mgr_feature.cpp:StartAbilityInvoke()
    ├── GetCallingUid()             # 权限检查
    ├── StartAbilityInner()
    │
    ▼
  ability_mgr_handler.cpp:ServiceMsgProcess()
    │
    ▼
  ability_worker.cpp:PostTask()
    │
    ▼
  ability_start_task.cpp:Execute()
    ├── AppManager::StartAbility()
    ├── AppSpawnClient::Spawn()
    └── AbilityRecord::ActiveAbility()
```

### stopAbility 调用链

```
JS 层:
  aafwk.stopAbility(want)
    │
    ▼
N-API 层:
  js_aafwk.cpp:JSAafwkStopAbility()
    ├── 参数解析（同 startAbility）
    │
    ▼
Native Framework:
  ability_manager.h:StopAbility()
    │
    ▼
IPC Client:
  abilityms_client.cpp:SendRequest(STOP_ABILITY)
    │
    ▼
AMS Service:
  ability_mgr_feature.cpp:StopAbilityInvoke()
    ├── GetCallingUid()
    ├── StopAbilityInner()
    │
    ▼
  ability_stop_task.cpp:Execute()
    └── AbilityRecord::StopAbility()
```

## 使用示例

### JavaScript 调用示例

```javascript
import aafwk from '@ohos.aafwk';

// 启动 Ability
function startAbility() {
    const want = {
        elementName: {
            deviceId: '',
            bundleName: 'com.example.app',
            abilityName: 'MainAbility'
        },
        want_param: {
            key: 'value'
        }
    };
    
    const result = aafwk.startAbility(want);
    if (result === 0) {
        console.log('Ability started successfully');
    } else {
        console.error('Failed to start Ability:', result);
    }
}

// 停止 Ability
function stopAbility() {
    const want = {
        elementName: {
            deviceId: '',
            bundleName: 'com.example.app',
            abilityName: 'ServiceAbility'
        }
    };
    
    const result = aafwk.stopAbility(want);
    console.log('Stop result:', result);
}
```

## 构建配置

**BUILD.gn**: `interfaces/kits/js/napi/BUILD.gn`

```gn
ohos_shared_library("aafwk") {
    include_dirs = [
        "//third_party/node/src",
        "${arkui_path}/napi/interfaces/kits",
    ]
    
    sources = [ "js_aafwk.cpp" ]
    
    deps = [
        "${ability_lite_path}/frameworks/abilitymgr_lite:abilitymanager",
        "${arkui_path}/napi/:ace_napi",
    ]
    
    external_deps = [
        "c_utils:utils",
        "hilog:libhilog",
        "ipc:ipc_core",
    ]
    
    relative_install_dir = "module"
    subsystem_name = "ability"
    part_name = "aafwk_native"
}
```

**输出产物**: `libaafwk.so`，安装到 `module/` 目录

## 已知限制

1. **仅支持同步调用**: 无 Promise/Callback 异步支持
2. **仅两个 API**: 仅暴露 `startAbility` 和 `stopAbility`
3. **错误处理简单**: 使用 `assert()` 进行参数校验
4. **返回值类型**: 返回 `int` 而非标准 N-API `napi_value`

## 相关链接

- [对外 Native API](03_Native_API.md)
- [内部 API](05_Inner_API.md)
- [架构说明](02_Architecture.md)
