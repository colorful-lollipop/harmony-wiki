# 01. Skia 库概览

> **文档版本**: 1.0
> **最后更新**: 2026-02-08
> **阅读时间**: 5 分钟

---

## 目录

- [1. 原始库简介](#1-原始库简介)
- [2. Skia 核心功能](#2-skia-核心功能)
- [3. Skia 在 OpenHarmony 中的定位](#3-skia-在-openharmony-中的定位)
- [4. 版本信息](#4-版本信息)
- [5. 技术架构](#5-技术架构)
- [6. 相关资源](#6-相关资源)

---

## 1. 原始库简介

### 1.1 基本信息

| 项目 | 信息 |
|------|------|
| **库名称** | Skia |
| **维护者** | Google |
| **开源协议** | BSD 3-Clause License |
| **上游地址** | https://skia.googlesource.com/skia.git |
| **官方网站** | https://skia.org/ |
| **GitHub 镜像** | https://github.com/google/skia |

### 1.2 项目描述

**Skia** 是 Google 开发的完整 2D 图形库，用于绘制文本、几何图形和图像。它是 Chrome、Android、Flutter 等众多项目的底层图形引擎。

#### 一句话描述
> 一个高性能、跨平台的 2D 图形渲染引擎，提供文本、几何图形和图像的完整绘制能力。

### 1.3 核心特性

- **跨平台**：支持 Windows、macOS、Linux、Android、iOS、Fuchsia
- **硬件加速**：支持 OpenGL、Vulkan、Metal 等图形 API
- **丰富的功能**：文本渲染、图像编解码、滤镜特效、路径绘制
- **高性能**：SIMD 优化、多线程渲染
- **灵活的 API**：C++ API，支持多种语言绑定

---

## 2. Skia 核心功能

### 2.1 2D 图形渲染

#### Canvas API
```cpp
// 基本绘制
canvas->drawRect(rect, paint);
canvas->drawPath(path, paint);
canvas->drawCircle(center, radius, paint);
canvas->drawLine(p0, p1, paint);

// 变换
canvas->translate(dx, dy);
canvas->rotate(angle);
canvas->scale(sx, sy);
canvas->skew(sx, sy);
```

#### Paint 系统
- 颜色、透明度、混合模式
- 线宽、端点样式、连接样式
- 着色器（Shader）、滤镜（Filter）
- 路径特效（PathEffect）

### 2.2 文本渲染

#### 字体管理
- 字体加载和管理
- 字体样式选择
- 嵌入字体支持

#### 文本排版
- 复杂文本支持（BiDi、Shaping）
- 段落布局（Paragraph）
- 多行文本、换行、对齐
- 字间距、行间距控制

#### 渲染效果
- 字体平滑
- 文本阴影
- 文本渐变
- 文本轮廓

### 2.3 图像处理

#### 编解码支持
| 格式 | 编码 | 解码 | 说明 |
|------|------|------|------|
| JPEG | ✓ | ✓ | 最常用的图像格式 |
| PNG | ✓ | ✓ | 无损压缩 |
| WebP | ✓ | ✓ | Google 现代格式 |
| HEIF | ✓ | ✓ | 高效图像格式 |
| BMP | ✗ | ✓ | 位图格式 |
| WBMP | ✗ | ✓ | 无线位图格式 |
| GIF | ✗ | ✓（单帧） | 动图仅支持单帧 |

#### 图像操作
- 缩放、旋转、裁剪
- 滤镜：模糊、锐化、颜色矩阵
- 色彩空间转换
- Alpha 混合

### 2.4 特效系统

#### 图像滤镜（ImageFilter）
```cpp
// 模糊效果
auto blur = SkImageFilters::Blur(sigmaX, sigmaY, nullptr);
paint.setImageFilter(blur);

// 阴影效果
auto shadow = SkImageFilters::DropShadow(dx, dy, sigma, color, nullptr);
paint.setImageFilter(shadow);
```

#### 路径特效（PathEffect）
- 虚线
- 路径变形
- 路径修剪

#### 颜色滤镜（ColorFilter）
- 颜色矩阵
- 混合模式
- 亮度/对比度调整

### 2.5 GPU 加速

#### 后端支持
- **OpenGL ES 2.0/3.0**: 主要移动端后端
- **Vulkan**: 现代 GPU API
- **Metal**: iOS/macOS 专用
- **Direct3D**: Windows 专用

#### 性能优化
- GPU 缓存
- 批量绘制
- 延迟渲染
- 资源池化

---

## 3. Skia 在 OpenHarmony 中的定位

### 3.1 核心图形引擎

在 OpenHarmony 中，**Skia 扮演着核心图形渲染引擎的角色**，是以下子系统的基础：

| 子系统 | 作用 | Skia 提供的能力 |
|--------|------|---------------|
| **graphic_2d** | 2D 图形库 | Drawing API 封装、Canvas、Paint |
| **ace_engine** | ArkUI 引擎 | UI 组件渲染、动画效果 |
| **graphics_effect** | 图形特效库 | 模糊、滤镜、特效渲染 |
| **text** | 文本渲染 | 字体管理、文本排版 |
| **color_manager** | 颜色管理 | 色彩空间转换、CMS |

### 3.2 渲染管线

```
应用层
  ↓ Drawing API (graphic_2d)
Drawing 接口层
  ↓ 封装调用
Skia (third_party/skia)
  ↓ 使用
GPU 后端 (OpenGL/Vulkan)
  ↓
硬件 GPU
```

### 3.3 与其他组件的关系

```
┌─────────────────────────────────────────┐
│         ArkUI 应用层                    │
└─────────────┬─────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      ace_engine (ArkUI 框架)          │
│  - 组件渲染                            │
│  - 动画系统                            │
│  - 事件处理                            │
└─────────────┬─────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│       graphic_2d (Drawing API)       │
│  - Canvas 绘制                         │
│  - 图像处理                            │
│  - 文本渲染                            │
└─────────────┬─────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│          Skia (图形引擎)               │
│  - 核心渲染逻辑                       │
│  - 图像编解码                         │
│  - 字体管理                           │
└─────────────┬─────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│       GPU 后端 (OpenGL/Vulkan)        │
└─────────────────────────────────────────┘
```

---

## 4. 版本信息

### 4.1 当前版本

| 项目 | 信息 |
|------|------|
| **上游版本** | m133 |
| **OH 组件版本** | 3.1 |
| **上游标签** | chrome/m133 |
| **代码分支** | chrome/m133 |

### 4.2 版本历史

| 版本 | 上游标签 | OH 版本 | 主要变化 |
|------|---------|---------|---------|
| m133 | chrome/m133 | 3.1 | 当前使用版本，性能优化 |
| m120 | chrome/m120 | 3.0 | 前一个稳定版本 |
| m115 | chrome/m115 | 2.1 | 初始版本 |

### 4.3 版本切换机制

OpenHarmony 支持通过编译选项切换 Skia 版本：

```gn
# build_overrides/skia.gni
if (skia_feature_upgrade) {
  skia_root_dir = "//third_party/skia/m133"
} else {
  skia_root_dir = "//third_party/skia"
}
```

通过 `bundle.json` 的 `skia_feature_upgrade` 特性控制。

---

## 5. 技术架构

### 5.1 目录结构

```
third_party/skia/
├── BUILD.gn                    # OHOS 主构建入口
├── m133/                       # M133 版本（当前使用）
│   ├── BUILD.gn               # Skia 原始构建配置
│   ├── gn/                    # GN 构建配置
│   │   ├── oh_skia.gni        # OHOS 特定配置
│   │   ├── skia.gni           # 主配置
│   │   └── core.gni           # 核心源配置
│   ├── include/               # 公共头文件
│   │   ├── core/              # 核心 API
│   │   ├── effects/           # 特效 API
│   │   ├── gpu/               # GPU API
│   │   └── codecs/            # 编解码 API
│   ├── src/                   # 源代码
│   │   ├── core/              # 核心实现
│   │   ├── gpu/               # GPU 后端
│   │   ├── codecs/            # 编解码器
│   │   ├── effects/           # 特效实现
│   │   ├── ports/             # 平台相关
│   │   │   └── skia_ohos/    # OHOS 特定实现
│   │   └── utils/            # 工具类
│   ├── modules/               # 扩展模块
│   │   ├── skparagraph/       # 文本排版
│   │   ├── skshaper/         # 文本塑形
│   │   └── svg/              # SVG 渲染
│   └── third_party/          # 内嵌第三方库
│       ├── expat/
│       ├── harfbuzz/
│       ├── icu/
│       ├── libjpeg-turbo/
│       ├── libpng/
│       ├── libwebp/
│       └── zlib/
└── build_overrides/
    └── skia.gni              # 版本控制
```

### 5.2 核心模块

#### Core
- Canvas：绘制上下文
- Paint：绘制属性
- Path：路径表示
- Surface：绘制目标
- Image：图像表示

#### Effects
- ImageFilter：图像滤镜
- PathEffect：路径特效
- ColorFilter：颜色滤镜
- MaskFilter：遮罩滤镜

#### GPU
- GrDirectContext：GPU 上下文
- GrRenderTarget：渲染目标
- GrTexture：纹理
- GrBuffer：缓冲区

#### Codecs
- ImageDecoder：图像解码
- ImageEncoder：图像编码
- SkAndroidCodec：Android 特定编解码

### 5.3 平台抽象层

Skia 通过 `ports/` 目录实现平台抽象：

| 平台 | 实现 |
|------|------|
| **Android** | `src/ports/SkDebug_android.cpp` |
| **iOS/macOS** | `src/ports/SkImageGeneratorCG.cpp` |
| **Windows** | `src/ports/SkImageGeneratorWIC.cpp` |
| **OpenHarmony** | `src/ports/SkDebug_ohos.cpp`、`src/ports/skia_ohos/` |
| **Linux** | `src/ports/SkOSFile_posix.cpp` |

---

## 6. 相关资源

### 6.1 官方文档

- [Skia 官网](https://skia.org/)
- [Skia 用户指南](https://skia.org/docs/user/)
- [Skia API 文档](https://api.skia.org/)
- [Skia 源码仓库](https://skia.googlesource.com/skia.git)
- [Skia GitHub](https://github.com/google/skia)

### 6.2 社区资源

- [Skia 博客](https://skia.org/dev/blogs)
- [Skia 讨论组](https://groups.google.com/g/skia-discuss)
- [Chromium Skia 文档](https://www.chromium.org/developers/design-documents/graphics/skia)

### 6.3 OpenHarmony 相关

- [OpenHarmony 图形子系统文档](https://docs.openharmony.cn/docs/application/dev/graphics/)
- [Graphic 2D 组件文档](https://docs.openharmony.cn/docs/application/dev/graphics/2d-graphics-overview.md)
- [本 Wiki](README.md)

---

## 附录

### A. Skia 性能基准

| 操作 | CPU 后端 | GPU 后端 |
|------|---------|---------|
| 简单矩形绘制 | ~1M ops/s | ~10M ops/s |
| 路径绘制 | ~100K ops/s | ~1M ops/s |
| 文本绘制 | ~50K ops/s | ~500K ops/s |
| 图像缩放 | ~20K ops/s | ~200K ops/s |

*注：实际性能取决于硬件和场景*

### B. Skia 与其他图形库对比

| 特性 | Skia | Cairo | Cairo-GL |
|------|------|-------|-----------|
| 2D 渲染 | ✓ | ✓ | ✓ |
| GPU 加速 | ✓ | ✗ | ✓ |
| 文本渲染 | ✓ | ✓ | ✓ |
| 跨平台 | ✓ | ✓ | 部分 |
| 性能 | 高 | 中 | 高 |
| 成熟度 | 非常高 | 高 | 中 |

---

**下一节**: [02. Patch 详细分析](02_Patches.md)
