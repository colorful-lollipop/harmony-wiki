# 架构说明

> 组件组件图、数据流、线程模型与关键时序

## 1. 整体架构

### 1.1 分层架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    ArkTS 应用层                              │
│  @ohos.url | @ohos.xml | @ohos.buffer | @ohos.util       │
└────────────────────────┬──────────────────────────────────┘
                         │ JS API 调用
┌────────────────────────▼──────────────────────────────────┐
│                 N-API 桥接层                                 │
│  native_module_*.cpp (URL/XML/Buffer/Container 实现)        │
│  • 参数转换: JS → C++                                        │
│  • 类型检查: napi_type_tag                                   │
│  • 错误码转换                                               │
└────────────────────────┬──────────────────────────────────┘
                         │ Native 调用
┌────────────────────────▼──────────────────────────────────┐
│                Native 实现层                                 │
│  • Buffer 内存管理                                          │
│  • XML 解析 (libxml2)                                       │
│  • 进程通信 (IPC/Binder)                                    │
│  • 线程池管理 (FFRT)                                        │
└────────────────────────┬──────────────────────────────────┘
                         │ 系统调用
┌────────────────────────▼──────────────────────────────────┐
│               OpenHarmony 系统服务                           │
│  • hilog 日志                                              │
│  • samgr 能力管理                                          │
│  • icu 国际化                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 模块关系图

```
┌──────────────────────────────────────────────────────────────┐
│                      js_api_module                           │
│  URL ──┐                                                     │
│  URI ──┼──▶ 数据处理 (Buffer/XML)                           │
│  XML ──┤                                                    │
│ Buffer ─┘                                                    │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      js_util_module                           │
│  Container ──▶ 容器 (List/Map/Set)                           │
│  Util ──────▶ 工具 (TextEncoder/LruBuffer)                   │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      js_sys_module                            │
│  Process ──▶ 进程/线程操作                                   │
│  Timer ────▶ 定时器                                          │
│  Console ──▶ 日志输出                                        │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                   js_concurrent_module                        │
│  Worker ───▶ 多线程通信                                      │
│  Taskpool ─▶ 任务调度                                        │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                        platform                              │
│  ohos/ │ android/ │ ios/ │ default                         │
│  平台抽象层 (适配不同操作系统)                                 │
└──────────────────────────────────────────────────────────────┘
```

## 2. N-API 调用流程

### 2.1 URL 创建时序图

```
participant User as "ArkTS 用户代码"
participant NAPI as "N-API 桥接层"
participant Native as "Native 实现"
participant System as "系统服务"

User->>NAPI: new URL('http://example.com')
NAPI->>NAPI: UrlStructor() 解析参数
NAPI->>Native: new URL(input, base)
Native->>Native: 验证 URL 格式
Native->>Native: 解析协议/主机/路径
Native-->NAPI: 返回 URL 对象
NAPI-->User: 返回 URL 实例
```

### 2.2 Buffer 读写时序图

```
participant User as "ArkTS 用户代码"
participant Buffer as "Buffer N-API"
participant Memory as "Native Buffer"

User->>Buffer: buffer.alloc(1024)
Buffer->>Buffer: 分配内存 (new Buffer)
Buffer->>Buffer: napi_wrap_s() 绑定对象
Buffer-->User: Buffer 实例

User->>Buffer: buf.writeString('hello')
Buffer->>Memory: WriteString()
Memory->>Memory: 内存拷贝
Buffer-->User: 写入字节数

User->>Buffer: buf.toString()
Buffer->>Memory: 读取数据
Memory->>Buffer: 返回字符串
Buffer-->User: 'hello'
```

### 2.3 Worker 通信时序图

```
participant Main as "主线程"
participant Worker as "Worker 线程"
participant NAPI as "N-API 桥接"
participant Queue as "消息队列"

Main->>Worker: new Worker('worker.js')
Worker->>NAPI: 创建 Worker 线程
NAPI->>Worker: 初始化线程环境
Worker->>Queue: 创建消息队列

Main->>Worker: worker.postMessage(data)
Main->>Queue: 发送消息
Queue->>Worker: 投递消息

Worker->>Worker: 处理消息
Worker->>Main: parentPort.postMessage(response)
Main->>Main: onmessage 回调

Main->>Worker: worker.terminate()
Worker->>NAPI: 清理资源
NAPI->>Worker: 退出线程
```

## 3. 线程模型

### 3.1 单线程模块

以下模块在主线程执行：
- **js_api_module**: URL、Buffer、XML 操作
- **js_util_module**: 容器、JSON、工具类
- **js_sys_module**: Process、Timer、Console

### 3.2 多线程模块

| 模块 | 线程模型 | 说明 |
|------|----------|------|
| **Worker** | 独立线程 | 每个 Worker 实例拥有独立线程 |
| **Taskpool** | 线程池 | 使用 FFRT 线程池调度 |
| **IPC** | 共享线程 | 与系统服务共享 IPC 线程 |

### 3.3 线程安全机制

```cpp
// Worker 中的线程同步示例
class Worker {
    std::mutex messageMutex_;              // 消息互斥锁
    std::condition_variable messageCv_;    // 消息条件变量
    MessageQueue messageQueue_;            // 消息队列
    
    void SendMessage(Message&& msg) {
        std::lock_guard<std::mutex> lock(messageMutex_);
        messageQueue_.Push(std::move(msg));
        messageCv_.notify_one();
    }
};
```

## 4. 内存管理

### 4.1 对象生命周期

```
┌─────────────────────────────────────────────────────────┐
│                   Buffer 对象生命周期                      │
├─────────────────────────────────────────────────────────┤
│  1. 创建阶段                                               │
│     new Buffer() → napi_wrap_s() 绑定到 JS 对象            │
│                                                           │
│  2. 使用阶段                                               │
│     JS 持有引用 → Native 持有指针                         │
│                                                           │
│  3. 销毁阶段                                               │
│     GC 回收 → FinalizeCallback → delete Buffer             │
└─────────────────────────────────────────────────────────┘
```

### 4.2 内存分配策略

| 场景 | 分配方式 | 位置 |
|------|----------|------|
| 小对象 (<4KB) | 堆分配 | native heap |
| 池化 Buffer | 内存池 | BufferPool |
| 大对象 (>1MB) | 直接分配 | system heap |

## 5. 错误处理机制

### 5.1 错误码定义

```cpp
// XML 模块错误码
static const int32_t ERROR_CODE = 401;  // 参数类型错误

// Buffer 模块错误处理
NAPI_CALL(env, napi_get_value_string_utf8(...));
if (status != napi_ok) {
    HILOG_ERROR("Failed to get string value");
    return nullptr;
}
```

### 5.2 异常传播

```
JS 层异常
    ↓
napi_throw_error(env, "401", "Parameter error")
    ↓
JS 层抛出 Error
```

## 6. 相关文档

- [01_Overview.md](./01_Overview.md) - 组件定位
- [03_API_js_api_module.md](./03_API_js_api_module.md) - API 详情
- [07_Build_Configuration.md](./07_Build_Configuration.md) - 构建配置

---

*文档版本: 1.0*
*最后更新: 2026-02-06*
