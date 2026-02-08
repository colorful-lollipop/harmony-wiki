# OH 构建适配

## 一、BUILD.gn 文件分析

### 1.1 构建配置概览

本库的 BUILD.gn 文件位于 `third_party/node/BUILD.gn`，是 OpenHarmony 构建系统（GN）的配置文件。该文件负责将 Node.js N-API 头文件集成到 OH 构建系统中。

**完整配置内容**：

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
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

config("node_header_config") {
  include_dirs = [ "src" ]
}

ohos_static_library("node_header_notice") {
  public_configs = [ ":node_header_config" ]
  sources = [
    "//third_party/node/src/js_native_api.h",
    "//third_party/node/src/js_native_api_types.h",
    "//third_party/node/src/node_api.h",
    "//third_party/node/src/node_api_types.h",
  ]
  license_file = "//third_party/node/LICENSE"
  part_name = "node"
  subsystem_name = "thirdparty"
}
```

### 1.2 配置模块详解

#### 头文件配置模块

```gn
config("node_header_config") {
  include_dirs = [ "src" ]
}
```

**配置说明**：

| 属性 | 值 | 说明 |
|------|-----|------|
| config 名称 | node_header_config | 配置模块的标识名称 |
| include_dirs | ["src"] | 头文件搜索路径 |

**作用**：定义头文件的搜索路径，使得依赖本库的其他模块能够找到 `node_api.h`、`js_native_api.h` 等头文件。

**路径解析**：`"src"` 是相对于库根目录的路径，最终解析为 `//third_party/node/src`。

#### 静态库模块

```gn
ohos_static_library("node_header_notice") {
  public_configs = [ ":node_header_config" ]
  sources = [
    "//third_party/node/src/js_native_api.h",
    "//third_party/node/src/js_native_api_types.h",
    "//third_party/node/src/node_api.h",
    "//third_party/node/src/node_api_types.h",
  ]
  license_file = "//third_party/node/LICENSE"
  part_name = "node"
  subsystem_name = "thirdparty"
}
```

**模块说明**：

| 属性 | 值 | 说明 |
|------|-----|------|
| 模块类型 | ohos_static_library | OH 静态库模块 |
| 模块名称 | node_header_notice | 库的唯一标识 |
| public_configs | [:node_header_config] | 对外暴露的配置 |
| sources | 4 个头文件 | 源文件列表 |
| license_file | LICENSE 文件 | 许可证声明 |
| part_name | "node" | 部件名称 |
| subsystem_name | "thirdparty" | 子系统名称 |

### 1.3 构建目标分析

**头文件库的特殊性**：

本库被构建为「静态库」，但实际上不包含任何可执行代码，仅用于：

1. **头文件导出**：通过 `public_configs` 向依赖模块传递头文件搜索路径
2. **许可证管理**：确保 LICENSE 文件被包含在构建产物中
3. **构建依赖**：作为依赖项被其他模块引用

**构建产物**：

| 产物类型 | 说明 |
|----------|------|
| 头文件 | 4 个 N-API 头文件被复制到构建输出目录 |
| 许可证 | LICENSE 文件被包含 |
| 静态库 | 生成 .a 或 .o 文件（仅包含符号表，无代码） |

## 二、与其他模块的集成方式

### 2.1 依赖声明示例

其他模块通过以下方式依赖本库：

```gn
# 方式一：通过 public_configs 获取头文件路径
ohos_shared_library("my_native_module") {
  sources = [
    "src/my_module.cc",
    # ... 其他源文件
  ]

  deps = [
    "//third_party/node:node_header_notice"
  ]

  # 不需要显式设置 include_dirs
  # 头文件路径通过 public_configs 自动传递
}
```

```gn
# 方式二：直接设置 include_dirs（不推荐）
ohos_shared_library("my_native_module") {
  sources = [
    "src/my_module.cc",
  ]

  include_dirs = [
    "//third_party/node/src"
  ]
}
```

**推荐方式**：使用 `deps` 依赖 `node_header_notice`，通过 `public_configs` 自动获取头文件路径。这确保了：
- 头文件路径的集中管理
- 构建系统能够正确追踪依赖关系
- 未来变更时只需修改本库的 BUILD.gn

