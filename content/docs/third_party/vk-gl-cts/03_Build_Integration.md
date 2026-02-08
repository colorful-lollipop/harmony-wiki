# VK-GL-CTS OpenHarmony 构建适配

## 构建系统概述

OpenHarmony 使用 GN (Generate Ninja) 构建系统，VK-GL-CTS 从原生的 CMake 完全迁移到了 GN 构建系统。

### 构建文件分布

| 文件/目录 | 类型 | 说明 |
|-----------|------|------|
| `BUILD.gn` | Group | 主入口，聚合所有模块 |
| `vk_gl_cts.gni` | Config | 公共编译配置 |
| `framework/**/BUILD.gn` | Library | 框架库构建定义 |
| `modules/**/BUILD.gn` | Library | 测试模块构建定义 |
| `external/**/BUILD.gn` | Library | 外部测试套件构建 |

## 根 BUILD.gn 分析

### 主入口定义

```gn
# Copyright (c) 2022 Shenzhen Kaihong Digital Industry Development Co., Ltd.
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")

# deqp build
group("deqp") {
  deps = [
    "//third_party/glslang:glslang",
    "//third_party/vk-gl-cts/framework/common:libdeqp_tcutil",
    "//third_party/vk-gl-cts/framework/delibs/debase:libdeqp_debase",
    "//third_party/vk-gl-cts/framework/delibs/decpp:libdeqp_decpp",
    "//third_party/vk-gl-cts/framework/delibs/depool:libdeqp_depool",
    "//third_party/vk-gl-cts/framework/delibs/destream:libdeqp_destream",
    "//third_party/vk-gl-cts/framework/delibs/dethread:libdeqp_dethread",
    "//third_party/vk-gl-cts/framework/delibs/deutil:libdeqp_deutil",
    "//third_party/vk-gl-cts/framework/egl/wrapper:libdeqp_eglwrapper",
    "//third_party/vk-gl-cts/framework/opengl:libdeqp_glutil",
    "//third_party/vk-gl-cts/framework/opengl/wrapper:libdeqp_glwrapper",
    "//third_party/vk-gl-cts/framework/qphelper:libdeqp_qphelper",
    "//third_party/vk-gl-cts/framework/randomshaders:libdeqp_randomshaders",
    "//third_party/vk-gl-cts/framework/referencerenderer:libdeqp_referencerenderer",
    "//third_party/vk-gl-cts/framework/xexml:libdeqp_xexml",
    "//third_party/vk-gl-cts/modules:deqp_modules",
  ]
}
```

### 关键依赖

