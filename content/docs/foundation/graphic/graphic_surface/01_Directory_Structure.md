# 目录结构与模块职责

## 目的
本文档说明 graphic_surface 项目的目录结构、文件组织和各模块职责。

## 适用范围
面向需要定位代码、理解模块划分的开发者。

## 完整目录树

```
/foundation/graphic/graphic_surface
├── surface/                          # Surface 核心实现
│   ├── include/                      # 内部头文件
│   │   ├── buffer_queue.h            # BufferQueue 定义
│   │   ├── buffer_queue_producer.h   # 生产者端
│   │   ├── buffer_queue_consumer.h   # 消费者端
│   │   ├── buffer_producer_listener.h # 生产者监听器
│   │   ├── surface.h                 # Surface 主类
│   │   ├── surface_buffer.h          # SurfaceBuffer 定义
│   │   ├── producer_surface.h        # 生产者 Surface
│   │   ├── consumer_surface.h        # 消费者 Surface
│   │   ├── surface_delegate.h       # Surface 委托接口
│   │   ├── surface_tunnel_handle.h  # Surface Tunnel Handle
│   │   └── native_window.h          # Native Window 接口
│   │
│   ├── src/                          # 实现源码
│   │   ├── buffer_queue.cpp          # BufferQueue 核心逻辑
│   │   ├── buffer_queue_producer.cpp # 生产者实现
│   │   ├── buffer_queue_consumer.cpp # 消费者实现
│   │   ├── surface.cpp               # Surface 实现
│   │   ├── surface_buffer_impl.cpp   # Buffer 实现
│   │   ├── producer_surface.cpp      # 生产者 Surface 实现
│   │   ├── consumer_surface.cpp      # 消费者 Surface 实现
│   │   ├── surface_delegate.cpp      # 委托实现
│   │   ├── surface_tunnel_handle.cpp # Tunnel Handle 实现
│   │   ├── native_window.cpp         # Native Window 实现
│   │   ├── buffer_client_producer.cpp # IPC 客户端
│   │   ├── buffer_utils.cpp          # Buffer 工具函数
│   │   ├── surface_utils.cpp         # Surface 工具函数
│   │   ├── metadata_helper.cpp       # 元数据辅助
│   │   ├── delegator_adapter.cpp     # 委托适配器
│   │   ├── producer_surface_delegator.cpp  # 生产者委托
│   │   ├── consumer_surface_delegator.cpp  # 消费者委托
│   │   └── buffer_extra_data_impl.cpp     # Buffer 扩展数据
│   │
│   ├── test/                         # 测试代码（忽略）
│   │   ├── unittest/                 # 单元测试
│   │   ├── fuzztest/                 # Fuzz 测试
│   │   └── systemtest/               # 系统测试
│   │
│   └── BUILD.gn                      # Surface 模块构建脚本
│
├── sync_fence/                       # 同步栅栏模块
│   ├── include/
│   │   ├── sync_fence.h              # SyncFence 接口
│   │   ├── native_fence.h            # Native Fence 接口
│   │   ├── frame_sched.h             # 帧调度器
│   │   ├── acquire_fence_manager.h   # AcquireFence 管理器
│   │   └── sync_fence_tracker.h     # SyncFence 追踪器
│   │
│   ├── src/
│   │   ├── sync_fence.cpp            # SyncFence 实现
│   │   ├── native_fence.cpp          # Native Fence 实现
│   │   ├── frame_sched.cpp           # 帧调度器实现
│   │   ├── acquire_fence_manager.cpp # AcquireFence 管理实现
│   │   └── sync_fence_tracker.cpp   # SyncFence 追踪实现
│   │
│   ├── test/                         # 测试代码（忽略）
│   └── BUILD.gn                      # Sync Fence 构建脚本
│
├── buffer_handle/                    # Buffer 句柄管理
│   ├── src/
│   │   └── buffer_handle.cpp         # BufferHandle 序列化/反序列化
│   │
│   ├── test/                         # 测试代码（忽略）
│   └── BUILD.gn                      # Buffer Handle 构建脚本
│
├── utils/                            # 工具模块
│   ├── frame_report/                 # 帧上报工具
│   │   ├── export/
│   │   │   └── frame_report.h        # 帧上报接口
│   │   ├── src/
│   │   │   └── frame_report.cpp      # 帧上报实现
│   │   └── test/                     # 测试代码（忽略）
│   │
│   ├── hebc_white_list/              # HEBC 白名单
│   │   ├── export/
│   │   │   └── hebc_white_list.h     # HEBC 白名单接口
│   │   ├── src/
│   │   └── test/                     # 测试代码（忽略）
│   │
│   ├── rs_frame_report_ext/          # RS 帧上报扩展
│   │   ├── src/
│   │   └── test/                     # 测试代码（忽略）
│   │
│   └── trace/                        # 追踪工具
│       ├── surface_trace.h           # Surface 追踪宏
│       └── surface_trace.cpp         # Surface 追踪实现
│
├── sandbox/                          # 沙箱相关
│   ├── sandbox_utils.h               # 沙箱工具接口
│   ├── sandbox_utils.cpp             # 沙箱工具实现
│   └── BUILD.gn
│
├── interfaces/                       # 接口定义
│   └── inner_api/                    # 模块间接口（内部 API）
│       ├── surface/                  # Surface 相关接口
│       │   ├── surface.h             # Surface 主接口
│       │   ├── surface_buffer.h      # Buffer 接口
│       │   ├── surface_type.h        # 类型定义
│       │   ├── surface_utils.h       # 工具函数
│       │   ├── window.h              # Native Window C 接口
│       │   ├── native_buffer.h      # Native Buffer C 接口
│       │   ├── native_buffer_inner.h # 内部 Native Buffer
│       │   ├── egl_surface.h         # EGL Surface 接口
│       │   ├── egl_data.h           # EGL 数据
│       │   ├── ibuffer_producer.h    # 生产者 IPC 接口
│       │   ├── ibuffer_producer_listener.h # 生产者监听器接口
│       │   ├── iconsumer_surface.h   # 消费者 Surface 接口
│       │   ├── ibuffer_consumer_listener.h   # 消费者监听器接口
│       │   ├── surface_delegate.h    # Surface 委托接口
│       │   ├── external_window.h     # 外部窗口接口（对外 C API）
│       │   ├── surface_tunnel_handle.h    # Tunnel Handle 接口
│       │   ├── common_types.h       # 通用类型
│       │   ├── buffer_common.h       # Buffer 通用定义
│       │   └── buffer_extra_data.h  # Buffer 扩展数据
│       │
│       ├── sync_fence/               # 同步栅栏接口
│       │   ├── sync_fence.h          # SyncFence 接口
│       │   └── native_fence.h        # Native Fence 接口
│       │
│       ├── buffer_handle/             # Buffer 句柄接口
│       │   ├── buffer_handle.h       # BufferHandle 结构
│       │   ├── buffer_handle_utils.h # BufferHandle 工具
│       │   ├── buffer_handle_parcel.h# IPC 序列化接口
│       │   └── buffer_handle_parcel.cpp # 实现
│       │
│       ├── common/                   # 通用定义
│       │   ├── graphic_common.h      # 图形通用类型和错误码
│       │   ├── graphic_common_c.h    # C 语言通用类型
│       │   └── metadata_convertor.h # 元数据转换器
│       │
│       └── utils/                    # 工具接口
│           ├── surface_aps_sdr_utils.h    # APS SDR 工具
│           ├── isurface_aps_plugin.h      # APS 插件接口
│           ├── native_fence.h             # Native Fence C 接口
│           ├── sync_fence.h               # Sync Fence C 接口
│           ├── buffer_handle.h             # BufferHandle C 接口
│           ├── buffer_handle_utils.h       # BufferHandle 工具 C 接口
│           └── buffer_handle_parcel.h     # BufferHandle Parcel C 接口
│
├── wiki/                             # Wiki 文档（本文档集）
│   ├── _work/                        # 工作目录
│   │   ├── NOTES.md                  # 工作笔记
│   │   └── PLAN.md                   # 工作计划
│   ├── appendix/                     # 附录
│   ├── 00_Overview.md                # 概览
│   ├── 01_Directory_Structure.md     # 目录结构（本文档）
│   ├── 02_Architecture.md            # 架构说明
│   ├── 03_External_API.md            # 对外 API
│   ├── 04_Internal_API.md            # 内部 API
│   ├── 05_GN_Targets.md              # GN Targets
│   ├── 06_Build_Artifacts.md         # 编译产物
│   ├── 07_Security_Review.md         # 安全评审
│   ├── 08_Troubleshooting.md         # 故障排查
│   ├── README.md                     # Wiki 首页
│   └── SUMMARY.md                    # 全局导航
│
├── figures/                          # 文档图片
│   ├── surface在系统架构中的位置（绿色部分为surface-buffer）.png
│   ├── Surface轮转流程.png
│   ├── position-of-a-surface-in-the-system-architecture.png
│   ├── surface-rotation-process.png
│   └── icon-notice.gif
│
├── bundle.json                       # 组件元数据
├── graphic_surface_config.gni        # 全局配置
├── LICENSE                           # Apache 2.0 许可证
├── README.md                         # 项目说明（中文）
├── README.en.md                      # 项目说明（英文）
└── OAT.xml                           # 开放原子测试（OAT）配置
```

