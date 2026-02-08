# OH 构建适配

## 3.1 BUILD.gn 结构分析

### 3.1.1 原始 BUILD.gn 配置

本库的 BUILD.gn 位于 `third_party/openGLES/BUILD.gn`，配置极为简洁：

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
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

config("libGLES_public_config") {
  include_dirs = [ "api" ]
}

ohos_static_library("libGLES") {
  public_configs = [ ":libGLES_public_config" ]
}
```

### 3.1.2 配置详解

| 配置项 | 值 | 说明 |
|-------|-----|------|
| **import** | `//build/ohos.gni` | 导入 OH 构建系统 |
| **config 类型** | `config("libGLES_public_config")` | 公开配置，包含头文件路径 |
| **include_dirs** | `["api"]` | 暴露 `api/` 目录下的所有头文件 |
| **构建目标** | `ohos_static_library("libGLES")` | 静态库目标 |
| **public_configs** | `[""]` | 公开给依赖者的配置 |
| **依赖** | 无 | 无外部依赖 |

### 3.1.3 为什么使用 static_library

尽管本库不包含任何可编译的源文件（仅有头文件），仍使用 `ohos_static_library` 而非 `group` 或 `source_set`：

| 方案 | 优点 | 缺点 |
|------|------|------|
| **static_library** | 与 OH 构建系统兼容性好，支持 public_configs | 生成空的目标文件 |
| **source_set** | 轻量，无目标文件生成 | 不支持 configs 传递 |
| **group** | 最轻量 | 无法传递 public_configs |

OpenHarmony 选择 `static_library` 的原因：
1. **一致性**：与其他 third_party 库使用相同的构建模板
2. **public_configs 支持**：支持将配置传递给依赖者
3. **兼容性**：与 NDK 构建系统集成良好

## 3.2 关键编译选项

### 3.2.1 头文件包含路径

```gn
config("libGLES_public_config") {
  include_dirs = [ "api" ]
}
```

此配置确保所有依赖 `opengles:libGLES` 的模块都能找到 OpenGL ES 头文件：

| 路径 | 内容 |
|------|------|
| `api/GLES/gl.h` | OpenGL ES 1.x 核心头文件 |
| `api/GLES2/gl2.h` | OpenGL ES 2.x 核心头文件 |
| `api/GLES2/gl2ext.h` | OpenGL ES 2.x 扩展头文件 |
| `api/GLES3/gl3.h` | OpenGL ES 3.x 核心头文件 |
| `api/GLES3/gl32.h` | OpenGL ES 3.2 头文件 |
| `api/GL/gl.h` | 桌面 OpenGL 头文件 |

### 3.2.2 无其他编译选项

本库**不包含**以下常见编译选项：

| 选项类型 | 是否使用 | 原因 |
|---------|---------|------|
| **defines** | ❌ | 无预处理器宏定义需求 |
| **cflags** | ❌ | 无编译的 C/C++ 源文件 |
| **ldflags** | ❌ | 无链接需求 |
| **sources** | ❌ | 无源文件 |
| **deps** | ❌ | 无外部依赖 |
| **external_deps** | ❌ | 无外部依赖 |

## 3.3 与上游构建系统的差异

### 3.3.1 上游构建系统

上游 Khronos OpenGL-Registry **不提供**构建系统：

- 无 CMakeLists.txt
- 无 Makefile
- 无 Bazel 构建文件
- 无 Ninja 构建文件

上游仅提供：
- 头文件（api/）
- 扩展规范（extensions/）
- XML 注册表（xml/）
- 工具脚本（genheaders.py）

### 3.3.2 OH 构建适配

| 维度 | 上游 | OpenHarmony |
|------|------|-------------|
| **构建系统** | 无 | GN（通过 BUILD.gn） |
| **目标类型** | N/A | ohos_static_library |
| **头文件暴露** | 手动包含 | 通过 public_configs 自动传递 |
| **配置管理** | 无 | config + public_configs |

### 3.3.3 适配说明

OH 对本库的构建适配极其简单，核心工作在于：

| 任务 | 说明 |
|------|------|
| **导入 GN** | `import("//build/ohos.gni")` |
| **配置头文件路径** | 定义 `libGLES_public_config` |
| **创建静态库目标** | 声明 `ohos_static_library("libGLES")` |
| **传递公开配置** | 设置 `public_configs` |

## 3.4 NDK 构建适配

### 3.4.1 NDK 头文件层

OpenHarmony 在 `interface/sdk_c/` 下创建了专门的 NDK 头文件层：

