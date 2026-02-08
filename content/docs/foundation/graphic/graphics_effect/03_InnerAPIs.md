# graphics_effect 内部 API 文档

## 概述

graphics_effect 是纯 C++ 库，通过 C++ 接口向上层（ArkUI/UIEffect/EffectKit）提供动视效能力。**本模块不提供 N-API**。

**证据**: `grep "napi_\|NAPI_MODULE"` 无匹配结果

---

## C++ API 分层

### 1. 渲染层 API (GERender)

**头文件**: `include/ge_render.h`  
**命名空间**: `OHOS::GraphicsEffectEngine`

#### 核心方法

| 方法 | 功能 | 参数 | 返回值 |
|-----|------|------|--------|
| `GERender()` | 构造函数 | - | - |
| `~GERender()` | 析构函数 | - | - |
| `DrawImageEffect()` | 绘制效果到画布 | `canvas`, `veContainer`, `image`, `src`, `dst`, `sampling` | void |
| `ApplyImageEffect()` | 应用效果返回 Image | `canvas`, `veContainer`, `context`, `sampling` | `std::shared_ptr<Image>` |
| `ApplyHpsGEImageEffect()` | HPS 优化管线 | `canvas`, `veContainer`, `context`, `outImage`, `brush` | `ApplyHpsGEResult` |
| `DrawShaderEffect()` | 绘制 Shader 效果 | `canvas`, `veContainer`, `bounds` | void |

#### 内部辅助方法

| 方法 | 功能 | 可见性 |
|-----|------|--------|
| `GenerateShaderFilter()` | 生成 Shader Filter | private |
| `GenerateShaderKawaseBlur()` | 生成 Kawase 模糊 | private |
| `GenerateExtShaderFrostedGlass()` | 生成毛玻璃效果 | private |
| `BeforeApplyShaderFilter()` | Shader 前置处理 | private |
| `AfterApplyShaderFilter()` | Shader 后置处理 | private |

**证据来源**: `include/ge_render.h:42-232`

---

### 2. 效果容器 API (GEVisualEffectContainer)

**头文件**: `include/ge_visual_effect_container.h`  
**命名空间**: `OHOS::Rosen::Drawing`

#### 核心方法

| 方法 | 功能 | 参数 |
|-----|------|------|
| `GEVisualEffectContainer()` | 构造函数 | - |
| `AddToChainedFilter()` | 添加效果到链 | `visualEffect` |
| `GetFilters()` | 获取效果列表 | const |
| `SetGeometry()` | 设置几何变换 | `matrix`, `bound`, `materialDst`, `geoWidth`, `geoHeight` |
| `RemoveFilterWithType()` | 按类型移除 | `typeToRemove` |
| `SetDisplayHeadroom()` | 设置显示余量 | `headroom` |

#### 缓存相关

| 方法 | 功能 |
|-----|------|
| `UpdateCacheDataFrom()` | 从容器更新缓存 |
| `UpdateCachedBlurImage()` | 更新模糊缓存 |
| `GetGEVisualEffect()` | 按名称获取效果 |

**证据来源**: `include/ge_visual_effect_container.h:24-59`

---

### 3. 动视效基类 API (GEVisualEffect)

**头文件**: `include/ge_visual_effect.h`  
**命名空间**: `OHOS::Rosen::Drawing`

#### 构造函数

```cpp
GEVisualEffect(const std::string& name,
    DrawingPaintType type = DrawingPaintType::BRUSH,
    const std::optional<Drawing::CanvasInfo>& canvasInfo = std::nullopt);
```

#### SetParam 重载 (参数设置)

