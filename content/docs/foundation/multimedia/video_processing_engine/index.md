# VPE 视频处理引擎概览

本文档介绍 VPE（Video Processing Engine）视频处理引擎的项目定位、核心能力、运行环境、关键概念和模块职责。

---

## 1 项目定位

### 1.1 产品定位

VPE（Video Processing Engine）是 OpenHarmony 多媒体子系统的核心视频图像处理引擎，提供专业的视频和图像数据处理能力。

**核心定位**：
- 作为 OpenHarmony 系统级视频图像处理基础设施
- 为转码、分享、显示后处理等场景提供底层算法支持
- 支持插件化架构，允许系统开发者扩展自定义算法

**证据位置**：`README.md:4` ✅

### 1.2 能力边界

| 能力分类 | 具体能力 | 支持情况 |
|---------|---------|---------|
| **色彩空间转换** | SDR↔HDR 转换、HDR Tone Mapping | ✅ 支持 |
| **细节增强** | 超分辨率（Super Resolution）、锐化 | ✅ 支持 |
| **动态元数据生成** | HDR Vivid、Dynamic Metadata | ✅ 支持 |
| **HDR 增强** | AI HDR、对比度增强 | ✅ 支持 |
| **可变帧率** | VRR（Variable Refresh Rate） | ✅ 支持 |

**证据位置**：`README.md:4-11` ✅

### 1.3 适用场景

| 场景 | 描述 |
|------|------|
| **视频播放** | HDR 视频渲染、色彩空间适配 |
| **图像浏览** | 图像缩放、超分增强、HDR 合成 |
| **视频通话** | 实时视频质量增强 |
| **内容分享** | 跨设备图像转码、格式转换 |
| **显示后处理** | 屏幕显示算法优化 |

---

## 2 核心能力详解

### 2.1 色彩空间转换（Color Space Conversion）

**能力描述**：
实现视频和图像在不同色彩空间之间的转换，包括：
- SDR 到 SDR：基础色彩空间转换
- HDR 到 SDR：HDR Tone Mapping 降级转换
- HDR 到 HDR：HDR 色彩空间适配

**子模块**：
- `colorspace_converter` - 图像色彩空间转换
- `colorspace_converter_video` - 视频色彩空间转换
- `colorspace_converter_display` - 显示色彩空间转换

**证据位置**：`framework/algorithm/colorspace_converter/` ✅

### 2.2 细节增强（Detail Enhancement）

**能力描述**：
提供图像和视频的清晰度增强算法，包括：
- 超分辨率（Super Resolution）：基于 AI 的分辨率提升
- 锐化（Sharpening）：边缘增强算法
- 缩放（Scaling）：高质量图像缩放

**子模块**：
- `detail_enhancer` - 图像细节增强
- `detail_enhancer_video` - 视频细节增强

**证据位置**：`framework/algorithm/detail_enhancer/` ✅

### 2.3 动态元数据生成（Dynamic Metadata Generation）

**能力描述**：
为 HDR 内容生成动态元数据，支持 HDR Vivid 标准：
- 图像动态元数据生成
- 视频动态元数据生成
- 元数据与图像的合成/分解

**子模块**：
- `metadata_generator` - 图像元数据生成
- `metadata_generator_video` - 视频元数据生成

**证据位置**：`framework/algorithm/metadata_generator/` ✅

### 2.4 HDR 增强（HDR Enhancement）

**能力描述**：
提供 AI 驱动的 HDR 增强能力：
- AI HDR 图像增强
- AI HDR 视频增强
- 对比度增强

**子模块**：
- `aihdr_enhancer` - 图像 HDR 增强
- `aihdr_enhancer_video` - 视频 HDR 增强
- `contrast_enhancer` - 对比度增强

**证据位置**：`framework/algorithm/aihdr_enhancer/` ✅

### 2.5 可变帧率（Variable Refresh Rate）

**能力描述**：
提供视频可变帧率预测和调整能力，优化显示流畅度。

**子模块**：
- `video_variable_refresh_rate` - 可变帧率算法

**证据位置**：`framework/algorithm/video_variable_refresh_rate/` ✅

---

## 3 运行环境

### 3.1 硬件要求

| 组件 | 最低要求 | 推荐配置 |
|------|---------|---------|
| CPU | ARMv8-A | ARMv8-A + 多核 |
| GPU | 支持 OpenGLES 3.0 | 支持 OpenCL |
| 内存 | 512MB | 1GB+ |
| 存储 | 10MB | 20MB+ |

### 3.2 软件依赖

| 依赖模块 | 功能 | 证据位置 |
|---------|------|---------|
| `graphic_graphic_surface` | 视频 Surface 支持 | `bundle.json` ✅ |
| `graphic_graphic_2d` | 图片 SurfaceBuffer 支持 | `bundle.json` ✅ |
| `multimedia_media_foundation` | PixelMap 支持 | `bundle.json` ✅ |
| `multimedia_image_framework` | Format 参数设置 | `bundle.json` ✅ |
| `third_party_skia` | 缩放算法 | `README.md:127` ✅ |
| `third_party_opencl` | OpenCL 计算（可选） | `framework/BUILD.gn` ✅ |

### 3.3 系统要求

| 要求 | 版本 | 说明 |
|------|------|------|
| OpenHarmony | 4.1+ | 标准系统 |
| 系统能力 | SystemCapability.Multimedia.VideoProcessingEngine | 声明在 NDK 中 |

**证据位置**：`bundle.json:17-18` ✅

### 3.4 编译要求

**编译命令**：
```bash
# 32位 ARM
./build.sh --product-name {product_name} --ccache --build-target video_processing_engine

# 64位 ARM
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target video_processing_engine
```

