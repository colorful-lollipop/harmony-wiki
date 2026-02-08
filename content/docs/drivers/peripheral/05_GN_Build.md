# GN 构建配置

本文档说明 `drivers/peripheral` 的 GN 构建系统配置。

## 5.1 构建系统概述

### 5.1.1 构建工具链

- **GN (Generate Ninja)**：元构建系统，生成 Ninja 构建文件
- **Ninja**：实际构建工具
- **hb (OpenHarmony Build)**：OpenHarmony 顶层构建工具

### 5.1.2 配置文件结构

```
peripheral/
├── BUILD.gn              # 模块构建入口
├── module.gni           # 模块配置片段
├── audio/
│   ├── BUILD.gn         # audio 模块构建
│   └── audio.gni        # audio 配置
├── input/
│   ├── BUILD.gn         # input 模块构建
│   └── *.gni            # 其他配置
├── sensor/
│   ├── BUILD.gn
│   └── sensor.gni
└── ...
```

---

## 5.2 关键 GN 文件

### 5.2.1 根目录 BUILD.gn

**证据位置**: `/Volumes/lexar/code/d/work/oh/drivers/peripheral/audio/BUILD.gn`

```gn
# audio/BUILD.gn

# 导入公共配置
import("//drivers/peripheral/audio/audio.gni")

# 构建 libhdi_audio 库
ohos_shared_library("hdi_audio") {
  sources = [
    "hdi_service/primary/audio_manager_service.c",
    "hdi_service/primary/audio_manager_driver.c",
    # ... 更多源文件
  ]

  include_dirs = [
    "interfaces/include",
    "interfaces/2.0/include",
    "hal/hdi_passthrough/include",
    "hal/hdi_binder/server/include",
    # ... 更多目录
  ]

  deps = [
    "//drivers/framework/core/common:hdf_core",
    "//drivers/framework/core/ability:hdf_ability",
    "//drivers/framework/core/host:hdf_host",
    "//ipheral/basedrivers/per:buffer_handle",
  ]

  public_deps = [
    "//drivers/peripheral/audio/interfaces:audio_interface",
  ]

  defines = [
    "__UNIX_THREAD_SUPPORT__",
    "__OHOS__",
  ]

  cflags = [
    "-Wall",
    "-Werror",
  ]

  # 标签，用于选择编译
  tags = [ "drivers", "peripheral", "audio" ]

  # 移除不需要的标签
  remove_tags = []
}
```

---

### 5.2.2 模块配置 .gni 文件

**证据位置**: `/Volumes/lexar/code/d/work/oh/drivers/peripheral/audio/audio.gni`

```gn
# audio.gni - 模块配置

# 模块名
peripheral_audio_module_name = "hdi_audio"

# 模块版本
peripheral_audio_module_version = "2.0"

# 依赖的接口定义
peripheral_audio_public_deps = [
  "//drivers/peripheral/audio/interfaces:audio_interface_v2_0",
  "//drivers/peripheral/audio/interfaces:audio_interface",
]

# 内部配置
peripheral_audio_cflags = []
if (ohos_kernel_type == "linux") {
  peripheral_audio_cflags += [ "__LINUX_USER__" ]
} else if (ohos_kernel_type == "liteos_m") {
  peripheral_audio_cflags += [ "__LITEOS__" ]
}
```

---

## 5.3 Target 类型

### 5.3.1 常用 Target 类型

| 类型 | GN 关键字 | 用途 |
|------|-----------|------|
| **动态库** | `ohos_shared_library` | HDI 服务实现 |
| **静态库** | `ohos_static_library` | HAL 实现、工具库 |
| **头文件** | `ohos_headers` | 接口头文件包 |
| **配置** | `ohos_source_set` | 轻量级源文件集合 |

### 5.3.2 头文件包 Target

**证据位置**: `audio/interfaces/BUILD.gn`

```gn
# interfaces/BUILD.gn

# 导出 audio_interface 头文件包
ohos_headers("audio_interface") {
  visibility = [ "*" ]
  sources = [
    "include/audio_manager.h",
    "include/audio_adapter.h",
    "include/audio_render.h",
    "include/audio_capture.h",
    "include/audio_types.h",
    "include/audio_volume.h",
    "include/audio_control.h",
    "include/audio_events.h",
    "include/audio_attribute.h",
    "include/audio_scene.h",
  ]

  include_dirs = [ "include" ]

  deps = [
    "//drivers/peripheral/base:buffer_handle",
  ]
}
```

---

## 5.4 主要 Targets 列表

### 5.4.1 Audio 模块 Targets

| Target | 类型 | 输出 | 主要功能 | 证据位置 |
|--------|------|------|----------|----------|
| `hdi_audio` | shared_library | `libhdi_audio.z.so` | 音频 HDI 服务 | `audio/BUILD.gn` |
| `audio_interface` | headers | - | 音频接口头文件 | `audio/interfaces/BUILD.gn` |
| `audio_interface_v2_0` | headers | - | 音频 2.0 接口 | `audio/interfaces/BUILD.gn` |

### 5.4.2 Input 模块 Targets

| Target | 类型 | 输出 | 主要功能 | 证据位置 |
|--------|------|------|----------|----------|
| `hdi_input` | shared_library | `libhdi_input.z.so` | 输入 HDI 服务 | `input/BUILD.gn` |
| `input_interface` | headers | - | 输入接口头文件 | `input/interfaces/BUILD.gn` |

### 5.4.3 Sensor 模块 Targets