### 2.2 完整的模块依赖链

```
用户模块 (ohos_shared_library)
    │
    ├── deps
    │     │
    │     ▼
    │   node_header_notice (third_party/node)
    │         │
    │         ├── public_configs
    │         │     │
    │         │     ▼
    │         │   include_dirs = ["src"]
    │         │
    │         └── sources (4个头文件)
    │
    └── 编译时自动获得 include_dirs
```

### 2.3 构建系统集成点

本库与 OpenHarmony 构建系统的集成点如下：

| 集成点 | 机制 | 说明 |
|--------|------|------|
| 头文件路径 | public_configs | 通过 GN 配置传递 include_dirs |
| 许可证声明 | license_file | 构建产物中包含许可证信息 |
| 部件注册 | part_name | 注册到 OH 组件系统中 |
| 子系统归属 | subsystem_name | 归属到 thirdparty 子系统 |

## 三、编译选项详解

### 3.1 NAPI_VERSION 配置

N-API 使用条件编译来支持不同版本的接口。默认的 NAPI_VERSION 在头文件中定义：

**文件位置**：`src/js_native_api.h`

```c
#ifndef NAPI_VERSION
#ifdef NAPI_EXPERIMENTAL
#define NAPI_VERSION NAPI_VERSION_EXPERIMENTAL
#else
#define NAPI_VERSION 8
#endif
#endif
```

**版本配置说明**：

| NAPI_VERSION | 对应特性 | 支持的引擎 |
|--------------|----------|------------|
| 5 | Date 类型支持 | Node.js 10+ |
| 6 | BigInt、Instance Data | Node.js 12+ |
| 7 | ArrayBuffer Detaching | Node.js 14+ |
| 8 | Object freeze/seal、Type tagging | Node.js 16+ |

当前使用版本 8，支持所有 N-API 稳定特性。

### 3.2 自定义 NAPI_VERSION

如果模块需要使用特定版本的 N-API，可以通过编译宏定义：

```gn
ohos_shared_library("my_module") {
  # ... 其他配置

  cflags = [
    "-DNAPI_VERSION=8"
  ]

  # 或者使用条件编译启用实验特性
  # cflags = [
  #   "-DNAPI_EXPERIMENTAL"
  # ]
}
```

### 3.3 编译器特性控制

N-API 头文件包含多个编译器相关的宏定义：

```c
// Windows DLL 导出控制
#ifndef NAPI_EXTERN
  #ifdef _WIN32
    #define NAPI_EXTERN __declspec(dllexport)
  #elif defined(__wasm32__)
    #define NAPI_EXTERN __attribute__((visibility("default"))) \
                        __attribute__((__import_module__("napi")))
  #else
    #define NAPI_EXTERN __attribute__((visibility("default")))
  #endif
#endif

// C++ extern "C" 封装
#ifdef __cplusplus
#define EXTERN_C_START extern "C" {
#define EXTERN_C_END }
#else
#define EXTERN_C_START
#define EXTERN_C_END
#endif
```

**平台兼容性**：

| 平台 | NAPI_EXTERN 定义 | 说明 |
|------|------------------|------|
| Windows | __declspec(dllexport) | DLL 导出声明 |
| Linux | __attribute__((visibility("default"))) | GCC 可见性属性 |
| macOS | __attribute__((visibility("default"))) | GCC 可见性属性 |
| WASM | __import_module__("napi") | WebAssembly 导入 |

## 四、最佳实践

### 4.1 正确的依赖使用方式

**推荐模式**：

```gn
# 1. 声明依赖
ohos_shared_library("my_native_addon") {
  sources = [
    "my_addon.cc",
  ]

  deps = [
    "//third_party/node:node_header_notice",
    # ... 其他依赖
  ]
}

# 2. 在源文件中包含头文件
# my_addon.cc
#include <node_api.h>
#include <js_native_api.h>
```

**错误模式**（应避免）：

```gn
# 错误：直接设置 include_dirs
ohos_shared_library("my_native_addon") {
  sources = [
    "my_addon.cc",
  ]

  include_dirs = [
    "//third_party/node/src"  # 不推荐
  ]
}
```

