# TEE Client 内部实现细节

## 1. 核心类/结构体职责

### 1.1 内部数据结构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        内部数据结构关系图                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  TEEC_Context (用户可见)                                                  │
│  ├── fd: TEE 设备文件描述符                                              │
│  ├── ta_path: TA 文件路径指针                                            │
│  ├── session_list ──► TEEC_Session 链表                                  │
│  ├── shrd_mem_list ──► TEEC_SharedMemory 链表                            │
│  └── share_buffer: 共享缓冲区                                            │
│                                                                          │
│  TEEC_ContextInner (内部实现)  @ frameworks/include/tee_client_inner.h    │
│  ├── context: TEEC_Context (用户结构副本)                                 │
│  ├── fd: 设备 FD                                                          │
│  ├── ops_cnt: 引用计数                                                    │
│  ├── session_list ──► TEEC_SessionInner 链表                             │
│  ├── shrd_mem_list ──► TEEC_SharedMemoryInner 链表                       │
│  ├── callFromService: 是否来自服务                                        │
│  └── head: 链表节点                                                       │
│                                                                          │
│  TEEC_SessionInner (内部实现)                                             │
│  ├── session: TEEC_Session (用户结构副本)                                 │
│  ├── context: 指向 TEEC_ContextInner                                      │
│  ├── ops_cnt: 引用计数                                                    │
│  └── head: 链表节点                                                       │
│                                                                          │
│  TEEC_SharedMemoryInner (内部实现)                                        │
│  ├── shm: TEEC_SharedMemory (用户结构副本)                                │
│  ├── context: 指向 TEEC_ContextInner                                      │
│  ├── offset: 共享内存偏移                                                 │
│  ├── ops_cnt: 引用计数                                                    │
│  └── head: 链表节点                                                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 结构体详细职责

| 结构体 | 职责 | Owner | 生命周期 |
|--------|------|-------|----------|
| `TEEC_Context` | 用户可见的 TEE 上下文句柄 | 用户（CA） | `InitializeContext` → `FinalizeContext` |
| `TEEC_ContextInner` | 内部上下文实现，管理会话和共享内存 | libteec | 与 Context 绑定 |
| `TEEC_Session` | 用户可见的会话句柄 | 用户（CA） | `OpenSession` → `CloseSession` |
| `TEEC_SessionInner` | 内部会话实现 | libteec | 与会话绑定 |
| `TEEC_SharedMemory` | 用户可见的共享内存描述 | 用户（CA） | `Register/Allocate` → `Release` |
| `TEEC_SharedMemoryInner` | 内部共享内存实现 | libteec | 与 SharedMemory 绑定 |
| `DaemonProcdata` | cadaemon 中 CA 进程数据 | cadaemon | CA 首次调用 → CA 死亡 |
| `TidData` | 线程 ID 跟踪数据 | cadaemon | 命令开始 → 命令结束 |

### 1.3 CaDaemonService 类

```cpp
// services/cadaemon/src/ca_daemon/cadaemon_service.h
class CaDaemonService : public SystemAbility, public CaDaemonStub {
private:
    // 进程数据管理
    std::mutex mProcDataLock;                    // 进程数据锁
    std::vector<sptr<Client>> mClients;         // 客户端列表
    std::mutex mClientLock;                      // 客户端列表锁
    bool registerToService_ = false;             // 注册标志
    
    // TEE 版本
    uint32_t mTeeVersion = 0;                    // TEE 版本号
    void *mDstbHandle = nullptr;                 // 分布式服务句柄
    
    // TUI 相关
    bool mIsNeedTui = false;                     // 是否需要 TUI
    
public:
    // SystemAbility 回调
    void OnStart() override;                     // 服务启动
    void OnStop() override;                      // 服务停止
    void OnAddSystemAbility(int32_t, const std::string&) override;
    
    // IPC 接口实现
    TEEC_Result InitializeContext(const char*, MessageParcel&) override;
    TEEC_Result FinalizeContext(TEEC_Context*) override;
    TEEC_Result OpenSession(...) override;
    TEEC_Result CloseSession(TEEC_Session*, TEEC_Context*) override;
    TEEC_Result InvokeCommand(...) override;
    TEEC_Result RegisterSharedMemory(...) override;
    TEEC_Result AllocateSharedMemory(...) override;
    TEEC_Result ReleaseSharedMemory(...) override;
    int32_t SetCallBack(const sptr<IRemoteObject>&) override;
    TEEC_Result SendSecfile(...) override;
    TEEC_Result GetTeeVersion(MessageParcel&) override;
    
    // 内部方法
    bool IsValidContext(const TEEC_Context*, const CallerIdentity&);
    DaemonProcdata* CallGetProcDataPtr(const CallerIdentity&);
    void ProcessCaDied(int32_t pid);
    
private:
    bool Init();
    void CreateTuiThread();
    void CreateDstbTeeService();
    int GetTEEVersion();
};
```

