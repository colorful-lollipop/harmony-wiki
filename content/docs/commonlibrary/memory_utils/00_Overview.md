# 项目概览

## 项目定位

`memory_utils` 是 OpenHarmony 公共基础库子系统（`commonlibrary`）中的**内存基础库部件**，为上层业务提供操作内存的系统库，确保上层业务的稳定性。

**证据来源**: `README.md:17`, `bundle.json:14-15`

```
subsystem: commonlibrary
part: memory_utils
description: memory base libraries
```

### 核心定位

| 维度 | 描述 |
|------|------|
| **子系统** | 公共基础库（commonlibrary） |
| **层级** | 系统库（Native 层） |
| **用户** | 多媒体服务、图形服务、内存管理服务 |
| **目标** | 提供高效、稳定的内存操作接口 |

---

## 核心能力

### 能力矩阵

| 模块 | 能力 | 适用场景 |
|------|------|----------|
| **libdmabufheap** | DMA 缓冲区分配与共享 | 多媒体编解码、GPU 渲染、零拷贝数据传输 |
| **libmeminfo** | 内存占用信息查询 | 内存监控、低内存查杀、性能分析 |
| **libpurgeablemem** | 可回收内存管理 | 图形图像缓存、大数据处理、内存敏感应用 |

### libdmabufheap 系统库

> 为多媒体相关服务提供分配共享内存的接口，通过在硬件设备和用户空间之间分配和共享内存，实现设备、进程间零拷贝内存，提升执行效率。

**证据来源**: `README_ZH.md:60-61`

| 能力 | 说明 |
|------|------|
| 零拷贝 | 避免 CPU 参与设备间数据传输 |
| DMA 堆支持 | 支持多种 DMA 堆设备（default, GPU, MEDIA_CODEC） |
| 缓存同步 | 支持 CPU/GPU 缓存一致性同步 |

**使用者**: 多媒体服务

### libmeminfo 系统库

> 提供内存占用查询接口，用于内存占用信息查询、低内存查杀等场景。

**证据来源**: `README_ZH.md:65`

| 能力 | 说明 |
|------|------|
| RSS 查询 | 获取进程常驻内存大小 |
| PSS 查询 | 获取比例共享内存（消除重复计算） |
| SwapPss 查询 | 获取交换分区内存占用 |
| GPU 内存查询 | 获取图形内存占用 |
| DMA 缓冲区查询 | 获取 DMA 缓冲区详细信息 |

**使用者**: 内存管理服务

### libpurgeable 系统库

> 为多媒体相关服务提供可丢弃类型内存的专用内存申请接口。在系统可用内存不足时，purgeable 内存被系统直接丢弃，实现内存快速回收。应用再次被使用时，已经被释放的 purgeable 内存能够进行重建。

**证据来源**: `README_ZH.md:73`

| 能力 | 说明 |
|------|------|
| 自动回收 | 内存压力时系统自动回收 |
| 数据重建 | 回收后通过回调函数重建数据 |
| 读保护 | BeginRead/EndRead 语义 |
| 写保护 | BeginWrite/EndWrite 语义 |

**使用者**: 图形图像相关服务

---

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **操作系统** | OpenHarmony 标准版 |
| **内核版本** | 支持 DMA-Buf 子系统、MAP_PURGEABLE、MAP_USEREXPTE |
| **系统能力** | SystemCapability.Kernel.Memory（NDK 接口需要） |

**证据来源**: `purgeable_memory.h:37`

### 运行时依赖

| 依赖库 | 用途 |
|--------|------|
| libutils.z.so | C/C++ 工具库 |
| libhilog.z.so | 日志系统 |
| libhitrace.z.so | 追踪系统 |
| libbegetutil.z.so | 初始化工具 |
| libipc_core.z.so | 进程间通信 |
| libmemmgrclient.z.so | 内存管理器客户端（libmeminfo 需要） |
| libmemorytracker_proxy_1.0.so | GPU 内存追踪代理（libmeminfo 需要） |

### 硬件要求

