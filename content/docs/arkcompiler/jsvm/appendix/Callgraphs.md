# 调用链图

> 关键调用链（入口→核心逻辑）

## 目的与适用范围

### 目的
本文档提供关键 API 的详细调用链，帮助开发者理解代码执行流程。

### 适用范围
- 需要理解 API 实现的开发者
- 需要进行性能优化的开发者
- 需要进行调试的开发者

---

## VM 初始化调用链

### OH_JSVM_Init

```
应用代码
  ↓
OH_JSVM_Init(options)
  ↓ (interface/kits/jsvm.h:323)
src/js_native_api_v8.cpp:1059
  ↓
v8::V8::Initialize()
  ↓ (V8 引擎初始化）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:323` - API 声明
- `src/js_native_api_v8.cpp:1059` - API 实现

### OH_JSVM_CreateVM

```
应用代码
  ↓
OH_JSVM_CreateVM(options, &result)
  ↓ (interface/kits/jsvm.h:333)
src/js_native_api_v8.cpp:1094
  ↓
v8::Isolate::New(create_params)
  ↓ (创建 V8 Isolate）
CreateIsolateData(isolate, blob)
  ↓ (创建 Isolate 数据）
SetContextEnv(context, env)
  ↓ (设置 Environment）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:333` - API 声明
- `src/js_native_api_v8.cpp:1094` - API 实现

---

## Environment 创建调用链

### OH_JSVM_CreateEnv

```
应用代码
  ↓
OH_JSVM_CreateEnv(vm, propertyCount, properties, &result)
  ↓ (interface/kits/jsvm.h:433)
src/js_native_api_v8.cpp:1174
  ↓
v8::Context::New(isolate)
  ↓ (创建 V8 Context）
SetContextEnv(context, env)
  ↓ (设置 Environment）
v8::Context::Enter()
  ↓ (进入 Context）
DefineProperties(env, global, propertyCount, properties)
  ↓ (定义全局属性）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:433` - API 声明
- `src/js_native_api_v8.cpp:1174` - API 实现

---

## 代码编译与执行调用链

### OH_JSVM_CompileScript

```
应用代码
  ↓
OH_JSVM_CompileScript(env, script, cachedData,
                    cacheDataLength, eagerCompile,
                    &cacheRejected, &result)
  ↓ (interface/kits/jsvm.h:501)
src/js_native_api_v8.cpp:1274
  ↓
ValidateString(env, script)
  ↓ (验证字符串）
v8::ScriptCompiler::Compile(context, code, origin)
  ↓ (V8 编译）
CreateJSVMScript(isolate, compiled_script)
  ↓ (创建 JSVM_Script）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:501` - API 声明
- `src/js_native_api_v8.cpp:1274` - API 实现

### OH_JSVM_RunScript

```
应用代码
  ↓
OH_JSVM_RunScript(env, script, &result)
  ↓ (interface/kits/jsvm.h:581)
src/js_native_api_v8.cpp:1550
  ↓
v8::Script::Run(context)
  ↓ (V8 执行）
V8ToLocalValue(v8::result)
  ↓ (转换为 JSVM_Value）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:581` - API 声明
- `src/js_native_api_v8.cpp:1550` - API 实现

---

## JS/C++ 函数调用调用链

### JS 调用 C++ 函数

```
JavaScript 代码
  ↓
调用导出的 C++ 函数
  ↓
V8 引擎
  ↓ (调用 C++ 回调）
JSVM Core
  ↓ (v8impl::CallbackWrapper::Call）
应用 C++ 函数
  ↓ (执行用户代码）
返回 JSVM_Value
  ↓
V8 引擎
  ↓ (转换为 v8::Value）
返回给 JavaScript
```

**证据位置**:
- `interface/kits/jsvm_types.h:157-167` - JSVM_Callback 定义
- `src/js_native_api_v8.cpp` - 回调实现

---

## Inspector 调用链

### OH_JSVM_OpenInspector

```
应用代码
  ↓
OH_JSVM_OpenInspector(env, "0.0.0.0", 9229)
  ↓ (interface/kits/jsvm.h:1719)
src/inspector/js_native_api_v8_inspector.cpp
  ↓
CreateInspectorSocketServer(env, host, port)
  ↓ (创建 WebSocket 服务器）
InspectorSocketServer::Start()
  ↓ (启动服务器）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:1719` - API 声明
- `src/inspector/js_native_api_v8_inspector.cpp` - API 实现
- `src/inspector/inspector_socket_server.cpp` - WebSocket 服务器

---

## 引用管理调用链

### OH_JSVM_CreateReference

```
应用代码
  ↓
OH_JSVM_CreateReference(env, value, initialRefcount, &result)
  ↓ (interface/kits/jsvm.h:823)
src/jsvm_reference.cpp
  ↓
CreateJSVMRef(value, initialRefcount)
  ↓ (创建引用对象）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:823` - API 声明
- `src/jsvm_reference.cpp` - 引用实现

### OH_JSVM_DeleteReference

```
应用代码
  ↓
OH_JSVM_DeleteReference(env, ref)
  ↓ (interface/kits/jsvm.h:853)
src/jsvm_reference.cpp
  ↓
DeleteJSVMRef(ref)
  ↓ (删除引用对象）
返回 JSVM_OK
```

**证据位置**:
- `interface/kits/jsvm.h:853` - API 声明
- `src/jsvm_reference.cpp` - 引用实现

---

## 相关链接

- [架构说明](./04_Architecture.md) - 组件关系
- [对外 API 详细文档](./05_Public_API_Details.md) - API 使用
