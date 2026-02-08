# 目录结构与模块职责

> surface_lite 代码组织与文件职责

---

## 1. 顶层目录结构

```
foundation/graphic/surface_lite/
├── bundle.json              # 组件配置文件 (OHOS 组件元数据)
├── BUILD.gn                 # 根构建脚本
├── LICENSE                  # Apache License 2.0
├── README.md                # 英文项目说明
├── README_zh.md             # 中文项目说明
├── OAT.xml                  # OSS 审计文件
├── .clang-format           # 代码格式配置
│
├── figures/                 # 文档图片资源
│   ├── position-of-a-surface-in-the-system-architecture.png
│   ├── surface-rotation-process.png
│   └── icon-notice.gif
│
├── interfaces/              # 接口定义目录
│   ├── kits/               # 对外公开 API
│   └── innerkits/          # 模块内部 API
│
├── frameworks/              # 框架实现代码
│   └── *.cpp / *.h         # C++ 源文件和私有头文件
│
└── test/                    # 测试代码
    ├── BUILD.gn             # 测试构建脚本
    ├── fuzztest/            # Fuzz 测试
    └── unittest/            # 单元测试
```

---

## 2. 接口层 (interfaces/)

### 2.1 对外 API (interfaces/kits/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| **surface.h** | 321 | Surface 主接口定义 | `class Surface` |
| **surface_buffer.h** | 155 | Buffer 接口定义 | `class SurfaceBuffer` |
| **surface_type.h** | 73 | 类型和常量定义 | `SURFACE_MAX_*`, `BufferConsumerUsage` |
| **ibuffer_consumer_listener.h** | 59 | 消费者监听接口 | `class IBufferConsumerListener` |

**使用方**:
- window_window_manager_lite
- arkui_ui_lite
- 其他图形/媒体模块

**包含关系**:
```
surface.h
  ├── surface_buffer.h
  ├── surface_type.h
  └── ibuffer_consumer_listener.h
      └── surface_type.h (间接)
```

### 2.2 内部 API (interfaces/innerkits/)

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| **surface_impl.h** | 243 | Surface 实现类 | `class SurfaceImpl` |
| **surface_buffer_impl.h** | 377 | Buffer 实现类 | `class SurfaceBufferImpl`, `BufferState` |
| **buffer_queue.h** | 226 | Buffer 队列核心 | `class BufferQueue` |
| **buffer_queue_consumer.h** | 61 | 消费者封装 | `class BufferQueueConsumer` |
| **buffer_producer.h** | 193 | 生产者抽象 | `class BufferProducer`, `SURFACE_REQUEST_CODE` |
| **buffer_common.h** | 46 | 公共定义 | `BufferErrorCode`, `RETURN_*_IF_FAIL` |

**模块内部使用**:
- frameworks/ 下的实现文件
- 不对外暴露

---

## 3. 实现层 (frameworks/)

### 3.1 源文件清单

| 文件 | 行数 | 职责 | 对应头文件 |
|------|------|------|-----------|
| **surface.cpp** | 39 | Surface 工厂方法 | interfaces/kits/surface.h |
| **surface_impl.cpp** | 291 | SurfaceImpl 实现 | interfaces/innerkits/surface_impl.h |
| **surface_buffer_impl.cpp** | 219 | SurfaceBufferImpl 实现 | interfaces/innerkits/surface_buffer_impl.h |
| **buffer_queue.cpp** | 447 | BufferQueue 实现 | interfaces/innerkits/buffer_queue.h |
| **buffer_queue_consumer.cpp** | 42 | BufferQueueConsumer 实现 | interfaces/innerkits/buffer_queue_consumer.h |
| **buffer_queue_producer.cpp** | 397 | BufferQueueProducer 实现 | frameworks/buffer_queue_producer.h |
| **buffer_client_producer.cpp** | 351 | BufferClientProducer 实现 | frameworks/buffer_client_producer.h |
| **buffer_manager.cpp** | 284 | BufferManager 实现 | frameworks/buffer_manager.h |

### 3.2 私有头文件

| 文件 | 职责 |
|------|------|
| **buffer_queue_producer.h** | BufferQueueProducer 类声明 |
| **buffer_client_producer.h** | BufferClientProducer 类声明 |
| **buffer_manager.h** | BufferManager 单例声明 |

### 3.3 文件职责详解

#### surface.cpp
```cpp
// 职责: 提供 Surface 的静态工厂方法
namespace OHOS {
    Surface* Surface::CreateSurface() {
        // 创建 SurfaceImpl 实例并初始化
    }
}
```
- **设计模式**: 工厂模式
- **目的**: 隐藏实现细节，对外提供统一创建接口

