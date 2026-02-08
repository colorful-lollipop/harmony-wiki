# 配置宏与 Feature Flags

## 目的
本文档说明 graphic_surface 的编译配置宏和 Feature Flags，包括用途、默认值和影响。

## 适用范围
面向需要：
- 控制编译特性的开发者
- 理解不同构建配置的架构师
- 集成 graphic_surface 的产品集成者
- 调试编译问题的工程师

## 全局配置

### graphic_surface_config.gni

**配置文件**：`/Volumes/lexar/code/d/work/oh/foundation/graphic/graphic_surface/graphic_surface_config.gni`

#### 根路径定义
```gni
graphic_surface_root = "//foundation/graphic/graphic_surface"
```

**用途**：作为其他 BUILD.gn 文件中引用的相对路径根。

#### 委托模式配置

| 宏 | 默认值 | 用途 | 证据 |
|------|---------|------|------|
| `graphic_2d_ext_delegator` | `false` | 启用 2D 扩展委托模式 | `graphic_surface_config.gni:17` |
| `graphic_2d_ext_delegator_gni` | `""` | 委托模式 GNI 配置文件路径 | `graphic_surface_config.gni:18` |

**使用场景**：
- 跨进程委托（生产者和消费者在不同进程）
- 插件扩展（APS 插件）
- 自定义 Buffer 管理策略

#### TV PQ 元数据配置

| 宏 | 默认值 | 用途 | 证据 |
|------|---------|------|------|
| `graphic_surface_feature_tv_metadata_enable` | `false` | 启用 TV PQ（Picture Quality）元数据 | `graphic_surface_config.gni:19`, `bundle.json:19` |

**影响**：
- 当启用时：编译 `RS_ENABLE_TV_PQ_METADATA` 定义
- 支持 HDR、色域转换等电视相关功能

#### 跨平台定义

```gni
rosen_cross_platform =
    current_os == "mac" || current_os == "mingw" ||
    current_os == "linux" || current_os == "android" ||
    current_os == "ios"
```

**影响**：
```gni
if (rosen_cross_platform) {
  rs_common_define += [ "ROSEN_TRACE_DISABLE" ]
}
```

**用途**：在非 OpenHarmony 平台（如 macOS、Linux）上禁用追踪功能。

## Feature Flags

### 1. ROSEN_TRACE_DISABLE

| 属性 | 值 |
|------|-----|
| Flag 名称 | `ROSEN_TRACE_DISABLE` |
| 默认值 | 条件编译（`rosen_cross_platform` 为真） |
| 定义位置 | `graphic_surface_config.gni:28` |
| 影响的模块 | Surface, SyncFence, FrameReport |

**用途**：
- 禁用 Surface 操作追踪
- 减少性能开销
- 跨平台兼容性

**代码示例**：
```cpp
// utils/trace/surface_trace.h
#ifndef ROSEN_TRACE_DISABLE
  #define SURFACE_TRACE_FUNC() \
      ScopedTrace trace(__FUNCTION__); \
      Trace::GetInstance()->OnTraceBegin(__FUNCTION__)
  #define SURFACE_TRACE_TIME(name, time) \
      Trace::GetInstance()->OnTraceTime(name, time)
#else
  #define SURFACE_TRACE_FUNC() ((void)0)
  #define SURFACE_TRACE_TIME(name, time) ((void)0)
#endif
```

### 2. RS_ENABLE_TV_PQ_METADATA

| 属性 | 值 |
|------|-----|
| Flag 名称 | `RS_ENABLE_TV_PQ_METADATA` |
| 默认值 | 未定义（禁用） |
| 条件 | `graphic_surface_feature_tv_metadata_enable` = true |
| 定义位置 | `surface/BUILD.gn` |
| 影响的模块 | Surface |

**用途**：
- 启用 TV PQ 元数据功能
- 支持 HDR、Wide Color Gamut 等电视特性
- 支持自定义 PQ 曲线