| 模块 | 硬件要求 |
|------|----------|
| libdmabufheap | 支持 DMA-Buf 的设备（GPU、视频编解码器等） |
| libmeminfo | 无特殊要求（读取 procfs 和 HDI） |
| libpurgeablemem | 支持 MAP_PURGEABLE 或 Ashmem 的内核 |

---

## 关键概念

### DMA-Buf

**DMA-Buf** 是 Linux 内核提供的缓冲区共享机制，允许不同设备（CPU、GPU、VPU 等）共享同一块物理内存，实现零拷贝数据传输。

**证据来源**: `libdmabufheap/include/dmabuf_alloc.h:19-21`

```c
#include <linux/dma-buf.h>
#include <linux/dma-heap.h>
```

**关键数据结构**:

```c
typedef struct {
    unsigned int fd;      // DMA buffer 文件描述符
    size_t size;          // 缓冲区大小
    __u64 heapFlags;      // 堆标志（用于所有者标识）
} DmabufHeapBuffer;
```

### Purgeable Memory（可回收内存）

**Purgeable Memory** 是一种特殊类型的内存，在系统内存压力时可以被回收，释放后需要通过回调函数重建数据。

**使用模式**:

```c
// 1. 创建时注册重建函数
OH_PurgeableMemory *mem = OH_PurgeableMemory_Create(size, RebuildFunc, para);

// 2. 使用前 BeginRead/BeginWrite (Pin 内存)
if (OH_PurgeableMemory_BeginRead(mem)) {
    void *data = OH_PurgeableMemory_GetContent(mem);
    // 访问数据...
    OH_PurgeableMemory_EndRead(mem);  // Unpin 内存
}

// 3. 内存可能被系统回收，再次访问时自动重建
// 4. 不需要时销毁
OH_PurgeableMemory_Destroy(&mem);
```

### UxPageTable（用户扩展页表）

**UxPageTable** 是用户空间页表的镜像，用于跟踪可回收内存的引用计数。

**证据来源**: `libpurgeablemem/cpp/include/ux_page_table.h`

```cpp
// UxPageTable — 用户扩展页表（内核页表镜像）
class UxPageTable {
    UxPageTableStruct *uxpt_;  // 指向 mmap(MAP_USEREXPTE) 的页表
};
```

### PSS（Proportional Set Size）

**PSS** 是比例共享内存大小，计算方式为进程独占内存 + 与其他进程共享内存按比例计算的部分。

**计算公式**:
```
PSS = 独占内存 + 共享内存 / 共享进程数
```

**用途**: 消除内存重复计算，更准确地反映进程的内存占用。

---

## 目录结构

