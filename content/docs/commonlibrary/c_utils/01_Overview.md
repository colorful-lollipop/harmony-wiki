# 项目概览

## 目的

本文档描述 c_utils 部件的项目定位、边界、核心能力与运行环境。

## 适用范围

- OpenHarmony 标准系统开发者
- 需要集成 c_utils 的模块开发者
- 架构师评估技术选型

---

## 项目定位

### 一句话描述

**c_utils 是 OpenHarmony 标准系统的 C++ 公共基础类库，提供跨平台的通用工具类和基础设施。**

### 在系统中的位置

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                      │
├─────────────────────────────────────────────────────────────────┤
│                        框架层 (Framework)                        │
│         Ability Runtime │ ArkUI │ Distributed Scheduler         │
├─────────────────────────────────────────────────────────────────┤
│                        服务层 (Services)                         │
│    SoftBus │ HDF │ PowerMgr │ Notification │ Multimedia ...     │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              公共基础库 (Common Libraries)                  │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │c_utils  │ │hilog    │ │ipc      │ │safwk    │          │  │
│  │  │【本文档】│ │(日志)   │ │(进程通信)│ │(SA框架) │          │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘          │  │
│  └───────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                        内核层 (Kernel)                           │
│              Linux Kernel │ LiteOS │ HDF Driver Framework        │
└─────────────────────────────────────────────────────────────────┘
```

### 与其他部件的关系

| 部件 | 关系 | 说明 |
|------|------|------|
| bounds_checking_function | 依赖 | 使用安全C库函数 |
| hilog | 依赖 | 日志输出（非Android/iOS平台）|
| rust_cxx | 依赖 | Rust FFI 互操作 |
| ipc | 被使用 | Parcel 用于 IPC 数据传输 |
| 各服务子系统 | 被使用 | 大量使用 RefBase, Parcel, 工具类 |

---

## 功能边界

### 包含的功能

| 类别 | 具体功能 | 代码证据 |
|------|----------|----------|
| **内存管理** | 引用计数、智能指针、RAII资源管理 | `refbase.h`, `unique_fd.h` |
| **序列化** | 数据容器、可序列化接口 | `parcel.h`, `flat_obj.h` |
| **并发** | 线程池、读写锁、信号量 | `thread_pool.h`, `rwlock.h`, `semaphore_ex.h` |
| **容器** | 线程安全Map/Queue、有序Vector | `safe_map.h`, `safe_queue.h`, `sorted_vector.h` |
| **文件系统** | 文件/目录操作、内存映射、匿名共享内存 | `file_ex.h`, `directory_ex.h`, `mapped_file.h`, `ashmem.h` |
| **字符串** | 字符串处理、Unicode转换、日期时间 | `string_ex.h`, `unicode_ex.h`, `datetime_ex.h` |
| **事件** | IO事件处理、定时器 | `io_event_*.h`, `timer.h` |
| **设计模式** | 单例、观察者 | `singleton.h`, `observer.h` |
| **错误处理** | 错误码定义框架 | `errors.h`, `common_*_errors.h` |

### 不包含的功能

| 功能 | 原因 | 替代方案 |
|------|------|----------|
| 网络通信 | 超出范围 | 使用 netstack 或 socket 直接编程 |
| 数据库 | 超出范围 | 使用 rdb 或 sqlite |
| 图形渲染 | 超出范围 | 使用 graphic_2d |
| XML/JSON 解析 | 超出范围 | 使用 cjson 或 xml 库 |
| 加密算法 | 超出范围 | 使用 OpenSSL 或 huks |
| N-API 接口 | 纯 C++ 库 | 上层自行封装 |

---

## 核心能力详解

### 1. 内存管理

#### RefBase - 引用计数基类

```cpp
// base/include/refbase.h:42-70
class RefCounter {
    // 强引用计数 + 弱引用计数管理
    // 支持自定义销毁回调
};

template <typename T>
class sptr {  // 强引用智能指针
    // 自动管理对象生命周期
};

template <typename T>
class wptr {  // 弱引用智能指针
    // 不阻止对象销毁，可提升为强引用
};
```

**关键特性**:
- 支持调试模式（DEBUG_REFBASE）跟踪引用
- 支持弱引用提升（AttemptIncStrong）
- 线程安全的引用计数（std::atomic）

#### unique_fd - RAII 资源管理

```cpp
// base/include/unique_fd.h
class unique_fd {
    // 自动关闭文件描述符
    // 支持移动语义
    // 支持自定义关闭函数
};
```

### 2. 序列化（Parcel）

```cpp
// base/include/parcel.h:44-80
class Parcelable : public virtual RefBase {
    virtual bool Marshalling(Parcel &parcel) const = 0;
    // 支持 IPC/RPC 行为标记
};