**代码示例**：
```cpp
// surface/src/buffer_queue.cpp
#ifdef RS_ENABLE_TV_PQ_METADATA
GSError BufferQueue::SetMetaData(const std::vector<GraphicHDRMetaData>& metaData)
{
    // TV PQ 元数据处理
    for (const auto& meta : metaData) {
        switch (meta.key) {
            case GraphicHDRMetaKey::HDR_METADATA_KEY_SMPTE2086:
                // 处理 HDR 元数据
                break;
            case GraphicHDRMetaKey::HDR_METADATA_KEY_CTA861_3:
                // 处理色域元数据
                break;
        }
    }
    return GSERROR_OK;
}
#else
GSError BufferQueue::SetMetaData(const std::vector<GraphicHDRMetaData>& metaData)
{
    return GSERROR_NOT_SUPPORT;  // 功能未启用
}
#endif
```

**相关 API**：
- `IBufferProducer::SetMetaData()` - 设置 HDR 元数据
- `IBufferProducer::SetSurfaceSourceType()` - 设置源类型（如 TV）

### 3. AI_SCHED_ENABLE

| 属性 | 值 |
|------|-----|
| Flag 名称 | `AI_SCHED_ENABLE` |
| 默认值 | 未定义（禁用） |
| 条件 | `defined(global_parts_info.hdf_drivers_interface_hwsched)` |
| 定义位置 | `utils/frame_report/BUILD.gn` |
| 影响的模块 | FrameReport |

**用途**：
- 启用 AI 帧调度
- 与 HDF HwSched 模块集成
- 智能帧率控制

**代码示例**：
```cpp
// utils/frame_report/src/frame_report.cpp
#ifdef AI_SCHED_ENABLE
FrameReport::ReportFrame(int32_t frameNum, int64_t timestamp, int64_t cost)
{
    // AI 调度逻辑
    aiScheduler->UpdateFrameCost(frameNum, cost);

    // 上报到 AI 调度器
    return GSERROR_OK;
}
#else
FrameReport::ReportFrame(int32_t frameNum, int64_t timestamp, int64_t cost)
{
    // 标准上报逻辑
    frameStats_[frameNum % STATS_SIZE] = {timestamp, cost};
    return GSERROR_OK;
}
#endif
```

**相关组件**：
- `HDF HwSched` - 硬件调度框架
- `drivers_interface_hwsched` - 调度驱动接口

### 4. FENCE_SCHED_ENABLE

| 属性 | 值 |
|------|-----|
| Flag 名称 | `FENCE_SCHED_ENABLE` |
| 默认值 | 条件编译 |
| 条件 | `!is_emulator && !build_ohos_sdk && current_os == "ohos"` |
| 定义位置 | `sync_fence/BUILD.gn` |
| 影响的模块 | SyncFence |

**用途**：
- 启用 Fence 调度
- 优化 GPU/CPU 同步
- 减少不必要的等待

**代码示例**：
```cpp
// sync_fence/src/frame_sched.cpp
#ifdef FENCE_SCHED_ENABLE
FrameScheduler::Update()
{
    // Fence 调度逻辑
    for (auto& fence : activeFences_) {
        if (fence->IsReady()) {
            WakeUpWaitingThreads(fence);
        }
    }
}
#else
FrameScheduler::Update()
{
    // 标准 Fence 等待逻辑
    // 无调度优化
}
#endif
```

## 日志与调试宏

### SURFACE_LOG_TAG

| 宏 | 值 |
|------|-----|
| 名称 | `SURFACE_LOG_TAG` |
| 定义 | `"Graphic"` |
| 用途 | Surface 模块的日志标签 |

**代码示例**：
```cpp
// surface/src/surface.cpp
#define SURFACE_LOG_TAG "Graphic"
#define LOG_TAG SURFACE_LOG_TAG

#include <hilog/log.h>

BLOGI("Message");  // 等同于 OH_LOGI(LOG_TAG, "Message")
BLOGE("Error: %s", error);
BLOGW("Warning: %s", warning);
```