## 模块职责

### surface/ - Surface 核心模块
**职责**：
- Surface 生命周期管理（创建、连接、断开、销毁）
- Buffer 分配、引用计数、释放
- BufferQueue 队列管理（Free/Dirty 队列）
- 生产者-消费者协调
- IPC 接口实现（IRemoteStub/IRemoteProxy）

**核心组件**：
| 组件 | 文件 | 职责 |
|------|------|------|
| BufferQueue | `buffer_queue.{h,cpp}` | Buffer 队列核心逻辑 |
| BufferQueueProducer | `buffer_queue_producer.{h,cpp}` | 生产者端实现，IRemoteStub |
| BufferQueueConsumer | `buffer_queue_consumer.{h,cpp}` | 消费者端实现 |
| Surface | `surface.{h,cpp}` | Surface 主类，工厂方法 |
| SurfaceBuffer | `surface_buffer_impl.cpp` | Buffer 实现，引用计数 |
| ProducerSurface | `producer_surface.{h,cpp}` | 生产者 Surface |
| ConsumerSurface | `consumer_surface.{h,cpp}` | 消费者 Surface |
| BufferClientProducer | `buffer_client_producer.{h,cpp}` | IPC 客户端，IRemoteProxy |
| NativeWindow | `native_window.{h,cpp}` | Native Window 接口适配 |