### 4.2 许可证声明

构建产物需要包含许可证声明。通过本库的 `license_file` 配置：

```gn
ohos_shared_library("my_native_addon") {
  # ... 其他配置

  # 自动包含 third_party/node 的许可证
  license_files = [
    "//third_party/node/LICENSE"
  ]
}
```

### 4.3 条件编译使用

如果模块需要支持不同版本的 N-API：

```c
// my_addon.c
#include <node_api.h>

// 使用 N-API v6+ 特性
#if NAPI_VERSION >= 6
static napi_value GetInstanceData(napi_env env, napi_callback_info info) {
  void* data = NULL;
  napi_status status = napi_get_instance_data(env, &data);
  // ...
}
#endif

// 使用 N-API v8 特性
#if NAPI_VERSION >= 8
static napi_value FreezeObject(napi_env env, napi_callback_info info) {
  napi_value obj;
  // ...
  napi_status status = napi_object_freeze(env, obj);
  // ...
}
#endif
```

## 五、常见问题

### Q1：为什么本库不包含 Node.js 源代码？

本库仅集成 N-API 头文件，用于定义 C/C++ 模块与 JavaScript 运行时交互的接口。Node.js 本身的源代码不在此库中，因为：

- N-API 被设计为引擎无关的接口层
- OpenHarmony 使用自研的 Ark 引擎
- 头文件已足够定义接口规范

### Q2：如何验证构建配置是否正确？

验证步骤：

```bash
# 1. 检查依赖是否正确解析
hb build -p //third_party/node:node_header_notice

# 2. 检查头文件是否可访问
# 在依赖模块的源文件中尝试包含头文件
#include <node_api.h>

# 3. 检查许可证是否包含
find out -name "LICENSE" | grep node
```

### Q3：如何排查头文件路径问题？

常见问题排查：

1. **确认依赖声明**：
   ```gn
   deps = [ "//third_party/node:node_header_notice" ]
   ```

2. **检查 include_dirs**：
   ```bash
   gn desc out// deps //your/module:target --all
   ```

3. **验证头文件存在**：
   ```bash
   ls -la third_party/node/src/*.h
   ```

### Q4：能否只依赖部分头文件？

可以，但建议完整依赖整个模块：

```gn
# 完整依赖（推荐）
deps = [ "//third_party/node:node_header_notice" ]

# 部分引用（不推荐）
# 需要手动管理 include_dirs
include_dirs = [ "//third_party/node/src" ]
```

### Q5：构建产物占用空间大吗？

不。本库仅包含头文件，构建产物很小：

| 产物 | 大小估算 |
|------|----------|
| 头文件总计 | ~110 KB |
| 静态库 | ~10 KB |
| 构建输出 | ~120 KB |

## 六、与其他第三方库的对比

### 6.1 对比表

| 特性 | third_party/node | third_party/curl | third_party/openssl |
|------|------------------|-------------------|---------------------|
| 库类型 | 纯头文件 | 完整库 | 完整库 |
| Patch 数量 | 0 | 多 | 多 |
| 主要用途 | N-API 接口 | HTTP 客户端 | 加密库 |
| 依赖复杂度 | 低 | 中 | 高 |

### 6.2 差异原因

| 库 | 差异原因 |
|-----|----------|
| node | N-API 设计为引擎无关，无需修改 |
| curl | HTTP 实现需适配 OH 网络栈 |
| openssl | 加密实现需适配 OH 安全框架 |

## 七、参考信息

### 7.1 相关文件

| 文件 | 位置 | 说明 |
|------|------|------|
| BUILD.gn | third_party/node/BUILD.gn | 构建配置主文件 |
| bundle.json | third_party/node/bundle.json | OH 组件配置 |
| js_native_api.h | src/js_native_api.h | N-API 核心接口 |
| node_api.h | src/node_api.h | Node.js 扩展接口 |

### 7.2 相关文档

- [OpenHarmony 构建系统文档](https://gitee.com/openharmony/docs)
- [GN 构建语言参考](https://gn.googlesource.com/gn/+/master/reference/)
- [N-API 官方文档](https://nodejs.org/api/n-api.html)
