# JS API 参考

---

## 目的

本文档提供分布式包管理服务 (DBMS) 对外 JavaScript API 的完整参考，包括 API 清单、参数定义、调用方式和错误处理。

---

## 适用范围

- ✅ 所有 JS API 方法清单
- ✅ 参数类型和校验规则
- ✅ 同步/异步调用模式
- ✅ 调用链路说明
- ❌ 详细的实现逻辑（请参考附录调用链）

---

## API 命名空间

DBMS 提供两个 JS API 命名空间：

| 命名空间 | 模块名 | 推荐度 | 特点 |
|----------|--------|--------|------|
| distributedBundle | libdistributedbundle.z.so | 旧版 | 单独方法（getRemoteAbilityInfo, getRemoteAbilityInfos）|
| bundle.distributedBundleManager | libdistributedbundlemanager.z.so | 新版 | 统一方法（getRemoteAbilityInfo 支持单/批量）|

**证据**:
- 旧版: `interfaces/kits/js/distributedBundle/native_module.cpp:54`
- 新版: `interfaces/kits/js/distributebundlemgr/native_module.cpp:53`

---

## API 清单

### distributedBundle 命名空间（旧版）

| 方法名 | 参数 | 返回值 | 模式 | 文件位置 |
|--------|------|--------|------|----------|
| getRemoteAbilityInfo | ElementName, locale?, callback? | Promise\<RemoteAbilityInfo> \| callback | 异步 | native_module.cpp:37 |
| getRemoteAbilityInfos | ElementName[], locale?, callback? | Promise\<RemoteAbilityInfo[]> \| callback | 异步 | native_module.cpp:38 |

**证据**: `interfaces/kits/js/distributedBundle/native_module.cpp:36-39`

### bundle.distributedBundleManager 命名空间（新版）

| 方法名 | 参数 | 返回值 | 模式 | 文件位置 |
|--------|------|--------|------|----------|
| getRemoteAbilityInfo | ElementName \| ElementName[], locale?, callback? | Promise\<RemoteAbilityInfo \| RemoteAbilityInfo[]> \| callback | 异步 | native_module.cpp:37 |

**证据**: `interfaces/kits/js/distributebundlemgr/native_module.cpp:36-38`

---

## ElementName 参数

### 结构定义

```javascript
{
    deviceId: string,      // 设备 ID（必需）
    bundleName: string,    // 应用包名（必需）
    moduleName: string,     // 模块名（可选）
    abilityName: string     // 能力名称（必需）
}
```

### 参数校验规则

#### deviceId
- **类型**: string
- **必需**: ✅
- **校验**: 非空字符串
- **示例**: "networkId" 或本地设备 ID

#### bundleName
- **类型**: string
- **必需**: ✅
- **校验**: 非空字符串
- **示例**: "com.example.application"

#### abilityName
- **类型**: string
- **必需**: ✅
- **校验**: 非空字符串
- **示例**: "EntryAbility"

#### moduleName
- **类型**: string
- **必需**: ❌（可选）
- **校验**: 可为空
- **示例**: "entry" 或 "feature"

**校验位置**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255`

---

## locale 参数

### 参数定义

- **类型**: string
- **必需**: ❌（可选）
- **格式**: 语言地区代码（如 "zh-CN", "en-US"）
- **用途**: 指定返回的 label 和 icon 应使用的语言

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:347-348`

---

## RemoteAbilityInfo 返回值

### 结构定义

```javascript
{
    elementName: ElementName,    // Ability 组件标识
    label: string,            // 本地化标签
    icon: string              // 图标（Base64 编码的 data URI）
}
```

### 字段说明

#### elementName
- **类型**: ElementName
- **内容**: 传入的 ElementName 参数（ deviceId, bundleName, moduleName, abilityName）

#### label
- **类型**: string
- **内容**: Ability 的本地化显示名称
- **编码**: UTF-8

#### icon
- **类型**: string
- **内容**: 图标数据，格式为 `data:image/png;base64,...` 或 `data:image/jpeg;base64,...`
- **编码**: Base64

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:152-168`

---

## 异步调用模式

### Callback 模式

**语法**:
```javascript
// 单个查询
distributedBundle.getRemoteAbilityInfo(elementName, locale, (err, result) => {
    if (err) {
        console.error(`Error: ${err.code}, ${err.message}`);
    } else {
        console.log('Result:', result);
    }
});

