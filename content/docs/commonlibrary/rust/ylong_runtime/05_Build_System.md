# 构建系统

## GN 构建配置

### 顶层配置 (bundle.json)

**证据**: `bundle.json:35-59`

```json
{
  "build": {
    "sub_component": [
      "//commonlibrary/rust/ylong_runtime/ylong_io:ylong_io",
      "//commonlibrary/rust/ylong_runtime/ylong_signal:ylong_signal"
    ],
    "inner_kit": [
      {
        "name": "//commonlibrary/rust/ylong_runtime/ylong_runtime:ylong_runtime"
      },
      {
        "name": "//commonlibrary/rust/ylong_runtime/ylong_runtime:ylong_runtime_static"
      },
      {
        "name": "//commonlibrary/rust/ylong_runtime/ylong_runtime_macros:ylong_runtime_macros"
      },
      {
        "name": "//commonlibrary/rust/ylong_runtime/ylong_signal:ylong_signal"
      },
      {
        "name": "//commonlibrary/rust/ylong_runtime/ylong_io:ylong_io"
      }
    ]
  }
}
```

## Targets 详解

### ylong_runtime (动态库)

**证据**: `ylong_runtime/BUILD.gn:16-42`

```gn
ohos_rust_shared_library("ylong_runtime") {
  part_name = "ylong_runtime"
  subsystem_name = "commonlibrary"
  
  crate_name = "ylong_runtime"
  edition = "2021"
  
  features = [
    "fs",
    "macros",
    "net",
    "sync",
    "time",
  ]
  
  sources = [ "src/lib.rs" ]
  deps = [
    "../ylong_io:ylong_io",
    "../ylong_runtime_macros:ylong_runtime_macros(${host_toolchain})",
  ]
  
  external_deps = [ "rust_libc:lib" ]
  
  innerapi_tags = [ "chipsetsdk" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `part_name` | `ylong_runtime` | 组件名 |
| `crate_name` | `ylong_runtime` | Rust crate 名 |
| `edition` | `2021` | Rust edition |
| `sources` | `["src/lib.rs"]` | 源码入口 |
| `deps` | ylong_io, ylong_runtime_macros | 内部依赖 |
| `external_deps` | `rust_libc:lib` | 系统依赖 |
| **产物** | `libylong_runtime.so` | 动态库 |

### ylong_runtime_static (静态库)

**证据**: `ylong_runtime/BUILD.gn:44-66`

```gn
ohos_rust_static_library("ylong_runtime_static") {
  part_name = "ylong_runtime"
  subsystem_name = "commonlibrary"
  
  crate_name = "ylong_runtime_static"
  edition = "2021"
  
  features = [
    "fs",
    "macros",
    "net",
    "sync",
    "time",
  ]
  
  sources = [ "src/lib.rs" ]
  deps = [
    "../ylong_io:ylong_io",
    "../ylong_runtime_macros:ylong_runtime_macros(${host_toolchain})",
  ]
  
  external_deps = [ "rust_libc:lib" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| **产物** | `libylong_runtime.a` | 静态库 |

### ylong_io (静态库)

**证据**: `ylong_io/BUILD.gn:16-31`

```gn
ohos_rust_static_library("ylong_io") {
  part_name = "ylong_runtime"
  subsystem_name = "commonlibrary"
  
  crate_name = "ylong_io"
  edition = "2021"
  
  features = [
    "tcp",
    "udp",
  ]
  
  sources = [ "src/lib.rs" ]
  
  external_deps = [ "rust_libc:lib" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| **产物** | `libylong_io.a` | 静态库 |

### ylong_signal (静态库)

**证据**: `ylong_signal/BUILD.gn:16-26`

```gn
ohos_rust_static_library("ylong_signal") {
  part_name = "ylong_runtime"
  subsystem_name = "commonlibrary"
  
  crate_name = "ylong_signal"
  edition = "2021"
  
  sources = [ "src/lib.rs" ]
  
  external_deps = [ "rust_libc:lib" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| **产物** | `libylong_signal.a` | 静态库 |

### ylong_runtime_macros (过程宏)

**证据**: `ylong_runtime_macros/BUILD.gn:16-24`

```gn
ohos_rust_proc_macro("ylong_runtime_macros") {
  part_name = "ylong_runtime"
  subsystem_name = "commonlibrary"
  
  crate_name = "ylong_runtime_macros"
  edition = "2021"
  
  sources = [ "src/lib.rs" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| **产物** | `ylong_runtime_macros.so` | 编译时展开 |

## 产物汇总

| 产物类型 | 模块 | 输出路径 (推测) |
|---------|------|----------------|
| `.so` | ylong_runtime | `out/{device}/commonlibrary/rust/ylong_runtime/` |
| `.a` | ylong_runtime_static | `out/{device}/commonlibrary/rust/ylong_runtime/` |
| `.a` | ylong_io | `out/{device}/commonlibrary/rust/ylong_runtime/` |
| `.a` | ylong_signal | `out/{device}/commonlibrary/rust/ylong_runtime/` |
| `.so` | ylong_runtime_macros | `out/{device}/commonlibrary/rust/ylong_runtime/` |

## Feature Flags 映射

| BUILD.gn features | Cargo features | 说明 |
|-------------------|---------------|------|
| `fs` | `fs` | 异步文件系统 |
| `macros` | `macros` | 过程宏支持 (`select!`) |
| `net` | `net` | 异步网络 |
| `sync` | `sync` | 同步原语 |
| `time` | `time` | 定时器 |

## 依赖关系图

```
ylong_runtime (.so)
    ├── ylong_io (.a)
    │       └── rust_libc
    ├── ylong_runtime_macros (proc_macro)
    └── rust_libc

ylong_runtime_static (.a)
    ├── ylong_io (.a)
    │       └── rust_libc
    ├── ylong_runtime_macros (proc_macro)
    └── rust_libc

ylong_signal (.a)
    └── rust_libc
```

## 系统依赖

### FFRT (可选)

当启用 `ffrt` feature 时，需要依赖 OpenHarmony FFRT：

```gn
external_deps = [ "ffrt:ffrt" ]
```

**证据**: `bundle.json:29` → `"ffrt"`

### rust_libc

所有模块都依赖 `rust_libc`：

```gn
external_deps = [ "rust_libc:lib" ]
```

## 使用方式

### 作为动态库依赖

```gn
# BUILD.gn
external_deps = [ "ylong_runtime:ylong_runtime" ]
```

### 作为静态库依赖

```gn
# BUILD.gn
external_deps = [ "ylong_runtime:ylong_runtime_static" ]
```

### Cargo 依赖

```toml
[dependencies]
ylong_runtime = { 
    git = "https://gitcode.com/openharmony/commonlibrary_rust_ylong_runtime.git",
    features = ["full"]
}
```
