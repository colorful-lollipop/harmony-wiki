# 03_Build_Integration.md - OpenHarmony 构建适配

## BUILD.gn 详解

### 完整配置

```gn
# Copyright (c) 2024 Huawei Device Co., Ltd.
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

config("openmax_config") {
  include_dirs = [ "api/1.1.2" ]
}

ohos_static_library("libopenmax_static") {
  public_configs = [ ":openmax_config" ]

  part_name = "openmax"
  subsystem_name = "thirdparty"
}
```

### 配置分析

#### 1. config("openmax_config")

| 属性 | 值 | 说明 |
|------|-----|------|
| `include_dirs` | `["api/1.1.2"]` | 头文件搜索路径 |

**作用**：
- 让依赖者可以通过 `#include <OMX_Core.h>` 引用头文件
- 路径指向 1.1.2 版本目录（当前使用版本）

#### 2. ohos_static_library("libopenmax_static")

| 属性 | 值 | 说明 |
|------|-----|------|
| `public_configs` | `[":openmax_config"]` | 公开配置（传递性） |
| `part_name` | `"openmax"` | OH 部件名称 |
| `subsystem_name` | `"thirdparty"` | 所属子系统 |

**特点**：
- 这是一个**空静态库**（无 sources）
- 目的是将 config 公开给其他依赖者
- 使用 `public_configs` 确保传递性（A 依赖 libopenmax，B 依赖 A，B 也能使用 openmax 头文件）

## bundle.json 详解

### 组件定义

```json
{
  "name": "@ohos/openmax",
  "description": "OpenMAX is a royalty-free...",
  "version": "3.1",
  "license": "MIT license",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/openmax"
  }
}
```

### 构建配置

```json
"component": {
  "name": "openmax",
  "subsystem": "thirdparty",
  "syscap": [],
  "features": [],
  "adapted_system_type": [
    "standard",
    "small"
  ],
  "rom": "0KB",
  "ram": "0KB"
}
```

**关键字段**：
- `adapted_system_type`: 支持 standard 和 small 系统
- `rom`/`ram`: 0KB（纯头文件，不产生二进制）

### Inner Kits 定义

```json
"build": {
  "sub_component": [ "//third_party/openmax:libopenmax_static" ],
  "inner_kits": [
    {
      "name": "//third_party/openmax:libopenmax_static",
      "header": {
        "header_base": "//third_party/openmax/api/1.1.2",
        "header_files": [
          "OMX_Audio.h",
          "OMX_Component.h",
          "OMX_ComponentExt.h",
          "OMX_ContentPipe.h",
          "OMX_Core.h",
          "OMX_CoreExt.h",
          "OMX_Image.h",
          "OMX_ImageExt.h",
          "OMX_Index.h",
          "OMX_IndexExt.h",
          "OMX_IVCommon.h",
          "OMX_Other.h",
          "OMX_Types.h",
          "OMX_Video.h",
          "OMX_VideoExt.h",
          "codec_omx_ext.h"
        ]
      }
    }
  ]
}
```

**作用**：
- `sub_component`: 声明构建目标
- `inner_kits`: 定义对外暴露的 SDK 接口
- `header_files`: 明确列出 17 个公开头文件

## 依赖者引用方式

### 在 BUILD.gn 中引用

#### 方式 1：直接依赖（推荐）

```gn
ohos_shared_library("my_codec") {
  external_deps = [
    "openmax:libopenmax_static",
  ]
}
```

**效果**：
- 自动获得 `api/1.1.2` 的 include 路径
- 可以 `#include <OMX_Core.h>`

#### 方式 2：通过子系统引用

```gn
ohos_shared_library("my_module") {
  deps = [
    "//third_party/openmax:libopenmax_static",
  ]
}
```

### 头文件引用规范

```c
// ✅ 正确 - 使用标准 OMX 头文件
#include <OMX_Core.h>
#include <OMX_Video.h>
#include <codec_omx_ext.h>

// ❌ 错误 - 不要使用相对路径
#include "../../../third_party/openmax/api/1.1.2/OMX_Core.h"
```