## 2. 内部 API 契约

### 2.1 稳定接口（可依赖）

| 接口 | 文件 | 稳定性 | 说明 |
|------|------|--------|------|
| `TEEC_*` API | `tee_client_api.h` | **稳定** | GlobalPlatform 标准，向后兼容 |
| `TEEC_PARAM_TYPES` | `tee_client_api.h` | **稳定** | 参数类型构造宏 |
| `TEEC_ERROR_*` | `tee_client_constants.h` | **稳定** | 错误码定义 |
| `TEEC_SUCCESS` | `tee_client_constants.h` | **稳定** | 成功返回值 |

### 2.2 内部接口（不保证稳定）

| 接口 | 文件 | 稳定性 | 说明 |
|------|------|--------|------|
| `TEEC_*Inner` 函数 | `tee_client_api.c` | ⚠️ 内部 | 内部实现细节可能变更 |
| `GetBnContext()` | `tee_client_inner.h` | ⚠️ 内部 | 内部上下文管理 |
| `GetBnSession()` | `tee_client_inner.h` | ⚠️ 内部 | 内部会话管理 |
| `GetBnShrMem()` | `tee_client_inner.h` | ⚠️ 内部 | 内部共享内存管理 |
| `PutBnContext()` | `tee_client_inner.h` | ⚠️ 内部 | 引用计数释放 |
| `PutBnSession()` | `tee_client_inner.h` | ⚠️ 内部 | 引用计数释放 |
| `PutBnShrMem()` | `tee_client_inner.h` | ⚠️ 内部 | 引用计数释放 |

### 2.3 钩子/回调接口

| 接口 | 类型 | 注册方式 | 说明 |
|------|------|----------|------|
| `Client::OnRemoteDied()` | 死亡通知 | `AddDeathRecipient()` | CA 进程死亡时回调 |
| `TeeClient::DeathNotifier` | 死亡通知 | IPC `AddDeathRecipient()` | 服务死亡时回调 |

## 3. 资源生命周期

### 3.1 上下文生命周期

```
创建: TEEC_InitializeContext() 
    │
    ├──► TEEC_ContextInner* contextInner = malloc() ──► ref_cnt = 1
    │
    ├──► SetContextToProcData() ──► 关联到进程数据
    │
    └──► PutBnContext() ──► ref_cnt = 2 (加入列表)
         
使用: 各种 API 操作
    │
    ├──► GetBnContext() ──► ref_cnt++ (使用时)
    │
    └──► PutBnContextAndReleaseFd() ──► ref_cnt-- (使用后)

销毁: TEEC_FinalizeContext()
    │
    ├──► CallFinalizeContext()
    │       │
    │       ├──► FindAndRemoveBnContext() ──► 从列表移除
    │       │
    │       └──► PutBnContextAndReleaseFd() ──► ref_cnt--
    │
    └──► ref_cnt == 0 ──► free(contextInner)
```

**Owner**: 用户（CA）拥有 `TEEC_Context`，libteec 拥有 `TEEC_ContextInner`

### 3.2 会话生命周期

```
创建: TEEC_OpenSession()
    │
    ├──► TEEC_Session* session = malloc()
    │
    ├──► CallGetBnContext() ──► 获取上下文（ref_cnt++）
    │
    ├──► TEEC_OpenSessionInner() ──► 与 TEE 建立会话
    │
    └──► PutBnSession() ──► ref_cnt = 1 (加入列表)
         PutBnContextAndReleaseFd() ──► ref_cnt--

销毁: TEEC_CloseSession()
    │
    ├──► GetBnContext() ──► 获取上下文（ref_cnt++）
    │
    ├──► FindAndRemoveSession() ──► 从列表移除
    │
    ├──► TEEC_CloseSessionInner() ──► 通知 TEE 关闭
    │
    ├──► PutBnSession() ──► ref_cnt-- → free(session)
    │
    └──► PutBnContextAndReleaseFd() ──► ref_cnt--
```

