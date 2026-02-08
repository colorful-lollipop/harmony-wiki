# API/接口差异

本文档记录 Rust 工具链在 OpenHarmony 环境下的 API 差异和行为变更。

## 概述

Rust 标准库 (`std`) 在 OpenHarmony 上基本保持与上游一致，但存在以下主要差异：

1. **目标环境检测** - `target_env` 增加 `"ohos"` 值
2. **线程本地存储** - 无原生 TLS，使用仿真方案
3. **系统调用接口** - OHOS 特定系统调用绑定
4. **I/O 限制** - 部分 Unix I/O 功能受限

---

## 目标环境差异

### target_env 配置

**上游行为**:
```rust
// Linux
println!("env: {}", std::env::consts::ENV);  // "gnu" 或 "musl"

// macOS
println!("env: {}", std::env::consts::ENV);  // "macos"

// Windows
println!("env: {}", std::env::consts::ENV);  // "msvc"
```

**OpenHarmony 差异**:
```rust
// OpenHarmony
println!("env: {}", std::env::consts::ENV);  // "ohos"
```

**影响范围**:
- `std::env::consts::ENV` 返回 `"ohos"`
- `std::env::consts::OS` 返回 `"linux"`
- `std::env::consts::FAMILY` 返回 `"unix"`

### 条件编译

```rust
// OpenHarmony 特定代码
#[cfg(target_env = "ohos")]
fn ohos_api() {
    // OpenHarmony 特定实现
    println!("Running on OpenHarmony!");
}

// 通用 Unix 实现
#[cfg(target_family = "unix")]
fn unix_api() {
    // 适用于所有 Unix 系统，包括 OHOS
}
```

---

## 线程本地存储 (TLS) 差异

### 原生 TLS 支持状态

| 目标平台 | 原生 TLS | 使用方案 |
|---------|---------|---------|
| Linux glibc | ✅ 支持 | `pthread_getspecific` |
| macOS | ✅ 支持 | `pthread` |
| Windows | ✅ 支持 | `Tls*` APIs |
| **OpenHarmony** | ❌ 不支持 | **仿真 TLS** |

### TLS API 行为差异

**代码示例**:

```rust
// 标准 TLS 使用方式 (所有平台相同)
use std::thread;

thread_local! {
    static THREAD_DATA: std::cell::RefCell<String> = std::cell::RefCell::new(String::new());
}

fn main() {
    THREAD_DATA.with(|data| {
        *data.borrow_mut() = "thread-local data".to_string();
    });
}
```

**内部实现差异**:

| 平台 | TLS 实现 |
|------|---------|
| Linux glibc | 动态链接 `libc.so` TLS 槽位 |
| musl | 静态 TLS 模型 |
| **OpenHarmony** | **仿真层 (emulated TLS)** |

**性能影响**:
- 仿真 TLS 比原生 TLS 慢约 10-20%
- `thread_local!` 宏行为一致，但内部查找开销增加

---

## 系统调用接口差异

### 可用系统调用

OpenHarmony 的 Linux 兼容层支持大部分标准 Linux 系统调用，但以下调用可能受限或不可用：

| 系统调用 | 状态 | 说明 |
|---------|------|------|
| `socket` | ✅ 可用 | 网络编程正常 |
| `poll`/`select` | ✅ 可用 | I/O 多路复用 |
| `pthread` | ✅ 可用 | 线程支持 |
| `mmap` | ✅ 可用 | 内存映射 |
| `fork`/`exec` | ⚠️ 受限 | 进程创建受限 |
| `ptrace` | ❌ 不可用 | 调试受限 |
| `signalfd` | ⚠️ 部分 | 信号处理受限 |
| `inotify` | ⚠️ 部分 | 文件监控受限 |

### libc 绑定差异

```rust
// 标准 Unix 系统调用
use libc::{c_int, c_char};

extern "C" {
    pub fn socket(domain: c_int, type_: c_int, protocol: c_int) -> c_int;
    pub fn bind(sockfd: c_int, addr: *const sockaddr, len: socklen_t) -> c_int;
}

// OHOS 特定可用，但可能返回 ENOSYS
extern "C" {
    pub fn fork() -> pid_t;  // 可能返回 -1，设置 errno
}
```

---

## I/O 和文件系统差异

### 标准 I/O 行为

```rust
// 文件 I/O - 行为一致
use std::fs::File;
use std::io::prelude::*;

fn read_file() -> std::io::Result<()> {
    let mut file = File::open("test.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(())
}
```

### 路径差异

```rust
// 路径处理
use std::path::Path;

let path = Path::new("/data/app/files");

// OHOS 应用沙箱路径
// 通常在 /data/app/<bundle_name>/ 目录下

#[cfg(target_os = "ohos")]
fn get_app_path() -> std::path::PathBuf {
    let mut path = std::path::PathBuf::from("/data/app");
    // 添加应用包名
    path.push(get_bundle_name());
    path
}
```

### 网络 I/O

```rust
// 标准网络编程 - 行为一致
use std::net::{TcpStream, TcpListener, UdpSocket};

fn network_demo() -> std::io::Result<()> {
    let stream = TcpStream::connect("127.0.0.1:8080")?;
    Ok(())
}
```

**注意**: OHOS 可能需要特定的网络权限配置。

---

## 标准库功能限制

### 可用功能

```toml
# Cargo.toml - 以下功能在 OHOS 上可用
[dependencies]
std = { version = "1.0", features = [
    "backtrace",           # 栈回溯
    "panic_unwind",        # panic 展开
    "alloc",               # 堆分配
    "thread",              # 线程
    "fs",                  # 文件系统
    "net",                 # 网络
    "os",                  # OS 信息
    "process",             # 进程
    "sync",                # 同步原语
] }
```

