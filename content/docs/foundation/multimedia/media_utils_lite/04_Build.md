# 构建配置

## 构建系统概述

`media_utils_lite` 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统，通过 `BUILD.gn` 和 `config.gni` 文件进行配置。

### 构建环境要求

| 要求 | 版本/说明 |
|------|-----------|
| C++ 标准 | C++11 或更高 |
| 构建工具 | GN + Ninja |
| 构建命令 | `hb build media_service` |
| Python | 3.7+ (用于 GN) |

## BUILD.gn 详解

### 文件位置

`foundation/multimedia/media_utils_lite/BUILD.gn`

### 主要 Target: media_common

```gn
lite_library("media_common") {
  # 根据内核类型决定输出类型
  if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"    # LiteOS-M 内核使用静态库
  } else {
    target_type = "shared_library"   # 其他内核使用动态库
  }

  # 源文件列表
  sources = [
    "src/format.cpp",
    "src/source.cpp",
  ]

  # 头文件搜索路径
  include_dirs = [
    "interfaces/kits",
    "//base/hiviewdfx/hilog_lite/interfaces/native/innerkits/hilog",
  ]

  # 平台相关的安全库依赖
  if (ohos_kernel_type == "liteos_m") {
    public_deps = [ "//third_party/bounds_checking_function:libsec_static" ]
  } else {
    public_deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
  }

  # 内核特定的依赖
  if (ohos_kernel_type == "liteos_m") {
    public_deps += [ "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_static" ]
    public_configs = [ ":media_common_public_config" ]
  } else {
    public_deps += [ "//foundation/graphic/surface_lite:surface_lite" ]
    
    # 特定开发板的硬件适配依赖
    if (board_name == "hispark_taurus" || board_name == "hispark_aries") {
      public_deps += [
        "$ohos_board_adapter_dir/media:hardware_media_sdk",
        "$ohos_board_adapter_dir/middleware:middleware_source_sdk",
      ]
    }
  }
}

# 公共配置
config("media_common_public_config") {
  defines = [ "SURFACE_DISABLED" ]
}
```

**证据来源**: `BUILD.gn:16-53`

### Target 汇总表

| Target 名称 | 类型 | 输出名 | 用途 |
|-------------|------|--------|------|
| `media_common` | static/shared library | libmedia_common.a / libmedia_common.so | 核心库 |

## config.gni 详解

### 文件位置

`foundation/multimedia/media_utils_lite/config.gni`

### 配置项

```gn
declare_args() {
  # 媒体直通模式开关
  enable_media_passthrough_mode = false
}
```

**证据来源**: `config.gni:16-18`

## bundle.json 组件配置

### 文件位置

`foundation/multimedia/media_utils_lite/bundle.json`

### 组件定义

```json
{
  "name": "@ohos/media_utils_lite",
  "description": "Definition of public information such as media error code, and data types required for recording and playing audio and video.",
  "version": "3.2",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "foundation/multimedia/media_utils_lite"
  },
  "component": {
    "name": "media_utils_lite",
    "subsystem": "multimedia",
    "syscap": [],
    "features": [],
    "adapted_system_type": [
      "mini",
      "small"
    ],
    "rom": "1024kB",
    "ram": "500kB",
    "deps": {
      "components": [],
      "third_party": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "type": "none",
          "name": "//foundation/multimedia/media_utils_lite:media_common",
          "header": {
            "header_files": [
              "hal_camera.h",
              "hal_display.h",
              "data_stream.h",
              "format.h",
              "media_errors.h",
              "media_info.h",
              "media_log.h",
              "source.h"
            ],
            "header_base": [
              "//foundation/multimedia/media_utils_lite/hals",
              "//foundation/multimedia/media_utils_lite/interfaces/kits"
            ]
          }
        }
      ],
      "test": []
    }
  }
}
```

**证据来源**: `bundle.json:1-54`

### Inner Kit 导出清单

