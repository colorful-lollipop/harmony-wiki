# 目录结构

## 目的

本文档描述 c_utils 的代码组织结构，帮助开发者快速定位代码。

## 适用范围

- 新加入的开发者熟悉代码库
- 模块集成时查找依赖
- 代码审查时理解上下文

---

## 顶层目录

```
commonlibrary/c_utils/
├── base/                    # 主要代码目录
│   ├── include/            # 对外接口头文件（32个）
│   ├── src/                # 源文件实现（24个cpp）
│   └── test/               # 测试代码（本文档忽略）
├── docs/                   # 官方开发文档
│   ├── en/                 # 英文文档（2篇）
│   └── zh-cn/              # 中文文档（22篇）
├── bundle.json             # 部件配置（组件名、依赖、feature flags）
├── BUILD.gn               # GN构建入口
├── Cargo.toml             # Rust配置
├── LICENSE                # Apache 2.0
└── README.md              # 项目说明
```

---

## Include 目录详解

**路径**: `base/include/`

### 按功能分类

#### 内存管理（4个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `refbase.h` | 引用计数基类 | RefBase, sptr<T>, wptr<T> |
| `unique_fd.h` | FD自动管理 | unique_fd, unique_file, unique_dir, unique_map |
| `flat_obj.h` | 扁平对象 | FlatObject |
| `nocopyable.h` | 禁止拷贝宏 | NoCopyable |

#### 序列化（2个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `parcel.h` | 数据序列化 | Parcel, Parcelable |
| `pubdef.h` | 公共定义 | 平台宏、基础类型 |

#### 并发与同步（6个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `thread_pool.h` | 线程池 | ThreadPool |
| `rwlock.h` | 读写锁 | RWLock, ReadLockGuard, WriteLockGuard |
| `semaphore_ex.h` | 信号量 | Semaphore |
| `thread_ex.h` | 线程增强 | ThreadEx |
| `safe_map.h` | 线程安全Map | SafeMap<K,V> |
| `safe_queue.h` | 线程安全队列 | SafeQueue<T> |
| `safe_block_queue.h` | 阻塞队列 | SafeBlockQueue<T> |

#### 容器（2个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `sorted_vector.h` | 有序Vector | SortedVector<T> |
| `singleton.h` | 单例模式 | Singleton<T> |

#### 文件系统（5个）

| 头文件 | 功能 | 关键类/函数 |
|--------|------|-------------|
| `file_ex.h` | 文件操作 | LoadStringFromFile, SaveStringToFile |
| `directory_ex.h` | 目录操作 | CreateDirectory, GetDirFiles |
| `mapped_file.h` | 内存映射 | MappedFile |
| `ashmem.h` | 匿名共享内存 | Ashmem |
| `datetime_ex.h` | 日期时间 | GetCurrentTime, FormatTime |

#### 字符串（2个）

| 头文件 | 功能 | 关键函数 |
|--------|------|----------|
| `string_ex.h` | 字符串增强 | StrToInt, Trim, Split, ReplaceStr |
| `unicode_ex.h` | Unicode转换 | UTF8ToUTF16, UTF16ToUTF8 |

#### 事件系统（4个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `timer.h` | 定时器 | Timer, TimerCallback |
| `io_event_handler.h` | IO事件处理 | IOEventHandler |
| `io_event_reactor.h` | IO事件反应器 | IOEventReactor |
| `io_event_common.h` | 事件类型定义 | IOEventType |

#### 设计模式（1个）

| 头文件 | 功能 | 关键类 |
|--------|------|--------|
| `observer.h` | 观察者模式 | Observer, Observable |

#### 错误处理（5个）

| 头文件 | 功能 |
|--------|------|
| `errors.h` | 基础错误码框架 |
| `common_errors.h` | 通用错误码 |
| `common_timer_errors.h` | 定时器错误码 |
| `common_event_sys_errors.h` | 事件系统错误码 |
| `common_mapped_file_errors.h` | 文件映射错误码 |

---

## Source 目录详解

**路径**: `base/src/`

### 源文件与头文件对应关系

| 源文件 | 对应头文件 | 功能说明 |
|--------|------------|----------|
| `refbase.cpp` | `refbase.h` | 引用计数实现（~20KB）|
| `parcel.cpp` | `parcel.h` | 序列化实现（~42KB）|
| `thread_pool.cpp` | `thread_pool.h` | 线程池实现 |
| `rwlock.cpp` | `rwlock.h` | 读写锁实现 |
| `semaphore_ex.cpp` | `semaphore_ex.h` | 信号量实现 |
| `observer.cpp` | `observer.h` | 观察者实现 |
| `file_ex.cpp` | `file_ex.h` | 文件操作实现（~11KB）|
| `directory_ex.cpp` | `directory_ex.h` | 目录操作实现（~16KB）|
| `mapped_file.cpp` | `mapped_file.h` | 内存映射实现（~18KB）|
| `ashmem.cpp` | `ashmem.h` | 匿名共享内存实现 |
| `string_ex.cpp` | `string_ex.h` | 字符串处理实现 |
| `unicode_ex.cpp` | - | Unicode转换（内部使用）|
| `datetime_ex.cpp` | `datetime_ex.h` | 日期时间实现 |
| `thread_ex.cpp` | `thread_ex.h` | 线程增强实现 |
| `timer.cpp` | `timer.h` | 定时器实现 |
| `timer_event_handler.cpp` | - | 定时器事件处理 |
| `io_event_handler.cpp` | `io_event_handler.h` | IO事件处理 |
| `io_event_reactor.cpp` | `io_event_reactor.h` | IO事件反应器（~11KB）|
| `io_event_epoll.cpp` | `io_event_epoll.h` | Epoll实现（Linux）|
| `event_handler.cpp` | `event_handler.h` | 通用事件处理 |
| `event_reactor.cpp` | `event_reactor.h` | 通用事件反应器 |
| `event_demultiplexer.cpp` | `event_demultiplexer.h` | 事件多路分解 |

