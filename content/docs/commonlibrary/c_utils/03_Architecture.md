# 架构说明

## 目的

本文档描述 c_utils 的整体架构设计，包括组件关系、数据流、线程模型和关键时序。

## 适用范围

- 架构师评估技术方案
- 开发者理解内部机制
- 代码审查时把握全局

---

## 组件架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              c_utils 组件架构                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         对外接口层 (Public API)                       │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │   │
│  │  │refbase.h │ │parcel.h  │ │file_ex.h │ │safe_map.h│ │timer.h   │  │   │
│  │  │unique_fd │ │          │ │directory │ │thread_   │ │observer.h│  │   │
│  │  │flat_obj  │ │          │ │_ex.h     │ │pool.h    │ │          │  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         核心实现层 (Core Implementation)              │   │
│  │                                                                     │   │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │   │
│  │   │  内存管理    │    │   序列化     │    │   并发控制   │            │   │
│  │   │  RefBase    │◄──►│   Parcel    │    │ ThreadPool  │            │   │
│  │   │  RefCounter │    │  Allocator  │    │  RWLock     │            │   │
│  │   │  sptr/wptr  │    │             │    │  Semaphore  │            │   │
│  │   └─────────────┘    └─────────────┘    └─────────────┘            │   │
│  │                                                                     │   │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │   │
│  │   │  文件系统    │    │   事件系统   │    │   容器      │            │   │
│  │   │  File/Direct│    │   Timer     │    │  SafeMap    │            │   │
│  │   │  MappedFile │    │ IOEventReac │    │  SafeQueue  │            │   │
│  │   │   Ashmem    │    │    tor      │    │SortedVector │            │   │
│  │   └─────────────┘    └─────────────┘    └─────────────┘            │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         平台适配层 (Platform Abstraction)             │   │
│  │                                                                     │   │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│  │   │  Linux   │  │  OHOS    │  │ Windows  │  │   iOS    │           │   │
│  │   │  (完整)   │  │  (完整)   │  │ (受限)   │  │  (受限)   │           │   │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘           │   │
│  │                                                                     │   │
│  │   平台宏定义: OHOS_PLATFORM, WINDOWS_PLATFORM, IOS_PLATFORM...       │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         外部依赖层 (External Dependencies)            │   │
│  │                                                                     │   │
│  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │   │
│  │   │bounds_checking│  │    hilog     │  │  rust_cxx    │             │   │
│  │   │  _function    │  │   (日志)      │  │  (Rust FFI)  │             │   │
│  │   │  (安全C库)    │  │              │  │              │             │   │
│  │   └──────────────┘  └──────────────┘  └──────────────┘             │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 关键组件关系

### 1. 内存管理与序列化

```mermaid
classDiagram
    class RefBase {
        +IncStrong()
        +DecStrong()
        +IncWeak()
        +DecWeak()
    }
    
    class RefCounter {
        +strongRefs
        +weakRefs
        +flags
    }
    
    class sptr~T~ {
        +operator->()
        +operator*()
    }
    
    class wptr~T~ {
        +AttemptIncStrong()
        +promote()
    }
    
    class Parcelable {
        <<abstract>>
        +Marshalling()
        +Unmarshalling()
    }
    
    class Parcel {
        +WriteInt32()
        +ReadInt32()
        +WriteParcelable()
        +ReadParcelable()
    }
    
    RefBase --> RefCounter : uses
    sptr~T~ --> RefBase : manages
    wptr~T~ --> RefBase : manages
    Parcelable --|> RefBase : extends
    Parcel --> Parcelable : serializes
```

### 2. 事件系统架构

```mermaid
classDiagram
    class EventReactor {
        +SetUp()
        +RunLoop()
        +CleanUp()
        +SwitchOn()
        +SwitchOff()
    }
    
    class EventHandler {
        +HandleEvent()
        +SetReactor()
    }
    
    class EventDemultiplexer {
        +Poll()
        +Register()
        +Unregister()
    }
    
    class IOEventReactor {
        +ScheduleTimer()
        +CancelTimer()
    }
    
    class IOEventHandler {
        +HandleReadEvent()
        +HandleWriteEvent()
    }
    
    class Timer {
        +Register()
        +Unregister()
        +Setup()
        +Shutdown()
    }
    
    class TimerEventHandler {
        +HandleTimeout()
    }
    
    EventReactor --> EventDemultiplexer : uses
    EventReactor --> EventHandler : manages
    IOEventReactor --|> EventReactor : extends
    IOEventHandler --|> EventHandler : extends
    Timer --> IOEventReactor : uses
    TimerEventHandler --|> IOEventHandler : extends
```

