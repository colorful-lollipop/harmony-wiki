# 03 - OH 构建适配

## 3.1 BUILD.gn 概述

libuv 的 BUILD.gn 文件实现了**多平台、多模式**的构建支持，是理解 OH 适配的关键文件。

### 文件位置

```
third_party/libuv/
├── BUILD.gn           # 主构建配置 (398 行)
├── libuv.gni          # GN 参数定义 (21 行)
└── libuv.gni          # 被其他模块 import
```

---

## 3.2 构建目标

### 3.2.1 输出目标

| 目标名称 | 类型 | 说明 |
|---------|------|------|
| `//third_party/libuv:uv` | shared_library | 动态库 (.so) |
| `//third_party/libuv:uv_static` | static_library | 静态库 (.a) |
| `:libuv_source` | ohos_source_set | 源文件集合 |

### 3.2.2 安装目标

| 目标名称 | 类型 | 说明 |
|---------|------|------|
| `:libuv.para` | ohos_prebuilt_para | 参数配置 |
| `:libuv.para.dac` | ohos_prebuilt_para | DAC 权限配置 |

---

## 3.3 构建模式

### 3.3.1 双模式架构

```
BUILD.gn
├── ohos_lite 模式      # 轻量系统
│   └── source_set + static_library/shared_library
│
└── 标准模式            # 标准系统
    └── ohos_source_set + ohos_static_library/ohos_shared_library
```

### 3.3.2 Lite 模式配置

```gn
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")
  import("//build/lite/ndk/ndk.gni")
  
  config("libuv_config") {
    include_dirs = [
      "include",
      "src",
      "src/unix",
    ]
    cflags = [
      "-Wno-unused-parameter",
      "-Wno-incompatible-pointer-types",
      "-D_GNU_SOURCE",
      "-D_POSIX_C_SOURCE=200112",
    ]
  }
  
  # 固定源文件列表
  source_set("libuv_source") {
    sources = [
      "src/fs-poll.c",
      "src/inet.c",
      # ... 共 30+ 个文件
    ]
  }
}
```

### 3.3.3 标准模式配置

```gn
# 标准系统使用 ohos_* 规则
ohos_source_set("libuv_source") {
  branch_protector_ret = "pac_ret"  # ARM64 PAC-RET 保护
  configs = [ ":libuv_config" ]
  cflags = [ "-fvisibility=hidden" ]  # 符号隐藏
  # ...
}
```

---

## 3.4 平台适配

### 3.4.1 平台检测逻辑

```gn
if (is_mac || (defined(is_ios) && is_ios)) {
  # macOS/iOS 配置
  sources += nonwin_srcs + [
    "src/unix/bsd-ifaddrs.c",
    "src/unix/darwin.c",
    "src/unix/kqueue.c",
    # ...
  ]
} else if (is_mingw || is_win) {
  # Windows 配置
  sources += [
    "src/win/async.c",
    "src/win/core.c",
    # ... 共 20+ 个文件
  ]
} else if (is_ohos || (defined(is_android) && is_android)) {
  # OHOS/Android 配置
  sources += nonwin_srcs + [
    "src/unix/linux.c",
    "src/unix/procfs-exepath.c",
    "src/unix/random-getentropy.c",
    # ...
  ]
} else if (is_linux) {
  # 标准 Linux 配置
  sources += nonwin_srcs + [ /* ... */ ]
}
```

### 3.4.2 OHOS 特有源文件

```gn
if (is_ohos) {
  # OHOS 特有文件
  sources += [ "src/unix/ohos/trace_ohos.c" ]
  
  if (is_emulator) {
    # 模拟器特有
    sources += [ "src/unix/ohos/log_ohos.c" ]
  }
  
  # 外部依赖
  external_deps += [
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
  ]
}
```

---

## 3.5 关键编译选项

### 3.5.1 条件编译定义

