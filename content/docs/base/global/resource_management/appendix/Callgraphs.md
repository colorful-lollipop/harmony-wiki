# 关键调用链

## 目的

本文档汇总 OpenHarmony 资源管理组件的关键调用链，从入口到核心逻辑的执行流程。

## 适用范围

本文档覆盖主要 API 的调用链，包括 JS NAPI、Native API 和内部实现。

## 调用链索引

| 序号 | 调用链名称 | 入口 | 核心逻辑 | 文件位置 |
|------|------------|------|----------|----------|
| 1 | getString (同步) | JS NAPI | 资源查找和匹配 | `04_NAPI.md` |
| 2 | getString (异步) | JS NAPI | 异步工作队列 | `04_NAPI.md` |
| 3 | 资源管理器初始化 | JS NAPI | HAP 加载 | `03_Architecture.md` |
| 4 | 原始文件访问 | JS NAPI | 文件路径检查 | `04_NAPI.md` |
| 5 | 系统资源管理器初始化 | JS NAPI | 系统资源加载 | `03_Architecture.md` |
| 6 | Native API 调用 | C API | Native 接口 | `05_InnerAPI.md` |
| 7 | 资源添加/移除 | JS NAPI | 资源覆盖 | `04_NAPI.md` |

## 详细调用链

### 1. getString (同步调用)

```
[JS 应用层]
resmgr.getStringSync(resId)
    ↓
[NAPI 层]
ResourceManagerNapiSyncImpl::GetStringSync(env, info)
    [interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp:344]
    ↓
    参数校验
    ↓
    ResourceManagerAddon::Unwrap(env, this)
    [interfaces/js/innerkits/core/src/resource_manager_addon.cpp:81]
    ↓
[核心框架层]
ResourceManagerImpl::GetString(resId, value)
    [frameworks/resmgr/src/resource_manager_impl.cpp]
    ↓
    HapResourceManager::FindResource(resId, config)
    [frameworks/resmgr/src/hap_resource_manager.cpp]
    ↓
    LocaleMatcher::MatchResource(resources, config)
    [frameworks/resmgr/src/locale_matcher.cpp]
    ↓
    HapResource::GetValue()
    [frameworks/resmgr/src/hap_resource.cpp]
    ↓
[NAPI 层]
ResourceManagerNapiSyncImpl::CreateString(env, value)
    ↓
[JS 应用层]
返回字符串值
```

**关键数据结构**:
- `ResourceManager` - 资源管理器
- `HapResourceManager` - HAP 资源管理器
- `LocaleMatcher` - 语言匹配器
- `HapResource` - HAP 资源
- `ResConfig` - 资源配置

**证据**: `04_NAPI.md`, `03_Architecture.md`

---

### 2. getString (异步调用)

```
[JS 应用层]
resmgr.getString(resId, callback)
    ↓
[NAPI 层]
ResourceManagerAddon::AddOnGetResource(env, info, "getString")
    [interfaces/js/innerkits/core/src/resource_manager_addon.cpp:159]
    ↓
    检测是 Callback 还是 Promise
    ↓
    ResourceManagerNapiAsyncImpl::GetResource(env, info)
    [interfaces/js/innerkits/core/src/resource_manager_napi_async_impl.cpp]
    ↓
    napi_create_promise() → 创建 Promise
    ↓
    napi_create_async_work() → 创建异步工作
    ↓
    napi_queue_async_work_with_qos() → 队列异步工作
    ↓
    返回 Promise
[JS 应用层]
继续执行，等待 Promise resolve
    ↓
[工作线程]
ExecuteAsyncWork(data)
    ↓
[核心框架层]
ResourceManagerImpl::GetString(resId, value)
    ↓
    HapResourceManager::FindResource(resId, config)
    ↓
    LocaleMatcher::MatchResource(resources, config)
    ↓
    HapResource::GetValue()
    ↓
[回调线程]
CompleteAsyncWork(env, status, data)
    ↓
    napi_resolve_deferred() → resolve Promise
    ↓
    或 napi_reject_deferred() → reject Promise
[JS 应用层]
Promise resolved/rejected，执行 then/catch
```

