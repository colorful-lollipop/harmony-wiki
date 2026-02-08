# 内部 API

## 目的

本文档描述 c_utils 内部模块的接口设计、依赖关系和实现细节。

## 适用范围

- 需要深入理解内部机制的开发者
- 进行代码审查或重构的工程师

---

## 模块划分

### 1. 内存管理模块 (refbase)

**文件**: `base/src/refbase.cpp` (~20KB)

#### 内部类设计

```cpp
// 引用计数器（内部实现）
class RefCounter {
    std::atomic<int> atomicStrong_;      // 强引用计数
    std::atomic<int> atomicWeak_;        // 弱引用计数  
    std::atomic<int> atomicRefCount_;    // RefCounter自身引用计数
    RefPtrCallback callback_;            // 销毁回调
    
    // 调试相关
    #ifdef DEBUG_REFBASE
    std::mutex trackerMutex;
    RefTracker* refTracker;
    unsigned int domainId_;
    bool enableTrack;
    #endif
};

// 弱引用计数器
class WeakRefCounter {
    std::atomic<int> atomicWeak_;
    RefCounter* refCounter_;
    void* cookie_;
};
```

#### 关键算法

**强引用计数管理** (`refbase.cpp:200-250`):
```cpp
void RefCounter::IncStrongRef(const void* objectId) {
    // 增加强引用计数
    atomicStrong_.fetch_add(1, std::memory_order_relaxed);
    
    // 首次强引用时处理
    if (atomicStrong_.load(std::memory_order_relaxed) == INITIAL_PRIMARY_VALUE + 1) {
        // 从弱引用提升或首次创建
        atomicStrong_.fetch_sub(INITIAL_PRIMARY_VALUE, std::memory_order_release);
    }
}

bool RefCounter::DecStrongRef(const void* objectId) {
    // 减少强引用计数
    if (atomicStrong_.fetch_sub(1, std::memory_order_release) == 1) {
        // 最后一个强引用，销毁对象
        std::atomic_thread_fence(std::memory_order_acquire);
        if (callback_ != nullptr) {
            callback_();
        }
        return true;
    }
    return false;
}
```

**弱引用提升** (`refbase.cpp:300-350`):
```cpp
bool RefCounter::AttemptIncStrong(const void* objectId, int& outCount) {
    // 尝试从弱引用提升为强引用
    int curCount = atomicStrong_.load(std::memory_order_relaxed);
    
    while (curCount > 0) {
        // CAS操作尝试增加强引用
        if (atomicStrong_.compare_exchange_weak(
            curCount, curCount + 1,
            std::memory_order_relaxed)) {
            return true;
        }
    }
    return false;  // 对象已销毁
}
```

#### 线程安全

- 所有计数器使用 `std::atomic` 保证原子性
- 内存序使用 `std::memory_order_relaxed`（性能优化）
- 销毁时使用 `std::atomic_thread_fence` 保证同步

---

### 2. 序列化模块 (parcel)

**文件**: `base/src/parcel.cpp` (~42KB)

#### 内部数据结构

```cpp
class Parcel {
    Allocator* allocator_;          // 内存分配器
    
    // 数据缓冲区
    uint8_t* data_;                 // 数据指针
    size_t dataSize_;               // 数据大小
    size_t dataCapacity_;           // 容量
    size_t maxDataCapacity_;        // 最大容量（默认200KB）
    
    // 读写游标
    size_t writeCursor_;            // 写入位置
    size_t readCursor_;             // 读取位置
    
    // 对象偏移表（用于IPC）
    binder_size_t* objectOffsets_;  // Binder对象偏移数组
    size_t nextObjectIdx_;          // 下一个对象索引
    size_t objectCursor_;           // 对象游标
    size_t objectsCapacity_;        // 对象容量
    
    bool writable_;                 // 是否可写
};
```

#### 内存分配策略

**容量计算** (`parcel.cpp:108-142`):
```cpp
size_t CalcNewCapacity(size_t minNewCapacity) {
    const size_t CAPACITY_THRESHOLD = 4096;  // 4KB阈值
    
    if (minNewCapacity <= CAPACITY_THRESHOLD) {
        // 小于4KB：按2的幂次增长（64, 128, 256, ...）
        size_t newCapacity = 64;
        while (newCapacity < minNewCapacity) {
            newCapacity *= 2;
        }
        return newCapacity;
    } else {
        // 大于4KB：按4KB步长增长
        size_t newCapacity = (minNewCapacity / CAPACITY_THRESHOLD) * CAPACITY_THRESHOLD;
        newCapacity += CAPACITY_THRESHOLD;
        return newCapacity;
    }
}
```

#### ARM32 对齐保护

