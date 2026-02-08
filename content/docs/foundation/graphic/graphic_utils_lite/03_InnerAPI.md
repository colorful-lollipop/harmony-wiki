# 内部 API（Inner API）

本文档详细说明 `graphic_utils_lite` 的内部 C++ API，包括模块间接口、依赖方向与稳定性标注。

## API 分类

本项目的 API 按使用范围分为两类：

| 类型 | 目录 | 使用范围 | 稳定性 |
|------|------|----------|--------|
| Kits API | `interfaces/kits/gfx_utils/` | 模块外部（框架层） | 稳定（Stable） |
| InnerKits API | `interfaces/innerkits/` | 模块间内部 | 较稳定（Semi-stable） |

## Kits API（外部 C++ 接口）

### 核心类型定义

#### 颜色与像素

**证据来源**：`interfaces/kits/gfx_utils/color.h`

| 类/结构 | 说明 | 主要方法 |
|---------|------|----------|
| `Color` | 颜色表示 | 构造函数、`FromArgb()`、`Blend()` |

#### 几何类型

**证据来源**：`interfaces/kits/gfx_utils/graphic_types.h`

| 类/结构 | 说明 | 关键成员 |
|---------|------|----------|
| `Point` | 2D 坐标点 | `int16_t x, y` |
| `FloatPoint` | 浮点坐标点 | `float x, y` |
| `CrownEvent` | 旋钮事件 | `rotate`, `angularVelocity` |

#### 区域与边界

**证据来源**：`interfaces/kits/gfx_utils/rect.h`

| 类/结构 | 说明 | 主要方法 |
|---------|------|----------|
| `Rect` | 矩形区域 | `Intersects()`, `Union()`, `IsEmpty()` |

#### 变换

**证据来源**：`interfaces/kits/gfx_utils/transform.h`

| 类/结构 | 说明 | 主要方法 |
|---------|------|----------|
| `Transform` | 基础变换 | `Rotate()`, `Scale()`, `Translate()` |

#### 缓冲区

**证据来源**：`interfaces/kits/gfx_utils/graphic_buffer.h`

| 类/结构 | 说明 | 关键成员 |
|---------|------|----------|
| `BufferInfo` | 图形缓冲区 | `phyAddr`, `virAddr`, `stride`, `rect` |

### 内存管理 API

**证据来源**：`interfaces/kits/gfx_utils/mem_api.h`

| 函数 | 说明 | 参数 |
|------|------|------|
| `ImageCacheMalloc` | 图像缓存分配 | `const ImageInfo& info` |
| `ImageCacheFree` | 图像缓存释放 | `ImageInfo& info` |
| `UIMalloc` | UI 内存分配 | `uint32_t size` |
| `UIFree` | UI 内存释放 | `void* buffer` |
| `UIRealloc` | 重新分配内存 | `void* buffer, uint32_t size` |

### 图形引擎 API

#### Diagram 模块

**证据来源**：`interfaces/kits/gfx_utils/diagram/` 目录

| 子模块 | 头文件 | 功能 |
|--------|--------|------|
| depiction | `depict_curve.h`, `depict_dash.h`, `depict_stroke.h` | 曲线与虚线绘制 |
| rasterizer | `rasterizer_scanline_antialias.h`, `rasterizer_cells_antialias.h` | 抗锯齿光栅化 |
| vertexgenerate | `vertex_generate_stroke.h`, `vertex_generate_dash.h` | 顶点生成 |
| vertexprimitive | `geometry_arc.h`, `geometry_curves.h`, `geometry_bezier_arc.h` | 图元几何 |

#### Paint API

**证据来源**：`interfaces/kits/gfx_utils/diagram/common/paint.h`

| 类 | 说明 |
|----|------|
| `Paint` | 绘制样式与属性 |

## InnerKits API（模块间内部 API）

### HAL 接口

#### GFX 引擎

**证据来源**：`interfaces/innerkits/hals/gfx_engines.h`

```cpp
class GfxEngines {
public:
    static GfxEngines* GetInstance();  // 单例获取

    bool InitDriver();                   // 初始化驱动
    void CloseDriver();                  // 关闭驱动

    // 填充区域
    bool GfxFillArea(const LiteSurfaceData& dstSurfaceData,
                     const Rect& fillArea,
                     const ColorType& color,
                     const OpacityType& opa);

    // Blit 操作（像素搬运）
    bool GfxBlit(const LiteSurfaceData& srcSurfaceData,
                 const Rect& srcRect,
                 const LiteSurfaceData& dstSurfaceData,
                 int16_t x,
                 int16_t y);
};
```

#### FrameBuffer 设备

**证据来源**：`interfaces/innerkits/hals/hi_fbdev.h`