- **glslang**: SPIR-V 着色器编译器
- **framework/delibs/**: dEQP 基础库
- **framework/opengl/**: OpenGL 工具库
- **framework/egl/**: EGL 工具库
- **modules/**: 测试模块

## vk_gl_cts.gni 核心配置

### 编译器标志

```gn
# C++ 编译选项
deqp_common_cflags_cc = [
  "-Wextra",                          # 启用额外警告
  "-Wno-long-long",                   # 忽略 long long 警告
  "-Wno-sign-conversion",             # 忽略符号转换警告
  "-std=c++17",                       # C++17 标准
  "-Wno-delete-non-virtual-dtor",     # 忽略非虚析构函数警告
  "-fwrapv",                          # 整数溢出环绕
  "-fexceptions",                     # 启用异常
  "-frtti",                           # 启用 RTTI
  "-mfloat-abi=softfp",               # 软浮点 ABI
  "-mfpu=neon-vfpv4",                 # ARM NEON 指令集
  "-Wno-header-hygiene",              # 忽略头文件卫生警告
  "-Wno-unused-command-line-argument", # 忽略未使用的命令行参数
  "-Wno-implicit-function-declaration", # 忽略隐式函数声明
  "-Wno-null-pointer-subtraction",    # 忽略空指针减法
  "-Wno-error=ignored-pragmas",       # pragma 忽略不报错
]

# C 编译选项
deqp_common_cflags = [
  "-Wextra",
  "-Wno-long-long",
  "-Wno-sign-conversion",
  "-std=c99",                         # C99 标准
  "-Wno-delete-non-virtual-dtor",
  "-fwrapv",
  "-fexceptions",
  "-mfloat-abi=softfp",
  "-mfpu=neon-vfpv4",
  "-Wno-unused-command-line-argument",
  "-Wno-implicit-function-declaration",
  "-Wno-null-pointer-subtraction",
]
```

### 预处理器定义

```gn
deqp_common_defines = [
  "CTS_USES_VULKAN",                  # 启用 Vulkan 支持
  "DEQP_SUPPORT_DRM=0",               # 禁用 DRM 支持
  "DEQP_TARGET_NAME=\"Default\"",     # 目标名称
  "DE_ASSERT_FAILURE_CALLBACK",       # 断言失败回调
  "DE_COMPILER=DE_COMPILER_CLANG",    # 使用 Clang 编译器
  "DE_DEBUG",                         # 调试模式
  "DE_MINGW=0",                       # 非 MinGW 环境
  "DE_OS=DE_OS_UNIX",                 # Unix 操作系统
]

# CPU 架构适配
if (target_cpu == "arm64") {
  deqp_common_defines += [
    "DE_PTR_SIZE=8",                  # 64 位指针
    "DE_CPU=DE_CPU_ARM_64",           # ARM64 架构
  ]
} else {
  deqp_common_defines += [
    "DE_PTR_SIZE=4",                  # 32 位指针
    "DE_CPU=DE_CPU_ARM",              # ARM 架构
  ]
}
```

### 包含路径

```gn
deqp_common_include_dirs = [
  "//third_party/vk-gl-cts/framework/opengl",
  "//third_party/vk-gl-cts/framework/opengl/wrapper",
  "//third_party/vk-gl-cts/framework/opengl/simplereference",
  "//third_party/vk-gl-cts/framework/randomshaders",
  "//third_party/vk-gl-cts/framework/common",
  "//third_party/vk-gl-cts/framework/xexml",
  "//third_party/vk-gl-cts/framework/qphelper",
  "//third_party/vk-gl-cts/framework/egl",
  "//third_party/vk-gl-cts/framework/egl/wrapper",
  "//third_party/vk-gl-cts/framework/referencerenderer",
  "//third_party/vk-gl-cts/framework/delibs/decpp",
  "//third_party/vk-gl-cts/framework/delibs/debase",
  "//third_party/vk-gl-cts/framework/delibs/deutil",
  "//third_party/vk-gl-cts/framework/delibs/dethread",
  "//third_party/vk-gl-cts/framework/delibs/depool",
  "//third_party/vk-gl-cts/framework/delibs/deimage",
  "//third_party/vk-gl-cts/framework/delibs/destream",
]
```

## 平台层构建配置

### framework/platform/BUILD.gn

这是最关键的 OHOS 适配构建文件：

```gn
# OHOS 平台特有配置
config("deqp_platform_ohos_config") {
  defines = [
    "DEQP_SUPPORT_GLES1=0",           # 禁用 GLES1
    "QP_SUPPORT_GLES1=0",
  ]
  
  cflags = [
    "-Wno-unused-variable",
    "-Wno-unused-function",
  ]
}

# OHOS 平台共享库
ohos_shared_library("libdeqp_ohos_platform") {
  sources = [
    "ohos/context/tcuOhosEglContextFactory.cpp",
    "ohos/context/tcuOhosNativeContext.cpp",
    "ohos/display/pixmap/tcuOhosNativePixmap.cpp",
    "ohos/display/pixmap/tcuOhosPixmapFactory.cpp",
    "ohos/display/tcuOhosEglDisplayFactory.cpp",
    "ohos/display/tcuOhosNativeDisplay.cpp",
    "ohos/display/window/tcuOhosNativeWindow.cpp",
    "ohos/display/window/tcuOhosWindowFactory.cpp",
    "ohos/tcuOhosPlatform.cpp",
  ]
  
  include_dirs = [
    "//third_party/vk-gl-cts/framework/platform/ohos",
    "//third_party/vk-gl-cts/framework/platform/ohos/display",
    "//third_party/vk-gl-cts/framework/platform/ohos/display/window",
    "//third_party/vk-gl-cts/framework/platform/ohos/display/pixmap",
    "//third_party/vk-gl-cts/framework/platform/ohos/context",
    "//third_party/vk-gl-cts/framework/platform/ohos/rosen_context",
  ]
  
  deps = [
    "//third_party/vk-gl-cts/framework/common:libdeqp_tcutil",
    "//third_party/vk-gl-cts/framework/opengl:libdeqp_glutil",
    "//third_party/vk-gl-cts/framework/opengl/wrapper:libdeqp_glwrapper",
    "//third_party/vk-gl-cts/framework/egl:libdeqp_eglutil",
    "//third_party/vk-gl-cts/framework/egl/wrapper:libdeqp_eglwrapper",
    "ohos/rosen_context:rosen_context",
  ]
  
  configs = [ ":deqp_platform_ohos_config" ]
}

# 可执行文件：glcts
ohos_executable("glcts") {
  sources = [ "ohos/testmain.cpp" ]
  
  deps = [
    ":libdeqp_ohos_platform",
    "ohos/rosen_context:rosen_context",
    # ... 其他依赖
  ]
  
  configs = [ ":deqp_platform_ohos_config" ]
}
```

### Rosen 上下文构建

```gn
# framework/platform/ohos/rosen_context/BUILD.gn

ohos_shared_library("rosen_context") {
  sources = [
    "ohos_context_i.cpp",
  ]
  
  include_dirs = [
    "//foundation/graphic/graphic_2d/rosen/modules/render_service_base/src/platform/ohos",
  ]
  
  deps = [
    "//foundation/graphic/graphic_2d/rosen/modules/render_service_base:librender_service_base",
  ]
}
```

## 与上游 CMake 的差异

### 构建系统对比

| 特性 | 上游 CMake | OpenHarmony GN |
|------|------------|----------------|
| **目标类型** | Executable/Library | ohos_executable/ohos_static_library/ohos_shared_library |
| **配置方式** | set() 变量 | gni 文件 import |
| **条件编译** | if() 语句 | GN 条件语句 |
| **平台检测** | CMAKE_SYSTEM_NAME | target_cpu / target_os |
| **输出目录** | CMAKE_BINARY_DIR | OHOS 标准输出目录 |

### 关键差异点

#### 1. 平台检测

**CMake (上游)**:
```cmake
if(ANDROID)
    add_definitions(-DDE_ANDROID=1)
elseif(WIN32)
    add_definitions(-DDE_MINGW=1)
elseif(UNIX)
    add_definitions(-DDE_OS_UNIX=1)
endif()
```

**GN (OpenHarmony)**:
```gn
if (target_cpu == "arm64") {
  defines += [ "DE_CPU=DE_CPU_ARM_64" ]
} else {
  defines += [ "DE_CPU=DE_CPU_ARM" ]
}

deqp_common_defines += [ "DE_OS=DE_OS_UNIX" ]
```

#### 2. 编译选项

**CMake (上游)**:
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
```

**GN (OpenHarmony)**:
```gn
deqp_common_cflags_cc = [ "-std=c++17" ]  # 升级到 C++17
```

#### 3. 库链接

**CMake (上游)**:
```cmake
target_link_libraries(deqp-vk 
    ${Vulkan_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
)
```

**GN (OpenHarmony)**:
```gn
deps = [
    "//third_party/vk-gl-cts/external/vulkancts/framework/vulkan:libdeqp_vkutil",
    "//foundation/graphic/graphic_2d/rosen/modules/render_service_base:librender_service_base",
]
```

## 特殊处理

### 1. 禁用 GLES1

OpenHarmony 不需要 GLES1 支持：

```gn
defines = [
  "DEQP_SUPPORT_GLES1=0",
  "QP_SUPPORT_GLES1=0",
]
```

### 2. 禁用 DRM

嵌入式场景不需要 DRM：

```gn
deqp_common_defines += [ "DEQP_SUPPORT_DRM=0" ]
```

### 3. ARM NEON 优化

启用 ARM NEON 指令集优化：

```gn
deqp_common_cflags_cc += [
  "-mfloat-abi=softfp",
  "-mfpu=neon-vfpv4",
]
```

### 4. Vulkan 生成代码

`build/external/vulkancts/framework/vulkan/` 目录包含大量生成代码：

```
这些文件由 Vulkan 代码生成工具生成，包含：
- vkBasicTypes.inl - 基本类型定义
- vkStructTypes.inl - 结构体定义
- vkFunctionPointerTypes.inl - 函数指针类型
- vkConcrete*Impl.inl - 接口实现
```

## 构建命令示例

### 构建整个 deqp

```bash
cd /path/to/openharmony
./build.sh --product <product_name> --target //third_party/vk-gl-cts:deqp
```

### 构建特定目标

```bash
# 构建平台库
./build.sh --target //third_party/vk-gl-cts/framework/platform:libdeqp_ohos_platform

# 构建 glcts 可执行文件
./build.sh --target //third_party/vk-gl-cts/framework/platform:glcts

# 构建 Vulkan 测试模块
./build.sh --target //third_party/vk-gl-cts/external/vulkancts/modules/vulkan:deqp_vk_execute
```

### 构建 XTS 图形测试

```bash
# 构建 gltest
./build.sh --target //test/xts/acts/graphic/gltest:gltest

# 构建 vktest
./build.sh --target //test/xts/acts/graphic/vktest:vktest
```

## 调试构建

### 开启详细输出

```bash
./build.sh --target //third_party/vk-gl-cts:deqp -v
```

### 查看编译命令

GN 生成 Ninja 文件后：

```bash
cd out/<product_name>
ninja -v //third_party/vk-gl-cts/framework/platform:libdeqp_ohos_platform
```

## 常见问题

### 问题 1: Rosen 头文件找不到

**现象**: 编译错误，找不到 `rosen_context_impl.h`

**解决**: 确保同步了 graphic_2d 子系统代码：
```bash
repo sync foundation/graphic/graphic_2d
```

### 问题 2: Vulkan 函数未定义

**现象**: 链接错误，Vulkan 函数未找到

**解决**: 检查是否包含了正确的生成代码目录：
```gn
include_dirs += [
  "//third_party/vk-gl-cts/build/external/vulkancts/framework/vulkan",
]
```

### 问题 3: NEON 指令编译失败

**现象**: ARM NEON 指令编译错误

**解决**: 检查编译器是否正确设置为 Clang：
```gn
# 应使用 Clang
deqp_common_defines += [ "DE_COMPILER=DE_COMPILER_CLANG" ]
```