**关键函数**:
- `napi_create_promise` - 创建 Promise
- `napi_create_async_work` - 创建异步工作
- `napi_queue_async_work_with_qos` - 队列异步工作
- `napi_resolve_deferred` - 解析 Promise
- `napi_reject_deferred` - 拒绝 Promise

**证据**: `04_NAPI.md`, `interfaces/js/innerkits/core/src/resource_manager_napi_async_impl.cpp`

---

### 3. 资源管理器初始化

```
[JS 应用层]
resmgr.getResourceManager(callback)
    ↓
[NAPI 层]
GetResourceManager(env, info)
    [interfaces/js/kits/src/resource_manager_napi.cpp:279]
    ↓
    获取 bundleName
    ↓
    ResourceManager::GetResourceManager(bundleName)
    ↓
[核心框架层]
ResourceManagerImpl::Init(isSystem)
    [frameworks/resmgr/src/resource_manager_impl.cpp:81-97]
    ↓
    创建 HapManager
    ↓
    加载 HAP 包
    ↓
        HapManager::LoadHap(hapPath)
        [frameworks/resmgr/src/hap_manager.cpp]
        ↓
        HapParser::ParseHap(hapPath)
        [frameworks/resmgr/src/hap_parser.cpp]
        ↓
        解析索引文件
        ↓
        解析资源配置
        ↓
        创建 HapResource
    ↓
    创建 LocaleMatcher
    ↓
    初始化 ResConfig
    ↓
[NAPI 层]
ResourceManagerAddon::New(env, resMgr)
    ↓
    napi_wrap() - 包装 C++ 对象
    ↓
[JS 应用层]
返回 ResourceManager 对象
```

**关键步骤**:
1. 加载 HAP 包
2. 解析 HAP 索引
3. 初始化资源管理器
4. 包装 C++ 对象为 JS 对象

**证据**: `03_Architecture.md`, `frameworks/resmgr/src/resource_manager_impl.cpp`

---

### 4. 原始文件访问

```
[JS 应用层]
mgr.getRawFile(path, callback)
    ↓
[NAPI 层]
ResourceManagerAddon::GetRawFile(env, info)
    [interfaces/js/innerkits/core/src/resource_manager_addon.cpp:317]
    ↓
    获取路径参数
    ↓
[核心框架层]
RawFileManager::GetRawFile(path)
    [frameworks/resmgr/src/raw_file_manager.cpp]
    ↓
    路径校验和规范化
    ↓
    realpath() - 解析真实路径
    [frameworks/resmgr/src/raw_file_manager.cpp:384-402]
    ↓
    检查路径是否在允许范围内
    ↓
    检查路径是否为系统路径
    ↓
    IsSystemPath() - 检查系统路径
    [frameworks/resmgr/src/resource_manager_impl.cpp:1337]
    ↓
    打开文件
    ↓
[NAPI 层]
CreateRawFileObject(env, rawFile)
    ↓
    napi_wrap() - 包装 C++ 对象
    ↓
[JS 应用层]
返回 RawFile 对象
```

**安全检查**:
1. 路径规范化 (`realpath`)
2. 路径范围检查
3. 系统路径检查

**证据**: `08_Security.md`, `frameworks/resmgr/src/raw_file_manager.cpp`

---

### 5. 系统资源管理器初始化

```
[JS 应用层 / SystemAbility]
resmgr.getSystemResourceManager(callback)
    ↓
[NAPI 层]
GetSystemResourceManager(env, info)
    [interfaces/js/kits/src/resource_manager_napi.cpp:280]
    ↓
[核心框架层]
SystemResourceManager::GetSystemResourceManager()
    [frameworks/resmgr/src/system_resource_manager.cpp:65-77]
    ↓
    检查沙箱路径是否存在
    ↓
    GetSystemResourceManager() - 沙箱版本
    ↓
        检查 /data/storage/el1/bundle/ohos.global.systemres/...
        ↓
        加载系统 HAP 包
    ↓
    或 GetSystemResourceManagerNoSandBox() - 非沙箱版本
    ↓
        检查 /system/app/ohos.global.systemres/SystemResources.hap
        ↓
        加载系统 HAP 包
    ↓
    ResourceManagerImpl::Init(true) - 初始化系统资源管理器
    ↓
    设置 isSystemResMgr_ = true
    ↓
[NAPI 层]
ResourceManagerAddon::New(env, resMgr)
    ↓
[JS 应用层]
返回系统 ResourceManager 对象
```

