# 05 - API 差异分析

本文档分析 liburing 在 OpenHarmony 中与上游版本的 API 差异。

---

## 结论

**liburing 在 OpenHarmony 中与上游版本无 API 差异。**

由于 OH 未应用任何 Patch，且未修改源码，所有 API 与上游 2.7 版本完全一致。

---

## 1. API 状态概述

### 1.1 差异分析结果

| 差异类型 | 数量 | 说明 |
|---------|------|------|
| **新增 API** | 0 | 无 OH 特定新增 |
| **修改 API** | 0 | 无行为变更 |
| **删除 API** | 0 | 无功能禁用 |
| **废弃 API** | 0 | 与上游一致 |

### 1.2 与上游版本对比

| 属性 | 上游 2.7 | OH 2.7 | 差异 |
|-----|---------|-------|------|
| 头文件 | liburing.h | liburing.h | ✅ 相同 |
| 数据结构 | io_uring, io_uring_sqe, io_uring_cqe | 相同 | ✅ 相同 |
| 核心函数 | 50+ 函数 | 相同 | ✅ 相同 |
| 辅助宏 | IOURINGINLINE 等 | 相同 | ✅ 相同 |

---

## 2. 标准 API 概览

### 2.1 核心 API 分类

```
┌─────────────────────────────────────────────────────────────┐
│                    liburing API 架构                         │
├─────────────────────────────────────────────────────────────┤
│  生命周期管理                                                │
│  ├── io_uring_queue_init()        # 初始化                   │
│  ├── io_uring_queue_init_params() # 带参数初始化              │
│  ├── io_uring_queue_init_mem()    # 预分配内存初始化           │
│  └── io_uring_queue_exit()        # 销毁                     │
├─────────────────────────────────────────────────────────────┤
│  提交队列 (SQ) 操作                                           │
│  ├── io_uring_get_sqe()           # 获取 SQE                 │
│  ├── io_uring_submit()            # 提交到内核               │
│  └── io_uring_submit_and_wait()   # 提交并等待               │
├─────────────────────────────────────────────────────────────┤
│  完成队列 (CQ) 操作                                           │
│  ├── io_uring_peek_cqe()          # 查看 CQE (非阻塞)         │
│  ├── io_uring_wait_cqe()          # 等待 CQE (阻塞)           │
│  ├── io_uring_wait_cqes()         # 等待多个 CQE              │
│  └── io_uring_cqe_seen()          # 标记 CQE 已处理           │
├─────────────────────────────────────────────────────────────┤
│  注册管理                                                    │
│  ├── io_uring_register_buffers()  # 注册缓冲区               │
│  ├── io_uring_register_files()    # 注册文件描述符           │
│  ├── io_uring_unregister_buffers()# 注销缓冲区               │
│  └── io_uring_unregister_files()  # 注销文件描述符           │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 操作准备函数 (io_uring_prep_xxx)

liburing 提供了 50+ 个操作准备函数：

#### 文件 I/O
```c
void io_uring_prep_read(struct io_uring_sqe *sqe, int fd, 
                        void *buf, unsigned nbytes, __u64 offset);
void io_uring_prep_write(struct io_uring_sqe *sqe, int fd,
                         const void *buf, unsigned nbytes, __u64 offset);
void io_uring_prep_readv(struct io_uring_sqe *sqe, int fd,
                         const struct iovec *iovecs, unsigned nr_vecs, __u64 offset);
void io_uring_prep_writev(struct io_uring_sqe *sqe, int fd,
                          const struct iovec *iovecs, unsigned nr_vecs, __u64 offset);
void io_uring_prep_read_fixed(struct io_uring_sqe *sqe, int fd,
                              void *buf, unsigned nbytes, __u64 offset, int buf_index);
void io_uring_prep_write_fixed(struct io_uring_sqe *sqe, int fd,
                               const void *buf, unsigned nbytes, __u64 offset, int buf_index);
```

#### 网络 I/O
```c
void io_uring_prep_accept(struct io_uring_sqe *sqe, int fd,
                          struct sockaddr *addr, socklen_t *addrlen, int flags);
void io_uring_prep_connect(struct io_uring_sqe *sqe, int fd,
                           const struct sockaddr *addr, socklen_t addrlen);
void io_uring_prep_send(struct io_uring_sqe *sqe, int sockfd,
                        const void *buf, size_t len, int flags);
void io_uring_prep_recv(struct io_uring_sqe *sqe, int sockfd,
                        void *buf, size_t len, int flags);
