# 构建配置

## 4.1 构建系统概述

i18n_lite 使用 **GN (Generate Ninja)** 作为构建系统，与 OpenHarmony 整体构建保持一致。

### 4.1.1 构建环境

| 项目 | 要求 |
|------|------|
| **构建系统** | GN + Ninja |
| **最低 Python 版本** | Python 3.8+ |
| **构建命令** | `hb build` (OpenHarmony 构建工具) |
| **目标系统** | Mini System, Small System |

### 4.1.2 构建入口

| 文件 | 说明 |
|------|------|
| `//base/global/i18n_lite/i18n_lite.gni` | GN 标志定义 |
| `//base/global/i18n_lite/frameworks/i18n/BUILD.gn` | 核心库构建配置 |
| `//base/global/i18n_lite/interfaces/kits/BUILD.gn` | 接口构建配置 |
| `//base/global/i18n_lite/interfaces/kits/js/builtin/BUILD.gn` | JS 模块构建配置 |

## 4.2 GN 标志配置

### 4.2.1 根标志文件

**文件**：`i18n_lite.gni`

```gni
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

declare_args() {
  i18n_lite_support_i18n_product = false
}
```

### 4.2.2 标志说明

| 标志 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `i18n_lite_support_i18n_product` | bool | false | 是否支持产品级 i18n 功能 |

**使用场景**：
- 当 `i18n_lite_support_i18n_product = true` 时，会添加额外的 include_dirs 和 deps
- 主要用于完整产品开发

## 4.3 Targets 清单

### 4.3.1 框架层 Targets

#### global_i18n (静态库)

**文件**：`frameworks/i18n/BUILD.gn:54-69`

```gni
if (ohos_kernel_type == "liteos_m") {
  static_library("global_i18n") {
    sources = locale_sources
    public_configs = [ ":locale_config" ]
    deps = [
      ":global_dat",
      "//third_party/bounds_checking_function:libsec_static",
    ]
    if (i18n_lite_support_i18n_product) {
      include_dirs = [
        "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
        "//commonlibrary/utils_lite/memory/include",
      ]
      defines = [ "I18N_PRODUCT" ]
      deps += [ "//commonlibrary/utils_lite:utils" ]
    }
  }
}
```

| 属性 | 值 |
|------|-----|
| **类型** | static_library |
| **源文件** | `locale_sources` (16 个 .cpp 文件) |
| **公共配置** | `locale_config` |
| **依赖** | `global_dat`, `bounds_checking_function` |

**源文件列表** (`frameworks/i18n/BUILD.gn:21-37`)：

| 序号 | 文件 | 说明 |
|------|------|------|
| 1 | `data_resource.cpp` | 资源数据管理 |
| 2 | `date_time_data.cpp` | 日期时间数据 |
| 3 | `date_time_format.cpp` | 日期时间格式化入口 |
| 4 | `date_time_format_impl.cpp` | 日期时间格式化实现 |
| 5 | `locale_info.cpp` | 区域信息 |
| 6 | `measure_format.cpp` | 度量格式化入口 |
| 7 | `measure_format_impl.cpp` | 度量格式化实现 |
| 8 | `number_data.cpp` | 数字数据 |
| 9 | `number_format.cpp` | 数字格式化入口 |
| 10 | `number_format_impl.cpp` | 数字格式化实现 |
| 11 | `plural_format.cpp` | 复数格式化入口 |
| 12 | `plural_format_impl.cpp` | 复数格式化实现 |
| 13 | `plural_rules.cpp` | 复数规则 |
| 14 | `str_util.cpp` | 字符串工具 |
| 15 | `week_info.cpp` | 周信息 |

#### global_i18n_simulator (静态库 - 非 liteos_m)

**文件**：`frameworks/i18n/BUILD.gn:85-90`

```gni
ohos_static_library("global_i18n_simulator") {
  sources = locale_sources
  configs += [ ":locale_config" ]
  deps = [ ":global_dat" ]
}
```

#### locale_lite (Lite Component)

**文件**：`frameworks/i18n/BUILD.gn:81-83`

```gni
lite_component("locale_lite") {
  features = [ ":global_i18n" ]
}
```

### 4.3.2 接口层 Targets

#### nativeapi_locale_simulator (静态库)

**文件**：`interfaces/kits/js/builtin/BUILD.gn:33-46`