| 重载类型 | 示例 |
|---------|------|
| `int32_t` | `SetParam("radius", 10)` |
| `float` | `SetParam("intensity", 0.5f)` |
| `double` | `SetParam("scale", 1.0)` |
| `int64_t` | `SetParam("flags", 0LL)` |
| `const char*` | `SetParam("name", "blur")` |
| `bool` | `SetParam("enabled", true)` |
| `uint32_t` | `SetParam("color", 0xFF0000)` |
| `std::shared_ptr<Image>` | `SetParam("maskImage", image)` |
| `std::shared_ptr<ColorFilter>` | `SetParam("colorFilter", cf)` |
| `Matrix` | `SetParam("transform", matrix)` |
| `std::pair<float, float>` | `SetParam("size", {100, 200})` |
| `std::vector<float>` | `SetParam("weights", weights)` |
| `Vector2f` / `Vector3f` / `Vector4f` | `SetParam("offset", Vector3f(10, 10, 0))` |
| `Color4f` | `SetParam("color", Color4f(1, 0, 0, 1))` |
| `GERRect` | `SetParam("bounds", rect)` |
| `GESDFBorderParams` | `SetParam("border", params)` |
| `GESDFShadowParams` | `SetParam("shadow", params)` |

**证据来源**: `include/ge_visual_effect.h:48-72`

#### 获取方法

| 方法 | 功能 |
|-----|------|
| `GetName()` | 获取效果名称 |
| `GetImpl()` | 获取内部实现 |
| `GetCanvasInfo()` | 获取画布信息 |
| `GetSupportHeadroom()` | 获取支持余量 |
| `GenerateShaderMask()` | 生成 Shader Mask |
| `GenerateShaderShape()` | 生成 Shader Shape |

---

## 效果类型清单

### 模糊效果 (Blur Filters)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GEKawaseBlurShaderFilter | `ge_kawase_blur_shader_filter.h` | Kawase 高质量模糊 |
| GEMesaBlurShaderFilter | `ge_mesa_blur_shader_filter.h` | Mesa 模糊融合 |
| GELinearGradientBlurShaderFilter | `ge_linear_gradient_blur_shader_filter.h` | 线性渐变模糊 |
| GEVariableRadiusBlurShaderFilter | `ge_variable_radius_blur_shader_filter.h` | 可变半径模糊 |
| GEFrostedGlassShaderFilter | `ge_frosted_glass_shader_filter.h` | 毛玻璃效果 |
| GEFrostedGlassBlurShaderFilter | `ge_frosted_blur_shader_filter.h` | 毛玻璃模糊 |

### 变形效果 (Distortion Filters)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GEDisplacementDistortShaderFilter | `ge_displacement_distort_shader_filter.h` | 置换变形 |
| GEBezierWarpShaderFilter | `ge_bezier_warp_shader_filter.h` | 贝塞尔曲线变形 |
| GEGridWarpShaderFilter | `ge_grid_warp_shader_filter.h` | 网格变形 |

### 颜色效果 (Color Filters)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GEGreyShaderFilter | `ge_grey_shader_filter.h` | 灰阶效果 |
| GEColorGradientShaderFilter | `ge_color_gradient_shader_filter.h` | 颜色渐变 |
| GEDispersionShaderFilter | `ge_dispersion_shader_filter.h` | 色散效果 |
| GEGammaCorrectionFilter | `ge_gamma_correction_filter.h` | Gamma 校正 |

### 光效 (Light Filters)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GEEdgeLightShaderFilter | `ge_edge_light_shader_filter.h` | 边缘发光 |
| GEContentLightShaderFilter | `ge_content_light_shader_filter.h` | 内容高光 |
| GEDirectionLightShaderFilter | `ge_direction_light_shader_filter.h` | 方向光 |
| GEBorderLightShader | `ge_border_light_shader.h` | 边框光 |
| GEAuroraNoiseShader | `ge_aurora_noise_shader.h` | 极光噪声 |
| GEContourDiagonalFlowLightShader | `ge_contour_diagonal_flow_light_shader.h` | 等高线流光 |
| GEParticleCircularHaloShader | `ge_particle_circular_halo_shader.h` | 粒子光环 |

