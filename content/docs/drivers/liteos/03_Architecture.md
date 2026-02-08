# 架构说明

## 整体架构

### 内核驱动架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    用户空间进程                              │
├─────────────────────────────────────────────────────────────┤
│                  标准文件系统接口                            │
│    open()   close()   read()   write()   ioctl()   mmap()   │
├─────────────────────────────────────────────────────────────┤
│                   /dev/hwlog_exception                       │
│              (字符设备节点，模式 0666)                        │
├─────────────────────────────────────────────────────────────┤
│                   hievent 驱动层                             │
│                   hievent_driver.c                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              环形缓冲区 (1024 字节)                   │   │
│  │    writeOffset → [Event Entry 1] → headOffset      │   │
│  │    writeOffset → [Event Entry 2] → ...              │   │
│  └─────────────────────────────────────────────────────┘   │
│           LosMux (互斥锁)  |  wait_queue (等待队列)         │
├─────────────────────────────────────────────────────────────┤
│                   hievent 事件层                             │
│                  hiview_hievent.c                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            HiviewHievent 事件对象                    │   │
│  │    eventid + time + Payload链表 + filePath[]         │   │
│  └─────────────────────────────────────────────────────┘   │
│           LogBufToException → /dev/hwlog_exception         │
├─────────────────────────────────────────────────────────────┤
│                   LiteOS_A 内核                             │
│         los_memory  |  los_mux  |  los_task  |  los_vm    │
└─────────────────────────────────────────────────────────────┘
```

**证据**: `hievent_driver.c:348-357` (file_operations_vfs), `hievent_driver.c:78` (g_hieventDev)

## 组件职责

### 驱动层 (hievent_driver.c)

| 组件 | 职责 | 关键实现 |
|-----|------|---------|
| **字符设备** | 提供 `/dev/hwlog_exception` 节点 | `register_driver()` at line 384 |
| **环形缓冲区** | 存储事件日志 | `HIEVENT_LOG_BUFFER = 1024` at line 56 |
| **读写接口** | 用户空间读写事件 | `HieventRead()`, `HieventWrite()` |
| **Poll 接口** | 事件通知机制 | `HieventPoll()` 返回 `POLLOUT \| POLLWRNORM` |
| **同步保护** | 防止竞态条件 | `LosMux` 互斥锁 |

### 事件层 (hiview_hievent.c)

| 组件 | 职责 | 关键实现 |
|-----|------|---------|
| **事件对象** | 封装事件数据 | `HiviewHievent` 结构体 |
| **Payload 管理** | 键值对链表 | `HiviewHieventPayload` |
| **路径附件** | 文件路径列表 | `filePath[MAX_PATH_NUMBER]` |
| **格式化** | 事件序列化 | `HiviewHieventConvertString()` |
| **上报** | 发送事件到驱动 | `LogBufToException()` |

## 数据结构

### HieventEntry (驱动层)

```c
struct HieventEntry {
    uint16_t len;       // 负载长度
    uint16_t hdrSize;   // 头部大小
    int32_t pid;        // 进程 ID
    int32_t tid;        // 线程 ID
    int32_t sec;        // 秒级时间戳
    int32_t nsec;       // 纳秒级时间戳
    char msg[0];        // 变长消息
};
```

**证据**: `hievent_driver.c:59-67`

### HiviewHievent (事件层)

```c
struct HiviewHievent {
    unsigned int eventid;     // 事件 ID
    long long time;            // 时间戳
    struct HiviewHieventPayload *head;  // Payload 链表头
    char *filePath[MAX_PATH_NUMBER];    // 文件路径数组
};
```

**证据**: `hiview_hievent.c:38-48`

### IdapHeader (上报格式)

```c
struct IdapHeader {
    char level;      // 日志级别
    char category;   // 分类
    char logType;    // 日志类型
    char sn;         // 序列号
};
```

**证据**: `hievent_driver.h:39-44`

## 关键时序

### 事件写入流程

```
用户态                              内核态
  │                                   │
  │  open("/dev/hwlog_exception")     │
  │─────────────────────────────────>│
  │                                   │ HieventOpen()
  │                                   │ return 0
  │<──────────────────────────────────│
  │                                   │
  │  write(fd, buffer, size)          │
  │─────────────────────────────────>│
  │                                   │ HieventWrite()
  │                                   │ HieventWriteInternal()
  │                                   │   ├─ 校验 CHECK_CODE
  │                                   │   ├─ 环形缓冲区写入
  │                                   │   └─ wake_up_interruptible()
  │                                   │ return 实际写入长度
  │<──────────────────────────────────│
```

### 事件读取流程

```
用户态                              内核态
  │                                   │
  │  read(fd, buffer, size)           │
  │─────────────────────────────────>│
  │                                   │ HieventRead()
  │                                   │   ├─ wait_event_interruptible()
  │                                   │   ├─ LOS_MuxAcquire()
  │                                   │   ├─ 环形缓冲区读取
  │                                   │   └─ LOS_MuxRelease()
  │<──────────────────────────────────│ return 读取数据
```

## 线程模型

### 初始化线程

| 阶段 | 函数 | 级别 |
|-----|------|------|
| 内核模块初始化 | `HieventInit()` | `LOS_INIT_LEVEL_KMOD_EXTENDED` |

**证据**: `hievent_driver.c:389` → `LOS_MODULE_INIT(HieventInit, LOS_INIT_LEVEL_KMOD_EXTENDED)`

### 读写操作

- **同步模型**: 单线程访问 + LosMux 互斥锁保护
- **阻塞行为**: `read()` 在无数据时阻塞 (`wait_event_interruptible`)
- **唤醒机制**: `write()` 后调用 `wake_up_interruptible()` 唤醒读取者

## 内存模型

### 内核缓冲区

| 参数 | 值 | 说明 |
|-----|---|------|
| `HIEVENT_LOG_BUFFER` | 1024 字节 | 环形缓冲区总大小 |
| `DRIVER_MODE` | 0666 | 设备节点权限 |
| `HIEVENT_LOG_BUFFER` | 1024 | 单次最大写入限制 |

### 内存分配

- **缓冲区分配**: `LOS_MemAlloc(OS_SYS_MEM_ADDR, HIEVENT_LOG_BUFFER)`
- **事件对象分配**: `LOS_MemAlloc(OS_SYS_MEM_ADDR, sizeof(*event))`
- **Payload 分配**: `LOS_MemAlloc(OS_SYS_MEM_ADDR, ...)`

## 依赖方向

```
hievent_driver.c
├── los_init.h          [初始化框架]
├── los_memory.h        [内存分配]
├── los_mux.h           [互斥锁]
├── los_task_pri.h      [任务]
├── los_vm_*.h          [虚拟内存]
└── poll.h             [Poll 机制]

hiview_hievent.c
├── hiview_hievent.h    [本模块头文件]
├── hievent_driver.h    [驱动层接口]
├── los_memory.h        [内存分配]
├── los_mux.h           [互斥锁]
└── los_task_pri.h      [任务]
```

---

*证据来源*:
- 架构图: 基于代码实现的逆向分析
- 函数签名: `hievent_driver.c`, `hiview_hievent.c`
- 宏定义: `hievent_driver.h`, `hiview_hievent.h`
- 初始化: `hievent_driver.c:389`