**代码** (`parcel.cpp:22-39`):
```cpp
#if defined(__arm__) && !defined(__aarch64__)
static const size_t ARM32_ADDR_ALIGN = 4;

static inline bool IsAligned(const void* p) {
    uintptr_t ptr = reinterpret_cast<uintptr_t>(p);
    return (ptr & (ARM32_ADDR_ALIGN - 1U)) == 0U;
}

#define RETURN_IF_NOTALIGNED_ON_ARM32(ptr, SZ)                          \
    do { if ((SZ) > 1U && !IsAligned((ptr))) {                          \
        return false;                                                   \
    } } while (0)
#endif
```

#### Binder 对象处理

```cpp
// Binder类型标识
static const int BINDER_TYPE_HANDLE = 0x73682a85;  // 远程对象句柄
static const int BINDER_TYPE_FD = 0x66642a85;      // 文件描述符

// 对象偏移表用于IPC时内核转换对象引用
```

---

### 3. 线程池模块 (thread_pool)

**文件**: `base/src/thread_pool.cpp` (~3KB)

#### 内部实现

```cpp
class ThreadPool {
    std::string myName_;                    // 线程池名称
    std::vector<std::thread> threads_;       // 工作线程数组
    std::deque<Task> tasks_;                // 任务队列
    std::mutex mutex_;                      // 保护任务队列
    std::condition_variable hasTaskToDo_;   // 有任务可执行
    std::condition_variable acceptNewTask_; // 可接受新任务
    
    size_t maxTaskNum_;                     // 最大任务数（背压）
    std::atomic<bool> running_;             // 运行状态
};
```

#### 任务调度流程

**添加任务** (`thread_pool.cpp:71-84`):
```cpp
void ThreadPool::AddTask(const Task& f) {
    if (threads_.empty()) {
        // 线程池未启动，直接执行
        f();
    } else {
        std::unique_lock<std::mutex> lock(mutex_);
        
        // 背压：任务队列满时等待
        while (Overloaded()) {
            acceptNewTask_.wait(lock);
        }
        
        tasks_.push_back(f);
        hasTaskToDo_.notify_one();  // 唤醒一个工作线程
    }
}
```

**工作线程主循环** (`thread_pool.cpp:117-125`):
```cpp
void ThreadPool::WorkInThread() {
    while (running_) {
        Task task = ScheduleTask();  // 阻塞获取任务
        if (task) {
            task();  // 执行任务
        }
    }
}
```

#### 背压机制

```cpp
bool ThreadPool::Overloaded() const {
    return (maxTaskNum_ > 0) && (tasks_.size() >= maxTaskNum_);
}
```

- 当 `maxTaskNum_` 设置且任务队列满时，AddTask 会阻塞等待
- 任务执行完成后通知可接受新任务

---

### 4. 定时器模块 (timer)

**文件**: `base/src/timer.cpp` (~8KB)

#### 内部数据结构

```cpp
class Timer {
    std::string name_;                      // 定时器名称
    int timeoutMs_;                         // epoll超时时间
    EventReactor* reactor_;                 // 事件反应器
    std::thread thread_;                    // 定时器线程
    std::mutex mutex_;                      // 保护数据结构
    
    // 定时器条目
    struct TimerEntry {
        uint32_t timerId;                   // 定时器ID
        uint32_t interval;                  // 间隔(ms)
        TimerCallback callback;             // 回调函数
        bool once;                          // 是否单次
        int timerFd;                        // timerfd
    };
    
    // 索引结构
    std::map<uint32_t, TimerEntryPtr> timerToEntries_;      // ID -> Entry
    std::map<uint32_t, TimerEntryList> intervalToTimers_;   // interval -> Entries
    std::map<int, uint32_t> timers_;                       // fd -> interval
};
```

#### 定时器复用机制

**相同间隔复用** (`timer.cpp:223-235`):
```cpp
int Timer::GetTimerFd(uint32_t interval) {
    // 检查是否已有相同间隔的timerfd
    if (intervalToTimers_.find(interval) == intervalToTimers_.end()) {
        return INVALID_TIMER_FD;
    }
    
    auto& entryList = intervalToTimers_[interval];
    for (const TimerEntryPtr& ptr : entryList) {
        if (!ptr->once) {
            // 找到非单次定时器，复用其timerfd
            return ptr->timerFd;
        }
    }
    return INVALID_TIMER_FD;
}
```

#### 线程安全设计

- 主线程操作（Register/Unregister）加锁保护
- 定时器线程回调时检查 `reactor_->IsLoopReady()`
- 单次定时器执行后自动清理

---

### 5. 事件系统模块

#### IOEventReactor

**文件**: `base/src/io_event_reactor.cpp` (~11KB)

