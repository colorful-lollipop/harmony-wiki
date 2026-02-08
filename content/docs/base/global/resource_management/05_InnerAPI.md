# 内部 API 文档

## 目的

本文档说明 OpenHarmony 资源管理组件的内部 API，包括 C++ 内部子系统接口、Native API 和各语言绑定的内部实现。

## 适用范围

本文档覆盖所有内部 API 接口，供子系统间调用和内部实现参考。

## 关键结论

| API 类型 | 文件位置 | 稳定性 | 使用场景 |
|----------|----------|----------|----------|
| Inner API | `interfaces/inner_api/` | 相对稳定 | 子系统间调用 |
| Native API | `interfaces/native/` | 稳定 | NDK 原生开发 |
| NAPI 内部 | `interfaces/js/innerkits/core/` | 内部实现 | JS NAPI 绑定 |
| ANI 内部 | `interfaces/ets/ani/` | 内部实现 | ETS ANI 绑定 |
| FFI 内部 | `interfaces/cj/src/` | 内部实现 | CJ FFI 绑定 |

## Inner API

### 头文件

**路径**: `interfaces/inner_api/include/`

**导出头文件**:
- `res_common.h` - 通用定义和常量
- `res_config.h` - 资源配置接口
- `resource_manager.h` - 资源管理器接口
- `rstate.h` - 资源状态定义

**证据**: `bundle.json:89-100`

### res_common.h

**路径**: `interfaces/inner_api/include/res_common.h`

**内容**:
- 通用宏定义
- 常量定义
- 错误码

**证据**: `interfaces/inner_api/include/res_common.h`

### res_config.h

**路径**: `interfaces/inner_api/include/res_config.h`

**内容**:
- `ResConfig` 类定义
- 资源配置接口

**关键方法**:
```cpp
class ResConfig {
public:
    void SetLocale(const std::string &locale);
    void SetDirection(Direction direction);
    void SetDeviceType(DeviceType deviceType);
    void SetScreenDensity(ScreenDensity screenDensity);
    void SetColorMode(ColorMode colorMode);
    // ...
};
```

**证据**: `interfaces/inner_api/include/res_config.h`

### resource_manager.h

**路径**: `interfaces/inner_api/include/resource_manager.h`

**内容**:
- `ResourceManager` 类定义
- 资源管理器接口

**关键方法**:
```cpp
class ResourceManager {
public:
    static ResourceManager *GetResourceManager();
    static ResourceManager *GetSystemResourceManager();

    bool GetStringById(uint32_t id, std::string &outValue);
    bool GetStringByName(const std::string &name, std::string &outValue);
    // ...
};
```

**证据**: `interfaces/inner_api/include/resource_manager.h`

### rstate.h

**路径**: `interfaces/inner_api/include/rstate.h`

**内容**:
- 资源状态枚举
- 资源加载状态

**证据**: `interfaces/inner_api/include/rstate.h`

## Native API

### 头文件

**路径**: `interfaces/native/resource/include/`

**导出头文件**:
- `ohresmgr.h` - Native 资源管理器接口
- `raw_file.h` - 原始文件接口
- `raw_dir.h` - 原始目录接口
- `raw_file_manager.h` - 原始文件管理器接口
- `resmgr_common.h` - 通用定义

**证据**: `bundle.json:158-164`

### ohresmgr.h

**路径**: `interfaces/native/resource/include/ohresmgr.h`

**内容**:
- `OhosResourceManager` 结构体
- Native 资源管理器接口

**关键接口**:
```c
OhosResourceManager* OH_ResourceManager_GetResourceManager(const char* bundleName);
bool OH_ResourceManager_GetString(OhosResourceManager* mgr, uint32_t id, char* outValue, size_t* len);
// ...
```

**证据**: `interfaces/native/resource/include/ohresmgr.h`

### raw_file.h

**路径**: `interfaces/native/resource/include/raw_file.h`

**内容**:
- `OhosRawFile` 结构体
- 原始文件接口

**关键接口**:
```c
OhosRawFile* OH_ResourceManager_GetRawFile(OhosResourceManager* mgr, const char* path);
int OH_RawFile_Close(OhosRawFile* rawFile);
// ...
```

**证据**: `interfaces/native/resource/include/raw_file.h`

## NAPI 内部实现

### ResourceManagerAddon

**路径**: `interfaces/js/innerkits/core/include/resource_manager_addon.h`

**职责**: JS 对象包装类，将 C++ ResourceManager 暴露给 JS

**关键方法**:
```cpp
class ResourceManagerAddon {
public:
    static napi_value AddOnGetResource(napi_env env, napi_callback_info info);
    static napi_value GetString(napi_env env, napi_callback_info info);
    // ...

private:
    ResourceManager *resMgr_;  // C++ 资源管理器
};
```

**证据**: `interfaces/js/innerkits/core/include/resource_manager_addon.h`

### ResourceManagerNapiSyncImpl

**路径**: `interfaces/js/innerkits/core/include/resource_manager_napi_sync_impl.h`

**职责**: 同步 NAPI 方法实现

