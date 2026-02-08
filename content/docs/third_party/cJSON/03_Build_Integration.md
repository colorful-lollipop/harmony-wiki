# OH 构建适配

## 概述

cJSON 库在 OpenHarmony 中的构建适配通过 `BUILD.gn` 文件完成。该文件针对 OH 生态系统进行了定制，支持两种构建模式。

## BUILD.gn 结构

```
third_party/cJSON/
├── BUILD.gn          # OH 构建配置
├── cJSON.c           # 核心源码
├── cJSON.h           # 头文件
├── cJSON_Utils.c     # 工具源码（lite 模式）
└── cJSON_Utils.h     # 工具头文件（lite 模式）
```

## 构建配置详解

### 两种构建模式

| 模式 | 条件 | 特点 |
|-----|------|------|
| **LiteOS 模式** | `defined(ohos_lite)` | 支持 cJSON_Utils，包含数学库链接 |
| **标准模式** | 其他 | 基础 cJSON，仅核心功能 |

### LiteOS 模式配置

```gn
if (defined(ohos_lite)) {
  # 编译配置
  config("cjson_config") {
    include_dirs = [ "//third_party/cJSON" ]  # 头文件搜索路径
    ldflags = [ "-lm" ]                       # 链接数学库
    defines = [ "CJSON_NESTING_LIMIT=(128)" ] # JSON 嵌套深度限制
  }

  # 源文件
  cjson_sources = [
    "cJSON.c",
    "cJSON_Utils.c",  # 包含工具库
  ]

  # IAR 编译器特殊处理
  if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
    cflags = [
      "--diag_suppress",
      "Pe513",
    ]
    cflags_cc = cflags
  }

  # 静态库
  static_library("cjson_static") {
    sources = cjson_sources
    public_configs = [ ":cjson_config" ]
  }

  # 动态库
  shared_library("cjson_shared") {
    sources = cjson_sources
    public_configs = [ ":cjson_config" }
  }
}
```

### 标准 OH 模式配置

```gn
else {
  import("//build/ohos.gni")

  # 编译配置
  config("cJSON_config") {
    include_dirs = [ "//third_party/cJSON" ]
    defines = [ "CJSON_NESTING_LIMIT=(128)" ]
  }

  # 静态库
  ohos_static_library("cjson_static") {
    branch_protector_ret = "pac_ret"  # PAC 栈保护
    sources = [ "cJSON.c" ]
    public_configs = [ ":cJSON_config" ]
    part_name = "cJSON"
    subsystem_name = "thirdparty"
  }

  # 动态库
  ohos_shared_library("cjson") {
    branch_protector_ret = "pac_ret"  # PAC 栈保护
    sources = [ "cJSON.c" ]
    public_configs = [ ":cJSON_config" ]
    innerapi_tags = [
      "chipsetsdk_sp",
      "platformsdk_indirect",
    ]
    part_name = "cJSON"
    subsystem_name = "thirdparty"
    install_images = [
      "system",     # 安装到 system 分区
      "updater",    # 安装到 updater 分区
    ]
  }
}
```

## 关键配置项说明

### 编译配置 (config)

| 配置项 | LiteOS | 标准 | 说明 |
|-------|--------|------|------|
| `include_dirs` | ✓ | ✓ | 头文件搜索路径 |
| `ldflags` | ✓ | ✗ | 链接数学库 `-lm` |
| `defines` | ✓ | ✓ | `CJSON_NESTING_LIMIT=128` |

### 目标配置 (static_library / shared_library)

| 配置项 | LiteOS | 标准 | 说明 |
|-------|--------|------|------|
| `sources` | cJSON.c + cJSON_Utils.c | 仅 cJSON.c | 源文件 |
| `public_configs` | ✓ | ✓ | 公开配置 |
| `branch_protector_ret` | ✗ | ✓ | PAC 栈保护 |
| `part_name` | ✗ | ✓ | 组件名 |
| `subsystem_name` | ✗ | ✓ | 子系统名 |
| `innerapi_tags` | ✗ | ✓ | API 标签 |
| `install_images` | ✗ | ✓ | 安装镜像 |

## 配置项详解

### CJSON_NESTING_LIMIT

```c
// cJSON.h 中的默认值
#ifndef CJSON_NESTING_LIMIT
#define CJSON_NESTING_LIMIT 1000
#endif
```

| 配置 | 值 | 目的 |
|-----|---|------|
| 上游默认 | 1000 | 支持深层嵌套 |
| OH 配置 | 128 | 减少内存占用，适合嵌入式 |