**关键路径**:
- 沙箱路径: `/data/storage/el1/bundle/ohos.global.systemres/`
- 非沙箱路径: `/system/app/ohos.global.systemres/SystemResources.hap`

**证据**: `03_Architecture.md`, `frameworks/resmgr/src/system_resource_manager.cpp`

---

### 6. Native API 调用

```
[C 应用层]
OH_ResourceManager_GetResourceManager(bundleName)
    [interfaces/native/resource/include/ohresmgr.h]
    ↓
[Native API 层]
ResourceManager *rm = ResourceManager::GetResourceManager(bundleName)
    [frameworks/resmgr/src/resource_manager.cpp]
    ↓
    返回 C++ ResourceManager
    ↓
[C 应用层]
OH_ResourceManager_GetString(rm, resId, buffer, len)
    ↓
[核心框架层]
ResourceManagerImpl::GetString(resId, value)
    ↓
    HapResourceManager::FindResource(resId, config)
    ↓
    LocaleMatcher::MatchResource(resources, config)
    ↓
    HapResource::GetValue()
    ↓
[C 应用层]
返回字符串值
```

**证据**: `05_InnerAPI.md`

---

### 7. 资源添加/移除

```
[JS 应用层]
mgr.addResource(hapPath, callback)
    ↓
[NAPI 层]
ResourceManagerNapiSyncImpl::AddResource(env, info)
    [interfaces/js/innerkits/core/src/resource_manager_napi_sync_impl.cpp:1055]
    ↓
    获取 HAP 路径
    ↓
[核心框架层]
ResourceManagerImpl::AddResource(hapPath)
    [frameworks/resmgr/src/resource_manager_impl.cpp]
    ↓
    HapManager::LoadHap(hapPath)
    ↓
    HapParser::ParseHap(hapPath)
    ↓
    添加到 HapResourceManager
    ↓
    更新资源索引
    ↓
[JS 应用层]
返回成功
```

**移除资源调用链**:
```
mgr.removeResource(hapPath, callback)
    ↓
ResourceManagerNapiSyncImpl::RemoveResource(env, info)
    ↓
ResourceManagerImpl::RemoveResource(hapPath)
    ↓
HapManager::RemoveHap(hapPath)
    ↓
从 HapResourceManager 移除
    ↓
返回成功
```

**证据**: `04_NAPI.md`

---

## 关键数据流

### 资源查找数据流

```
用户输入 (resId, config)
    ↓
ResourceManagerImpl::GetString
    ↓
HapResourceManager::FindResource
    ↓
    遍历所有 HapResource
    ↓
    LocaleMatcher::MatchResource
    ↓
        计算每个资源的匹配分数
        ↓
        选择分数最高的资源
    ↓
HapResource::GetValue
    ↓
    从索引文件读取资源值
    ↓
返回资源值
```

### HAP 解析数据流

```
HAP 文件
    ↓
HapParser::ParseHap
    ↓
    读取索引文件 (resources.index)
    ↓
    读取资源配置 (config.json)
    ↓
    创建 HapResource 对象
    ↓
    加载资源数据
    ↓
HapResourceManager::LoadResource
    ↓
    建立资源索引
    ↓
资源可用
```

## 相关文档

- [N-API 接口](04_NAPI.md) - JavaScript API 详细文档
- [架构设计](03_Architecture.md) - 组件架构和数据流
- [内部 API](05_InnerAPI.md) - C++ 内部接口

---

**生成时间**: 2026-02-06
**证据来源**: 代码分析和调用链追踪
