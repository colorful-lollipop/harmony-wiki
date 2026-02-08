# 附录 A：关键调用链

## 任务生命周期

### spawn → 执行流程

```
用户代码
    │
    ▼
ylong_runtime::spawn(future)
    │
    ├──► Task::new(future)
    │       │
    │       └──► ylong_runtime/src/task/mod.rs
    │
    ├──► Runtime::spawn(task)
    │       │
    │       └──► ylong_runtime/src/executor/mod.rs
    │
    └──► TaskQueue::push(task)
            │
            └──► ylong_runtime/src/executor/queue.rs
                    │
                    ▼
            ┌───────────────────┐
            │   Worker Thread   │
            │                   │
            │ TaskQueue::pop()  │
            │         │         │
            │         ▼         │
            │   poll(task)      │
            │         │         │
            ├─────────┼─────────┤
            │         │         │
            ▼         ▼         ▼
         完成    阻塞     取消
```

**证据**: 
- `ylong_runtime/src/spawn.rs` → spawn 函数入口
- `ylong_runtime/src/task/mod.rs` → Task 定义
- `ylong_runtime/src/executor/mod.rs` → Runtime spawn

### 异步 IO 流程

```
用户代码
    │
    ▼
TcpStream::connect(addr)
    │
    ├──► Reactor::new()
    │       │
    │       └──► ylong_runtime/src/executor/driver.rs
    │
    ├──► connect() (non-blocking)
    │       │
    │       └──► ylong_io/src/sys/unix/tcp/socket.rs
    │
    └──► epoll_ctl(ADD interest)
            │
            └──► ylong_io/src/sys/unix/epoll.rs
                    │
                    ▼
            ┌───────────────────┐
            │   Event Loop     │
            │                   │
            │ epoll_wait()      │
            │         │         │
            │         ▼         │
            │   connect event   │
            │         │         │
            │         ▼         │
            │   wake task       │
            │         │         │
            ▼         ▼         ▼
         Ready   Pending   Error
```

**证据**:
- `ylong_runtime/src/net/sys/tcp/` → TCP 连接入口
- `ylong_io/src/sys/unix/tcp/socket.rs` → 系统调用绑定
- `ylong_io/src/sys/unix/epoll.rs` → epoll 操作

## 同步原语调用链

### Mutex lock 流程

```
用户代码
    │
    ▼
Mutex::lock()
    │
    ├──► Contention::wait()
    │       │
    │       └──► ylong_runtime/src/sync/waiter.rs
    │
    └──► Waker::wake()
            │
            └──► Reactor::wake()
                    │
                    ▼
            ┌───────────────────┐
            │   Worker Thread   │
            │                   │
            │ wake() → poll()  │
            │         │         │
            │         ▼         │
            │   lock acquired   │
            └───────────────────┘
```

**证据**: `ylong_runtime/src/sync/mutex.rs`

## FFRT 集成调用链

```
ylong_runtime spawn
    │
    ▼
ffrt::spawn(fn)
    │
    ├──► ylong_runtime/src/ffrt/spawner.rs
    │
    └──► ylong_ffrt (FFI)
            │
            └──► ylong_ffrt/src/lib.rs
                    │
                    ▼
            ┌───────────────────┐
            │   FFRT (C++)      │
            │                   │
            │ task_submit()     │
            │         │         │
            │         ▼         │
            │   FFRT 调度       │
            └───────────────────┘
```

**证据**: `ylong_runtime/src/ffrt/`, `ylong_ffrt/src/`

## 定时器调用链

```
sleep(duration)
    │
    ├──► Timer::new()
    │       │
    │       └──► ylong_runtime/src/time/timer.rs
    │
    ├──► Timer::sleep()
    │       │
    │       └──► ylong_runtime/src/time/sleep.rs
    │
    └──► Waker::wake()
            │
            └──► 时间轮驱动
                    │
                    ▼
            ┌───────────────────┐
            │   Timer Driver    │
            │   (时间轮)         │
            │                   │
            │ insert(duration)  │
            │         │         │
            │         ▼         │
            │   超时触发        │
            │         │         │
            ▼         ▼         ▼
         Ready   Pending   Error
```

**证据**: `ylong_runtime/src/time/wheel.rs`, `ylong_runtime/src/time/driver.rs`