| 函数 | 说明 |
|------|------|
| `LiteSurfaceData* GetDevSurfaceData()` | 获取显示表面数据 |
| `LayerRotateType GetLayerRotateType()` | 获取图层旋转类型 |
| `void HiFbdevInit()` | 初始化 FrameBuffer |
| `void HiFbdevClose()` | 关闭 FrameBuffer |
| `void LcdFlush()` | LCD 刷新 |

### 同步原语

#### 互斥锁

**证据来源**：`interfaces/innerkits/graphic_mutex.h`

| 类 | 说明 |
|----|------|
| `GraphicMutex` | 互斥锁实现 |

#### 信号量

**证据来源**：`interfaces/innerkits/graphic_semaphore.h`

| 类 | 说明 |
|----|------|
| `GraphicSemaphore` | 信号量实现 |

#### 锁管理器

**证据来源**：`interfaces/innerkits/graphic_locker.h`

| 类 | 说明 |
|----|------|
| `GraphicLocker` | RAII 锁管理器 |

### 线程与定时器

#### 线程

**证据来源**：`interfaces/innerkits/graphic_thread.h`

| 类 | 说明 |
|----|------|
| `GraphicThread` | 线程封装 |

#### 定时器

**证据来源**：`interfaces/innerkits/graphic_timer.h`

| 类 | 说明 |
|----|------|
| `GraphicTimer` | 定时器管理（单例） |

### 性能与统计

#### 性能监控

**证据来源**：`interfaces/innerkits/graphic_performance.h`

| 类 | 说明 |
|----|------|
| `GraphicPerformance` | 性能统计接口 |

#### NEON 优化

**证据来源**：`interfaces/innerkits/graphic_neon_utils.h`

| 类 | 说明 |
|----|------|
| `GraphicNeonUtils` | ARM NEON SIMD 指令封装 |

### 系统配置

#### 配置结构

**证据来源**：`interfaces/innerkits/graphic_config.h`

| 配置项 | 说明 |
|--------|------|
| `ColorMode` | 颜色模式枚举 |
| `TransformAlgorithm` | 变换算法 |
| `BlurLevel` | 模糊级别 |
| `ScreenShape` | 屏幕形状 |

## 依赖方向图

```
┌─────────────────────────────────────────────────────────────┐
│                        调用关系                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   arkui_ui_lite                                             │
│       │                                                     │
│       ├─→ interfaces/kits/gfx_utils/                        │
│       │       (Color, Rect, Transform, Paint, ...)          │
│       │                                                     │
│   window_manager_lite                                       │
│       │                                                     │
│       ├─→ interfaces/kits/gfx_utils/                        │
│       │       (BufferInfo, TransAffine, ...)               │
│       │                                                     │
│       └─→ interfaces/innerkits/hals/                        │
│               (GfxEngines, HiFbdev)                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## API 稳定性分级

### Level 1：稳定（Stable）

以下接口经过充分测试，向后兼容：

| 接口 | 位置 | 证据 |
|------|------|------|
| `Color` | `color.h` | 广泛使用于 UI 框架 |
| `Rect` | `rect.h` | 基础数据结构 |
| `Point` | `graphic_types.h` | 基础类型定义 |
| `BufferInfo` | `graphic_buffer.h` | 缓冲区标准结构 |
| `GfxEngines::GetInstance` | `gfx_engines.h` | 单例接口 |

### Level 2：较稳定（Semi-stable）

以下接口可能在次要版本中调整：

| 接口 | 位置 | 说明 |
|------|------|------|
| `GraphicTimer` | `graphic_timer.h` | 定时器精度可能调整 |
| `GraphicNeonUtils` | `graphic_neon_utils.h` | SIMD 优化可能演进 |
| `GraphicPerformance` | `graphic_performance.h` | 统计维度可能增加 |

### Level 3：内部使用（Internal）

以下接口仅限框架层使用：

| 接口 | 位置 | 说明 |
|------|------|------|
| `LiteSurfaceData` | `lite_wm_type.h` | 窗口类型，由 window_manager_lite 定义 |
| `GfxFuncs*` | `display_gfx.h` | 驱动函数指针，由 hdi_display 定义 |

## 错误码约定

本项目使用以下错误码约定（通过返回值 bool 表示成功/失败）：

| 返回值 | 含义 |
|--------|------|
| `true` | 操作成功 |
| `false` | 操作失败，具体原因通过日志输出 |

**日志输出示例**：

**证据来源**：`frameworks/hals/gfx_engines.cpp`（日志调用）

```cpp
// 典型错误处理模式
if (!InitDriver()) {
    GRAPHIC_LOG_ERROR("GfxEngines init failed");
    return false;
}
```

---

*最后更新时间：2026-02-06*