// 批量查询
distributedBundle.getRemoteAbilityInfos(elementNames, locale, (err, results) => {
    if (err) {
        console.error(`Error: ${err.code}, ${err.message}`);
    } else {
        console.log('Results:', results);
    }
});
```

### Promise 模式

**语法**:
```javascript
// 单个查询
try {
    const result = await distributedBundle.getRemoteAbilityInfo(elementName, locale);
    console.log('Result:', result);
} catch (err) {
    console.error(`Error: ${err.code}, ${err.message}`);
}

// 批量查询
try {
    const results = await distributedBundle.getRemoteAbilityInfos(elementNames, locale);
    console.log('Results:', results);
} catch (err) {
    console.error(`Error: ${err.code}, ${err.message}`);
}
```

**实现方式**:
- 使用 `napi_create_promise()` 创建 Promise
- 使用 `napi_create_async_work()` 创建异步工作
- 使用 `napi_resolve_deferred()` 或 `napi_reject_deferred()` 完成或拒绝

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:358-404`

---

## 错误码

### JS 错误对象结构

```javascript
{
    code: number,      // 错误码
    message: string,   // 错误消息
    name: string       // 错误名称（可选）
}
```

### 主要错误码

| 错误码 | 说明 | 常量 | 场景 |
|--------|------|--------|------|
| 17700001 | Bundle name not found | ERR_BUNDLE_MANAGER_BUNDLE_NOT_FOUND | 指定的包名不存在 |
| 17700003 | Ability name not found | ERR_BUNDLE_MANAGER_ABILITY_NOT_FOUND | 指定的 Ability 不存在 |
| 17700007 | Device ID not found | ERR_BUNDLE_MANAGER_DEVICE_ID_NOT_EXIST | 设备 ID 无效或不存在 |
| 17700027 | Distributed service not running | ERR_BUNDLE_MANAGER_SERVICE_INTERNAL_ERROR | DBMS 服务未运行 |
| 401 | Parameter error | ERROR_PARAM_CHECK_ERROR | 参数校验失败 |
| 17700002 | Permission denied | ERR_BUNDLE_MANAGER_PERMISSION_DENIED | 权限不足 |

**证据**:
- 错误码定义: `README_zh.md:35-39`
- JS 错误构造: `interfaces/kits/js/distributedBundle/distributed_bundle.cpp:212-214`

---

## 参数校验

### ElementName 校验

**校验位置**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255`

```cpp
static bool ParseElementName(napi_env env, OHOS::AppExecFwk::ElementName &elementName, napi_value args)
{
    // 1. 类型检查
    napi_status status = napi_typeof(env, args, &valueType);
    if ((status != napi_ok)|| (valueType != napi_object)) {
        APP_LOGE("args not object type");
        return false;
    }

    // 2. 解析 deviceId（必需）
    if (!CommonFunc::ParseStringPropertyFromObject(env, args, "deviceId", true, deviceId)) {
        APP_LOGE("begin to parse ElementName deviceId failed");
        return false;
    }

    // 3. 解析 bundleName（必需）
    if (!CommonFunc::ParseStringPropertyFromObject(env, args, "bundleName", true, bundleName)) {
        APP_LOGE("begin to parse ElementName bundleName failed");
        return false;
    }

    // 4. 解析 abilityName（必需）
    if (!CommonFunc::ParseStringPropertyFromObject(env, args, "abilityName", true, abilityName)) {
        APP_LOGE("begin to parse ElementName abilityName failed");
        return false;
    }

    // 5. 解析 moduleName（可选）
    if (!CommonFunc::ParseStringPropertyFromObject(env, args, "moduleName", false, moduleName)) {
        APP_LOGE("begin to parse ElementName moduleName failed");
        return false;
    }

    return true;
}
```

### 数组长度校验

**批量查询限制**: 最大 10 个元素

```cpp
if (asyncCallbackInfo->elementNames.size() > GET_REMOTE_ABILITY_INFO_MAX_SIZE) {
    BusinessError::ThrowError(env, ERROR_PARAM_CHECK_ERROR,
        "BusinessError 401: The number of ElementNames is greater than 10");
    return nullptr;
}
```

**证据**: `interfaces/kits/js/distributedBundle/distributed_bundle.cpp:262-266`

---

## 调用链路

### 单个查询流程

```
JavaScript 层
    │
    ▼ getRemoteAbilityInfo(elementName, locale)
    │
    ├──────────────────────────────────────────────────────────────────┐
    │  distributedBundle.cpp: GetRemoteAbilityInfo()          │
    │  1. ParseElementName() - 解析参数               │
    │  2. napi_create_async_work() - 创建异步工作     │
    │     ┌─────────────────────────────────────────────┐       │
    │     │ 工作线程（Execute）                        │       │
    │     │ InnerGetRemoteAbilityInfo()              │       │
    │     │   └── GetDistributedBundleMgr()         │       │
    │     │         └── IDistributedBms::           │       │
    │     │             GetRemoteAbilityInfo()           │       │
    │     └─────────────────────────────────────────────┘       │
    │  3. 主线程（Complete）                            │
    │     - ConvertRemoteAbilityInfo() - 构造 JS 对象  │
    │     - napi_resolve_deferred() - 解析 Promise   │
    │     └── napi_call_function() - 调用回调     │
    └──────────────────────────────────────────────────────────────────┘
