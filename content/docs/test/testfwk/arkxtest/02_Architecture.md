# 架构设计

## 整体架构

ArkXtest 采用分层客户端-服务器架构，支持多语言绑定，通过 IPC 通信实现测试框架与系统服务的交互。

### 架构图

```mermaid
graph TB
    subgraph "测试应用层"
        JS["ArkTS/JS\n测试脚本"]
        CJ["Cangjie\n测试脚本"]
    end

    subgraph "N-API/ANI 绑定层"
        NAPI["N-API\nlibuitest.z.so\nlibperftest.z.so"]
        ANI["ANI\nlibuitest_ani.so\nlibperftest_ani.so"]
        FFI["Cangjie FFI\nlibcj_ui_test_ffi.z.so"]
    end

    subgraph "IPC 通信层"
        IPC["IPC Transactor\nCommonEvent/IPC"]
    end

    subgraph "服务端层"
        UITS["UiTest\nDaemon"]
        PFTS["PerfTest\nDaemon"]
        TS["TestServer\nSA 5502"]
    end

    subgraph "系统服务层"
        A11Y["Accessibility"]
        WM["WindowManager"]
        INPUT["InputSystem"]
        CE["CommonEvent"]
        CPU["CPUFreq"]
        MEM["Memory"]
    end

    JS --> NAPI
    CJ --> FFI
    FFI --> NAPI
    
    NAPI --> IPC
    ANI --> IPC
    
    IPC --> UITS
    IPC --> PFTS
    
    UITS --> A11Y
    UITS --> WM
    UITS --> INPUT
    
    PFTS --> TS
    
    TS --> CE
    TS --> CPU
    TS --> MEM
```

## UiTest 架构

### 组件图

```mermaid
graph TB
    subgraph "客户端"
        Driver["Driver\nUiDriver.create()"]
        By["By/On\n控件选择器"]
        Component["Component\n控件对象"]
        UiWindow["UiWindow\n窗口对象"]
        Observer["UIEventObserver\n事件监听器"]
    end

    subgraph "N-API 绑定"
        NAPI["napi/\nuitest_napi.cpp"]
    end

    subgraph "IPC"
        IPC["connection/\nipc_transactor.cpp"]
    end

    subgraph "服务端核心"
        Selector["WidgetSelector\n控件选择器"]
        Operator["WidgetOperator\n控件操作"]
        WindowOp["WindowOperator\n窗口操作"]
        UIAction["UiAction\n输入注入"]
        Dump["DumpHandler\n布局导出"]
    end

    Driver --> By
    Driver --> Component
    Driver --> UiWindow
    Driver --> Observer
    
    By --> NAPI
    Component --> NAPI
    UiWindow --> NAPI
    Observer --> NAPI
    
    NAPI --> IPC
    IPC --> Selector
    IPC --> Operator
    IPC --> WindowOp
    IPC --> UIAction
    IPC --> Dump
```

### 目录职责

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `core/` | 核心逻辑：控件选择、组件操作、窗口管理 | `widget_selector.cpp`, `widget_operator.cpp` |
| `server/` | 服务端守护进程 | `server_main.cpp`, `system_ui_controller.cpp` |
| `connection/` | IPC 通信 | `ipc_transactor.cpp` |
| `input/` | 输入事件注入 | `ui_input.cpp` |
| `record/` | 录制回放 | `ui_record.cpp`, `pointer_tracker.cpp` |
| `addon/` | 扩展功能：截图 | `screen_copy.cpp` |
| `napi/` | ArkTS-Dynamic 绑定 | `uitest_napi.cpp` |
| `ets/ani/` | ArkTS-Static 绑定 | `uitest_ani.cpp` |
| `cj/` | Cangjie FFI | `uitest_ffi.cpp` |

**证据**: `uitest/BUILD.gn:30-246` 定义了各模块 targets

### 关键时序

#### 控件查找流程

```mermaid
sequenceDiagram
    participant JS as 测试脚本
    participant NAPI as uitest_napi.cpp
    participant IPC as ipc_transactor
    participant Server as uitest_server
    participant A11Y as Accessibility

    JS->>NAPI: Driver.findComponent(By)
    NAPI->>NAPI: 参数解析/验证
    NAPI->>IPC: SendRequest(GET_COMPONENT)
    IPC->>Server: CommonEvent IPC
    Server->>A11Y: Query UI Tree
    A11Y-->>Server: ElementInfo List
    Server->>Server: WidgetSelector 匹配
    Server-->>IPC: WidgetInfo JSON
    IPC-->>NAPI: ApiReplyInfo
    NAPI-->>JS: Component 对象
```

