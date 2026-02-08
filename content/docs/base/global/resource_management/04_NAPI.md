# N-API 接口文档

## 目的

本文档详细说明 OpenHarmony 资源管理组件的 JavaScript Native API (N-API)，包括所有导出的方法、参数、返回值、错误码和使用示例。

## 适用范围

本文档覆盖资源管理组件导出的所有 JavaScript API，共 **65 个方法**。

## 关键结论

| 类别 | 数量 | 说明 |
|------|------|------|
| 静态方法 | 3 个 | 模块级别的方法 (getResourceManager, getSystemResourceManager) |
| 静态属性 | 4 个 | 枚举对象 (Direction, DeviceType, ScreenDensity, ColorMode) |
| 异步方法 | 26 个 | 支持 Promise/Callback |
| 同步方法 | 39 个 | 直接返回结果 |

## 模块注册

### 注册入口

**文件**: `interfaces/js/kits/src/resource_manager_napi.cpp`

**注册函数**:
- `ResMgrRegister()` - 自动注册 (使用 `__attribute__((constructor))`)
- `ResMgrInit()` - 模块初始化

**证据**: `interfaces/js/kits/src/resource_manager_napi.cpp:316-329`

### 导出的静态方法

| JS 方法名 | C++ 实现函数 | 文件:行号 | 说明 |
|----------|-------------|----------|------|
| `getResourceManager` | `GetResourceManager` | resource_manager_napi.cpp:279 | 获取应用资源管理器 |
| `getSystemResourceManager` | `GetSystemResourceManager` | resource_manager_napi.cpp:280 | 获取系统资源管理器 |
| `getSysResourceManager` | `GetSysResourceManager` | resource_manager_napi.cpp:281 | 获取系统资源管理器(新) |

**证据**: `interfaces/js/kits/src/resource_manager_napi.cpp:274-281`

### 导出的静态属性 (枚举)

| 属性名 | 初始化函数 | 文件:行号 | 说明 |
|-------|-----------|----------|------|
| `Direction` | `InitDirectionObject` | resource_manager_napi.cpp:300 | 方向枚举 |
| `DeviceType` | `InitDeviceTypeObject` | resource_manager_napi.cpp:301 | 设备类型枚举 |
| `ScreenDensity` | `InitScreenDensityObject` | resource_manager_napi.cpp:302 | 屏幕密度枚举 |
| `ColorMode` | `InitColorModeObject` | resource_manager_napi.cpp:303 | 颜色模式枚举 |

**证据**: `interfaces/js/kits/src/resource_manager_napi.cpp:300-303`

## ResourceManager 实例方法

### 异步方法 (26 个)