```

### 批量查询流程

```
JavaScript 层
    │
    ▼ getRemoteAbilityInfos(elementNames, locale)
    │
    ├──────────────────────────────────────────────────────────────────┐
    │  distributed_bundle_mgr.cpp: GetRemoteAbilityInfos()       │
    │  1. ParseElementNames() - 解析数组参数           │
    │  2. napi_create_async_work() - 创建异步工作     │
    │     ┌─────────────────────────────────────────────┐       │
    │     │ 工作线程（Execute）                        │       │
    │     │ InnerGetRemoteAbilityInfos()             │       │
    │     │   └── GetDistributedBundleMgr()         │       │
    │     │         └── IDistributedBms::           │       │
    │     │             GetRemoteAbilityInfos()          │       │
    │     └─────────────────────────────────────────────┘       │
    │  3. 主线程（Complete）                            │
    │     - ConvertRemoteAbilityInfos() - 构造 JS 数组  │
    │     - napi_resolve_deferred() - 解析 Promise   │
    │     └── napi_call_function() - 调用回调     │
    └──────────────────────────────────────────────────────────────────┘
```

---

## 使用示例

### 单个查询（旧版 API）

```javascript
import distributedBundle from '@ohos.distributedBundle.distributedBundle';

try {
    const result = await distributedBundle.getRemoteAbilityInfo({
        deviceId: 'networkId123',
        bundleName: 'com.example.app',
        abilityName: 'MainAbility',
        moduleName: 'entry'
    }, 'zh-CN');

    console.log('Label:', result.label);
    console.log('Icon length:', result.icon.length);
} catch (err) {
    console.error('Error:', err.code, err.message);
}
```

### 批量查询（新版 API）

```javascript
import distributedBundle from '@ohos.bundle.distributedBundleManager';

try {
    const results = await distributedBundle.getRemoteAbilityInfo([
        {
            deviceId: 'networkId123',
            bundleName: 'com.example.app',
            abilityName: 'MainAbility'
        },
        {
            deviceId: 'networkId456',
            bundleName: 'com.example.another',
            abilityName: 'SubAbility'
        }
    ], 'zh-CN');

    for (const info of results) {
        console.log(`${info.elementName.bundleName}: ${info.label}`);
    }
} catch (err) {
    console.error('Error:', err.code, err.message);
}
```

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| 旧版模块注册 | interfaces/kits/js/distributedBundle/native_module.cpp:49-64 |
| 新版模块注册 | interfaces/kits/js/distributebundlemgr/native_module.cpp:49-56 |
| 参数校验逻辑 | interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255 |
| 批量查询限制 | interfaces/kits/js/distributedBundle/distributed_bundle.cpp:262-266 |
| 异步工作模式 | interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:358-404 |
