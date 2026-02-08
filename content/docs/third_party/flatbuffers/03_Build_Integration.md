# OpenHarmony 构建适配

本文档详细说明 FlatBuffers 在 OpenHarmony 系统中的构建配置，包括 GN 构建系统适配、编译选项配置以及与上游构建系统的差异分析。

## 3.1 构建系统概述

### 3.1.1 上游构建系统

FlatBuffers 上游项目支持多种构建系统：

| 构建系统 | 用途 | 优先级 |
|---------|------|--------|
| **CMake** | 主要构建系统，适合跨平台编译 | 主要 |
| **Bazel** | Google 内部构建系统，支持 gRPC 测试 | 可选 |
| **Meson** | 实验性支持 | 极少使用 |

### 3.1.2 OpenHarmony 构建系统

OpenHarmony 采用 **GN (Generate Ninja)** 作为主要构建系统：

```
OpenHarmony 构建流程:
1. GN 配置文件 (.gn, BUILD.gn) → 2. Ninja 构建文件 (.ninja) → 3. 编译链接
```

FlatBuffers 在 OH 中的构建适配通过 `BUILD.gn` 文件实现。

---

## 3.2 BUILD.gn 配置详解

### 3.2.1 文件位置与结构

```
third_party/flatbuffers/
├── BUILD.gn              # OH GN 构建配置（主文件）
├── CMakeLists.txt        # 上游 CMake 配置
└── BUILD.bazel          # 上游 Bazel 配置
```

### 3.2.2 完整 BUILD.gn 内容

```gn
# Copyright 2024 Huawei Technologies Co., Ltd
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ============================================================================
import("//build/ohos.gni")

config("flatbuffers_include") {
  include_dirs = [ "./include/" ]
}

ohos_static_library("libflatbuffers_static") {
  part_name = "flatbuffers"
  subsystem_name = "thirdparty"
  public_configs = [ ":flatbuffers_include" ]
}

ohos_copy("flatbuffers_for_cangjie") {
  sources = [ 
    "cangjie/decode.cj",
    "cangjie/flatbuffer_object.cj",
    "cangjie/table.cj",
  ]

  outputs = [ "${target_out_dir}/cangjie/{{source_file_part}}" ]
  part_name = "flatbuffers"
  subsystem_name "thirdparty"
}
```

### 3.2.3 配置元素详解

#### 1. 导入声明

```gn
import("//build/ohos.gni")
```

**说明**: 导入 OpenHarmony 的 GN 构建模板库，包含 `ohos_static_library`、`ohos_copy` 等 OH 专用模板。

#### 2. Include 配置

```gn
config("flatbuffers_include") {
  include_dirs = [ "./include/" ]
}
```

| 配置项 | 值 |
|-------|------|
| **配置名称** | `flatbuffers_include` |
| **include_dirs** | `./include/` (FlatBuffers 头文件目录) |
| **用途** | 为依赖此库的其他模块提供头文件搜索路径 |

#### 3. 静态库目标

```gn
ohos_static_library("libflatbuffers_static") {
  part_name = "flatbuffers"
  subsystem_name = "thirdparty"
  public_configs = [ ":flatbuffers_include" ]
}
```

| 属性 | 值 | 说明 |
|-----|------|------|
| **part_name** | `flatbuffers` | 组件名称，用于模块化管理 |
| **subsystem_name** | `thirdparty` | 子系统名称 |
| **public_configs** | `flatbuffers_include` | 公开给依赖者的配置 |

**源文件自动推导**:

GN 构建系统会自动推导源文件，默认包含：
- `src/` 目录下的 `.cc`/`.cpp` 文件
- `include/` 目录下的头文件（不参与编译，仅供引用）

#### 4. 仓颉资源复制任务

```gn
ohos_copy("flatbuffers_for_cangjie") {
  sources = [ 
    "cangjie/decode.cj",
    "cangjie/flatbuffer_object.cj",
    "cangjie/table.cj",
  ]

  outputs = [ "${target_out_dir}/cangjie/{{source_file_part}}" ]
  part_name = "flatbuffers"
  subsystem_name = "thirdparty"
}
```

| 属性 | 值 | 说明 |
|-----|------|------|
| **sources** | 3 个 `.cj` 文件 | 仓颉语言绑定源文件 |
| **outputs** | `${target_out_dir}/cangjie/{{source_file_part}}` | 输出目录 |
| **任务类型** | `ohos_copy` | 文件复制任务，非编译任务 |

---

## 3.3 编译选项分析

### 3.3.1 OH 编译选项

FlatBuffers 在 OH 中使用的编译选项主要继承自全局配置：

| 选项类型 | 来源 | 说明 |
|---------|------|------|
| **C++ 标准** | 全局配置 | 通常为 C++14 |
| **警告级别** | 全局配置 | 遵循 OH 统一标准 |
| **优化级别** | 全局配置 | Release 模式为 -O2/-O3 |
| **位置无关代码** | 全局配置 | 根据目标类型设置 |

### 3.3.2 与上游差异

| 方面 | 上游 (CMake) | OpenHarmony (GN) |
|-----|-------------|------------------|
| **构建系统** | CMake | GN |
| **静态库目标** | `flatbuffers` | `libflatbuffers_static` |
| **包含路径** | 通过 `target_include_directories` | 通过 `config` |
| **仓颉支持** | 无 | 新增 `ohos_copy` 任务 |
| **gRPC 集成** | 可选 | 包含 Patch 修复 |

### 3.3.3 gRPC 编译选项（Patch）

通过 Patch 引入的 gRPC 相关编译选项：

