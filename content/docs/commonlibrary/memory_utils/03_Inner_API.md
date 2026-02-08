# 内部 API 文档

本文档描述 memory_utils 各模块的内部 C/C++ API，供二次开发参考。

> **注意**: 以下 API 主要供 OpenHarmony 系统内部使用，不属于 NDK 接口。

---

## 模块概览

| 模块 | 头文件 | 语言 | 稳定性 |
|------|--------|------|--------|
| libdmabufheap | dmabuf_alloc.h | C | 稳定 |
| libmeminfo | meminfo.h | C++ | 稳定 |
| libpurgeablemem | purgeable_mem.h 等 | C++ | 稳定 |
| libpurgeablemem | purgeable_mem_c.h 等 | C | 稳定 |

---

## libdmabufheap 内部接口

### 头文件

**路径**: `libdmabufheap/include/dmabuf_alloc.h`

**证据来源**: `bundle.json:42-44`

### 数据结构

#### DmabufHeapBufferSyncType

```c
// dmabuf_alloc.h:32-36
typedef enum {
    DMA_BUF_HEAP_BUF_SYNC_RW = DMA_BUF_SYNC_RW,   // 读写同步
    DMA_BUF_HEAP_BUF_SYNC_READ = DMA_BUF_SYNC_READ, // 读同步
    DMA_BUF_HEAP_BUF_SYNC_WRITE = DMA_BUF_SYNC_WRITE, // 写同步
} DmabufHeapBufferSyncType;
```

**说明**: DMA 缓冲区同步类型，与 Linux DMA-Buf 定义保持一致。

#### DmabufHeapBuffer

```c
// dmabuf_alloc.h:38-42
typedef struct {
    unsigned int fd;      // DMA buffer 文件描述符
    size_t size;          // 缓冲区大小
    __u64 heapFlags;      // 堆标志（用于所有者标识）
} DmabufHeapBuffer;
```

| 字段 | 类型 | 说明 |
|------|------|------|
| fd | unsigned int | DMA 缓冲区的文件描述符 |
| size | size_t | 缓冲区大小（字节） |
| heapFlags | __u64 | 堆标志，用于标识所有者 |

#### DmaHeapFlagOwnerId

```c
// dmabuf_alloc.h:44-49
enum DmaHeapFlagOwnerId {
    DMA_OWNER_DEFAULT,    // 默认所有者
    DMA_OWNER_GPU,        // GPU
    DMA_OWNER_MEDIA_CODEC,// 媒体编解码器
    COUNT_DMA_OWNER,
};
```

**说明**: 预定义的 DMA 堆所有者 ID。

### 函数 API

#### SetOwnerIdForHeapFlags

```c
// dmabuf_alloc.h:51
void SetOwnerIdForHeapFlags(DmabufHeapBuffer *buffer, enum DmaHeapFlagOwnerId ownerId);
```

**功能**: 根据所有者 ID 设置 DMA 缓冲区的堆标志。

| 参数 | 类型 | 说明 |
|------|------|------|
| buffer | DmabufHeapBuffer* | 目标缓冲区 |
| ownerId | DmaHeapFlagOwnerId | 所有者 ID |

**头文件**: `dmabuf_alloc.h:51`

---

#### DmabufHeapOpen

```c
// dmabuf_alloc.h:53
int DmabufHeapOpen(const char *heapName);
```

**功能**: 打开指定的 DMA 堆设备。

| 参数 | 类型 | 说明 |
|------|------|------|
| heapName | const char* | DMA 堆名称（如 "default", "gpu", "media_codec"） |
| **返回值** | int | 成功返回文件描述符；失败返回 -1 并设置 errno |

**系统调用**: `open("/dev/dma_heap/{heapName}", O_RDWR | O_CLOEXEC)`

**错误码**:
- `ENOENT`: 设备不存在
- `EACCES`: 权限不足
- `EMFILE`: 文件描述符耗尽

**头文件**: `dmabuf_alloc.h:53`

---

#### DmabufHeapClose

```c
// dmabuf_alloc.h:55
int DmabufHeapClose(unsigned int fd);
```

**功能**: 关闭 DMA 堆设备文件描述符。