**关键方法**:
```cpp
class ResourceManagerNapiSyncImpl {
public:
    static napi_value GetStringSync(napi_env env, napi_callback_info info);
    static napi_value GetColorSync(napi_env env, napi_callback_info info);
    // ...
};
```

**证据**: `interfaces/js/innerkits/core/include/resource_manager_napi_sync_impl.h`

### ResourceManagerNapiAsyncImpl

**路径**: `interfaces/js/innerkits/core/include/resource_manager_napi_async_impl.h`

**职责**: 异步 NAPI 方法实现

**关键方法**:
```cpp
class ResourceManagerNapiAsyncImpl {
public:
    static napi_value GetResource(napi_env env, napi_callback_info info);
    static void ExecuteAsyncWork(napi_env env, void *data);

private:
    static AsyncFuncMatch asyncFuncMatch[];
};
```

**证据**: `interfaces/js/innerkits/core/include/resource_manager_napi_async_impl.h`

## ANI 内部实现

### resmgr_ani.h

**路径**: `interfaces/ets/ani/resourceManager/include/resmgr_ani.h`

**职责**: ANI 桥接实现

**关键方法**:
```cpp
extern "C" {
    void* ANI_Resource_Manager_GetResourceManager(ANI_VM *vm, ANI_ENV *env, ANI_CallbackInfo *info);
    void* ANI_Resource_Manager_GetString(ANI_VM *vm, ANI_ENV *env, ANI_CallbackInfo *info);
    // ...
}
```

**证据**: `interfaces/ets/ani/resourceManager/include/resmgr_ani.h`

## FFI 内部实现

### resource_manager_ffi.h

**路径**: `interfaces/cj/src/resource_manager_ffi.h`

**职责**: Cangjie FFI 接口

**关键方法**:
```cpp
extern "C" {
    CJ_Resource_Manager* CJ_GetResourceManager(const char* bundleName);
    bool CJ_GetString(CJ_Resource_Manager* mgr, uint32_t id, char* outValue, size_t* len);
    // ...
}
```

**证据**: `interfaces/cj/src/resource_manager_ffi.h`

## 模块依赖关系

### 内部依赖

```
Native API (interfaces/native/)
    ↓ 依赖
Inner API (interfaces/inner_api/)
    ↓ 依赖
核心框架 (frameworks/resmgr/)
```

### 外部依赖

| 模块 | 依赖的接口 | 用途 |
|------|-----------|------|
| NAPI 内部 | Inner API | JS NAPI 实现 |
| ANI 内部 | Inner API | ETS ANI 实现 |
| FFI 内部 | Inner API | CJ FFI 实现 |
| Native API | Inner API | C Native 接口 |

## 接口稳定性

### 稳定接口

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| Inner API | 相对稳定 | 子系统间调用 |
| Native API | 稳定 | NDK 接口，向后兼容 |
| NAPI 接口 | 稳定 | 公开的 JS API |

### 不稳定接口

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| NAPI 内部实现 | 不稳定 | 实现细节，可能变更 |
| ANI 内部实现 | 不稳定 | 实现细节，可能变更 |
| FFI 内部实现 | 不稳定 | 实现细节，可能变更 |

**判断依据**:
- 位于 `include/` 目录的接口相对稳定
- 位于 `src/` 目录的实现不稳定
- 没有明确声明的内部接口不稳定

**证据**: 目录结构和头文件位置

## 可替换点

### 资源解析器

**位置**: `HapParser` 及其子类

**可替换性**: 可扩展新的 HAP 版本解析器

**证据**: `frameworks/resmgr/include/hap_parser.h`

### 语言匹配器

**位置**: `LocaleMatcher`

**可替换性**: 可替换为其他匹配算法

**证据**: `frameworks/resmgr/include/locale_matcher.h`

### 资源后端

**位置**: `HapResourceManager`

**可替换性**: 可替换为其他资源存储格式

**证据**: `frameworks/resmgr/include/hap_resource_manager.h`

## 使用示例

### C++ 使用 Inner API

```cpp
#include "resource_manager.h"

using namespace OHOS::Global::Resource;

// 获取资源管理器
ResourceManager *resMgr = ResourceManager::GetResourceManager();

// 获取字符串
std::string value;
if (resMgr->GetStringById(0x1000000, value)) {
    std::cout << "String: " << value << std::endl;
}
```

**证据**: `interfaces/inner_api/include/resource_manager.h`

### C 使用 Native API

```c
#include "ohresmgr.h"

// 获取资源管理器
OhosResourceManager *mgr = OH_ResourceManager_GetResourceManager("com.example.app");

// 获取字符串
char buffer[256];
size_t len = sizeof(buffer);
if (OH_ResourceManager_GetString(mgr, 0x1000000, buffer, &len)) {
    printf("String: %s\n", buffer);
}
```

**证据**: `interfaces/native/resource/include/ohresmgr.h`

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [目录结构](02_DirectoryStructure.md) - 代码组织和模块职责
- [N-API 接口](04_NAPI.md) - JavaScript API 详细文档

---

**生成时间**: 2026-02-06
**证据来源**: interfaces/inner_api/, interfaces/native/, bundle.json