**对外接口**：
- `interfaces/inner_api/surface/surface.h` - Surface C++ 接口
- `interfaces/inner_api/surface/surface_buffer.h` - Buffer 接口
- `interfaces/inner_api/surface/ibuffer_producer.h` - 生产者 IPC 接口
- `interfaces/inner_api/surface/iconsumer_surface.h` - 消费者接口

**对外 C API**：
- `interfaces/inner_api/surface/external_window.h` - 外部窗口接口
- `interfaces/inner_api/surface/native_buffer.h` - Native Buffer C 接口
- `interfaces/inner_api/surface/window.h` - Native Window C 接口

### sync_fence/ - 同步栅栏模块
**职责**：
- GPU/CPU 同步机制（基于 Linux dma_fence）
- 跨进程 Fence 管理
- Fence 合并、等待、信号
- AcquireFence/ReleaseFence 管理
- Fence 追踪与调试

**核心组件**：
| 组件 | 文件 | 职责 |
|------|------|------|
| SyncFence | `sync_fence.{h,cpp}` | Fence 主类 |
| NativeFence | `native_fence.{h,cpp}` | Native Fence 包装 |
| FrameScheduler | `frame_sched.{h,cpp}` | 帧调度器 |
| AcquireFenceManager | `acquire_fence_manager.{h,cpp}` | AcquireFence 管理 |
| SyncFenceTracker | `sync_fence_tracker.{h,cpp}` | Fence 追踪 |