void io_uring_prep_sendmsg(struct io_uring_sqe *sqe, int fd,
                           const struct msghdr *msg, unsigned flags);
void io_uring_prep_recvmsg(struct io_uring_sqe *sqe, int fd,
                           struct msghdr *msg, unsigned flags);
```

#### 文件系统操作
```c
void io_uring_prep_openat(struct io_uring_sqe *sqe, int dfd,
                          const char *path, int flags, mode_t mode);
void io_uring_prep_close(struct io_uring_sqe *sqe, int fd);
void io_uring_prep_statx(struct io_uring_sqe *sqe, int dfd,
                         const char *path, int flags, unsigned mask, struct statx *statxbuf);
void io_uring_prep_mkdirat(struct io_uring_sqe *sqe, int dfd,
                           const char *path, mode_t mode);
void io_uring_prep_unlinkat(struct io_uring_sqe *sqe, int dfd,
                            const char *path, int flags);
void io_uring_prep_renameat(struct io_uring_sqe *sqe, int olddfd,
                            const char *oldpath, int newdfd, const char *newpath, unsigned flags);
```

#### 高级操作
```c
void io_uring_prep_splice(struct io_uring_sqe *sqe, int fd_in, int64_t off_in,
                          int fd_out, int64_t off_out, unsigned int nbytes, unsigned int flags);
void io_uring_prep_poll_add(struct io_uring_sqe *sqe, int fd, unsigned poll_mask);
void io_uring_prep_timeout(struct io_uring_sqe *sqe, struct __kernel_timespec *ts,
                           unsigned count, unsigned flags);
void io_uring_prep_futex_wait(struct io_uring_sqe *sqe, uint32_t *futex,
                              uint64_t val, uint64_t mask, uint32_t futex_flags, unsigned int flags);
void io_uring_prep_futex_wake(struct io_uring_sqe *sqe, uint32_t *futex,
                              uint64_t val, uint64_t mask, uint32_t futex_flags, unsigned int flags);
```

---

## 3. 数据结构

### 3.1 核心数据结构

#### io_uring (主结构)

```c
struct io_uring {
    struct io_uring_sq sq;      // 提交队列
    struct io_uring_cq cq;      // 完成队列
    unsigned flags;              // 标志位
    int ring_fd;                 // io_uring 文件描述符
    unsigned features;           // 内核支持的特性
    int enter_ring_fd;           // 用于 io_uring_enter
    __u8 int_flags;              // 内部标志
    __u8 pad[3];
    unsigned pad2;
};
```

#### io_uring_sq (提交队列)

```c
struct io_uring_sq {
    unsigned *khead;             // 内核头指针
    unsigned *ktail;             // 内核尾指针
    unsigned *kring_mask;        // 队列掩码
    unsigned *kring_entries;     // 队列大小
    unsigned *kflags;            // 队列标志
    unsigned *kdropped;          // 丢弃计数
    unsigned *array;             // 索引数组
    struct io_uring_sqe *sqes;   // SQE 数组
    unsigned sqe_head;           // 用户头
    unsigned sqe_tail;           // 用户尾
    size_t ring_sz;              // 环大小
    void *ring_ptr;              // 映射地址
    unsigned ring_mask;          // 本地掩码
    unsigned ring_entries;       // 本地大小
    unsigned pad[2];
};
```

#### io_uring_sqe (提交队列项)

```c
struct io_uring_sqe {
    __u8 opcode;                 // 操作码 (IORING_OP_xxx)
    __u8 flags;                  // 标志位 (IOSQE_xxx)
    __u16 ioprio;                // I/O 优先级
    __s32 fd;                    // 文件描述符
    union {
        __u64 off;               // 偏移量
        __u64 addr2;
        struct {
            __u32 cmd_op;
            __u32 __pad1;
        };
    };
    union {
        __u64 addr;              // 地址
        __u64 splice_off_in;
    };
    __u32 len;                   // 长度
    union {
        __u32 rw_flags;
        __u32 fsync_flags;
        __u32 poll_events;
        __u32 poll32_events;
        __u32 sync_range_flags;
        __u32 msg_flags;
        __u32 timeout_flags;
        __u32 accept_flags;
        __u32 cancel_flags;
        __u32 open_flags;
        __u32 statx_flags;
        __u32 fadvise_advice;
        __u32 splice_flags;
        __u32 rename_flags;
        __u32 unlink_flags;
        __u32 hardlink_flags;
        __u32 xattr_flags;
        __u32 msg_ring_flags;
        __u32 uring_cmd_flags;
        __u32 waitid_flags;
        __u32 futex_flags;
        __u32 install_fd_flags;
    };
    __u64 user_data;             // 用户数据 (传递给 CQE)
    union {
        struct {
            __u16 buf_index;     // 缓冲区索引
            __u16 personality;
        };
        __u64 file_index;
    };
    union {
        struct {
            __u64 addr3;
            __u64 __pad2[1];
        };
        struct {
            __u32 level;         // 用于 getsockopt/setsockopt
            __u32 optname;
        };
    };
};
```

#### io_uring_cqe (完成队列项)

```c
struct io_uring_cqe {
    __u64 user_data;             // 用户数据 (来自 SQE)
    __s32 res;                   // 操作结果
    __u32 flags;                 // 标志位 (CQE_F_xxx)
    union {
        __u64 big_cqe[0];        // 大 CQE 扩展
    };
};
```

---

## 4. 操作码列表

### 4.1 支持的 I/O 操作码

```c
// 文件 I/O
IORING_OP_NOP              // 空操作
IORING_OP_READV            // 向量读取
IORING_OP_WRITEV           // 向量写入
IORING_OP_FSYNC            // 文件同步
IORING_OP_READ_FIXED       // 固定缓冲区读取
IORING_OP_WRITE_FIXED      // 固定缓冲区写入
IORING_OP_READ             // 读取
IORING_OP_WRITE            // 写入
IORING_OP_READ_MULTISHOT   // 多读 (内核 6.0+)
IORING_OP_FADVISE          // 文件预读建议
IORING_OP_MADVISE          // 内存预读建议
IORING_OP_FALLOCATE        // 文件预分配
IORING_OP_OPENAT           // 打开文件
IORING_OP_OPENAT2          // 打开文件 (扩展)
IORING_OP_CLOSE            // 关闭文件
IORING_OP_STATX            // 获取文件状态
IORING_OP_SYNC_FILE_RANGE  // 同步文件范围
IORING_OP_FTRUNCATE        // 文件截断