| Target | 类型 | 输出 | 主要功能 | 证据位置 |
|--------|------|------|----------|----------|
| `hdi_sensor` | shared_library | `libhdi_sensor.z.so` | 传感器 HDI 服务 | `sensor/BUILD.gn` |
| `sensor_interface` | headers | - | 传感器接口头文件 | `sensor/interfaces/BUILD.gn` |

---

## 5.5 deps / public_deps / libs

### 5.5.1 依赖类型说明

| 类型 | 说明 | 传递性 |
|------|------|--------|
| `deps` | 私有依赖，仅当前 Target 使用 | 不传递 |
| `public_deps` | 公开依赖，会传递给使用者 | 传递 |
| `libs` | 系统库链接 | 不传递 |

### 5.5.2 常见依赖

```gn
# HDF 框架依赖
deps = [
  "//drivers/framework/core/common:hdf_core",
  "//drivers/framework/core/ability:hdf_ability",
  "//drivers/framework/core/host:hdf_host",
]

# 公共组件依赖
public_deps = [
  "//drivers/peripheral/base:buffer_handle",
]

# 系统库
libs = [ "libc.so", "libm.so" ]
```

---

## 5.6 defines / configs

### 5.6.1 常用宏定义

| 宏定义 | 说明 | 使用场景 |
|--------|------|----------|
| `__OHOS__` | OpenHarmony 平台标识 | 通用 |
| `__UNIX_THREAD_SUPPORT__` | Unix 线程支持 | Audio |
| `__LINUX_USER__` | Linux 用户态 | Linux 平台 |
| `__LITEOS__` | LiteOS 内核 | LiteOS 平台 |

### 5.6.2 编译器配置

```gn
# 编译器标志
cflags = [
  "-Wall",           # 开启所有警告
  "-Werror",         # 警告视为错误
  "-O2",             # 优化级别
]

# C++ 特有
cppflags = [
  "-std=c++11",
  "-fno-rtti",
  "-fexceptions",
]

# 链接标志
ldflags = [
  "-Wl,-z,relro",
  "-Wl,-z,now",
]
```

---

## 5.7 条件编译

### 5.7.1 内核类型条件编译

```gn
# 根据内核类型选择配置
if (ohos_kernel_type == "linux") {
  defines = [ "__LINUX_USER__" ]
  sources += [ "hal/linux/audio_linux.c" ]
} else if (ohos_kernel_type == "liteos_m") {
  defines = [ "__LITEOS__" ]
  sources += [ "hal/liteos/audio_liteos.c" ]
}
```

### 5.7.2 产品配置

```gn
# 根据产品配置选择
if (product_name == "phone") {
  defines += [ "ENABLE_AUDIO_HDMI" ]
} else if (product_name == "tv") {
  defines += [ "ENABLE_AUDIO_SPDIF" ]
}
```

---

## 5.8 编译产物映射

### 5.8.1 产物路径规则

```
out/{product}/{target}/drivers/peripheral/{module}/
├── lib{module}.z.so          # 动态库
├── lib{module}.a            # 静态库（如有）
└── {module}_interfaces/     # 头文件包
    └── include/
        ├── *.h
        └── ...
```

### 5.8.2 实际产物示例

| 模块 | 动态库路径 | 头文件路径 |
|------|-----------|-----------|
| Audio | `out/hispark_taurus/drivers/peripheral/audio/libhdi_audio.z.so` | `out/.../audio/interfaces/include/` |
| Input | `out/hispark_taurus/drivers/peripheral/input/libhdi_input.z.so` | `out/.../input/interfaces/include/` |
| Sensor | `out/hispark_taurus/drivers/peripheral/sensor/libhdi_sensor.z.so` | `out/.../sensor/interfaces/include/` |

---

## 5.9 构建命令

### 5.9.1 全量构建

```bash
# 使用 hb 构建整个系统
hb set
hb build

# 或单独构建 peripheral
hb build --build-target drivers_peripheral
```

### 5.9.2 单模块构建

```bash
# 构建 audio 模块
gn gen out/hispark_taurus
ninja -C out/hispark_taurus drivers/peripheral/audio:hdi_audio

# 构建 input 模块
ninja -C out/hispark_taurus drivers/peripheral/input:hdi_input

# 构建 sensor 模块
ninja -C out/hispark_taurus drivers/peripheral/sensor:hdi_sensor
```

### 5.9.3 构建产物验证

```bash
# 检查动态库
ls -la out/hispark_taurus/drivers/peripheral/audio/libhdi_audio.z.so

# 检查符号表
nm -D out/hispark_taurus/drivers/peripheral/audio/libhdi_audio.z.so | grep GetAllAdapters

# 检查依赖
ldd out/hispark_taurus/drivers/peripheral/audio/libhdi_audio.z.so
```

---

## 5.10 常见构建问题

### 5.10.1 头文件未找到

**问题**: `fatal error: xxx.h: No such file or directory`

**解决**: 检查 `include_dirs` 配置
```gn
include_dirs = [
  "interfaces/include",
  # 添加缺失的路径
]
```

### 5.10.2 链接错误

**问题**: `undefined reference to 'xxx'`

**解决**: 检查 `deps` 和 `libs` 配置
```gn
deps = [
  "//drivers/peripheral/base:buffer_handle",
]
```

### 5.10.3 条件编译错误

**问题**: 条件分支未覆盖所有平台

**解决**: 添加完整的条件分支
```gn
if (ohos_kernel_type == "linux") {
  # Linux 特定代码
} else if (ohos_kernel_type == "liteos_m") {
  # LiteOS 特定代码
} else {
  # 默认实现
  sources += [ "hal/common/audio_common.c" ]
}
```

---

**下一节**: [安全风险评审](06_Security_Review.md) - 了解安全风险
