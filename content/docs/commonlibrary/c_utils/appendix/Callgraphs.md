# 关键调用链

## 目的

本文档记录 c_utils 关键功能的调用链，帮助理解代码执行流程。

---

## 1. RefBase 生命周期

### 创建与首次引用

```
new MyClass()  (继承 RefBase)
  │
  ├──► RefBase::RefBase()
  │       │
  │       ├──► new RefCounter()
  │       │       ├──► atomicStrong_ = INITIAL_PRIMARY_VALUE (1<<28)
  │       │       ├──► atomicWeak_ = 0
  │       │       └──► atomicRefCount_ = 1
  │       │
  │       └──► refCounter_ = RefCounter
  │
  └──► 返回对象指针

sptr<MyClass> obj = ptr
  │
  └──► sptr::sptr(T* other)
          │
          └──► RefBase::IncStrongRef(this)
                  │
                  └──► RefCounter::IncStrongRefCount()
                          │
                          └──► atomicStrong_.fetch_add(1)
                                  │
                                  └──► 值变为 INITIAL_PRIMARY_VALUE + 1
                                          │
                                          └──► 首次引用，减去 INITIAL_PRIMARY_VALUE
                                                  │
                                                  └──► 最终强引用计数 = 1
```

### 引用计数增加

```
sptr<MyClass> obj2 = obj
  │
  └──► sptr::sptr(const sptr& other)
          │
          ├──► other.m_ptr->IncStrongRef(this)
          │       │
          │       └──► RefCounter::IncStrongRefCount()
          │               └──► atomicStrong_.fetch_add(1)  // 计数+1
          │
          └──► m_ptr = other.m_ptr
```

### 引用计数减少（释放）

```
obj = nullptr  (或 obj 离开作用域)
  │
  └──► sptr::~sptr()
          │
          └──► RefBase::DecStrongRef(this)
                  │
                  └──► RefCounter::DecStrongRefCount()
                          │
                          ├──► atomicStrong_.fetch_sub(1)
                          │       │
                          │       └──► 返回旧值
                          │
                          └──► 如果旧值 == 1 (最后一个引用)
                                  │
                                  ├──► callback_()  // 销毁对象
                                  │       │
                                  │       └──► RefBase::~RefBase()
                                  │               │
                                  │               └──► delete refCounter_
                                  │                       │
                                  │                       └──► RefCounter::~RefCounter()
                                  │                               └──► 如果弱引用也为0，销毁计数器
                                  │
                                  └──► delete this
```

### 弱引用提升

```
wptr<MyClass> weak = obj;
sptr<MyClass> strong = weak.promote();
  │
  └──► wptr::promote()
          │
          └──► WeakRefCounter::AttemptIncStrongRef()
                  │
                  └──► RefCounter::AttemptIncStrong(objectId, outCount)
                          │
                          ├──► 加载当前强引用计数 curCount
                          │
                          ├──► while (curCount > 0)
                          │       │
                          │       └──► CAS(curCount, curCount + 1)
                          │               │
                          │               ├──► 成功: 返回 true (提升成功)
                          │               └──► 失败: 重试 (其他线程修改了计数)
                          │
                          └──► 如果 curCount == 0
                                  │
                                  └──► 返回 false (对象已销毁)
```

---

## 2. Parcel 序列化

### 写入整数

```
Parcel::WriteInt32(value)
  │
  ├──► EnsureWritableCapacity(sizeof(int32_t))
  │       │
  │       ├──► GetWritableBytes() < desireCapacity ?
  │       │       │
  │       │       └──► CalcNewCapacity(dataSize_ + desireCapacity)
  │       │               │
  │       │               ├──► 如果 <= 4KB: 按2的幂次增长
  │       │               └──► 如果 > 4KB: 按4KB步长增长
  │       │
  │       └──► allocator_>Reallocate(data_, newCapacity)
  │
  ├──► WriteDataInplace(&value, sizeof(int32_t))
  │       │
  │       ├──► memcpy(data_ + writeCursor_, &value, sizeof(int32_t))
  │       │
  │       ├──► writeCursor_ += sizeof(int32_t)
  │       │
  │       └──► dataSize_ = max(dataSize_, writeCursor_)
  │
  └──► return true
```

### 写入字符串

```
Parcel::WriteString(str)
  │
  ├──► WriteInt32(str.length())  // 先写长度
  │       └──► [见写入整数流程]
  │
  ├──► EnsureWritableCapacity(str.length())
  │       └──► [见容量检查流程]
  │
  ├──► WriteDataInplace(str.c_str(), str.length())
  │       └──► memcpy
  │
  └──► return true
```

### 读取整数

```
Parcel::ReadInt32()
  │
  ├──► CheckReadable(sizeof(int32_t))
  │       │
  │       └──► GetReadableBytes() >= sizeof(int32_t) ?
  │
  ├──► RETURN_IF_NOTALIGNED_ON_ARM32(data_ + readCursor_, sizeof(int32_t))
  │       └──► ARM32平台检查4字节对齐
  │
  ├──► memcpy(&value, data_ + readCursor_, sizeof(int32_t))
  │
  ├──► readCursor_ += sizeof(int32_t)
  │
  └──► return value
```

### 写入 Parcelable 对象

```
Parcel::WriteParcelable(parcelable)
  │
  ├──► parcelable->Marshalling(*this)
  │       │
  │       └──► [用户自定义序列化逻辑]
  │               ├──► WriteInt32(field1)
  │               ├──► WriteString(field2)
  │               └──► ...
  │
  └──► return true
```

---

## 3. ThreadPool 任务处理

### 启动线程池

