# OH 构建适配

本文档详细介绍 libusb 在 OpenHarmony 构建系统中的集成配置，包括 GN 构建脚本、编译选项和平台特定适配。

---

## 构建系统概述

libusb 使用 OpenHarmony 的 GN 构建系统（`.gni`）进行构建，配置文件为 `BUILD.gn`。

### 构建流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    libusb 构建流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  libusb-1.0.28.tar.gz                                           │
│        │                                                       │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  install.sh 脚本处理                                    │   │
│  │  - 解压源码归档                                          │   │
│  │  - 复制源文件到 gen 目录                                │   │
│  │  - 根据平台选择源文件列表                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│        │                                                       │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  libusb_action (action)                                 │   │
│  │  - 定义输出文件列表                                       │   │
│  │  - 声明输入文件                                          │   │
│  │  - 配置构建参数                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│        │                                                       │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  libusb_source (source_set)                              │   │
│  │  - 应用编译配置                                           │   │
│  │  - 编译源文件                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│        │                                                       │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  libusb (shared_library)                                 │   │
│  │  - 链接生成动态库                                         │   │
│  │  - 配置公共头文件                                         │   │
│  │  - 设置安装路径                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## BUILD.gn 结构详解

### 文件位置

```
//third_party/libusb/BUILD.gn
```

### 关键配置段

#### 1. 路径定义

```gn
LIBUSB_DIR = rebase_path("//third_party/libusb")
```

**说明**: 定义 libusb 根目录路径，用于后续配置引用。

#### 2. 源码处理 Action

```gn
action("libusb_action") {
  script = "//third_party/libusb/install.sh"
  outputs = [
    "${target_gen_dir}/libusb-1.0.28/libusb/core.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/descriptor.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/hotplug.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/io.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/sync.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/strerror.c",
  ]
  # ... 平台特定源文件
}
```

**功能**: 声明构建输出（预处理后的源文件）

#### 3. 编译配置

```gn
config("libusb_config") {
  include_dirs = [
    get_label_info(":libusb_action", "target_gen_dir") +
        "/libusb-1.0.28/libusb",
    get_label_info(":libusb_action", "target_gen_dir") +
        "/libusb-1.0.28/libusb/os",
  ]
  cflags = [
    "-U__ANDROID__",
    "-UUSE_UDEV",
    "-Wno-#warnings",
    "-Wno-error=sign-compare",
    "-Wno-error=switch",
    "-Wno-error=pragma-pack",
  ]
}
```

**说明**: 定义编译器的包含路径和标志

---

## 平台特定配置

### Linux/OHOS 平台配置

```gn
if (is_linux || is_ohos) {
  include_dirs += [ "${LIBUSB_DIR}/linux" ]
  cflags += [ "-DPLATFORM_POSIX" ]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `include_dirs/linux` | `${LIBUSB_DIR}/linux` | OHOS 特定头文件目录 |
| `PLATFORM_POSIX` | 定义 | 标识 POSIX 平台环境 |
| `is_linux \|\| is_ohos` | 条件 | Linux 和 OHOS 共用配置 |

**源文件选择**:
```gn
outputs += [
  "${target_gen_dir}/libusb-1.0.28/libusb/os/events_posix.c",
  "${target_gen_dir}/libusb-1.0.28/libusb/os/linux_netlink.c",
  "${target_gen_dir}/libusb-1.0.28/libusb/os/linux_usbfs.c",
  "${target_gen_dir}/libusb-1.0.28/libusb/os/threads_posix.c",
]
```

### macOS 平台配置

```gn
if (is_mac) {
  outputs += [
    "${target_gen_dir}/libusb-1.0.28/libusb/os/darwin_usb.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/events_posix.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/threads_posix.c",
  ]
  cflags += [
    "-Wno-unused-parameter",
    "-Wno-deprecated-declarations",
    "-DPLATFORM_POSIX",
  ]
  frameworks = [
    "CoreFoundation.framework",
    "IOKit.framework",
    "Security.framework",
  ]
  libs = [ "objc" ]
}
```

### Windows 平台配置

```gn
if (is_mingw || is_win) {
  outputs += [
    "${target_gen_dir}/libusb-1.0.28/libusb/os/events_windows.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/threads_windows.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/windows_common.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/windows_usbdk.c",
    "${target_gen_dir}/libusb-1.0.28/libusb/os/windows_winusb.c",
  ]
  cflags += [
    "-Werror",
    "-Wno-unused-function",
    "-Wno-unused-parameter",
    "-DPLATFORM_WINDOWS",
  ]
  include_dirs += [ "${LIBUSB_DIR}/windows" ]
}
```

---

## 编译选项详解

### 全局编译标志

| 标志 | 值 | 说明 |
|------|-----|------|
| `-U__ANDROID__` | 取消定义 | 禁用 Android 特定代码路径 |
| `-UUSE_UDEV` | 取消定义 | 禁用 udev 设备管理集成 |
| `-Wno-#warnings` | 警告抑制 | 忽略 # 符号相关警告 |
| `-Wno-error=sign-compare` | 警告降级 | 符号比较警告不视为错误 |
| `-Wno-error=switch` | 警告降级 | switch 语句警告不视为错误 |
| `-Wno-error=pragma-pack` | 警告降级 | pragma pack 警告不视为错误 |