### 内部头文件（不对外暴露）

| 头文件 | 用途 |
|--------|------|
| `unicode_ex.h` | Unicode转换内部定义 |
| `utils_log.h` | 日志宏定义 |
| `timer_event_handler.h` | 定时器事件处理器内部定义 |
| `event_handler.h` | 事件处理器内部定义 |
| `event_reactor.h` | 事件反应器内部定义 |
| `event_demultiplexer.h` | 事件多路分解器内部定义 |
| `io_event_epoll.h` | Epoll特定定义 |

### Rust 源码

**路径**: `base/src/rust/`

| 文件 | 功能 |
|------|------|
| `lib.rs` | Rust库入口 |
| `ashmem.rs` | 匿名共享内存Rust绑定 |
| `directory_ex.rs` | 目录操作Rust绑定 |
| `file_ex.rs` | 文件操作Rust绑定 |
| `Cargo.toml` | Rust包配置 |

---

## 模块职责

### 核心模块划分

```
┌─────────────────────────────────────────────────────────────────┐
│                        c_utils 模块架构                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐         │
│  │   基础工具     │ │   内存管理     │ │   并发工具     │         │
│  │ ├─ string_ex  │ │ ├─ refbase    │ │ ├─ thread_pool│         │
│  │ ├─ datetime   │ │ ├─ unique_fd  │ │ ├─ rwlock     │         │
│  │ ├─ unicode    │ │ └─ flat_obj   │ │ ├─ semaphore  │         │
│  │ └─ errors     │ │               │ │ └─ safe_*     │         │
│  └───────────────┘ └───────────────┘ └───────────────┘         │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐         │
│  │   文件系统     │ │   序列化       │ │   事件系统     │         │
│  │ ├─ file_ex    │ │ ├─ parcel     │ │ ├─ timer      │         │
│  │ ├─ directory  │ │ └─ parcelable │ │ ├─ io_event   │         │
│  │ ├─ mapped_file│ │               │ │ └─ event_*    │         │
│  │ └─ ashmem     │ │               │ │               │         │
│  └───────────────┘ └───────────────┘ └───────────────┘         │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐ ┌───────────────┐                           │
│  │   设计模式     │ │   跨语言       │                           │
│  │ ├─ singleton  │ │ └─ rust/*     │                           │
│  │ └─ observer   │ │               │                           │
│  └───────────────┘ └───────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

### 依赖关系

```
                    ┌──────────────┐
                    │    errors    │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  refbase     │◄──│   parcel     │   │  safe_*      │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ file_ex      │  │ thread_pool  │  │ io_event     │
│ directory_ex │  │ rwlock       │  │ timer        │
│ mapped_file  │  │ semaphore    │  │ observer     │
│ ashmem       │  └──────────────┘  └──────────────┘
└──────────────┘
```

---

## 关键文件路径速查

### 常用接口

| 功能 | 头文件路径 | 源文件路径 |
|------|------------|------------|
| 智能指针 | `base/include/refbase.h` | `base/src/refbase.cpp` |
| 序列化 | `base/include/parcel.h` | `base/src/parcel.cpp` |
| 文件操作 | `base/include/file_ex.h` | `base/src/file_ex.cpp` |
| 目录操作 | `base/include/directory_ex.h` | `base/src/directory_ex.cpp` |
| 线程池 | `base/include/thread_pool.h` | `base/src/thread_pool.cpp` |
| 线程安全Map | `base/include/safe_map.h` | （模板，无cpp）|
| 定时器 | `base/include/timer.h` | `base/src/timer.cpp` |

### 配置文件

| 文件 | 路径 | 用途 |
|------|------|------|
| 部件配置 | `bundle.json` | 组件元数据、依赖、feature flags |
| 构建配置 | `base/BUILD.gn` | GN构建规则、targets、编译选项 |
| Rust配置 | `base/src/rust/Cargo.toml` | Rust包配置 |
| 根Rust配置 | `Cargo.toml` | 工作区配置 |

---

## 相关跳转

- [项目概览](01_Overview.md) - 功能定位与边界
- [架构说明](03_Architecture.md) - 详细架构设计
- [对外 API](04_Public_API.md) - 接口使用指南
