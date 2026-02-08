# graphics_effect 架构说明

## 整体架构

graphics_effect 采用三层架构设计，从上到下依次为：

```
┌─────────────────────────────────────────────────────────────────┐
│                    接口层 (Interface Layer)                     │
│     ArkUI / UIEffect / EffectKit (通过 C++ 接口调用)             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   实现层 (Implementation Layer)                  │
├─────────────────┬─────────────────────┬─────────────────────────┤
│   GERender      │  GEVisualEffect     │ GEVisualEffectContainer │
│   (渲染编排)     │   (效果实现)         │   (效果容器)            │
└─────────────────┴─────────────────────┴─────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    渲染管线 (Rendering Pipeline)                 │
│   Shader Filter → Filter Composer → Canvas / Image               │
└─────────────────────────────────────────────────────────────────┘
```

**证据来源**: `README.md:9-18` - "Graphics Effect的分层说明"

---

## 核心组件

### 1. GERender (渲染编排器)

**定位**: 渲染管线的编排中心，负责将动视效应用到图像

**头文件**: `include/ge_render.h`

**核心职责**:
1. 接收 `GEVisualEffectContainer`（效果链）
2. 生成对应的 `GEShaderFilter`
3. 执行渲染管线
4. 输出结果到 Canvas 或 Image

**关键 API**:

| API | 功能 | 返回类型 |
|-----|------|---------|
| `DrawImageEffect()` | 绘制到画布 | void |
| `ApplyImageEffect()` | 应用效果返回 Image | `std::shared_ptr<Image>` |
| `ApplyHpsGEImageEffect()` | HPS 优化管线 | `ApplyHpsGEResult` |
| `GenerateShaderFilter()` | 生成 Shader Filter | `std::shared_ptr<GEShaderFilter>` |

**证据来源**: `include/ge_render.h:42-84`

### 2. GEVisualEffect (动视效基类)

**定位**: 所有动视效的基类，定义统一的参数设置接口

**头文件**: `include/ge_visual_effect.h`

**核心职责**:
1. 持有效果名称和类型
2. 管理效果参数 (`SetParam`)
3. 生成对应的 Shader Mask/Shape

**参数类型支持** (20+ 重载):

| 类型 | 示例 |
|-----|------|
| 数值 | `int32_t`, `float`, `double`, `int64_t` |
| 几何 | `Vector2f`, `Vector3f`, `Vector4f`, `Matrix`, `GERRect` |
| 图像 | `std::shared_ptr<Image>` |
| Shader | `GEShaderMask`, `GEShaderShape` |
| SDF | `GESDFBorderParams`, `GESDFShadowParams` |

**证据来源**: `include/ge_visual_effect.h:48-72`

### 3. GEVisualEffectContainer (动视效容器)

**定位**: 管理多个动视效的链式组合

**头文件**: `include/ge_visual_effect_container.h`

**核心职责**:
1. 存储效果链 (`filterVec_`)
2. 添加/移除效果
3. 管理几何信息和缓存

**关键 API**:

| API | 功能 |
|-----|------|
| `AddToChainedFilter()` | 添加效果到链 |
| `GetFilters()` | 获取效果列表 |
| `SetGeometry()` | 设置几何变换 |
| `RemoveFilterWithType()` | 按类型移除 |

**证据来源**: `include/ge_visual_effect_container.h:29-55`

---

## 效果类型体系

```
IGEFilterType (ge_filter_type.h)
├── GEShaderFilter (ge_shader_filter.h)
│   ├── Blur Filters
│   │   ├── GEKawaseBlurShaderFilter
│   │   ├── GEMesaBlurShaderFilter
│   │   ├── GELinearGradientBlurShaderFilter
│   │   └── GEVariableRadiusBlurShaderFilter
│   ├── Distortion Filters
│   │   ├── GEDisplacementDistortShaderFilter
│   │   ├── GEBezierWarpShaderFilter
│   │   └── GEGridWarpShaderFilter
│   ├── Color Filters
│   │   ├── GEGreyShaderFilter
│   │   ├── GEColorGradientShaderFilter
│   │   ├── GEDispersionShaderFilter
│   │   └── GEGammaCorrectionFilter
│   └── Light Filters
│       ├── GEEdgeLightShaderFilter
│       ├── GEContentLightShaderFilter
│       ├── GEDirectionLightShaderFilter
│       └── GEBorderLightShader
├── GEShader (ge_shader.h)
└── GEShaderMask (ge_shader_mask.h)
```

