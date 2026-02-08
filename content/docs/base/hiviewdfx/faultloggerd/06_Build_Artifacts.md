# FaultLoggerd 编译产物

## 目的

本文档介绍 faultloggerd 的编译产物、安装路径和运行时加载关系。

## 适用范围

- 目标读者：系统开发者、构建工程师、测试工程师
- 内容范围：生产代码产物

## 产物总览

### 系统服务产物

| 产物类型 | 文件名 | 安装路径 | 说明 |
|---------|--------|------------|------|
| **守护进程** | `faultloggerd` | `/system/bin/faultloggerd` | 主服务进程 |
| **崩溃 dump 工具** | `processdump` | `/system/bin/processdump` | 崩溃日志生成工具 |
| **主动抓栈工具** | `dumpcatcher` | `/system/bin/dumpcatcher` | 命令行抓栈工具（Debug 版本） |
| **崩溃验证工具** | `crashvalidator` | `/system/bin/crashvalidator` | 崩溃日志验证工具 |

### 动态库产物（.so）

| 库名称 | 文件名 | 安装路径 | 加载者 |
|---------|--------|------------|--------|
| `libdfx_dumpcatcher` | `libdfx_dumpcatcher.z.so` | `/system/lib/` | DumpCatcher SDK 使用 |
| `libunwinder` | `libunwinder.z.so` | `/system/lib/` | 符号解析引擎 |
| `libfaultloggerd` | `libfaultloggerd.z.so` | `/system/lib/` | faultloggerd 客户端库 |
| `libbacktrace_local` | `libbacktrace_local.z.so` | `/system/lib/` | 本地回栈库 |
| `libdfx_procinfo` | `libdfx_procinfo.z.so` | `/system/lib/` | 进程信息查询库 |
| `libasync_stack` | `libasync_stack.z.so` | `/system/lib/` | 异步栈跟踪库 |
| `libjson_stack_formatter` | `libjson_stack_formatter.z.so` | `/system/lib/` | JSON 格式化库 |
| `crash_exception` | `libcrash_exception.z.so` | `/system/lib/` | 崩溃异常处理库 |
| `dfx_signalhandler` | `libdfx_signalhandler.z.so` | `/system/lib/` | 信号处理库 |
| `kernel_snapshot` | `libkernel_snapshot.z.so` | `/system/lib/` | 内核快照处理库 |
| `libstacktrace_rust` | `libstacktrace_rust.so` | `/system/lib/` | Rust 栈跟踪库 |
| `libpanic_handler` | `libpanic_handler.so` | `/system/lib/` | Rust panic 处理器库 |
| `librustc_demangle` | `librustc_demangle.so` | `/system/lib/` | Rust 符号解析库 |

### 配置文件产物

| 文件名 | 安装路径 | 说明 |
|---------|------------|------|
| `faultloggerd.cfg` | `/system/etc/faultloggerd.cfg` | 服务启动配置 |
| `faultloggerd_config.json` | `/system/etc/faultloggerd_config.json` | 运行时配置 |
| `faultloggerd.para` | `/system/etc/param/faultloggerd.para` | 参数配置 |
| `faultloggerd.para.dac` | `/system/etc/param/faultloggerd.para.dac` | DAC 权限 |
| `faultlogger.conf` | `/system/etc/faultlogger.conf` | 日志显示配置 |
| `fault_coredump.json` | `/system/etc/fault_coredump.json` | Coredump UID 白名单 |

## 运行时加载关系

### 进程启动时加载

```
应用进程启动
    ↓
加载 libdfx_signalhandler.so (signal_handler 库)
    ↓
注册信号处理器
```

**证据**：`interfaces/innerkits/signal_handler` - 应用启动时通过链接加载

### 主动抓栈时加载

```
应用调用 DumpCatcher
    ↓
加载 libdfx_dumpcatcher.so
    ↓
通过 Socket 连接 faultloggerd
    ↓
加载 libunwinder.so
    ↓
加载 libbacktrace_local.so
```

**证据**：`interfaces/innerkits/dump_catcher/BUILD.gn` - 依赖链明确

### 故障日志生成时加载

```
进程崩溃
    ↓
触发信号处理器
    ↓
fork 子进程执行 processdump
    ↓
加载 libunwinder.so
    ↓
加载 libdfx_procinfo.so
    ↓
生成崩溃日志
```

**证据**：`tools/process_dump/BUILD.gn` - processdump 依赖明确

## 关键产物详解

### 1. faultloggerd（守护进程）

**输出位置**：`out/<product>/exe.unstripped/faultloggerd`
**安装路径**：`/system/bin/faultloggerd`
**功能**：
- 监听 Unix Domain Socket（`faultloggerd.server`、`faultloggerd.crash.server`、`faultloggerd.sdkdump.server`）
- 管理临时崩溃日志文件
- 处理客户端请求（文件描述符、SDK dump、pipe 管理）
- 上报 HiSysEvent 事件
- 使用 epoll 管理多客户端连接

**启动方式**：
- 通过 init 配置文件（`faultloggerd.cfg`）
- 守护进程运行

**能力要求**（`services/config/faultloggerd.cfg:22-75`）：
- UID：`faultloggerd`
- GID：`system`、`log`、`faultloggerd`、`readproc`
- Capability：`CAP_DAC_READ_SEARCH`、`CAP_KILL`

### 2. processdump（崩溃 dump 工具）

**输出位置**：`out/<product>/exe.unstripped/processdump`
**安装路径**：`/system/bin/processdump`
**功能**：
- fork 子进程后执行
- 读取父进程 `/proc/[pid]/` 信息
- 调用 unwinder 解析调用栈
- 生成完整的崩溃日志文件
- 支持混合栈（C++ + JS）
- 支持 coredump 生成