```cpp
class IOEventReactor {
    int epollFd_;                           // epoll文件描述符
    std::map<int, IOEventHandler*> handlers_; // fd -> handler
    std::atomic<bool> switchedOn_;          // 开关状态
    std::atomic<bool> loopReady_;           // 循环就绪
    
    // 管道用于唤醒
    int wakeFd_[2];
};
```

#### 事件循环

```cpp
uint32_t IOEventReactor::RunLoop(int timeoutMs) {
    struct epoll_event events[MAX_EVENTS];
    
    while (switchedOn_) {
        int nfds = epoll_wait(epollFd_, events, MAX_EVENTS, timeoutMs);
        
        for (int i = 0; i < nfds; i++) {
            int fd = events[i].data.fd;
            auto it = handlers_.find(fd);
            if (it != handlers_.end()) {
                it->second->HandleEvent(events[i].events);
            }
        }
    }
}
```

---

### 6. 文件系统模块

#### 文件操作

**文件**: `base/src/file_ex.cpp` (~11KB)

**关键实现细节**:

```cpp
// 最大文件大小限制：32MB
static const size_t MAX_FILE_SIZE = 32 * 1024 * 1024;

bool LoadStringFromFile(const std::string& filePath, std::string& content) {
    // 1. 打开文件（O_RDONLY | O_CLOEXEC）
    unique_fd fd(open(filePath.c_str(), O_RDONLY | O_CLOEXEC));
    if (fd < 0) return false;
    
    // 2. 获取文件大小
    struct stat st;
    if (fstat(fd, &st) != 0) return false;
    
    // 3. 检查大小限制
    if (st.st_size > MAX_FILE_SIZE) return false;
    
    // 4. 读取内容
    content.resize(st.st_size);
    ssize_t n = read(fd, &content[0], st.st_size);
    return n == st.st_size;
}
```

#### 目录操作

**文件**: `base/src/directory_ex.cpp` (~16KB)

**递归遍历实现**:
```cpp
void GetDirFiles(const std::string& path, std::vector<std::string>& files) {
    DIR* dir = opendir(path.c_str());
    if (dir == nullptr) return;
    
    struct dirent* entry;
    while ((entry = readdir(dir)) != nullptr) {
        // 跳过 . 和 ..
        if (strcmp(entry->d_name, ".") == 0 || 
            strcmp(entry->d_name, "..") == 0) {
            continue;
        }
        
        std::string fullPath = path + "/" + entry->d_name;
        
        if (entry->d_type == DT_DIR) {
            // 递归处理子目录
            GetDirFiles(fullPath, files);
        } else {
            files.push_back(fullPath);
        }
    }
    closedir(dir);
}
```

#### 内存映射文件

**文件**: `base/src/mapped_file.cpp` (~18KB)

```cpp
class MappedFile {
    void* data_;
    size_t size_;
    int fd_;
    int flags_;
    
public:
    bool Open(const std::string& fileName, int flags) {
        fd_ = open(fileName.c_str(), flags);
        if (fd_ < 0) return false;
        
        struct stat st;
        if (fstat(fd_, &st) != 0) {
            close(fd_);
            return false;
        }
        
        size_ = st.st_size;
        int prot = (flags == O_RDONLY) ? PROT_READ : (PROT_READ | PROT_WRITE);
        data_ = mmap(nullptr, size_, prot, MAP_SHARED, fd_, 0);
        
        return data_ != MAP_FAILED;
    }
};
```

---

## 模块依赖关系

```
                    ┌──────────────┐
                    │    errors    │
                    └──────┬───────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   refbase    │◄──│    parcel    │   │  safe_map    │
└──────┬───────┘   └──────┬───────┘   └──────────────┘
       │                  │
       │                  ▼
       │           ┌──────────────┐
       │           │ unique_fd    │
       │           └──────────────┘
       │
       ▼
┌──────────────┐
│  thread_pool │
└──────────────┘
       │
       ▼
┌──────────────┐
│    timer     │◄── io_event_reactor
└──────────────┘
       │
       ▼
┌──────────────┐
│   ashmem     │
└──────────────┘
```

---

## 可替换点

| 模块 | 可替换点 | 说明 |
|------|----------|------|
| Parcel | Allocator | 自定义内存分配器 |
| ThreadPool | Task | 自定义任务类型 |
| Timer | EventReactor | 可替换事件后端 |
| IOEventReactor | epoll | 可替换为其他多路复用 |

---

## 相关跳转

- [架构说明](03_Architecture.md) - 整体架构设计
- [对外 API](04_Public_API.md) - 对外暴露接口
- [关键调用链](appendix/Callgraphs.md) - 详细调用关系