class Parcel {
    // 数据写入/读取
    // 内存管理（自动扩容）
    // 对象引用处理
};
```

**使用场景**: IPC/RPC 数据传输、跨进程对象传递

### 3. 并发工具

#### ThreadPool

```cpp
// base/include/thread_pool.h
class ThreadPool {
    // 固定大小线程池
    // 任务队列管理
    // 支持优雅关闭
};
```

#### 同步原语

| 类 | 功能 | 实现 |
|----|------|------|
| `RWLock` | 读写锁 | pthread_rwlock |
| `Semaphore` | 信号量 | sem_t |
| `Mutex` | 互斥锁 | std::mutex（容器内部使用）|

### 4. 线程安全容器

```cpp
// base/include/safe_map.h:27-50
template <typename K, typename V>
class SafeMap {
    // std::mutex 保护所有操作
    // 提供 Insert, Find, Erase, Iterate 等接口
};
```

**注意**: 迭代时持有锁，不适合长时间操作。

### 5. 文件系统

```cpp
// base/include/file_ex.h
bool LoadStringFromFile(const std::string& filePath, std::string& content);
bool SaveStringToFile(const std::string& filePath, const std::string& content);

// base/include/mapped_file.h
class MappedFile {
    // 内存映射文件操作
    // 支持只读/读写模式
};

// base/include/ashmem.h
class Ashmem {
    // 匿名共享内存
    // 支持跨进程共享
};
```

---

## 运行环境

### 支持的平台

| 平台 | 支持程度 | 限制 |
|------|----------|------|
| **OHOS/Linux** | 完整支持 | 无限制 |
| **Windows** | 受限支持 | 仅 Parcel, RefBase, String |
| **macOS** | 受限支持 | 仅 Parcel, RefBase, String |
| **iOS** | 受限支持 | Directory, Parcel, RefBase, RWLock, String |
| **Android** | 受限支持 | 无 hilog |

### 平台适配代码

```gn
# base/BUILD.gn:31-45
if (current_os == "ios") { defines += [ "IOS_PLATFORM" ] }
if (current_os == "win" || current_os == "mingw") { defines += [ "WINDOWS_PLATFORM" ] }
if (current_os == "mac") { defines += [ "MAC_PLATFORM" ] }
if (current_os == "ohos") { defines += [ "OHOS_PLATFORM" ] }
```

### 系统要求

- **最低版本**: OpenHarmony 3.1（标准系统）
- **CPU 架构**: arm, arm64, x86_64
- **编译器**: Clang（支持 C++11/14/17）

---

## 关键概念

### 1. 强引用 vs 弱引用

| 特性 | 强引用 (sptr) | 弱引用 (wptr) |
|------|---------------|---------------|
| 阻止销毁 | 是 | 否 |
| 直接访问 | 是（operator->） | 否（需提升）|
| 使用场景 | 正常持有对象 | 打破循环引用 |
| 生命周期 | 与对象相同 | 可长于对象 |

### 2. Parcel 数据布局

```
┌──────────────────────────────────────────────────────┐
│ Parcel Data Layout                                   │
├──────────────────────────────────────────────────────┤
│ [Header] 数据大小、对象数量、偏移量表                  │
├──────────────────────────────────────────────────────┤
│ [Data Section] 原始数据（int, string, array...）      │
├──────────────────────────────────────────────────────┤
│ [Object Section] Binder对象引用（用于IPC）             │
├──────────────────────────────────────────────────────┤
│ [Padding] 对齐填充                                   │
└──────────────────────────────────────────────────────┘
```

### 3. 线程安全级别

| 级别 | 说明 | 示例 |
|------|------|------|
| **线程安全** | 多线程可直接使用 | SafeMap, SafeQueue, RWLock |
| **非线程安全** | 需外部同步 | Parcel（单线程使用）|
| **const 线程安全** | 只读操作线程安全 | string_ex 函数 |

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [架构说明](03_Architecture.md) - 详细架构设计
- [对外 API](04_Public_API.md) - 接口使用指南
- [GN Targets](06_GN_Targets.md) - 构建配置
