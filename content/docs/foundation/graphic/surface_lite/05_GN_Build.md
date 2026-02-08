# GN 构建系统

> surface_lite 构建配置与产物说明

---

## 1. 概述

### 1.1 构建系统

surface_lite 使用 **GN (Generate Ninja)** 构建系统，这是 OpenHarmony 轻量系统的标准构建工具。

### 1.2 构建文件位置

```
foundation/graphic/surface_lite/
├── BUILD.gn                 # 根构建脚本
├── bundle.json              # 组件元数据
└── test/
    └── BUILD.gn             # 测试构建脚本
```

---

## 2. 根 BUILD.gn 详解

### 2.1 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/graphic/surface_lite/BUILD.gn`

### 2.2 完整配置

```gn
# Copyright (c) 2020-2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ...

import("//build/lite/config/component/lite_component.gni")
import("//build/lite/ndk/ndk.gni")

# ============================================
# Target 1: 组件包装目标
# ============================================
lite_component("surface_lite") {
  features = [ ":surface" ]
  public_deps = features
}

# ============================================
# Target 2: NDK 库导出
# ============================================
ndk_lib("surface_lite_ndk") {
  lib_extension = ".so"
  deps = [ ":surface" ]
  head_files = [ "interfaces/kits" ]
}

# ============================================
# Target 3: 主共享库 (核心产物)
# ============================================
shared_library("surface") {
  sources = [
    "frameworks/buffer_client_producer.cpp",
    "frameworks/buffer_manager.cpp",
    "frameworks/buffer_queue.cpp",
    "frameworks/buffer_queue_consumer.cpp",
    "frameworks/buffer_queue_producer.cpp",
    "frameworks/surface.cpp",
    "frameworks/surface_buffer_impl.cpp",
    "frameworks/surface_impl.cpp",
  ]
  
  include_dirs = [
    "frameworks",
    "//drivers/peripheral/base",
    "//drivers/peripheral/display/interfaces/include",
  ]
  
  public_configs = [ ":surface_public_config" ]
  
  public_deps = [ "//foundation/graphic/graphic_utils_lite:utils_lite" ]
  
  deps = [
    "//drivers/peripheral/display/hal:hdi_display",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
  
  ldflags = [
    "-ldisplay_gfx",
    "-ldisplay_gralloc",
    "-ldisplay_layer",
  ]
  
  cflags = [ "-fPIC" ]
  cflags += [ "-Wall" ]
  cflags_cc = cflags
}

# ============================================
# Config: 公共头文件路径
# ============================================
config("surface_public_config") {
  include_dirs = [
    "interfaces/innerkits",
    "interfaces/kits",
    "//foundation/graphic/graphic_utils_lite/interfaces/kits",
  ]
}
```

---

## 3. Target 详解

### 3.1 surface (shared_library)

**类型**: `shared_library`

**输出产物**: `libsurface.so`

**用途**: 核心功能库，提供 Surface/Buffer 管理能力

#### 源文件 (8个)

| 源文件 | 功能 |
|--------|------|
| `buffer_client_producer.cpp` | 跨进程 Producer 代理 |
| `buffer_manager.cpp` | Buffer 内存管理 |
| `buffer_queue.cpp` | Buffer 队列核心 |
| `buffer_queue_consumer.cpp` | Consumer 端封装 |
| `buffer_queue_producer.cpp` | Producer 端实现 + IPC 处理 |
| `surface.cpp` | Surface 工厂方法 |
| `surface_buffer_impl.cpp` | Buffer 实现 + 序列化 |
| `surface_impl.cpp` | Surface 实现 + IPC 处理 |

#### 私有包含路径

```gn
include_dirs = [
    "frameworks",                                    # 私有头文件
    "//drivers/peripheral/base",                    # 驱动基础接口
    "//drivers/peripheral/display/interfaces/include", # 显示 HAL
]
```

#### 公共依赖 (public_deps)

```gn
public_deps = [ "//foundation/graphic/graphic_utils_lite:utils_lite" ]
```

**传播效果**: 依赖 `surface` 的目标会自动获得 `utils_lite` 及其头文件路径

#### 私有依赖 (deps)

| 依赖 | 用途 |
|------|------|
| `//drivers/peripheral/display/hal:hdi_display` | 显示 HAL 接口 |
| `//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single` | IPC 通信 |
| `//third_party/bounds_checking_function:libsec_shared` | 安全函数库 |

#### 编译标志

| 标志 | 值 | 说明 |
|------|-----|------|
| `cflags` | `-fPIC` | 位置无关代码 (共享库必需) |
| `cflags` | `-Wall` | 启用所有警告 |
| `ldflags` | `-ldisplay_gfx` | 链接显示图形库 |
| `ldflags` | `-ldisplay_gralloc` | 链接图形内存分配器 |
| `ldflags` | `-ldisplay_layer` | 链接显示图层库 |