## 多版本管理

### 当前版本布局

```
api/
├── 1.0/           # 1.0 版本（向后兼容）
├── 1.1.2/         # 当前默认版本 ⭐
├── 1.2.0/         # 1.2.0 版本（预留）
└── cpipes/        # Content Pipes
```

### 版本切换

如需切换到 1.2.0：

```gn
# BUILD.gn 修改
config("openmax_config") {
  include_dirs = [ "api/1.2.0" ]  # 修改此处
}
```

```json
// bundle.json 修改
"inner_kits": [{
  "header": {
    "header_base": "//third_party/openmax/api/1.2.0",  // 修改此处
    "header_files": [ /* ... */ ]
  }
}]
```

## 与上游构建系统的差异

### 上游构建方式

OpenMAX IL 上游（Khronos）**不提供构建系统**，只有头文件：

```
OpenMAX-IL-Registry/
├── api/           # 只有头文件
├── extensions/    # 扩展文档
└── index.php      # Web 索引
```

### OH 构建适配

| 方面 | 上游 | OH 适配 |
|------|------|---------|
| **构建系统** | 无 | GN 构建系统 |
| **库类型** | 头文件集合 | 包装为 static_library |
| **依赖管理** | 手动 include | external_deps 声明 |
| **版本管理** | 目录命名 | BUILD.gn 配置 + bundle.json |

## 特殊配置说明

### 为什么没有 sources？

```gn
ohos_static_library("libopenmax_static") {
  # 无 sources 属性！
  public_configs = [ ":openmax_config" ]
}
```

因为 OpenMAX IL 是**纯头文件库**：
- 只有 `.h` 文件
- 无 `.c`/`.cpp` 实现文件
- 实现由芯片厂商提供（如 Rockchip OMX IL）

### 为什么使用 public_configs？

```gn
# 依赖关系：A -> libopenmax_static, B -> A
# 使用 public_configs 后，B 也能使用 openmax 头文件

ohos_static_library("A") {
  deps = [ "//third_party/openmax:libopenmax_static" ]
  # 自动继承 openmax_config 的 include_dirs
}

ohos_shared_library("B") {
  deps = [ ":A" ]
  # 也能使用 #include <OMX_Core.h>
}
```

## 构建诊断

### 常见问题

#### 问题 1：找不到头文件

```
error: 'OMX_Core.h' file not found
#include <OMX_Core.h>
         ^~~~~~~~~~~~
```

**解决**：检查 BUILD.gn 是否声明了依赖

```gn
external_deps = [
  "openmax:libopenmax_static",  # 确保有此声明
]
```

#### 问题 2：版本不匹配

```
error: unknown type name 'OMX_NEW_TYPE'
```

**原因**：使用了 1.2.0 特有的类型，但 include 的是 1.1.2

**解决**：确认 BUILD.gn 中 `include_dirs` 指向正确版本

## 构建优化建议

### 头文件预编译（可选）

对于频繁编译的模块，可考虑预编译头文件：

```gn
config("openmax_pch") {
  cflags = [ "-include", "third_party/openmax/api/1.1.2/OMX_Core.h" ]
}
```

### 依赖优化

由于 openmax 是纯头文件库，不会产生实际链接：
- 可以放心添加到 `external_deps`
- 不会增加最终二进制大小
- 不会增加运行时依赖

## 总结

| 特性 | 说明 |
|------|------|
| **构建目标** | `libopenmax_static`（空静态库，仅传递 include 路径） |
| **头文件路径** | `api/1.1.2` |
| **公开头文件** | 17 个（16 个标准 + 1 个 OH 扩展） |
| **依赖方式** | `external_deps = ["openmax:libopenmax_static"]` |
| **ROM/RAM** | 0KB（纯头文件） |
| **适用系统** | standard, small |
