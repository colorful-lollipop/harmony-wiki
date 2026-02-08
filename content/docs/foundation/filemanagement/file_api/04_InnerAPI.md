# 内部 API

## 简介

本文档描述 File API 项目的内部 API，包括模块接口、依赖方向和稳定性评估。

## 模块层次

```
┌─────────────────────────────────────────────────────────────┐
│                    对外接口层                                  │
│  N-API / ANI / FFI / C API                                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    核心实现层                                  │
│  mod_fs / mod_fileio / mod_hash / ...                        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    Native 组件层                               │
│  TaskSignal / RemoteURI / Environment                        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    工具库层                                    │
│  filemgmt_libn / filemgmt_libfs / filemgmt_libhilog          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    系统服务层                                  │
│  libuv / IPC / DFS / Kernel                                  │
└─────────────────────────────────────────────────────────────┘
```

## 工具库 API (Inner Kits)

### filemgmt_libn

**稳定性**: ⭐⭐⭐⭐⭐ 稳定

**头文件**: `utils/filemgmt_libn/include/`

#### NVal 类

**文件**: `utils/filemgmt_libn/include/n_val.h`

**用途**: N-API 值封装

**稳定性**: 稳定

```cpp
class NVal {
public:
    // 创建值
    static NVal CreateObject(napi_env env);
    static NVal CreateInt64(napi_env env, int64_t value);
    static NVal CreateUTF8String(napi_env env, const std::string &str);
    static NVal CreateUndefined(napi_env env);
    
    // 类型检查
    bool TypeIs(napi_valuetype type) const;
    bool IsBuffer() const;
    bool IsArrayBuffer() const;
    
    // 类型转换
    std::tuple<bool, int64_t> ToInt64() const;
    std::tuple<bool, std::unique_ptr<char[]>, size_t> ToUTF8String() const;
    std::tuple<bool, void*, size_t> ToBuffer() const;
    std::tuple<bool, std::unique_ptr<char[]>, size_t> ToUTF8StringPath() const;
    
    // 属性操作
    bool AddProp(const std::vector<napi_property_descriptor> &props);
    static napi_property_descriptor DeclareNapiProperty(const char *name, napi_value val);
    static napi_property_descriptor DeclareNapiFunction(const char *name, napi_callback func);
};
```

#### NClass 类

**文件**: `utils/filemgmt_libn/include/n_class.h`

**用途**: JS 类定义和实例化

**稳定性**: 稳定

```cpp
class NClass {
public:
    // 定义类
    static bool DefineClass(napi_env env,
                           const std::string &className,
                           const napi_callback constructor,
                           const std::vector<napi_property_descriptor> &props);
    
    // 实例化
    static napi_value InstantiateClass(napi_env env,
                                       const std::string &className,
                                       std::initializer_list<napi_value> args);
    
    // 实体管理
    template <class T>
    static T *GetEntityOf(napi_env env, napi_value obj);
    
    template <class T>
    static bool SetEntityFor(napi_env env, napi_value obj, std::unique_ptr<T> entity);
    
    template <class T>
    static T *RemoveEntityOfFinal(napi_env env, napi_value obj);
};
```

#### NFuncArg 类

**文件**: `utils/filemgmt_libn/include/n_func_arg.h`

**用途**: 参数处理

**稳定性**: 稳定

```cpp
class NFuncArg {
public:
    NFuncArg(napi_env env, napi_callback_info info);
    
    // 初始化参数
    bool InitArgs(size_t minArgc, size_t maxArgc);
    
    // 访问参数
    napi_value operator[](size_t pos) const;
    size_t GetArgc() const;
    napi_value GetThisVar() const;
};

// 常量
enum NARG_CNT { ZERO, ONE, TWO, THREE, FOUR };
enum NARG_POS { FIRST, SECOND, THIRD, FOURTH };
```

#### NExporter 类

**文件**: `utils/filemgmt_libn/include/n_exporter.h`

**用途**: 模块导出基类

**稳定性**: 稳定

```cpp
class NExporter {
public:
    NExporter(napi_env env, napi_value exports);
    virtual ~NExporter() = default;
    
    virtual bool Export() = 0;
    virtual std::string GetClassName() = 0;
    
protected:
    NVal exports_;
};
```

#### 异步工作类

**文件**: 
- `utils/filemgmt_libn/include/n_async/n_async_work_promise.h`
- `utils/filemgmt_libn/include/n_async/n_async_work_callback.h`

**用途**: Promise 和 Callback 异步模式

**稳定性**: 稳定

```cpp
// Promise 模式
class NAsyncWorkPromise : public NAsyncWork {
public:
    NAsyncWorkPromise(napi_env env, NVal thisVar);
    NVal Schedule(const std::string &procedureName,
                  NContextCBExec cbExec,
                  NContextCBComplete cbComplete);
};

// Callback 模式
class NAsyncWorkCallback : public NAsyncWork {
public:
    NAsyncWorkCallback(napi_env env, NVal thisVar, NVal cb);
    NVal Schedule(const std::string &procedureName,
                  NContextCBExec cbExec,
                  NContextCBComplete cbComplete);
};

// 回调类型
using NContextCBExec = std::function<UniError(napi_env)>;
using NContextCBComplete = std::function<NVal(napi_env, UniError)>;
```

### filemgmt_libfs

**稳定性**: ⭐⭐⭐⭐ 稳定（部分 API 可能变化）

**头文件**: `utils/filemgmt_libfs/include/`

#### FSResult 类

**文件**: `utils/filemgmt_libfs/include/fs_result.h`

**用途**: 文件操作结果封装

**稳定性**: 稳定

```cpp
template <typename T, typename E>
class FSResult {
public:
    bool HasValue() const;
    T &Value();
    E &Error();
    // ...
};
```