**影响**：
- 减少 JSON 解析时的最大栈使用量
- 防止深层嵌套导致的栈溢出
- 适合资源受限的嵌入式设备

### PAC 栈保护

```gn
branch_protector_ret = "pac_ret"
```

**目的**：启用指针认证码（PAC）栈保护，提高安全性。

**效果**：
- 防止 ROP（Return-Oriented Programming）攻击
- 检测栈上的返回地址篡改
- 需要硬件支持（ARMv8.3+）

### 内部 API 标签

```gn
innerapi_tags = [
  "chipsetsdk_sp",       # 芯片组 SDK
  "platformsdk_indirect", # 平台 SDK（间接）
]
```

**说明**：
- `chipsetsdk_sp`：标记为芯片组 SDK 的内部 API
- `platformsdk_indirect`：标记为平台 SDK 的间接依赖

### 安装镜像

```gn
install_images = [
  "system",
  "updater",
]
```

**目的**：将动态库安装到指定分区。

| 分区 | 说明 |
|-----|------|
| system | 系统主分区 |
| updater | 升级分区 |

## 与上游构建系统的差异

| 特性 | 上游（CMake） | OH（GN） |
|-----|--------------|----------|
| 构建系统 | CMake | GN |
| 嵌套限制 | 可配置（默认 1000） | 固定为 128 |
| cJSON_Utils | 可选编译 | lite 模式默认包含 |
| 栈保护 | 无 | PAC |
| 安装目标 | 系统目录 | system/updater |
| 版本检测 | CMake Config | 无 |

### CMake 选项对应关系

| CMake 选项 | GN 等效 | 说明 |
|-----------|---------|------|
| `-DENABLE_CJSON_TEST` | - | OH 不编译测试 |
| `-DENABLE_CJSON_UTILS` | lite 模式自动包含 | 工具库 |
| `-DENABLE_TARGET_EXPORT` | - | OH 使用 GN 导出 |
| `-DBUILD_SHARED_LIBS` | `ohos_shared_library` | 动态库 |
| `-DENABLE_LOCALES` | - | OH 未启用 |

## 编译选项说明

### IAR 编译器特殊处理

```gn
if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe513",
  ]
  cflags_cc = cflags
}
```

**说明**：
- `--diag_suppress Pe513`：抑制 IAR 编译器警告
- 针对嵌入式设备使用 IAR 编译器的特殊配置

## 使用方式

### 依赖声明

```gn
# 静态链接
deps = [ "//third_party/cJSON:cjson_static" ]

# 动态链接
deps = [ "//third_party/cJSON:cjson" ]

# 仅头文件
deps = [ "//third_party/cJSON/" ]
```

### 头文件引用

```c
#include <cjson/cJSON.h>
```

## 构建产物

### LiteOS 模式

| 产物 | 类型 | 文件 |
|-----|------|------|
| libcjson_static.a | 静态库 | `out/.../obj/third_party/cJSON/libcjson_static.a` |
| libcjson_shared.so | 动态库 | `out/.../obj/third_party/cJSON/libcjson_shared.so` |

### 标准模式

| 产物 | 类型 | 文件 |
|-----|------|------|
| libcjson_static.a | 静态库 | `out/.../obj/third_party/cJSON/libcjson_static.a` |
| libcjson.so | 动态库 | `out/.../obj/third_party/cJSON/libcjson.so` |

## 常见问题

### Q1: 如何启用 cJSON_Utils？

**回答**：cJSON_Utils 仅在 LiteOS 模式（`ohos_lite`）下自动包含。标准 OH 模式不包含此模块。

### Q2: 如何修改嵌套深度限制？

**回答**：在模块的 BUILD.gn 中添加：

```gn
config("my_config") {
  defines = [ "CJSON_NESTING_LIMIT=256" ]
}
```

### Q3: PAC 保护是否可选？

**回答**：PAC 保护在标准 OH 模式中**强制启用**，无法通过配置禁用。

## 版本兼容性

| OH 版本 | cJSON 版本 | 兼容性 |
|---------|-----------|-------|
| 4.0+ | 3.1 | ✓ |
| 3.2+ | 3.1 | ✓ |
| 3.1+ | 3.1 | ✓ |

## 总结

| 配置项 | OH 适配 | 说明 |
|-------|--------|------|
| 构建系统 | ✓ | GN 构建系统 |
| 嵌套限制 | ✓ | 128（上游 1000） |
| 栈保护 | ✓ | PAC |
| 安装配置 | ✓ | system/updater |
| 特殊编译器 | ✓ | IAR 支持 |
| Patch | ✗ | 无需 Patch |