```bzl
# bazel/copts.bwl (Patch 后)

GRPC_LLVM_WARNING_FLAGS = [
  "-Wall",
  "-Wextra", 
  "-Werror",
  "-Wconversion",
  "-Wno-sign-conversion",
]

GRPC_DEFAULT_COPTS = select({
    "//:use_strict_warning": GRPC_LLVM_WARNING_FLAGS + ["-DUSE_STRICT_WARNING=1"],
    "//conditions:default": [],
}) + select({
    "@bazel_tools//src/conditions:windows": ["/std:c++14"],
    "//conditions:default": ["-std=c++14"],
})
```

---

## 3.4 组件配置 (bundle.json)

### 3.4.1 bundle.json 内容

```json
{
    "name": "@ohos/flatbuffers",
    "description": "Memory Efficient Serialization Library.",
    "version": "v25.2.10",
    "license": "Apache License 2.0",
    "pubiishAs": "code-segment",
    "segment": {
      "destPath": "third_party/flatbuffers"
    },
    "dirs": {},
    "scripts": {},
    "component": {
      "name": "flatbuffers",
      "subsystem": "thirdparty",
      "syscap": [],
      "features": [],
      "adapted_system_type": [
        "standard",
        "small"
      ],
      "rom": "0",
      "ram": "0",
      "deps": {
        "components": [],
        "third_party": []
      },
      "build": {
        "sub_component": [ "//third_party/flatbuffers:libflatbuffers_static" ],
        "inner_kits": [
                {
                    "name": "//third_party/flatbuffers:libflatbuffers_static",
                    "header": {
                        "header_base": "//third_party/flatbuffers/include",
                        "header_files": []
                    }
                },
                {
                    "name": "//third_party/flatbuffers:flatbuffers_for_cangjie"
                }
        ],
        "test": []
      }
    }
}
```

### 3.4.2 关键配置说明

| 配置项 | 值 | 说明 |
|-------|------|------|
| **subsystem** | `thirdparty` | 属于第三方库子系统 |
| **adapted_system_type** | `["standard", "small"]` | 支持标准系统和轻量系统 |
| **sub_component** | `libflatbuffers_static` | 静态库作为子组件 |
| **inner_kits** | 2 个 | 提供给其他模块的接口 |

#### inner_kits 配置

| Kit 名称 | 类型 | 提供内容 |
|---------|------|---------|
| `libflatbuffers_static` | 头文件 Kit | FlatBuffers C++ 头文件 |
| `flatbuffers_for_cangjie` | 资源 Kit | 仓颉语言绑定文件 |

---

## 3.5 构建产物

### 3.5.1 产物清单

| 产物类型 | 路径 | 说明 |
|---------|------|------|
| **静态库** | `out/.../obj/third_party/flatbuffers/libflatbuffers_static.a` | FlatBuffers 静态库 |
| **头文件** | `out/.../include/flatbuffers/` | 公开头文件 |
| **仓颉资源** | `out/.../cangjie/` | 仓颉语言绑定文件 |

### 3.5.2 链接方式

FlatBuffers 在 OH 中采用**静态链接**方式：

```
应用/模块
    │
    ├── 静态链接 flatbuffers
    │       │
    │       ▼
    └── libflatbuffers_static.a
```

**优势**:
- 无运行时依赖，部署简单
- 编译器优化更充分（跨模块内联）
- 无符号冲突问题

**劣势**:
- 可执行文件体积增大
- 库更新需要重新编译

---

## 3.6 构建验证

### 3.6.1 验证命令

```bash
# 编译 FlatBuffers 静态库
hb set
hb build -b flatbuffers

# 或使用 GN 直接编译
gn gen out/flatbuffers
ninja -C out/flatbuffers flatbuffers
```

### 3.6.2 验证清单

- [ ] 静态库编译成功
- [ ] 头文件可正常包含
- [ ] 仓颉资源复制成功
- [ ] 无编译警告

---

## 3.7 常见问题

### Q1: 如何添加新的编译选项？

在 `BUILD.gn` 中添加新的 `config`：

```gn
config("flatbuffers_flags") {
  defines = [ "FLATBUFFERS_ASSERT=1" ]
  cflags_cc = [ "-ffloat-store" ]
}

ohos_static_library("libflatbuffers_static") {
  # ... 其他配置
  configs += [ ":flatbuffers_flags" ]
}
```

### Q2: 如何包含 FlatBuffers 到其他模块？

在依赖模块的 `BUILD.gn` 中：

```gn
ohos_executable("my_app") {
  # ... 其他配置
  deps = [
    "//third_party/flatbuffers:libflatbuffers_static",
  ]
}
```

### Q3: 如何排查构建问题？

```bash
# 查看详细构建日志
ninja -C out/flatbuffers -v flatbuffers

# 检查 GN 配置
gn args out/flatbuffers --list
```

---

## 3.8 小结

FlatBuffers 在 OpenHarmony 中的构建适配遵循 OH 统一规范：

1. **构建系统**: CMake → GN 迁移，通过 `BUILD.gn` 配置
2. **编译选项**: 继承全局配置，仅通过 Patch 添加 gRPC 相关的 C++14 标准
3. **组件化**: 通过 `bundle.json` 定义组件元数据，支持标准系统和轻量系统
4. **仓颉支持**: 通过 `ohos_copy` 任务提供仓颉语言绑定文件

整体适配策略是**最小化修改**，保留上游代码结构，仅添加 OH 特有的配置和文件。

---

## 参考资料

- [OpenHarmony GN 构建指南](https://gitee.com/openharmony/build)
- [GN 语言文档](https://gn.googlesource.com/gn/+/HEAD/docs/language.md)
- [FlatBuffers CMake 构建](https://github.com/google/flatbuffers/blob/master/CMakeLists.txt)