#### surface_impl.cpp
```cpp
// 职责: Surface 的核心实现
class SurfaceImpl : public Surface {
    // - 初始化 BufferQueue/Producer/Consumer
    // - 属性设置代理到 Producer
    // - Buffer 操作代理到 Producer/Consumer
    // - IPC 消息处理
};
```
- **核心方法**: `Init()`, `DoIpcMsg()`, `WriteIoIpcIo()`
- **模式**: 代理模式 (Producer/Consumer 代理)

#### surface_buffer_impl.cpp
```cpp
// 职责: SurfaceBuffer 实现 + IPC 序列化
class SurfaceBufferImpl : public SurfaceBuffer {
    // - 基础属性管理 (size, addr, usage)
    // - 额外数据管理 (extDatas_ map)
    // - IPC 序列化/反序列化
};
```
- **关键数据结构**: `ExtraData`, `SurfaceBufferData`
- **方法**: `ReadFromIpcIo()`, `WriteToIpcIo()`

#### buffer_queue.cpp
```cpp
// 职责: Buffer 队列核心管理
class BufferQueue {
    // - freeList_/dirtyList_/allBuffers_ 管理
    // - Buffer 状态机转换
    // - 线程同步 (mutex/cond)
    // - Buffer 分配/释放策略
};
```
- **核心数据结构**: `std::list<SurfaceBufferImpl*>`
- **同步原语**: `pthread_mutex_t`, `pthread_cond_t`

#### buffer_queue_consumer.cpp
```cpp
// 职责: Consumer 端代理封装
class BufferQueueConsumer {
    // 仅包装 BufferQueue 的消费者方法
};
```
- **设计**: 简化接口，仅暴露 `AcquireBuffer()`/`ReleaseBuffer()`

#### buffer_queue_producer.cpp
```cpp
// 职责: Producer 端实现 + IPC 处理
class BufferQueueProducer : public BufferProducer {
    // - 本地 Producer 操作
    // - IPC 消息处理表 (g_ipcMsgHandleList)
    // - 消费者回调通知
};
```
- **关键表**: `g_ipcMsgHandleList[20]` - IPC 请求分发
- **回调**: `consumerListener_->OnBufferAvailable()`

#### buffer_client_producer.cpp
```cpp
// 职责: 跨进程 Producer 代理
class BufferClientProducer : public BufferProducer {
    // - 通过 IPC 发送请求到远程 BufferQueueProducer
    // - 处理 IPC 响应和内存映射
};
```
- **核心**: `SendRequest()` 调用
- **映射**: `BufferManager::MapBuffer()`

#### buffer_manager.cpp
```cpp
// 职责: Buffer 内存管理单例
class BufferManager {
    // - Gralloc 接口封装
    // - Buffer 分配/释放
    // - 内存映射/解映射
    // - Cache 刷新
};
```
- **单例模式**: `static BufferManager instance`
- **底层接口**: `GrallocFuncs* grallocFucs_`

---

## 4. 测试目录 (test/)

### 4.1 目录结构

```
test/
├── BUILD.gn                     # 测试构建配置
├── fuzztest/                    # Fuzz 测试 (待填充)
└── unittest/
    └── graphic_surface_test.cpp # 单元测试用例
```

### 4.2 测试内容 (graphic_surface_test.cpp)

| 测试用例 | 测试内容 |
|----------|----------|
| surface_buffer_001 | Buffer 初始状态检查 |
| surface_buffer_002 | Buffer 设置/获取 size |
| surface_buffer_003 | Buffer 设置/获取 format |
| surface_set_001 | 设置队列大小 |
| surface_set_002 | 设置宽高 |
| surface_set_003 | 设置格式 |
| surface_set_004 | 设置 stride 对齐 |
| surface_set_005 | 设置 size |
| surface_set_006 | 设置 usage |
| surface_set_007 | 设置/获取 user data |
| surface_001 | Surface 创建 |
| surface_002 | Request/Flush Buffer |
| surface_003 | Request/Cancel Buffer |
| surface_004 | Acquire/Release Buffer |
| surface_005 | 队列大小变更 |
| surface_006 | Buffer 轮转流程 |

**注意**: 本文档不包含测试代码的详细分析。

---

## 5. 文件依赖关系

### 5.1 头文件包含图

