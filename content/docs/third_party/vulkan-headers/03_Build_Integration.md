# OH 构建适配

## 3.1 构建系统概述

### 构建系统类型

| 属性 | 值 |
|------|-----|
| **上游构建系统** | CMake |
| **OH 构建系统** | GN (Generate Ninja) |
| **构建目标类型** | 静态库 (ohos_static_library) |

### 构建流程

```
┌─────────────────────┐
│   GN 构建配置        │
│   (BUILD.gn)        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   头文件编译         │
│   (Header-only)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   静态库输出         │
│   libvulkan_headers.a│
└─────────────────────┘
```

## 3.2 BUILD.gn 结构说明

### 完整 BUILD.gn 文件

```gn
# Copyright 2018-2023 The ANGLE Project Authors.
# Copyright 2019-2023 LunarG, Inc.
#
# SPDX-License-Identifier: Apache-2.0

import("//build/ohos.gni")

is_ohos = current_os == "ohos"
is_android = current_os == "android"
is_mac = current_os == "ios" || current_os == "tvos" || current_os == "mac"
is_win = current_os == "win" || current_os == "mingw"
is_fuchsia = current_os == "fuchsia"
is_apple = current_os == "apple"

config("vulkan_headers_config") {
  include_dirs = [ "include" ]
  defines = []

  if (is_ohos) {
    defines += [ "VK_USE_PLATFORM_OHOS" ]
  }

  if (is_win) {
    defines += [ "VK_USE_PLATFORM_WIN32_KHR" ]
  }
  if (defined(vulkan_use_x11) && vulkan_use_x11) {
    defines += [ "VK_USE_PLATFORM_XCB_KHR" ]
  }
  if (defined(vulkan_use_wayland) && vululkan_use_wayland) {
    defines += [ "VK_USE_PLATFORM_WAYLAND_KHR" ]
    if (defined(vulkan_wayland_include_dirs)) {
      include_dirs += vulkan_wayland_include_dirs
    }
  }
  if (is_android) {
    defines += [ "VK_USE_PLATFORM_ANDROID_KHR" ]
  }
  if (is_fuchsia) {
    defines += [ "VK_USE_PLATFORM_FUCHSIA" ]
  }
  if (is_apple) {
    defines += [ "VK_USE_PLATFORM_METAL_EXT" ]
  }
  if (is_mac) {
    defines += [ "VK_USE_PLATFORM_MACOS_MVK" ]
  }
  if (is_ios) {
    defines += [ "VK_USE_PLATFORM_IOS_MVK" ]
  }
  if (defined(is_ggp) && is_ggp) {
    defines += [ "VK_USE_PLATFORM_GGP" ]
  }
  if (is_clang) {
    cflags = [ "-Wno-redundant-parens" ]
  }
}

# Vulkan headers only, no compiled sources.
ohos_static_library("vulkan_headers") {
  sources = [
    "include/vk_video/vulkan_video_codec_av1std.h",
    "include/vk_video/vulkan_video_codec_av1std_decode.h",
    "include/vk_video/vulkan_video_codec_av1std_encode.h",
    "include/vk_video/vulkan_video_codec_h264std.h",
    "include/vk_video/vulkan_video_codec_h264std_decode.h",
    "include/vk_video/vulkan_video_codec_h264std_encode.h",
    "include/vk_video/vulkan_video_codec_h265std.h",
    "include/vk_video/vulkan_video_codec_h265std_decode.h",
    "include/vk_video/vulkan_video_codec_h265std_encode.h",
    "include/vk_video/vulkan_video_codecs_common.h",
    "include/vulkan/vk_icd.h",
    "include/vulkan/vk_layer.h",
    "include/vulkan/vk_platform.h",
    "include/vulkan/vulkan.h",
    "include/vulkan/vulkan.hpp",
    "include/vulkan/vulkan_core.h",
    "include/vulkan/vulkan_ohos.h",
    "include/vulkan/vulkan_screen.h",
  ]
  public_configs = [ ":vulkan_headers_config" ]

  license_file = "//third_party/vulkan-headers/LICENSES/Apache-2.0.txt"
}
```

