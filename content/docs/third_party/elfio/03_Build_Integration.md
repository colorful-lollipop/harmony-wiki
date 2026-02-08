# OH 构建适配

## 构建配置概述

### BUILD.gn 结构

ELFIO 在 OpenHarmony 中的构建配置位于 `third_party/elfio/BUILD.gn`。

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/ohos.gni")

config("elfio_public_config") {
  include_dirs = [ 
    "//third_party/elfio/elfio",
    "//third_party/elfio/c_wrapper"
  ]
}

ohos_shared_library("elfio") {
  sources = [
    "./c_wrapper/elfio_c_wrapper.cpp",
    "//third_party/elfio/elfio/elfio_demo.cpp",
  ]

  include_dirs = [
    "//third_party/elfio",
    "./c_wrapper/",
    "./elfio/",
    "./",
  ]

  public_configs = [ ":elfio_public_config" ]

  license_file = "//third_party/elfio/LICENSE.txt"

  subsystem_name = "thirdparty"
  part_name = "elfio"
}
```

---

## 关键配置说明

### 1. 构建目标类型

```gn
ohos_shared_library("elfio")
```

| 属性 | 值 |
|------|-----|
| **目标类型** | 共享库（shared_library） |
| **原因** | 提供给多个模块共享使用 |
| **替代方案** | header_only（但 OH 采用共享库形式） |

### 2. 源文件配置

```gn
sources = [
  "./c_wrapper/elfio_c_wrapper.cpp",      # C 接口实现
  "//third_party/elfio/elfio/elfio_demo.cpp",  # 空文件（仅用于编译）
]
```

| 文件 | 作用 | 说明 |
|------|------|------|
| `elfio_c_wrapper.cpp` | C 包装器实现 | 必须，包含所有 C 接口 |
| `elfio_demo.cpp` | 空源文件 | 仅用于触发编译，无实际代码 |

**说明**：
- ELFIO 本身是纯头文件库
- 添加空源文件是为了能够编译出共享库
- C 包装器提供了完整的 C 语言接口

### 3. 头文件路径配置

```gn
include_dirs = [
  "//third_party/elfio",           # 库根目录
  "./c_wrapper/",                  # C 包装器头文件
  "./elfio/",                      # ELFIO 头文件目录
  "./",                            # 当前目录
]
```

### 4. 公共配置

```gn
config("elfio_public_config") {
  include_dirs = [ 
    "//third_party/elfio/elfio",
    "//third_party/elfio/c_wrapper"
  ]
}

public_configs = [ ":elfio_public_config" ]
```

**作用**：导出给依赖模块使用的头文件路径。

---

## 与上游构建系统的差异

### 上游构建系统

上游使用 CMake：

```cmake
cmake_minimum_required(VERSION 3.10)
project(elfio)

add_library(elfio STATIC headers)
target_include_directories(elfio INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

### OH 构建适配

| 对比项 | 上游 CMake | OH BUILD.gn |
|--------|------------|-------------|
| **目标类型** | STATIC library + INTERFACE | ohos_shared_library |
| **编译** | 头文件无需编译 | 需要编译出共享库 |
| **C 包装器** | 可选 | 必须（默认包含） |
| **输出** | 头文件 + 静态库 | 共享库（.so） |

---

## 依赖模块的使用方式

### 方式 1：external_deps（推荐）

```gn
external_deps = [ "elfio:elfio" ]
```

| 优点 | 缺点 |
|------|------|
| 自动处理依赖 | 需要 OH 构建系统支持 |
| 传递 public_configs | 灵活性较低 |
| 头文件路径自动添加 | - |

### 方式 2：直接 include

```gn
include_dirs += [ 
  "$ark_third_party_root/elfio",
  "$ark_third_party_root/elfio/elfio",
]

configs += [ "$ark_third_party_root/elfio:elfio_public_config" ]
```

| 优点 | 缺点 |
|------|------|
| 灵活性高 | 需要手动管理依赖 |
| 可定制配置 | 可能遗漏配置 |
| 适合特殊情况 | 维护成本高 |

---

## 编译选项

### 默认编译选项

OH 构建使用默认编译选项：

```gn
# 无特殊 defines
# 无特殊 cflags/cflags_cc
# 无特殊 configs
```

### 已知兼容性

| 选项 | 状态 | 备注 |
|------|------|------|
| `-Wno-error=missing-field-initializers` | 部分模块添加 | 用于 irtoc 模块 |
| C++ 标准 | ISO C++14+ | ELFIO 要求 |

---

## 构建产物

### 输出文件

```
out/.../
├── libelfio.so              # 共享库
├── libelfio.so.link         # 链接文件（符号链接）
└── ...
```

### 头文件

```
third_party/elfio/
├── elfio/                   # ELFIO 头文件
│   ├── elfio.hpp
│   ├── elfio_*.hpp
│   └── ...
└── c_wrapper/               # C 包装器头文件
    ├── elfio_c_wrapper.h
    ├── elfio_c_wrapper.cpp
    └── elf_types_c_wrapper.hpp
```

---

## 常见问题

### Q1: 为什么编译成共享库而不是头文件库？

**原因**：
1. 提供统一的库文件（.so）
2. 便于版本管理和更新
3. 支持 C 语言的链接

### Q2: elfio_demo.cpp 的作用是什么？

**作用**：
- 空源文件，仅用于触发编译
- 没有实际代码逻辑
- 确保共享库能够被编译出来

### Q3: 如何调试 ELFIO 相关问题？

**建议**：
1. 检查 `public_configs` 是否正确传递
2. 验证头文件路径是否完整
3. 确认依赖声明方式正确

---

## 相关文档

- [README](./README.md)
- [Patch 分析](./02_Patches.md)
- [使用场景](./04_Usage_in_OH.md)
