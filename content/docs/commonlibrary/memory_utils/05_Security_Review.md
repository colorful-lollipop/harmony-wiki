# 安全风险评审

本文档对 memory_utils 各模块进行安全风险分析，包括攻击面、信任边界和潜在安全风险。

---

## 评审范围

| 模块 | 状态 | 说明 |
|------|------|------|
| libdmabufheap | ✅ 已评审 | DMA 缓冲区分配 |
| libmeminfo | ✅ 已评审 | 内存信息查询 |
| libpurgeablemem | ✅ 已评审 | 可回收内存管理 |
| libmemleak | ⏭️ 不在代码库中 | 规划中 |
| libspeculative | ⏭️ 不在代码库中 | 规划中 |

---

## 攻击面清单

### 1. libdmabufheap 攻击面

| 攻击面 | 类型 | 描述 |
|--------|------|------|
| `DmabufHeapOpen()` | 系统调用 | 打开 `/dev/dma_heap/{heapName}` 设备节点 |
| `DmabufHeapBufferAlloc()` | ioctl | 通过 DMA_HEAP_IOCTL_ALLOC 分配缓冲区 |
| `DmabufHeapBufferFree()` | 系统调用 | 关闭文件描述符 |
| `DmabufHeapBufferSyncStart/End()` | ioctl | DMA_BUF_IOCTL_SYNC 同步缓存 |
| heapName 参数 | 用户输入 | DMA 堆名称字符串 |

**证据来源**: `libdmabufheap/include/dmabuf_alloc.h:53-63`

---

### 2. libmeminfo 攻击面

| 攻击面 | 类型 | 描述 |
|--------|------|------|
| `GetRssByPid(pid)` | procfs | 读取 `/proc/{pid}/statm` |
| `GetPssByPid(pid)` | procfs | 读取 `/proc/{pid}/smaps_rollup` |
| `GetSwapPssByPid(pid)` | procfs | 读取 `/proc/{pid}/smaps_rollup` |
| `GetGraphicsMemory(pid)` | HDI | 调用 MemoryTracker 服务 |
| `GetDmaInfo(pid)` | 动态库 | 调用 libmemmgrclient.z.so |
| pid 参数 | 用户输入 | 进程 ID |

**证据来源**: `libmeminfo/include/meminfo.h:88-100`

---

### 3. libpurgeablemem 攻击面

| 攻击面 | 类型 | 描述 |
|--------|------|------|
| `OH_PurgeableMemory_Create()` | mmap | 创建可回收内存映射 |
| `OH_PurgeableMemory_BeginRead()` | 页表操作 | 引用计数管理 |
| `OH_PurgeableMemory_BeginWrite()` | 页表操作 | 引用计数管理 |
| `OH_PurgeableMemory_Destroy()` | munmap | 释放内存映射 |
| ModifyFunc 回调 | 用户回调 | 数据重建函数 |
| size 参数 | 用户输入 | 内存大小 |

