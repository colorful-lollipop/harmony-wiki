# 关键调用链

本文档描述方舟工具链的核心调用链，从入口到核心逻辑的完整路径追踪。

## 调试连接建立调用链

### 场景：DevEco Studio 连接调试服务器

```
DevEco Studio (客户端)
    │
    ▼ (WebSocket 连接)
WebSocketServer::AcceptNewConnection()
  └─> WebSocketServer::HttpHandShake()
      └─> WebSocketServer::ProtocolUpgrade()
          └─> WebSocketBase::SetConnectionState(OPEN)
              │
              ▼ (协议消息)
Inspector::OnConnect()
  └─> Inspector::SendMessage()
      └─> WebSocketBase::SendReply()
          │
          ▼ (初始化调试会话)
DebuggerImpl::DebuggerImpl()
  └─> DebuggerImpl::Enable()
      └─> Frontend::DebuggerEnabled()
```

## 断点设置调用链

```
DebuggerImpl::SetBreakpointByUrl()
  │
  ├─> 参数解析：url, lineNumber, columnNumber
  ├─> DebuggerImpl::ProcessSingleBreakpoint()
  │    │
  │    ├─> Script::GetSourceURL() - 获取脚本 URL
  │    ├─> DebugInfoExtractor::GetLineNumberTable() - 获取行号表
  │    └─> JSDebugger::AddBreakpoint() - 注册断点
  │
  └─> Frontend::BreakpointResolved()
```

## 断点命中调用链

```
运行时触发断点
    │
    ▼
JSPtHooks::Breakpoint(breakpointId)
    │
    ├─> DebuggerImpl::Pause()
    │    │
    │    ├─> DebuggerImpl::NotifyPaused()
    │    │    │
    │    │    ├─> DebuggerImpl::CollectCallFrames() - 收集调用栈
    │    │    └─> Frontend::Paused(reason, callFrames) - 发送暂停事件
    │    │
    │    └─> DebuggerImpl::SetState(State::PAUSED)
    │
    └─> return true (暂停执行)
```

## CallFrame 求值调用链

```
Frontend::EvaluateOnCallFrame(callFrameId, expression)
    │
    ▼
DebuggerImpl::EvaluateOnCallFrame()
    │
    ├─> CallFrame& frame = callFrames_[callFrameId]
    ├─> DebuggerExecutor::Evaluate()
    │    │
    │    ├─> DebuggerExecutor::GetValue(scope, name) - 获取变量
    │    │    ├─> LocalScope::GetValue()
    │    │    ├─> LexicalScope::GetValue()
    │    │    ├─> ModuleScope::GetValue()
    │    │    └─> GlobalScope::GetValue()
    │    │
    │    └─> RuntimeHelper::Evaluate(expression)
    │
    └─> Frontend::Response(result)
```

## CPU Profiler 调用链

```
ProfilerImpl::Start()
    │
    ▼
DFXJSNApi::StartCpuProfilerForInfo(vm)
    │
    └─> CpuProfiler::Start() - 启动采样定时器
    
───────────────────────────────────────────

ProfilerImpl::Stop()
    │
    ▼
DFXJSNApi::StopCpuProfilerForInfo(vm)
    │
    ├─> CpuProfiler::Stop() - 停止采样
    ├─> CpuProfiler::Export() - 导出 Profile 数据
    └─> ProfilerImpl::SerializeProfile() - 序列化
        │
        └─> Profile::Serialize() - 转换为 JSON
            │
            ├─> Profile::AddProperty("napiTime", ...)
            ├─> Profile::AddProperty("gcTime", ...)
            └─> Profile::AddProperty("samples", ...)
```

## Heap Snapshot 调用链

```
HeapProfilerImpl::TakeHeapSnapshot()
    │
    ▼
DFXJSNApi::DumpHeapSnapshot(vm, stream)
    │
    ├─> HeapSnapshot::Build() - 构建堆快照
    │    │
    │    ├─> HeapSnapshot::CollectNodes() - 收集堆节点
    │    ├─> HeapSnapshot::CollectEdges() - 收集引用边
    │    └─> HeapSnapshot::Serialize() - 序列化
    │
    └─> Stream::Write(chunk) - 分块传输
         │
         ├─> HeapProfilerStream::Write()
         └─> Frontend::HeapSnapshot(chunk)
```

## GC 触发调用链

```
HeapProfilerImpl::CollectGarbage()
    │
    ▼
JSNApi::TriggerGC(vm, GCType::FULL_GC)
    │
    ├─> EcmaVM::CollectGarbage()
    │    ├─> Heap::CollectGarbage()
    │    └─> SharedHeap::CollectGarbage()
    │
    └─> Frontend::GarbageCollected()
```

## 协议消息分发调用链

```
WebSocketBase::HandleDataFrame()
    │
    ▼
Dispatcher::Dispatch(request, response)
    │
    ├─> request.GetDomain() - 获取域标识
    ├─> dispatcher = dispatchers_[domain] - 查找分发器
    │
    └─> DispatcherBase::Dispatch()
         │
         ├─> DebuggerImpl::Dispatch() - 调试域
         ├─> RuntimeImpl::Dispatch() - 运行时域
         ├─> ProfilerImpl::Dispatch() - CPU 分析域
         ├─> HeapProfilerImpl::Dispatch() - 堆分析域
         └─> ... 其他域
```

## 文件描述符生命周期

```
平台初始化
    │
    ▼
FdsanExchangeOwnerTag(fd)
    │
    ├─> Unix: fdsan_close_with_tag() 或 close()
    └─> Windows: CloseHandle()
```

---

*相关文档：[02_Architecture.md](./02_Architecture.md) | [03_NAPI_Reference.md](./03_NAPI_Reference.md) | [04_Internal_API.md](./04_Internal_API.md)*