#### FSError 类

**文件**: `utils/filemgmt_libfs/include/fs_error.h`

**用途**: 错误码定义

**稳定性**: 稳定

```cpp
enum class FSError {
    SUCCESS = 0,
    INVALID_PARAM,
    PERMISSION_DENIED,
    FILE_NOT_FOUND,
    // ...
};
```

## Native 组件 API

### TaskSignal

**稳定性**: ⭐⭐⭐⭐ 稳定

**头文件**: `interfaces/kits/native/task_signal/task_signal.h`

**用途**: 任务取消信号

```cpp
class TaskSignal {
public:
    int32_t Cancel();
    bool IsCanceled();
    bool CheckCancelIfNeed(const std::string &path);
    
    void SetRemoteTask(bool isRemote);
    bool IsRemoteTask() const;
    
    void SetDfsCopyTask(bool isDfsCopy);
    bool IsDfsCopyTask() const;
};
```

**依赖方向**: 
- 上层: `mod_fs` (Copy 等操作)
- 下层: 无

### RemoteUri

**稳定性**: ⭐⭐⭐⭐ 稳定

**头文件**: `interfaces/kits/native/remote_uri/remote_uri.h`

**用途**: 远程 URI 处理

```cpp
class RemoteUri {
public:
    static bool IsRemoteUri(const std::string &path, int &fd, const int &flags);
    static std::string GetCallingPkgName();
    // ...
};
```

**依赖方向**:
- 上层: `mod_fs`, `mod_fileio`
- 下层: `data_share`, `ipc`

## 模块接口稳定性

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| `filemgmt_libn` | ⭐⭐⭐⭐⭐ | 核心框架，高度稳定 |
| `filemgmt_libfs` | ⭐⭐⭐⭐ | 工具库，可能扩展 |
| `filemgmt_libhilog` | ⭐⭐⭐⭐⭐ | 日志库，稳定 |
| `task_signal` | ⭐⭐⭐⭐ | Native 组件，稳定 |
| `remote_uri` | ⭐⭐⭐⭐ | Native 组件，稳定 |
| `environment_native` | ⭐⭐⭐ | 可能随系统变化 |
| `fileio_native` | ⭐⭐ | 旧版，维护模式 |

## 可替换点

### 1. 异步框架替换

**当前**: LibN (基于 libuv)

**可替换性**: 中等

**替换成本**: 高（需重写所有 NAPI 模块）

**接口边界**: `utils/filemgmt_libn/include/n_async/`

### 2. 日志系统替换

**当前**: HiLog

**可替换性**: 高

**替换成本**: 低

**接口边界**: `utils/filemgmt_libhilog/filemgmt_libhilog.h`

```cpp
// 替换为其他日志系统
#ifdef USE_CUSTOM_LOG
    #define HILOGE(...) CustomLogError(__VA_ARGS__)
#else
    // 原 HiLog 实现
#endif
```

### 3. 文件系统后端替换

**当前**: Linux 系统调用 (通过 libuv)

**可替换性**: 中等

**替换成本**: 中等

**接口边界**: `utils/filemgmt_libfs/`

## 版本兼容性

### API 版本策略

| 类型 | 说明 | 示例 |
|------|------|------|
| 稳定 API | 保证向后兼容 | LibN 框架 |
| 实验性 API | 可能变化 | 新功能预览 |
| 废弃 API | 计划移除 | 旧版接口 |

### 废弃接口清单

| 接口 | 替代方案 | 废弃版本 | 移除计划 |
|------|----------|----------|----------|
| `mod_fileio` | `mod_fs` | 3.0 | 待定 |
| `fileio_native` | 直接使用 `mod_fs` | 3.0 | 待定 |

## 接口使用建议

### 新模块开发

1. **使用 LibN 框架**: 继承 `NExporter`，使用 `NVal`, `NClass`, `NAsyncWork`
2. **遵循命名规范**: 类名以 `NExporter` 结尾，文件名以 `_n_exporter` 结尾
3. **正确管理资源**: 使用 `FDGuard`, `NRef` 等 RAII 类
4. **统一错误处理**: 使用 `UniError` 传递错误

### 示例模板

```cpp
// my_feature_n_exporter.h
#pragma once
#include "n_exporter.h"

class MyFeatureNExporter : public OHOS::FileManagement::LibN::NExporter {
public:
    MyFeatureNExporter(napi_env env, napi_value exports);
    bool Export() override;
    std::string GetClassName() override;
    
private:
    static napi_value MySyncMethod(napi_env env, napi_callback_info info);
    static napi_value MyAsyncMethod(napi_env env, napi_callback_info info);
};

// my_feature_n_exporter.cpp
#include "my_feature_n_exporter.h"
#include "n_val.h"
#include "n_func_arg.h"
#include "n_async/n_async_work_promise.h"

bool MyFeatureNExporter::Export() {
    return exports_.AddProp({
        NVal::DeclareNapiFunction("mySyncMethod", MySyncMethod),
        NVal::DeclareNapiFunction("myAsyncMethod", MyAsyncMethod),
    });
}

napi_value MyFeatureNExporter::MyAsyncMethod(napi_env env, napi_callback_info info) {
    NFuncArg funcArg(env, info);
    funcArg.InitArgs(NARG_CNT::ONE, NARG_CNT::TWO);
    
    auto [succ, path] = NVal(env, funcArg[NARG_POS::FIRST]).ToUTF8StringPath();
    // ... 实现逻辑
    
    NVal thisVar(env, funcArg.GetThisVar());
    return NAsyncWorkPromise(env, thisVar).Schedule("MyMethod", cbExec, cbComplete).val_;
}
```