### BUFFER_QUEUE_LOG_TAG

| 宏 | 值 |
|------|-----|
| 名称 | `BUFFER_QUEUE_LOG_TAG` |
| 定义 | `"BufferQueue"` |
| 用途 | BufferQueue 模块的日志标签 |

### SYNC_FENCE_LOG_TAG

| 宏 | 值 |
|------|-----|
| 名称 | `SYNC_FENCE_LOG_TAG` |
| 定义 | `"SyncFence"` |
| 用途 | SyncFence 模块的日志标签 |

### SURFACE_ENABLE_FRAME_STATS

| 宏 | 值 |
|------|-----|
| 名称 | `SURFACE_ENABLE_FRAME_STATS` |
| 默认值 | 已定义（启用） |
| 定义位置 | `surface/BUILD.gn` |
| 用途 | 启用帧统计功能 |

**代码示例**：
```cpp
#ifdef SURFACE_ENABLE_FRAME_STATS
class SurfaceStats {
public:
    void RecordRequest(int64_t timestamp) {
        requestCount_++;
        lastRequestTime_ = timestamp;
    }

    void RecordFlush(int64_t timestamp) {
        flushCount_++;
        lastFlushTime_ = timestamp;
        CalculateLatency(lastRequestTime_, lastFlushTime_);
    }
};
#endif
```

## 魔术数字与常量

### Surface 魔术数字

| 常量 | 值 | 用途 | 证据 |
|--------|-----|------|------|
| `MAGIC_INIT` | `0x16273849` | Surface 初始化标识 | `surface/src/buffer_queue_producer.cpp:182` |
| `SURFACE_DEFAULT_QUEUE_SIZE` | `3` | 默认 Buffer 队列大小 | `surface/src/buffer_queue.cpp:xxx` |
| `SURFACE_MAX_QUEUE_SIZE` | `64` | 最大 Buffer 队列大小 | `surface/src/buffer_queue.cpp:xxx` |

**Magic Number 验证**：
```cpp
// surface/src/buffer_queue_producer.cpp:174-187
bool CheckIsAlive()
{
    if (magicNum_ != MAGIC_INIT) {
        static const bool isBeta =
            system::GetParameter("const.logsystem.versiontype", "") == "beta";
        if (isBeta) {
            raise(42);  // 触发崩溃报告
        }
        return false;
    }
    return true;
}
```

### Buffer 状态常量

| 常量 | 值 | 用途 | 证据 |
|--------|-----|------|------|
| `BUFFER_STATE_RELEASED` | `0` | Buffer 在 freeList_ | `surface/include/buffer_queue.h` |
| `BUFFER_STATE_REQUESTED` | `1` | Buffer 被生产者请求 | `surface/include/buffer_queue.h` |
| `BUFFER_STATE_FLUSHED` | `2` | Buffer 被生产者提交 | `surface/include/buffer_queue.h` |
| `BUFFER_STATE_ACQUIRED` | `3` | Buffer 被消费者获取 | `surface/include/buffer_queue.h` |
| `BUFFER_STATE_ATTACHED` | `4` | Buffer 被附加到队列 | `surface/include/buffer_queue.h` |

## 错误码定义

### GSERROR 范围