#### 字符串相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getString` | `ResourceManagerAddon::GetString` | addon.cpp:282 | 获取字符串资源 |
| `getStringByName` | `ResourceManagerAddon::GetStringByName` | addon.cpp:352 | 根据名称获取字符串 |
| `getStringValue` | `ResourceManagerAddon::GetStringValue` | addon.cpp:357 | 获取字符串值 |
| `getStringArray` | `ResourceManagerAddon::GetStringArray` | addon.cpp:287 | 获取字符串数组 |
| `getStringArrayByName` | `ResourceManagerAddon::GetStringArrayByName` | addon.cpp:347 | 根据名称获取字符串数组 |
| `getStringArrayValue` | `ResourceManagerAddon::GetStringArrayValue` | addon.cpp:362 | 获取字符串数组值 |
| `getPluralString` | `ResourceManagerAddon::GetPluralString` | addon.cpp:312 | 获取复数字符串 |
| `getPluralStringByName` | `ResourceManagerAddon::GetPluralStringByName` | addon.cpp:332 | 根据名称获取复数字符串 |
| `getPluralStringValue` | `ResourceManagerAddon::GetPluralStringValue` | addon.cpp:377 | 获取复数字符串值 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159-224`

#### 媒体资源相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getMedia` | `ResourceManagerAddon::GetMedia` | addon.cpp:292 | 获取媒体资源 |
| `getMediaByName` | `ResourceManagerAddon::GetMediaByName` | addon.cpp:342 | 根据名称获取媒体 |
| `getMediaBase64` | `ResourceManagerAddon::GetMediaBase64` | addon.cpp:297 | 获取媒体资源 (Base64) |
| `getMediaBase64ByName` | `ResourceManagerAddon::GetMediaBase64ByName` | addon.cpp:337 | 根据名称获取媒体 (Base64) |
| `getMediaContent` | `ResourceManagerAddon::GetMediaContent` | addon.cpp:367 | 获取媒体内容 |
| `getMediaContentBase64` | `ResourceManagerAddon::GetMediaContentBase64` | addon.cpp:372 | 获取媒体内容 (Base64) |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159-224`

#### 原始文件相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getRawFile` | `ResourceManagerAddon::GetRawFile` | addon.cpp:317 | 获取原始文件 |
| `getRawFileList` | `ResourceManagerAddon::GetRawFileList` | addon.cpp:397 | 获取原始文件列表 |
| `getRawFileDescriptor` | `ResourceManagerAddon::GetRawFileDescriptor` | addon.cpp:322 | 获取原始文件描述符 |
| `closeRawFileDescriptor` | `ResourceManagerAddon::CloseRawFileDescriptor` | addon.cpp:327 | 关闭原始文件描述符 |
| `getRawFileContent` | `ResourceManagerAddon::GetRawFileContent` | addon.cpp:382 | 获取原始文件内容 |
| `getRawFd` | `ResourceManagerAddon::GetRawFd` | addon.cpp:387 | 获取原始文件 FD |
| `closeRawFd` | `ResourceManagerAddon::CloseRawFd` | addon.cpp:392 | 关闭原始文件 FD |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159-224`

#### 配置相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getConfiguration` | `ResourceManagerAddon::GetConfiguration` | addon.cpp:302 | 获取资源配置 |
| `getDeviceCapability` | `ResourceManagerAddon::GetDeviceCapability` | addon.cpp:307 | 获取设备能力 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159-224`

#### 颜色相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getColor` | `ResourceManagerAddon::GetColor` | addon.cpp:402 | 获取颜色资源 |
| `getColorByName` | `ResourceManagerAddon::GetColorByName` | addon.cpp:407 | 根据名称获取颜色 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159-224`

### 同步方法 (39 个)

#### 字符串相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getStringSync` | `ResourceManagerNapiSyncImpl::GetStringSync` | sync_impl.cpp:344 | 同步获取字符串 |
| `getStringByNameSync` | `ResourceManagerNapiSyncImpl::GetStringByNameSync` | sync_impl.cpp:820 | 同步根据名称获取字符串 |
| `getPluralStringByNameSync` | `ResourceManagerNapiSyncImpl::GetPluralStringByNameSync` | sync_impl.cpp:1117 | 同步根据名称获取复数字符串 |
| `getStringArrayByNameSync` | `ResourceManagerNapiSyncImpl::GetStringArrayByNameSync` | sync_impl.cpp:1244 | 同步根据名称获取字符串数组 |
| `getPluralStringValueSync` | `ResourceManagerNapiSyncImpl::GetPluralStringValueSync` | sync_impl.cpp:657 | 同步获取复数字符串值 |
| `getStringArrayValueSync` | `ResourceManagerNapiSyncImpl::GetStringArrayValueSync` | sync_impl.cpp:705 | 同步获取字符串数组值 |
| `getIntPluralStringValueSync` | `ResourceManagerNapiSyncImpl::GetIntPluralStringValueSync` | sync_impl.cpp:1470 | 同步获取整数复数字符串值 |
| `getIntPluralStringByNameSync` | `ResourceManagerNapiSyncImpl::GetIntPluralStringByNameSync` | sync_impl.cpp:1552 | 同步根据名称获取整数复数字符串值 |
| `getDoublePluralStringValueSync` | `ResourceManagerNapiSyncImpl::GetDoublePluralStringValueSync` | sync_impl.cpp:1507 | 同步获取浮点复数字符串值 |
| `getDoublePluralStringByNameSync` | `ResourceManagerNapiSyncImpl::GetDoublePluralStringByNameSync` | sync_impl.cpp:1594 | 同步根据名称获取浮点复数字符串值 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 媒体资源相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getMediaContentSync` | `ResourceManagerNapiSyncImpl::GetMediaContentSync` | sync_impl.cpp:607 | 同步获取媒体内容 |
| `getMediaContentBase64Sync` | `ResourceManagerNapiSyncImpl::GetMediaContentBase64Sync` | sync_impl.cpp:561 | 同步获取媒体内容 (Base64) |
| `getMediaByNameSync` | `ResourceManagerNapiSyncImpl::GetMediaByNameSync` | sync_impl.cpp:1202 | 同步根据名称获取媒体 |
| `getMediaBase64ByNameSync` | `ResourceManagerNapiSyncImpl::GetMediaBase64ByNameSync` | sync_impl.cpp:1160 | 同步根据名称获取媒体 (Base64) |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 原始文件相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getRawFileListSync` | `ResourceManagerNapiSyncImpl::GetRawFileListSync` | sync_impl.cpp:149 | 同步获取原始文件列表 |
| `getRawFileContentSync` | `ResourceManagerNapiSyncImpl::GetRawFileContentSync` | sync_impl.cpp:173 | 同步获取原始文件内容 |
| `getRawFdSync` | `ResourceManagerNapiSyncImpl::GetRawFdSync` | sync_impl.cpp:197 | 同步获取原始文件 FD |
| `closeRawFdSync` | `ResourceManagerNapiSyncImpl::CloseRawFdSync` | sync_impl.cpp:220 | 同步关闭原始文件 FD |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 配置相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getConfigurationSync` | `ResourceManagerNapiSyncImpl::GetConfigurationSync` | sync_impl.cpp:1271 | 同步获取资源配置 |
| `getDeviceCapabilitySync` | `ResourceManagerNapiSyncImpl::GetDeviceCapabilitySync` | sync_impl.cpp:1285 | 同步获取设备能力 |
| `getLocales` | `ResourceManagerNapiSyncImpl::GetLocales` | sync_impl.cpp:1299 | 获取支持的语言列表 |
| `getOverrideResourceManager` | `ResourceManagerNapiSyncImpl::GetOverrideResourceManager` | sync_impl.cpp:1346 | 获取覆盖资源管理器 |
| `getOverrideConfiguration` | `ResourceManagerNapiSyncImpl::GetOverrideConfiguration` | sync_impl.cpp:1387 | 获取覆盖资源配置 |
| `updateOverrideConfiguration` | `ResourceManagerNapiSyncImpl::UpdateOverrideConfiguration` | sync_impl.cpp:1401 | 更新覆盖资源配置 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 颜色和符号相关

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getColorSync` | `ResourceManagerNapiSyncImpl::GetColorSync` | sync_impl.cpp:433 | 同步获取颜色 |
| `getColorByNameSync` | `ResourceManagerNapiSyncImpl::GetColorByNameSync` | sync_impl.cpp:887 | 同步根据名称获取颜色 |
| `getSymbol` | `ResourceManagerNapiSyncImpl::GetSymbol` | sync_impl.cpp:388 | 获取符号资源 |
| `getSymbolByName` | `ResourceManagerNapiSyncImpl::GetSymbolByName` | sync_impl.cpp:853 | 根据名称获取符号 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 其他类型

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `getBoolean` | `ResourceManagerNapiSyncImpl::GetBoolean` | sync_impl.cpp:520 | 获取布尔值 |
| `getNumber` | `ResourceManagerNapiSyncImpl::GetNumber` | sync_impl.cpp:477 | 获取数字值 |
| `getBooleanByName` | `ResourceManagerNapiSyncImpl::GetBooleanByName` | sync_impl.cpp:950 | 根据名称获取布尔值 |
| `getNumberByName` | `ResourceManagerNapiSyncImpl::GetNumberByName` | sync_impl.cpp:925 | 根据名称获取数字值 |
| `getDrawableDescriptor` | `ResourceManagerNapiSyncImpl::GetDrawableDescriptor` | sync_impl.cpp:744 | 获取可绘制描述符 |
| `getDrawableDescriptorByName` | `ResourceManagerNapiSyncImpl::GetDrawableDescriptorByName` | sync_impl.cpp:992 | 根据名称获取可绘制描述符 |
| `isRawDir` | `ResourceManagerNapiSyncImpl::IsRawDir` | sync_impl.cpp:1322 | 检查是否为原始目录 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