| 参数 | 类型 | 说明 |
|------|------|------|
| fd | unsigned int | 要关闭的文件描述符 |
| **返回值** | int | 成功返回 0；失败返回 -1 并设置 errno |

**头文件**: `dmabuf_alloc.h:55`

---

#### DmabufHeapBufferAlloc

```c
// dmabuf_alloc.h:57
int DmabufHeapBufferAlloc(unsigned int heapFd, DmabufHeapBuffer *buffer);
```

**功能**: 从指定的 DMA 堆分配一个缓冲区。

| 参数 | 类型 | 说明 |
|------|------|------|
| heapFd | unsigned int | DMA 堆设备文件描述符 |
| buffer | DmabufHeapBuffer* | 输出参数，成功时填充 fd 和 size |
| **返回值** | int | 成功返回 0；失败返回负值错误码 |

**系统调用**: `ioctl(heapFd, DMA_HEAP_IOCTL_ALLOC, &data)`

**头文件**: `dmabuf_alloc.h:57`

---

#### DmabufHeapBufferFree

```c
// dmabuf_alloc.h:59
int DmabufHeapBufferFree(DmabufHeapBuffer *buffer);
```

**功能**: 释放 DMA 缓冲区。

| 参数 | 类型 | 说明 |
|------|------|------|
| buffer | DmabufHeapBuffer* | 要释放的缓冲区 |
| **返回值** | int | 成功返回 0；失败返回负值错误码 |

**操作**: 关闭 buffer->fd

**头文件**: `dmabuf_alloc.h:59`

---

#### DmabufHeapBufferSyncStart

```c
// dmabuf_alloc.h:61
int DmabufHeapBufferSyncStart(unsigned int bufferFd, DmabufHeapBufferSyncType syncType);
```

**功能**: 开始 DMA 缓冲区同步。

| 参数 | 类型 | 说明 |
|------|------|------|
| bufferFd | unsigned int | DMA 缓冲区文件描述符 |
| syncType | DmabufHeapBufferSyncType | 同步类型 |
| **返回值** | int | 成功返回 0；失败返回负值错误码 |

**系统调用**: `ioctl(bufferFd, DMA_BUF_IOCTL_SYNC, &sync)`

**头文件**: `dmabuf_alloc.h:61`

---

#### DmabufHeapBufferSyncEnd

```c
// dmabuf_alloc.h:63
int DmabufHeapBufferSyncEnd(unsigned int bufferFd, DmabufHeapBufferSyncType syncType);
```

**功能**: 结束 DMA 缓冲区同步。

| 参数 | 类型 | 说明 |
|------|------|------|
| bufferFd | unsigned int | DMA 缓冲区文件描述符 |
| syncType | DmabufHeapBufferSyncType | 同步类型 |
| **返回值** | int | 成功返回 0；失败返回负值错误码 |

**头文件**: `dmabuf_alloc.h:63`

---

### 使用示例

```c
#include "dmabuf_alloc.h"
#include <stdio.h>
#include <unistd.h>

int main() {
    DmabufHeapBuffer buffer = {0};
    
    // 1. 打开 DMA 堆
    int heapFd = DmabufHeapOpen("default");
    if (heapFd < 0) {
        printf("Failed to open DMA heap\n");
        return -1;
    }
    
    // 2. 分配缓冲区
    int ret = DmabufHeapBufferAlloc(heapFd, &buffer);
    if (ret < 0) {
        printf("Failed to allocate buffer\n");
        DmabufHeapClose(heapFd);
        return -1;
    }
    
    // 3. 开始同步
    DmabufHeapBufferSyncStart(buffer.fd, DMA_BUF_HEAP_BUF_SYNC_RW);
    
    // 4. 使用缓冲区进行 DMA 传输...
    
    // 5. 结束同步
    DmabufHeapBufferSyncEnd(buffer.fd, DMA_BUF_HEAP_BUF_SYNC_RW);
    
    // 6. 释放缓冲区
    DmabufHeapBufferFree(&buffer);
    DmabufHeapClose(heapFd);
    
    return 0;
}
```

---

## libmeminfo 内部接口

### 头文件

**路径**: `libmeminfo/include/meminfo.h`

