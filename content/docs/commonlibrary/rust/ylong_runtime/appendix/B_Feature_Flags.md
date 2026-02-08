# 附录 B：Feature Flags

## 可用 Feature

| Feature | 默认启用 | 依赖 | 说明 |
|---------|:--------:|------|------|
| `default` | ✅ | task, iter | 默认功能 |
| `full` | ❌ | 全部 | 全部功能 |
| `fs` | ❌ | - | 异步文件系统 |
| `macros` | ✅ | ylong_runtime_macros | 过程宏支持 |
| `net` | ❌ | ylong_io | 异步网络 |
| `sync` | ❌ | - | 同步原语 |
| `time` | ❌ | - | 定时器 |
| `signal` | ❌ | - | 信号处理 |
| `process` | ❌ | - | 进程管理 |
| `metrics` | ❌ | - | 性能指标 |
| `ffrt` | ❌ | ffrt 系统依赖 | 使用 FFRT 调度器 |
| `current_thread_runtime` | ❌ | - | 单线程运行时 |
| `multi_instance_runtime` | ❌ | - | 多线程运行时 |

## 互斥规则

### ffrt 互斥

`ffrt` 与以下 features **互斥**：

- `current_thread_runtime`
- `multi_instance_runtime`
- `metrics`

**证据**: `ylong_runtime/src/lib.rs:19-29`

```rust
#[cfg(all(
    feature = "ffrt",
    any(feature = "current_thread_runtime", feature = "multi_instance_runtime")
))]
compile_error!("Feature ffrt can not be enabled with feature current_thread_runtime or feature multi_instance_runtime");

#[cfg(all(feature = "ffrt", not(target_os = "linux")))]
compile_error!("Feature ffrt only works on linux currently");

#[cfg(all(feature = "ffrt", feature = "metrics"))]
compile_error!("Feature ffrt can not be enabled with feature metrics");
```

## 配置组合

### OpenHarmony 标准配置

```toml
[dependencies]
ylong_runtime = {
    features = ["fs", "macros", "net", "sync", "time"]
}
```

### 独立应用 (Rust 原生调度器)

```toml
[dependencies]
ylong_runtime = {
    features = ["fs", "macros", "net", "sync", "time", "current_thread_runtime"]
}
```

### 最小配置

```toml
[dependencies]
ylong_runtime = {
    features = ["macros"]  # 仅基础功能
}
```

### 完整功能

```toml
[dependencies]
ylong_runtime = {
    features = ["full"]  # 或显式列出所有 features
}
```

## GN vs Cargo Feature 映射

| GN BUILD.gn | Cargo Cargo.toml | 说明 |
|-------------|------------------|------|
| `"fs"` | `features = ["fs"]` | 异步文件系统 |
| `"macros"` | `features = ["macros"]` | 过程宏 |
| `"net"` | `features = ["net"]` | 网络 |
| `"sync"` | `features = ["sync"]` | 同步原语 |
| `"time"` | `features = ["time"]` | 定时器 |

**证据**: `ylong_runtime/BUILD.gn:23-29`
