# 架构说明

## 整体架构

graphic_2d 采用 **客户端-服务器 (Client-Server)** 渲染架构，基于 **Rosen 渲染框架**：

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    APPLICATION                                          │
│                         (ArkUI / Native Applications)                                   │
└─────────────────────────────────┬───────────────────────────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                               │
│                    (rosen/modules/render_service_client)                                │
│                                                                                         │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                   │
│   │   RSNode    │  │  RSAnimation│  │  RSModifier │  │ RSUIDirector│                   │
│   │  Hierarchy  │  │   System    │  │    NG       │  │   (Scene)   │                   │
│   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                   │
│          │                │                │                │                          │
│          └────────────────┴────────────────┴────────────────┘                          │
│                              RSTransactionHandler                                       │
│                                    │                                                    │
└────────────────────────────────────┼────────────────────────────────────────────────────┘
                                     │
                              IPC / Marshalling
                                     │
┌────────────────────────────────────┼────────────────────────────────────────────────────┐
│                         BASE LAYER │ (rosen/modules/render_service_base)                │
│                                    ▼                                                    │
│   ┌─────────────────────────────────────────────────────────────────────────────┐      │
│   │                    RSIRenderService / RSIRenderServiceConnection             │      │
│   │                           IPC Interface Definitions                          │      │
│   └─────────────────────────────────────────────────────────────────────────────┘      │
│   ┌──────────────────────────┐  ┌──────────────────────────┐  ┌──────────────────┐     │
│   │     RSRenderNode         │  │    RSRenderAnimation     │  │  RSRenderModifier│     │
│   │     (Server Nodes)       │  │    (Server Animation)    │  │  (Server Modifiers)  │
│   └──────────────────────────┘  └──────────────────────────┘  └──────────────────┘     │
│   ┌─────────────────────────────────────────────────────────────────────────────┐      │
│   │                    RSDrawable / RSRenderNodeDrawableAdapter                  │      │
│   │                           Drawable System                                   │      │
│   └─────────────────────────────────────────────────────────────────────────────┘      │
└────────────────────────────────────┬────────────────────────────────────────────────────┘
                                     │
┌────────────────────────────────────┼────────────────────────────────────────────────────┐
│                         SERVER LAYER (rosen/modules/render_service)                    │
│                                    │                                                    │
│   ┌────────────────────────────────┴────────────────────────────────┐                  │
│   │                       RSMainThread                              │                  │
│   │  • Command processing  • Tree updates  • Prepare() phase     │                  │
│   │  • Dirty region calc   • Sync to Render Thread                │                  │
│   └────────────────────────────────┬────────────────────────────────┘                  │
│                                    │                                                    │
│                                    ▼ Sync (RSRenderThreadParams)                       │
│   ┌─────────────────────────────────────────────────────────────────┐                  │
│   │                     RSUniRenderThread                           │                  │
│   │  • GPU rendering  • Drawable execution  • RenderEngine          │                  │
│   │  • Memory management between frames                              │                  │
│   │         ┌─────────────────────────┐                             │                  │
│   │         │   RSBaseRenderEngine    │                             │                  │
│   │         │  (OpenGL/Vulkan/Raster) │                             │                  │
│   │         └─────────────────────────┘                             │                  │
│   └────────────────────────────────┬────────────────────────────────┘                  │
│                                    │                                                    │
│                                    ▼ Layers                                            │
│   ┌─────────────────────────────────────────────────────────────────┐                  │
│   │                    HWC / Hardware Composer                       │                  │
│   │              • Hardware composition  • Buffer submission          │                  │
│   └─────────────────────────────────────────────────────────────────┘                  │
└────────────────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              DISPLAY / OUTPUT                                          │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 节点类型系统

### 客户端节点 (RSUINodeType)

| 节点类型 | 值 | 职责 |
|---------|------|------|
| `RS_NODE` | 0x0001u | 基础节点 |
| `DISPLAY_NODE` | 0x0011u | 显示/屏幕节点 |
| `SURFACE_NODE` | 0x0021u | Surface 内容节点 |
| `PROXY_NODE` | 0x0041u | 服务端节点代理 |
| `CANVAS_NODE` | 0x0081u | Canvas 绘制节点 |
| `EFFECT_NODE` | 0x0101u | 效果处理节点 |
| `ROOT_NODE` | 0x1081u | 节点树根节点 |
| `CANVAS_DRAWING_NODE` | 0x2081u | Canvas 绘制 |