**证据来源**: `libpurgeablemem/interfaces/kits/c/purgeable_memory.h`

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            信任边界                                      │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                     内核空间                                      │   │
│  │   /dev/dma_heap/*  │  procfs  │  MAP_PURGEABLE  │  Ashmem     │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                ▲                                         │
│                                │ 系统调用/IOCTL                         │
│                                │                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                     用户空间                                      │   │
│  │                                                                  │   │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │   │ libdmabufheap│  │ libmeminfo │  │ libpurgeablemem       │  │   │
│  │   │              │  │             │  │                       │  │   │
│  │   │ • open()     │  │ • procfs   │  │ • mmap()              │  │   │
│  │   │ • ioctl()    │  │ • HDI      │  │ • munmap()            │  │   │
│  │   │ • close()   │  │             │  │ • pthread_rwlock      │  │   │
│  │   └─────────────┘  └─────────────┘  │ • ModifyFunc 回调     │  │   │
│  │                                      └─────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                     应用层 (不可信)                               │   │
│  │   任意第三方应用                                                  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 边界说明

| 边界 | 方向 | 说明 |
|------|------|------|
| 应用 → libdmabufheap | 信任边界 | 应用提供的参数需验证 |
| 应用 → libmeminfo | 信任边界 | pid 参数需验证 |
| 应用 → libpurgeablemem | 信任边界 | size 和回调函数需验证 |
| libdmabufheap → 内核 | 边界 | 依赖内核安全机制 |
| libmeminfo → 内核/HDI | 边界 | 依赖 procfs/HDI 访问控制 |
| libpurgeablemem → 内核 | 边界 | 依赖 mmap/MAP_PURGEABLE 安全机制 |

---

## 数据流图

```
┌──────────────┐                           ┌──────────────┐
│   应用进程   │                           │   内核空间   │
│              │                           │              │
└──────┬───────┘                           └──────┬───────┘
       │                                          │
       │ DmabufHeapOpen("/dev/dma_heap/default")  │
       ├─────────────────────────────────────────►│ open()
       │                                          │
       │ ioctl(heapFd, DMA_HEAP_IOCTL_ALLOC)     │
       ├─────────────────────────────────────────►│ 分配 DMA 缓冲区
       │                                          │
       │ ioctl(bufferFd, DMA_BUF_IOCTL_SYNC)      │
       ├─────────────────────────────────────────►│ 缓存同步
       │                                          │
       │ close(bufferFd)                          │
       ├─────────────────────────────────────────►│ 释放缓冲区
```

```
┌──────────────┐                           ┌──────────────┐
│   应用进程   │                           │   内存追踪   │
│              │                           │    服务      │
└──────┬───────┘                           └──────┬───────┘
       │                                          │
       │ GetGraphicsMemory(pid)                   │
       ├─────────────────────────────────────────►│ GetDevMem()
       │                                          │
       │◄─────────────────────────────────────────┤ 返回内存信息
```

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   应用进程   │      │ libpurgeable │      │   内核空间   │
│              │      │     mem      │      │              │
└──────┬───────┘      └──────┬───────┘      └──────┬───────┘
       │                     │                     │
       │ Create(size, func)   │                     │
       ├────────────────────►│                     │
       │                     │ mmap(MAP_PURGEABLE) │
       │                     ├────────────────────►│
       │                     │                     │ 分配物理页
       │                     │                     │ ◄─ 可能回收
       │ BeginRead()         │                     │
       ├────────────────────►│                     │
       │                     │ 触发重建 (if purged) │
       │                     │ func(data, size)     │
       │                     │◄────────────────────┤
       │                     │                     │
       │ GetContent()        │                     │
       │◄────────────────────┤                     │
       │                     │                     │
       │ EndRead()           │                     │
       ├────────────────────►│                     │
       │                     │ munmap()            │
       │                     ├────────────────────►│ 释放内存
```

---

## 可利用风险点

### 风险 1: 空指针解引用 (空指针解引用)

**严重程度**: 🟡 中

**证据位置**: `libpurgeablemem/c/src/purgeable_memory.c:86`

```c
// BeginRead 函数实现
bool OH_PurgeableMemory_BeginRead(OH_PurgeableMemory *purgObj)
{
    // ... 
    if (purgObj == NULL) {
        return false;  // 有检查
    }
    // 但后续操作中可能存在未检查的路径
}
```

**触发条件**:
1. 调用 `BeginRead`/`BeginWrite` 时传入 NULL 指针
2. 虽然有 NULL 检查返回 false，但调用者可能忽略返回值

**潜在影响**:
- 应用程序崩溃（拒绝服务）
- 可能的竞态条件导致 Use-After-Free

**修复建议**:
```c
// 添加更严格的参数验证
bool OH_PurgeableMemory_BeginRead(OH_PurgeableMemory *purgObj)
{
    if (purgObj == NULL || purgObj->dataPtr == NULL) {
        return false;
    }
    // ...
}
```

---

### 风险 2: 整数溢出 (整数安全)

**严重程度**: 🟡 中

**证据位置**: `libpurgeablemem/c/src/purgeable_mem_c.c`

```c
// size 参数可能未做溢出检查
void *dataPtr = mmap(NULL, size, PROT_READ | PROT_WRITE,
                     MAP_ANONYMOUS | MAP_PURGEABLE, -1, 0);
```

**触发条件**:
1. 调用 `OH_PurgeableMemory_Create` 时传入过大的 size 值
2. size 接近 SIZE_MAX 导致 mmap 失败或行为异常

**潜在影响**:
- 内存分配失败导致服务降级
- 意外的内存占用

**修复建议**:
```c
// 添加大小限制检查
#define MAX_PURGEABLE_SIZE (1024 * 1024 * 1024) // 1GB

OH_PurgeableMemory *OH_PurgeableMemory_Create(
    size_t size,
    OH_PurgeableMemory_ModifyFunc func,
    void *funcPara)
{
    if (size == 0 || size > MAX_PURGEABLE_SIZE) {
        return NULL;
    }
    // ...
}
```

---

### 风险 3: 文件描述符泄漏 (资源管理)

**严重程度**: 🟡 中

**证据位置**: `libdmabufheap/src/dmabuf_alloc.c`

```c
int DmabufHeapOpen(const char *heapName)
{
    int heapFd = open(heapName, O_RDWR | O_CLOEXEC);
    // 错误处理后可能泄漏
    if (heapFd < 0) {
        return -1;
    }
    return heapFd;
}

int DmabufHeapBufferAlloc(unsigned int heapFd, DmabufHeapBuffer *buffer)
{
    // 如果 ioctl 失败，heapFd 仍保持打开状态
    int ret = ioctl(heapFd, DMA_HEAP_IOCTL_ALLOC, &data);
    if (ret < 0) {
        return ret;
    }
    // ...
}
```

**触发条件**:
1. `DmabufHeapBufferAlloc` 失败时，调用者未关闭 heapFd
2. 资源泄漏累积导致文件描述符耗尽

**潜在影响**:
- 拒绝服务（文件描述符耗尽）
- 系统资源耗尽

**修复建议**:
```c
// 提供资源管理封装
class DmabufHeap {
public:
    bool Open(const char *heapName) {
        fd_ = DmabufHeapOpen(heapName);
        return fd_ >= 0;
    }
    
    bool Alloc(DmabufHeapBuffer *buffer) {
        int ret = DmabufHeapBufferAlloc(fd_, buffer);
        if (ret < 0) {
            return false;
        }
        return true;
    }
    
    ~DmabufHeap() {
        if (fd_ >= 0) {
            DmabufHeapClose(fd_);
        }
    }
    
private:
    int fd_ = -1;
};
```

---

### 风险 4: 回调函数未验证 (代码注入)

**严重程度**: 🔴 高

**证据位置**: `libpurgeablemem/interfaces/kits/c/purgeable_memory.h:72`

```c
typedef bool (*OH_PurgeableMemory_ModifyFunc)(void *, size_t, void *);

OH_PurgeableMemory *OH_PurgeableMemory_Create(
    size_t size,
    OH_PurgeableMemory_ModifyFunc func,  // 未验证的回调
    void *funcPara)
```

**触发条件**:
1. 恶意应用传入伪造的 ModifyFunc
2. 回调函数执行任意代码

**潜在影响**:
- 任意代码执行
- 内存破坏

**修复建议**:
```c
// NDK 层无法验证函数指针来源，这是设计限制
// 但可以在文档中明确说明：
// - ModifyFunc 必须在同一进程内注册
// - 禁止跨进程传递函数指针
// - 建议使用静态函数或经过验证的回调

// 或者添加签名验证（如果使用 C++）：
#if USING_CXX
static_assert(std::is_function<ModifyFunc>::value, "Must be a function");
#endif
```

---

### 风险 5: 竞态条件 (竞态条件)

**严重程度**: 🟡 中

**证据位置**: `libpurgeablemem/common/src/ux_page_table_c.c`

```c
// 页表引用计数操作使用 CAS，但仍有竞态窗口
static void UxpteAdd(uxpte_t *pte, size_t incNum) {
    do {
        old = UxpteLoad(pte);
        if (IsUxpteUnderReclaim(old)) {
            sched_yield();  // 竞态窗口
            continue;
        }
        newVal = old + incNum;
    } while (!UxpteCAS_(pte, old, newVal));
}
```

**触发条件**:
1. 多线程并发调用 BeginRead/EndWrite
2. 内存回收与引用计数操作同时进行

**潜在影响**:
- 引用计数不一致
- Use-After-Free
- 内存泄漏

**修复建议**:
```c
// 使用更严格的锁机制
static pthread_mutex_t uxpt_lock = PTHREAD_MUTEX_INITIALIZER;

static void UxpteAdd(uxpte_t *pte, size_t incNum) {
    pthread_mutex_lock(&uxpt_lock);
    // 临界区操作
    pthread_mutex_unlock(&uxpt_lock);
}
```

---

### 风险 6: procfs 路径遍历 (路径安全)

**严重程度**: 🟡 中

**证据位置**: `libmeminfo/src/meminfo.cpp`

```cpp
uint64_t GetRssByPid(const int pid)
{
    char path[64];
    snprintf(path, sizeof(path), "/proc/%d/statm", pid);
    // pid 未验证，可能导致路径遍历
    int fd = open(path, O_RDONLY);
    // ...
}
```

**触发条件**:
1. pid 为负数或特殊值（如 `../`）
2. 尝试访问非预期的 procfs 路径

**潜在影响**:
- 绕过安全检查
- 读取意外文件

**修复建议**:
```cpp
bool IsValidPid(int pid) {
    return pid > 0 && pid < PID_MAX_LIMIT;
}

uint64_t GetRssByPid(const int pid)
{
    if (!IsValidPid(pid)) {
        return 0;
    }
    // ...
}
```

---

## 安全机制评估

### 已有的安全机制

| 机制 | 模块 | 说明 |
|------|------|------|
| CFI 防护 | 所有模块 | cfi=true, cfi_cross_dso=true |
| PAC 指针认证 | 所有模块 | branch_protector="pac_ret" |
| O_CLOEXEC | libdmabufheap | 文件描述符在 exec 时自动关闭 |
| NULL 检查 | libpurgeablemem | 主要 API 包含 NULL 参数检查 |

**证据来源**: `BUILD.gn` 配置

### 缺失的安全机制

| 机制 | 模块 | 建议 |
|------|------|------|
| 参数验证 | libpurgeablemem | size 范围检查 |
| 资源管理 | libdmabufheap | RAII 封装 |
| pid 验证 | libmeminfo | pid 有效性检查 |

---

## 安全最佳实践

### 对开发者的建议

#### 1. 使用 RAII 管理资源

```cpp
// ✅ 推荐：RAII 管理 DMA 缓冲区
class DmabufBuffer {
public:
    bool Alloc(const char *heapName, size_t size) {
        heapFd_ = DmabufHeapOpen(heapName);
        if (heapFd_ < 0) return false;
        
        DmabufHeapBuffer buffer;
        int ret = DmabufHeapBufferAlloc(heapFd_, &buffer);
        if (ret < 0) {
            DmabufHeapClose(heapFd_);
            return false;
        }
        buffer_ = buffer;
        return true;
    }
    
    ~DmabufBuffer() {
        if (buffer_.fd > 0) {
            DmabufHeapBufferFree(&buffer_);
        }
        if (heapFd_ > 0) {
            DmabufHeapClose(heapFd_);
        }
    }
    
private:
    int heapFd_ = -1;
    DmabufHeapBuffer buffer_ = {0};
};
```

#### 2. 验证所有输入参数

```c
// ✅ 推荐：参数验证
bool SafeBeginRead(OH_PurgeableMemory *purgObj) {
    if (purgObj == NULL) {
        HILOG_ERROR("Null purgObj");
        return false;
    }
    if (purgObj->dataSizeInput > MAX_SIZE) {
        HILOG_ERROR("Invalid size");
        return false;
    }
    return OH_PurgeableMemory_BeginRead(purgObj);
}
```

#### 3. 正确处理错误返回值

```c
// ❌ 不推荐：忽略返回值
OH_PurgeableMemory_BeginRead(purgObj);  // 可能返回 false
void *data = OH_PurgeableMemory_GetContent(purgObj);  // 访问可能无效

// ✅ 推荐：检查返回值
if (OH_PurgeableMemory_BeginRead(purgObj)) {
    void *data = OH_PurgeableMemory_GetContent(purgObj);
    // 使用数据
    OH_PurgeableMemory_EndRead(purgObj);
} else {
    // 处理重建失败
}
```

#### 4. 保护回调函数

```c
// ✅ 推荐：验证回调来源
static OH_PurgeableMemory_ModifyFunc registeredFunc = NULL;

void RegisterModifyFunc(OH_PurgeableMemory_ModifyFunc func) {
    // 只允许注册来自可信模块的函数
    if (IsCallerTrusted()) {
        registeredFunc = func;
    }
}
```

---

## 检查局限性声明

### 已检查范围

| 模块 | 文件 | 检查内容 |
|------|------|----------|
| libdmabufheap | dmabuf_alloc.c | 文件操作、ioctl 调用 |
| libmeminfo | meminfo.cpp | procfs 读取、HDI 调用 |
| libpurgeablemem | 所有 C/C++ 文件 | mmap、锁操作、回调处理 |

### 未检查范围

| 范围 | 说明 |
|------|------|
| 内核 DMA-Buf 实现 | 未审查内核代码 |
| HDI 服务安全性 | 未审查 MemoryTracker 服务 |
| 第三方依赖 | c_utils、hilog 等未审查 |
| 测试代码 | 按照规范忽略测试 |

### 工具和方法

- **静态分析**: 代码审查
- **动态测试**: 未进行（超出 Wiki 生成范围）
- **模糊测试**: 未进行
- **渗透测试**: 未进行

---

## 安全相关配置

### 编译时安全选项

| 选项 | 值 | 说明 |
|------|-----|------|
| sanitize.cfi | true | Control Flow Integrity |
| sanitize.cf i_cross_dso | true | 跨 DSO CFI |
| branch_protector | pac_ret | 指针认证码 |

### 运行时安全

| 机制 | 启用条件 |
|------|----------|
| ASLR | 系统默认启用 |
| SELinux/AppArmor | 根据系统配置 |
| seccomp | 根据系统配置 |

---

## 相关安全公告

**注意**: 本 Wiki 基于代码审查，未发现 CVE 级别的安全漏洞。如需安全响应，请联系 OpenHarmony 安全团队。

---

**最后更新**: 2026-02-06
**评审方法**: 代码静态审查