#### 输入事件流程

```mermaid
sequenceDiagram
    participant JS as 测试脚本
    participant NAPI as uitest_napi.cpp
    participant IPC as ipc_transactor
    participant Server as uitest_server
    participant Input as InputSystem

    JS->>NAPI: Component.click()
    NAPI->>IPC: SendRequest(INJECT_ACTION)
    IPC->>Server: Action JSON
    Server->>Input: InjectEvent(Touch/Key)
    Input-->>Server: EventSent
    Server-->>IPC: Result
    IPC-->>NAPI: ApiReplyInfo
    NAPI-->>JS: void
```

## PerfTest 架构

### 三层架构

```mermaid
graph TB
    subgraph "前端层"
        NAPI["N-API Client\nperftest_client"]
        ANI["ANI Client\nperftest_ani"]
    end

    subgraph "连接层"
        IPC["IPC Transactor\napi_caller_proxy/stub"]
    end

    subgraph "核心层"
        PT["PerfTest\n测试编排"]
        Strategy["PerfTestStrategy\n策略配置"]
        Handler["FrontendApiHandler\nAPI 分发"]
    end

    subgraph "采集层"
        Duration["DurationCollection\n执行时间"]
        CPU["CpuCollection\nCPU指标"]
        Memory["MemoryCollection\n内存指标"]
        AppStart["AppStartTimeCollection\n启动时间"]
        PageSwitch["PageSwitchTimeCollection\n页面切换"]
        FPS["ListSwipeFpsCollection\n帧率"]
    end

    NAPI --> IPC
    ANI --> IPC
    IPC --> PT
    PT --> Strategy
    PT --> Handler
    Handler --> Duration
    Handler --> CPU
    Handler --> Memory
    Handler --> AppStart
    Handler --> PageSwitch
    Handler --> FPS
```

### 目录职责

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `core/` | 测试编排、策略配置、API 处理 | `perf_test.cpp`, `perf_test_strategy.cpp` |
| `collection/` | 数据采集器 | `duration_collection.cpp`, `cpu_collection.cpp` |
| `connection/` | IPC 通信 | `api_caller_client.cpp`, `ipc_transactor.cpp` |
| `napi/` | ArkTS-Dynamic 绑定 | `perftest_napi.cpp` |
| `ani/` | ArkTS-Static 绑定 | `perftest_ani.cpp` |

**证据**: `perftest/BUILD.gn:32-68` 定义了核心 targets

### 测试执行时序

```mermaid
sequenceDiagram
    participant JS as 测试脚本
    participant NAPI as perftest_napi.cpp
    participant IPC as api_caller_client
    participant Server as perftest_server
    participant Coll as DataCollector
    participant JSCode as actionCode

    JS->>NAPI: PerfTest.create(PerfTestStrategy)
    NAPI->>IPC: InitAndConnectPeer
    IPC->>Server: 连接/启动 Daemon
    
    loop N 次迭代
        Server->>NAPI: Execute actionCode
        NAPI->>JSCode: 调用用户 JS 函数
        JSCode-->>NAPI: 返回结果
        NAPI-->>Server: CallbackResult
        
        Server->>Coll: StartCollection
        Coll-->>Server: 开始采集
        
        Server->>Coll: StopCollection
        Coll-->>Server: 采集数据
    end
    
    Server-->>NAPI: 性能数据 JSON
    NAPI-->>JS: PerfTest 对象
    
    JS->>NAPI: getMeasureResult()
    NAPI->>IPC: GetResult
    IPC->>Server: GetMeasureResult
    Server-->>IPC: PerfMeasureResult
    IPC-->>NAPI: Result
    NAPI-->>JS: 统计数据
```

## TestServer SA 架构

### SA 生命周期

```mermaid
stateDiagram-v2
    [*] --> 未启动: 设备启动
    
    未启动 --> OnDemand: 首次客户端连接
    
    OnDemand --> 运行中: CreateSession() 成功
    
    运行中 --> 等待清理: RemoveTestServer()
    
    等待清理 --> 未启动: Caller 计数归零
    
    运行中 --> 未启动: 开发者模式禁用
```

### 权限验证流程

