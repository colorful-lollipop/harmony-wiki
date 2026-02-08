# 03_Build_Integration.md - OH 构建适配详解

## 1. BUILD.gn 完整分析

### 1.1 文件位置

```
//third_party/astc-encoder/BUILD.gn
```

### 1.2 完整源码

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
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

config("astc_encoder_config") {
  include_dirs = [ "//third_party/astc-encoder/Source" ]
}

ohos_source_set("astc_encoder_static") {
  public_configs = [ ":astc_encoder_config" ]
  sources = [
    "//third_party/astc-encoder/Source/astcenc_averages_and_directions.cpp",
    "//third_party/astc-encoder/Source/astcenc_block_sizes.cpp",
    "//third_party/astc-encoder/Source/astcenc_color_quantize.cpp",
    "//third_party/astc-encoder/Source/astcenc_color_unquantize.cpp",
    "//third_party/astc-encoder/Source/astcenc_compress_symbolic.cpp",
    "//third_party/astc-encoder/Source/astcenc_compute_variance.cpp",
    "//third_party/astc-encoder/Source/astcenc_decompress_symbolic.cpp",
    "//third_party/astc-encoder/Source/astcenc_diagnostic_trace.cpp",
    "//third_party/astc-encoder/Source/astcenc_entry.cpp",
    "//third_party/astc-encoder/Source/astcenc_find_best_partitioning.cpp",
    "//third_party/astc-encoder/Source/astcenc_ideal_endpoints_and_weights.cpp",
    "//third_party/astc-encoder/Source/astcenc_image.cpp",
    "//third_party/astc-encoder/Source/astcenc_integer_sequence.cpp",
    "//third_party/astc-encoder/Source/astcenc_mathlib.cpp",
    "//third_party/astc-encoder/Source/astcenc_mathlib_softfloat.cpp",
    "//third_party/astc-encoder/Source/astcenc_partition_tables.cpp",
    "//third_party/astc-encoder/Source/astcenc_percentile_tables.cpp",
    "//third_party/astc-encoder/Source/astcenc_pick_best_endpoint_format.cpp",
    "//third_party/astc-encoder/Source/astcenc_quantization.cpp",
    "//third_party/astc-encoder/Source/astcenc_symbolic_physical.cpp",
    "//third_party/astc-encoder/Source/astcenc_weight_align.cpp",
    "//third_party/astc-encoder/Source/astcenc_weight_quant_xfer_tables.cpp",
  ]
  if (defined(global_parts_info) &&
      (defined(global_parts_info.graphic_graphic_2d_ext) ||
       defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
    defines = [ "ASTC_CUSTOMIZED_ENABLE" ]
    sources += [
      "//third_party/astc-encoder/Source/astcenccli_platform_dependents.cpp",
    ]
    if (target_cpu == "arm64" || is_emulator) {
      defines += [ "SUT_PATH_X64" ]
    }
  }
  if (defined(global_parts_info) &&
      defined(global_parts_info.product_hmos_sdk_product_hmos_sdk)) {
    defines += [ "BUILD_HMOS_SDK" ]
  }
  part_name = "astc-encoder"
  subsystem_name = "thirdparty"
}

ohos_shared_library("astc_encoder_shared") {
  public_configs = [ ":astc_encoder_config" ]
  deps = [ ":astc_encoder_static" ]
  install_enable = true
  part_name = "astc-encoder"
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "thirdparty"
  install_images = [ "system" ]
}
```

---

## 2. 构建目标结构

### 2.1 目标层次

```
BUILD.gn
├── config("astc_encoder_config")          # 公共配置
│   └── include_dirs
├── ohos_source_set("astc_encoder_static") # 静态源文件集
│   ├── 核心源文件（25 个）
│   ├── 条件源文件（1 个）
│   └── 条件宏定义
└── ohos_shared_library("astc_encoder_shared") # 共享库
    └── 依赖: astc_encoder_static
```

### 2.2 目标说明

| 目标名 | 类型 | 说明 |
|--------|------|------|
| `astc_encoder_config` | config | 公共头文件包含路径配置 |
| `astc_encoder_static` | source_set | 静态源文件集合（用于内部链接） |
| `astc_encoder_shared` | shared_library | 共享库目标（对外暴露） |

### 2.3 依赖关系

```
依赖者模块
    ↓
astc_encoder_shared (共享库)
    ↓
astc_encoder_static (静态源文件集)
    ↓
astc_encoder_config (包含路径配置)
```

---

## 3. 配置详解

### 3.1 公共配置（astc_encoder_config）

```gn
config("astc_encoder_config") {
  include_dirs = [ "//third_party/astc-encoder/Source" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `include_dirs` | `//third_party/astc-encoder/Source` | 头文件搜索路径 |

**作用**：
- 为依赖此库的模块提供头文件搜索路径
- 包含主要 API 头文件 `astcenc.h`

### 3.2 静态源文件集（astc_encoder_static）

#### 3.2.1 基础源文件（22 个核心文件）

| 序号 | 源文件 | 功能描述 |
|------|--------|---------|
| 1 | `astcenc_averages_and_directions.cpp` | 平均值和方向计算 |
| 2 | `astcenc_block_sizes.cpp` | ASTC 块大小处理 |
| 3 | `astcenc_color_quantize.cpp` | 颜色量化 |
| 4 | `astcenc_color_unquantize.cpp` | 颜色反量化 |
| 5 | `astcenc_compress_symbolic.cpp` | 符号压缩核心 |
| 6 | `astcenc_compute_variance.cpp` | 方差计算 |
| 7 | `astcenc_decompress_symbolic.cpp` | 符号解压核心 |
| 8 | `astcenc_diagnostic_trace.cpp` | 诊断追踪 |
| 9 | `astcenc_entry.cpp` | 入口点和上下文管理 |
| 10 | `astcenc_find_best_partitioning.cpp` | 最佳分区查找 |
| 11 | `astcenc_ideal_endpoints_and_weights.cpp` | 理想端点和权重计算 |
| 12 | `astcenc_image.cpp` | 图像数据结构 |
| 13 | `astcenc_integer_sequence.cpp` | 整数序列处理 |
| 14 | `astcenc_mathlib.cpp` | 数学库 |
| 15 | `astcenc_mathlib_softfloat.cpp` | 软浮点实现 |
| 16 | `astcenc_partition_tables.cpp` | 分区表 |
| 17 | `astcenc_percentile_tables.cpp` | 百分位表 |
| 18 | `astcenc_pick_best_endpoint_format.cpp` | 最佳端点格式选择 |
| 19 | `astcenc_quantization.cpp` | 量化处理 |
| 20 | `astcenc_symbolic_physical.cpp` | 符号到物理转换 |
| 21 | `astcenc_weight_align.cpp` | 权重对齐 |
| 22 | `astcenc_weight_quant_xfer_tables.cpp` | 权重量化传输表 |

#### 3.2.2 条件源文件

```gn
if (defined(global_parts_info) &&
    (defined(global_parts_info.graphic_graphic_2d_ext) ||
     defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
  sources += [
    "//third_party/astc-encoder/Source/astcenccli_platform_dependents.cpp",
  ]
}
```

| 条件 | 添加的文件 | 说明 |
|------|-----------|------|
| `graphic_graphic_2d_ext` 或 `product_hmos_sdk_product_hmos_sdk` 定义 | `astcenccli_platform_dependents.cpp` | 平台适配代码 |

**文件功能**：
- CPU 核心数查询（`get_cpu_count()`）
- 线程管理（`launch_threads()`）
- 高精度计时（`get_time()`）

### 3.3 条件编译定义（defines）

#### 3.3.1 ASTC_CUSTOMIZED_ENABLE

```gn
if (defined(global_parts_info) &&
    (defined(global_parts_info.graphic_graphic_2d_ext) ||
     defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
  defines = [ "ASTC_CUSTOMIZED_ENABLE" ]
}
```

| 触发条件 | 用途 |
|---------|------|
| 部件 `graphic_graphic_2d_ext` 存在 | 启用图形 2D 扩展支持 |
| 产品 `product_hmos_sdk_product_hmos_sdk` 存在 | 启用 HMOS SDK 支持 |

#### 3.3.2 SUT_PATH_X64

```gn
if (target_cpu == "arm64" || is_emulator) {
  defines += [ "SUT_PATH_X64" ]
}
```

| 触发条件 | 用途 |
|---------|------|
| 目标 CPU 为 ARM64 | ARM64 架构特定处理 |
| 或处于模拟器环境 | 模拟器特定处理 |

#### 3.3.3 BUILD_HMOS_SDK

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.product_hmos_sdk_product_hmos_sdk)) {
  defines += [ "BUILD_HMOS_SDK" ]
}
```

| 触发条件 | 用途 |
|---------|------|
| 产品 `product_hmos_sdk_product_hmos_sdk` 存在 | HMOS SDK 构建标记 |

### 3.4 组件元数据

```gn
part_name = "astc-encoder"
subsystem_name = "thirdparty"
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `part_name` | "astc-encoder" | OH 部件名称 |
| `subsystem_name` | "thirdparty" | 所属子系统 |

### 3.5 共享库配置（astc_encoder_shared）

```gn
ohos_shared_library("astc_encoder_shared") {
  public_configs = [ ":astc_encoder_config" ]
  deps = [ ":astc_encoder_static" ]
  install_enable = true
  part_name = "astc-encoder"
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "thirdparty"
  install_images = [ "system" ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `public_configs` | `:astc_encoder_config` | 公开的头文件路径配置 |
| `deps` | `:astc_encoder_static` | 依赖静态源文件集 |
| `install_enable` | true | 允许安装到系统 |
| `innerapi_tags` | ["platformsdk"] | 内部 API 标签，供平台 SDK 使用 |
| `install_images` | ["system"] | 安装到 system 镜像 |

---

## 4. 与上游构建系统对比

### 4.1 上游 CMake 构建

```cmake
# 上游 CMakeLists.txt 概览
cmake_minimum_required(VERSION 3.15)
project(astcenc)

# 支持多种 SIMD 变体
add_library(astcenc-sse2 STATIC ...)
add_library(astcenc-sse4.1 STATIC ...)
add_library(astcenc-avx2 STATIC ...)
add_library(astcenc-neon STATIC ...)

# CLI 可执行文件
add_executable(astcenc ...)
```

### 4.2 差异对比

| 项目 | 上游 CMake | OpenHarmony GN |
|------|-----------|----------------|
| **构建工具** | CMake 3.15+ | GN + Ninja |
| **目标类型** | 多 SIMD 变体静态库 + CLI 可执行文件 | 单一共享库 |
| **SIMD 处理** | 编译时多目标（sse2/sse4.1/avx2/neon） | 运行时或单一目标 |
| **输出格式** | 静态库 (.a) + 可执行文件 | 共享库 (.z.so) |
| **安装位置** | 系统标准路径 | `out/{product}/thirdparty/astc-encoder/` |
| **平台适配** | 通过 CMake 平台检测 | 通过 GN 条件编译 |

### 4.3 OH 构建优势

1. **单一目标**：简化了依赖管理，无需选择 SIMD 变体
2. **共享库**：减小可执行文件体积，便于系统更新
3. **条件编译**：根据产品配置灵活选择功能
4. **GN 集成**：与 OH 构建系统无缝集成

---

## 5. 编译输出

### 5.1 输出路径

```
out/{product}/thirdparty/astc-encoder/
├── libastc_encoder_shared.z.so          # 共享库
└── ...
```

### 5.2 输出文件说明

| 文件 | 类型 | 说明 |
|------|------|------|
| `libastc_encoder_shared.z.so` | 共享库 | OH 格式的共享库（压缩） |

### 5.3 安装位置

根据 `install_images = ["system"]`，库文件将安装到：
```
system/lib/libastc_encoder_shared.z.so
```

---

## 6. 在其他模块中使用

### 6.1 使用 deps 依赖（内部模块）

```gn
# 在 BUILD.gn 中
ohos_shared_library("my_module") {
  deps = [
    "//third_party/astc-encoder:astc_encoder_shared",
  ]
  # 自动继承 astc_encoder_config 的 include_dirs
}
```

### 6.2 使用 external_deps 依赖（推荐）

```gn
# 在 BUILD.gn 中
ohos_shared_library("my_module") {
  external_deps = [
    "astc-encoder:astc_encoder_shared",
  ]
  # 自动继承 astc_encoder_config 的 include_dirs
}
```

### 6.3 头文件引用

```cpp
// C++ 代码中
#include "astcenc.h"

// 使用 API
astcenc_config config;
astcenc_config_init(..., &config);
```

---

## 7. 构建配置调试

### 7.1 查看完整构建配置

```bash
# 生成构建配置（不实际编译）
./build.sh --product-name {product} --build-target //third_party/astc-encoder:astc_encoder_shared --export-flags

# 查看 GN 配置
./prebuilts/build-tools/linux-x64/bin/gn args out/{product}
```

### 7.2 验证条件编译

```bash
# 检查 global_parts_info 中是否包含相关部件
grep "graphic_graphic_2d_ext" out/{product}/build_configs/parts_info.json
grep "product_hmos_sdk_product_hmos_sdk" out/{product}/build_configs/parts_info.json
```

### 7.3 构建日志分析

```bash
# 详细构建日志
./build.sh --product-name {product} --build-target //third_party/astc-encoder:astc_encoder_shared --verbose

# 检查编译定义
# 在生成的 ninja 文件中搜索 defines
```

---

## 8. 常见问题

### 8.1 找不到头文件

**问题**：编译时提示 `astcenc.h: No such file or directory`

**解决**：确保正确依赖目标：
```gn
# 正确：依赖共享库目标
deps = [ "//third_party/astc-encoder:astc_encoder_shared" ]

# 错误：依赖配置目标
deps = [ "//third_party/astc-encoder:astc_encoder_config" ]
```

### 8.2 链接错误

**问题**：未定义的引用（undefined reference）

**可能原因**：
1. 依赖了配置目标而非库目标
2. 链接顺序问题

**解决**：
```gn
# 使用 public_deps 确保传递依赖
public_deps = [ "//third_party/astc-encoder:astc_encoder_shared" ]
```

### 8.3 条件编译未生效

**问题**：`ASTC_CUSTOMIZED_ENABLE` 等宏未定义

**检查**：
1. 确认产品配置包含 `graphic_graphic_2d_ext` 或 `product_hmos_sdk_product_hmos_sdk`
2. 检查 `global_parts_info` 是否正确传递

---

## 9. 总结

### 9.1 BUILD.gn 设计特点

1. **分层结构**：配置层 → 静态源文件层 → 共享库层
2. **条件编译**：根据产品配置灵活启用功能
3. **零 Patch**：通过 GN 配置完成所有适配

### 9.2 关键配置点

| 配置项 | 重要性 | 说明 |
|--------|--------|------|
| `include_dirs` | ⭐⭐⭐ | 头文件路径 |
| `sources` | ⭐⭐⭐ | 源文件列表 |
| `defines` | ⭐⭐ | 条件编译宏 |
| `innerapi_tags` | ⭐⭐ | API 可见性控制 |
| `install_images` | ⭐ | 安装目标分区 |

### 9.3 维护建议

1. **升级时**：验证源文件列表是否与上游一致
2. **调试时**：使用 `--verbose` 查看完整编译命令
3. **优化时**：考虑是否需要启用 SIMD 优化（当前 BUILD.gn 未显式配置）