**Owner**: 用户（CA）拥有 `TEEC_Session`，libteec 拥有 `TEEC_SessionInner`

### 3.3 共享内存生命周期

```
创建方式1: TEEC_RegisterSharedMemory()
    │
    ├──► 用户提供 buffer（用户拥有）
    │
    └──► TEEC_SharedMemoryInner* shmInner = malloc()
         memcpy(shmInner, sharedMem)
         shmInner->ops_cnt = 1

创建方式2: TEEC_AllocateSharedMemory()
    │
    ├──► shmInner->buffer = malloc(size)（libteec 拥有）
    │
    └──► TEEC_AllocateSharedMemoryInner()
         └──► mmap() ──► 映射到 TEE 可见内存

使用: OpenSession / InvokeCommand
    │
    ├──► GetBnShmByOffset() ──► 获取内部结构
    │
    └──► ops_cnt++（使用时）
         ops_cnt--（使用后）

销毁: TEEC_ReleaseSharedMemory()
    │
    ├──► 查找共享内存
    │
    ├──► munmap() ──► 解除映射（分配式）
    │
    ├──► free(buffer) ──► 释放缓冲区（分配式）
    │
    └──► free(shmInner)
```

**Owner**:
- 注册式：用户拥有 buffer，libteec 拥有 `TEEC_SharedMemoryInner`
- 分配式：libteec 拥有 buffer 和 `TEEC_SharedMemoryInner`

### 3.4 引用计数规则

| 操作 | ref_cnt 变化 | 说明 |
|------|--------------|------|
| 初始化 | +1 → +2 | 创建 + 加入列表 |
| 获取 (GetBn*) | +1 | 使用时 |
| 释放 (PutBn*) | -1 | 使用后 |
| 从列表移除 | -1 | 销毁时 |
| ref_cnt == 0 | free() | 资源释放 |

## 4. 线程安全机制

### 4.1 全局锁

| 锁 | 类型 | 保护数据 | 位置 |
|----|------|----------|------|
| `g_mutexTidList` | `pthread_mutex_t` | `g_teecTidList` | `cadaemon_service.cpp:45` |
| `mProcDataLock` | `std::mutex` | `g_teecProcDataList` | `cadaemon_service.cpp` |
| `mClientLock` | `std::mutex` | `mClients` 向量 | `cadaemon_service.cpp` |
| `g_hilogLock` | `pthread_mutex_t` | 日志输出 | `tee_hilog_lock.h` |

### 4.2 锁使用模式

```cpp
// 1. TID 列表锁（C 风格）
int lockRet = TidMutexLock();
// ... 操作 g_teecTidList ...
TidMutexUnlock(lockRet);

// 2. 进程数据锁（C++ RAII 风格）
{
    lock_guard<mutex> autoLock(mProcDataLock);
    // ... 操作 g_teecProcDataList ...
}

// 3. 客户端列表锁
{
    lock_guard<mutex> autoLock(mClientLock);
    // ... 操作 mClients ...
}
```

### 4.3 线程模型

```
┌─────────────────────────────────────────────────────────────────┐
│                        cadaemon 进程                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  主线程 (IPC Handler)                                             │
│  ├── OnRemoteRequest() ──► 处理 IPC 请求                         │
│  │   ├── 加锁 ──► 操作共享数据 ──► 解锁                           │
│  │   └── 调用 Inner API                                          │
│  │                                                               │
│  └── 阻塞在 IPC 框架的事件循环                                     │
│                                                                  │
│  TUI 线程 (条件编译)                                              │
│  ├── TeeTuiThreadWork()                                          │
│  │   └── 监听 /sys/kernel/tui/c_state                            │
│  │                                                               │
│  └── 通过全局变量与主线程通信                                      │
│                                                                  │
│  工作线程池 (TID 跟踪)                                            │
│  ├── 每个 CA 命令在独立线程中执行                                  │
│  │   └── AddTidData() / RemoveTidFromList()                      │
│  │                                                               │
│  └── SIGUSR1 信号用于取消操作                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 5. 内存管理策略

### 5.1 内存分配策略

| 场景 | 分配方式 | 释放方式 | 说明 |
|------|----------|----------|------|
| 临时缓冲区 | `malloc()` | `free()` | 函数内部临时使用 |
| 上下文/会话/共享内存 | `malloc()` | 引用计数归零时 `free()` | 通过 Get/Put 管理生命周期 |
| 字符串缓冲区 | `malloc(size)` | `free()` | 路径、名称等字符串 |
| ION 内存 | `mmap()` | `munmap()` | 共享内存映射 |
| Ashmem | `Ashmem::Create()` | 自动回收 | IPC 共享内存 |

### 5.2 内存清零策略

| 数据类型 | 清零方式 | 位置 |
|----------|----------|------|
| 敏感数据（key, auth） | `memset_s()` | 多处使用 |
| 结构体初始化 | `memset_s(ptr, size, 0, size)` | 创建时 |
| IPC 传出数据 | 指针字段清零 | `WriteSession()`, `WriteOperation()` |
| 临时缓冲区 | `memset_s()` 或 `calloc()` | 分配时 |

### 5.3 内存对齐

```c
// 共享内存对齐要求
#define TEEC_MEM_ALIGN 0x1000  // 4KB 对齐