```
/Volumes/lexar/code/d/work/oh/commonlibrary/memory_utils/
├── libdmabufheap/           # DMA (Direct Memory Access) 内存分配链接库
│   ├── include/            # 头文件目录
│   │   └── dmabuf_alloc.h  # DMA 堆分配主接口
│   ├── src/                # 源代码目录
│   │   └── dmabuf_alloc.c  # DMA 堆分配实现
│   └── test/               # 测试用例目录
│       └── dmabuf_alloc_test.cpp
│
├── libmeminfo/              # 内存占用查询库
│   ├── include/            # 头文件目录
│   │   └── meminfo.h       # 内存信息查询主接口
│   ├── src/                # 源代码目录
│   │   └── meminfo.cpp     # 内存信息查询实现
│   └── test/               # 测试用例目录
│       └── meminfo_test.cpp
│
├── libpurgeablemem/         # 可丢弃类型内存管理库
│   ├── cpp/                 # C++ 实现
│   │   ├── include/        # C++ 头文件
│   │   │   ├── purgeable_mem.h         # 可回收内存主类
│   │   │   ├── purgeable_mem_base.h   # 基类
│   │   │   ├── purgeable_mem_builder.h# 构建器
│   │   │   ├── purgeable_ashmem.h     # Ashmem 方案
│   │   │   └── ux_page_table.h        # 页表封装
│   │   └── src/            # C++ 源码
│   │       ├── purgeable_mem.cpp
│   │       ├── purgeable_mem_base.cpp
│   │       ├── purgeable_ashmem.cpp
│   │       └── ux_page_table.cpp
│   │
│   ├── c/                   # C 接口实现
│   │   ├── include/        # C 头文件
│   │   │   ├── purgeable_mem_c.h       # C 接口
│   │   │   ├── purgeable_mem_builder_c.h
│   │   │   └── pm_log_c.h
│   │   └── src/            # C 源码
│   │       ├── purgeable_mem_c.c
│   │       └── purgeable_memory.c
│   │
│   ├── common/              # 公共代码（C/C++ 共享）
│   │   ├── include/        # 公共头文件
│   │   │   ├── pm_state_c.h     # 状态码定义
│   │   │   ├── pm_util.h        # 工具函数
│   │   │   ├── pm_ptr_util.h    # 指针工具
│   │   │   └── ux_page_table_c.h
│   │   └── src/            # 公共源码
│   │       ├── pm_state_c.c
│   │       └── ux_page_table_c.c
│   │
│   ├── interfaces/          # 接口定义
│   │   └── kits/c/         # NDK 接口
│   │       ├── purgeable_memory.h    # NDK 头文件
│   │       └── libpurgeable_memory.ndk.json  # NDK 接口清单
│   │
│   └── test/               # 测试用例目录
│       ├── purgeable_c_test.cpp
│       ├── purgeable_cpp_test.cpp
│       ├── purgeable_memory_test.cpp
│       └── purgeableashmem_test.cpp
│
├── figures/                 # 架构图目录
│   ├── en-us_image_fwk.png
│   └── zh-cn_image_fwk.png
│
├── bundle.json             # 部件配置文件
├── purgeable_mem_config.gni # 构建配置导入
└── README.md / README_ZH.md  # 项目说明文档
```

**证据来源**: `README.md:35-44`, `README_ZH.md:36-51`

---

## 模块关系图

```
┌─────────────────────────────────────────────────────────────┐
│                     上层服务                                  │
│   多媒体服务    图形图像服务    内存管理服务    系统服务       │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│libdmabufheap  │ │libmeminfo     │ │libpurgeablemem│
│ DMA 内存分配   │ │ 内存信息查询  │ │ 可回收内存管理│
└───────────────┘ └───────────────┘ └───────────────┘
        │                │                │
        ▼                ▼                ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ DMA-Buf 子系统 │ │  procfs/HDI   │ │MAP_PURGEABLE │
│ Linux 内核    │ │  Linux 内核   │ │  Ashmem      │
└───────────────┘ └───────────────┘ └───────────────┘
```

**证据来源**: `README.md:21-31` (架构图)

---

## 版本信息

| 属性 | 值 |
|------|-----|
| **部件版本** | 3.1.0 |
| **ROM 占用** | 120KB |
| **RAM 占用** | 200KB |
| **NDK API 版本** | 1.0 |
| **NDK 起始版本** | 10 |

**证据来源**: `bundle.json:3`, `bundle.json:17-18`, `purgeable_memory.h:24-26`

---

## 使用说明

### 启用部件

系统开发者可以通过配置 `productdefine/common/products` 下的产品定义 JSON 文件来启用或停用本部件：

```json
"commonlibrary:memory_utils":{}
```

**证据来源**: `README_ZH.md:81-83`

### NDK 链接方式

使用 purgeable_memory NDK 接口时，需要链接以下库：

```cmake
target_link_libraries(your_target PRIVATE
    libpurgeable_memory_ndk.z.so
)
```

**证据来源**: `purgeable_memory.h:34`

### 编译开关

| 特性 | 描述 | 默认值 |
|------|------|--------|
| memory_utils_purgeable_ashmem_enable | 启用 Ashmem 方案 | 启用 |

**证据来源**: `bundle.json:78-79`

---

## 相关资料

- [OpenHarmony 官方文档](https://www.openharmony.cn/)
- [DMA-Buf 文档](https://www.kernel.org/doc/html/latest/driver-api/dma-buf.html)
- [NDK 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/ndk/Readme.md)

---

**最后更新**: 2026-02-06