**证据来源**: `rosen/modules/render_service_client/core/ui/rs_node.h`

### 服务端节点 (RSRenderNodeType)

服务端节点类型与客户端镜像对应，额外包含：
- `SCREEN_NODE` - 物理显示屏幕
- `LOGICAL_DISPLAY_NODE` - 逻辑显示

---

## 线程模型 (三阶段管线)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  RSMainThread (主线程)                                                       │
│  文件: rosen/modules/render_service/core/pipeline/main_thread/                │
│                                                                              │
│  职责:                                                                        │
│  • 处理 IPC 命令 (RecvRSTransactionData)                                      │
│  • 更新渲染节点树                                                              │
│  • 执行 Prepare() 阶段 (RSNodeVisitor)                                       │
│  • 计算脏区域                                                                  │
│  • 同步到渲染线程: RSRenderThreadParams                                       │
│                                                                              │
│  关键方法:                                                                     │
│  • Init() - 初始化事件循环                                                     │
│  • Start() - 启动主循环                                                       │
│  • PostTask() - 任务入队                                                     │
│  • ScheduleTask() - 异步任务调度                                              │
└──────────────────────┬──────────────────────────────────────────────────────┘
                       │
                       │ Sync stagingRenderThreadParams
                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  RSUniRenderThread (渲染线程)                                                 │
│  文件: rosen/modules/render_service/core/pipeline/render_thread/              │
│                                                                              │
│  职责:                                                                        │
│  • GPU 渲染执行                                                               │
│  • Drawable 生成与执行                                                        │
│  • RenderEngine 操作                                                          │
│  • 帧间内存管理                                                               │
│                                                                              │
│  关键方法:                                                                     │
│  • Render() - 主渲染循环                                                     │
│  • Sync() - 从主线程同步参数                                                   │
│  • PostRTTask() - 渲染任务入队                                               │
│  • GetRenderEngine() - 获取 GPU 上下文                                        │
│  • ClearMemoryCache() - 内存管理                                              │
└──────────────────────┬──────────────────────────────────────────────────────┘
                       │
                       │ Layers/Surfaces
                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Hardware Thread / HWC (硬件合成线程)                                         │
│                                                                              │
│  职责:                                                                        │
│  • 硬件合成                                                                    │
│  • 显示 buffer 提交                                                           │
│  • releaseInHardwareThreadTaskNum_ (Surface 参数)                             │
│                                                                              │
│  集成方式:                                                                     │
│  • RSUniRenderProcessor → HWC layers                                         │
│  • RSSurfaceRenderNodeDrawable → buffer release                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Drawable 系统

Drawable 定义渲染顺序（Slot 枚举）：

```
RSDrawableSlot 渲染顺序:
├── SAVE_ALL                    - 状态保存
├── MASK → OUTLINE             - 预处理属性 (过渡效果前)
├── BG_SAVE_BOUNDS → BG_RESTORE_BOUNDS  - 背景 (裁剪区域)
├── SAVE_FRAME → RESTORE_FRAME - 内容/帧
├── FG_SAVE_BOUNDS → FG_RESTORE_BOUNDS  - 前景 (裁剪区域)
└── POINT_LIGHT → PIXEL_STRETCH - 后处理特效 (叠加层后)
```

### Drawable 类型分类

| 类型 | 示例 | 职责 |
|------|------|------|
| **Property Drawables** | RSBackgroundColorDrawable, RSBorderDrawable | 属性渲染 |
| **Filter Drawables** | RSBackgroundFilterDrawable, RSCompositingFilterDrawable | 滤镜处理 |
| **Container Drawables** | RSChildrenDrawable, RSCustomModifierDrawable | 子节点渲染 |
| **Misc Drawables** | RSColorPickerDrawable, RSPointLightDrawable | 特殊渲染 |

