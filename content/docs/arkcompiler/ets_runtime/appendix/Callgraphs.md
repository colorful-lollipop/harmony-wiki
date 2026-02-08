# 附录：关键调用链

本文档描述 ArkCompiler ETS Runtime 的关键调用链，从入口到核心逻辑的完整调用路径。

## 字节码执行调用链

### 1. 模块加载调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    模块加载调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  napi_load_module()                                             │
│      │                                                         │
│      ├──► napi_load_module_with_info()                          │
│      │       │                                                  │
│      │       ├──► JsPandafileManager::LoadAbcFile()           │
│      │       │       │                                         │
│      │       │       ├──► JsPandafile::Open()                 │
│      │       │       │       │                                 │
│      │       │       │       ├──► PandaFile::Open()           │
│      │       │       │       │       ├──► FileMapper::Map()   │
│      │       │       │       │       └──► FileHeader::Parse() │
│      │       │       │       │                                 │
│      │       │       │       └──► JsPandafile::Parse()       │
│      │       │       │               ├──► ConstantPool::Load()│
│      │       │       │               ├──► ClassReader::Load() │
│      │       │       │               └──► MethodReader::Load()│
│      │       │       │                                         │
│      │       │       └──► JsPandafileManager::AddEntry()      │
│      │       │                                                 │
│      │       └──► JSModuleManager::HostInitializeImport()     │
│      │                                                               │
│      └──► napi_get_named_property()                            │
│              │                                                  │
│              └──► 返回模块 exports 对象                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/napi/jsnapi.cpp:500-600`
- `ecmascript/jspandafile/js_pandafile.cpp:100-200`

### 2. 函数执行调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    函数执行调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  napi_call_function()                                           │
│      │                                                         │
│      ├──► napi_get_reference_value()                           │
│      │       │                                                  │
│      │       └──► 获取 JS Function 对象                         │
│      │                                                          │
│      ├──► 准备调用参数                                          │
│      │                                                          │
│      └──► Runtime::Invoke()                                     │
│              │                                                  │
│              ├──► Interpreter::Call()                          │
│              │       │                                          │
│              │       ├──► StackChecker::Check()                 │
│              │       ├──► FrameHandler::NewFrame()              │
│              │       │                                          │
│              │       ├──► Interpret()                          │
│              │       │       │                                  │
│              │       │       ├──► 指令分发                       │
│              │       │       ├──► 操作数解析                     │
│              │       │       └──► 执行指令                      │
│              │       │                                          │
│              │       └──► FrameHandler::PopFrame()              │
│              │                                                   │
│              └──► 返回执行结果                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/napi/jsnapi.cpp:1500-1600`
- `ecmascript/interpreter/interpreter.cpp:200-400`

### 3. 对象创建调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    对象创建调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  napi_create_object()                                           │
│      │                                                          │
│      └──► ObjectFactory::NewObject()                           │
│              │                                                   │
│              ├──► ObjectFactory::AllocateObject()               │
│              │       │                                           │
│              │       ├──► Heap::Allocate()                      │
│              │       │       │                                   │
│              │       │       └──► Allocator::Allocate()         │
│              │       │                                           │
│              │       └──► JSHClass::AllocHClass()               │
│              │                                                    │
│              ├──► JSHClass::SetPrototype()                     │
│              │                                                    │
│              └──► JSObject::Initialize()                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/napi/jsnapi.cpp:900-1000`
- `ecmascript/object_factory.cpp:100-300`

## N-API 核心调用链

### 4. 异步工作调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    异步工作调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  napi_create_async_work()                                       │
│      │                                                          │
│      ├──► 分配 AsyncWork 数据结构                               │
│      │                                                          │
│      └──► napi_queue_async_work()                               │
│              │                                                   │
│              ├──► TaskPool::SubmitTask()                        │
│              │       │                                           │
│              │       ├──► 创建 Task                              │
│              │       │                                           │
│              │       └──► 调度执行                                │
│              │                                                   │
│              └──► 完成后触发回调                                  │
│                      │                                           │
│                      └──► napi_call_as_function()               │
│                              │                                   │
│                              └──► 调用 JS 完成回调                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/napi/jsnapi.cpp:1700-1900`
- `common_components/taskpool/taskpool.cpp:100-300`

### 5. 字符串创建调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    字符串创建调用链                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  napi_create_string_utf8()                                      │
│      │                                                          │
│      ├──► 校验输入参数                                           │
│      │                                                          │
│      └──► EcmaString::Concat() / EcmaString::Flat()            │
│              │                                                   │
│              ├──► StringTable::InternString()                   │
│              │       │                                           │
│              │       ├──► 查找现有字符串                         │
│              │       │                                           │
│              │       └──► 分配新字符串                           │
│              │                                                   │
│              └──► 返回 TaggedString                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/napi/jsnapi.cpp:850-950`
- `ecmascript/ecma_string.cpp:100-300`

## 内存管理调用链

### 6. 对象分配调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    对象分配调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Heap::Allocate(size, type)                                     │
│      │                                                          │
│      ├──► CheckRuntimeTLS()                                     │
│      │                                                          │
│      ├──► GetAllocator()->Allocate(size)                        │
│      │       │                                                   │
│      │       ├──► TLAB::Allocate()                              │
│      │       │       │                                           │
│      │       │       └──► 从本地缓冲区分配                       │
│      │       │                                                   │
│      │       └──► HeapAllocator::Allocate()                     │
│      │               │                                           │
│      │               └──► 从堆空间分配                           │
│      │                                                           │
│      └──► InitializeObject(object, type)                        │
│              │                                                   │
│              ├──► 设置对象头                                     │
│              │                                                           │
│              └──► 初始化属性表                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/mem/heap.cpp:200-400`
- `ecmascript/mem/thread_local_allocation_buffer.cpp:100-200`

### 7. GC 触发调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    GC 触发调用链                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GC::CollectGarbage(zone)                                       │
│      │                                                          │
│      ├──► Heap::WaitUntilConcurrentMarkFinished()               │
│      │                                                          │
│      ├──► Heap::PrepareForGC()                                 │
│      │       │                                                   │
│      │       └──► 设置 GC 标记                                  │
│      │                                                           │
│      ├──► GCTracer::Sweep()                                     │
│      │       │                                                   │
│      │       ├──► ParallelSweep()                               │
│      │       │       │                                           │
│      │       │       ├──► SweepOldSpace()                       │
│      │       │       └──► SweepYoungSpace()                     │
│      │       │                                                   │
│      │       └──► ProcessFreeLists()                            │
│      │                                                           │
│      └──► Heap::FinishGC()                                      │
│              │                                                   │
│              └──► 恢复应用线程                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**代码位置**：
- `ecmascript/mem/gc*.cpp`（多个 GC 实现文件）
- `ecmascript/mem/heap.cpp:500-700`

## 相关文档

- [架构说明](../03_Architecture.md)
- [N-API 参考](../04_NAPI_Reference.md)
- [内部 API](../05_Inner_API.md)
