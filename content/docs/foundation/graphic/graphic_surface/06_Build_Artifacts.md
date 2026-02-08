# 编译产物

## 目的
本文档说明 graphic_surface 的编译产物，包括输出文件、安装路径、运行时加载关系和产物用途。

## 适用范围
面向需要：
- 了解构建输出的开发者
- 进行系统集成的工程师
- 分析库依赖关系的架构师
- 部署和测试 OpenHarmony 系统的运维工程师

## 产物清单

### 核心共享库（.so）

| 产物 | Target | 大小（估计） | 用途 |
|------|---------|--------------|------|
| `libsurface.so` | `surface:surface` | ~800KB | Surface 核心库（生产者/消费者） |
| `libsync_fence.so` | `sync_fence:sync_fence` | ~200KB | 同步栅栏库（GPU/CPU 同步） |
| `libbuffer_handle.so` | `buffer_handle:buffer_handle` | ~50KB | Buffer 句柄库（IPC 序列化） |

### 符号链接（Compatibility Links）

| 符号链接 | 目标 | 用途 |
|---------|------|------|
| `libnative_buffer.so` → `libsurface.so` | `surface:surface` | Native Buffer C API 兼容性 |
| `libnative_window.so` → `libsurface.so` | `surface:surface` | Native Window C API 兼容性 |
| `libnative_fence.so` → `libsync_fence.so` | `sync_fence:sync_fence` | Native Fence C API 兼容性 |

### 静态库（.a）

| 产物 | Target | 大小（估计） | 用途 |
|------|---------|--------------|------|
| `libsurface_static.a` | `surface:surface_static` | ~750KB | Surface 静态版本（静态链接） |
| `libsync_fence_static.a` | `sync_fence:sync_fence_static` | ~180KB | SyncFence 静态版本 |
| `libbuffer_handle_static.a` | `buffer_handle:buffer_handle_static` | ~40KB | BufferHandle 静态版本 |
| `libframe_report.a` | `frame_report:frame_report` | ~30KB | 帧上报工具 |
| `libhebc_white_list.a` | `hebc_white_list:hebc_white_list` | ~20KB | HEBC 白名单工具 |
| `libsandbox_utils.a` | `sandbox:sandbox_utils` | ~10KB | 沙箱工具 |

## 产物详细信息

### libsurface.so（Surface 核心库）

#### 库信息
```bash
$ readelf -h /usr/lib/libsurface.so
ELF Header:
  Magic:   7f 45 4c 46 01
  Class:                           64-bit
  Type:                            Shared object file
```

#### 导出符号（示例）
```
# Surface 类
CreateSurface
CreateSurfaceAsConsumer
CreateSurfaceAsProducer

# 生产者/消费者接口
ProducerSurface::RequestBuffer
ProducerSurface::FlushBuffer
ConsumerSurface::AcquireBuffer
ConsumerSurface::ReleaseBuffer

# IPC 接口
IBufferProducer::Connect
IBufferProducer::Disconnect
IBufferProducer::RequestBuffer
# ... 50+ 方法
```

#### 依赖关系
```
libsurface.so 依赖：
  ├─ libbuffer_handle.so
  ├─ libsync_fence.so
  ├─ libsandbox_utils.a
  ├─ libframe_report.a
  ├─ libhebc_white_list.a
  ├─ librs_frame_report_ext.a
  ├─ libc.so
  ├─ libhilog.so
  ├─ libhitrace.so
  ├─ libeventhandler.so
  ├─ libipc.so
  └─ libdrivers_interface_display*.so
```

#### 运行时依赖
```bash
$ ldd /usr/lib/libsurface.so
  linux-vdso.so.1  =>  (0x00007fff0000)
  libbuffer_handle.so => /usr/lib/libbuffer_handle.so
  libsync_fence.so => /usr/lib/libsync_fence.so
  libhilog.so => /usr/lib/libhilog.so
  libhitrace.so => /usr/lib/libhitrace.so
  # ...
```

#### 安装路径
```
/usr/lib/libsurface.so                      # 标准 OpenHarmony 系统路径
/usr/lib64/libsurface.so                    # 64 位系统路径
/system/lib/libsurface.so                    # 部分设备上的系统分区路径
/vendor/lib/libsurface.so                    # 厂商特定路径
```