**证据来源**: `bundle.json:50-53`

### 命名空间

```cpp
namespace OHOS {
namespace MemInfo {
```

### 数据结构

#### DmaNodeInfo

```cpp
// meminfo.h:27-49
struct DmaNodeInfo {
    char process[MAX_STRING_LEN];       // 进程名
    int process_size;                   // 进程名长度
    int pid;                            // 进程 ID
    int fd;                             // 文件描述符
    int64_t size_bytes;                 // 大小（字节）
    int64_t ino;                        // inode 号
    int exp_pid;                        // 导出进程 ID
    char exp_task_comm[MAX_STRING_LEN]; // 导出任务名
    int exp_task_comm_size;
    char buf_name[MAX_STRING_LEN];      // 缓冲区名称
    int buf_name_size;
    char exp_name[MAX_STRING_LEN];      // 导出进程名
    int exp_name_size;
    bool can_reclaim;                   // 是否可回收
    bool is_reclaim;                    // 是否已回收
    char buf_type[MAX_STRING_LEN];      // 缓冲区类型
    int buf_type_size;
    char reclaim_info[MAX_STRING_LEN];  // 回收信息
    int reclaim_info_size;
    char leak_type[MAX_STRING_LEN];     // 泄漏类型
    int leak_type_size;
};
```

#### DmaNodeInfoWrapper

```cpp
// meminfo.h:51-85
struct DmaNodeInfoWrapper {
    std::string process;
    int pid;
    int fd;
    int64_t size_bytes;
    int64_t ino;
    int exp_pid;
    std::string exp_task_comm;
    std::string buf_name;
    std::string exp_name;
    bool can_reclaim;
    bool is_reclaim;
    std::string buf_type;
    std::string reclaim_info;
    std::string leak_type;
    
    void print() const;
};
```

### 函数 API

#### GetDmaInfo

```cpp
// meminfo.h:88
std::vector<DmaNodeInfoWrapper> GetDmaInfo(int pid);
```

**功能**: 获取指定进程的 DMA 缓冲区信息（去重后）。

| 参数 | 类型 | 说明 |
|------|------|------|
| pid | int | 目标进程 ID，-1 表示所有进程 |
| **返回值** | std::vector<DmaNodeInfoWrapper> | DMA 缓冲区信息列表 |

**数据来源**: `libmemmgrclient.z.so`

**头文件**: `meminfo.h:88`

---

#### GetRssByPid

```cpp
// meminfo.h:91
uint64_t GetRssByPid(const int pid);
```

**功能**: 获取指定进程的 RSS（常驻内存大小）。

| 参数 | 类型 | 说明 |
|------|------|------|
| pid | const int | 目标进程 ID |
| **返回值** | uint64_t | RSS 大小（字节）；失败返回 0 |

**数据来源**: `/proc/{pid}/statm`

**头文件**: `meminfo.h:91`

---

#### GetPssByPid

```cpp
// meminfo.h:94
uint64_t GetPssByPid(const int pid);
```

**功能**: 获取指定进程的 PSS（比例共享内存大小）。

| 参数 | 类型 | 说明 |
|------|------|------|
| pid | const int | 目标进程 ID |
| **返回值** | uint64_t | PSS 大小（字节）；失败返回 0 |

**数据来源**: `/proc/{pid}/smaps_rollup`

**头文件**: `meminfo.h:94`

---

#### GetSwapPssByPid

```cpp
// meminfo.h:97
uint64_t GetSwapPssByPid(const int pid);
```

**功能**: 获取指定进程的 SwapPss（交换分区 PSS）。

| 参数 | 类型 | 说明 |
|------|------|------|
| pid | const int | 目标进程 ID |
| **返回值** | uint64_t | SwapPss 大小（字节）；失败返回 0 |

**数据来源**: `/proc/{pid}/smaps_rollup`

**头文件**: `meminfo.h:97`

---

#### GetGraphicsMemory

```cpp
// meminfo.h:100
bool GetGraphicsMemory(const int pid, uint64_t &gl, uint64_t &graph);
```

**功能**: 获取指定进程的 GPU 图形内存。

