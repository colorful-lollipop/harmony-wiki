# 构建适配

## 3.1 构建系统概述

### 3.1.1 构建系统选择

OpenHarmony 使用 GN（Generate Ninja）作为其构建系统。OpenCL-Headers 作为第三方组件，需要适配 GN 构建体系。

**构建系统对比**：

| 方面 | 上游（CMake） | OpenHarmony（GN） |
|------|---------------|-------------------|
| 构建配置格式 | CMakeLists.txt | BUILD.gn |
| 构建目标 | install headers | shared library + headers |
| 输出产物 | 头文件安装包 | libopencl_wrapper.so + 头文件 |
| 平台支持 | 通用 | 专门针对 OH 平台优化 |

### 3.1.2 构建目标概览

OpenCL-Headers 在 OpenHarmony 中定义了两个主要构建目标：

| 目标名称 | 类型 | 输出 | 主要功能 |
|----------|------|------|----------|
| `libcl` | ohos_shared_library | libopencl_wrapper.so | 动态加载包装器共享库 |
| `opencl_headers` | source_set | 无（头文件集合） | 头文件导出 |

## 3.2 BUILD.gn 配置详解

### 3.2.1 完整 BUILD.gn 文件

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

config("cl_config") {
  cflags = [
    "-std=c++17",
    "-Wno-error=implicit-fallthrough",
    "-Wno-deprecated-declarations",
  ]
}

config("cl_public_config") {
  include_dirs = [
    "opencl-headers-CL",
    "include",
  ]
}

ohos_shared_library("libcl") {
  visibility = [ "*" ]
  sources = [ "src/opencl_wrapper.cpp" ]
  configs = [ ":cl_config" ]
  public_configs = [ ":cl_public_config" ]
  output_name = "opencl_wrapper"
  output_extension = "so"
  innerapi_tags = [ "platformsdk_indirect" ]
  part_name = "opencl-headers"
  subsystem_name = "thirdparty"
}

config("opencl_headers_public_config") {
  include_dirs = [
    "opencl-headers-CL",
    "include",
  ]
}

source_set("opencl_headers") {
  sources = [
    "CL/cl.h",
    "CL/cl_d3d10.h",
    "CL/cl_d3d11.h",
    "CL/cl_dx9_media_sharing.h",
    "CL/cl_dx9_media_sharing_intel.h",
    "CL/cl_egl.h",
    "CL/cl_ext.h",
    "CL/cl_ext_intel.h",
    "CL/cl_function_types.h",
    "CL/cl_gl.h",
    "CL/cl_gl_ext.h",
    "CL/cl_half.h",
    "CL/cl_icd.h",
    "CL/cl_layer.h",
    "CL/cl_platform.h",
    "CL/cl_va_api_media_sharing_intel.h",
    "CL/cl_version.h",
    "CL/opencl.h",
  ]
  public_configs = [ ":opencl_headers_public_config" ]
}

group("cl_tests") {
  deps = [ "tests:tests" ]
}
```

### 3.2.2 编译配置说明

**通用配置（cl_config）**：

```gn
config("cl_config") {
  cflags = [
    "-std=c++17",                          # 使用 C++17 标准
    "-Wno-error=implicit-fallthrough",    # 忽略隐式 fallthrough 警告
    "-Wno-deprecated-declarations",       # 忽略废弃声明警告
  ]
}
```

**配置项详解**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cflags` | 编译器标志数组 | 应用到 `libcl` 目标的 C 编译选项 |
| `-std=c++17` | 强制使用 C++17 | 使用现代 C++ 标准，支持更广泛的语言特性 |
| `-Wno-error=implicit-fallthrough` | 非错误警告 | 避免因 switch-case 隐式 fallthrough 导致的编译失败 |
| `-Wno-deprecated-declarations` | 非错误警告 | 允许使用标记为 deprecated 的 API |

**公共配置（cl_public_config）**：

```gn
config("cl_public_config") {
  include_dirs = [
    "opencl-headers-CL",
    "include",
  ]
}
```

**配置项详解**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `include_dirs` | 头文件搜索路径 | 导出给使用者的公共头文件路径 |

### 3.2.3 共享库目标配置

```gn
ohos_shared_library("libcl") {
  visibility = [ "*" ]
  sources = [ "src/opencl_wrapper.cpp" ]
  configs = [ ":cl_config" ]
  public_configs = [ ":cl_public_config" ]
  output_name = "opencl_wrapper"
  output_extension = "so"
  innerapi_tags = [ "platformsdk_indirect" ]
  part_name = "opencl-headers"
  subsystem_name = "thirdparty"
}
```

