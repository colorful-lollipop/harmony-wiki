# HAL 模块说明

本文档描述 `common/hal/` 目录下各硬件抽象层模块的功能和接口。

## 模块总览

| 模块 | 路径 | 功能 | 适用系统 |
|------|------|------|---------|
| display | `common/hal/display/` | 显示 HDI | 标准系统 |
| media | `common/hal/media/` | 媒体 HDI (音频/相机/编解码) | 标准系统 |
| ai | `common/hal/ai/` | AI 推理 HDI | 标准系统 |
| multimedia | `common/hal/multimedia/` | 多媒体库 | 标准系统 |
| middleware | `common/hal/middleware/` | 中间件 (FFmpeg) | 标准系统 |
| update | `common/hal/update/` | OTA 升级 | 全部 |
| usb | `common/hal/usb/` | USB 驱动 | 标准系统 |

## display 模块

**路径**: `common/hal/display/`

### 功能描述

提供 OpenHarmony 显示子系统的 HDI (Hardware Driver Interface) 实现。

### 目录结构

```
common/hal/display/
├── BUILD.gn
└── source/
    └── display_device/
        ├── src/
        │   ├── core/        # 核心接口
        │   │   ├── hdi_device_interface.h
        │   │   ├── hdi_layer.h
        │   │   └── hdi_display.h
        │   ├── composer/   # 显示合成
        │   └── drm/        # DRM 实现
        └── libs/           # 预编译库
```

### 关键头文件

| 文件 | 功能 |
|------|------|
| `source/display_device/src/core/hdi_device_interface.h` | 显示设备接口 |
| `source/display_device/src/core/hdi_layer.h` | 显示图层接口 |
| `source/display_device/src/core/hdi_display.h` | 显示操作接口 |
| `source/display_device/src/composer/hdi_composer.h` | 显示合成器接口 |

### 产物

| 产物 | 路径模式 |
|------|---------|
| `libhdi_display.z.so` | `display/source/display_device/libs/` |

## media 模块

**路径**: `common/hal/media/`

### 功能描述

提供完整的媒体处理 HDI 实现，包括：

- **Audio**: 音频录制和播放
- **Camera**: 相机拍照和预览
- **Codec**: 视频编解码
- **Format**: 媒体格式处理
- **VideoDisplay**: 视频显示

### 目录结构

```
common/hal/media/
├── BUILD.gn
├── audio/                    # 音频 HDI
│   └── hi3751v350/
│       └── linux_standard/
│           ├── inc/         # 头文件
│           └── libs/        # 预编译库
├── camera/                   # 相机 HDI
│   └── [版本号]/
│       ├── inc/
│       └── libs/
├── codec/                    # 编解码 HDI
├── format/                   # 格式处理
└── videodisplay/             # 视频显示
    └── [版本号]/
        ├── inc/
        └── libs/
```

### Audio 子模块

**代码证据**: `common/hal/media/audio/hi3751v350/linux_standard/libs/`

| 产物 | 说明 |
|------|------|
| `libhdi_audio.z.so` | 音频 HDI 库 |

### Camera 子模块

提供 Camera Service HDI 接口，用于：

- 相机初始化和配置
- 拍照 (Capture)
- 预览 (Preview)
- 录像 (Recording)
- 帧回调处理

### Codec 子模块

提供编解码器 HDI 接口，支持：

- **编码器**: H.264, H.265, VP9
- **解码器**: H.264, H.265, VP9, AVS2

### VideoDisplay 子模块

**产物**: `libhdi_videodisplayer.so`

提供视频显示功能。

## ai 模块

**路径**: `common/hal/ai/`

### 功能描述

提供 AI 推理 HDI 接口，支持硬件加速的神经网络推理。

### 目录结构

```
common/hal/ai/
└── BUILD.gn
```

### 产物

| 产物 | 说明 |
|------|------|
| `libhdi_ai.z.so` | AI HDI 库 |

### 关键能力

- 模型加载和执行
- 张量 (Tensor) 管理
- 算子 (Operator) 支持

## multimedia 模块

**路径**: `common/hal/multimedia/`

### 功能描述

提供多媒体基础库，实现跨平台的媒体处理功能。

### 目录结构

```
common/hal/multimedia/
└── [版本号]/
    ├── inc/              # 头文件
    └── libs/             # 预编译库
```

### 产物

| 产物 | 说明 |
|------|------|
| `libhdi_media.so` | 多媒体库 |

## middleware 模块

**路径**: `common/hal/middleware/`

### 功能描述

提供中间件支持，主要包括：

- FFmpeg 适配层
- 音视频解封装/封装
- 滤镜和特效处理

### 构建配置

**代码证据**: `common/hal/middleware/BUILD.gn`

## update 模块

**路径**: `common/hal/update/`

### 功能描述

提供系统 OTA 升级服务，支持：

- 增量升级
- 全量升级
- 升级包校验
- 回滚机制

### 目录结构

```
common/hal/update/
└── BUILD.gn
```

## usb 模块

**路径**: `common/hal/usb/`

### 功能描述

提供 USB 主机和设备驱动支持。

### 子模块

| 子模块 | 功能 |
|--------|------|
| host | USB 主机驱动 |
| device | USB 设备驱动 |

## HAL 模块构建

### 根 BUILD.gn

**代码证据**: `common/hal/BUILD.gn`

```gn
# HAL 子系统构建定义
hal_subsystem {
  parts = [
    "ai",
    "display",
    "media",
    "middleware",
    "multimedia",
    "update",
    "usb",
  ]
}
```

### 各模块 BUILD.gn 模式

各子模块遵循以下模式：

```gn
# module/BUILD.gn
ohos_shared_library("module_name") {
  sources = [
    "src/*.cpp",
    "src/*.c",
  ]
  
  include_dirs = [
    "inc/",
    "//common/hal/include/",
  ]
  
  deps = [
    "//drivers/framework/ability/hdi_adapter:hdi_header",
    "//third_party/cjson: cjson",
  ]
  
  visibility = [
    "//device/soc/hisilicon/*",
  ]
}
```

## 与框架层接口

### HDI 调用链

```
OpenHarmony 框架 (foundation/)
    ↓ HDI 调用
HAL 实现 (common/hal/)
    ↓
芯片 SDK (hi*/sdk_*/)
    ↓
硬件寄存器操作
```

### 头文件搜索路径

```
//common/hal/模块/inc/
//common/hal/模块/[芯片]/inc/
//drivers/framework/ability/hdi_adapter/
```

## 相关文档

- 芯片 SDK: [06_SDK_Architecture.md](06_SDK_Architecture.md)
- 构建系统: [07_Build_System.md](07_Build_System.md)
- 安全评审: [09_Security_Review.md](09_Security_Review.md)
