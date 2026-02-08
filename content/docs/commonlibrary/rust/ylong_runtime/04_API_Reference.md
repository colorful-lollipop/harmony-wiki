# API 参考

## 模块概览

ylong_runtime 提供以下主要 Rust 模块，通过 feature flags 控制启用：

| 模块 | Feature | 主要类型 | 说明 |
|------|---------|----------|------|
| `task` | 默认 | `spawn`, `block_on`, `JoinHandle` | 异步任务管理 |
| `sync` | `sync` | `Mutex`, `RwLock`, `Semaphore`, `Channel` | 同步原语 |
| `net` | `net` | `TcpStream`, `TcpListener`, `UdpSocket` | 异步网络 |
| `fs` | `fs` | `File`, `Dir`, `OpenOptions` | 异步文件系统 |
| `time` | `time` | `sleep`, `interval`, `Timeout` | 定时器 |
| `iter` | 默认 | `ParallelIterator` | 并行计算 |
| `signal` | `signal` | `Signal`, `SignalSet` | 信号处理 |
| `process` | `process` | `Command`, `Child` | 进程管理 |

## 核心 API

### 任务管理 (task)

**证据**: `ylong_runtime/src/task/mod.rs`, `ylong_runtime/src/spawn.rs`

#### spawn

```rust
/// 在运行时中异步执行给定的 future。
pub fn spawn<T>(future: T) -> JoinHandle<T::Output>
where
    T: Future + Send + 'static,
    T::Output: Send + 'static,
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `future` | `impl Future + Send + 'static` | 要执行的异步任务 |

**示例**:

```rust
use ylong_runtime::spawn;

spawn(async {
    println!("Hello from async task!");
}).await.unwrap();
```

#### block_on

```rust
/// 在当前线程阻塞执行给定的 future，直到完成。
pub fn block_on<F>(future: F) -> F::Output
where
    F: Future,
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `future` | `impl Future` | 要执行的异步任务 |

#### spawn_blocking

```rust
/// 在阻塞线程池中执行闭包（用于阻塞 IO 操作）。
pub fn spawn_blocking<F, R>(f: F) -> JoinHandle<R>
where
    F: FnOnce() -> R + Send + 'static,
    R: Send,
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `f` | `FnOnce() -> R + Send + 'static` | 要执行的阻塞任务 |

### 同步原语 (sync)

**证据**: `ylong_runtime/src/sync/mutex.rs`, `ylong_runtime/src/sync/rwlock.rs`, `ylong_runtime/src/sync/semaphore.rs`

#### Mutex

```rust
/// 异步互斥锁。
pub struct Mutex<T: ?Sized> {
    // ...
}

/// 尝试获取锁。
pub async fn lock(&self) -> MutexGuard<'_, T>
```

#### RwLock

```rust
/// 异步读写锁。
pub struct RwLock<T: ?Sized> {
    // ...
}

/// 获取读锁。
pub async fn read(&self) -> RwLockReadGuard<'_, T>

/// 获取写锁。
pub async fn write(&mut self) -> RwLockWriteGuard<'_, T>
```

#### Semaphore

```rust
/// 异步信号量。
pub struct Semaphore {
    // ...
}

/// 尝试获取一个 permit。
pub async fn acquire(&self) -> Permit<'_>

/// 尝试获取多个 permit。
pub async fn acquire_many(&self, n: u32) -> Result<Permit<'_>, AcquireError>
```

#### Channel

```rust
/// 多生产者单消费者通道。
pub mod mpsc {
    pub fn channel<T>(cap: usize) -> (Sender<T>, Receiver<T>);
    
    pub struct Sender<T> {
        // ...
    }
    
    pub struct Receiver<T> {
        // ...
    }
}
```

### 网络 IO (net)

**证据**: `ylong_runtime/src/net/mod.rs`

#### TcpStream

```rust
/// 异步 TCP 流。
pub struct TcpStream {
    // ...
}

/// 建立 TCP 连接。
pub async fn connect<A: ToSocketAddrs>(addr: A) -> io::Result<TcpStream>

/// 使用指定所有权建立连接。
pub async fn connect_with_owner<A: ToSocketAddrs>(
    addr: A,
    uid: uid_t,
    gid: gid_t,
) -> io::Result<TcpStream>
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `addr` | `ToSocketAddrs` | 目标地址 |
| `uid` | `uid_t` | 用户 ID (可选) |
| `gid` | `gid_t` | 组 ID (可选) |

**证据**: `ylong_io/src/sys/unix/tcp/stream.rs:42`

#### TcpListener

```rust
/// 异步 TCP 监听器。
pub struct TcpListener {
    // ...
}

/// 绑定地址并监听。
pub async fn bind<A: ToSocketAddrs>(addr: A) -> io::Result<TcpListener>
```

#### UdpSocket

```rust
/// 异步 UDP socket。
pub struct UdpSocket {
    // ...
}

/// 绑定地址。
pub async fn bind<A: ToSocketAddrs>(addr: A) -> io::Result<UdpSocket>

/// 发送数据。
pub async fn send(&self, buf: &[u8], target: SocketAddr) -> io::Result<usize>