**配置项详解**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `visibility` | `[ "*" ]` | 允许所有模块访问该目标 |
| `sources` | 源文件列表 | 编译所需的源文件 |
| `configs` | 配置列表 | 私有配置（编译选项） |
| `public_configs` | 配置列表 | 导出给使用者的配置（头文件路径等） |
| `output_name` | `opencl_wrapper` | 输出库名称 |
| `output_extension` | `so` | 输出格式（共享库） |
| `innerapi_tags` | `["platformsdk_indirect"]` | 内部 API 标签，表示间接平台 SDK 依赖 |
| `part_name` | `opencl-headers` | 所属部件名称 |
| `subsystem_name` | `thirdparty` | 所属子系统名称 |

### 3.2.4 头文件目标配置

```gn
source_set("opencl_headers") {
  sources = [
    "CL/cl.h",
    "CL/cl_d3d10.h",
    # ... 更多头文件
  ]
  public_configs = [ ":opencl_headers_public_config" ]
}
```

**配置项详解**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | 头文件列表 | 所有需要导出的 OpenCL 头文件 |
| `public_configs` | 配置列表 | 包含头文件搜索路径配置 |

**头文件清单**：

| 头文件 | 功能 |
|--------|------|
| `CL/cl.h` | OpenCL 主头文件 |
| `CL/cl_version.h` | 版本宏定义 |
| `CL/cl_platform.h` | 平台类型定义 |
| `CL/cl_ext.h` | 通用扩展 |
| `CL/cl_ext_intel.h` | Intel 扩展 |
| `CL/cl_d3d10.h` | Direct3D 10 互操作 |
| `CL/cl_d3d11.h` | Direct3D 11 互操作 |
| `CL/cl_dx9_media_sharing.h` | DirectX 9 媒体共享 |
| `CL/cl_dx9_media_sharing_intel.h` | Intel DirectX 9 媒体共享 |
| `CL/cl_egl.h` | EGL 互操作 |
| `CL/cl_gl.h` | OpenGL 互操作 |
| `CL/cl_gl_ext.h` | OpenGL 扩展 |
| `CL/cl_half.h` | 半精度浮点 |
| `CL/cl_icd.h` | ICD 支持 |
| `CL/cl_layer.h` | Layer 支持 |
| `CL/cl_function_types.h` | 函数类型定义 |
| `CL/cl_va_api_media_sharing_intel.h` | Intel VA-API 媒体共享 |

## 3.3 头文件路径配置

### 3.3.1 包含路径结构

```text
opencl-headers/
├── CL/                                    # 上游原始头文件
│   ├── cl.h
│   └── ...
├── opencl-headers-CL/                    # OH 适配头文件目录
│   └── CL/                                # 结构与原始 CL/ 一致
│       ├── cl.h
│       └── ...
├── include/                               # OH 特有头文件
│   └── opencl_wrapper.h
└── BUILD.gn                               # 构建配置
```

### 3.3.2 公共包含路径

**配置位置**：`cl_public_config` 和 `opencl_headers_public_config`

```gn
config("cl_public_config") {
  include_dirs = [
    "opencl-headers-CL",  # OH 适配后的头文件
    "include",            # OH 特有包装器头文件
  ]
}
```

**路径说明**：

| 路径 | 内容 | 使用场景 |
|------|------|----------|
| `opencl-headers-CL/` | OpenCL 标准头文件 | 编译时包含 `<CL/opencl.h>` |
| `include/` | OH 包装器头文件 | 编译时包含 `<opencl_wrapper.h>` |

### 3.3.3 包含方式

**方式一：标准 OpenCL API**：

```cpp
// 使用标准 OpenCL API（需要链接 opencl_wrapper.so）
#include <CL/opencl.h>
```

**方式二：OH 包装器接口**：

```cpp
// 使用 OH 动态加载包装器
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>
```

## 3.4 与上游构建的差异

### 3.4.1 构建产物差异

| 方面 | 上游 CMake | OpenHarmony GN |
|------|------------|----------------|
| 主要产物 | 头文件安装包 | 共享库 + 头文件 |
| 头文件安装 | make install | 通过 inner_kits 导出 |
| 库文件 | 无 | libopencl_wrapper.so |
| 测试产物 | 可执行测试 | 测试目标 cl_tests |

### 3.4.2 构建配置差异

| 配置项 | 上游 | OpenHarmony |
|--------|------|-------------|
| 语言标准 | C99/C++11 | C++17 |
| 编译器标志 | 通用 | OH 特定警告抑制 |
| 依赖管理 | 无 | 与构建系统集成 |
| 平台支持 | 运行时检测 | 编译时配置 |

