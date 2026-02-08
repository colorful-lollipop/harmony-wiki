# 架构设计

## 整体架构

device_qemu 采用分层架构设计，与 OpenHarmony 系统紧密集成：

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统架构                          │
├─────────────────────────────────────────────────────────────────┤
│  User Space                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Apps       │ │ Services    │ │ HDF Utils   │ │ VFS        ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘│
├─────────────────────────────────────────────────────────────────┤
│  Kernel Space (LiteOS)                                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    HDF Core Framework                       ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐││
│  │  │ Driver      │ │ Device      │ │ Host Node Manager      │││
│  │  │ Manager     │ │ Manager     │ │ (udev integration)      │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────────┘││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│  Device Simulation Layer (device_qemu)                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      ││
│  │  │ VirtIO   │  │  UART    │  │  CHAR   │  │  MMZ     │      ││
│  │  │ Drivers  │  │  Driver  │  │ Drivers │  │  Driver  │      ││
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│  QEMU Emulation Layer                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  CPU │ Memory │ VirtIO MMIO │ UART │ Timer │ GIC │ ...     ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│  Host OS (Linux/macOS)                                          │
└─────────────────────────────────────────────────────────────────┘
```

## 驱动架构

### HDF 驱动模型

device_qemu 中的驱动遵循 OpenHarmony HDF (Hardware Driver Foundation) 框架：

**证据**: `drivers/virtio/BUILD.gn:29`
```gn
import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")

hdf_driver(module_name) {
  sources = [ ... ]
  include_dirs = [ ... ]
}
```

### 驱动类型

| 驱动类型 | 模块 | 实现方式 | 说明 |
|----------|------|----------|------|
| **HDF Driver** | `uart/`, `virtio/` | `hdf_driver()` | 使用 HDF 框架注册 |
| **Kernel Module** | `char/` | `kernel_module()` | 内核模块方式加载 |

**证据**: `drivers/char/BUILD.gn:23-26`
```gn
module_switch = defined(LOSCFG_DRIVERS_PLATFORM_CHAR_DEVICE)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  deps = [ "mmz" ]
}
```

## VirtIO 设备架构

### VirtIO 设备栈

```
┌─────────────────────────────────────────────────────────────┐
│                     VirtIO Device Stack                      │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │              OpenHarmony VirtIO Drivers                │  │
│  │  virtblock │ virtnet │ virtgpu │ virtinput │ virtrng │  │
│  └───────────────────────────────────────────────────────┘  │
│                            │                                │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              VirtIO Protocol Layer                     │  │
│  │       virtqueue │ descriptors │ available/used rings   │  │
│  └───────────────────────────────────────────────────────┘  │
│                            │                                │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              VirtIO MMIO Transport                       │  │
│  │     (Memory-mapped I/O register access)                │  │
│  └───────────────────────────────────────────────────────┘  │
│                            │                                │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              QEMU VirtIO Backend                         │  │
│  │   block.img │ tap0 │ display │ input │ /dev/urandom   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### VirtIO 驱动列表

| 驱动 | 文件 | 功能 | 证据位置 |
|------|------|------|----------|
| **virtblock** | `virtblock.c` | 虚拟块设备 (磁盘) | `drivers/virtio/BUILD.gn:33` |
| **virtnet** | `virtnet.c` | 虚拟网络接口卡 | `drivers/virtio/BUILD.gn:35` |
| **virtgpu** | `virtgpu.c` | 虚拟 GPU 显示输出 | `drivers/virtio/BUILD.gn:34` |
| **virtinput** | `virtinput.c` | 虚拟键盘/鼠标 | `drivers/virtio/BUILD.gn:36` |
| **virtrng** | `virtrng.c` | 虚拟随机数生成器 | `drivers/virtio/BUILD.gn:40-41` |
| **virtmmio** | `virtmmio.c` | VirtIO MMIO 基础框架 | `drivers/virtio/BUILD.gn:37` |

## 数据流

