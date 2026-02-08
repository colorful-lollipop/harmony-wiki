# 构建系统

## 1. 概述

本文档描述 `resource_management_lite` 组件的 GN 构建系统配置，包括 Targets 定义、依赖关系、编译产物等。

**构建系统特点**：
- 支持 liteos_a、liteos_m、Windows (mingw) 多平台
- 条件编译：liteos_a 使用 C++ 源码，其他使用 C 源码
- 静态库和共享库两种产物形态

---

## 2. 构建配置文件

### 2.1 关键构建文件

| 文件 | 路径 | 用途 |
|------|------|------|
| `BUILD.gn` | `frameworks/resmgr_lite/BUILD.gn` | 主要构建配置 |
| `bundle.json` | 根目录 | 组件元数据配置 |
| `CMakeLists.txt` | `frameworks/resmgr_lite/CMakeLists.txt` | CMake 构建（模拟器） |

### 2.2 BUILD.gn 结构概览

```
frameworks/resmgr_lite/BUILD.gn
├── 条件导入
├── 源码选择 (global_sources)
├── Config 定义
│   ├── global_resmgr_config
│   ├── global_public_config
│   └── global_resmgr_mingw_config
└── Target 定义
    ├── global_resmgr (liteos_m static_library)
    ├── global_resmgr (liteos_a shared_library)
    ├── global_manager_lite (lite_component)
    └── global_resmgr_simulator (ohos_static_library)
```

---

## 3. 源码选择逻辑

### 3.1 条件编译

```gn
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")
} else {
  import("//build/ohos.gni")
}
```

### 3.2 平台源码差异

```gn
if (defined(ohos_lite) && ohos_kernel_type == "liteos_a") {
  # liteos_a: 使用 C++ 源码
  global_sources += [
    "src/global.cpp",
    "src/hap_manager.cpp",
    "src/hap_resource.cpp",
    "src/locale_matcher.cpp",
    "src/lock.cpp",
    "src/res_config_impl.cpp",
    "src/res_desc.cpp",
    "src/res_locale.cpp",
    "src/resource_manager_impl.cpp",
    "src/utils/hap_parser.cpp",
    "src/utils/string_utils.cpp",
    "src/utils/utils.cpp",
  ]
} else {
  # liteos_m / mingw: 使用 C 源码
  global_sources += [
    "src/global.c",
    "src/global_utils.c",
  ]
}
```

| 平台 | 源码 | 说明 |
|------|------|------|
| **liteos_a** | C++ (11 个文件) | 完整功能 |
| **liteos_m** | C (2 个文件) | 基础功能 |
| **Windows (mingw)** | C (2 个文件) | 模拟器 |

**证据来源**：
- `frameworks/resmgr_lite/BUILD.gn` (lines 20-41)

---

## 4. Config 定义

### 4.1 global_resmgr_config

```gn
config("global_resmgr_config") {
  include_dirs = [
    "include",
    "//base/global/resource_management_lite/interfaces/inner_api/include",
    "//commonlibrary/utils_lite/include",
    "//third_party/bounds_checking_function/include",
  ]

  if (defined(ohos_lite) && ohos_kernel_type == "liteos_a") {
    include_dirs += [
      "//third_party/zlib",
      "//third_party/zlib/contrib/minizip",
      "//commonlibrary/utils_lite/memory",
      "//base/global/i18n_lite/interfaces/kits/i18n/include/",
    ]
  }
}
```

| 属性 | 值 |
|------|-----|
| `include_dirs` | 头文件搜索路径 |
| 条件扩展 | liteos_a 额外包含 zlib、i18n 路径 |

---

### 4.2 global_public_config

```gn
config("global_public_config") {
  include_dirs = [
    "//base/global/resource_management_lite/frameworks/resmgr_lite/include",
    "//base/global/resource_management_lite/interfaces/inner_api/include",
  ]
}
```

| 属性 | 值 |
|------|-----|
| `include_dirs` | 对外公共头文件路径 |

---

### 4.3 global_resmgr_mingw_config