| 宏 | 启用条件 | 作用 |
|---|---------|------|
| `USE_FFRT` | `libuv_use_ffrt && is_ohos` | FFRT 集成 |
| `ASYNC_STACKTRACE` | `enable_async_stack && is_ohos` | 异步堆栈 |
| `ENABLE_WORKER_PRIORITY` | `enable_worker_prio && is_ohos` | Worker 优先级 |
| `USE_OHOS_DFX` | `use_ohos_dfx && is_ohos && !is_emulator` | DFX 诊断 |
| `SUPPORT_INTERRUPT` | `use_ohos_dfx && is_ohos && !is_emulator` | 中断支持 |

### 3.5.2 编译器标志

```gn
if (is_linux || is_ohos) {
  cflags += [
    "-Wno-incompatible-pointer-types",
    "-D_GNU_SOURCE",
    "-D_POSIX_C_SOURCE=200112",
  ]
}

if (enable_uv_statisic && is_ohos) {
  cflags += [ "-Wno-frame-address" ]  # for __builtin_return_address
}
```

### 3.5.3 链接器选项

```gn
if (is_ohos) {
  output_extension = "so"
  external_deps = [ "hilog:libhilog" ]
  
  if (is_clang && target_cpu == "arm64" && defined(build_ext_path)) {
    ldflags = [
      "-Wl,--emit-relocs",
      "-Wl,--no-relax",
      "-mno-fix-cortex-a53-843419"
    ]
  }
}
```

---

## 3.6 libuv.gni 详解

### 3.6.1 参数定义

```gn
declare_args() {
  # FFRT 集成开关
  libuv_use_ffrt = false
  
  # 异步堆栈跟踪
  enable_async_stack = true
  
  # UV 统计功能
  enable_uv_statisic = false
  
  # DFX 诊断框架
  use_ohos_dfx = true
  
  # Worker 优先级
  enable_worker_prio = true
}
```

### 3.6.2 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| `libuv_use_ffrt` | bool | false | 启用 FFRT 任务调度 |
| `enable_async_stack` | bool | true | 启用异步堆栈跟踪 |
| `enable_uv_statisic` | bool | false | 启用 UV 统计 |
| `use_ohos_dfx` | bool | true | 启用 OHOS DFX |
| `enable_worker_prio` | bool | true | 启用 Worker 优先级 |

### 3.6.3 使用方式

其他模块可以在 BUILD.gn 中覆盖这些参数：

```gn
# 启用 FFRT
libuv_use_ffrt = true

# 禁用异步堆栈
enable_async_stack = false

import("//third_party/libuv/libuv.gni")
```

---

## 3.7 源文件组织

### 3.7.1 公共源文件 (common_source)

```gn
common_source = [
  "src/fs-poll.c",         # 文件系统轮询
  "src/idna.c",            # 国际化域名
  "src/inet.c",            # 网络地址处理
  "src/random.c",          # 随机数生成
  "src/strscpy.c",         # 安全字符串拷贝
  "src/threadpool.c",      # 线程池
  "src/thread-common.c",   # 线程通用代码
  "src/timer.c",           # 定时器
  "src/uv-common.c",       # 通用 UV 代码
  "src/uv-data-getter-setters.c",
  "src/version.c",         # 版本信息
  "src/strtok.c",          # 字符串分割
]
```

### 3.7.2 非 Windows 源文件 (nonwin_srcs)

```gn
nonwin_srcs = [
  "src/unix/async.c",      # 异步句柄
  "src/unix/core.c",       # 核心功能
  "src/unix/dl.c",         # 动态加载
  "src/unix/fs.c",         # 文件系统
  "src/unix/getaddrinfo.c", # DNS 解析
  "src/unix/getnameinfo.c",
  "src/unix/loop.c",       # 事件循环
  "src/unix/pipe.c",       # 管道
  "src/unix/process.c",    # 进程管理
  "src/unix/signal.c",     # 信号处理
  "src/unix/stream.c",     # 流
  "src/unix/tcp.c",        # TCP
  "src/unix/thread.c",     # 线程
  "src/unix/tty.c",        # TTY
  "src/unix/udp.c",        # UDP
]
```

