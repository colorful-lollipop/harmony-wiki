# OpenHarmony 构建适配

## 一、构建系统概述

### 1.1 支持的构建系统

Unity 框架原生支持多种构建系统，在 OpenHarmony 集成中主要使用以下几种：

| 构建系统 | 文件位置 | OpenHarmony 支持状态 |
|----------|----------|---------------------|
| CMake | `CMakeLists.txt` | ✅ 完全支持 |
| Meson | `meson.build` | ✅ 完全支持 |
| Make | `Makefile` | ⚠️ 未包含，需手动创建 |
| PlatformIO | `platformio-build.py` | ⚠️ 嵌入式专用 |

**CMake** 是 Unity 推荐的构建系统，也是 OpenHarmony 项目中使用最广泛的构建工具。通过 CMakeLists.txt 配置文件，可以将 Unity 集成到任何支持 CMake 的项目中。

**Meson** 是近年来流行的现代化构建系统，以速度快和易用性好著称。Unity 提供了完整的 meson.build 配置，支持通过 Meson 选项自定义构建行为。

### 1.2 构建文件结构

Unity 库的构建相关文件结构如下：

```
unity/
├── CMakeLists.txt          # CMake 主配置
├── meson.build             # Meson 构建配置
├── meson_options.txt       # Meson 构建选项
├── platformio-build.py     # PlatformIO 构建脚本
├── unityConfig.cmake       # CMake 包配置文件
└── CMakeListsunity.txt    # CMake 子项目配置
```

## 二、CMake 构建配置详解

### 2.1 CMakeLists.txt 结构

Unity 的 CMakeLists.txt 文件结构清晰，主要包含以下部分：

```cmake
# CMakeLists.txt 主要配置

# 1. 项目元信息
cmake_minimum_required(VERSION 3.0)
project(Unity C)

# 2. 构建选项
option(UNITY_SUPPORT_64 "Support 64-bit integers" OFF)
option(UNITY_INCLUDE_EXEC_TIME "Include execution time" OFF)

# 3. 源文件配置
set(UNITY_SOURCES
    ${UNITY_DIR}/src/unity.c
    CACHE INTERNAL ""
)

# 4. 头文件配置
set(UNITY_INCLUDE_DIRS
    ${UNITY_DIR}/src
    CACHE INTERNAL ""
)

# 5. 导出配置
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    unityConfigVersion.cmake
    VERSION ${UNITY_VERSION}
    COMPATIBILITY SameMajorVersion
)
```

### 2.2 关键编译选项

Unity 通过 CMake 选项提供构建定制能力，以下是常用选项：

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `UNITY_SUPPORT_64` | BOOL | OFF | 启用 64 位整数支持 |
| `UNITY_INCLUDE_EXEC_TIME` | BOOL | OFF | 包含执行时间统计 |
| `UNITY_USE_PARAM_TEST` | BOOL | OFF | 启用参数化测试支持 |
| `UNITY_OUTPUT_CHAR` | STRING | "" | 自定义字符输出函数 |
| `UNITY_OUTPUT_FLUSH` | STRING | "" | 自定义输出刷新函数 |

### 2.3 在 OpenHarmony 项目中使用 CMake 集成

在 OpenHarmony 的 CMake 项目中集成 Unity，有以下几种方式：

**方式一：作为子目录**

```cmake
add_subdirectory(third_party/unity)

# 链接 Unity 库
target_link_libraries(my_test PRIVATE unity)
```

**方式二：直接包含源文件**

```cmake
# 添加 Unity 源文件
set(UNITY_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/third_party/unity/src/unity.c
)

# 添加头文件路径
target_include_directories(my_test PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/third_party/unity/src
)

# 编译目标
add_executable(my_test
    ${UNITY_SOURCES}
    test/test_my_module.c
)
```

**方式三：使用 Unity 提供的 CMake 脚本**

```cmake
include(third_party/unity/unityConfig.cmake)

# 或者手动配置
set(UNITY_DIR third_party/unity)
include(${UNITY_DIR}/CMakeListsunity.txt)
```

## 三、Meson 构建配置详解

### 3.1 meson.build 结构

Unity 的 meson.build 文件采用模块化设计：

```meson
# meson.build 主要配置

project('unity', 'c',
    version : '2.6.0',
    default_options : [
        'c_std=c99',
        'warning_level=1'
    ])

# 1. 配置选项
unity_support_64 = get_option('unity_support_64')
unity_include_exec_time = get_option('unity_include_exec_time')

# 2. 源文件
unity_sources = files('src/unity.c')

# 3. 头文件
unity_headers = files(
    'src/unity.h',
    'src/unity_int_types.h',
    'src/unity_build.h',
    'src/unity_internals.h'
)

# 4. 库构建
unity_lib = static_library('unity',
    unity_sources,
    include_directories : ['src'],
    c_args : unity_c_args
)
```