### 3.4.3 代码适配差异

**上游代码**：

```cpp
// 上游仅提供头文件，无实现
#include <CL/opencl.h>
```

**OH 适配**：

```cpp
// 新增动态加载包装器
_WRAPPER
#include#define USE_OPENCL <CL/opencl.h>
#include <opencl 使用包装_wrapper.h>

//器初始化
bool success = OHOS::InitOpenCL();
```

## 3.5 集成指南

### 3.5.1 基础集成

**步骤一：添加依赖**：

在目标模块的 BUILD.gn 中添加依赖：

```gn
ohos_executable("my_app") {
  deps = [
    "//third_party/opencl-headers:libcl",
  ]
}
```

**步骤二：包含头文件**：

```cpp
#include <CL/opencl.h>        // 标准 OpenCL API
// 或
#include <opencl_wrapper.h>   // OH 包装器
```

**步骤三：链接库**：

```gn
ohos_executable("my_app") {
  deps = [
    "//third_party/opencl-headers:libcl",
  ]
}
```

### 3.5.2 完整集成示例

**BUILD.gn 配置**：

```gn
import("//build/ohos.gni")

ohos_executable("opencl_sample") {
  sources = [
    "main.cpp",
  ]

  deps = [
    "//third_party/opencl-headers:libcl",
  ]

  cflags = [
    "-Wall",
    "-Wextra",
  ]
}
```

**源代码**：

```cpp
#include <CL/opencl.h>
#include <iostream>

int main() {
    // 初始化 OpenCL（通过 OH 包装器）
    // 包装器会在首次调用时自动初始化

    cl_uint platformCount = 0;
    clGetPlatformIDs(0, nullptr, &platformCount);

    if (platformCount == 0) {
        std::cout << "No OpenCL platform found" << std::endl;
        return -1;
    }

    std::cout << "Found " << platformCount << " OpenCL platform(s)" << std::endl;

    // 继续 OpenCL 操作...
    return 0;
}
```

### 3.5.3 高级集成

**使用 OH 特有功能**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>

int main() {
    // 手动初始化 OpenCL
    if (!OHOS::InitOpenCL()) {
        std::cerr << "Failed to initialize OpenCL" << std::endl;
        return -1;
    }

    // 使用 OpenCL API
    cl_platform_id platform;
    clGetPlatformIDs(1, &platform, nullptr);

    // 清理
    return 0;
}
```

**外部库加载**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>

int main() {
    void* handle = nullptr;
    
    // 外部加载 OpenCL 库
    if (!OHOS::InitOpenCLExtern(&handle)) {
        std::cerr << "Failed to load OpenCL library" << std::endl;
        return -1;
    }

    // 使用 OpenCL API...

    // 外部卸载
    OHOS::UnLoadCLExtern(handle);
    
    return 0;
}
```

## 3.6 常见构建问题

### 3.6.1 头文件找不到

**错误信息**：

```
fatal error: 'CL/opencl.h' file not found
```

**解决方案**：

确认 deps 中包含 `opencl-headers`：

```gn
ohos_executable("my_app") {
  deps = [
    "//third_party/opencl-headers:opencl_headers",
    "//third_party/opencl-headers:libcl",
  ]
}
```

### 3.6.2 链接错误

**错误信息**：

```
ld: cannot find -lopencl_wrapper
```

**解决方案**：

确保链接了正确的目标：

```gn
ohos_executable("my_app") {
  deps = [
    "//third_party/opencl-headers:libcl",
  ]
}
```

### 3.6.3 版本兼容问题

**问题**：上游 API 变更导致编译错误

**解决方案**：

1. 检查 `CL_TARGET_OPENCL_VERSION` 定义
2. 确保使用兼容的 API 集合

```cpp
#define CL_TARGET_OPENCL_VERSION 120
#include <CL/opencl.h>
```

## 3.7 构建产物说明

### 3.7.1 输出文件

| 文件 | 类型 | 用途 |
|------|------|------|
| `libopencl_wrapper.so` | 共享库 | OpenCL 动态加载包装器 |
| 头文件目录 | 头文件 | OpenCL API 定义 |

### 3.7.2 库依赖关系

```
libopencl_wrapper.so
    │
    ├── libc++.so           # C++ 标准库
    ├── libdl.so            # 动态加载库
    └── libOpenCL.so (运行时) # OpenCL 驱动（ICD）
```

---

*本章节详细说明了 OpenCL-Headers 在 OpenHarmony 中的构建适配配置，包括 BUILD.gn 的各项配置项说明和集成指南。*
