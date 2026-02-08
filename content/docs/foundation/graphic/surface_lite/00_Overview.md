# 项目概览

> surface_lite - OpenHarmony 轻量图形 Surface Buffer 模块

---

## 1. 项目定位

### 1.1 一句话描述

**surface_lite** 是 OpenHarmony 图形子系统中的**共享内存管理模块**，用于在图形和媒体场景中管理和传输共享内存。

### 1.2 所属子系统

- **子系统**: `graphic` (图形子系统)
- **组件名**: `@ohos/surface_lite`
- **源码路径**: `foundation/graphic/surface_lite`

### 1.3 适用系统类型

| 系统类型 | 支持状态 | 说明 |
|----------|----------|------|
| **small** (轻量) | ✅ 完全支持 | 主要目标平台 |
| standard (标准) | ⚠️ 有限支持 | 推荐使用完整版 surface |
| large (大型) | ❌ 不支持 | - |

---

## 2. 核心能力

### 2.1 主要功能

1. **共享内存管理**
   - 虚拟内存分配 (BUFFER_CONSUMER_USAGE_SORTWARE)
   - 物理内存分配 (BUFFER_CONSUMER_USAGE_HARDWARE)
   - 缓存物理内存 (BUFFER_CONSUMER_USAGE_*_CACHE)

2. **跨进程传输**
   - 控制结构: IPC 句柄传输（有拷贝）
   - 图形/媒体数据: 共享内存（零拷贝）

3. **生产者-消费者模式**
   - Producer: UI 渲染、视频解码
   - Consumer: WMS 合成、显示输出

4. **Buffer 队列管理**
   - Free 队列: 可申请的 Buffer
   - Dirty 队列: 待消费的 Buffer
   - 支持队列大小动态调整 (1-10)

### 2.2 典型应用场景

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   UI 渲染引擎    │────▶│  Surface Buffer │────▶│  窗口管理(WMS)  │
│   (Producer)    │     │   (共享内存)     │     │   (Consumer)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                                              │
         │              跨进程传输                        │
         └──────────────────────────────────────────────┘
                           IPC 控制 + 共享内存数据
```

**具体场景**:
- 图形送显与合成
- 媒体播放与录制
- 相机预览数据流

---

## 3. 项目边界

### 3.1 包含范围

| 功能 | 状态 | 说明 |
|------|------|------|
| Buffer 分配与释放 | ✅ | 通过 BufferManager 单例 |
| Buffer 队列管理 | ✅ | BufferQueue 实现 |
| 跨进程 IPC 传输 | ✅ | 基于 Lite IPC |
| 缓存同步 | ✅ | FlushCache 机制 |
| 物理/虚拟内存 | ✅ | 支持多种 Usage 类型 |

### 3.2 不包含范围

| 功能 | 说明 | 归属模块 |
|------|------|----------|
| **窗口管理** | 窗口创建/销毁/布局 | window_window_manager_lite |
| **渲染绘制** | 2D/3D 渲染 | arkui_ui_lite / graphic_2d |
| **显示驱动** | 硬件显示控制 | drivers_peripheral_display |
| **图形工具** | 像素格式、矩形运算 | graphic_graphic_utils_lite |
| **N-API 绑定** | JS 接口封装 | 本模块无 JS 接口 |

### 3.3 上下游依赖

```
                    ┌─────────────────────────────────────┐
                    │         应用层 (JS/Native)           │
                    └─────────────────┬───────────────────┘
                                      │ 使用
                    ┌─────────────────▼───────────────────┐
                    │    window_window_manager_lite       │
                    │         (窗口管理器)                 │
                    └─────────────────┬───────────────────┘
                                      │ 调用
┌──────────────────┐    ┌─────────────▼──────────────┐    ┌──────────────────┐
│  graphic_utils   │◀───│     surface_lite           │───▶│   IPC 模块       │
│   (图形工具)      │    │  (本模块 - 共享内存管理)     │    │ (跨进程通信)      │
└──────────────────┘    └─────────────┬──────────────┘    └──────────────────┘
                                      │ 调用
                    ┌─────────────────▼───────────────────┐
                    │    drivers_peripheral_display       │
                    │       (显示驱动 HAL)                 │
                    └─────────────────────────────────────┘
```

---

## 4. 运行环境

### 4.1 硬件要求

| 资源 | 要求 | 说明 |
|------|------|------|
| RAM | ~50KB | 运行时内存占用 |
| ROM | 110KB | 代码段大小 |
| 共享内存 | 可选 | 用于跨进程传输 |
| 连续物理内存 | 可选 | 用于硬件加速场景 |

### 4.2 软件依赖

**必须依赖**:
- `graphic_utils_lite` - 图形工具库
- `ipc` - 进程间通信
- `drivers_peripheral_display` - 显示驱动

**编译依赖**:
- `bounds_checking_function` - 安全函数库

**链接依赖**:
- `libdisplay_gfx` - 显示图形库
- `libdisplay_gralloc` - 图形内存分配器
- `libdisplay_layer` - 显示图层库

---

## 5. 关键概念

### 5.1 Surface

**定义**: 用于管理共享内存的抽象接口，提供 Buffer 的申请、释放、属性设置等能力。

**两种角色**:
1. **Consumer (消费者)**: 创建 Surface，消费 Buffer (如 WMS)
2. **Producer (生产者)**: 通过 IPC 连接到 Consumer 的 Surface，生产 Buffer (如 UI)

### 5.2 BufferQueue

**定义**: 管理 Buffer 生命周期的核心队列，维护三个列表:
- `freeList_`: 空闲 Buffer 列表
- `dirtyList_`: 待消费 Buffer 列表
- `allBuffers_`: 所有 Buffer 列表

**状态机**:
```
FREE ──RequestBuffer()──▶ REQUEST ──FlushBuffer()──▶ DIRTY
 ▲                                                    │
 │                                                    │
 └────────ReleaseBuffer()────────ACQUIRE◀──AcquireBuffer()