```
interface/sdk_c/third_party/openGLES/
├── BUILD.gn                    # NDK 重导出配置
└── api/                       

interface/sdk_c/graphic/graphic_2d/
├── GLES2/
│   ├── BUILD.gn               # GLES2 NDK
│   ├── gl2.h                 → third_party/openGLES/api/GLES2/gl2.h
│   ├── gl2ext.h              → third_party/openGLES/api/GLES2/gl2ext.h
│   └── gl2platform.h         → third_party/openGLES/api/GLES2/gl2platform.h
├── GLES3/
│   ├── BUILD.gn              # GLES3 NDK
│   ├── gl3.h                 → third_party/openGLES/api/GLES3/gl3.h
│   └── gl3ext.h              → third_party/openGLES/api/GLES3/gl3ext.h
└── GL4/
    └── BUILD.gn              # GL4 NDK
```

### 3.4.2 GLES2 NDK 构建配置

```gn
ohos_ndk_headers("GLES2_header") {
  dest_dir = "$ndk_headers_out_dir/GLES2"
  sources = [
    "../../../third_party/openGLES/GLES2/gl2.h",
    "../../../third_party/openGLES/GLES2/gl2ext.h",
    "../../../third_party/openGLES/GLES2/gl2platform.h",
  ]
}

ohos_ndk_library("libGLESv2_ndk") {
  output_name = "GLESv2"
  output_extension = "so"
  ndk_description_file = "./libGLESv2.ndk.json"
  system_capability = "SystemCapability.Graphic.Graphic2D.GLES2"
  system_capability_headers = [
    "GLES2/gl2.h",
    "GLES2/gl2ext.h",
    "GLES2/gl2platform.h",
  ]
}
```

### 3.4.3 NDK 配置详解

| 配置项 | 值 | 说明 |
|-------|-----|------|
| **ohos_ndk_headers** | `GLES2_header` | NDK 头文件目标 |
| **dest_dir** | `$ndk_headers_out_dir/GLES2` | 头文件输出目录 |
| **sources** | `[gl2.h, gl2ext.h, gl2platform.h]` | 源头文件列表 |
| **ohos_ndk_library** | `libGLESv2_ndk` | NDK 库目标 |
| **output_name** | `GLESv2` | 输出库名称 |
| **system_capability** | `SystemCapability.Graphic.Graphic2D.GLES2` | 系统能力标识 |

## 3.5 特殊处理

### 3.5.1 静态库空目标处理

由于 `libGLES` 静态库不包含任何源文件，实际不会生成 `.a` 目标文件。此设计是 OH 构建系统的约定俗成：

```gn
ohos_static_library("libGLES") {
  public_configs = [ ":libGLES_public_config" ]
  # 无 sources，无实际目标文件生成
}
```

### 3.5.2 依赖者配置传递

当其他模块依赖 `opengles:libGLES` 时，通过 `public_external_deps` 接收配置：

```gn
ohos_shared_library("GLESv2") {
  # ...
  public_external_deps = [
    "egl:libEGL",
    "opengles:libGLES",        # 依赖此库，自动获取 public_configs
  ]
  # ...
}
```

### 3.5.3 多平台支持

本库通过头文件自然支持多平台，无需额外配置：

| 平台 | 支持情况 | 说明 |
|------|---------|------|
| **OHOS（OpenHarmony）** | ✅ | 主要目标平台 |
| **Linux** | ✅ | 使用相同的头文件 |
| **macOS** | ✅ | 使用相同的头文件 |
| **Windows** | ✅ | 使用相同的头文件 |

## 3.6 构建验证

### 3.6.1 构建命令

```bash
# 构建 libGLES 目标
hb build -p graphic_2d -T //third_party/openGLES:libGLES

# 构建 NDK 头文件
hb build -p graphic_2d -T //interface/sdk_c/third_party/openGLES:libGLES
hb build -p graphic_2d -T //interface/sdk_c/graphic/graphic_2d/GLES2:GLES2_header
```

### 3.6.2 验证步骤

| 步骤 | 验证内容 | 预期结果 |
|------|---------|---------|
| **1. 头文件包含** | 依赖模块能否找到头文件 | 编译通过 |
| **2. NDK 头文件** | NDK 应用能否引用 GLES 头文件 | NDK 构建通过 |
| **3. 配置传递** | public_configs 是否正确传递 | 依赖模块编译通过 |

## 3.7 常见问题

### Q1：为什么使用 static_library 而非 group？

使用 `static_library` 可以利用 GN 的 public_configs 机制，将头文件路径配置自动传递给依赖者。使用 `group` 无法实现此功能。

### Q2：是否会生成实际的 .a 文件？

不会。因为 `ohos_static_library` 没有指定 `sources`，GN 不会编译任何源文件，也就不会生成实际的静态库文件。但构建目标仍然有效，可用于依赖声明和配置传递。

### Q3：如何添加新的 OpenGL ES 扩展？

如需添加新的扩展支持：
1. 同步上游 Khronos OpenGL-Registry 的新扩展
2. 将扩展头文件添加到 `api/` 目录
3. 验证构建通过