```
interfaces/kits/surface.h
    ├── interfaces/kits/ibuffer_consumer_listener.h
    ├── interfaces/kits/surface_buffer.h
    │   └── <map> (STL)
    └── interfaces/kits/surface_type.h
        └── gfx_utils/pixel_format_utils.h (external)

interfaces/innerkits/surface_impl.h
    ├── interfaces/innerkits/buffer_producer.h
    │   ├── interfaces/innerkits/buffer_queue.h
    │   │   ├── interfaces/innerkits/surface_buffer_impl.h
    │   │   │   ├── interfaces/innerkits/buffer_common.h
    │   │   │   │   └── gfx_utils/graphic_log.h (external)
    │   │   │   ├── <ipc_skeleton.h> (external)
    │   │   │   └── interfaces/kits/surface_buffer.h
    │   │   └── <list>, <map> (STL)
    │   └── interfaces/kits/surface_buffer.h
    ├── interfaces/innerkits/buffer_queue_consumer.h
    ├── interfaces/kits/ibuffer_consumer_listener.h
    ├── interfaces/kits/surface.h
    └── interfaces/kits/surface_type.h

frameworks/buffer_manager.h
    ├── interfaces/innerkits/surface_buffer_impl.h
    ├── interfaces/innerkits/surface_type.h
    └── display_gralloc.h (external)
```

### 5.2 编译依赖图

```
shared_library("surface")
    ├── sources: [frameworks/*.cpp]
    ├── include_dirs:
    │   ├── frameworks/                    (private)
    │   ├── //drivers/peripheral/base      (external)
    │   └── //drivers/peripheral/display/interfaces/include (external)
    ├── public_configs:
    │   └── surface_public_config
    │       ├── interfaces/innerkits/      (public)
    │       ├── interfaces/kits/           (public)
    │       └── //foundation/graphic/graphic_utils_lite/interfaces/kits (external)
    ├── public_deps:
    │   └── //foundation/graphic/graphic_utils_lite:utils_lite
    └── deps:
        ├── //drivers/peripheral/display/hal:hdi_display
        ├── //foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single
        └── //third_party/bounds_checking_function:libsec_shared
```

---

## 6. 职责边界

### 6.1 模块职责

| 模块 | 职责 | 不做的 |
|------|------|--------|
| **Surface** | Buffer 生命周期管理 | 不管理窗口 |
| **BufferQueue** | 队列状态管理 | 不分配内存 |
| **BufferManager** | 内存分配/映射 | 不管理队列状态 |
| **Producer** | 生产 Buffer | 不消费 Buffer |
| **Consumer** | 消费 Buffer | 不生产 Buffer |

### 6.2 文件职责矩阵

```
                    创建   配置   请求   提交   获取   释放   IPC   内存
surface.cpp         ●
surface_impl.cpp    ●      ●      ●      ●      ●      ●      ●
surface_buffer_impl.cpp                         ●            ●      ○
buffer_queue.cpp           ●      ●      ●      ●      ●            ○
buffer_queue_consumer.cpp                         ●      ●
buffer_queue_producer.cpp  ●      ●      ●      ●            ●
buffer_client_producer.cpp ●      ●      ●      ●      ○      ○      ●
buffer_manager.cpp                                            ●      ●

● = 主要职责
○ = 次要职责/触发
```

---

## 7. 代码统计

### 7.1 代码行数统计

| 目录 | 文件数 | 代码行数 | 占比 |
|------|--------|----------|------|
| interfaces/kits | 4 | ~600 | 15% |
| interfaces/innerkits | 6 | ~1100 | 28% |
| frameworks | 11 | ~2300 | 57% |
| **总计** | **21** | **~4000** | **100%** |

### 7.2 类数量统计

| 类型 | 数量 | 类名 |
|------|------|------|
| 对外接口类 | 2 | Surface, SurfaceBuffer |
| 实现类 | 2 | SurfaceImpl, SurfaceBufferImpl |
| 队列管理类 | 3 | BufferQueue, BufferQueueConsumer, BufferQueueProducer |
| Producer 类 | 2 | BufferProducer(抽象), BufferClientProducer |
| 管理器 | 1 | BufferManager |
| 监听器接口 | 1 | IBufferConsumerListener |
| **总计** | **11** | - |

---

## 8. 开发规范

### 8.1 文件命名规范

| 类型 | 命名模式 | 示例 |
|------|----------|------|
| 对外头文件 | `*.h` | surface.h, surface_buffer.h |
| 内部头文件 | `*_impl.h`, `buffer_*.h` | surface_impl.h, buffer_queue.h |
| 实现文件 | `*.cpp` | surface_impl.cpp, buffer_queue.cpp |
| 私有头文件 | `buffer_*.h` (frameworks/) | buffer_manager.h |

### 8.2 代码组织原则

1. **接口与实现分离**: interfaces/ vs frameworks/
2. **对外/对内分离**: kits/ vs innerkits/
3. **单一职责**: 每个类有明确职责边界
4. **依赖倒置**: 依赖抽象 (BufferProducer) 而非具体实现

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
