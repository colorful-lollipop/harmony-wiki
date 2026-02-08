# N-API 参考指南

本文档描述方舟工具链与 ArkCompiler Runtime 交互的 N-API 接口使用方式。虽然本仓库不是典型的 N-API 模块实现仓库，但工具链通过 N-API 与运行时进行深度交互。

## 仓库性质说明

方舟工具链（arkcompiler/toolchain）是调试器（Debugger）和性能分析器（Profiler）的实现仓库，**不包含典型的 N-API 模块导出**。相反，它通过链接 ArkCompiler Runtime 并调用其内部 N-API 来实现调试功能。

**关键 N-API 头文件**（位于 ets_runtime 仓库）：
- `ecmascript/napi/include/jsnapi.h` - 核心 N-API 接口
- `ecmascript/napi/include/jsnapi_expo.h` - 扩展 N-API 接口
- `ecmascript/napi/include/dfx_jsnapi.h` - DFX 调试接口

## VM 生命周期管理

### 创建与销毁

**API**: `JSNApi::CreateEcmaVM()` / `JSNApi::DestroyJSVM()`

**位置**: `tooling/dynamic/client/ark_multi/main.cpp:xx`

```cpp
#include "ecmascript/napi/include/jsnapi_expo.h"

// 创建 VM
panda::ecmascript::EcmaVM *vm = panda::JSNApi::CreateEcmaVM(g_runtimeOptions);

// 设置 Bundle 模式
panda::JSNApi::SetBundle(vm, !g_runtimeOptions.GetMergeAbc());

// 执行 Panda 文件
bool ret = panda::JSNApi::Execute(vm, file, entry);

// 销毁 VM
panda::JSNApi::DestroyJSVM(vm);
```

**Fuzz 测试用法**（`test/fuzztest/common_fuzzer/common_fuzzer.h`）：

```cpp
#include "ecmascript/napi/include/jsnapi.h"

EcmaVM* common_fuzzer::GetEcvm()
{
    RuntimeOption option;
    option.SetLogLevel(RuntimeOption::LOG_LEVEL::ERROR);
    auto vm = JSNApi::CreateJSVM(option);
    return vm;
}

void common_fuzzer::DestroyEcvm(EcmaVM* vm)
{
    JSNApi::DestroyJSVM(vm);
}
```

### 参数配置

通过 `RuntimeOption` 类配置 VM 选项：

| 配置项 | 方法 | 说明 |
|--------|------|------|
| 日志级别 | `SetLogLevel()` | ERROR / WARNING / INFO / DEBUG |
| 内存限制 | `SetGcType()` | GC 类型配置 |
| ABC 合并 | `GetMergeAbc()` | 是否合并 ABC 文件 |

## 调试器功能绑定

### 全局对象函数绑定

**API**: `JSNApi::GetGlobalObject()`, `ObjectRef::Set()`

**位置**: `tooling/dynamic/backend/debugger_executor.cpp:xx`

```cpp
// 设置调试器访问器到全局对象
void DebuggerExecutor::SetDebuggerAccessor(const EcmaVM *vm, const Local<JSValueRef> &globalEnv)
{
    auto setStr = StringRef::NewFromUtf8(vm, "debuggerSetValue");
    auto getStr = StringRef::NewFromUtf8(vm, "debuggerGetValue");
    Local<ObjectRef> globalObj = JSNApi::GetGlobalObject(vm, globalEnv);
    
    // 绑定函数到全局对象
    globalObj->Set(vm, setStr, FunctionRef::New(const_cast<panda::EcmaVM*>(vm),
        globalEnv, DebuggerExecutor::DebuggerSetValue));
    globalObj->Set(vm, getStr, FunctionRef::New(const_cast<panda::EcmaVM*>(vm),
        globalEnv, DebuggerExecutor::DebuggerGetValue));
}
```

### 常用 API 清单

| API | 用途 | C++ 类型 |
|-----|------|----------|
| `JSNApi::CreateEcmaVM()` | 创建 ECMAScript VM | `EcmaVM*` |
| `JSNApi::DestroyJSVM()` | 销毁 VM | void |
| `JSNApi::GetGlobalObject()` | 获取全局对象 | `Local<ObjectRef>` |
| `JSNApi::Execute()` | 执行脚本 | `bool` |
| `JSNApi::SetBundle()` | 设置 Bundle 模式 | void |
| `StringRef::NewFromUtf8()` | 创建 UTF-8 字符串 | `Local<StringRef>` |
| `FunctionRef::New()` | 创建 JS 函数 | `Local<FunctionRef>` |
| `ObjectRef::Set()` | 设置对象属性 | `bool` |

## 性能统计中的 N-API

CPU Profiler 单独统计 N-API 调用时间，用于性能分析。

### Profile 类时间字段

**位置**: `tooling/dynamic/base/pt_types.h`

```cpp
class Profile {
    int64_t GetNapiTime() const { return napiTime_; }
    Profile &SetNapiTime(int64_t napiTime) { napiTime_ = napiTime; return *this; }
    
private:
    int64_t napiTime_ {0};           // N-API 调用耗时
    int64_t gcTime_ {0};             // GC 耗时
    int64_t cInterpreterTime_ {0};   // C++ 解释器耗时
    int64_t asmInterpreterTime_ {0}; // ASM 解释器耗时
    int64_t aotTime_ {0};            // AOT 编译耗时
    int64_t builtinTime_ {0};        // 内置函数耗时
    int64_t arkuiEngineTime_ {0};    // ArkUI 引擎耗时
    int64_t runtimeTime_ {0};        // 运行时耗时
    int64_t otherTime_ {0};          // 其他耗时
};
```

### 时间字段序列化

**位置**: `tooling/static/types/profile_result.cpp`

```cpp
builder.AddProperty("napiTime", profileInfo.napiTime);
builder.AddProperty("gcTime", profileInfo.gcTime);
// ... 其他字段
```

## N-API 模块加载（热重载）

测试用例中涉及 N-API 模块加载功能：

**位置**: `test/autotest/testcases/toolchain/hotreload/TestHotReloadEnhanced.py`

| 方法 | 说明 |
|------|------|
| `napiLdModule` | N-API 加载模块 |
| `napiLdModuleWithInfo` | 带信息的 N-API 模块加载 |

## 与外部 N-API 模块的对比

| 特性 | 本仓库（工具链） | 典型 N-API 模块 |
|------|------------------|-----------------|
| N-API_MODULE 宏 | 无 | 有 |
| N-API 导出函数 | 无 | 有 |
| JS 命名空间导出 | 无 | 有 |
| 链接方式 | 静态链接运行时 | 动态加载 .so |
| 头文件位置 | 外部依赖 | 模块内 include/ |

## 扩展开发指南

如需在此仓库基础上扩展 N-API 功能：

1. **包含头文件**: `#include "ecmascript/napi/include/jsnapi.h"`
2. **使用 JSNApi 命名空间**: 调用 `JSNApi::` 静态方法
3. **处理 JS 值**: 使用 `Local<T>` 模板类
4. **创建 JS 字符串**: `StringRef::NewFromUtf8(vm, "text")`
5. **绑定 C++ 函数**: `FunctionRef::New(vm, callback)`

---

*相关文档：[00_Overview.md](./00_Overview.md) | [02_Architecture.md](./02_Architecture.md) | [04_Internal_API.md](./04_Internal_API.md)*