```

### 5.3 Buffer 使用类型

| 类型 | 用途 | Cache |
|------|------|-------|
| `BUFFER_CONSUMER_USAGE_SORTWARE` | 虚拟内存 | 无 |
| `BUFFER_CONSUMER_USAGE_HARDWARE` | 物理内存 | 无 |
| `BUFFER_CONSUMER_USAGE_HARDWARE_CONSUMER_CACHE` | 物理内存 | 消费者端有 Cache |
| `BUFFER_CONSUMER_USAGE_HARDWARE_PRODUCER_CACHE` | 物理内存 | 生产者端有 Cache |

### 5.4 IPC 机制

**跨进程通信方式**:
- **控制信息**: IPC 消息传输 (SvcIdentity, IpcIo)
- **数据**: 共享内存零拷贝

**请求码** (共20个):
```cpp
REQUEST_BUFFER, FLUSH_BUFFER, CANCEL_BUFFER,
SET_QUEUE_SIZE, GET_QUEUE_SIZE, SET_WIDTH_AND_HEIGHT,
GET_WIDTH, GET_HEIGHT, SET_FORMAT, GET_FORMAT,
SET_STRIDE_ALIGNMENT, GET_STRIDE_ALIGNMENT, GET_STRIDE,
SET_SIZE, GET_SIZE, SET_USAGE, GET_USAGE,
SET_USER_DATA, GET_USER_DATA
```

---

## 6. 设计约束

### 6.1 尺寸限制

| 参数 | 最小值 | 最大值 | 默认值 |
|------|--------|--------|--------|
| 宽度 | 1 | 7680 | - |
| 高度 | 1 | 7680 | - |
| Buffer 大小 | 1 | 58982400 (8K×8K) | - |
| 队列大小 | 1 | 10 | 1 |
| Stride 对齐 | 4 | 32 | 4 |

### 6.2 重要须知

> ⚠️ **内存泄漏风险**: 由于使用了共享内存，而共享内存的管理任务在首次创建 Surface 的进程中，如果该进程异常退出且没有回收处理，会发生严重的内存泄漏。

> ⚠️ **内存碎片风险**: Surface 一般用作图形/媒体中大块内存的跨进程传输。不建议用在小内存传输的场景，容易造成内存碎片化影响典型场景的性能。

---

## 7. 相关资源

### 7.1 相关仓库

| 仓库 | 说明 | 关系 |
|------|------|------|
| [window_window_manager_lite](https://gitee.com/openharmony/window_window_manager_lite) | 窗口管理器 | 上游使用者 |
| [graphic_graphic_utils_lite](https://gitee.com/openharmony/graphic_graphic_utils_lite) | 图形工具 | 依赖 |
| [arkui_ui_lite](https://gitee.com/openharmony/arkui_ui_lite) | UI 框架 | 上游使用者 |

### 7.2 文档资源

- [OpenHarmony 图形子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/图形子系统.md)
- [IPC C API 开发指南](https://gitee.com/openharmony/docs/blob/master/en/application-dev/ipc/ipc-capi-development-guideline.md)

---

## 8. 快速开始

### 8.1 编译命令

```bash
# 编译 surface_lite 模块
hb build surface_lite

# 输出产物
out/{product}/libs/libsurface.so
```

### 8.2 使用示例

```cpp
#include "surface.h"
#include "surface_buffer.h"

// Consumer 端 (如 WMS)
void ConsumerExample() {
    // 1. 创建 Surface
    OHOS::Surface* surface = OHOS::Surface::CreateSurface();
    
    // 2. 设置 Buffer 属性
    surface->SetWidthAndHeight(1920, 1080);
    surface->SetFormat(IMAGE_PIXEL_FORMAT_ARGB8888);
    surface->SetQueueSize(3);
    
    // 3. 注册消费者监听
    class MyListener : public OHOS::IBufferConsumerListener {
        void OnBufferAvailable() override {
            // 有新的 Buffer 可消费
            OHOS::SurfaceBuffer* buffer = surface->AcquireBuffer();
            // ... 合成处理
            surface->ReleaseBuffer(buffer);
        }
    };
    MyListener listener;
    surface->RegisterConsumerListener(listener);
}

// Producer 端 (如 UI)
void ProducerExample() {
    // 1. 通过 IPC 获取 Surface (实际从 Consumer 传递)
    OHOS::Surface* surface = /* 通过 IPC 获取 */;
    
    // 2. 请求 Buffer
    OHOS::SurfaceBuffer* buffer = surface->RequestBuffer(1);
    
    // 3. 绘制内容
    void* addr = buffer->GetVirAddr();
    // ... 写入像素数据
    
    // 4. 提交 Buffer
    surface->FlushBuffer(buffer);
}
```

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