### 3.2 surface_lite (lite_component)

**类型**: `lite_component`

**用途**: OpenHarmony 组件注册目标

```gn
lite_component("surface_lite") {
  features = [ ":surface" ]
  public_deps = features
}
```

**特点**:
- 包装 `surface` 目标，用于 OHOS 组件系统
- `features` 指定包含的子目标
- `public_deps` 将依赖传播给使用者

### 3.3 surface_lite_ndk (ndk_lib)

**类型**: `ndk_lib`

**用途**: NDK (Native Development Kit) 导出

```gn
ndk_lib("surface_lite_ndk") {
  lib_extension = ".so"
  deps = [ ":surface" ]
  head_files = [ "interfaces/kits" ]
}
```

**输出**:
- 库文件: `libsurface.so`
- 头文件: `interfaces/kits/` 下的所有头文件

**用途**: 供应用开发者使用 Native API

### 3.4 surface_public_config (config)

**类型**: `config`

**用途**: 公共头文件路径配置

```gn
config("surface_public_config") {
  include_dirs = [
    "interfaces/innerkits",       # 内部接口头文件
    "interfaces/kits",            # 对外接口头文件
    "//foundation/graphic/graphic_utils_lite/interfaces/kits",  # 工具库头文件
  ]
}
```

**应用**: 被 `surface` 目标通过 `public_configs` 引用，传播给依赖者

---

## 4. 测试 BUILD.gn

### 4.1 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/graphic/surface_lite/test/BUILD.gn`

### 4.2 完整配置

```gn
# Copyright (c) 2020-2021 Huawei Device Co., Ltd.
# ...

import("//build/lite/config/test.gni")

# ============================================
# Target 1: 测试组
# ============================================
group("surface_lite_test") {
  if (ohos_build_type == "debug") {
    deps = [ ":surface_lite_unittest_door" ]
  }
}

# ============================================
# Target 2: 单元测试 (仅 Debug 模式)
# ============================================
if (ohos_build_type == "debug") {
  unittest("surface_lite_unittest_door") {
    output_extension = "bin"
    output_dir = "$root_out_dir/test/unittest/graphic"
    sources = [ "unittest/graphic_surface_test.cpp" ]
    deps = [
      "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
      "//foundation/graphic/surface_lite:surface",
    ]
  }
}
```

### 4.3 Target 详解

#### surface_lite_test (group)

**类型**: `group`

**用途**: 测试入口，仅在 Debug 模式下包含单元测试

#### surface_lite_unittest_door (unittest)

**类型**: `unittest`

**条件**: `ohos_build_type == "debug"`

**输出**:
- 文件名: `surface_lite_unittest_door.bin`
- 路径: `$root_out_dir/test/unittest/graphic/`

**源文件**:
- `unittest/graphic_surface_test.cpp`

**依赖**:
- `ipc:ipc_single` - IPC 模块
- `surface_lite:surface` - 被测库

---

## 5. 依赖关系图

### 5.1 Target 依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                      surface_lite (lite_component)              │
│                           (组件入口)                             │
└─────────────────────────────┬───────────────────────────────────┘
                              │ features
┌─────────────────────────────▼───────────────────────────────────┐
│                        surface (shared_library)                 │
│                         (核心功能库)                             │
│                                                                 │
│   ┌─────────────────┐    ┌──────────────────┐                  │
│   │ public_configs  │───▶│ surface_public   │                  │
│   │                 │    │ _config          │                  │
│   └─────────────────┘    └──────────────────┘                  │
│                                                                 │
│   ┌─────────────────┐    ┌──────────────────┐                  │
│   │ public_deps     │───▶│ graphic_utils_   │                  │
│   │                 │    │ lite:utils_lite   │                  │
│   └─────────────────┘    └──────────────────┘                  │
│                                                                 │
│   ┌─────────────────┐                                           │
│   │ deps            │───▶ display/hal:hdi_display               │
│   │                 │───▶ ipc:ipc_single                        │
│   │                 │───▶ bounds_checking_function              │
│   └─────────────────┘                                           │
│                                                                 │
│   ┌─────────────────┐                                           │
│   │ ldflags         │───▶ -ldisplay_gfx                         │
│   │                 │───▶ -ldisplay_gralloc                     │
│   │                 │───▶ -ldisplay_layer                       │
│   └─────────────────┘                                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ deps
┌─────────────────────────────▼───────────────────────────────────┐
│                   surface_lite_ndk (ndk_lib)                    │
│                      (NDK 导出包)                                │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 模块间依赖