### 3.2 meson_options.txt 配置

```meson
option('unity_support_64',
    type : 'boolean',
    value : false,
    description : 'Enable 64-bit integer support'
)

option('unity_include_exec_time',
    type : 'boolean',
    value : false,
    description : 'Include execution time in test results'
)
```

### 3.3 在 OpenHarmony 中使用 Meson

OpenHarmony 的某些模块使用 Meson 作为构建系统，集成 Unity 的方式如下：

```meson
unity_dep = subproject('unity')

test_sources = [
    'test/test_example.c',
    'test/test_utils.c',
]

executable('my_unity_tests',
    test_sources,
    include_directories : ['third_party/unity/src'],
    dependencies : [unity_dep]
)
```

## 四、BUILD.gn 集成（推荐配置）

### 4.1 GN 构建配置说明

虽然 Unity 当前未提供 `BUILD.gn` 文件，但为了更好地集成到 OpenHarmony 的 GN 构建系统，建议添加以下配置。以下配置可作为参考实现：

```gn
# BUILD.gn 参考配置（可添加到 unity 目录）

config("unity_config") {
  include_dirs = [ "src" ]
  defines = []
  
  if (ohos_kernel_type == "liteos_m") {
    defines += [ "UNITY_SUPPORT_64" ]
  }
}

static_library("unity") {
  sources = [
    "src/unity.c",
  ]
  
  public_configs = [ ":unity_config" ]
  
  public_deps = []
}

# 单元测试模板
template("ohos_unittest") {
  unittest_target = target_name + "_target"
  
  static_library(unittest_target) {
    sources = invoker.sources
    include_dirs = invoker.include_dirs
    configs = [ "//third_party/unity:unity_config" ]
    
    deps = [
      "//third_party/unity:unity",
    ]
  }
}
```

### 4.2 使用示例

在 OpenHarmony 的模块中集成 Unity 进行单元测试：

```gn
# foundation/xxx/module/BUILD.gn

ohos_unittest("xxx_unittest") {
  sources = [
    "test/test_xxx.c",
    "test/test_xxx_utils.c",
  ]
  
  include_dirs = [
    ".",
    "//third_party/unity/src",
  ]
}
```

## 五、编译选项详解

### 5.1 预处理器宏

Unity 的行为可以通过预处理器宏进行定制：

| 宏名称 | 说明 | 推荐场景 |
|--------|------|----------|
| `UNITY_INT_WIDTH` | 指定整数宽度 | 特定硬件平台 |
| `UNITY_LONG_WIDTH` | 指定长整数宽度 | 64 位平台适配 |
| `UNITY_FLOAT_TYPE` | 指定浮点类型 | 特殊浮点硬件 |
| `UNITY_DOUBLE_TYPE` | 指定双精度类型 | 特殊浮点硬件 |
| `UNITY_EXCLUDE_FLOAT` | 排除浮点支持 | 极致资源受限环境 |
| `UNITY_OUTPUT_CHAR` | 自定义字符输出 | 无标准输出环境 |
| `UNITY_OUTPUT_FLUSH` | 自定义输出刷新 | 实时系统 |

### 5.2 编译器标志

在 OpenHarmony 编译 Unity 时，常用的编译器标志配置：

```makefile
# 优化级别
CFLAGS += -O0  # 调试时使用

# 警告处理
CFLAGS += -Wall -Wextra -Werror

# 标准版本
CFLAGS += -std=c99

# 平台特定
ifeq ($(TARGET_ARCH), arm)
    CFLAGS += -mcpu=cortex-m4
endif
```

### 5.3 构建产物

Unity 编译后生成的主要产物：

| 产物 | 说明 | 用途 |
|------|------|------|
| `libunity.a` | 静态库文件 | 链接到测试程序 |
| `unity.h` | 主头文件 | 测试代码包含 |
| `unity_int_types.h` | 整数类型头文件 | 跨平台类型定义 |

## 六、常见问题

### Q1：如何在裸机环境使用 Unity？

在裸机环境使用 Unity，需要自定义输出函数：

```c
#define UNITY_OUTPUT_CHAR(c) uart_send(c)
#define UNITY_OUTPUT_FLUSH() uart_flush()
```

### Q2：如何减小 Unity 的代码体积？

可以通过排除不需要的功能来减小体积：

```c
#define UNITY_EXCLUDE_FLOAT      // 排除浮点支持
#define UNITY_EXCLUDE_DOUBLE     // 排除双精度支持
#define UNITY_EXCLUDE_MEMORY     // 排除内存比较
```

### Q3：如何处理测试结果的 JSON 输出？

Unity 本身不提供 JSON 输出，可以通过以下方式实现：

1. 重定义 `UNITY_OUTPUT_CHAR` 收集输出
2. 使用第三方工具（如 Unity Fixtures）处理输出
3. 自定义测试运行器解析标准输出