**配置目的**: 确保 libusb 在 OHOS 环境下编译通过，不受 Android 或 udev 相关代码影响。

### OHOS 特定定义

| 宏定义 | 说明 |
|--------|------|
| `__OHOS__` | 由编译器隐式定义，标识 OpenHarmony 系统 |
| `PLATFORM_POSIX` | 标识 POSIX 兼容平台 |

---

## 动态库配置

```gn
ohos_shared_library("libusb") {
  deps = [ ":libusb_source" ]
  public_configs = [ ":libusb_public_config" ]
  output_name = "libusb_shared"
  install_images = [ "system" ]
  subsystem_name = "thirdparty"
  part_name = "libusb"
  innerapi_tags = [ "chipsetsdk_sp" ]
}
```

### 配置参数说明

| 参数 | 值 | 说明 |
|------|-----|------|
| `output_name` | `libusb_shared` | 输出动态库名称 |
| `install_images` | `["system"]` | 安装到 system 分区 |
| `subsystem_name` | `thirdparty` | 所属子系统 |
| `part_name` | `libusb` | 组件名称 |
| `innerapi_tags` | `["chipsetsdk_sp"]` | API 标签（内部） |

### 输出文件

| 文件类型 | 文件名 | 路径 |
|---------|-------|------|
| 动态库 | `liblibusb_shared.so` | system/lib/ |
| 头文件 | `libusb.h` | system/include/libusb/ |
| 头文件 | `libusbi.h` | system/include/libusb/ |

---

## 头文件配置

```gn
config("libusb_public_config") {
  include_dirs = [
    get_label_info(":libusb_action", "target_gen_dir") + "/libusb-1.0.28",
    get_label_info(":libusb_action", "target_gen_dir") +
        "/libusb-1.0.28/libusb",
  ]
}
```

**对外暴露头文件**:

| 头文件 | 路径 | 用途 |
|-------|------|------|
| `libusb.h` | `libusb/libusb.h` | 公共 API 头文件 |
| `libusbi.h` | `libusb/libusbi.h` | 内部接口头文件 |

---

## install.sh 脚本分析

### 脚本位置

```
//third_party/libusb/install.sh
```

### 功能说明

```bash
#!/bin/bash
# 安装脚本功能：
# 1. 解压 libusb-1.0.28.tar.gz
# 2. 将源文件复制到 target_gen_dir
# 3. 根据平台条件过滤源文件
```

### 输入输出

| 类型 | 文件 | 说明 |
|------|------|------|
| 输入 | `libusb-1.0.28.tar.gz` | 预编译的源码归档 |
| 输出 | `libusb/core.c` 等 | 平台特定的源文件列表 |

---

## 构建产物

### 动态库信息

```bash
$ file system/lib/libusb_shared.so
ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked
```

### 库依赖

```bash
$ readelf -d system/lib/libusb_shared.so
Dynamic section at offset 0x1c80 contains 24 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 NEEDED               Shared library: [libc.so]
```

**依赖说明**: libusb 仅依赖系统 libc，不依赖其他第三方库。

---

## 使用方式

### 作为子系统依赖

```gn
deps = [
  "//third_party/libusb:libusb",
]
```

### 作为静态库依赖

```gn
deps = [
  "//third_party/libusb:libusb_source",
]
```

---

## 常见问题

### Q1: 如何启用调试日志？

**答**: 在编译时添加 `-DLIBUSB_DEBUG=4` 标志。

### Q2: 如何排除特定平台代码？

**答**: 通过平台条件判断控制源文件选择。

### Q3: 如何更新 libusb 版本？

**答**:
1. 替换 `libusb-1.0.28.tar.gz` 为新版本归档
2. 更新 `outputs` 中的版本号路径
3. 重新应用 Patch
4. 验证构建和测试

---

*文档版本: 1.0*
*最后更新: 2026-02-07*