```
surface_lite
    ├── 依赖:
    │   ├── graphic_utils_lite (public)
    │   ├── ipc (private)
    │   ├── hdi_display (private)
    │   └── bounds_checking_function (private)
    │
    └── 被依赖:
        ├── window_window_manager_lite
        └── arkui_ui_lite
```

---

## 6. 编译命令

### 6.1 完整编译

```bash
# 编译 surface_lite 组件
hb build surface_lite

# 或编译整个系统
hb build
```

### 6.2 仅编译库

```bash
# 使用 ninja 直接编译 (在 out 目录)
ninja -C out/{product_name} surface
```

### 6.3 编译测试

```bash
# 编译测试 (仅 Debug 模式)
hb build surface_lite_test

# 运行测试
./out/{product}/test/unittest/graphic/surface_lite_unittest_door.bin
```

---

## 7. 输出产物

### 7.1 产物清单

| Target | 产物类型 | 产物名称 | 输出路径 |
|--------|----------|----------|----------|
| surface | 共享库 | `libsurface.so` | `$root_out_dir/libs/` |
| surface_lite_unittest_door | 可执行文件 | `surface_lite_unittest_door.bin` | `$root_out_dir/test/unittest/graphic/` |

### 7.2 产物详情

#### libsurface.so

**类型**: ELF shared library

**依赖**:
- `libdisplay_gfx.so`
- `libdisplay_gralloc.so`
- `libdisplay_layer.so`
- `libipc_single.so`
- `libsec_shared.so`
- `libutils_lite.so`

**导出符号**: (部分)
```
OHOS::Surface::CreateSurface()
OHOS::Surface::~Surface()
OHOS::SurfaceImpl::SurfaceImpl()
OHOS::BufferQueue::BufferQueue()
...
```

### 7.3 安装路径

在系统镜像中的位置:
```
system/lib/libsurface.so
```

---

## 8. bundle.json 组件配置

### 8.1 完整配置

```json
{
  "name": "@ohos/surface_lite",
  "description": "Graphic shared memory",
  "version": "3.1",
  "license": "Apache License 2.0",
  "pubiishAs": "code-segment",
  "segment": {
    "destPath": "foundation/graphic/surface_lite"
  },
  "dirs": {},
  "scripts": {},
  "component": {
    "name": "surface_lite",
    "subsystem": "graphic",
    "feature": [],
    "adapted_system_type": [ "small" ],
    "rom": "110KB",
    "ram": "~50KB",
    "deps": {
      "third_party": [
        "bounds_checking_function"
      ],
      "components": [
        "drivers_peripheral_display",
        "graphic_utils_lite",
        "ipc"
      ]
    },
    "build": {
      "sub_component": [
        "//foundation/graphic/surface_lite:surface_lite",
        "//foundation/graphic/surface_lite/test:surface_lite_test"
      ],
      "inner_kits": [],
      "test": []
    }
  }
}
```

### 8.2 关键字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/surface_lite` | npm 风格的包名 |
| `component.name` | `surface_lite` | 组件名 |
| `component.subsystem` | `graphic` | 所属子系统 |
| `component.adapted_system_type` | `["small"]` | 适用系统类型 |
| `component.rom` | `110KB` | ROM 占用 |
| `component.ram` | `~50KB` | RAM 占用 |
| `component.deps.third_party` | `["bounds_checking_function"]` | 第三方依赖 |
| `component.deps.components` | `["drivers_peripheral_display", ...]` | 组件依赖 |
| `component.build.sub_component` | GN 目标路径 | 构建入口 |

---

## 9. 构建变量

### 9.1 预定义变量

| 变量 | 说明 |
|------|------|
| `$root_out_dir` | 输出根目录 |
| `$target_out_dir` | 当前目标输出目录 |
| `ohos_build_type` | 构建类型 (debug/release) |

### 9.2 条件编译

```gn
# Debug 模式特定代码
if (ohos_build_type == "debug") {
  # 测试相关配置
}
```

---

## 10. 常见问题

### 10.1 编译错误

**错误**: `undefined reference to 'GrallocInitialize'`

**原因**: 缺少显示驱动依赖

**解决**: 确保 `drivers_peripheral_display` 组件已编译

### 10.2 链接错误

**错误**: `cannot find -ldisplay_gralloc`

**原因**: 链接库路径未配置

**解决**: 检查 `ldflags` 和系统库路径

### 10.3 头文件找不到

**错误**: `'surface.h' file not found`

**原因**: 未正确依赖 `surface` 目标

**解决**: 在 `deps` 或 `public_deps` 中添加 `//foundation/graphic/surface_lite:surface`

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