| 头文件 | 路径 | 导出类型 |
|--------|------|----------|
| `hal_camera.h` | `hals/` | HAL 接口 |
| `hal_display.h` | `hals/` | HAL 接口 |
| `data_stream.h` | `interfaces/kits/` | 数据流接口 |
| `format.h` | `interfaces/kits/` | 格式化数据 |
| `media_errors.h` | `interfaces/kits/` | 错误码 |
| `media_info.h` | `interfaces/kits/` | 媒体枚举 |
| `media_log.h` | `interfaces/kits/` | 日志宏 |
| `source.h` | `interfaces/kits/` | 媒体源 |

## 依赖关系分析

### 内部依赖

```
media_utils_lite (本组件)
├── interfaces/kits/  (头文件目录)
├── src/               (实现代码)
└── hals/              (HAL 接口)
```

### 外部依赖

| 依赖路径 | 用途 | 来源 |
|----------|------|------|
| `//third_party/bounds_checking_function:libsec_*` | 安全字符串函数 | third_party |
| `//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_*` | 日志框架 | hiviewdfx 子系统 |
| `//foundation/graphic/surface_lite:surface_lite` | 图形表面管理 | graphic 子系统 |
| `$ohos_board_adapter_dir/media:hardware_media_sdk` | 硬件媒体 SDK | 板级适配 |
| `$ohos_board_adapter_dir/middleware:middleware_source_sdk` | 中间件 SDK | 板级适配 |

## 编译产物

### 输出产物

| 产物 | 类型 | 位置 |
|------|------|------|
| `libmedia_common.a` | 静态库 (liteos_m) | `out/.../libs/` |
| `libmedia_common.so` | 动态库 (Linux) | `out/.../libs/` |

### 安装路径

产物将被安装到系统镜像的以下位置：

```
system/lib/                    # 动态库 (.so)
system/lib/ohos.elf/           # 符号链接
system/lib/module/             # 模块化产物
```

### 运行时加载

```cpp
// 静态库链接 (liteos_m)
# 在目标文件中直接包含所有符号

// 动态库加载 (Linux)
dlopen("libmedia_common.so", RTLD_LAZY);
```

## 编译命令

### 全量构建

```bash
# 设置开发板
hb set

# 编译媒体服务
hb build media_service
```

### 单仓构建

```bash
# 在仓库根目录下执行
hb build media_utils_lite
```

### 清理构建

```bash
hb build -c
```

## 条件编译

### SURFACE_DISABLED 宏

当在 liteos_m 内核或无 Surface 环境编译时，会定义 `SURFACE_DISABLED` 宏：

```cpp
// BUILD.gn 中配置
config("media_common_public_config") {
  defines = [ "SURFACE_DISABLED" ]
}

// 源代码中使用
#ifndef SURFACE_DISABLED
#include "surface.h"
#endif
```

### ohos_kernel_type 判断

```gn
# 根据内核类型选择不同的构建配置
if (ohos_kernel_type == "liteos_m") {
  target_type = "static_library"
  public_deps += [ "...:hilog_static" ]
  public_deps += [ "...:libsec_static" ]
} else {
  target_type = "shared_library"
  public_deps += [ "...:surface_lite" ]
  public_deps += [ "...:libsec_shared" ]
}
```

### board_name 判断

```gn
# 针对特定开发板的硬件适配
if (board_name == "hispark_taurus" || board_name == "hispark_aries") {
  public_deps += [
    "$ohos_board_adapter_dir/media:hardware_media_sdk",
    "$ohos_board_adapter_dir/middleware:middleware_source_sdk",
  ]
}
```

## 常见构建问题

### 问题 1: 找不到头文件

**现象**:
```
fatal error: 'xxx.h' file not found
```

**解决方案**:
```bash
# 确保在 OpenHarmony 根目录执行
source build.sh
hb set
hb build
```

### 问题 2: 依赖冲突

**现象**:
```
error: dependency cycle detected
```

**解决方案**:
检查 `bundle.json` 中的依赖配置，确保无循环依赖。

### 问题 3: HAL 接口未实现

**现象**:
```
undefined reference to 'HalCreateVideoProcessor'
```

**解决方案**:
确保板级适配层已实现相应的 HAL 接口。
