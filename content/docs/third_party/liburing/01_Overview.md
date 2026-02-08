# 01 - 原始库简介与 OpenHarmony 定位

本文档介绍 liburing 原始库的功能特性，以及在 OpenHarmony 系统中的作用定位。

---

## 1. 原始库简介

### 1.1 基本信息

| 属性 | 详情 |
|-----|------|
| **库名称** | liburing |
| **版本** | 2.7 |
| **许可证** | MIT / LGPL (双许可) |
| **上游维护者** | Jens Axboe (Linux 内核 I/O 子系统维护者) |
| **上游地址** | https://github.com/axboe/liburing |
| **官方文档** | https://kernel.dk/io_uring.pdf |

### 1.2 功能概述

**liburing** 是 Linux [io_uring](https://kernel.dk/io_uring.pdf) 异步 I/O 接口的官方用户态封装库。它提供了：

1. **简化的初始化/销毁接口** - 封装复杂的 io_uring 设置流程
2. **丰富的操作封装** - 提供各类 I/O 操作的便捷准备函数
3. **高效的内核通信** - 优化系统调用路径，支持批量提交
4. **零拷贝支持** - 支持 splice、registered buffers 等高级特性

### 1.3 io_uring 核心概念

io_uring 是 Linux 内核 5.1+ 引入的高性能异步 I/O 子系统，核心设计：

```
┌─────────────────────────────────────────────────────────────┐
│                        应用程序                               │
├─────────────────────────────────────────────────────────────┤
│  提交队列 (SQ)        │        完成队列 (CQ)                  │
│  ┌───────────────┐    │    ┌───────────────┐                 │
│  │ io_uring_sqe  │────┼───▶│ io_uring_cqe  │                 │
│  │ (Submission   │    │    │ (Completion   │                 │
│  │  Queue Entry) │    │    │  Queue Entry) │                 │
│  └───────────────┘    │    └───────────────┘                 │
├─────────────────────────────────────────────────────────────┤
│                     共享内存映射                              │
├─────────────────────────────────────────────────────────────┤
│                      内核 io_uring                            │
│                   ┌──────────────┐                           │
│                   │  异步执行 I/O │                           │
│                   └──────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

**关键特点**:
- **共享内存队列**: 用户态与内核通过共享内存通信，避免系统调用开销
- **批量处理**: 支持批量提交 SQE 和批量收割 CQE
- **Polling 模式**: 支持内核轮询模式，实现零中断 I/O
- **无需上下文切换**: 提交和完成不经过系统调用（批量模式下）

### 1.4 支持的操作类型

liburing 支持的操作覆盖几乎所有 I/O 场景：

#### 文件 I/O
| 操作 | 函数 | 说明 |
|-----|------|------|
| 读 | `io_uring_prep_read` | 异步读取 |
| 写 | `io_uring_prep_write` | 异步写入 |
| 向量读 | `io_uring_prep_readv` | 分散读取 |
| 向量写 | `io_uring_prep_writev` | 聚集写入 |
| 预读建议 | `io_uring_prep_fadvise` | 文件预读提示 |

#### 网络 I/O
| 操作 | 函数 | 说明 |
|-----|------|------|
| 接受连接 | `io_uring_prep_accept` | 异步 accept |
| 连接 | `io_uring_prep_connect` | 异步 connect |
| 发送 | `io_uring_prep_send` | 异步发送 |
| 接收 | `io_uring_prep_recv` | 异步接收 |
| 多播接收 | `io_uring_prep_recv_multishot` | 单次提交多次接收 |

#### 文件系统操作
| 操作 | 函数 | 说明 |
|-----|------|------|
| 打开 | `io_uring_prep_openat` | 异步打开文件 |
| 关闭 | `io_uring_prep_close` | 异步关闭 |
| 创建目录 | `io_uring_prep_mkdirat` | 异步 mkdir |
| 删除 | `io_uring_prep_unlinkat` | 异步删除 |
| 重命名 | `io_uring_prep_renameat` | 异步重命名 |
| 创建链接 | `io_uring_prep_linkat` | 异步硬链接 |
| 状态获取 | `io_uring_prep_statx` | 异步 stat |

#### 高级特性
| 操作 | 函数 | 说明 |
|-----|------|------|
| 管道转发 | `io_uring_prep_splice` | 零拷贝管道传输 |
| 缓冲区注册 | `io_uring_register_buffers` | 注册固定内存 |
| 文件表注册 | `io_uring_register_files` | 注册文件描述符 |
| 事件轮询 | `io_uring_prep_poll_add` | 异步 epoll 替代 |
| Futex 同步 | `io_uring_prep_futex_wait` | 用户态同步原语 |

### 1.5 版本 2.7 新特性

liburing 2.7 同步 Linux 内核 6.10，新增：

1. **send/recv bundle 支持** - 批量发送/接收多个数据包
2. **accept nowait** - 非阻塞 accept 支持 CQE_F_MORE 标志
3. **BIND/LISTEN 支持** - 套接字绑定和监听操作
4. **64 位 advise** - 支持大文件的 fadvise/madvise
5. **内存修复** - 修复 io_uring_queue_init_mem() 返回值问题

---

## 2. OpenHarmony 定位

### 2.1 在 OH 中的角色

**当前状态**: 基础设施型第三方库

```
┌──────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                           │
├──────────────────────────────────────────────────────────────┤
│                    应用层 (Applications)                      │
│                     (可能通过 dlopen 使用)                     │
├──────────────────────────────────────────────────────────────┤
│                    框架层 (Framework)                         │
│              TODO(需确认): 是否有框架模块依赖                    │
├──────────────────────────────────────────────────────────────┤
│                    系统服务 (System Services)                 │
│              TODO(需确认): 是否有服务使用 liburing              │
├──────────────────────────────────────────────────────────────┤
│                    内核层 (Kernel)                            │
│              ┌─────────────────────────────────┐            │
│              │        Linux io_uring            │            │
│              │  (需要内核 5.1+ 支持)             │            │
│              └─────────────────────────────────┘            │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 适用场景分析

#### 适用场景 ✅

1. **高性能网络服务器**
   - 场景: Web 服务器、游戏服务器、实时通信服务
   - 优势: 单线程处理数万并发连接
   - OH 应用: 系统服务、分布式软总线

2. **高速存储访问**
   - 场景: 数据库、日志系统、缓存服务
   - 优势: 充分利用 NVMe SSD 低延迟特性
   - OH 应用: 文件系统服务、数据库引擎

3. **多媒体处理**
   - 场景: 视频流、音频处理、图像编码
   - 优势: 零拷贝传输、确定性延迟
   - OH 应用: 多媒体框架、相机服务

4. **实时数据处理**
   - 场景: 传感器数据、IoT 数据采集
   - 优势: 可预测的性能表现
   - OH 应用: 物联网服务、边缘计算

#### 不适用场景 ❌

1. **简单 I/O 操作** - 同步 I/O 更简单且足够
2. **低并发应用** - 开销可能超过收益
3. **旧内核系统** - 需要 Linux 5.1+ 
4. **无持久化需求** - 纯内存操作无需 io_uring

### 2.3 与 OH 其他 I/O 方案的对比

| 方案 | 复杂度 | 性能 | 适用场景 | OH 现状 |
|-----|-------|------|---------|--------|
| **同步 I/O** | 低 | 低 | 简单应用 | 广泛使用 |
| **epoll** | 中 | 中 | 网络并发 | 标准方案 |
| **io_uring** | 高 | 极高 | 高性能 I/O | 预备支持 |
| **AIO** | 高 | 中 | 遗留系统 | 不推荐 |

### 2.4 OH 集成的特殊考虑

#### 优势
- ✅ **零 Patch 维护** - 直接集成上游代码
- ✅ **官方库支持** - 由内核维护者维护
- ✅ **稳定 API** - 向后兼容设计
- ✅ **丰富文档** - 技术论文、man 页面齐全

#### 挑战
- ⚠️ **内核依赖** - 需要确认 OH 内核版本支持
- ⚠️ **学习曲线** - 异步编程模型较复杂
- ⚠️ **调试困难** - 异步错误处理较困难
- ⚠️ **使用场景待明确** - 当前未发现明确依赖者

### 2.5 未来展望

liburing 在 OH 中的潜在发展方向：

1. **框架层集成**
   - 在 ACE/ArkUI 中支持异步 I/O
   - 为网络模块提供 io_uring 后端

2. **系统服务优化**
   - 分布式软总线的高性能通信
   - 存储服务的 I/O 加速

3. **开发者生态**
   - 提供 NAPI 绑定供 JS/TS 调用
   - 提供 Rust FFI 支持

---

## 3. 快速开始

### 3.1 头文件引用

```c
#include <liburing.h>
```

### 3.2 基础示例

```c
#include <liburing.h>
#include <stdio.h>
#include <string.h>

int main() {
    struct io_uring ring;
    
    // 初始化 io_uring，队列大小 32
    int ret = io_uring_queue_init(32, &ring, 0);
    if (ret < 0) {
        fprintf(stderr, "io_uring init failed: %s\n", strerror(-ret));
        return 1;
    }
    
    // 获取 SQE 并准备读操作
    struct io_uring_sqe *sqe = io_uring_get_sqe(&ring);
    io_uring_prep_read(sqe, fd, buf, size, offset);
    io_uring_sqe_set_data(sqe, user_data);
    
    // 提交操作
    io_uring_submit(&ring);
    
    // 等待完成
    struct io_uring_cqe *cqe;
    io_uring_wait_cqe(&ring, &cqe);
    
    // 处理结果
    if (cqe->res >= 0) {
        printf("Read %d bytes\n", cqe->res);
    }
    
    // 标记完成并清理
    io_uring_cqe_seen(&ring, cqe);
    io_uring_queue_exit(&ring);
    
    return 0;
}
```

### 3.3 更多资源

- **上游示例**: `liburing/examples/` 目录
- **测试用例**: `liburing/test/` 目录
- **技术论文**: https://kernel.dk/io_uring.pdf
- **Man 页面**: `man io_uring_setup`

---

## 4. 参考资料

1. [liburing GitHub](https://github.com/axboe/liburing)
2. [Efficient IO with io_uring (论文)](https://kernel.dk/io_uring.pdf)
3. [Kernel 文档 - io_uring](https://www.kernel.org/doc/html/latest/io_uring.html)
4. [io-uring 邮件列表](https://lore.kernel.org/io-uring/)

---

*本文档版本: 1.0*  
*最后更新: 2026-02-08*