// 网络 I/O
IORING_OP_SENDMSG          // 发送消息
IORING_OP_RECVMSG          // 接收消息
IORING_OP_SEND             // 发送
IORING_OP_RECV             // 接收
IORING_OP_TIMEOUT          // 超时
IORING_OP_TIMEOUT_REMOVE   // 移除超时
IORING_OP_ACCEPT           // 接受连接
IORING_OP_CONNECT          // 建立连接
IORING_OP_SEND_ZC          // 零拷贝发送
IORING_OP_SENDMSG_ZC       // 零拷贝发送消息
IORING_OP_BIND             // 绑定 (2.7+)
IORING_OP_LISTEN           // 监听 (2.7+)

// 高级操作
IORING_OP_ASYNC_CANCEL     // 异步取消
IORING_OP_LINK_TIMEOUT     // 链接超时
IORING_OP_POLL_ADD         // 添加轮询
IORING_OP_POLL_REMOVE      // 移除轮询
IORING_OP_SPLICE           // 管道转发
IORING_OP_TEE              // TEE 操作
IORING_OP_SHUTDOWN         // 关闭连接
IORING_OP_RENAMEAT         // 重命名
IORING_OP_UNLINKAT         // 删除
IORING_OP_MKDIRAT          // 创建目录
IORING_OP_SYMLINKAT        // 创建符号链接
IORING_OP_LINKAT           // 创建硬链接
IORING_OP_MSG_RING         // 向其他 ring 发送消息
IORING_OP_URING_CMD        // 通用命令
IORING_OP_WAITID           // 等待进程状态

// 缓冲区管理
IORING_OP_PROVIDE_BUFFERS  // 提供缓冲区
IORING_OP_REMOVE_BUFFERS   // 移除缓冲区

// 文件描述符管理
IORING_OP_FILES_UPDATE     // 更新文件集
IORING_OP_FIXED_FD_INSTALL // 安装固定 FD

// Futex 同步
IORING_OP_FUTEX_WAIT       // Futex 等待
IORING_OP_FUTEX_WAKE       // Futex 唤醒
IORING_OP_FUTEX_WAITV      // 向量 Futex 等待