### 受限/不可用功能

| 功能 | 状态 | 替代方案 |
|------|------|---------|
| `std::process::Command::new("sh")` | ⚠️ 受限 | 使用 `std::process::Command::new("bm")` |
| `std::env::home_dir` | ⚠️ 可能受限 | 使用应用沙箱路径 |
| `std::env::var("HOME")` | ⚠️ 受限 | 使用 `std::env::var("APP_DATA_HOME")` |
| `Ctrl-C` 信号处理 | ⚠️ 受限 | 使用 OHOS 生命周期 API |

---

## FFI 和 C 互操作

### CXX 互操作

```rust
// 使用 cxx 进行 C++ 互操作
#[cxx::bridge]
mod ffi {
    unsafe extern "C++" {
        include!("my_module.h");
        fn initialize();
        fn process_data(data: &[u8]) -> i32;
    }
}
```

### OHOS NDK 绑定

```rust
// 访问 OHOS NDK API
#[cfg(target_os = "ohos")]
mod ohos_ndk {
    use libc::{c_int, c_char};

    extern "C" {
        // OHOS 基础能力
        pub fn OH_ACCElemInit(context: *mut c_void) -> c_int;
        pub fn OH_ACCElemFinalize(context: *mut c_void);
        pub fn OH_ACCElemRun(context: *mut c_void, data: *const c_char) -> c_int;
    }
}
```

---

## 新增 API

### OpenHarmony 特定 API

虽然 Rust 标准库保持不变，但 OpenHarmony 提供了额外的绑定库：

| 库 | 用途 | 使用方式 |
|------|------|---------|
| `yOHos` | OHOS 能力访问 | `use yOHos::*` |
| `ffi_ohos` | FFI 绑定 | `use ffi_ohos::*` |

### OHOS 能力访问示例

```rust
// 注意：以下为示例，实际 API 请参考 OHOS NDK
#[cfg(target_os = "ohos")]
mod ohos_api {
    use std::ffi::CString;

    pub fn get_device_info() -> DeviceInfo {
        // 调用 OHOS C API
        unsafe {
            let mut info = std::mem::zeroed();
            OH_DeviceInfo_Get(&mut info);
            info
        }
    }

    pub fn get_ability_context() -> AbilityContext {
        // 获取 ability 上下文
        unsafe {
            let context = OH_Ability_GetContext();
            AbilityContext::from_ptr(context)
        }
    }
}
```

---

## 性能差异

### 基准测试注意事项

```rust
// 线程创建开销 (OHOS 可能略高)
fn thread_creation_benchmark() {
    use std::time::Instant;

    let start = Instant::now();
    let handles: Vec<_> = (0..100)
        .map(|_| std::thread::spawn(|| {
            // 简单工作
            let _ = 1 + 1;
        }))
        .collect();

    for handle in handles {
        handle.join().unwrap();
    }

    let duration = start.elapsed();
    println!("Thread creation: {:?}", duration);
}
```

### 内存分配器

```rust
// OHOS 默认使用系统分配器
#[global_allocator]
static ALLOC: std::alloc::System = std::alloc::System;

// 如需jemalloc，需自行集成
// 性能敏感场景可考虑:
#[cfg(target_os = "ohos")]
{
    // 评估 jemalloc 或 mimalloc
}
```

---

## 调试和日志差异

### 错误处理

```rust
// 标准错误处理 - 行为一致
use std::io::{Error, ErrorKind};

fn risky_operation() -> std::io::Result<()> {
    Err(Error::new(ErrorKind::Other, "OHOS specific error"))
}
```

### 日志框架

```rust
// 推荐使用 log 或 env_logger
use log::{info, warn, error};

#[cfg(target_os = "ohos")]
fn log_demo() {
    info!("OHOS info log");
    warn!("OHOS warning log");
    error!("OHOS error log");
}
```

---

## 迁移注意事项

### 从 Linux 迁移

| 检查项 | Linux | OHOS | 行动 |
|--------|-------|------|------|
| TLS | 原生 | 仿真 | 测试 TLS 性能 |
| 文件系统 | `/home` | `/data/app` | 更新路径 |
| 网络 | 完整 | 需权限 | 检查权限配置 |
| 信号 | 完整 | 受限 | 避免信号 IPC |

### 最佳实践

```rust
// 跨平台代码示例
use std::path::PathBuf;

#[cfg(target_os = "ohos")]
fn get_config_dir() -> PathBuf {
    PathBuf::from("/data/app/config")
}

#[cfg(target_os = "linux")]
fn get_config_dir() -> PathBuf {
    PathBuf::from(std::env::var("HOME").unwrap_or("/root".to_string()))
}

#[cfg(target_os = "macos")]
fn get_config_dir() -> PathBuf {
    let mut path = std::env::home_dir().unwrap_or(PathBuf::from("/Users/root"));
    path.push("Library/Application Support");
    path
}
```

---

## 总结

| 差异类别 | 影响程度 | 迁移难度 |
|---------|---------|---------|
| `target_env` | 低 | 无需修改 |
| TLS 仿真 | 中 | 需性能测试 |
| 系统调用 | 中 | 需代码审查 |
| 路径 | 低 | 需更新 |
| 权限 | 中 | 需配置 |

**总体评估**: Rust 标准库在 OpenHarmony 上的兼容性良好，大部分代码无需修改即可运行。