```
ThreadPool::Start(numThreads)
  │
  ├──► 检查 threads_.empty() (防止重复启动)
  │
  ├──► running_ = true
  │
  ├──► threads_.reserve(numThreads)
  │
  └──► for i in 0..numThreads-1:
          │
          ├──► std::thread t([this] { WorkInThread(); })
          │
          ├──► pthread_setname_np(t.native_handle(), name_ + to_string(i))
          │
          └──► threads_.push_back(move(t))
```

### 添加任务

```
ThreadPool::AddTask(task)
  │
  ├──► 如果 threads_.empty()
  │       │
  │       └──► task()  // 直接执行（线程池未启动）
  │
  └──► 否则
          │
          ├──► lock(mutex_)
          │
          ├──► while (Overloaded())  // 背压检查
          │       │
          │       └──► acceptNewTask_.wait(lock)
          │
          ├──► tasks_.push_back(task)
          │
          ├──► unlock(mutex_)
          │
          └──► hasTaskToDo_.notify_one()  // 唤醒一个工作线程
```

### 工作线程主循环

```
WorkInThread()
  │
  └──► while (running_)
          │
          ├──► task = ScheduleTask()
          │       │
          │       ├──► lock(mutex_)
          │       │
          │       ├──► while (tasks_.empty() && running_)
          │       │       │
          │       │       └──► hasTaskToDo_.wait(lock)
          │       │
          │       ├──► task = tasks_.front()
          │       │
          │       ├──► tasks_.pop_front()
          │       │
          │       ├──► if (maxTaskNum_ > 0)
          │       │       │
          │       │       └──► acceptNewTask_.notify_one()
          │       │
          │       └──► unlock(mutex_)
          │
          ├──► if (task)
          │       │
          │       └──► task()  // 执行用户任务
          │
          └──► [循环继续]
```

### 停止线程池

```
ThreadPool::Stop()
  │
  ├──► lock(mutex_)
  │
  ├──► running_ = false
  │
  ├──► hasTaskToDo_.notify_all()  // 唤醒所有线程
  │
  ├──► unlock(mutex_)
  │
  └──► for each thread in threads_
          │
          └──► thread.join()  // 等待线程结束
```

---

## 4. Timer 定时器

### 注册定时器

```
Timer::Register(callback, interval, once)
  │
  ├──► lock(mutex_)
  │
  ├──► timerFd = GetTimerFd(interval)  // 检查是否可复用
  │       │
  │       └──► 遍历 intervalToTimers_[interval]
  │               │
  │               └──► 找到非单次定时器，返回其 timerFd
  │
  ├──► 如果 timerFd == INVALID_TIMER_FD
  │       │
  │       └──► DoRegister(callback, interval, once, timerFd)
  │               │
  │               └──► reactor_->ScheduleTimer(cb, interval, timerFd, once)
  │                       │
  │                       ├──► timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK)
  │                       │
  │                       ├──► timerfd_settime(timerFd, 0, &its, nullptr)
  │                       │
  │                       └──► epoll_ctl(epollFd_, EPOLL_CTL_ADD, timerFd, ...)
  │
  ├──► 生成 timerId
  │
  ├──► 创建 TimerEntry
  │
  ├──► intervalToTimers_[interval].push_back(entry)
  │
  ├──► timerToEntries_[timerId] = entry
  │
  ├──► unlock(mutex_)
  │
  └──► return timerId
```

### 定时器触发

```
[定时器线程] MainLoop()
  │
  ├──► reactor_->SetUp()
  │       │
  │       └──► epoll_create1(EPOLL_CLOEXEC)
  │
  ├──► reactor_->RunLoop(timeoutMs)
  │       │
  │       └──► while (switchedOn_)
  │               │
  │               ├──► epoll_wait(epollFd_, events, maxEvents, timeoutMs)
  │               │
  │               └──► for each event
  │                       │
  │                       ├──► handler = handlers_[event.data.fd]
  │                       │
  │                       └──► handler->HandleEvent(event.events)
  │                               │
  │                               └──► OnTimer(fd)
  │                                       │
  │                                       ├──► lock(mutex_)
  │                                       │
  │                                       ├──► entryList = intervalToTimers_[interval]
  │                                       │
  │                                       ├──► unlock(mutex_)
  │                                       │
  │                                       └──► for each entry in entryList
  │                                               │
  │                                               ├──► if (entry.timerFd == fd)
  │                                               │       │
  │                                               │       └──► entry.callback()
  │                                               │
  │                                               └──► if (entry.once)
  │                                                       │
  │                                                       └──► 标记为待清理
  │
  └──► reactor_->CleanUp()
```

---

## 5. 文件操作

### 读取文件到字符串

```
LoadStringFromFile(filePath, content)
  │
  ├──► unique_fd fd(open(filePath.c_str(), O_RDONLY | O_CLOEXEC))
  │       │
  │       └──► 如果 fd < 0，返回 false
  │
  ├──► fstat(fd, &st)
  │
  ├──► 如果 st.st_size > MAX_FILE_LENGTH (32MB)
  │       │
  │       └──► 返回 false
  │
  ├──► content.resize(st.st_size)
  │
  ├──► read(fd, &content[0], st.st_size)
  │
  └──► return (n == st.st_size)
```

### 递归获取目录文件

```
GetDirFiles(path, files)
  │
  ├──► DIR* dir = opendir(path.c_str())
  │
  └──► while (entry = readdir(dir))
          │
          ├──► 跳过 "." 和 ".."
          │
          ├──► fullPath = path + "/" + entry->d_name
          │
          ├──► if (entry->d_type == DT_DIR)
          │       │
          │       └──► GetDirFiles(fullPath, files)  // 递归
          │
          └──► else
                  │
                  └──► files.push_back(fullPath)
```

---

## 相关跳转

- [架构说明](03_Architecture.md) - 整体架构
- [内部 API](05_Inner_API.md) - 模块接口