```gni
ohos_static_library("nativeapi_locale_simulator") {
  sources = [ "src/locale_module.cpp" ]

  include_dirs = [
    "include",
    "//base/global/resource_management_lite/interfaces/inner_api/include",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/base",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/jsi",
    "//commonlibrary/utils_lite/include",
  ]
  deps = [ "//third_party/bounds_checking_function:libsec_static" ]

  configs = [ ":nativeapi_locale_simulator_config" ]
}
```

#### i18n_dat (预编译数据文件)

**文件**：`interfaces/kits/BUILD.gn:16-21`

```gni
ohos_prebuilt_etc("i18n_dat") {
  source = "//base/global/i18n_lite/frameworks/i18n/i18n.dat"
  module_install_dir = "system/i18n"
  part_name = "i18n_lite"
  subsystem_name = "global"
}
```

### 4.3.3 数据复制 Target

#### global_dat

**文件**：`frameworks/i18n/BUILD.gn:47-50`

```gni
copy("global_dat") {
  sources = [ "i18n.dat" ]
  outputs = [ "$root_out_dir/data/i18n.dat" ]
}
```

## 4.4 配置详情

### 4.4.1 locale_config

**文件**：`frameworks/i18n/BUILD.gn:39-45`

```gni
config("locale_config") {
  include_dirs = [
    "//base/global/i18n_lite/interfaces/kits/i18n/include",
    "//base/global/i18n_lite/frameworks/i18n/include",
    "//third_party/bounds_checking_function/include",
  ]
}
```

| 属性 | 值 |
|------|-----|
| **类型** | config |
| **include_dirs** | 3 个目录 |

### 4.4.2 nativeapi_locale_simulator_config

**文件**：`interfaces/kits/js/builtin/BUILD.gn:16-31`

```gni
config("nativeapi_locale_simulator_config") {
  cflags = [
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECIMP=//",
    "-D_STDIO_S_DEFINED",
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
    "-pipe",
    "-Wdate-time",
    "-Wfloat-equal",
    "-Wformat=2",
    "-Wshadow",
  ]
}
```

**编译器标志说明**：

| 标志 | 说明 |
|------|------|
| `-D_*_S` | 使用安全版本的 C 标准库函数 |
| `-Wformat=2` | 增强格式警告 |
| `-Wshadow` | 变量遮蔽警告 |

## 4.5 依赖关系

### 4.5.1 组件依赖

| 依赖组件 | 类型 | 说明 |
|----------|------|------|
| `utils_lite` | 组件依赖 | 基础工具库 |
| `bounds_checking_function` | 第三方依赖 | 内存安全函数 |

**来源**：`bundle.json:50-54`

```json
"deps": {
  "components": [ "utils_lite" ],
  "third_party": [ "bounds_checking_function" ]
}
```

### 4.5.2 Target 依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                        Target 依赖                              │
│                                                                  │
│   ┌─────────────────────┐                                       │
│   │ nativeapi_locale_   │                                       │
│   │ simulator           │                                       │
│   └──────────┬──────────┘                                       │
│              │                                                  │
│              ▼                                                  │
│   ┌─────────────────────┐     ┌─────────────────────┐            │
│   │ bounds_checking_    │────▶│ libsec_static       │            │
│   │ function            │     │ / libsec_shared     │            │
│   └─────────────────────┘     └─────────────────────┘            │
│                                                                │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    global_i18n                         │   │
│   │  ┌─────────────────────────────────────────────────────┐│   │
│   │  │ locale_sources (16 个 .cpp 文件)                   ││   │
│   │  └─────────────────────────────────────────────────────┘│   │
│   │                          │                               │   │
│   │                          ▼                               │   │
│   │  ┌─────────────────────────────────────────────────────┐│   │
│   │  │ deps: [global_dat, bounds_checking_function]       ││   │
│   │  └─────────────────────────────────────────────────────┘│   │
│   └─────────────────────────────────────────────────────────┘   │
│                          │                                      │
│                          ▼                                      │
│               ┌─────────────────────┐                            │
│               │     i18n.dat       │                            │
│               │  (复制到 out/)      │                            │
│               └─────────────────────┘                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 4.6 编译产物

### 4.6.1 产物清单