/// 接收数据。
pub async fn recv(&self, buf: &mut [u8]) -> io::Result<(usize, SocketAddr)>
```

### 文件系统 (fs)

**证据**: `ylong_runtime/src/fs/mod.rs`, `ylong_runtime/src/fs/async_file.rs`

#### File

```rust
/// 异步文件。
pub struct File {
    // ...
}

/// 打开文件。
pub async fn open<P: AsRef<Path>>(path: P) -> io::Result<File>

/// 创建文件（写入模式）。
pub async fn create<P: AsRef<Path>>(path: P) -> io::Result<File>

/// 读取文件内容。
pub async fn read_to_string<P: AsRef<Path>>(path: P) -> io::Result<String>

/// 写入文件。
pub async fn write<P: AsRef<Path>, C: AsRef<[u8]>>(
    path: P,
    contents: C,
) -> io::Result<()>
```

#### AsyncDir

```rust
/// 异步目录操作。
pub async fn read_dir<P: AsRef<Path>>(path: P) -> io::Result<ReadDir>

/// 创建目录。
pub async fn create_dir<P: AsRef<Path>>(path: P) -> io::Result<()>

/// 递归创建目录。
pub async fn create_dir_all<P: AsRef<Path>>(path: P) -> io::Result<()>

/// 删除目录。
pub async fn remove_dir<P: AsRef<Path>>(path: P) -> io::Result<()>

/// 删除文件。
pub async fn remove_file<P: AsRef<Path>>(path: P) -> io::Result<()>
```

### 定时器 (time)

**证据**: `ylong_runtime/src/time/mod.rs`, `ylong_runtime/src/time/sleep.rs`

#### sleep

```rust
/// 异步等待指定时间。
pub async fn sleep(duration: Duration) -> ()
```

#### interval

```rust
/// 创建定时器间隔。
pub fn interval(period: Duration) -> Interval
```

#### Timeout

```rust
/// 为 Future 添加超时。
pub async fn timeout<T>(
    duration: Duration,
    future: T,
) -> Result<T::Output, Elapsed>
```

### 并行计算 (iter)

**证据**: `ylong_runtime/src/iter/mod.rs`, `ylong_runtime/src/iter/parallel/mod.rs`

#### ParallelIterator

```rust
/// 并行迭代器 trait。
pub trait ParallelIterator {
    type Item;
    
    /// 收集到 Vec。
    fn collect<C: FromParallelIterator<Self::Item>>(self) -> C
    where
        Self: Sized;
    
    /// 对每个元素执行闭包。
    fn for_each<U, F>(self, f: F)
    where
        Self: Sized,
        U: Unpin,
        F: Fn(Self::Item) -> U + Send + Sync;
    
    /// 映射并行处理。
    fn map<U, F>(self, f: F) -> Map<Self, F>
    where
        Self: Sized,
        F: Fn(Self::Item) -> U + Send + Sync;
}
```

#### 支持的集合

```rust
// Vec 并行处理
let result: Vec<i32> = (0..1000)
    .into_par_iter()
    .map(|x| x * 2)
    .collect();

// HashMap 并行构建
use ylong_runtime::iter::parallel::collections::HashMap;

let map: HashMap<i32, i32> = (0..1000)
    .into_par_iter()
    .map(|x| (x, x * 2))
    .collect();
```

### 进程管理 (process)

**证据**: `ylong_runtime/src/process/mod.rs`, `ylong_runtime/src/process/command.rs`

#### Command

```rust
/// 子进程配置。
pub struct Command {
    // ...
}

/// 创建子进程。
pub fn command(program: &str) -> Command

impl Command {
    /// 设置程序参数。
    pub fn arg(&mut self, arg: &str) -> &mut Command
    
    /// 添加环境变量。
    pub fn env(&mut self, key: &str, val: &str) -> &mut Command
    
    /// 设置工作目录。
    pub fn current_dir(&mut self, dir: PathBuf) -> &mut Command
    
    /// 设置用户 ID。
    pub fn uid(&mut self, id: u32) -> &mut Command
    
    /// 设置组 ID。
    pub fn gid(&mut self, id: u32) &mut Command
    
    /// 启动进程。
    pub async fn output(&mut self) -> io::Result<Output>
    
    /// 等待进程完成。
    pub async fn status(&mut self) -> io::Result<ExitStatus>
}
```

### 信号处理 (signal)

**证据**: `ylong_runtime/src/signal/mod.rs`

#### Signal

```rust
/// Unix 信号处理。
pub fn signal<S>(signal: S) -> Result<Signal, SignalError>
where
    S: Into<SignalKind>,
```

#### SignalKind

```rust
/// 信号类型。
pub enum SignalKind {
    /// 中断信号 (SIGINT)
    interrupt,
    /// 终止信号 (SIGTERM)
    terminate,
    /// 自定义信号。
    other(i32),
}
```

## Feature Flags 配置

### 完整功能

```toml
[dependencies]
ylong_runtime = { features = ["full"] }
```

### 最小功能

```toml
[dependencies]
ylong_runtime = { features = [] }  # 仅 task + iter
```

### OpenHarmony 推荐

```toml
[dependencies]
ylong_runtime = {
    features = ["fs", "macros", "net", "sync", "time"]
}
```

**证据**: `ylong_runtime/BUILD.gn:23-29`
