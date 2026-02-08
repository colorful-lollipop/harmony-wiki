# OH 构建适配

## 概述

EGL-Registry 在 OpenHarmony 中的构建适配相对简单，因为它是 **纯头文件注册表**，不包含可编译的源代码。适配工作主要集中在 GN 构建配置和头文件导出。

---

## 构建配置详解

### BUILD.gn 文件结构

**文件路径**：`third_party/EGL/BUILD.gn`

```gn
# Copyright (c) 2021-2023 Huawei Device Co., Ltd.
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

config("libEGL_public_config") {
  include_dirs = [ "api" ]
  defines = [ "ENABLE_EGL" ]
  if (current_os == "ohos") {
    defines += [ "OHOS_PLATFORM" ]
  }
}

ohos_static_library("libEGL") {
  public_configs = [ ":libEGL_public_config" ]
}
```

### 配置项解析

#### 1. 导入语句

```gn
import("//build/ohos.gni")
```

**作用**：导入 OpenHarmony 的 GN 构建模板定义

**说明**：
- `ohos.gni` 定义了 OHOS 平台特有的构建模板
- 包括 `ohos_static_library`、`ohos_shared_library` 等

#### 2. 公共配置 (libEGL_public_config)

```gn
config("libEGL_public_config") {
  include_dirs = [ "api" ]
  defines = [ "ENABLE_EGL" ]
  if (current_os == "ohos") {
    defines += [ "OHOS_PLATFORM" ]
  }
}
```

**配置说明**：

| 配置项 | 值 | 作用 | 必需性 |
|--------|-----|------|--------|
| `include_dirs` | `["api"]` | 添加头文件搜索路径 | ✅ 必须 |
| `defines` | `["ENABLE_EGL"]` | 启用 EGL 功能的全局宏 | ✅ 必须 |
| `defines` | `["OHOS_PLATFORM"]` | OHOS 平台条件编译宏 | ✅ OHOS 必须 |

**条件编译说明**：

```c
// 在 eglplatform.h 中的使用
#elif defined(OHOS_PLATFORM)

struct NativeWindow;

typedef void*                   EGLNativeDisplayType;
typedef void*                   EGLNativePixmapType;
typedef struct NativeWindow*    EGLNativeWindowType;
```

#### 3. 静态库目标

```gn
ohos_static_library("libEGL") {
  public_configs = [ ":libEGL_public_config" ]
}
```

**目标说明**：

| 属性 | 值 | 说明 |
|------|-----|------|
| `ohos_static_library` | `libEGL` | OHOS 静态库模板 |
| `public_configs` | `libEGL_public_config` | 导出公共配置 |

**为什么是静态库？**

虽然 EGL-Registry 是纯头文件库，但使用 `ohos_static_library` 模板的原因：
1. **标准化导出**：通过库目标的 public_configs 机制导出头文件配置
2. **依赖管理**：方便其他模块通过 `deps` 引用
3. **构建兼容**：与 OH 构建系统保持一致

---

## 头文件结构

### 导出头文件

```
api/
└── EGL/
    ├── egl.h                  # EGL 1.5 核心 API
    ├── eglplatform.h           # 平台类型定义（包含 OHOS）
    └── eglext.h               # 扩展函数声明
```

### 头文件说明

| 头文件 | 用途 | OH 适配 |
|--------|------|---------|
| `egl.h` | EGL 核心函数和枚举定义 | 无需适配 |
| `eglext.h` | EGL 扩展函数和枚举声明 | 无需适配 |
| `eglplatform.h` | 平台相关的原生类型定义 | 有 OHOS_PLATFORM 分支 |

### eglplatform.h 平台分支

```c
// 条件编译结构
#if defined(EGL_NO_PLATFORM_SPECIFIC_TYPES)
  // 无平台特定类型
#elif defined(OHOS_PLATFORM)
  // OHOS 平台定义 ⭐
#elif defined(_WIN32)
  // Windows 平台定义
#elif defined(__ANDROID__)
  // Android 平台定义
#elif defined(__unix__)
  // Unix 平台定义
#else
  #error "Platform not recognized"
#endif
```

---

## 与上游构建系统的差异

### 上游构建系统

