# GN 构建目标与编译配置

> **目的**: 理解 GN 构建系统、目标类型和编译配置  
> **适用范围**: 构建维护者、添加新 C API 的开发者  
> **生成时间**: 2025-02-06

---

## 1. GN 构建系统概述

### 1.1 什么是 GN

**GN**（Generate Ninja）是 OpenHarmony 使用的元构建系统：
- 使用 `.gn` 和 `.gni` 文件定义构建规则
- 生成 Ninja 构建文件
- 支持跨平台编译（Linux/macOS/Windows → OpenHarmony）

### 1.2 关键文件

| 文件 | 路径 | 作用 |
|------|------|------|
| `ndk_targets.gni` | 根目录 | NDK 构建目标列表（287+ 目标） |
| `BUILD.gn` | 各模块目录 | 模块构建配置 |
| `ndk.gni` | `//build/ohos/ndk/` | NDK 构建模板定义 |

---

## 2. NDK 构建目标类型

### 2.1 Target 类型统计

| Target 类型 | 数量 | 用途 |
|------------|------|------|
| `ohos_ndk_library` | 131 | 构建 NDK 共享库（.so） |
| `ohos_ndk_headers` | 136 | 安装头文件到 sysroot |
| `ohos_copy` | 6 | 文件复制操作 |
| `ohos_ndk_copy` | 2 | NDK 专用复制 |
| `ohos_ndk_toolchains` | 若干 | 工具链配置 |
| `group` | 若干 | 目标分组 |

### 2.2 ohos_ndk_library

**功能**: 构建 NDK 共享库

**参数说明**:
```gn
ohos_ndk_library("libexample_ndk") {
    output_name = "example"           # 输出库名（自动加 lib 前缀）
    output_extension = "so"           # 输出扩展名，默认 z.so
    ndk_description_file = "./example.ndk.json"  # API 符号定义
    min_compact_version = "12"        # 最小兼容版本
    system_capability = "SystemCapability.Example.Module"  # SysCap
    system_capability_headers = [     # 关联头文件
        "example/example.h",
    ]
}
```

### 2.3 ohos_ndk_headers

**功能**: 安装头文件到 SDK sysroot

**参数说明**:
```gn
ohos_ndk_headers("example_header") {
    dest_dir = "$ndk_headers_out_dir/example"  # 目标目录
    sources = [                                # 源文件
        "./include/example.h",
        "./include/example_types.h",
    ]
}
```

---

## 3. 典型 BUILD.gn 模式

### 3.1 标准双 Target 模式

```gn
# 标准双 target 模式（最常见）
import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")

# 1. 构建共享库
ohos_ndk_library("libexample_ndk") {
    output_name = "example"
    output_extension = "so"
    ndk_description_file = "./example.ndk.json"
    min_compact_version = "12"
    system_capability = "SystemCapability.Example.Module"
    system_capability_headers = [
        "example/example.h",
    ]
}

# 2. 安装头文件
ohos_ndk_headers("example_header") {
    dest_dir = "$ndk_headers_out_dir/example"
    sources = [
        "./include/example.h",
        "./include/example_types.h",
    ]
}
```

### 3.2 多 Library 单 Headers 模式

```gn
# security/huks/BUILD.gn
ohos_ndk_library("libhuks_ndk") { ... }
ohos_ndk_library("libhuks_external_crypto") { ... }
ohos_ndk_headers("huks_header") { ... }
```

### 3.3 复杂多 Target 模式

```gn
# multimedia/image_framework/BUILD.gn（21 个 target）
ohos_ndk_library("libpixelmap_ndk") { ... }
ohos_ndk_library("libpixelmap") { ... }
ohos_ndk_library("libpicture") { ... }
ohos_ndk_library("libimage_common") { ... }
# ... 更多 library 和 headers
```

---

## 4. ndk_targets.gni 分析

### 4.1 文件结构