**证据位置**：`README.md:165-173` ✅

---

## 4 关键概念

### 4.1 Surface 与 SurfaceBuffer

**概念说明**：
- **Surface**：OpenHarmony 图形系统的显示缓冲区接口
- **SurfaceBuffer**：存储图像数据的共享内存对象

**使用场景**：
- 视频解码器输出到 SurfaceBuffer
- VPE 处理输入输出均使用 SurfaceBuffer
- 通过 DMA（Direct Memory Access）零拷贝传输

**证据位置**：`framework/capi/video_processing/include/video_processing_interface.h` ✅

### 4.2 PixelMap

**概念说明**：
OpenHarmony 图像框架中的图像数据抽象，表示一个可处理的图像像素缓冲区。

**使用场景**：
- 图像处理 C API 的输入输出参数
- 支持多种像素格式和色彩空间

**证据位置**：`interfaces/kits/c/image_processing.h:204-217` ✅

### 4.3 算法插件扩展

**概念说明**：
VPE 采用插件化架构，允许系统开发者注册自定义算法实现。

**核心机制**：
- **ExtensionBase**：所有算法插件的基类
- **ExtensionManager**：插件注册和管理
- **Capability**：插件能力声明（支持的色彩空间、像素格式等）

**证据位置**：`framework/algorithm/extension_manager/` ✅

### 4.4 System Ability（SA）

**概念说明**：
OpenHarmony 的系统能力机制，VPE 服务以独立进程运行。

**特点**：
- 按需启动（lazy start）
- 自动重启（auto-restart）
- 独立进程隔离

**证据位置**：`services/sa_profile/66134.json` ✅

---

## 5 目录结构

### 5.1 顶层结构

```
/foundation/multimedia/video_processing_engine/
├── framework/                          # 框架代码
│   ├── algorithm/                      # 算法框架
│   ├── capi/                           # CAPI 层
│   ├── common/                         # 公共代码
│   └── dfx/                            # DFX 代码
├── interfaces/                         # 接口层
│   ├── inner_api/                      # 系统内部接口
│   └── kits/                           # 应用接口
│       ├── c/                          # C API 头文件
│       └── js/                        # JS API 头文件
├── services/                           # 服务代码
├── sa_profile/                         # SA 配置文件
└── BUILD.gn / config.gni               # 构建配置
```

**证据位置**：`README.md:134-161` ✅

### 5.2 Framework 目录详解

| 目录 | 职责 | 关键文件 |
|------|------|---------|
| `algorithm/` | 算法框架和插件 | extension_manager, 各算法框架 |
| `capi/image_processing/` | 图像 CAPI 实现 | image_processing.cpp, *_native.cpp |
| `capi/video_processing/` | 视频 CAPI 实现 | video_processing.cpp, *_native.cpp |
| `common/` | 公共代码 | vpe_context, algorithm_utils |
| `dfx/` | 调试和追踪 | vpe_log, vpe_trace |

### 5.3 Services 目录详解

| 目录 | 职责 | 关键文件 |
|------|------|---------|
| `src/` | 服务端实现 | video_processing_server.cpp |
| `algorithm/` | 算法工厂 | video_processing_algorithm_*.cpp |
| `utils/` | 工具类 | vpe_sa_utils, configuration_helper |
| `include/` | 服务头文件 | video_processing_client.h |

### 5.4 Interfaces 目录详解

| 目录 | 职责 | 关键文件 |
|------|------|---------|
| `inner_api/` | Inner API | detail_enhancer_image.h, colorspace_converter.h |
| `kits/c/` | C API 头文件 | image_processing.h, video_processing.h |
| `kits/js/` | JS API 头文件 | detail_enhance_napi.h, native_module_ohos_imageprocessing.cpp |
| `kits/taihe/` | Taihe/ArkTS 接口 | ani_constructor.cpp |

---

## 6 模块职责矩阵

### 6.1 按功能分类

| 功能 | 框架层 | CAPI 层 | 服务层 |
|------|-------|--------|-------|
| 色彩空间转换 | `colorspace_converter*` | `*_colorspace_converter_*` | `VideoProcessingServer` |
| 细节增强 | `detail_enhancer*` | `*_detail_enhancer_*` | `VideoProcessingServer` |
| 元数据生成 | `metadata_generator*` | `*_metadata_generator_*` | `VideoProcessingServer` |
| HDR 增强 | `aihdr_enhancer*` | `*_aihdr_enhancer_*` | `VideoProcessingServer` |
| 可变帧率 | `video_variable_refresh_rate` | - | `VideoProcessingServer` |

### 6.2 按接口分类

| 接口类型 | 位置 | 导出方式 |
|---------|------|---------|
| JS/TS N-API | `interfaces/kits/js/` | `napi_module_register` |
| C API | `interfaces/kits/c/` | `__attribute__((visibility))` |
| Inner API | `interfaces/inner_api/` | `__attribute__((visibility))` |
| SA IPC | `services/` | Binder + IDL |

---

## 7 相关文档链接

| 文档 | 说明 |
|------|------|
| [README.md](../README.md) | Wiki 使用指南 |
| [SUMMARY.md](./SUMMARY.md) | 全局导航 |
| [Architecture.md](./Architecture.md) | 详细架构设计 |
| [NAPI_Reference.md](./NAPI_Reference.md) | API 参考 |
| [Build_System.md](./Build_System.md) | 构建系统 |
| [Security_Review.md](./Security_Review.md) | 安全评审 |

---

## 8 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，完整覆盖项目定位和能力 |