// 套接字
IORING_OP_SOCKET           // 创建套接字
IORING_OP_SENDTO           // 发送到地址
IORING_OP_RECV_MULTISHOT   // 多接收
```

---

## 5. 配置影响的 API

### 5.1 OH 构建配置对 API 的影响

虽然 API 本身无差异，但 OH 的构建配置可能影响部分功能：

| 配置项 | 影响 | 说明 |
|-------|------|------|
| **源文件选择** | 功能完整 | 4 个核心源文件包含全部 API |
| **FFI 排除** | 无影响 | FFI 是独立功能，不影响 C API |
| **nolibc 排除** | 无影响 | OH 使用标准 libc |
| **version.c 排除** | 轻微影响 | 版本函数不可用，但版本宏可用 |

### 5.2 版本信息获取

由于未编译 version.c，以下函数**不可用**：

```c
// 不可用 (链接错误)
int io_uring_major_version(void);
int io_uring_minor_version(void);
bool io_uring_check_version(int major, int minor);
```

**替代方案**: 使用头文件中的宏

```c
// 可用 (头文件定义)
#include <liburing/io_uring_version.h>

#define IO_URING_VERSION_MAJOR 2
#define IO_URING_VERSION_MINOR 7

// 编译时检查
#if IO_URING_CHECK_VERSION(2, 7)
    // 版本 2.7+ 的代码
#endif
```

---

## 6. 使用示例

### 6.1 基础使用模式

```c
#include <liburing.h>
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    struct io_uring ring;
    int ret;
    
    // 1. 初始化 io_uring
    ret = io_uring_queue_init(32, &ring, 0);
    if (ret < 0) {
        fprintf(stderr, "io_uring init failed: %d\n", ret);
        return 1;
    }
    
    // 2. 打开文件
    int fd = open("test.txt", O_RDONLY);
    if (fd < 0) {
        perror("open");
        return 1;
    }
    
    // 3. 准备读取操作
    char buffer[4096];
    struct io_uring_sqe *sqe = io_uring_get_sqe(&ring);
    io_uring_prep_read(sqe, fd, buffer, sizeof(buffer), 0);
    io_uring_sqe_set_data(sqe, (void *)1);  // 用户数据
    
    // 4. 提交操作
    ret = io_uring_submit(&ring);
    if (ret < 0) {
        fprintf(stderr, "submit failed: %d\n", ret);
        return 1;
    }
    
    // 5. 等待完成
    struct io_uring_cqe *cqe;
    ret = io_uring_wait_cqe(&ring, &cqe);
    if (ret < 0) {
        fprintf(stderr, "wait_cqe failed: %d\n", ret);
        return 1;
    }
    
    // 6. 处理结果
    if (cqe->res >= 0) {
        printf("Read %d bytes\n", cqe->res);
    } else {
        fprintf(stderr, "Read error: %d\n", cqe->res);
    }
    
    // 7. 标记完成并清理
    io_uring_cqe_seen(&ring, cqe);
    close(fd);
    io_uring_queue_exit(&ring);
    
    return 0;
}
```

### 6.2 GN 构建配置

```gn
# BUILD.gn
ohos_executable("my_uring_app") {
  sources = [ "my_uring_app.c" ]
  deps = [ "//third_party/liburing:liburing" ]
}
```

---

## 7. 兼容性说明

### 7.1 内核版本兼容性

liburing API 的可用性取决于内核版本：

| 内核版本 | 支持的特性 |
|---------|-----------|
| 5.1+ | 基础 io_uring 操作 |
| 5.5+ | 网络 I/O (send/recv/accept) |
| 5.6+ | recvmsg/sendmsg, splice |
| 5.7+ | 链接超时, 异步取消 |
| 5.8+ | 轮询多触发 (multishot) |
| 5.10+ | 文件系统操作 (openat/statx/close) |
| 5.11+ | 缓冲区提供 (provided buffers) |
| 5.15+ | 更多操作类型 |
| 6.0+ | 多读/多接收 |
| 6.1+ | 多播 accept |
| 6.4+ | futex 支持 |
| 6.7+ | bind/listen |
| 6.10+ | send/recv bundle |

### 7.2 OH 内核兼容性

**TODO(需确认)**: 确认 OH 使用的内核版本及支持的 io_uring 特性。

---

## 8. 总结

### API 差异总结

| 项目 | 状态 |
|-----|------|
| API 变更 | ✅ 无变更 |
| 行为差异 | ✅ 无差异 |
| 功能限制 | ⚠️ version 函数不可用 (使用宏替代) |
| 兼容性 | ✅ 与上游完全一致 |

### 开发者建议

1. **直接参考上游文档** - liburing 官方文档完全适用
2. **使用头文件宏** - 避免使用 version.c 的函数
3. **检查内核支持** - 确保使用的 API 被内核支持
4. **参考示例代码** - `examples/` 目录提供丰富示例

---

*本文档版本: 1.0*  
*最后更新: 2026-02-08*