| 参数 | 类型 | 说明 |
|------|------|------|
| pid | const int | 目标进程 ID |
| gl | uint64_t& | 输出：GL 内存大小 |
| graph | uint64_t& | 输出：图形内存大小 |
| **返回值** | bool | 成功返回 true；失败返回 false |

**数据来源**: HDI `MemoryTracker::GetDevMem()`

**头文件**: `meminfo.h:100`

---

### 使用示例

```cpp
#include "meminfo.h"
#include <iostream>

int main() {
    int pid = getpid();
    
    // 1. 获取 RSS
    uint64_t rss = OHOS::MemInfo::GetRssByPid(pid);
    std::cout << "RSS: " << rss << " bytes" << std::endl;
    
    // 2. 获取 PSS
    uint64_t pss = OHOS::MemInfo::GetPssByPid(pid);
    std::cout << "PSS: " << pss << " bytes" << std::endl;
    
    // 3. 获取 GPU 内存
    uint64_t gl = 0, graph = 0;
    if (OHOS::MemInfo::GetGraphicsMemory(pid, gl, graph)) {
        std::cout << "GL Memory: " << gl << " bytes" << std::endl;
        std::cout << "Graph Memory: " << graph << " bytes" << std::endl;
    }
    
    // 4. 获取 DMA 缓冲区信息
    auto dmaInfo = OHOS::MemInfo::GetDmaInfo(pid);
    for (const auto &info : dmaInfo) {
        std::cout << "DMA Buffer: " << info.buf_name
                  << " Size: " << info.size_bytes << " bytes" << std::endl;
    }
    
    return 0;
}
```

---

## libpurgeablemem 内部接口 (C++)

### 头文件

| 头文件 | 用途 |
|--------|------|
| `purgeable_mem_base.h` | 基类定义 |
| `purgeable_mem.h` | 可回收内存主类 |
| `purgeable_mem_builder.h` | 构建器模式 |
| `purgeable_ashmem.h` | Ashmem 方案 |
| `ux_page_table.h` | 页表封装 |

**证据来源**: `bundle.json:58-68`

### 命名空间

```cpp
namespace OHOS {
namespace PurgeableMem {
```

### PurgeableMemBase 基类

```cpp
// purgeable_mem_base.h
class PurgeableMemBase {
public:
    virtual ~PurgeableMemBase() = default;
    
    // 读保护
    bool BeginRead();
    void EndRead();
    
    // 写保护
    bool BeginWrite();
    void EndWrite();
    
    // 获取内容
    void *GetContent();
    size_t GetContentSize() const;
    
protected:
    virtual bool Pin() = 0;
    virtual bool Unpin() = 0;
    virtual bool IsPurged() = 0;
    virtual int GetPinStatus() const = 0;
    virtual bool CreatePurgeableData() = 0;
    virtual void AfterRebuildSucc() = 0;
    
    void *dataPtr_ = nullptr;
    size_t dataSizeInput_ = 0;
    std::unique_ptr<PurgeableMemBuilder> builder_;
    std::mutex dataLock_;
    unsigned int buildDataCount_ = 0;
    bool isDataValid_ = true;
};
```

**头文件**: `libpurgeablemem/cpp/include/purgeable_mem_base.h`

---

### PurgeableMem 主类

```cpp
// purgeable_mem.h
class PurgeableMem : public PurgeableMemBase {
public:
    PurgeableMem(size_t dataSize, std::unique_ptr<PurgeableMemBuilder> builder);
    ~PurgeableMem();
    
    void ResizeData(size_t newSize) override;

protected:
    bool Pin() override;
    bool Unpin() override;
    bool IsPurged() override;
    int GetPinStatus() const override;
    bool CreatePurgeableData() override;
    void AfterRebuildSucc() override;
    std::string ToString() const override;

private:
    std::unique_ptr<UxPageTable> pageTable_ = nullptr;
};
```

**头文件**: `libpurgeablemem/cpp/include/purgeable_mem.h`

---

### PurgeableAshMem 类

