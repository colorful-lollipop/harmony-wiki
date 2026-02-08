# 目录结构

## 顶层结构

```
ylong_runtime/
├── docs/                          # 用户文档
│   └── user_guide.md
├── figures/                       # 文档中的图片资源
├── wiki/                          # 本工程 Wiki
│   ├── README.md
│   ├── SUMMARY.md
│   ├── 01_Overview.md
│   ├── 02_Directory_Structure.md
│   ├── 03_Architecture.md
│   ├── 04_API_Reference.md
│   ├── 05_Build_System.md
│   ├── 06_Security_Review.md
│   └── appendix/
├── ylong_runtime/                 # ⭐ 主 crate（用户直接依赖）
├── ylong_io/                     # IO 底层模块
├── ylong_ffrt/                   # FFRT 适配器
├── ylong_runtime_macros/         # 过程宏
└── ylong_signal/                 # 信号处理模块
```

## ylong_runtime 主模块详解

```
ylong_runtime/src/
├── builder/                       # 运行时构建器
│   ├── common_builder.rs          # 通用构建配置
│   ├── current_thread_builder.rs  # 单线程运行时构建
│   └── multi_thread_builder.rs    # 多线程运行时构建
├── executor/                      # 执行器核心
│   ├── driver.rs                  # IO/Timer 驱动
│   ├── worker.rs                  # 工作线程
│   ├── async_pool.rs              # 异步任务池
│   ├── blocking_pool.rs          # 阻塞任务池
│   └── ...
├── ffrt/                          # FFRT 适配器（条件编译）
│   ├── ffrt_task.rs              # FFRT 任务封装
│   └── ffrt_timer.rs             # FFRT 定时器
├── fs/                            # 异步文件系统
│   ├── async_file.rs             # 异步文件
│   ├── async_dir.rs              # 异步目录
│   └── open_options.rs           # 打开选项
├── io/                            # 异步 IO traits
│   ├── async_read.rs             # 异步读 trait
│   ├── async_write.rs            # 异步写 trait
│   ├── buffered/                 # 缓冲 IO
│   │   ├── async_buf_reader.rs
│   │   └── async_buf_writer.rs
│   └── ...
├── iter/                          # 并行迭代器
│   ├── parallel/                 # 并行集合
│   │   └── collections/         # Vec、HashMap 等
│   └── pariter/                  # 迭代器核心实现
├── net/                           # 异步网络
│   ├── tcp/                      # TCP 连接
│   └── sys/                      # 系统调用绑定
├── process/                       # 进程管理
│   ├── command.rs                # 命令构建
│   └── pty_process/             # PTY 进程
├── signal/                        # 信号处理
│   ├── unix/                     # Unix 信号
│   └── windows/                  # Windows 信号
├── sync/                          # 同步原语
│   ├── mutex.rs                  # 互斥锁
│   ├── rwlock.rs                 # 读写锁
│   ├── semaphore.rs             # 信号量
│   └── mpsc/                     # 多生产者单消费者通道
├── task/                          # 任务管理
│   ├── task.rs                   # 任务定义
│   ├── join_handle.rs            # join 句柄
│   ├── builder.rs                # 任务构建器
│   └── join_set.rs               # 任务集合
├── time/                          # 定时器
│   ├── sleep.rs                  # 异步 sleep
│   ├── timer.rs                  # 定时器
│   └── wheel.rs                  # 时间轮实现
├── builder.rs
├── executor.rs
├── lib.rs                         # ⭐ 主入口
└── ...
```

## ylong_io 模块详解

```
ylong_io/src/
├── lib.rs                        # 主入口
├── poll.rs                       # epoll/kqueue 封装
├── source.rs                     # 事件源
├── waker.rs                      # 任务唤醒器
├── token.rs                      # 事件令牌
├── interest.rs                   # IO 兴趣集合
└── sys/                          # 系统相关实现
    ├── unix/                     # Unix (Linux/macOS)
    │   ├── epoll.rs             # epoll 实现
    │   ├── tcp/                 # TCP socket
    │   ├── udp/                 # UDP socket
    │   ├── uds/                 # Unix Domain Socket
    │   └── kqueue.rs            # macOS kqueue
    └── windows/                  # Windows
        ├── iocp.rs              # IOCP 实现
        ├── tcp/                 # TCP socket
        └── udp/                 # UDP socket
```

## ylong_ffrt 模块详解

```
ylong_ffrt/src/
├── lib.rs                        # FFRT FFI 入口
├── task.rs                       # 任务封装
├── sys_event.rs                  # 系统事件
└── config.rs                     # 配置
```

## 模块职责总结

| 模块 | 职责 | 稳定性 |
|------|------|--------|
| `ylong_runtime` | 对外 API 层 | Stable |
| `ylong_io` | IO 事件驱动底层 | Stable |
| `ylong_ffrt` | FFRT 调度器适配 | Stable |
| `ylong_runtime_macros` | 编译时过程宏 | Stable |
| `ylong_signal` | 信号处理 | Stable |

**注意**: 所有 `test/`、`tests/`、`benches/`、`examples/` 目录内容不计入模块职责分析
