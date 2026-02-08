# 编译产物

## 产物概述

device_qemu 的编译产物主要是内核模块和静态库，这些产物会被链接到 OpenHarmony 内核镜像中。

### 产物分类

| 产物类型 | 来源 | 最终去向 |
|----------|------|----------|
| **内核模块 (.o/.a)** | `kernel_module()` | 链接到内核镜像 |
| **HDF 驱动 (.o/.a)** | `hdf_driver()` | 链接到内核镜像 |
| **静态库 (.a)** | source_set | 作为链接输入 |

## 驱动模块产物

### char 模块

| 源文件 | 构建目标 | 产物类型 | 证据位置 |
|--------|----------|----------|----------|
| `char/mmz/mmz.c` | `char/mmz` | 静态库 | `drivers/char/mmz/BUILD.gn:27` |
| `char/` | `char` | 内核模块 | `drivers/char/BUILD.gn:23-31` |

**产物映射**:

```
drivers/char/mmz/BUILD.gn:27
┌─────────────────────────────────────────────┐
│  kernel_module(module_name) {               │
│    sources = [ "mmz.c" ]                    │  ──→  libmmz.o → libmmz.a
│  }                                           │
└─────────────────────────────────────────────┘

drivers/char/BUILD.gn:23-31
┌─────────────────────────────────────────────┐
│  kernel_module(module_name) {              │
│    deps = [ "mmz" ]                         │  ──→  libchar.a
│  }                                           │       (包含 mmz)
└─────────────────────────────────────────────┘
```

### uart 模块

| 源文件 | 构建目标 | 产物类型 | 证据位置 |
|--------|----------|----------|----------|
| `uart/uart.c` | `uart` | HDF 驱动 | `drivers/uart/BUILD.gn:27-33` |
| `uart/uart_pl011.c` | `uart` | HDF 驱动 | `drivers/uart/BUILD.gn:28` |

**产物映射**:

```
drivers/uart/BUILD.gn:27-33
┌─────────────────────────────────────────────┐
│  hdf_driver(module_name) {                  │
│    sources = [                              │
│      "uart.c",        ──→ libuart.o        │
│      "uart_pl011.c",  ──→ libuart_pl011.o  │
│    ]                                        │  ──→  libhdf_uart.a
│  }                                           │
└─────────────────────────────────────────────┘
```

### virtio 模块

| 源文件 | 构建目标 | 产物类型 | 证据位置 |
|--------|----------|----------|----------|
| `virtio/fakesdio.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:32` |
| `virtio/virtblock.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:33` |
| `virtio/virtgpu.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:34` |
| `virtio/virtinput.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:35` |
| `virtio/virtmmio.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:36` |
| `virtio/virtnet.c` | `virtio` | HDF 驱动 | `drivers/virtio/BUILD.gn:37` |
| `virtio/virtrng.c` | `virtio` | HDF 驱动 (条件) | `drivers/virtio/BUILD.gn:40-41` |

**产物映射**:

```
drivers/virtio/BUILD.gn:31-44
┌─────────────────────────────────────────────┐
│  hdf_driver(module_name) {                  │
│    sources = [                              │
│      "fakesdio.c",   ──→ libfakesdio.o     │
│      "virtblock.c",  ──→ libvirtblock.o    │
│      "virtgpu.c",    ──→ libvirtgpu.o      │
│      "virtinput.c",  ──→ libvirtinput.o    │
│      "virtmmio.c",   ──→ libvirtmmio.o     │
│      "virtnet.c",    ──→ libvirtnet.o      │
│    ]                                        │
│    if (defined(LOSCFG_HW_RANDOM_ENABLE)) {  │
│      sources += [ "virtrng.c" ] ──→ ...    │
│    }                                        │  ──→  libvirtio.a
│  }                                           │
└─────────────────────────────────────────────┘
```

## 产物清单汇总

### 按模块分类