| 错误范围 | 起始值 | 类别 |
|---------|--------|------|
| 0 | `GSERROR_OK` | 成功 |
| 30001000 | `GSERROR_NO_MEMORY` | 内存操作 |
| 30001001 | `GSERROR_INVALID_OPERATIONS` | 无效操作 |
| 40001000 | `GSERROR_INVALID_ARGUMENTS` | 无效参数 |
| 40001001 | `GSERROR_INVALID_BUFFER_ID` | 无效 Buffer ID |
| 40301000 | `GSERROR_NO_PERMISSION` | 权限错误 |
| 40401000 | `GSERROR_NO_CONSUMER` | 无消费者 |
| 40401001 | `GSERROR_NO_PRODUCER` | 无生产者 |
| 40601000 | `GSERROR_NO_BUFFER` | 无可用 Buffer |
| 40601001 | `GSERROR_INVALID_ENTRY` | 无效条目 |
| 41201000 | `GSERROR_API_FAILED` | API 失败 |
| 41202000 | `GSERROR_NO_CONSUMER` | 消费者未连接 |
| 41203000 | `GSERROR_NO_ENTRY` | 无条目 |
| 41204000 | `GSERROR_INVALID_OPERATIONS` | 无效操作 |
| 41205000 | `GSERROR_IO_ERROR` | I/O 错误 |
| 41206000 | `GSERROR_CONSUMER_IS_CONNECTED` | 消费者已连接 |
| 41207000 | `GSERROR_BUFFER_STATE_INVALID` | Buffer 状态无效 |
| 41208000 | `GSERROR_ENTRY_NOT_EXIST` | 条目不存在 |
| 41209000 | `GSERROR_BUFFER_QUEUE_FULL` | Buffer 队列已满 |
| 41210000 | `GSERROR_CONSUMER_DISCONNECTED` | 消费者断开 |
| 41211000 | `GSERROR_CONSUMER_DISCONNECTED` | 消费者断开（重复定义） |
| 50001000 | `GSERROR_API_FAILED` | API 失败（通用） |
| 50101000 | `GSERROR_NOT_SUPPORT` | 功能不支持 |
| 50401000 | `GSERROR_BINDER` | Binder 错误 |
| 60001000 | `GSERROR_EGL_ERROR` | EGL 错误 |

### IPC 方法码

| 方法码 | 值 | 方法名称 |
|--------|-----|---------|
| `BUFFER_PRODUCER_REQUEST_BUFFER` | `0` | RequestBuffer |
| `BUFFER_PRODUCER_CANCEL_BUFFER` | `1` | CancelBuffer |
| `BUFFER_PRODUCER_FLUSH_BUFFER` | `2` | FlushBuffer |
| `BUFFER_PRODUCER_GET_QUEUE_SIZE` | `3` | GetQueueSize |
| `BUFFER_PRODUCER_SET_QUEUE_SIZE` | `4` | SetQueueSize |
| `BUFFER_PRODUCER_GET_NAME` | `5` | GetName |
| `BUFFER_PRODUCER_SET_NAME` | `6` | SetName |
| `BUFFER_PRODUCER_SET_DEFAULT_WIDTH_AND_HEIGHT` | `7` | SetDefaultWidthAndHeight |
| `BUFFER_PRODUCER_SET_DEFAULT_USAGE` | `8` | SetDefaultUsage |
| `BUFFER_PRODUCER_CONNECT` | `35` | Connect |
| `BUFFER_PRODUCER_DISCONNECT` | `36` | Disconnect |
| `BUFFER_PRODUCER_ATTACH_BUFFER` | `43` | AttachBuffer |
| `BUFFER_PRODUCER_DETACH_BUFFER` | `44` | DetachBuffer |
| `BUFFER_PRODUCER_SET_TRANSFORM` | `9` | SetTransform |
| `BUFFER_PRODUCER_SET_SCALING_MODE` | `10` | SetScalingMode |
| `BUFFER_PRODUCER_SET_METADATA` | `24` | SetMetaData |
| `BUFFER_PRODUCER_REQUEST_BUFFERS` | `38` | RequestBuffers（批量） |
| `BUFFER_PRODUCER_FLUSH_BUFFERS` | `39` | FlushBuffers（批量） |

## 编译控制

### Release vs Debug 构建

| 构建类型 | 定义 | 影响 |
|---------|------|------|
| Debug | `NDEBUG` 未定义 | 启用断言、调试日志、符号表 |
| Release | `NDEBUG` 已定义 | 禁用断言、优化代码、移除符号 |

### Sanitizers