// 分配时对齐
size_t alignedSize = (size + TEEC_MEM_ALIGN - 1) & ~(TEEC_MEM_ALIGN - 1);
```

## 6. 错误处理策略

### 6.1 错误传播

```
┌─────────────────────────────────────────────────────────────┐
│                        错误传播路径                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TEE / TA                                                    │
│     │                                                        │
│     ├──► TZDriver (内核错误码)                               │
│     │       │                                                │
│     │       └──► 转换为 TEEC_Result                          │
│     │               @ tee_client_api.c                       │
│     │                                                        │
│     └──► TA 错误码                                           │
│             │                                                │
│             └──► 通过 returnOrigin 返回错误来源               │
│                                                              │
│  libteec / cadaemon                                          │
│     │                                                        │
│     ├──► 参数校验失败 ──► TEEC_ERROR_BAD_PARAMETERS          │
│     │                                                        │
│     ├──► 内存分配失败 ──► TEEC_ERROR_OUT_OF_MEMORY           │
│     │                                                        │
│     ├──► 权限验证失败 ──► TEEC_ERROR_ACCESS_DENIED           │
│     │                                                        │
│     └──► 内部错误 ──► TEEC_ERROR_GENERIC                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 6.2 错误处理模式

```cpp
// 1. 立即返回模式
if (context == nullptr) {
    tloge("invalid context!\n");
    return TEEC_ERROR_BAD_PARAMETERS;
}

// 2. 清理后返回模式
TEEC_Result result = TEEC_SUCCESS;
void* buffer = malloc(size);
if (buffer == nullptr) {
    result = TEEC_ERROR_OUT_OF_MEMORY;
    goto cleanup;
}
// ... 其他操作 ...

cleanup:
    free(buffer);
    return result;

// 3. 异常日志模式
if (ioctl(fd, cmd, arg) != 0) {
    tloge("ioctl failed: %d, errno=%d\n", ret, errno);
    LogException(ret, uuid, origin, TYPE_IOCTL_FAIL);
    return TEEC_ERROR_GENERIC;
}
```

## 7. 性能优化策略

### 7.1 零拷贝优化

```
┌─────────────────────────────────────────────────────────────┐
│                        数据传输优化                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  小数据量 (<= 4MB + 256B)                                    │
│     │                                                        │
│     ├──► Ashmem 共享内存                                     │
│     │       └──► 一次内存拷贝（kernel space）                 │
│     │                                                        │
│     └──► 通过 MessageParcel 传递 FD                          │
│                                                              │
│  大数据量                                                    │
│     │                                                        │
│     ├──► ION 共享内存                                        │
│     │       └──► FD 传递（无拷贝）                           │
│     │                                                        │
│     └──► 通过 SCM_RIGHTS 传递 FD                             │
│             @ tee_client_socket.c                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 7.2 缓存策略

| 缓存 | 类型 | 说明 |
|------|------|------|
| `g_teecProcDataList` | 进程数据缓存 | CA 进程级数据缓存 |
| `g_teecTidList` | TID 缓存 | 线程跟踪缓存 |
| 上下文列表 | 会话缓存 | 每个进程的上下文列表 |

---

**文档版本**: 1.0
**更新时间**: 2026-02-07