#### 符号链接
```bash
$ ls -la /usr/lib/libnative_*.so
lrwxrwxrwx 1 root root libnative_buffer.so -> libsurface.so
lrwxrwxrwx 1 root root libnative_window.so -> libsurface.so
```

**用途**：提供 NDK 兼容性，允许 Native 应用使用标准 C API 名称。

### libsync_fence.so（SyncFence 库）

#### 库信息
```bash
$ readelf -h /usr/lib/libsync_fence.so
ELF Header:
  Magic:   7f 45 4c 46 01
  Class:                           64-bit
  Type:                            Shared object file
```

#### 导出符号（示例）
```
# SyncFence 类
SyncFence::Create
SyncFence::Destroy
SyncFence::Wait
SyncFence::WaitAndReset
SyncFence::Merge
SyncFence::Clone

# Native Fence C API
NativeFenceCreate
NativeFenceDestroy
NativeFenceWait
# ...
```

#### 依赖关系
```
libsync_fence.so 依赖：
  ├─ libc.so
  ├─ libhilog.so
  ├─ libhisysevent.so
  ├─ libeventhandler.so
  ├─ libipc.so
  └─ libinit.so
```

#### 安装路径
```
/usr/lib/libsync_fence.so
/usr/lib64/libsync_fence.so
```

#### 符号链接
```bash
$ ls -la /usr/lib/libnative_fence.so
lrwxrwxrwx 1 root root libnative_fence.so -> libsync_fence.so
```

### libbuffer_handle.so（BufferHandle 库）

#### 库信息
```bash
$ readelf -h /usr/lib/libbuffer_handle.so
ELF Header:
  Magic:   7f 45 4c 46 01
  Class:                           64-bit
  Type:                            Shared object file
```

#### 导出符号（示例）
```
# BufferHandle 序列化/反序列化
WriteBufferHandle
ReadBufferHandle

# BufferHandle 工具
GetBufferHandleSize
IsBufferHandleValid
```

#### 依赖关系
```
libbuffer_handle.so 依赖：
  ├─ libc.so
  ├─ libcutils.so
  ├─ libhilog.so
  └─ libipc.so
```

#### 安装路径
```
/usr/lib/libbuffer_handle.so
/usr/lib64/libbuffer_handle.so
```

## 运行时加载关系

### 典型加载顺序

#### 场景 1：UI 应用（生产者）
```
1. 应用启动
   ↓
2. 加载 @ohos.window N-API（在高层框架）
   ↓
3. N-API 加载 libnative_window.so（符号链接）
   ↓ 实际加载 libsurface.so
   ↓
4. libsurface.so 加载依赖：
   ├─ libbuffer_handle.so
   ├─ libsync_fence.so
   └─ libhilog.so, libipc.so, ...
   ↓
5. 应用调用 Surface API
   - RequestBuffer()
   - FlushBuffer()
   ↓
6. Surface 通过 IPC 调用消费者（WindowManager）
```

#### 场景 2：WindowManager（消费者）
```
1. WindowManager 启动
   ↓
2. 加载 libsurface.so
   ↓
3. 创建 ConsumerSurface（IConsumerSurface::Create）
   ↓
4. ConsumerSurface 加载依赖：
   ├─ libbuffer_handle.so
   ├─ libsync_fence.so
   └─ libhilog.so, libipc.so, ...
   ↓
5. 注册到 SystemAbility（如果使用 SA 模式）
   ↓
6. 提供生产者接口给应用（IBufferProducer）
```

### 依赖树（运行时）

```
应用层（Native/C++）
  │
  ├─ libnative_window.so (符号链接)
  │   └─► libsurface.so
  │        ├─► libbuffer_handle.so
  │        ├─► libsync_fence.so
  │        ├─► libhilog.so
  │        ├─► libhitrace.so
  │        ├─► libipc.so
  │        └─► libeventhandler.so
  │
  └─ libc.so (系统库）

系统服务（WindowManager/RS）
  │
  └─► libsurface.so
       ├─► libbuffer_handle.so
       ├─► libsync_fence.so
       └─► ...（同上）
```

## 库使用场景

### 场景 1：原生应用渲染

**库使用**：
```
Native 应用（C++）
  └─► libnative_window.so (符号链接)
       └─► libsurface.so
            ├─ RequestBuffer() ──► 获取 Buffer
            └─ FlushBuffer() ──► 提交 Buffer
```