graphic_surface 默认启用以下 sanitizers：

| Sanitizer | 定义 | 用途 | 证据 |
|-----------|------|------|------|
| `boundary_sanitize` | `_FORTIFY_SOURCE` | 数组边界检查 |
| `integer_overflow` | `_INTEGER_OVERFLOW` | 整数溢出检查 |
| `ubsan` | `_UBSAN` | 未定义行为检查 |

**GN 配置**：
```gni
# surface/BUILD.gn, buffer_handle/BUILD.gn, hebc_white_list/BUILD.gn
boundary_sanitize = true
integer_overflow = true
ubsan = true
```

## 编译命令示例

### 标准 Debug 构建
```bash
# 标准 Debug 构建（含符号表）
hb build graphic_surface

# 带优化信息的 Debug 构建
hb build graphic_surface --gn-args='is_debug=true'
```

### 标准 Release 构建
```bash
# 标准 Release 构建（Strip 符号）
hb build graphic_surface --build-type release

# 启用所有 Feature Flags
hb build graphic_surface \
  --build-option graphic_surface_feature_tv_metadata_enable=true \
  --build-option graphic_2d_ext_delegator=true
```

### 带 Sanitizers 构建
```bash
# 启用 AddressSanitizer
hb build graphic_surface --gn-args='use_asan=true'

# 启用 ThreadSanitizer
hb build graphic_surface --gn-args='use_tsan=true'

# 启用 UndefinedBehaviorSanitizer
hb build graphic_surface --gn-args='use_ubsan=true'
```

## 配置文件位置

### HEBC 白名单配置

**文件路径**：`/etc/graphics_game/config/graphics_game.json`

**用途**：定义可使用 HEBC（High Efficiency Buffer Cache）的应用白名单。

**格式示例**：
```json
{
  "hebcList": [
    "com.example.game1",
    "com.example.game2",
    "com.example.game3"
  ]
}
```

**相关代码**：
- `utils/hebc_white_list/hebc_white_list.cpp:Init()` - 读取配置
- `utils/hebc_white_list/hebc_white_list.cpp:Check()` - 检查应用是否在白名单

### 系统属性配置

graphic_surface 读取以下系统属性：

| 属性名 | 用途 | 默认值 |
|---------|------|--------|
| `const.logsystem.versiontype` | 系统版本类型（beta/release） | `"release"` |
| `const.debuggable` | 是否可调试 | `0` |

**代码示例**：
```cpp
// surface/src/buffer_queue_producer.cpp
static const bool isBeta =
    system::GetParameter("const.logsystem.versiontype", "") == "beta";

if (isBeta && magicNum_ != MAGIC_INIT) {
    raise(42);  // Beta 版本严格检查
}
```

## 常见配置问题

### 问题 1：Feature Flag 未生效

**症状**：启用 Feature Flag 后，相关代码未编译

**检查清单**：
1. 确认 Flag 在 BUILD.gn 中正确设置
2. 确认依赖的组件存在
3. 清理构建缓存：`hb clean`
4. 重新编译：`hb build graphic_surface`

### 问题 2：宏定义冲突

**症状**：编译错误或运行时行为异常

**解决方法**：
```bash
# 检查宏定义
hb build graphic_surface --gn-args='defines=["MACRO1","MACRO2"]'

# 检查预处理器输出
hb build graphic_surface -v | grep "MACRO"
```

### 问题 3：交叉编译配置错误

**症状**：为目标平台编译时失败

**检查清单**：
1. 确认 `current_os` 设置正确
2. 确认 `current_cpu` 设置正确（arm64, x86_64）
3. 确认工具链正确配置

## 相关跳转
- [GN Targets 与编译产物](05_GN_Targets.md) - GN 构建系统
- [对外 API](03_External_API.md) - API 使用示例
- [常见问题](08_Troubleshooting.md) - 配置问题调试
- [目录结构与模块职责](01_Directory_Structure.md) - BUILD.gn 文件位置
