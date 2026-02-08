# 项目概览（Overview）

本文档介绍 `graphic_utils_lite` 项目的定位、边界、核心能力与运行环境。

## 项目定位

### 所属层级

`graphic_utils_lite` 是 OpenHarmony 图形子系统的基础设施层组件，位于以下位置：

```
┌─────────────────────────────────────────────────┐
|              OpenHarmony 系统架构               │
├─────────────────────────────────────────────────┤
|                   应用层                        │
├─────────────────────────────────────────────────┤
|                框架层（Framework）              │
│  ┌───────────────────────────────────────────┐ │
│  │           ArkUI（UI 框架）                 │ │
│  └───────────────────────────────────────────┘ │
├─────────────────────────────────────────────────┤
|              系统服务层（System Services）       │
│  ┌───────────────────────────────────────────┐ │
│  │        Window Manager（窗口管理器）         │ │
│  │        Surface（图形表面）                  │ │
│  └───────────────────────────────────────────┘ │
├─────────────────────────────────────────────────┤
|              基础服务层（Base Services）         │
│  ┌───────────────────────────────────────────┐ │
│  │      graphic_utils_lite（本项目）           │ │
│  │      公共图形工具 + 硬件抽象层              │ │
│  └───────────────────────────────────────────┘ │
├─────────────────────────────────────────────────┤
|               内核层（Kernel）                  │
├─────────────────────────────────────────────────┤
|               硬件层（Hardware）                │
└─────────────────────────────────────────────────┘
```

**证据来源**：`README.md` 依赖关系图

### 项目边界

`graphic_utils_lite` 包含以下核心功能：

| 功能模块 | 说明 | 边界 |
|----------|------|------|
| Utils | 公共数据结构、OS 适配层 | 图形子系统内部使用 |
| Diagram | 2D 图形引擎（光栅化、图元） | 渲染核心引擎 |
| Hals | 硬件抽象层（FrameBuffer、GFX） | 驱动适配桥梁 |

**不包含**：
- 窗口管理（由 `window_window_manager_lite` 提供）
- UI 控件组件（由 `arkui_ui_lite` 提供）
- 图形缓冲区管理（由 `graphic_surface_lite` 提供）

## 核心能力

### 1. 公共数据结构（Utils）

**证据位置**：`interfaces/kits/gfx_utils/` 目录下头文件

提供图形子系统通用的数据结构定义：

| 数据类型 | 用途 |
|----------|------|
| `Color` | 颜色表示与操作 |
| `Rect` | 矩形区域 |
| `Point` / `FloatPoint` | 坐标点 |
| `BufferInfo` | 图形缓冲区信息 |
| `Transform` | 基础变换 |
| `TransAffine` | 仿射变换矩阵 |

### 2. 2D 图形引擎（Diagram）

**证据位置**：`frameworks/diagram/` 目录下实现文件

提供完整的 2D 矢量图形渲染能力：

| 子模块 | 功能 |
|--------|------|
| `depiction` | 平滑曲线生成、虚线绘制 |
| `rasterizer` | 抗锯齿光栅化、裁剪处理 |
| `vertexgenerate` | 顶点数据生成 |
| `vertexprimitive` | 弧线、贝塞尔曲线、路径处理 |

**支持的图形特性**：