**流程**：
1. 应用调用 `OHNativeWindow_CreateSurface()`
2. 应用调用 `OHNativeWindow_RequestBuffer()`
3. libsurface.so 分配 Buffer（通过 BufferHandle）
4. 应用渲染到 Buffer
5. 应用调用 `OHNativeWindow_NativeWindowFlushBuffer()`
6. libsurface.so 通过 IPC 传输 Buffer 到 WindowManager

### 场景 2：跨进程 Buffer 共享

**库使用**：
```
生产者进程（应用）
  └─► libsurface.so
       └─► BufferClientProducer (IPC 客户端）
            └─► SendRequest(IBufferProducer) ──► Binder IPC

消费者进程（WindowManager）
  └─► libsurface.so
       └─► BufferQueueProducer (IPC 服务端）
            └─► OnRemoteRequest(IBufferProducer) ◄── Binder IPC
```

**数据流**：
1. Producer 调用 `RequestBuffer()`
2. BufferClientProducer 序列化参数到 MessageParcel
3. Binder IPC 传输到 BufferQueueProducer
4. BufferQueueProducer 反序列化并调用 BufferQueue
5. BufferQueue 从 freeList_ 获取或分配 Buffer
6. BufferHandle 通过 MessageParcel 传回（包含 fd）
7. Binder IPC fd passing 传输共享内存句柄
8. Producer 获取 BufferHandle 和 fd
9. Producer 写入共享内存
10. Producer 调用 `FlushBuffer()`
11. Buffer 通过 IPC 传回 BufferQueue
12. BufferQueue 放入 dirtyList_
13. Consumer 通过 `AcquireBuffer()` 获取 Buffer
14. Consumer 从共享内存读取并合成

### 场景 3：GPU 同步

**库使用**：
```
应用
  └─► libsurface.so
       └─► RequestBuffer(buffer, acquireFence) ──►

GPU HAL
  └─► Render(buffer)
       └─► SetFence(gpuFence) ◄─── GPU 完成信号

应用
  └─► gpuFence->Wait() ──►

WindowManager（消费者）
  └─► libsurface.so
       └─► AcquireBuffer(buffer, acquireFence)
       └─► acquireFence->Wait() ──► 等待 GPU 完成
       └─► Composite(buffer)
       └─► Display(buffer)
       └─► SetFence(displayFence) ◄─── 显示完成信号
       └─► ReleaseBuffer(buffer, releaseFence)
```

## 库配置与优化

### Strip 符号表（Release 构建）

Release 构建会移除符号表以减小库大小：

```bash
$ ls -lh /usr/lib/libsurface.so
-rw-r--r-- 1 root root 850K  # Release 版本

$ ls -lh /out/debug/libsurface.so
-rw-r--r-- 1 root root 2.1M  # Debug 版本（含符号表）
```

### PIC（位置无关代码）

所有共享库编译为 PIC 以支持运行时加载：

```bash
$ readelf -h /usr/lib/libsurface.so | grep PIC
  Type:                              DYN (Shared object file)
  Flags:                              0x00000000
  ... PIC 标志通常存在
```

### Soname（共享对象名称）

```bash
$ readelf -d /usr/lib/libsurface.so | grep SONAME
  0x0000000000000000e (SONAME)              Library soname: [libsurface.so.4.1]
```

**版本信息**：`libsurface.so.4.1` 表示版本 4.1

### NEEDED（动态依赖）

```bash
$ readelf -d /usr/lib/libsurface.so | grep NEEDED
  0x000000000000000f (NEEDED)             Shared library: [libbuffer_handle.so]
  0x0000000000000010 (NEEDED)             Shared library: [libsync_fence.so]
  0x0000000000000011 (NEEDED)             Shared library: [libhilog.so]
  0x0000000000000012 (NEEDED)             Shared library: [libhitrace.so]
  # ... 其他依赖
```

## 版本兼容性

### ABI 兼容性

graphic_surface v4.1 保证以下 ABI 兼容性：

| 版本 | 兼容性 | 说明 |
|------|---------|------|
| 4.0 → 4.1 | ✅ 向后兼容 | 新增功能，不破坏现有 API |
| 4.1 → 4.0 | ⚠️ 不兼容 | 新版本可能使用新特性 |

**关键兼容性原则**：
1. 不移除现有符号
2. 不修改符号签名
3. 新增符号不影响现有功能

### NDK 兼容性

符号链接提供 NDK 兼容性：