```gn
config("global_resmgr_mingw_config") {
  cflags = [
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECIMP=//",
    "-D_STDIO_S_DEFINED",
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
  ]
}
```

| 属性 | 值 |
|------|-----|
| `cflags` | Windows 安全函数宏定义 |

**证据来源**：
- `frameworks/resmgr_lite/BUILD.gn` (lines 43-78)

---

## 5. Target 定义

### 5.1 liteos_m: static_library

```gn
if (defined(ohos_lite)) {
  if (ohos_kernel_type == "liteos_m") {
    static_library("global_resmgr") {
      sources = global_sources
      public_configs = [ ":global_resmgr_config" ]
      deps = [ "//third_party/bounds_checking_function:libsec_static" ]
    }
  }
}
```

| 属性 | 值 |
|------|-----|
| `type` | `static_library` |
| `sources` | `global_sources` |
| `public_configs` | `global_resmgr_config` |
| `deps` | `bounds_checking_function:libsec_static` |
| **产物** | `libglobal_resmgr.a` |

---

### 5.2 liteos_a: shared_library

```gn
  } else {
    shared_library("global_resmgr") {
      sources = global_sources
      configs += [ ":global_resmgr_config" ]
      deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
      if (ohos_kernel_type == "liteos_a") {
        public_deps = [
          "//base/global/i18n_lite/frameworks/i18n:global_i18n",
          "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
          "//build/lite/config/component/zlib:zlib_shared",
        ]
      }
    }
  }
```

| 属性 | 值 |
|------|-----|
| `type` | `shared_library` |
| `sources` | `global_sources` |
| `configs` | `global_resmgr_config` |
| `deps` | `bounds_checking_function:libsec_shared` |
| `public_deps` | `global_i18n`、`hilog_shared`、`zlib_shared` |
| **产物** | `libglobal_resmgr.so` |

---

### 5.3 lite_component

```gn
  lite_component("global_manager_lite") {
    features = [ ":global_resmgr" ]
  }
```

| 属性 | 值 |
|------|-----|
| `type` | `lite_component` |
| `features` | 包含 `global_resmgr` |

---

### 5.4 simulator: ohos_static_library

```gn
} else {
  ohos_static_library("global_resmgr_simulator") {
    sources = global_sources
    public_configs = [ ":global_public_config" ]
    external_deps = [ "bounds_checking_function:libsec_static" ]
    configs = [ ":global_resmgr_mingw_config" ]
    subsystem_name = "global"
    part_name = "resource_management_lite"
  }
}
```

| 属性 | 值 |
|------|-----|
| `type` | `ohos_static_library` (模拟器) |
| `sources` | `global_sources` |
| `public_configs` | `global_public_config` |
| `external_deps` | `bounds_checking_function:libsec_static` |
| `configs` | `global_resmgr_mingw_config` |
| `subsystem_name` | `global` |
| `part_name` | `resource_management_lite` |
| **产物** | `libglobal_resmgr_simulator.a` |

---

## 6. 依赖关系

### 6.1 内部依赖

```
global_resmgr (liteos_m)
    └── bounds_checking_function:libsec_static

global_resmgr (liteos_a)
    ├── bounds_checking_function:libsec_shared
    ├── global_i18n (i18n_lite)
    ├── hilog_shared (hilog_lite)
    └── zlib_shared (zlib)

global_resmgr_simulator
    └── bounds_checking_function:libsec_static
```

### 6.2 bundle.json 中的依赖声明

```json
{
  "component": {
    "name": "resource_management_lite",
    "deps": {
      "components": [
        "utils_lite",
        "bounds_checking_function"
      ]
    }
  }
}
```

| 组件 | 类型 | 用途 |
|------|------|------|
| `utils_lite` | 组件依赖 | 通用工具函数 |
| `bounds_checking_function` | 组件依赖 | 安全字符串函数 |
| `global_i18n_lite` | Target 依赖 | 国际化支持 |
| `hilog_lite` | Target 依赖 | 日志输出 |
| `zlib` | Target 依赖 | ZIP 解压 |