**证据来源**: `rosen/modules/render_service_base/include/drawable/rs_drawable.h`

---

## 动画系统

### 客户端动画

```
RSAnimation
├── RSPropertyAnimation
│   ├── RSCurveAnimation
│   ├── RSSpringAnimation
│   ├── RSInterpolatingSpringAnimation
│   ├── RSKeyframeAnimation
│   └── RSPathAnimation
├── RSTransition
└── RSDummyAnimation
```

### 服务端动画

```
RSRenderAnimation
├── RSRenderPropertyAnimation
│   ├── RSRenderCurveAnimation
│   ├── RSRenderSpringAnimation
│   ├── RSRenderInterpolatingSpringAnimation
│   ├── RSRenderKeyframeAnimation
│   └── RSRenderPathAnimation
├── RSRenderTransition
└── RSRenderParticleAnimation
```

**证据来源**:
- `rosen/modules/render_service_client/core/animation/rs_animation.h`
- `rosen/modules/render_service_base/include/animation/rs_render_animation.h`

---

## Modifier 系统 (NG)

```
RSModifier
├── Geometry Modifiers:
│   ├── RSFrameModifier
│   ├── RSBoundsModifier
│   ├── RSTransformModifier
│   └── RSBoundsClipModifier
├── Appearance Modifiers:
│   ├── RSAlphaModifier
│   ├── RSVisibilityModifier
│   ├── RSBorderModifier
│   ├── RSShadowModifier
│   └── [20+ more]
├── Background Modifiers:
│   ├── RSBackgroundColorModifier
│   ├── RSBackgroundImageModifier
│   └── RSBackgroundShaderModifier
└── Foreground Modifiers:
    ├── RSForegroundColorModifier
    └── RSForegroundShaderModifier
```

**证据来源**: `rosen/modules/render_service_client/core/modifier_ng/rs_modifier_ng.h`

---

## IPC 通信架构

### 核心接口

| 接口 | 文件 | 职责 |
|------|------|------|
| **RSIRenderService** | `rs_irender_service.h` | 主渲染服务接口 |
| **RSIClientToServiceConnection** | `rs_iclient_to_service_connection.h` | 客户端→服务连接 |
| **RSIClientToRenderConnection** | `rs_iclient_to_render_connection.h` | 渲染连接 |

### 事务码组织

| 范围 | 功能组 |
|------|--------|
| 0x000000-0x000FFF | 核心操作 (提交、节点创建、焦点) |
| 0x001000-0x001FFF | Screen ID 查询 |
| 0x002000-0x002FFF | 虚拟屏幕管理 |
| 0x005000-0x005FFF | 刷新率与 VSync |
| 0x006000-0x006FFF | 屏幕属性 |
| 0x008000-0x008FFF | HDR、色域 |
| 0x00A000-0x00AFFF | 内存图形 |

**证据来源**: `rs_irender_service_ipc_interface_code.h`

---

## 2D 绘图引擎

### Canvas 层次

```
CoreCanvas (core_canvas.h)
└── Canvas (canvas.h)
    ├── OverDrawCanvas    - Overdraw 可视化
    ├── NoDrawCanvas      - 非绘制 Canvas
    │   └── RecordingCanvas - 命令录制
    └── (平台适配器 via SkiaCanvas)
```

### 核心绘图类

| 类 | 文件 | 职责 |
|------|------|------|
| **Canvas** | `canvas.h` | 2D 绘制表面 |
| **Paint** | `paint.h` | 绘制属性 (颜色、样式、效果) |
| **Brush** | `brush.h` | 填充样式 |
| **Pen** | `pen.h` | 描边样式 |
| **Path** | `path.h` | 几何路径 |
| **Matrix** | `matrix.h` | 变换矩阵 |
| **Font** | `font.h` | 字体渲染 |
| **Bitmap** | `bitmap.h` | 位图数据 |

**证据来源**: `rosen/modules/2d_graphics/include/draw/`

---

## 相关文档

- [目录结构](02_Directory_Structure.md) - 模块职责划分
- [N-API 接口](04_N-API.md) - 对外 JS API
- [安全评审](07_Security.md) - 信任边界与攻击面
