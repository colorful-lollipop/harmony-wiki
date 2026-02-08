# ImageEffect 项目概览

## 项目定位

ImageEffect 是 OpenHarmony 多媒体子系统的图像效果处理模块，提供标准化的图像编辑能力。该模块采用分层架构设计，底层通过 C/C++ 实现高性能图像处理，上层通过 N-API 向应用提供统一的 C 语言接口。

### 核心能力

| 能力 | 说明 | 代码证据 |
|------|------|----------|
| **图像滤镜** | 支持亮度、对比度、裁剪等基础效果 | `efilter/filterimpl/` |
| **自定义滤镜** | 支持开发者通过回调注册自定义效果 | `custom/filter_delegate.h` |
| **色彩空间管理** | 支持 HDR/SDR 转换、元数据处理 | `colorspace/` |
| **流水线处理** | 支持多滤镜串联、动态协商 | `effect/pipeline/` |
| **GPU 加速** | 支持 OpenGL 渲染环境 | `render_environment/` |

### 运行环境

- **系统版本**：OpenHarmony 4.0+
- **SysCap**：`SystemCapability.Multimedia.ImageEffect.Core`
- **子系统**：multimedia
- **依赖组件**：hitrace、hilog、napi、image_framework、graphic_2d、skia 等

## 目录结构

```
foundation/multimedia/image_effect/
├── frameworks/native/                    # 核心实现代码
│   ├── capi/                             # C API 封装层
│   │   ├── image_effect.cpp              # ImageEffect C 接口实现
│   │   ├── image_effect_filter.cpp       # Filter C 接口实现
│   │   ├── native_common_utils.cpp       # 工具函数
│   │   └── native_effect_base.h          # C 结构体定义
│   ├── effect/                           # 效果处理引擎
│   │   ├── base/                         # Effect 基类
│   │   ├── manager/                      # 管理器（内存、色彩空间）
│   │   └── pipeline/                     # 流水线核心
│   │       ├── core/                     # 管道核心逻辑
│   │       ├── filters/                  # 源/汇过滤器
│   │       └── factory/                  # 工厂模式
│   ├── efilter/                          # 滤镜实现
│   │   ├── base/                         # 滤镜基类
│   │   ├── custom/                       # 自定义滤镜
│   │   └── filterimpl/                   # 预置滤镜
│   │       ├── brightness/               # 亮度滤镜
│   │       ├── contrast/                 # 对比度滤镜
│   │       └── crop/                     # 裁剪滤镜
│   ├── render_environment/               # GPU 渲染环境
│   └── utils/                            # 工具类
├── interfaces/                           # 接口定义
│   ├── inner_api/native/                 # 内部 API（模块间）
│   │   ├── base/                         # 基础类型
│   │   ├── common/                       # 通用工具
│   │   ├── colorspace/                   # 色彩空间
│   │   ├── custom/                       # 自定义滤镜
│   │   ├── effect/                       # 效果主类
│   │   ├── efilter/                      # 滤镜
│   │   ├── memory/                       # 内存管理
│   │   └── utils/                        # 工具
│   └── kits/native/                      # 对外 N-API
│       ├── image_effect.h                # 主接口
│       ├── image_effect_filter.h         # 滤镜接口
│       └── image_effect_errors.h         # 错误码
├── test/                                 # 测试（不计入文档）
├── BUILD.gn                              # 根构建配置
├── bundle.json                           # 组件配置
└── config.gni                            # 项目配置
```

## 模块职责

### frameworks/native/capi

**职责**：C API 封装层，桥接 C++ 实现与外部调用。

**关键组件**：
- `OH_ImageEffect_*`：图像效果创建与管理
- `OH_EffectFilter_*`：滤镜创建与参数设置
- `NativeCommonUtils`：类型转换工具

**证据**：`interfaces/kits/native/image_effect.h:66`（OH_ImageEffect_Create 定义）

### frameworks/native/effect

**职责**：图像效果处理核心引擎。

**关键组件**：
- `ImageEffect`：主处理类，管理滤镜链
- `PipelineCore`：流水线调度
- `ColorSpaceManager`：色彩空间转换
- `EffectMemoryManager`：内存分配管理

**证据**：`interfaces/inner_api/native/effect/image_effect_inner.h`（ImageEffect 类定义）

### frameworks/native/efilter

**职责**：滤镜基类与实现。

**关键组件**：
- `EFilter`：滤镜抽象基类
- `EFilterFactory`：滤镜工厂，支持自动注册
- 预置实现：亮度、对比度、裁剪

**证据**：`interfaces/inner_api/native/efilter/efilter.h`（EFilter 类定义）

### frameworks/native/render_environment

**职责**：GPU 渲染环境管理，支持 OpenGL 加速。

**关键组件**：
- `RenderEnvironment`：EGL 上下文管理
- `RenderParam`：渲染参数

**证据**：`render_environment/render_environment.h`

### interfaces/kits/native

**职责**：对外 N-API 接口定义，供应用层调用。

**关键头文件**：
- `image_effect.h`：主入口函数
- `image_effect_filter.h`：滤镜接口
- `image_effect_errors.h`：错误码定义

**证据**：`bundle.json:86-96`（NDK 配置）

## 依赖关系

### 系统依赖

| 组件 | 用途 | 证据 |
|------|------|------|
| napi | N-API 运行时 | `BUILD.gn` external_deps |
| image_framework | 图像数据处理 | `BUILD.gn` external_deps |
| graphic_2d | 2D 图形渲染 | `BUILD.gn` external_deps |
| graphic_surface | 图形缓冲区管理 | `BUILD.gn` external_deps |
| skia | 2D 图形库 | `BUILD.gn` external_deps |
| hilog | 日志输出 | `BUILD.gn` external_deps |
| hitrace | 性能追踪 | `BUILD.gn` external_deps |

### 内部依赖

```
kits/native (对外 API)
    │
    ▼
capi (C API 封装)
    │
    ├── effect (图像效果核心)
    │       │
    │       ├── pipeline (流水线)
    │       ├── memory (内存管理)
    │       └── colorspace (色彩空间)
    │
    ├── efilter (滤镜)
    │       │
    │       ├── base (基类)
    │       └── filterimpl (预置实现)
    │
    └── custom (自定义滤镜)
```

## 关键概念

### EffectBuffer

EffectBuffer 是效果处理的核心数据结构，封装了图像数据及其元信息。

**定义**：`interfaces/inner_api/native/base/effect_buffer.h`

**关键字段**：
- `BufferInfo`：缓冲区信息（尺寸、格式）
- `ExtraInfo`：额外信息（色彩空间、HDR 元数据）
- `DataType`：数据类型

### 流水线模式

ImageEffect 采用流水线模式处理图像数据，支持多滤镜串联：

1. **Source Filter**：图像数据输入源
2. **Processing Filters**：一个或多个效果滤镜
3. **Sink Filter**：输出目标

**证据**：`effect/pipeline/filters/source/image_source_filter.cpp`

### 滤镜注册机制

通过工厂模式实现滤镜的自动注册：

```cpp
// 注册宏
REGISTER_EFILTER_FACTORY(BrightnessEFilter)
```

**证据**：`efilter/filterimpl/brightness/brightness_efilter.cpp`

## 相关文档

| 文档 | 说明 |
|------|------|
| [01_N-API_Reference.md](./01_N-API_Reference.md) | N-API 接口详解 |
| [02_Architecture.md](./02_Architecture.md) | 内部架构详解 |
| [03_Build_and_Targets.md](./03_Build_and_Targets.md) | 构建配置 |
| [04_Security_Review.md](./04_Security_Review.md) | 安全评估 |