```cpp
// purgeable_ashmem.h
class PurgeableAshMem : public PurgeableMemBase {
public:
    PurgeableAshMem(size_t dataSize, std::unique_ptr<PurgeableMemBuilder> builder);
    ~PurgeableAshMem();
    
    void ResizeData(size_t newSize) override;

protected:
    bool Pin() override;
    bool Unpin() override;
    bool IsPurged() override;
    int GetPinStatus() const override;
    bool CreatePurgeableData() override;
    void AfterRebuildSucc() override;
    std::string ToString() const override;

private:
    int ashmemFd_ = -1;
    AshmemPinState pin_ = PIN_STATE_UNPIN;
};
```

**头文件**: `libpurgeablemem/cpp/include/purgeable_ashmem.h`

---

### UxPageTable 类

```cpp
// ux_page_table.h
class UxPageTable {
public:
    explicit UxPageTable(void *dataPtr, size_t dataSize);
    ~UxPageTable();
    
    bool GetUxpte(void *addr, size_t size);
    bool PutUxpte(void *addr, size_t size);
    bool CheckPresent(void *addr, size_t size);
    
private:
    UxPageTableStruct *uxpt_ = nullptr;
};
```

**头文件**: `libpurgeablemem/cpp/include/ux_page_table.h`

---

### PurgeableMemBuilder 构建器

```cpp
// purgeable_mem_builder.h
class PurgeableMemBuilder {
public:
    PurgeableMemBuilder() = default;
    virtual ~PurgeableMemBuilder() = default;
    
    virtual bool BuildAll(void *data, size_t size);
    virtual void Append(std::unique_ptr<PurgeableMemBuilder> next);
    
protected:
    std::unique_ptr<PurgeableMemBuilder> next_ = nullptr;
};
```

**头文件**: `libpurgeablemem/cpp/include/purgeable_mem_builder.h`

---

## libpurgeablemem 内部接口 (C)

### 头文件

| 头文件 | 用途 |
|--------|------|
| `purgeable_mem_c.h` | C 接口 |
| `purgeable_mem_builder_c.h` | C 构建器 |
| `pm_state_c.h` | 状态码 |
| `ux_page_table_c.h` | C 页表操作 |

### 状态码定义

```c
// pm_state_c.h
typedef enum {
    PM_OK = 0,                       // 成功
    PM_MMAP_PURG_FAIL,               // mmap 可回收内存失败
    PM_MMAP_UXPT_FAIL,               // 用户页表分配失败
    PM_UXPT_OUT_RANGE,               // 地址越界
    PM_UXPT_NO_PRESENT,              // 页表项不存在
    PM_LOCK_READ_FAIL,               // 读锁获取失败
    PMB_BUILD_ALL_FAIL,               // 数据重建失败
    PM_MALLOC_FAIL,                  // 内存分配失败
    PM_INVALID_ARGUMENT,              // 无效参数
    PM_UNKNOWN_ERROR,                // 未知错误
} PMState;
```

**证据来源**: `libpurgeablemem/common/include/pm_state_c.h`

---

## API 稳定性标注

| API | 头文件 | 稳定性 | 标注依据 |
|-----|--------|--------|----------|
| `DmabufHeapOpen` | dmabuf_alloc.h | 稳定 | 系统库导出，Bundle 声明 |
| `DmabufHeapBufferAlloc` | dmabuf_alloc.h | 稳定 | 系统库导出，Bundle 声明 |
| `GetRssByPid` | meminfo.h | 稳定 | 系统库导出，Bundle 声明 |
| `GetPssByPid` | meminfo.h | 稳定 | 系统库导出，Bundle 声明 |
| `PurgeableMem` | purgeable_mem.h | 稳定 | C++ 实现类 |
| `PurgeableAshMem` | purgeable_ashmem.h | 稳定 | 可选方案 |
| `OH_PurgeableMemory_*` | purgeable_memory.h | **NDK 稳定** | NDK 接口，版本 1.0 |

---

## 依赖方向

```
应用层
   │
   ├──► libdmabufheap ───────────► 内核 (DMA-Buf)
   │
   ├──► libmeminfo ──────────────► procfs / HDI
   │
   └──► libpurgeablemem ─────────► 内核 (MAP_PURGEABLE / Ashmem)
          │
          └──► libipc_core ──────► IPC
          └──► libhitrace ───────► 追踪
```

---

**最后更新**: 2026-02-06