---

## 数据流

### 1. Parcel 序列化数据流

```
┌──────────────┐     WriteInt32()      ┌──────────────┐
│   业务对象    │ ─────────────────────►│              │
│  (int32_t)   │                       │   Parcel     │
└──────────────┘                       │   Buffer     │
                                       │              │
┌──────────────┐     WriteString()     │  ┌────────┐  │
│   业务对象    │ ─────────────────────►│  │ Header │  │
│  (string)    │                       │  ├────────┤  │
└──────────────┘                       │  │ Data   │  │
                                       │  │ Section│  │
┌──────────────┐  WriteParcelable()    │  ├────────┤  │
│  Parcelable  │ ─────────────────────►│  │ Object │  │
│    对象      │                       │  │ Section│  │
└──────────────┘                       │  └────────┘  │
                                       └──────────────┘
                                                │
                                                │ IPC/RPC
                                                ▼
                                       ┌──────────────┐
                                       │   对端进程    │
                                       │  ReadXXX()   │
                                       └──────────────┘
```

### 2. 线程池任务流

```
┌──────────────┐
│   提交任务    │
│  AddTask()   │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│   任务队列    │◄────│   工作线程    │
│  (deque)     │     │  WorkInThread│
└──────┬───────┘     └──────────────┘
       │                     ▲
       │  ScheduleTask()     │
       └─────────────────────┘
              获取并执行任务
```

### 3. 定时器事件流

```
┌──────────────┐
│  Register()  │
│  注册定时器   │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  timerfd_    │────►│  epoll_wait  │
│  (timer fd)  │     │  等待事件    │
└──────────────┘     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  超时事件     │
                     │ OnTimer(fd)  │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  执行回调     │
                     │   callback   │
                     └──────────────┘
```

---

## 线程模型

### 1. ThreadPool 线程模型

```
主线程                    ThreadPool                    工作线程
  │                          │                             │
  │──► Start(numThreads) ───►│                             │
  │                          │──► 创建 numThreads 个线程 ──►│
  │                          │                             │
  │──► AddTask(task) ───────►│                             │
  │                          │──► 加入任务队列 ────────────►│
  │                          │                             │──► ScheduleTask()
  │                          │◄────────────────────────────│
  │                          │                             │──► 执行任务
  │                          │                             │
  │──► Stop() ──────────────►│                             │
  │                          │──► running_=false ─────────►│
  │                          │◄────────────────────────────│
  │                          │（等待所有线程join）           │
```

**关键实现** (`base/src/thread_pool.cpp:34-56`):
- 使用 `std::thread` 创建工作线程
- 使用 `std::mutex` + `std::condition_variable` 同步
- 任务队列使用 `std::deque<Task>`
- 支持任务数量限制（背压机制）

### 2. Timer 线程模型

```
主线程                      Timer                        定时器线程
  │                          │                              │
  │──► Setup() ─────────────►│                              │
  │                          │──► 创建定时器线程 ───────────►│
  │                          │                              │──► MainLoop()
  │                          │                              │    ├──► reactor_->SetUp()
  │                          │                              │    └──► reactor_->RunLoop()
  │                          │                              │         └──► epoll_wait()
  │──► Register(callback) ──►│                              │
  │                          │──► 创建 timerfd ─────────────►│──► 超时触发
  │                          │                              │    └──► callback()
  │                          │                              │
  │──► Unregister(timerId) ─►│                              │
  │                          │──► 关闭 timerfd ─────────────►│
  │                          │                              │
  │──► Shutdown() ──────────►│                              │
  │                          │──► reactor_->SwitchOff() ────►│──► 退出循环
  │                          │◄─────────────────────────────│（join或detach）
```

**关键实现** (`base/src/timer.cpp:36-78`):
- 独立线程运行事件循环
- 使用 `timerfd` + `epoll` 实现高精度定时
- 支持单次和周期定时器
- 线程名通过 `prctl(PR_SET_NAME)` 设置

### 3. IOEventReactor 线程模型

