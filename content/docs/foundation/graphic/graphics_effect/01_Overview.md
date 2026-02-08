# graphics_effect 项目概述

## 项目定位

graphics_effect 是 OpenHarmony 图形子系统的核心部件，提供动视效（Visual Effect）算法能力。

**证据来源**: `README.md:4` - "Graphics Effect是OpenHarmony图形子系统的重要部件"

### 定位说明

```
┌─────────────────────────────────────────────────────────────┐
│                   OpenHarmony 系统                          │
├─────────────────────────────────────────────────────────────┤
│                   图形子系统 (Graphic)                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              graphics_effect 部件                    │   │
│  │     提供模糊、阴影、渐变、灰阶、边缘光等动视效能力    │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                   上层应用 (ArkUI/系统 UI)                  │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

graphics_effect 提供以下核心动视效能力：

| 能力类别 | 具体效果 | 头文件 |
|---------|---------|--------|
| **模糊效果** | Kawase Blur, Mesa Blur, Linear Gradient Blur, Variable Radius Blur, Frosted Glass | `ge_kawase_blur_shader_filter.h`, `ge_mesa_blur_shader_filter.h` |
| **变形效果** | Displacement Distort, Bezier Warp, Grid Warp | `ge_displacement_distort_shader_filter.h`, `ge_bezier_warp_shader_filter.h` |
| **颜色效果** | Grey (灰阶), Color Gradient, Dispersion (色散), Gamma Correction | `ge_grey_shader_filter.h`, `ge_color_gradient_shader_filter.h` |
| **光效** | Edge Light (边缘光), Content Light, Direction Light, Border Light, Contour Diagonal Flow Light | `ge_edge_light_shader_filter.h`, `ge_border_light_shader.h` |
| **遮罩效果** | Ripple (波纹), PixelMap, Image Shader, Wave Gradient | `ge_ripple_shader_mask.h`, `ge_pixel_map_shader_mask.h` |
| **SDF 效果** | SDF Edge Light, SDF Shadow, SDF Border, SDF From Image | `sdf/ge_sdf_edge_light.h`, `sdf/ge_sdf_shadow_shader.h` |
| **动态加载** | Dot Matrix, Flow Light Sweep, Complex Shader | `ext/gex_dot_matrix_shader.h`, `ext/gex_complex_shader.h` |

**证据来源**: `include/` 目录头文件列表

## 运行环境

### 支持平台

| 平台 | 支持状态 | 配置标志 |
|-----|---------|---------|
| OpenHarmony (OHOS) | ✅ 支持 | `ge_is_ohos = current_os == "ohos"` |
| Linux | ✅ 支持 | `ge_is_linux = current_os == "linux"` |
| macOS | ✅ 支持 | `ge_is_mac = current_os == "mac"` |
| iOS | ✅ 支持 | `ge_is_ios = current_os == "ios"\|"tvos"` |
| Windows | ✅ 支持 | `ge_is_win = current_os == "win"\|"mingw"` |

**证据来源**: `config.gni:21-25`

### 依赖环境

| 依赖项 | 版本/要求 | 用途 |
|-------|----------|------|
| C++ Standard | C++17 | 编译标准 |
| Skia | M133+ (可选) | 图形渲染 (`USE_M133_SKIA`) |
| OpenHarmony SDK | 标准版 | 系统 API |

**证据来源**: `CLAUDE.md:122` - "-std=c++17"

## 代码规模

| 指标 | 数量 |
|-----|------|
| 头文件 | 69 |
| 源文件 (src/) | 60 |
| SDF 模块 | 11 |
| Extension 模块 | 4 |
| Test (不计入) | 30+ |

## 关键概念

### 1. GEVisualEffect (动视效对象)

动视效的基本单元，通过 `SetParam()` 方法设置参数：

```cpp
// 证据: include/ge_visual_effect.h:48-72
void SetParam(const std::string& tag, int32_t param);
void SetParam(const std::string& tag, float param);
void SetParam(const std::string& tag, const std::shared_ptr<Drawing::Image> param);
// ... 20+ 重载
```

### 2. GEVisualEffectContainer (动视效容器)

管理多个动视效的链式应用：

```cpp
// 证据: include/ge_visual_effect_container.h:29
void AddToChainedFilter(std::shared_ptr<Drawing::GEVisualEffect> visualEffect);
```

### 3. GERender (渲染引擎)

负责将动视效应用到图像并渲染：

```cpp
// 证据: include/ge_render.h:48-50, 82-84
void DrawImageEffect(Drawing::Canvas& canvas, ...);
std::shared_ptr<Drawing::Image> ApplyImageEffect(Drawing::Canvas& canvas, ...);
```

## 版本信息

| 属性 | 值 |
|-----|------|
| 当前版本 | 1.0 |
| License | Apache License 2.0 |
| 子系统 | graphic |
| 部件名 | graphics_effect |

**证据来源**: `bundle.json:2-6`

## 相关文档

- [架构说明](02_Architecture.md) - 详细架构设计
- [内部 API](03_InnerAPIs.md) - API 参考
- [构建指南](04_Build.md) - 编译配置