### 配置说明

#### 平台判断变量

| 变量 | 平台 | 说明 |
|------|------|------|
| `is_ohos` | OpenHarmony | 主要适配平台 |
| `is_android` | Android | Android 平台支持 |
| `is_mac` | iOS/tvOS/macOS | Apple 平台 |
| `is_win` | Windows/MinGW | Windows 平台 |
| `is_fuchsia` | Fuchsia | Google Fuchsia |
| `is_apple` | Apple (Metal) | Metal 图形 API |

#### OH 特有配置

```gn
if (is_ohos) {
  defines += [ "VK_USE_PLATFORM_OHOS" ]
}
```

**作用**：
- 定义 `VK_USE_PLATFORM_OHOS` 宏
- 启用 vulkan_ohos.h 中的 OHOS 平台扩展
- 其他模块可通过该宏检测是否在 OH 平台编译

## 3.3 关键编译选项

### 宏定义

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `VK_USE_PLATFORM_OHOS` | 1 | 启用 OHOS 平台支持 |
| `VK_USE_PLATFORM_ANDROID_KHR` | 条件定义 | Android 平台支持 |
| `VK_USE_PLATFORM_WIN32_KHR` | 条件定义 | Windows 平台支持 |
| `VK_USE_PLATFORM_XCB_KHR` | 条件定义 | X11 XCB 支持 |
| `VK_USE_PLATFORM_WAYLAND_KHR` | 条件定义 | Wayland 支持 |
| `VK_USE_PLATFORM_FUCHSIA` | 条件定义 | Fuchsia 支持 |
| `VK_USE_PLATFORM_METAL_EXT` | 条件定义 | Apple Metal 支持 |
| `VK_USE_PLATFORM_MACOS_MVK` | 条件定义 | macOS MoltenVK 支持 |
| `VK_USE_PLATFORM_IOS_MVK` | 条件定义 | iOS MoltenVK 支持 |
| `VK_USE_PLATFORM_GGP` | 条件定义 | Google Games Platform |

### 头文件包含路径

```gn
include_dirs = [ "include" ]
```

**路径结构**：
```
third_party/vulkan-headers/
└── include/
    ├── vulkan/
    │   ├── vulkan.h                    # 主头文件
    │   ├── vulkan_core.h               # 核心 API
    │   ├── vulkan.hpp                 # C++ 封装
    │   ├── vulkan_ohos.h               # OH 特有扩展
    │   ├── vulkan_screen.h             # QNX Screen
    │   └── vk_icd.h                    # Installable Client Driver
    └── vk_video/
        ├── vulkan_video_codec_av1std.h
        ├── vulkan_video_codec_h264std.h
        ├── vulkan_video_codec_h265std.h
        └── vulkan_video_codecs_common.h
```

### 编译器选项

```gn
if (is_clang) {
  cflags = [ "-Wno-redundant-parens" ]
}
```

**作用**：抑制 Clang 编译器的冗余括号警告

## 3.4 与上游构建系统的差异

### CMake vs BUILD.gn

| 特性 | 上游 CMake | OH BUILD.gn |
|------|-----------|-------------|
| **源文件处理** | 自动扫描头文件 | 显式列出源文件 |
| **平台检测** | CMake 平台变量 | GN 条件判断 |
| **OH 支持** | 无 | 添加 `is_ohos` 分支 |
| **OH 特有文件** | 无 | 显式添加 vulkan_ohos.h |
| **输出类型** | 头文件安装 | 静态库 |

### 差异说明

#### 1. 源文件显式列出

**CMake 方式**（上游）：
```cmake
# CMakeLists.txt 中使用 file(GLOB) 自动收集
file(GLOB HEADERS "include/*.h")
```

**BUILD.gn 方式**（OH）：
```gn
sources = [
  "include/vulkan/vulkan.h",
  "include/vulkan/vulkan_core.h",
  "include/vulkan/vulkan_ohos.h",  # 显式添加 OH 特有文件
  # ...
]
```