| 模块 | 源文件数 | 构建目标 | 产物 | 条件依赖 |
|------|----------|----------|------|----------|
| **char** | 1 | kernel_module | libchar.a | - |
| **char/mmz** | 1 | kernel_module | libmmz.a | - |
| **uart** | 2 | hdf_driver | libhdf_uart.a | `LOSCFG_DRIVERS_HDF_PLATFORM_UART` |
| **virtio** | 6-7 | hdf_driver | libvirtio.a | `LOSCFG_HW_RANDOM_ENABLE` |

### 产物依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                    Link Dependency Graph                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   libvirtio.a ┐                                             │
│   libchar.a   ┤                                             │
│   libhdf_uart.a┤  ──→  OpenHarmony Kernel Image             │
│   libmmz.a    ┘                                             │
│                                                              │
│  Dependencies:                                               │
│  ┌──────────────┐                                           │
│  │ libchar.a    │ depends on libmmz.a                        │
│  └──────────────┘                                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 安装路径

### 构建产物安装

device_qemu 编译产物会安装到 OpenHarmony 构建系统的输出目录：

| 产物类型 | 预期路径模式 | 说明 |
|----------|--------------|------|
| **内核模块** | `out/<platform>/drivers/` | 链接到内核镜像 |
| **静态库** | `out/<platform>/lib/` | 作为链接输入 |
| **符号文件** | `out/<platform>/*.dbg` | 调试信息 |

### 平台对应关系

| 平台 | 输出目录模式 |
|------|--------------|
| ARM 虚拟 | `out/arm_virt/` |
| ARM MPS2-AN386 | `out/arm_mps2_an386/` |
| ARM MPS3-AN547 | `out/arm_mps3_an547/` |
| RISC-V 32位 | `out/riscv32_virt/` |
| RISC-V 64位 | `out/riscv64_virt/` |
| x86_64 虚拟 | `out/x86_64_virt/` |
| ESP32 | `out/esp32/` |
| SmartL_E802 | `out/SmartL_E802/` |

### 运行时加载关系

```
┌─────────────────────────────────────────────────────────────┐
│                    Runtime Loading                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   QEMU Startup                                               │
│       │                                                      │
│       ▼                                                      │
│   ┌─────────────────────────────────────────────────────┐   │
│   │   Load Kernel Image (OHOS_Image)                     │   │
│   │   - Contains: libvirtio.a + libchar.a + libmmz.a     │   │
│   │   - Optionally: libhdf_uart.a (if UART enabled)      │   │
│   └─────────────────────────────────────────────────────┘   │
│       │                                                      │
│       ▼                                                      │
│   ┌─────────────────────────────────────────────────────┐   │
│   │   HDF Driver Initialization                          │   │
│   │   - Register VirtIO drivers (block/net/gpu/input)   │   │
│   │   - Register UART driver (if enabled)               │   │
│   │   - Initialize MMZ (if char device enabled)         │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 调试符号

### DWARF 调试信息

编译产物包含 DWARF 调试信息，支持以下调试操作：

| 调试功能 | 工具 | 用途 |
|----------|------|------|
| **源码级调试** | GDB / LLDB | 断点、单步、变量查看 |
| **反汇编** | objdump | 汇编分析 |
| **符号表** | nm | 符号查找 |
| **调用栈** | addr2line | 地址到源码映射 |

### 符号文件

| 符号类型 | 说明 | 示例 |
|----------|------|------|
| **函数符号** | 驱动入口和导出函数 | `VirtIoInit`, `UartInit` |
| **数据符号** | 全局变量和结构体 | `virtio_dev`, `uart_config` |
| **调试符号** | 行号信息 | *.dbg 文件 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [GN 构建](04_GN_Build.md) | 构建配置详解 |
| [支持的平台](07_Platforms.md) | 各平台构建配置 |
| [常见问题](08_Troubleshooting.md) | 构建问题排查 |
| [附录：调用链](appendix/Callgraphs.md) | 关键调用链 |
