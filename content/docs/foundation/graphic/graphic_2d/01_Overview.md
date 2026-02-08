# graphic_2d 项目概览

## 项目定位

**graphic_2d** 是 OpenHarmony 图形子系统的核心仓库，提供完整的图形界面能力。

### 核心能力

graphic_2d 基于 **Rosen 渲染框架**，实现了客户端-服务器渲染架构：

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application Layer                         │
│                   (ArkUI / Native Apps)                          │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                   render_service_client                          │
│          (客户端 API: RSNode, RSAnimation, RSModifier)            │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼ IPC/Marshalling
┌─────────────────────────────────────────────────────────────────┐
│                   render_service_base                            │
│            (IPC 接口定义、共享数据结构、事务处理)                   │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                   render_service (Server)                       │
│            (渲染管线: Main Thread → Render Thread → HWC)         │
└─────────────────────────────────────────────────────────────────┘
```

### 主要功能模块

| 模块 | 能力描述 | 证据位置 |
|------|----------|----------|
| **Render Service** | UI 框架绘制能力，将 ArkUI 控件描述转换为绘制树信息，多窗口流畅和空间态 UI 共享的核心底层机制 | `README_zh.md:22` |
| **Drawing** | 提供图形子系统内部的标准化接口，完成 2D 渲染、3D 渲染和渲染引擎的管理等基本功能 | `README_zh.md:23` |
| **Animation** | 提供动画引擎的相关能力 | `README_zh.md:24` |
| **Effect** | 主要完成图片效果、渲染特效等效果处理能力，包括多效果的串联、并联处理 | `README_zh.md:25` |
| **显示与内存管理** | 图形栈与硬件解耦的主要模块，定义 OpenHarmony 显示与内存管理的能力 | `README_zh.md:26` |

---

## 系统能力 (SysCap)

graphic_2d 声明的 System Capabilities：

| System Capability | 说明 |
|-------------------|------|
| `SystemCapability.Graphic.Graphic2D.ColorManager.Core` | 颜色管理器核心能力 |
| `SystemCapability.Graphic.Graphic2D.EGL` | EGL 支持 |
| `SystemCapability.Graphic.Graphic2D.GLES2` | OpenGL ES 2.0 支持 |
| `SystemCapability.Graphic.Graphic2D.GLES3` | OpenGL ES 3.x 支持 |
| `SystemCapability.Graphic.Graphic2D.NativeBuffer` | Native Buffer 支持 |
| `SystemCapability.Graphic.Graphic2D.NativeDrawing` | Native Drawing API |
| `SystemCapability.Graphic.Graphic2D.NativeImage` | Native Image 支持 |
| `SystemCapability.Graphic.Graphic2D.NativeVsync` | Native VSync 支持 |
| `SystemCapability.Graphic.Graphic2D.NativeWindow` | Native Window 支持 |
| `SystemCapability.Graphic.Graphic2D.WebGL` | WebGL 1.0 支持 |
| `SystemCapability.Graphic.Graphic2D.WebGL2` | WebGL 2.0 支持 |
| `SystemCapability.Graphics.Vulkan` | Vulkan 支持 |
| `SystemCapability.Graphics.Drawing` | 2D 绘图能力 |

**证据来源**: `bundle.json:15-31`

---

## 依赖关系

### 主要外部依赖

| 依赖组件 | 用途 |
|----------|------|
| `skia` | 2D 图形引擎后端 |
| `egl` / `opengles` | OpenGL 图形 API |
| `vulkan-headers` / `vulkan-loader` | Vulkan 图形 API |
| `ipc` | 进程间通信框架 |
| `samgr` / `safwk` | 系统能力管理框架 |
| `ability_runtime` | 能力运行时 |
| `bundle_framework` | 包管理框架 |
| `window_manager` | 窗口管理 |
| `image_framework` | 图像处理框架 |
| `hilog` | 日志系统 |
| `ffrt` | 函数流运行时 |

**证据来源**: `bundle.json:72-134`

### 相关仓库

graphic_2d 与以下 OpenHarmony 仓库协同工作：

- [arkui_ace_engine](https://gitee.com/openharmony/arkui_ace_engine) - ArkUI 引擎
- [ability_ability_runtime](https://gitee.com/openharmony/ability_ability_runtime) - 能力运行时
- [multimedia_player_framework](https://gitee.com/openharmony/multimedia_player_framework) - 多媒体框架
- [multimedia_image_framework](https://gitee.com/openharmony/multimedia_image_framework) - 图像框架
- [windowmanager](https://gitee.com/openharmony/windowmanager) - 窗口管理
- [third_party_skia](https://gitee.com/openharmony/third_party_skia) - Skia 图形库
- [third_party_opengles](https://gitee.com/openharmony/third_party_opengles) - OpenGL ES

---

## 版本信息

| 项目 | 值 |
|------|-----|
| **组件名** | `graphic_2d` |
| **子系统** | `graphic` |
| **版本** | 3.1 |
| **License** | Apache License 2.0 |
| **发布路径** | `foundation/graphic/graphic_2d` |

**证据来源**: `bundle.json:2-8`

---

## 快速开始

### 构建

```bash
# 在 OpenHarmony 根目录下执行
./build.sh --product-name <product> --build-target graphic_2d

# 构建特定模块
./build.sh --product-name <product> --build-target librender_service
./build.sh --product-name <product> --build-target librender_service_client
./build.sh --product-name <product> --build-target 2d_graphics
```

### 运行

graphic_2d 作为系统服务运行，主要组件：

- **render_service** - 渲染服务进程 (SA)
- **librender_service_client.so** - 客户端库
- **librender_service_base.so** - 基础库
- **2d_graphics.so** - 2D 绘图引擎

---

## 相关文档

- [目录结构](02_Directory_Structure.md) - 模块划分
- [架构说明](03_Architecture.md) - 组件图与数据流
- [N-API 接口](04_N-API.md) - 对外 JS API
- [GN 构建](06_Build.md) - 构建配置与产物
- [安全评审](07_Security.md) - 安全风险分析