**对外接口**：
- `interfaces/inner_api/sync_fence/sync_fence.h` - SyncFence C++ 接口
- `interfaces/inner_api/utils/sync_fence.h` - SyncFence C 接口
- `interfaces/inner_api/utils/native_fence.h` - Native Fence C 接口

### buffer_handle/ - Buffer 句柄模块
**职责**：
- BufferHandle 结构定义
- IPC 序列化/反序列化（MessageParcel）
- 文件描述符传递（fd passing）
- BufferHandle 工具函数

**核心组件**：
| 组件 | 文件 | 职责 |
|------|------|------|
| BufferHandle 序列化 | `buffer_handle.cpp` | WriteBufferHandle/ReadBufferHandle |

**对外接口**：
- `interfaces/inner_api/buffer_handle/buffer_handle.h` - BufferHandle 结构
- `interfaces/inner_api/buffer_handle/buffer_handle_utils.h` - 工具函数
- `interfaces/inner_api/buffer_handle/buffer_handle_parcel.h` - IPC 序列化接口
- `interfaces/inner_api/utils/buffer_handle.h` - C 接口

### utils/ - 工具模块

#### utils/frame_report/ - 帧上报
**职责**：
- 帧性能数据采集
- 帧时间戳上报
- 帧率统计

**对外接口**：
- `utils/frame_report/export/frame_report.h` - 帧上报接口

#### utils/hebc_white_list/ - HEBC 白名单
**职责**：
- HEBC（High Efficiency Buffer Cache）白名单管理
- Buffer 缓存策略配置

**对外接口**：
- `utils/hebc_white_list/export/hebc_white_list.h` - HEBC 白名单接口

#### utils/rs_frame_report_ext/ - RS 帧上报扩展
**职责**：
- Render Service 帧上报扩展
- 与 RS 模块集成

#### utils/trace/ - 追踪工具
**职责**：
- Surface 操作追踪
- 性能分析点标记

**接口**：
- `utils/trace/surface_trace.h` - 追踪宏定义

### sandbox/ - 沙箱模块
**职责**：
- 沙箱环境适配
- 安全文件描述符读取
- 权限检查辅助

**核心组件**：
| 组件 | 文件 | 职责 |
|------|------|------|
| SandboxUtils | `sandbox_utils.{h,cpp}` | 沙箱工具函数 |

### interfaces/ - 接口定义
**职责**：
- 定义模块间 API 契约
- 提供统一的头文件目录
- 区分内部 API 和外部 API

#### interfaces/inner_api/surface/
**Surface 相关接口**：
- **C++ 接口**：`surface.h`, `surface_buffer.h`, `ibuffer_producer.h`
- **C 接口**：`external_window.h`, `native_buffer.h`, `window.h`
- **接口定义**：`iconsumer_surface.h`, `surface_delegate.h`

#### interfaces/inner_api/sync_fence/
**同步栅栏接口**：
- **C++ 接口**：`sync_fence.h`
- **C 接口**：`native_fence.h` (在 utils/ 中也有)

#### interfaces/inner_api/buffer_handle/
**Buffer 句柄接口**：
- **结构定义**：`buffer_handle.h`
- **工具函数**：`buffer_handle_utils.h`
- **IPC 序列化**：`buffer_handle_parcel.h`