```mermaid
sequenceDiagram
    participant Client as Test Client
    participant SA as TestServer SA
    participant SAMgr as SystemAbilityManager
    
    Client->>SAMgr: GetSystemAbility(5502)
    SAMgr->>SA: OnStart()
    
    Note over SA: IsRootVersion()?\nconst.debuggable
    Note over SA: IsDeveloperMode()?\nconst.security.developermode.state
    
    alt 开发者模式已启用
        SA->>SAMgr: Publish(this)
        SAMgr-->>Client: SA Proxy
        Client->>SA: CreateSession()
    else 开发者模式未启用
        SA-->>Client: 拒绝连接
    end
```

**证据**: `testserver/src/service/test_server_service.cpp:92-104` 实现了权限检查

### IPC 接口定义

| 方法 | 功能 | 权限要求 |
|------|------|----------|
| `CreateSession` | 创建会话，追踪客户端 | 无 |
| `SetPasteData` | 设置剪贴板 | `ARKXTEST_PASTEBOARD_ENABLE` |
| `ChangeWindowMode` | 窗口模式切换 | 无 |
| `TerminateWindow` | 终止窗口 | 无 |
| `MinimizeWindow` | 最小化窗口 | 无 |
| `PublishCommonEvent` | 发布系统事件 | `ohos.permission.PUBLISH_SYSTEM_COMMON_EVENT` |
| `FrequencyLock` | CPU 频率锁定 | `ohos.permission.MANAGE_SECURE_SETTINGS` |
| `CollectProcessMemory` | 采集内存 | 无 |
| `CollectProcessCpu` | 采集 CPU | 无 |
| `SpDaemonProcess` | SmartPerf 控制 | 无 |

**证据**: `testserver/src/ITestServerInterface.idl` 定义了 IPC 接口

## 线程模型

### UiTest 线程模型

```mermaid
graph TB
    subgraph "JS 主线程"
        JST["ArkTS/JS 运行时"]
    end
    
    subgraph "IPC 线程"
        IPC["IPC 异步线程\n(async/await)"]
    end
    
    subgraph "服务端线程"
        ServerMain["主线程\n消息处理"]
        InputThread["输入注入线程"]
    end
    
    subgraph "系统线程"
        A11Y["Accessibility 线程"]
    end
    
    JST --"IPC 调用"--> IPC
    IPC --"CommonEvent"--> ServerMain
    ServerMain --"系统调用"--> A11Y
    ServerMain --"输入注入"--> InputThread
```

### PerfTest 回调线程模型

```mermaid
graph TB
    subgraph "客户端线程"
        ClientMain["JS 主线程"]
    end
    
    subgraph "IPC 通信"
        IPCSync["同步 IPC 调用"]
    end
    
    subgraph "服务端线程"
        ServerThread["Daemon 主线程"]
    end
    
    subgraph "回调执行"
        CallbackThread["threadsafefn\n回调线程"]
    end
    
    ClientMain --"actionCode\n序列化"--> IPCSync
    IPCSync --> ServerThread
    ServerThread --"threadsafefn call"--> CallbackThread
    CallbackThread --"执行 JS 函数"--> ClientMain
```

**证据**: `perftest/napi/src/callback_code_napi.cpp` 使用 `napi_call_threadsafe_function`

## 数据流

### UiTest 数据流

```
测试脚本 → N-API → IPC → 服务端 → Accessibility/Input → 系统 → 返回结果
```

| 阶段 | 数据格式 | 说明 |
|------|----------|------|
| JS → N-API | JS Object | 方法参数 |
| N-API → IPC | JSON | ApiCallInfo 序列化 |
| IPC 传输 | Parcel | IPC 消息 |
| IPC → 服务端 | JSON | ApiReplyInfo 反序列化 |
| 服务端 → 系统 | 系统 API 调用 | 无 |
| 返回 | 逆向传递 | 同样的路径返回 |

**证据**: `uitest/connection/include/ipc_transactor.h` 定义了 `ApiCallInfo` 和 `ApiReplyInfo`

### PerfTest 性能数据流

```
测试策略 → IPC → 服务端 → 数据采集器 → 统计分析 → 返回
```

| 阶段 | 数据内容 | 说明 |
|------|----------|------|
| PerfTestStrategy | 配置信息 | 指标类型、迭代次数、超时等 |
| actionCode | JS 函数引用 | 用于回调执行 |
| 采集数据 | 原始性能数据 | 时间戳、数值 |
| 统计结果 | 最大/最小/平均值 | PerfMeasureResult |