---

## 7. 编译产物

### 7.1 产物清单

| 平台 | Target | 产物类型 | 产物名称 | 位置（预估） |
|------|--------|----------|----------|--------------|
| **liteos_m** | `global_resmgr` | 静态库 | `libglobal_resmgr.a` | `out/{product}/libs/` |
| **liteos_a** | `global_resmgr` | 共享库 | `libglobal_resmgr.so` | `out/{product}/libs/` |
| **模拟器** | `global_resmgr_simulator` | 静态库 | `libglobal_resmgr_simulator.a` | `out/{product}/libs/` |

### 7.2 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                          运行时加载链                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  应用 ──► dlopen/dlsym ──► libglobal_resmgr.so (liteos_a)          │
│                                                                      │
│  应用 ──► 链接 ──► libglobal_resmgr.a (liteos_m)                   │
│                                                                      │
│  应用 ──► 链接 ──► libglobal_resmgr_simulator.a (模拟器)           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.3 动态依赖

**liteos_a 共享库依赖**：

```
libglobal_resmgr.so
    ├── libsec_shared.so        (bounds_checking_function)
    ├── libglobal_i18n.so       (global_i18n_lite)
    ├── libhilog_shared.so      (hilog_lite)
    └── libzlib_shared.so       (zlib)
```

---

## 8. CMake 构建（模拟器）

### 8.1 CMakeLists.txt 位置

```
frameworks/resmgr_lite/CMakeLists.txt
```

### 8.2 主要配置

```cmake
# CMakeLists.txt 概览
cmake_minimum_required(VERSION 3.16)
project(global_resmgr)

# 包含目录
include_directories(
    include/
    ../include/
    ../../interfaces/inner_api/include/
)

# 源文件
file(GLOB_RECURSE sources "src/*.c" "src/*.cpp")

# 目标定义
add_library(global_resmgr STATIC ${sources})

# 链接库
target_link_libraries(global_resmgr sec_static)
```

---

## 9. 编译选项

### 9.1 条件编译宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `ohos_lite` | 构建系统 | OpenHarmony Lite 系统 |
| `liteos_a` | 构建系统 | LiteOS A Kernel |
| `liteos_m` | 构建系统 | LiteOS M Kernel |
| `_WIN32` / `_WIN64` | 编译器 | Windows 平台 |

### 9.2 编译器标志

**Mingw 专用**：

| 标志 | 说明 |
|------|------|
| `-D_INC_STRING_S` | Windows 安全字符串 |
| `-D_INC_WCHAR_S` | Windows 宽字符安全函数 |
| `-D_SECIMP=//` | 安全实现路径 |
| `-D_STDIO_S_DEFINED` | 标准输入输出安全函数 |

---

## 10. 编译命令示例

### 10.1 liteos_a 编译

```bash
# 完整系统编译
./build.sh --product {product} --target cpu_type

# 单独编译组件
hb build -p {part_name}
```

### 10.2 liteos_m 编译

```bash
# 使用 lite_component
hb build -p global -f
```

### 10.3 模拟器编译

```bash
mkdir build && cd build
cmake ..
make -j4
```

---

## 11. 常见问题

### Q1: 编译 liteos_a 提示缺少 i18n 头文件？

**解决**：确保 `global_i18n_lite` 子模块已初始化：

```bash
git submodule update --init --recursive
```

### Q2: 链接错误：undefined reference to `strcpy_s`？

**解决**：确认链接了 `bounds_checking_function` 库。

### Q3: 模拟器编译失败？

**解决**：检查 `CMakeLists.txt` 中的路径配置是否正确。

---

## 12. 文档链接

| 主题 | 文档 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](./01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| C API | [03_C_API.md](./03_C_API.md) |
| C++ API | [04_Cpp_API.md](./04_Cpp_API.md) |
| 安全分析 | [06_Security_Analysis.md](./06_Security_Analysis.md) |
| 常见问题 | [07_Troubleshooting.md](./07_Troubleshooting.md) |