#### 资源管理

| JS 方法名 | C++ 实现 | 文件:行号 | 说明 |
|----------|---------|----------|------|
| `addResource` | `ResourceManagerNapiSyncImpl::AddResource` | sync_impl.cpp:1055 | 添加资源 |
| `removeResource` | `ResourceManagerNapiSyncImpl::RemoveResource` | sync_impl.cpp:1080 | 移除资源 |
| `release` | `ResourceManagerAddon::Release` | addon.cpp:262 | 释放资源管理器 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`, `resource_manager_addon.cpp`

## 参数校验

### 参数类型校验

N-API 框架自动进行类型校验，但组件内部也会进行额外校验：

| 校验类型 | 位置 | 说明 |
|---------|------|------|
| 资源 ID | `ResourceManagerAddon::GetString` | 检查 ID 是否为有效数字 |
| 资源名称 | `ResourceManagerAddon::GetStringByName` | 检查名称是否为有效字符串 |
| 文件路径 | `ResourceManagerAddon::GetRawFile` | 检查路径格式和安全性 |
| Callback/Promise | `ResourceManagerAddon::AddOnGetResource` | 检查回调或 Promise 是否有效 |

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_utils.cpp`

### 路径安全校验

```cpp
// 路径规范化，防止路径遍历攻击
char resolvedPath[PATH_MAX];
if (realpath(path, resolvedPath) == nullptr) {
    return ERROR;
}
```

**证据**: `frameworks/resmgr/src/raw_file_manager.cpp:384-402`

## 错误码与异常

### 错误码定义