**原因**：
- GN 不推荐使用 glob 收集源文件
- 显式列出便于控制哪些文件被编译

#### 2. 平台配置分支

**上游**：CMake 使用 `if(WIN32)`、`if(ANDROID)` 等平台变量

**OH**：使用 `is_ohos`、`is_android` 等 GN 变量

### OH 特有的构建修改

| 修改类型 | 上游 | OH | 说明 |
|---------|------|-----|------|
| **平台宏** | 无 | `VK_USE_PLATFORM_OHOS` | 启用 OHOS 平台支持 |
| **源文件** | 无 | `vulkan_ohos.h` | 新增 OH 特有头文件 |
| **配置** | 无 | `is_ohos` 判断 | 平台条件编译 |

## 3.5 特殊处理

### 头文件库特性

Vulkan-Headers 是**纯头文件库**，这意味着：

1. **无编译单元**
   - 库中不包含 `.c` 或 `.cpp` 源文件
   - 所有源文件都是 `.h` 头文件
   - 编译时不会产生目标文件 (.o/.obj)

2. **静态库本质**
   - `ohos_static_library` 目标本质上是头文件的归档
   - 链接时不会增加二进制大小
   - 只会添加头文件搜索路径

3. **链接方式**
   ```gn
   public_configs = [ ":vulkan_headers_config" ]
   ```
   - 通过 `public_configs` 导出配置
   - 依赖模块只需声明 `deps` 即可获得头文件路径

### OH 特有头文件的导出

```gn
sources = [
  # ...
  "include/vulkan/vulkan_ohos.h",      # OH 特有
  "include/vulkan/vulkan_screen.h",    # QNX 平台
]
```

**说明**：
- 这些头文件被包含在静态库中
- 依赖模块通过 `public_configs` 获得头文件路径
- 无需额外配置即可使用 OH 扩展

### 许可证导出

```gn
license_file = "//third_party/vulkan-headers/LICENSES/Apache-2.0.txt"
```

**作用**：
- 符合 OpenHarmony 许可证管理要求
- 自动化许可证检查

## 3.6 依赖关系

### 内部依赖

| 依赖类型 | 依赖项 | 说明 |
|---------|--------|------|
| **系统依赖** | ohos.gni | OH 构建系统模板 |
| **构建依赖** | build/ohos.gni | 构建配置模板 |

### 被依赖情况

| 依赖模块 | 依赖类型 | 使用方式 |
|---------|---------|---------|
| interface/sdk_c/graphic/graphic_2d | 头文件引用 | 直接引用 vulkan 头文件 |
| third_party/skia | public_deps | 传递 vulkan_headers 配置 |
| test/xts/acts/graphic/graphicvulkannapitest | deps | 静态链接 |

## 3.7 构建验证

### 编译命令

```bash
# 编译 vulkan-headers
hb build //third_party/vulkan-headers:vulkan_headers

# 验证头文件
hdc hilog | grep -E "vulkan|VK_OHOS"
```

### 验证项

| 验证项 | 期望结果 |
|--------|---------|
| **编译成功** | 无编译错误 |
| **头文件可达** | include 路径正确 |
| **OH 宏定义** | `VK_USE_PLATFORM_OHOS` 已定义 |
| **OH 扩展可用** | `vulkan_ohos.h` 可被 include |

### 常见问题

#### 问题 1：头文件找不到

**症状**：
```
fatal error: 'vulkan/vulkan.h' file not found
```

**解决方案**：
```gn
# 确保依赖配置正确
deps = [
  "//third_party/vulkan-headers:vulkan_headers",
]
```

#### 问题 2：OH 扩展未定义

**症状**：
```
error: 'VK_OHOS_surface' undeclared
```

**解决方案**：
```gn
# 确保包含 vulkan_ohos.h
#include <vulkan/vulkan_ohos.h>

# 确保编译时定义了 VK_USE_PLATFORM_OHOS
```