| Target | 类型 | 产物路径 | 用途 |
|--------|------|----------|------|
| `global_i18n` | .a / .so | `out/{product}/libs/libglobal_i18n.a` | 核心静态库 |
| `global_i18n_simulator` | .a | `out/{product}/libs/libglobal_i18n_simulator.a` | 模拟器静态库 |
| `nativeapi_locale_simulator` | .a | `out/{product}/libs/libnativeapi_locale_simulator.a` | JS API 库 |
| `i18n_dat` | .dat | `out/{product}/system/i18n/i18n.dat` | 区域数据 |

### 4.6.2 安装路径

| 产物 | 安装路径 |
|------|----------|
| **i18n.dat** | `/system/i18n/i18n.dat` |
| **静态库** | `{out_dir}/libs/` |
| **头文件** | `{include_dir}/i18n/` |

### 4.6.3 产物大小估算

| 产物 | 估算大小 | 说明 |
|------|----------|------|
| `libglobal_i18n.a` | ~500KB | 未压缩的静态库 |
| `i18n.dat` | ~83KB | 二进制区域数据 |
| `libnativeapi_locale_simulator.a` | ~50KB | JS API 静态库 |

## 4.7 运行时加载关系

### 4.7.1 数据文件加载

```
┌──────────┐         ┌────────────────┐         ┌─────────────┐
│  应用    │         │  global_i18n   │         │  i18n.dat   │
│          │  Load   │  (静态链接)     │  Read   │  (只读)     │
│          │ ──────▶ │                │ ──────▶ │             │
└──────────┘         └────────────────┘         └─────────────┘
```

**加载路径** (`data_resource.cpp:29-33`)：

```cpp
#ifdef I18N_PRODUCT
static const char *DATA_RESOURCE_PATH = "system/i18n/i18n.dat";
#else
static const char *DATA_RESOURCE_PATH = "/storage/data/i18n.dat";
#endif
```

### 4.7.2 静态链接关系

```
应用可执行文件
    │
    ├── libglobal_i18n.a (链接时嵌入)
    │       │
    │       ├── libsec_*.a (bounds_checking_function)
    │       │
    │       └── i18n.dat (编译时复制，运行时装载)
    │
    └── libnativeapi_locale_simulator.a (JS 引擎链接)
```

## 4.8 构建命令

### 4.8.1 全量构建

```bash
# 设置构建环境
source build.sh

# 全量构建
hb build -f
```

### 4.8.2 增量构建

```bash
# 只构建 i18n_lite
hb build -f -p global_i18n_lite
```

### 4.8.3 独立构建测试

```bash
# 进入构建目录
cd out/{product}/

# 重新生成 ninja 文件
gn gen out/{product}

# 构建
ninja global_i18n
```

## 4.9 构建变体

### 4.9.1 按内核类型

| 内核类型 | Target | 产物类型 |
|----------|--------|----------|
| `liteos_m` | `global_i18n` | 静态库 (.a) |
| 其他 | `global_i18n_simulator` | 静态库 (.a) |

### 4.9.2 按产品配置

| 配置 | 标志 | 额外依赖 |
|------|------|----------|
| 默认 | `i18n_lite_support_i18n_product = false` | 无 |
| 产品级 | `i18n_lite_support_i18n_product = true` | `utils_lite`, `hilog_lite` |

## 4.10 常见构建问题

### 4.10.1 编译错误：找不到头文件

**问题**：编译时提示找不到 `date_time_format.h`

**解决方案**：

```bash
# 检查 include_dirs 配置
# 确保以下路径存在：
# - //base/global/i18n_lite/interfaces/kits/i18n/include
# - //base/global/i18n_lite/frameworks/i18n/include
```

### 4.10.2 链接错误：未定义的引用

**问题**：`GLOBAL_GetLanguage` 未定义

**解决方案**：

```bash
# 确保链接了以下库：
# - libnativeapi_locale_simulator.a
# - libglobal_i18n.a
# - libsec_*.a (bounds_checking_function)
```

### 4.10.3 运行时错误：i18n.dat 找不到

**问题**：应用启动时报错找不到 i18n.dat

**解决方案**：

```bash
# 检查 i18n.dat 是否已复制到正确位置
ls out/{product}/system/i18n/i18n.dat

# 如果不存在，手动复制
cp frameworks/i18n/i18n.dat out/{product}/system/i18n/
```

---

*最后更新：2026-02-06*