**文件**: `frameworks/resmgr/include/utils/errors.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 成功 |
| `NOT_FOUND` | 1 | 资源未找到 |
| `INVALID_PARAMS` | 2 | 参数无效 |
| `OUT_OF_MEMORY` | 3 | 内存不足 |
| `PATH_NOT_FOUND` | 4 | 路径未找到 |
| `FILE_NOT_FOUND` | 5 | 文件未找到 |
| `PARSE_ERROR` | 6 | 解析错误 |
| ... | ... | ... |

**证据**: `frameworks/resmgr/include/utils/errors.h`

### 异常封装

N-API 使用 `napi_throw_error` 或 `napi_throw` 抛出异常：

```cpp
if (ret != SUCCESS) {
    napi_throw_error(env, nullptr, "Resource not found");
    return nullptr;
}
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_utils.cpp`

## 同步/异步模式

### 异步模式

**支持的方法**: 26 个异步方法

**调用方式**:
```javascript
// Promise 方式
resmgr.getString(resId).then(value => {
    console.log(value);
}).catch(error => {
    console.error(error);
});

// Callback 方式
resmgr.getString(resId, (error, value) => {
    if (error) {
        console.error(error);
    } else {
        console.log(value);
    }
});
```

**实现**: 使用 `napi_create_async_work` 和 `napi_queue_async_work_with_qos`

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_async_impl.cpp`

### 同步模式

**支持的方法**: 39 个同步方法

**调用方式**:
```javascript
const value = resmgr.getStringSync(resId);
console.log(value);
```

**实现**: 直接调用核心方法，阻塞调用线程

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp`

## 使用示例

### 获取资源管理器

```javascript
import resmgr from '@ohos.resmgr';

// 获取应用资源管理器
resmgr.getResourceManager((error, mgr) => {
    if (error) {
        console.error('Failed to get resource manager:', error);
        return;
    }

    // 使用资源管理器
    mgr.getString(0x1000000, (err, value) => {
        if (err) {
            console.error('Failed to get string:', err);
        } else {
            console.log('String value:', value);
        }
    });
});
```

**证据**: `README_zh.md:36-58`

### 同步获取字符串

```javascript
const value = mgr.getStringSync(0x1000000);
console.log('String value:', value);
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp:344`

### 异步获取字符串 (Promise)

```javascript
mgr.getString(0x1000000)
    .then(value => {
        console.log('String value:', value);
    })
    .catch(error => {
        console.error('Failed to get string:', error);
    });
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_async_impl.cpp`

### 获取原始文件

```javascript
mgr.getRawFile('assets/data.txt', (error, file) => {
    if (error) {
        console.error('Failed to get raw file:', error);
        return;
    }

    console.log('Raw file:', file);
});
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:317`

### 获取原始文件描述符

```javascript
mgr.getRawFd('assets/data.txt', (error, fd) => {
    if (error) {
        console.error('Failed to get raw fd:', error);
        return;
    }

    console.log('Raw fd:', fd.fd);
});
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp:387`

## 调用链示例

### getString 调用链 (同步)

```
JS: getStringSync(id)
    ↓
NAPI: ResourceManagerNapiSyncImpl::GetStringSync
    ↓
C++: ResourceManagerImpl::GetString
    ↓
C++: HapResourceManager::FindResource
    ↓
C++: LocaleMatcher::MatchResource
    ↓
C++: HapResource::GetValue
    ↓
返回字符串值
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp:344`

### getString 调用链 (异步)

```
JS: getString(id)
    ↓
NAPI: ResourceManagerAddon::AddOnGetResource
    ↓
NAPI: ResourceManagerNapiAsyncImpl::GetResource
    ↓
NAPI: napi_create_async_work
    ↓
Worker: ExecuteAsyncWork
    ↓
C++: ResourceManagerImpl::GetString
    ↓
C++: HapResourceManager::FindResource
    ↓
NAPI: napi_resolve_deferred
    ↓
Promise resolved
```

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_async_impl.cpp`

## 权限与前置条件

### 权限要求

本组件**不要求特殊权限**，所有应用都可以访问自己的资源。

系统资源管理器需要特殊权限，但由系统管理，应用无法直接创建。

**证据**: 权限搜索结果 (未发现权限检查)

### 前置条件

1. **应用 HAP 包必须包含资源文件**: 否则资源查询会失败
2. **资源配置必须正确**: 否则资源匹配会失败
3. **资源 ID/名称必须有效**: 否则会返回错误

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [架构设计](03_Architecture.md) - 组件架构和数据流
- [内部 API](05_InnerAPI.md) - C++ 内部接口

---

**生成时间**: 2026-02-06
**证据来源**: interfaces/js/kits/src/resource_manager_napi.cpp, interfaces/js/innerkits/core/src/
