# SmartPerf WASM/JS 绑定接口

## 概述

SmartPerf 使用 **Emscripten WebAssembly** 技术将 trace_streamer 编译为 WASM 模块，使浏览器可以直接调用 C++ 解析能力。**注意**：此项目不使用 Node-API (N-API)，而是采用 EMSCRIPTEN_KEEPALIVE 导出函数机制。

**代码位置**: `smartperf_host/trace_streamer/src/rpc/wasm_func.cpp`

## WASM 模块清单

### 主 WASM 导出函数

| 函数名 | 用途 | JS 调用示例 |
|--------|------|------------|
| `Initialize` | 初始化 WASM，注册回调函数 | `_Initialize(callbackPtr)` |
| `InitializeSplitFile` | 初始化文件分割 | `_InitializeSplitFile(ptr, len)` |
| `TraceStreamerParseDataEx` | 解析 trace 数据 | `_TraceStreamerParseDataEx(data, isFinish)` |
| `TraceStreamerParseDataOver` | 解析完成通知 | `_TraceStreamerParseDataOver()` |
| `TraceStreamerSqlQueryEx` | SQL 查询（回调返回 JSON） | `_TraceStreamerSqlQueryEx(sqlLen)` |
| `TraceStreamerSqlQueryToProtoCallback` | SQL 查询（返回 Protobuf） | `_TraceStreamerSqlQueryToProtoCallback(sqlLen)` |
| `TraceStreamerSqlOperateEx` | SQL 操作（DDL/DML） | `_TraceStreamerSqlOperateEx(sqlLen)` |
| `TraceStreamerSqlMetricsQuery` | Metrics 查询 | `_TraceStreamerSqlMetricsQuery(metricsId)` |
| `TraceStreamerReset` | 重置 WASM 状态 | `_TraceStreamerReset()` |
| `TraceStreamerCancel` | 取消 SQL 查询 | `_TraceStreamerCancel()` |
| `WasmExportDatabase` | 导出数据库 | `_WasmExportDatabase()` |
| `TraceStreamerSetLogLevel` | 设置日志级别 | `_TraceStreamerSetLogLevel(level)` |
| `TraceStreamerSetThirdPartyDataDealer` | 设置第三方数据处理回调 | `_TraceStreamerSetThirdPartyDataDealer(ptr)` |
| `TraceStreamerInitThirdPartyConfig` | 初始化第三方配置 | `_TraceStreamerInitThirdPartyConfig(ptr, len)` |
| `TraceStreamerDownloadELFEx` | 下载 ELF 文件（符号化） | `_TraceStreamerDownloadELFEx(ptr, len)` |
| `UpdateTraceTime` | 更新 trace 时间范围 | `_UpdateTraceTime(start, end)` |

**代码证据**: `smartperf_host/trace_streamer/src/rpc/wasm_func.cpp` (25+ 个 EMSCRIPTEN_KEEPALIVE 导出)

### SDK WASM 导出函数

**代码位置**: `smartperf_host/trace_streamer/sdk/demo_sdk/sdk/wasm_func.cpp`

| 函数名 | 用途 |
|--------|------|
| `Init` | 初始化 SDK WASM |
| `InitPluginName` | 初始化插件名称 |
| `TraceStreamerGetPluginNameEx` | 获取插件名称 |
| `TraceStreamer_In_ParseDataOver` | 解析完成通知 |
| `TraceStreamer_In_JsonConfig` | 获取 JSON 配置 |
| `ParserData` | 解析第三方数据 |
| `TraceStreamerSqlOperateEx` | SQL 操作 |
| `TraceStreamerSqlQueryEx` | SQL 查询 |

### SDK C API 函数

**代码位置**: `smartperf_host/trace_streamer/sdk/demo_sdk/sdk/ts_sdk_api.cpp`

| 函数名 | 用途 |
|--------|------|
| `SDKSetTableName` | 设置表名 |
| `SDKAppendCounterObject` | 添加计数器对象 |
| `SDKAppendCounter` | 添加计数器数据 |
| `SDKAppendSliceObject` | 添加切片对象 |
| `SDKAppendSlice` | 添加切片数据 |
| `SetRpcServer` | 设置 RPC 服务器 |

## JS 调用示例

### 1. 初始化 WASM

**代码位置**: `smartperf_host/ide/src/trace/database/TraceWorker.ts`

```typescript
// 加载 WASM 模块
const wasmModule = await loadWasmModule();

// 设置回调函数
const callbackPtr = this.registerCallback((result: number) => {
    console.log('WASM callback:', result);
});

// 初始化
wasmModule._Initialize(callbackPtr);
```

### 2. 解析 Trace 数据

```typescript
async parseTraceData(fileData: Uint8Array, isFinish: boolean = false): Promise<boolean> {
    // 分配内存
    const dataPtr = this.wasmModule._malloc(fileData.length);
    
    // 写入数据
    this.wasmModule.HEAPU8.set(fileData, dataPtr);
    
    // 调用解析
    const result = this.wasmModule._TraceStreamerParseDataEx(dataPtr, fileData.length, isFinish ? 1 : 0);
    
    // 释放内存
    this.wasmModule._free(dataPtr);
    
    return result === 0;
}
```

### 3. SQL 查询

```typescript
async executeSqlQuery(sql: string): Promise<any[]> {
    const sqlBuffer = new TextEncoder().encode(sql);
    const sqlPtr = this.wasmModule._malloc(sqlBuffer.length);
    
    this.wasmModule.HEAPU8.set(sqlBuffer, sqlPtr);
    
    // 触发查询，结果通过回调返回
    this.wasmModule._TraceStreamerSqlQueryEx(sqlPtr, sqlBuffer.length);
    
    this.wasmModule._free(sqlPtr);
    
    // 等待回调结果
    return await this.waitForQueryResult();
}
```

### 4. 重置状态

```typescript
async reset(): Promise<void> {
    this.wasmModule._TraceStreamerReset();
    this.queryResult = null;
    this.queryResolve = null;
}
```

## 数据表定义

### perf_napi_async 表

**注意**：`perf_napi_async` 中的 "napi" 指的是 **Linux NAPI（New API）网络子系统**，不是 Node-API。

**代码位置**: `smartperf_host/trace_streamer/src/table/hiperf/perf_napi_async_table.cpp`

**表结构** (11 列):

| 列名 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键 |
| ts | INTEGER | 时间戳 |
| traceid | INTEGER | Trace ID |
| cpu_id | INTEGER | CPU ID |
| thread_id | INTEGER | 线程 ID |
| process_id | INTEGER | 进程 ID |
| caller_callchainid | INTEGER | 调用者调用链 ID |
| callee_callchainid | INTEGER | 被调用者调用链 ID |
| perf_sample_id | INTEGER | Perf 采样 ID |
| event_count | INTEGER | 事件计数 |
| event_type_id | INTEGER | 事件类型 ID |

## 回调机制

### 回调注册

```cpp
// C++ 端注册回调
void RegisterCallback(void* userData, void (*callback)(int eventType, const char* data, void* userData)) {
    // 保存回调函数指针和用户数据
}
```

### 回调类型

| 事件类型 | 说明 |
|----------|------|
| 0 | 初始化完成 |
| 1 | 解析进度 |
| 2 | 解析完成 |
| 3 | 查询完成 |
| 4 | 错误事件 |

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [Inner Kit API](03_InnerAPI.md)