**依赖库**：
- `libunwinder.z.so`
- `libdfx_dumpcatcher.z.so`
- `libfaultloggerd.z.so`
- `libasync_stack.z.so`
- `libbacktrace_local.z.so`
- `libdfx_procinfo.z.so`
- `dfx_local_handler_src`（静态库）

**调用链**：
```
processdump
  → libunwinder::UnwindStack()    (符号解析)
  → ProcInfo::GetMaps()              (读取 maps)
  → ProcInfo::GetThreadInfo()         (读取线程信息)
  → FaultLoggerdClient::RequestFileDescriptor()  (申请 FD)
```

### 3. dumpcatcher（主动抓栈工具）

**输出位置**：`out/<product>/exe.unstripped/dumpcatcher`
**安装路径**：`/system/bin/dumpcatcher`
**功能**：
- 命令行抓取指定进程/线程的调用栈
- 支持参数：`-p <pid>`、`-p <pid> -t <tid>`
- 输出到 stdout
- 支持 JSON 格式和文本格式

**依赖库**：
- `libdfx_dumpcatcher.z.so`
- `libfaultloggerd.z.so`
- `libbacktrace_local.z.so`
- `libjson_stack_formatter.z.so`

**权限要求**：
- 调用者必须是 root 或 system 用户
- 或者只能抓取自己用户拥有的进程

### 4. crashvalidator（崩溃验证工具）

**输出位置**：`out/<product>/exe.unstripped/crashvalidator`
**安装路径**：`/system/bin/crashvalidator`
**功能**：
- 验证崩溃日志完整性
- 解析崩溃日志格式

**依赖库**：
- `libfaultloggerd.z.so`
- `libdfx_dumpcatcher.z.so`

## 日志文件路径

### 崩溃临时日志

**路径**：`/data/log/faultlog/temp/`
**命名格式**：`cppcrash-{pid}-{timestamp}`
**示例**：
```
/data/log/faultlog/temp/cppcrash-1234-1501930043627
```

**内容**：
- 进程信息（PID、UID、进程名）
- 故障信息（信号类型、错误码、崩溃地址）
- 调用栈
- 寄存器
- 崩溃栈内存
- 虚拟内存映射

**生命周期**：
- 由 Hiview 提取简易信息后移动到：`/data/log/faultlog/faultlogger/`
- 由 TempFileManager 定期清理

### Coredump 文件

**路径**：`/data/log/faultlog/coredump/`
**命名格式**：`core-{pid}-{timestamp}`
**权限控制**：基于 `fault_coredump.json` UID 白名单

## 运行时依赖

### 库加载顺序

```
应用进程启动
    ↓
[阶段1] libdfx_signalhandler.so (信号处理)
    ↓
[阶段2] libbacktrace_local.so (本地回栈)
    ↓
[阶段3] libdfx_procinfo.so (进程信息)
    ↓
[阶段4] libasync_stack.so (异步栈，可选)
```

### Socket 路径

| Socket 名称 | 用途 |
|-----------|------|
| `/dev/unix/socket/faultloggerd.server` | 通用服务请求 |
| `/dev/unix/socket/faultloggerd.crash.server` | 崩溃日志请求 |
| `/dev/unix/socket/faultloggerd.sdkdump.server` | SDK dump 请求 |

### Hiview 集成

```
faultloggerd 生成崩溃日志
    ↓
Hiview 监听新增文件
    ↓
Hiview 提取简易信息
    ↓
Hiview 生成 HiSysEvent 事件
    ↓
存储到 /data/log/faultlog/faultlogger/
```

## 工具链

### 抓栈工具链

```
dumpcatcher (CLI 工具)
    ↓ [调用]
libdfx_dumpcatcher.so
    ↓ [通过 Socket]
faultloggerd 服务
    ↓ [返回 pipe]
processdump 工具
    ↓ [读取 pipe]
生成调用栈
    ↓ [写入 pipe]
dumpcatcher
    ↓ [显示到 stdout]
调用者
```

## 调试符号

### unstripped vs stripped

**unstripped 目录**：
- `out/<product>/exe.unstripped/faultloggerd`
- 包含调试符号
- 用于符号化崩溃日志

**stripped 目录**：
- `out/<product>/exe/faultloggerd`
- 去除调试符号
- 实际安装到系统

### addr2line 工具使用

**未 stripped 二进制**：
```bash
addr2line -e out/<product>/exe.unstripped/faultloggerd 0x1234
```

**输出**：
```
/path/to/source/file.cpp:123 (discriminator 123)
/path/to/source/file.cpp:456
```

## 安装验证

### 检查服务状态

```bash
ps -A | grep faultloggerd
```

**预期输出**：
```
system       114?        00:00:00   0      0   S    faultloggerd
```

### 检查 Socket 权限

```bash
ls -l /dev/unix/socket/faultloggerd.*
```

**预期输出**：
```
srw-rw-rw- 1 root root 0 Jan 1 00:00 /dev/unix/socket/faultloggerd.server
```

### 检查库文件

```bash
ls -l /system/lib/libdfx_*.so
```

### 检查崩溃日志

```bash
ls -l /data/log/faultlog/temp/
```

## TODO

- [ ] 📋 补充每个产物的详细文件大小
- [ ] 📋 补充启动参数说明
- [ ] 📋 补充环境变量说明

## 相关跳转

- [GN 构建系统](05_GN_Targets.md) - 构建目标和依赖
- [项目定位与核心能力](00_Overview.md) - 项目概览
- [对外 API](03_External_API.md) - API 使用说明