```
┌─────────────────────────────────────────────────────────────┐
│                    IOEventReactor 线程模型                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   单线程事件循环                                              │
│   ┌─────────────────────────────────────────────────────┐  │
│   │                  Event Loop                         │  │
│   │  ┌─────────────┐                                    │  │
│   │  │  epoll_wait │◄─────────────────────────────┐    │  │
│   │  └──────┬──────┘                              │    │  │
│   │         │ 有事件                               │    │  │
│   │         ▼                                     │    │  │
│   │  ┌─────────────┐     ┌─────────────┐         │    │  │
│   │  │ 查找Handler │────►│ HandleEvent │         │    │  │
│   │  └─────────────┘     └──────┬──────┘         │    │  │
│   │                             │                │    │  │
│   │                             ▼                │    │  │
│   │  ┌─────────────────────────────────────┐    │    │  │
│   │  │          业务回调处理                │    │    │  │
│   │  └─────────────────────────────────────┘    │    │  │
│   │                             │                │    │  │
│   │                             └────────────────┘    │  │
│   │                                                   │  │
│   └───────────────────────────────────────────────────┘  │
│                                                             │
│   事件类型：                                                 │
│   - 读事件 (EPOLLIN)                                        │
│   - 写事件 (EPOLLOUT)                                       │
│   - 定时器事件 (timerfd)                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 关键时序

### 1. RefBase 生命周期时序

```mermaid
sequenceDiagram
    participant Client as 客户端代码
    participant sptr as sptr<T>
    participant RefBase as RefBase对象
    participant Counter as RefCounter

    Client->>RefBase: new T()
    Client->>sptr: sptr<T>(obj)
    sptr->>RefBase: IncStrong()
    RefBase->>Counter: incStrong()
    
    Note over Client,Counter: 对象使用中...
    
    Client->>sptr: sptr = nullptr
    sptr->>RefBase: DecStrong()
    RefBase->>Counter: decStrong()
    
    alt 强引用计数 == 0
        RefBase->>RefBase: OnLastStrongRef()
        RefBase->>RefBase: delete this
    end
```

### 2. Parcel 写入/读取时序

```mermaid
sequenceDiagram
    participant App as 应用程序
    participant Parcel as Parcel
    participant Allocator as Allocator

    App->>Parcel: WriteInt32(val)
    Parcel->>Allocator: ReallocIfNeeded()
    Parcel->>Parcel: memcpy(data, &val, sizeof(int32_t))
    
    App->>Parcel: WriteString(str)
    Parcel->>Parcel: WriteInt32(str.length())
    Parcel->>Parcel: WriteData(str.c_str(), str.length())
    
    Note over App,Allocator: 数据传输...
    
    App->>Parcel: ReadInt32()
    Parcel->>Parcel: memcpy(&val, data, sizeof(int32_t))
    Parcel-->>App: return val
    
    App->>Parcel: ReadString()
    Parcel->>Parcel: ReadInt32(&len)
    Parcel->>Parcel: 读取len字节
    Parcel-->>App: return string
```

### 3. ThreadPool 启动/停止时序

```mermaid
sequenceDiagram
    participant Main as 主线程
    participant Pool as ThreadPool
    participant Thread1 as 工作线程1
    participant ThreadN as 工作线程N

    Main->>Pool: Start(4)
    loop 创建4个工作线程
        Pool->>Pool: std::thread(&WorkInThread)
        Pool->>Thread1: 启动
    end
    Pool-->>Main: return ERR_OK
    
    Main->>Pool: AddTask(task)
    Pool->>Pool: tasks_.push_back(task)
    Pool->>Pool: hasTaskToDo_.notify_one()
    Pool-->>Thread1: 唤醒
    Thread1->>Pool: ScheduleTask()
    Pool-->>Thread1: return task
    Thread1->>Thread1: task()
    
    Main->>Pool: Stop()
    Pool->>Pool: running_ = false
    Pool->>Pool: hasTaskToDo_.notify_all()
    
    loop 等待所有线程
        Pool->>Thread1: join()
        Pool->>ThreadN: join()
    end
```

---

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           模块依赖关系                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   errors (基础)                                                             │
│     │                                                                       │
│     ├──► refbase ────┬──► parcel ◄──┬──► file_ex                            │
│     │                │              │                                       │
│     │                └──► unique_fd │                                       │
│     │                               │                                       │
│     ├──► safe_map ◄─────────────────┤                                       │
│     │                               │                                       │
│     ├──► safe_queue                 │                                       │
│     │                               │                                       │
│     ├──► thread_pool ◄──────────────┤                                       │
│     │                               │                                       │
│     ├──► timer ◄────────────────────┘                                       │
│     │                                                                       │
│     └──► observer                                                           │
│                                                                             │
│   外部依赖:                                                                  │
│   - bounds_checking_function (所有模块)                                      │
│   - hilog (非Android/iOS平台)                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织
- [内部 API](05_Inner_API.md) - 模块接口详情
- [关键调用链](appendix/Callgraphs.md) - 详细调用关系