### 3.7.3 OHOS 特有源文件

```gn
if (is_ohos) {
  # OHOS 跟踪支持
  sources += [ "src/unix/ohos/trace_ohos.c" ]
  
  # 模拟器日志支持
  if (is_emulator) {
    sources += [ "src/unix/ohos/log_ohos.c" ]
  }
  
  # 异步堆栈支持
  if (is_ohos && enable_async_stack) {
    sources += [ "src/dfx/async_stack/libuv_async_stack.c" ]
  }
}
```

---

## 3.8 与上游构建系统对比

### 3.8.1 上游构建方式

| 方式 | 工具 | 说明 |
|-----|------|------|
| Autotools | configure + make | Unix 传统方式 |
| CMake | cmake | 跨平台现代方式 |

### 3.8.2 OH 构建方式

| 方式 | 工具 | 说明 |
|-----|------|------|
| GN | BUILD.gn | OH 统一构建系统 |

### 3.8.3 关键差异

| 方面 | 上游 | OH 版本 |
|-----|------|--------|
| 构建系统 | CMake/Autotools | GN |
| 平台支持 | 通用 | OH 特有优化 |
| 功能扩展 | 无 | FFRT、DFX、异步堆栈 |
| 安全特性 | 基础 | PAC-RET、fvisibility=hidden |
| 安装位置 | /usr/local | system/updater |

---

## 3.9 配置示例

### 3.9.1 完整依赖声明

```gn
ohos_shared_library("uv") {
  deps = [
    ":libuv.para",
    ":libuv.para.dac",
    ":libuv_source",
  ]
  public_configs = [ ":libuv_config" ]
  subsystem_name = "thirdparty"
  part_name = "libuv"
  innerapi_tags = [ "platformsdk" ]
  
  if (is_ohos) {
    output_extension = "so"
    external_deps = [ "hilog:libhilog" ]
    
    install_images = [
      "system",
      "updater",
    ]
  }
}
```

### 3.9.2 其他模块引用示例

```gn
# 使用动态库
ohos_shared_library("my_module") {
  deps = [ "//third_party/libuv:uv" ]
  external_deps = [ "libuv:uv" ]
}

# 使用静态库
ohos_static_library("my_module") {
  deps = [ "//third_party/libuv:uv_static" ]
}
```

---

## 3.10 构建调试

### 3.10.1 查看编译定义

```bash
# 在 out 目录查看编译命令
cat out/xxx/build.log | grep "libuv"
```

### 3.10.2 验证宏定义

```c
// 在代码中添加调试信息
#ifdef USE_FFRT
  #pragma message("USE_FFRT is defined")
#endif

#ifdef USE_OHOS_DFX
  #pragma message("USE_OHOS_DFX is defined")
#endif
```

### 3.10.3 常见问题

| 问题 | 原因 | 解决 |
|-----|------|------|
| 找不到 FFRT 头文件 | libuv_use_ffrt 未启用 | 检查 libuv.gni |
| HiLog 未定义 | is_emulator 判断错误 | 检查 target 配置 |
| 符号冲突 | 未使用 hidden visibility | 检查 cflags |

---

## 3.11 总结

BUILD.gn 的关键设计要点：

| 设计 | 说明 |
|-----|------|
| **双模式支持** | 同时支持 lite 和标准系统 |
| **多平台适配** | Linux、OHOS、Android、macOS、Windows |
| **可配置特性** | 通过 libuv.gni 参数控制功能 |
| **安全加固** | PAC-RET、符号隐藏 |
| **模块化设计** | source_set + library 分离 |

---

*注: 修改 BUILD.gn 后需重新执行 gn gen 和 ninja 构建*