```gni
# 1. NDK 库目标列表（287 个）
_ndk_library_targets = [
    "//interface/sdk_c/sensors/miscdevice/vibrator:lib_vibrator_ndk",
    "//interface/sdk_c/sensors/miscdevice/vibrator:ndk_vibrator_header",
    # ... 287+ 个目标
]

# 2. 基础库（musl libc）
_ndk_base_libs = [
    "//interface/sdk_c/third_party/musl/ndk_script/adapter:libc_ndk",
    # ...
]

# 3. Sysroot UAPI
_ndk_sysroot_uapi = [
    "//interface/sdk_c/third_party/musl/ndk_script:musl_sysroot",
]

# 4. CMake/Ninja 工具链
_ndk_cmake = [ ... ]
_ndk_ninja = [ ... ]

# 5. 所有目标汇总
all_ndk_targets_list = _ndk_library_targets + _ndk_base_libs + 
                       _ndk_sysroot_uapi + _ndk_cmake + _ndk_ninja
```

### 4.2 目标分类

| 类别 | 数量 | 示例 |
|------|------|------|
| **多媒体** | 60+ | audio_framework, camera, av_codec, image_framework |
| **图形** | 20+ | native_drawing, native_window, EGL, GLES, Vulkan |
| **安全** | 10+ | huks, access_token, asset, crypto |
| **网络** | 10+ | net_http, net_websocket, wifi, bluetooth |
| **ArkUI** | 15+ | napi, ace_engine, window_manager, display_manager |
| **基础服务** | 30+ | hilog, resource_management, fileio |

---

## 5. 添加新 C API 指南

### 5.1 步骤 1: 创建目录结构

```
interface_sdk_c/
└── your_module/
    ├── include/
    │   └── your_module/
    │       └── your_api.h
    ├── your_api.ndk.json
    └── BUILD.gn
```

### 5.2 步骤 2: 编写头文件

```c
// include/your_module/your_api.h
#ifndef YOUR_API_H
#define YOUR_API_H

#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

// API 函数声明
int32_t OH_YourModule_DoSomething(int32_t param);

#ifdef __cplusplus
}
#endif

#endif  // YOUR_API_H
```

### 5.3 步骤 3: 创建 .ndk.json

```json
[
    {
        "name": "OH_YourModule_DoSomething"
    }
]
```

### 5.4 步骤 4: 编写 BUILD.gn

```gn
import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")

ohos_ndk_library("libyourmodule_ndk") {
    output_name = "yourmodule"
    output_extension = "so"
    ndk_description_file = "./your_api.ndk.json"
    min_compact_version = "12"
    system_capability = "SystemCapability.Your.Module"
    system_capability_headers = [
        "your_module/your_api.h",
    ]
}

ohos_ndk_headers("yourmodule_header") {
    dest_dir = "$ndk_headers_out_dir/your_module"
    sources = [
        "./include/your_module/your_api.h",
    ]
}
```

### 5.5 步骤 5: 添加到 ndk_targets.gni

```gni
_ndk_library_targets = [
    # ... 现有目标
    "//interface/sdk_c/your_module:libyourmodule_ndk",
    "//interface/sdk_c/your_module:yourmodule_header",
]
```

---

## 6. .ndk.json 格式详解

### 6.1 基本格式

```json
[
    {
        "name": "FunctionName"
    },
    {
        "first_introduced": "12",
        "name": "NewFunction"
    },
    {
        "name": "GlobalVariable",
        "type": "variable"
    }
]
```

### 6.2 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 符号名称 |
| `type` | string | 否 | 符号类型（默认 function，可选 variable） |
| `first_introduced` | string | 否 | 首次引入版本 |

---

## 7. 代码证据

| 结论 | 证据文件 | 关键内容 |
|------|----------|----------|
| GN 模板定义 | `docs/howto_add.md` | 第 47-130 行 |
| 目标列表 | `ndk_targets.gni` | 第 17-287 行 |
| BUILD.gn 示例 | `arkui/napi/BUILD.gn` | 标准模式 |
| 复杂示例 | `multimedia/image_framework/BUILD.gn` | 多 target 模式 |

---

## 8. 相关跳转

- **上一章**: [内部 API](./04_Internal_API.md)
- **下一章**: [编译产物](./06_Build_Artifacts.md)
- **构建指南**: `docs/howto_add.md`
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**GN 构建文档 - 基于代码生成**