| C API | 符号链接 | 目标库 |
|-------|---------|---------|
| Native Window | `libnative_window.so` | `libsurface.so` |
| Native Buffer | `libnative_buffer.so` | `libsurface.so` |
| Native Fence | `libnative_fence.so` | `libsync_fence.so` |

**兼容性保证**：
- Native 应用使用标准名称链接
- 不依赖具体版本号
- 跨版本兼容

## 库大小分析

### 各模块大小估算

| 模块 | 代码大小 | 符号表大小 | 总大小（Strip 后） |
|--------|---------|------------|------------------|
| Surface | ~600KB | ~50KB | ~850KB |
| SyncFence | ~150KB | ~20KB | ~200KB |
| BufferHandle | ~30KB | ~5KB | ~50KB |
| **总计** | **~780KB** | **~75KB** | **~1.1MB** |

### 内存占用（运行时）

运行时内存占用（近似）：

| 组件 | 常驻内存 | 峰值内存 |
|--------|----------|----------|
| Surface（生产者） | ~500KB | ~2MB（含 Buffer） |
| Surface（消费者） | ~600KB | ~10MB（含 Buffer 队列） |
| SyncFence | ~100KB | ~500KB（含 Fence 缓存） |
| **总计** | **~1.2MB** | **~12.5MB** |

**注**：实际内存占用取决于：
- Buffer 队列大小
- Buffer 分配数量
- Fence 缓存大小
- 是否使用 HEBC

## 库验证与测试

### 符号完整性检查

```bash
# 检查导出符号
$ nm -D /usr/lib/libsurface.so | grep Surface

# 预期符号：
CreateSurface
CreateSurfaceAsConsumer
CreateSurfaceAsProducer
# ... 其他关键符号
```

### 依赖完整性检查

```bash
# 检查运行时依赖
$ ldd /usr/lib/libsurface.so

# 预期依赖：
libbuffer_handle.so => /usr/lib/libbuffer_handle.so
libsync_fence.so => /usr/lib/libsync_fence.so
libhilog.so => /usr/lib/libhilog.so
# ...
```

### 功能测试

**使用单元测试**：
```bash
# 运行 Surface 单元测试
./surface_test_unittest

# 预期：
PASS: BufferQueueTest.RequestBuffer
PASS: BufferQueueTest.FlushBuffer
# ... 所有测试通过
```

**使用集成测试**：
```bash
# 跨进程测试
./surface_ipc_test

# 预期：
Create consumer surface: OK
Create producer surface: OK
Request/Flush buffer: OK
# ... 测试通过
```

## 常见问题

### 问题 1：库加载失败

**症状**：
```
dlopen failed: libsurface.so: cannot open shared object file
```

**原因**：
1. 库未正确安装
2. 路径不在 LD_LIBRARY_PATH
3. 依赖库缺失

**解决方案**：
```bash
# 1. 检查库是否存在
ls -la /usr/lib/libsurface.so

# 2. 检查符号链接
ls -la /usr/lib/libnative_*.so

# 3. 检查依赖
ldd /usr/lib/libsurface.so

# 4. 设置库路径
export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
```

### 问题 2：符号未定义

**症状**：
```
undefined reference to 'CreateSurfaceAsConsumer'
```

**原因**：
1. 链接错误的库版本
2. 头文件与库不匹配

**解决方案**：
```bash
# 1. 检查导出符号
nm -D /usr/lib/libsurface.so | grep CreateSurfaceAsConsumer

# 2. 检查头文件版本
grep CreateSurfaceAsConsumer interfaces/inner_api/surface/surface.h

# 3. 重新编译（使用正确的库版本）
```

### 问题 3：运行时崩溃

**症状**：
应用启动后立即崩溃

**原因**：
1. 依赖库版本不匹配
2. Feature Flag 未正确设置

**解决方案**：
```bash
# 1. 检查崩溃栈
gdb -p [pid] -batch -ex "bt" -ex "quit"

# 2. 查看日志
hdc shell hilog -T Graphic -e

# 3. 验证 Feature Flags
# 检查使用的功能是否在编译时启用
```

## 相关跳转
- [GN Targets 与编译产物](05_GN_Targets.md) - Target 定义与依赖
- [目录结构与模块职责](01_Directory_Structure.md) - BUILD.gn 文件位置
- [对外 API](03_External_API.md) - 库提供的 API
- [常见问题](08_Troubleshooting.md) - 加载问题调试