#### interfaces/inner_api/common/
**通用定义**：
- **类型定义**：`graphic_common.h`, `graphic_common_c.h`
- **元数据**：`metadata_convertor.h`

#### interfaces/inner_api/utils/
**工具接口**：
- **C 接口集合**：native_fence, sync_fence, buffer_handle 等
- **插件接口**：`isurface_aps_plugin.h`
- **工具**：`surface_aps_sdr_utils.h`

## 模块依赖关系

### 依赖方向
```
┌─────────────────────────────────────────┐
│  应用层 / 上层框架                       │
│  (WindowManager, UI Framework, etc.)    │
└───────────────┬─────────────────────────┘
                │ uses
┌───────────────▼─────────────────────────┐
│  surface (核心模块)                       │
│  ┌──────────────────────────────────┐   │
│  │ BufferQueue (队列管理)          │   │
│  │ Surface (生命周期)               │   │
│  │ SurfaceBuffer (Buffer 管理)     │   │
│  │ ProducerSurface/ConsumerSurface │   │
│  └──────────────────────────────────┘   │
└───────┬──────────────────────┬───────────┘
        │ uses                │ uses
┌───────▼────────┐   ┌────────▼────────────┐
│ sync_fence     │   │ buffer_handle       │
│ (同步机制)     │   │ (IPC 序列化)        │
└────────────────┘   └────────────────────┘
        │                    │ uses
        └────────┬───────────┘
                 ▼
        ┌─────────────────┐
        │ utils/*         │
        │ (工具模块)      │
        └─────────────────┘
                 │ uses
                 ▼
        ┌─────────────────┐
        │ sandbox/        │
        │ (沙箱适配)      │
        └─────────────────┘
```

### 关键依赖

| 模块 | 依赖模块 | 依赖原因 |
|------|---------|---------|
| surface | sync_fence | Buffer 完成信号、GPU/CPU 同步 |
| surface | buffer_handle | BufferHandle IPC 传输 |
| surface | utils/frame_report | 帧性能统计 |
| buffer_handle | sandbox | 安全的 fd 读取 |
| 所有模块 | hilog/hitrace | 日志和追踪 |
| 所有模块 | ipc | 跨进程通信 |

## API 稳定性分类

### 稳定 API（对外）
- **证据**：`interfaces/inner_api/surface/external_window.h` - 明确标记为外部接口
- **证据**：`interfaces/inner_api/surface/native_buffer.h` - C 接口，稳定
- **证据**：`interfaces/inner_api/surface/window.h` - Native Window C 接口

### 半稳定 API（模块间）
- **证据**：`interfaces/inner_api/surface/surface.h` - C++ Surface 接口
- **证据**：`interfaces/inner_api/surface/iconsumer_surface.h` - 消费者接口
- **证据**：`interfaces/inner_api/sync_fence/sync_fence.h` - SyncFence 接口

### 内部 API（不稳定）
- **证据**：`surface/include/` 目录下头文件 - 实现细节
- **证据**：`surface/src/` 目录下实现 - 可随时变更

### IPC 接口（稳定）
- **证据**：`interfaces/inner_api/surface/ibuffer_producer.h` - 跨进程接口
- **证据**：`interfaces/inner_api/surface/ibuffer_producer_listener.h` - 回调接口

## 测试模块（忽略）

测试模块分布在各子目录的 `test/` 下，包括：
- `unittest/` - 单元测试
- `fuzztest/` - Fuzz 测试
- `systemtest/` - 系统测试

**注意**：本文档不引用测试代码作为业务证据。

## 相关跳转
- [概览](00_Overview.md) - 项目定位与核心能力
- [架构说明](02_Architecture.md) - 组件设计与数据流
- [对外 API](03_External_API.md) - 稳定 API 详细说明
- [内部 API](04_Internal_API.md) - 模块间接口说明