### 遮罩效果 (Mask Effects)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GERippleShaderMask | `ge_ripple_shader_mask.h` | 波纹遮罩 |
| GEPixelMapShaderMask | `ge_pixel_map_shader_mask.h` | 像素图遮罩 |
| GEImageShaderMask | `ge_image_shader_mask.h` | 图像遮罩 |
| GELinearGradientShaderMask | `ge_linear_gradient_shader_mask.h` | 线性渐变遮罩 |
| GERadialGradientShaderMask | `ge_radial_gradient_shader_mask.h` | 径向渐变遮罩 |
| GEFrameGradientShaderMask | `ge_frame_gradient_shader_mask.h` | 框架渐变遮罩 |
| GEWaveGradientShaderMask | `ge_wave_gradient_shader_mask.h` | 波动渐变遮罩 |
| GEDoubleRippleShaderMask | `ge_double_ripple_shader_mask.h` | 双波纹遮罩 |

### 复合效果 (Composite Effects)

| 效果 | 头文件 | 说明 |
|-----|--------|------|
| GECircleFlowlightEffect | `ge_circle_flowlight_effect.h` | 圆形流光 |
| GEColorGradientEffect | `ge_color_gradient_effect.h` | 颜色渐变效果 |
| GEMagnifierShaderFilter | `ge_magnifier_shader_filter.h` | 放大镜 |
| GEAIBarShaderFilter | `ge_aibar_shader_filter.h` | AI 栏效果 |
| GEMapColorByBrightnessShaderFilter | `ge_map_color_by_brightness_shader_filter.h` | 亮度映射 |
| GEMaskTransitionShaderFilter | `ge_mask_transition_shader_mask.h` | 遮罩过渡 |
| GEWaterRippleFilter | `ge_water_ripple_filter.h` | 水波纹 |
| GEWaterDropletTransitionFilter | `ge_water_droplet_transition_filter.h` | 水滴过渡 |
| GESoundWaveFilter | `ge_sound_wave_filter.h` | 声波动效 |

---

## Shader 参数标签

效果参数通过字符串标签识别，常用标签定义在 `ge_shader_filter_params.h`：

| 标签 | 类型 | 说明 |
|-----|------|------|
| `"radius"` | float | 模糊半径 |
| `"intensity"` | float | 效果强度 |
| `"color"` | uint32_t/Color4f | 颜色 |
| `"offset"` | Vector2f/Vector3f | 偏移 |
| `"scale"` | float | 缩放 |
| `"brightness"` | float | 亮度 |
| `"saturation"` | float | 饱和度 |
| `"maskImage"` | Image | 遮罩图像 |

---

## 稳定性标注

### 稳定接口

| 接口 | 状态 | 依据 |
|-----|------|------|
| `GERender` | 稳定 | 核心类，API 完整 |
| `GEVisualEffectContainer` | 稳定 | 容器类，API 完整 |
| `GEVisualEffect::SetParam` | 稳定 | 通用参数设置 |

### 实验性接口

| 接口 | 状态 | 说明 |
|-----|------|------|
| `ApplyHpsGEImageEffect()` | 稳定 | HPS 优化已成熟 |
| SDF 系列 | 稳定 | SDF 架构已完善 |
| Extension 系列 | 稳定 | 动态加载已稳定 |

---

## 使用示例

### 基本用法

```cpp
#include "ge_render.h"
#include "ge_visual_effect.h"
#include "ge_visual_effect_container.h"

using namespace OHOS::GraphicsEffectEngine;
using namespace OHOS::Rosen::Drawing;

// 1. 创建渲染器
auto geRender = std::make_shared<GERender>();

// 2. 创建效果容器
auto veContainer = std::make_shared<GEVisualEffectContainer>();

// 3. 创建动视效
auto blurEffect = std::make_shared<GEVisualEffect>("blur");
blurEffect->SetParam("radius", 10.0f);

// 4. 添加到容器
veContainer->AddToChainedFilter(blurEffect);

// 5. 应用效果
auto canvas = Drawing::Canvas();
auto image = std::make_shared<Drawing::Image>();
auto resultImage = geRender->ApplyImageEffect(canvas, *veContainer,
    {/* context */}, Drawing::SamplingOptions());
```

**证据来源**: `README.md:41-58` - 官方示例代码

---

## 相关文档

- [项目概述](01_Overview.md) - 定位和核心能力
- [架构说明](02_Architecture.md) - 详细架构设计
- [构建指南](04_Build.md) - 编译配置
