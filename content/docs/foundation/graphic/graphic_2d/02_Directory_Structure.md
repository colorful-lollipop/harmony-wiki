# 目录结构与模块职责

## 顶层目录

```
foundation/graphic/graphic_2d/
├── figures/                      # Markdown 引用的图片目录
├── frameworks/                   # 框架代码目录
│   ├── animation_server/         # AnimationServer 代码
│   ├── bootanimation/            # 开机动画目录
│   ├── fence/                    # Fence 同步代码
│   ├── opengl_wrapper/           # OpenGL 封装 (EGL, GLES)
│   ├── surface/                  # Surface 管理代码
│   ├── surfaceimage/             # Native Image 代码
│   ├── text/                     # 文本渲染服务
│   ├── vsync/                   # VSync 代码
│   └── vulkan_layers/           # Vulkan 层实现
├── rosen/                        # Rosen 渲染框架 (核心)
│   ├── build/                   # 构建说明
│   ├── doc/                     # 文档
│   ├── include/                 # 对外头文件
│   ├── modules/                 # 子系统各模块代码
│   ├── samples/                 # 示例代码
│   ├── test/                   # 开发测试代码
│   └── tools/                   # 工具代码
├── interfaces/                   # 接口存放目录
│   ├── inner_api/              # 内部 Native 接口
│   └── kits/                    # JS/N-API 外部接口
│       ├── napi/               # Node.js API
│       ├── ani/                 # ArkNative Interface
│       └── cj/                  # CangjieScript FFI
├── utils/                       # 工具库
├── etc/                         # 配置文件
├── graphic_test/               # 测试框架
├── BUILD.gn                    # 根构建配置
├── bundle.json                 # 组件配置
├── graphic_config.gni          # Feature flags
└── CLAUDE.md                   # Claude Code 指导文件
```

---

## Rosen 模块详解

### 核心渲染模块

| 目录 | 职责 | 关键类/文件 |
|------|------|-------------|
| **render_service/** | 渲染服务 (服务端实现) | RSMainThread, RSRenderThread, HWC |
| **render_service_base/** | IPC 接口与共享数据结构 | RSIRenderService, RSNode, RSDrawable |
| **render_service_client/** | 客户端 API | RSNode, RSAnimation, RSModifier |
| **2d_graphics/** | 2D 绘图引擎 | Canvas, Paint, Path, Brush |
| **2d_engine/** | 2D 渲染引擎后端 | Skia 适配器 |
| **effect/** | 效果处理 | ColorPicker, Filter, EffectChain |
| **composer/** | 显示合成与 VSync | VSync, DisplayComposer |

### 功能模块

| 目录 | 职责 | 关键类/文件 |
|------|------|-------------|
| **animation/** | 动画引擎 | RSAnimation, RSKeyframeAnimation |
| **hyper_graphic_manager/** | 帧率管理 | HGM, RefreshRatePolicy |
| **platform/** | 平台抽象层 | PlatformAdapter, Window |
| **render_service_profiler/** | 性能分析 | Profiler, Trace |
| **ressched/** | 资源调度 | ResourceScheduler |
| **safuzz/** | 安全工具 | SecurityChecker |
| **frame_analyzer/** | 帧分析 | FrameAnalyzer |
| **frame_load/** | 帧加载 | FrameLoad |
| **frame_report/** | 帧报告 | FrameReport |
| **glfw_render_context/** | GLFW 平台适配 | GLFWContext |
| **create_pixelmap_surface/** | PixelMap Surface 创建 | PixelMapSurface |

---

## 接口层 (interfaces/)

### N-API 外部接口

| 目录 | 模块名 | JS 命名空间 | 导出类 |
|------|--------|-------------|--------|
| **kits/napi/graphic/drawing/** | drawing | `graphics.drawing` | Canvas, Font, Path, Brush, Pen, Matrix |
| **kits/napi/graphic/ui_effect/** | uieffect | `uieffect` | Filter, VisualEffect, Mask |
| **kits/napi/graphic/effect_kit/** | effectKit | `effectKit` | ColorPicker, Filter |
| **kits/napi/graphic/color_manager/** | colorSpaceManager | `graphics.colorSpaceManager` | ColorSpace, ColorSpaceManager |
| **kits/napi/graphic/animation/** | windowAnimationManager | `animation.windowAnimationManager` | RSWindowAnimationManager |
| **kits/napi/graphic/hdr_capability/** | hdrCapability | `graphics.hdrCapability` | HDR 能力查询 |
| **kits/napi/graphic/webgl/** | webgl | `webgl` | WebGLRenderingContext, WebGL2 |
| **kits/napi/graphic/hyper_graphic_manager/** | hgm | `graphics.hgm` | HGM 接口 |

### Inner API 内部接口

| 目录 | 头文件基础路径 | 导出内容 |
|------|---------------|---------|
| **inner_api/composer/** | `interfaces/inner_api/composer` | VSyncReceiver, Composer |
| **inner_api/surface/** | `interfaces/inner_api/surface` | NativeImage, Surface |
| **inner_api/bootanimation/** | `interfaces/inner_api/bootanimation` | BootAnimationUtils |
| **inner_api/common/** | `interfaces/inner_api/common` | GraphicCommon |

---

## 框架层 (frameworks/)

### 图形框架

| 目录 | 产物 | 职责 |
|------|------|------|
| **opengl_wrapper/** | libEGL.so, libGLESv2.so, libGLESv3.so, libGLv4.so | OpenGL API 封装 |
| **surfaceimage/** | libnative_image.so | Native Image 处理 |
| **bootanimation/** | bootanimation 可执行文件 | 开机动画 |
| **text/** | rosen_text | 文本渲染 |
| **vsync/** | libvsync.so | VSync 管理 |
| **vulkan_layers/** | vulkan_swapchain_layer | Vulkan 层 |

---

## 模块职责总览

### 数据流方向

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Application                                     │
│                    (RSNode Tree, RSAnimation)                                │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         render_service_client                                 │
│                    (UI 接口, 属性修改, 动画控制)                               │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                        IPC Transaction
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         render_service_base                                  │
│                   (IPC 存根/代理, 节点序列化, 权限校验)                         │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          render_service                                       │
│              (渲染管线: Main Thread → Render Thread → HWC)                    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          2d_graphics                                         │
│                          (Skia Canvas 绘图)                                  │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Platform Backends                                   │
│                      (OpenGL/Vulkan/Raster)                                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 测试目录 (已忽略)

根据 Wiki 规范，以下测试相关内容**不作为业务证据来源**：

- `rosen/test/` - 开发测试代码
- `*_test.*` - 单元测试文件
- `unittest/` - 单元测试目录
- `fuzz/` - 模糊测试目录
- `test/` - 测试相关目录

---

## 相关文档

- [架构说明](03_Architecture.md) - 组件图与线程模型
- [N-API 接口](04_N-API.md) - 对外 API 详细清单
- [GN 构建](06_Build.md) - 构建配置