| 特性 | Feature Flag | 说明 |
|------|--------------|------|
| 线帽样式 | `GRAPHIC_ENABLE_LINECAP_FLAG` | 线条端点样式 |
| 线连接样式 | `GRAPHIC_ENABLE_LINEJOIN_FLAG` | 线条转角样式 |
| 椭圆 | `GRAPHIC_ENABLE_ELLIPSE_FLAG` | 椭圆绘制 |
| 贝塞尔弧线 | `GRAPHIC_ENABLE_BEZIER_ARC_FLAG` | 贝塞尔曲线弧 |
| 弧线 | `GRAPHIC_ENABLE_ARC_FLAG` | 圆弧绘制 |
| 圆角矩形 | `GRAPHIC_ENABLE_ROUNDEDRECT_FLAG` | 圆角矩形 |
| 虚线生成 | `GRAPHIC_ENABLE_DASH_GENERATE_FLAG` | 虚线效果 |
| 模糊效果 | `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 高斯模糊 |
| 阴影效果 | `GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG` | 阴影渲染 |
| 渐变填充 | `GRAPHIC_ENABLE_GRADIENT_FILL_FLAG` | 线性/径向渐变 |
| 图案填充 | `GRAPHIC_ENABLE_PATTERN_FILL_FLAG` | 纹理填充 |

**证据来源**：`BUILD.gn` 第 111-125 行 `graphic_utils_public_config`

### 3. 硬件抽象层（Hals）

**证据位置**：`frameworks/hals/` 目录下实现文件

封装显示硬件驱动接口：

| 接口 | 功能 |
|------|------|
| `GfxEngines` | GFX 图形加速引擎（单例模式） |
| `HiFbdevInit` | FrameBuffer 初始化 |
| `LcdFlush` | LCD 刷新同步 |
| `GetDevSurfaceData` | 获取显示表面数据 |

## 运行环境

### 适配系统类型

**证据来源**：`bundle.json` 第 18 行

```json
"adapted_system_type": [ "mini", "small" ]
```

| 系统类型 | 说明 | 典型设备 |
|----------|------|----------|
| mini | 轻量系统 | 智能手表、IoT 设备 |
| small | 标准系统 | 智能手机、平板 |

### 硬件要求

| 要求 | 说明 |
|------|------|
| 显示驱动 | FrameBuffer 或 HDI 显示驱动 |
| 内存 | RAM 占用 ~50KB |
| 存储 | ROM 占用 ~450KB |

### 依赖系统

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| `hilog_lite` | `//base/hiviewdfx/hilog_lite` | 日志输出 |
| `bounds_checking_function` | `//third_party/bounds_checking_function` | 内存安全 |
| `hdi_display` | `//drivers/peripheral/display/hal` | 显示硬件接口 |

### 调用链

```
ArkUI 组件渲染
    ↓
    └─→ [通过内部 API 调用]
    ↓
graphic_utils_lite (Diagram 模块)
    ↓
    └─→ GfxEngines (HAL 接口)
    ↓
HDI 显示驱动 (libdisplay_*.so)
    ↓
FrameBuffer / LCD 硬件
```

## 关键概念

### 1. 渲染流程

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   数据准备   │ →  │   图元生成   │ →  │   光栅化    │ →  │   显示输出  │
│ (Path/Shape)│    │(Vertex Gen) │    │(Rasterizer) │    │(HAL/GFX)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
     ↑                  ↑                  ↑
  geometry_*         vertexgen          rasterizer
```

### 2. 缓冲区管理

```
┌─────────────────────────────────────────────┐
│              BufferInfo 结构                 │
├─────────────────────────────────────────────┤
│ physAddr: 物理地址（显存地址）               │
│ virAddr:  虚拟地址（CPU 访问地址）           │
│ stride:   行字节 stride                      │
│ rect:     有效区域矩形                       │
│ mode:     像素格式（ColorMode 枚举）         │
└─────────────────────────────────────────────┘
```

### 3. 像素格式（ColorMode）

**证据来源**：`interfaces/kits/gfx_utils/graphic_types.h` 第 67-116 行

| 格式 | 说明 | 位深 |
|------|------|------|
| `ARGB8888` | 32 位 ARGB | 8+8+8+8 |
| `RGB565` | 16 位 RGB | 5+6+5 |
| `L8` | 8 位灰度 | 8 |
| `YUV420SP` | YUV 420 格式 | 12 |

## 产物清单

| 产物名称 | 类型 | 说明 |
|----------|------|------|
| `libgraphic_utils.so` | 动态库 | Utils + Diagram 模块 |
| `libgraphic_utils.a` | 静态库 | liteos_m 内核版本 |
| `libgraphic_hals.so` | 动态库 | HAL 模块（非 liteos_m） |

## 相关文档

| 文档 | 说明 |
|------|------|
| [架构说明](01_Architecture.md) | 详细组件图与数据流 |
| [内部 API](03_InnerAPI.md) | 模块接口清单 |
| [GN 构建配置](04_Build.md) | 编译配置详解 |
| [安全评估](05_Security.md) | 安全风险与建议 |

---

*最后更新时间：2026-02-06*
