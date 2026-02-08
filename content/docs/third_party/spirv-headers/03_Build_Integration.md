# OH 构建适配

## BUILD.gn 结构说明

```
//third_party/spirv-headers/BUILD.gn
```

该文件为 OpenHarmony 的 GN 构建系统提供适配支持。

### 配置文件定义

```gn
config("spv_headers_public_config") {
  include_dirs = [ "include" ]
}
```

定义了公开的包含目录配置，供依赖模块引用。

### 源文件集合

```gn
source_set("spv_headers") {
  sources = [
    "include/spirv/1.2/GLSL.std.450.h",
    "include/spirv/1.2/OpenCL.std.h",
    "include/spirv/1.2/spirv.h",
    "include/spirv/1.2/spirv.hpp",
    "include/spirv/unified1/GLSL.std.450.h",
    "include/spirv/unified1/NonSemanticClspvReflection.h",
    "include/spirv/unified1/NonSemanticDebugPrintf.h",
    "include/spirv/unified1/OpenCL.std.h",
    "include/spirv/unified1/spirv.h",
    "include/spirv/unified1/spirv.hpp",
  ]

  public_configs = [ ":spv_headers_public_config" ]
}
```

### 关键编译选项

| 选项 | 值 | 说明 |
|------|-----|------|
| `include_dirs` | `["include"]` | 头文件搜索根目录 |
| `sources` | 10 个头文件 | 参与构建的头文件列表 |
| `public_configs` | `:spv_headers_public_config` | 暴露给依赖者的配置 |

## 与上游构建系统的差异

### 上游构建系统

上游使用 CMake 构建：

```cmake
# 主要目标
add_library(spirv-headers INTERFACE)

# 头文件安装路径
install(DIRECTORY include/ DESTINATION include)
```

### OH 构建适配

OH 使用 GN 构建系统，适配方式如下：

| 方面 | 上游 CMake | OH GN |
|------|-----------|-------|
| 构建类型 | INTERFACE 库 | source_set |
| 头文件处理 | 目录级别引用 | 文件级别显式列出 |
| 配置传递 | target_include_directories | public_configs |

### 适配原因

1. **GN 构建要求**：GN 需要显式列出源文件，无法像 CMake INTERFACE 库那样仅声明目录
2. **依赖追踪**：显式列出文件便于 GN 进行增量编译和依赖追踪
3. **版本控制**：同时包含 1.2 版本和 unified1 统一版本的头文件

## 特殊处理

### 包含的版本

| 版本 | 目录 | 说明 |
|------|------|------|
| SPIR-V 1.2 | `include/spirv/1.2/` | 旧版本向后兼容 |
| 统一版本 | `include/spirv/unified1/` | 最新统一视图 |

### 未包含的版本

- `1.0` 和 `1.1` 版本的头文件未包含在 BUILD.gn 中
- 这些版本的头文件仍存在于目录中，但未参与 OH 构建
- 如需支持旧版本，可手动添加到 sources 列表

## 使用方式

### 直接依赖

```gn
spirv_headers = "//third_party/spirv-headers:spv_headers"
```

### 头文件引用

```cpp
#include "spirv/unified1/spirv.h"
#include "spirv/unified1/GLSL.std.450.h"
```

## 构建注意事项

1. **纯头文件库**：该库不产生 .o 目标文件，仅提供头文件和配置
2. **依赖传播**：通过 `public_configs` 传播 include_dirs 配置
3. **无需编译**：依赖该库的头文件即可使用，无需链接步骤