### 块设备读写流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    Block I/O Data Flow                           │
├─────────────────────────────────────────────────────────────────┤
│  1. User Space                                                   │
│     [File I/O] → [VFS] → [Block I/O Subsystem]                   │
│                                                                  │
│  2. HDF Layer                                                    │
│     [HDF Block Driver] → [VirtIO Protocol] → [Virtqueue]         │
│                                                                  │
│  3. VirtIO Transport                                             │
│     [Descriptor] → [Available Ring] → [QEMU Backend]             │
│                                                                  │
│  4. QEMU Emulation                                               │
│     [I/O Request] → [Host File System] → [Return Completion]    │
└─────────────────────────────────────────────────────────────────┘
```

### 网络数据流

```
┌─────────────────────────────────────────────────────────────────┐
│                    Network I/O Data Flow                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│  │  App     │──▶│  LWIP    │──▶│ VirtIO   │──▶│  QEMU    │    │
│  │          │   │  Stack   │   │  Net     │   │  TAP     │    │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘    │
│                                                        │        │
│                                                        ▼        │
│                                                ┌──────────────┐ │
│                                                │ Host Network │ │
│                                                │ (eth0/WiFi)  │ │
│                                                └──────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 线程模型

### 驱动线程模型

device_qemu 驱动在 LiteOS 内核中运行，遵循内核线程模型：

```
┌─────────────────────────────────────────────────────────────┐
│                    Thread Model                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    LiteOS Kernel                          ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              ││
│  │  │  Task     │  │  Task     │  │  ISR     │              ││
│  │  │ (Thread)  │  │ (Thread)  │  │ Handler  │              ││
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘              ││
│  │       │              │             │                     ││
│  │       ▼              ▼             ▼                     ││
│  │  ┌─────────────────────────────────────────────────────┐  ││
│  │  │              Driver Entry Points                    │  ││
│  │  │   Init() │ Dispatch() │ Probe() │ Release()        │  ││
│  │  └─────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### VirtIO 中断处理

```
┌─────────────────────────────────────────────────────────────┐
│                    VirtIO Interrupt Flow                     │
├─────────────────────────────────────────────────────────────┤
│  QEMU Hardware                                               │
│       │ (Raise IRQ)                                          │
│       ▼                                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  GIC / PLIC Interrupt Controller                      │    │
│  └─────────────────────────────────────────────────────┘    │
│       │ (Deliver to CPU)                                    │
│       ▼                                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  LiteOS Interrupt Handler                             │    │
│  │   - Save context                                      │    │
│  │   - Disable IRQ                                       │    │
│  │   - Schedule bottom half / task                      │    │
│  └─────────────────────────────────────────────────────┘    │
│       │                                                     │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  VirtIO Driver ISR                                    │    │
│  │   - Read interrupt status                            │    │
│  │   - Process virtqueue (available ring)               │    │
│  │   - Signal waiters (semaphore/event)                  │    │
│  │   - Acknowledge interrupt                             │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 设备注册与发现

### HDF 设备注册流程

```
┌─────────────────────────────────────────────────────────────┐
│                    Device Registration Flow                  │
├─────────────────────────────────────────────────────────────┤
│  1. Driver Entry                                              │
│     [module_init] → [HDF Driver Register]                    │
│                                                                  │
│  2. Device Manager                                             │
│     [Match Device ID] → [Load Driver] → [Probe Device]        │
│                                                                  │
│  3. Device Initialization                                     │
│     [Allocate Resources] → [Configure Hardware]                │
│                            → [Register Interfaces]             │
│                                                                  │
│  4. Ready for I/O                                             │
│     [Enable IRQ] → [Start Worker Thread]                      │
└─────────────────────────────────────────────────────────────┘
```

**证据**: `drivers/uart/BUILD.gn:27-34`
```gn
module_switch = defined(LOSCFG_DRIVERS_HDF_PLATFORM_UART)
module_name = "hdf_uart"
hdf_driver(module_name) {
  sources = [
    "uart.c",
    "uart_pl011.c",
  ]
}
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](01_Project_Overview.md) | 项目定位与核心能力 |
| [目录结构](02_Directory_Structure.md) | 详细目录结构 |
| [GN 构建](04_GN_Build.md) | 构建配置详解 |
| [支持的平台](07_Platforms.md) | 各平台详细配置 |
| [安全评审](06_Security_Review.md) | 安全风险分析 |