上游 EGL-Registry 使用 **GNU Make**：

```
api/
├── Makefile          # 主要构建入口
├── genheaders.py     # 头文件生成脚本
└── validate          # XML 验证工具
```

**构建目标**：
- 验证 `egl.xml` 格式正确性
- 从 `egl.xml` 生成 `egl.h`、`eglext.h`
- 编译测试程序

### OH 构建适配

| 方面 | 上游 | OpenHarmony |
|------|------|-------------|
| 构建系统 | GNU Make | GN |
| 输出 | 头文件 + 工具 | 头文件 |
| 平台适配 | 无（用户自定义） | 内置 OHOS 支持 |
| 配置方式 | 编译时参数 | GN config |

### 适配策略

1. **保留原始头文件**：直接使用上游生成的头文件
2. **添加 OHOS 平台分支**：在 `eglplatform.h` 中添加 `OHOS_PLATFORM` 支持
3. **GN 配置**：使用 OH 构建系统的标准 GN 配置方式

---

## 特殊处理说明

### 1. 静态库为空

由于 EGL-Registry 是纯头文件库，`libEGL` 静态库：
- **不包含任何 .o 文件**
- **仅用于配置导出**
- **头文件通过 `include_dirs` 导出**

### 2. 条件编译宏

| 宏 | 定义位置 | 作用 |
|-----|---------|------|
| `ENABLE_EGL` | BUILD.gn | 启用 EGL 功能（通用） |
| `OHOS_PLATFORM` | BUILD.gn (ohos) | OHOS 平台条件编译 |
| `EGL_NO_PLATFORM_SPECIFIC_TYPES` | 用户代码 | 禁用平台特定类型 |

### 3. 头文件包含方式

**正确方式**（推荐）：

```c
#include <EGL/egl.h>
#include <EGL/eglext.h>
```

**错误方式**：

```c
#include "third_party/EGL/api/EGL/egl.h"  // 不推荐
```

通过 `//third_party/EGL:libEGL` 依赖会自动添加 `api/` 到 include 路径。

---

## 依赖关系

### 被依赖情况

EGL 库作为头文件提供者，被以下模块依赖：

```gn
# 示例：在其他模块的 BUILD.gn 中
deps = [
  "//third_party/EGL:libEGL",  # 获取 EGL 头文件
]
```

### 内部依赖

EGL-Registry 内部无依赖：
- **上游依赖**：无
- **第三方依赖**：无
- **OH 依赖**：无

---

## 构建验证

### 验证步骤

1. **头文件语法检查**：

```bash
# 检查头文件语法
gcc -E -I third_party/EGL/api -c third_party/EGL/api/EGL/egl.h
```

2. **包含路径验证**：

```bash
# 验证 include_dirs 配置
gn check --check-generated third_party/EGL/BUILD.gn
```

3. **OHOS_PLATFORM 宏验证**：

```c
// 测试代码
#if defined(OHOS_PLATFORM)
#warning "OHOS_PLATFORM is defined"
#endif
```

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 头文件找不到 | include_dirs 未配置 | 添加 deps 引用 |
| OHOS_PLATFORM 未定义 | 未正确配置 BUILD.gn | 检查 config 定义 |
| NativeWindow 未定义 | 缺少头文件包含 | 包含 OHOS native_window 头文件 |

---

## 维护建议

### 版本升级流程

1. **上游更新**：
   - 从 KhronosGroup/EGL-Registry 获取新版本
   - 同步 `api/` 目录下的头文件

2. **OH 适配保留**：
   - 保留 `BUILD.gn`
   - 检查 `eglplatform.h` 变更，保留 OHOS_PLATFORM 分支
   - 检查 `eglext.h` 中是否需要添加新扩展

3. **测试验证**：
   - 验证图形驱动的编译
   - 检查头文件兼容性

### 配置最佳实践

```gn
# 推荐：使用 public_configs 导出配置
ohos_static_library("myEGL") {
  deps = [ "//third_party/EGL:libEGL" ]
}

# 错误：不推荐直接引用头文件路径
ohos_static_library("myEGL") {
  include_dirs = [ "third_party/EGL/api" ]
}
```

通过 `deps` 引用可以自动继承 public_configs 中的所有配置。