**证据来源**: `CLAUDE.md:62-73`

---

## SDF 子系统

SDF (Signed Distance Field) 用于高质量的形状渲染和特效：

```
src/sdf/ & include/sdf/
├── GESDFShaderShape (基类)
│   ├── GESDFRRectShaderShape (圆角矩形)
│   ├── GESDFPixelmapShaderShape (像素图)
│   ├── GESDFTransformShaderShape (变换)
│   └── GESDFUnionOpShaderShape (组合)
├── GESDFEdgeLight (边缘光)
├── GESDFShadowShader (阴影)
├── GESDFBorderShader (边框)
├── GESDFColorShader (颜色)
├── GESDFClipShader (裁剪)
└── GESDFFromImageFilter (图像转 SDF)
```

**证据来源**: `CLAUDE.md:77-86`, `src/sdf/` 目录

---

## 动态加载框架

Extension 模块支持运行时动态加载自定义 shader 算法：

```
src/ext/ & include/ext/
├── GEXDotMatrixShader (点阵效果)
├── GEXFlowLightSweepShader (流光扫描)
├── GEXComplexShader (复杂组合)
└── GEXMarshallingHelper (序列化支持)
```

**证据来源**: `CLAUDE.md:88-95`

---

## 数据流

### 典型渲染流程

```
┌──────────────┐     ┌───────────────────────┐     ┌──────────────┐
│   输入 Image  │────▶│  GEVisualEffectContainer │────▶│   输出 Image  │
└──────────────┘     │  (效果链定义)           │     │  (带特效)     │
                     └───────────────────────┘     └──────────────┘
                              │
                              ▼
                     ┌───────────────────────┐
                     │      GERender          │
                     │   (渲染管线编排)        │
                     └───────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │  Shader      │ │   Filter     │ │   Canvas     │
     │  Generation  │ │  Composer    │ │   Output     │
     └──────────────┘ └──────────────┘ └──────────────┘
```

**证据来源**: `include/ge_render.h` - 渲染 API 设计和注释

---

## 线程模型

### 渲染线程

| 阶段 | 线程 | 说明 |
|-----|------|------|
| 效果创建 | 主线程 | GEVisualEffect 参数设置 |
| Shader 编译 | 渲染线程 | RuntimeEffect 编译 |
| 图像处理 | 渲染线程 | GPU 加速 Shader 执行 |
| 缓存管理 | 渲染线程 | IGECacheProvider |

**注意事项**:
- Shader 编译可能阻塞渲染线程
- 建议预编译常用 Shader
- 缓存需要考虑线程安全

---

## 依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        外部依赖                                   │
├─────────────────────────────────────────────────────────────────┤
│  graphic_2d (2D 绘图)                                           │
│  hilog (日志)                                                   │
│  bounds_checking_function (安全函数)                             │
│  c_utils (工具库)                                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     graphics_effect                             │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  GERender ──▶ GEVisualEffectContainer ──▶ GEShaderFilter│  │
│  │       │                    │                     │        │  │
│  │       │                    ▼                     ▼        │  │
│  │       │            GEVisualEffect        SDF / Extension │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**证据来源**: `BUILD.gn:130-147` - 外部依赖声明

---

## 相关文档

- [项目概述](01_Overview.md) - 定位和核心能力
- [内部 API](03_InnerAPIs.md) - 详细 API 文档
- [构建指南](04_Build.md) - 编译配置
