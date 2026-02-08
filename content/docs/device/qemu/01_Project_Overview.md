# 项目概览

## 项目定位

### 基本信息

| 项目 | 说明 |
|------|------|
| **项目名称** | device_qemu |
| **全称** | OpenHarmony QEMU Device Simulator |
| **仓库地址** | https://gitee.com/openharmony/device_qemu |
| **类型** | 设备模拟器仓库 |
| **许可证** | Apache-2.0 |

### 核心使命

device_qemu 是 OpenHarmony 官方提供的 QEMU 虚拟设备支持仓库，其核心使命是：

1. **解除硬件依赖**: 使 OpenHarmony 内核能够在无需物理开发板的环境中运行
2. **加速开发迭代**: 提供标准化、可重复的虚拟硬件环境
3. **支撑测试验证**: 为内核和驱动提供可控制的测试环境

### 适用范围

**适用场景**:
- OpenHarmony 内核 (LiteOS_A / LiteOS_M) 开发与调试
- 驱动开发与验证
- 系统集成测试
- CI/CD 自动构建测试

**约束条件**:
- 仅支持 OpenHarmony 内核，不支持其他操作系统
- QEMU 版本要求: 6.2.0 及以上
- 模拟硬件性能受限于 QEMU 模拟能力

## 核心能力

### 支持的硬件抽象

| 能力 | 说明 | 证据位置 |
|------|------|----------|
| **VirtIO 块设备** | 虚拟磁盘存储 | `drivers/virtio/virtblock.c` |
| **VirtIO 网络** | 虚拟网络接口 | `drivers/virtio/virtnet.c` |
| **VirtIO GPU** | 虚拟图形输出 | `drivers/virtio/virtgpu.c` |
| **VirtIO 输入设备** | 虚拟键盘/鼠标 | `drivers/virtio/virtinput.c` |
| **VirtIO RNG** | 虚拟随机数生成器 | `drivers/virtio/virtrng.c` |
| **UART 串口** | PL011 兼容串口 | `drivers/uart/uart_pl011.c` |
| **字符设备** | 平台字符设备 | `drivers/char/` |
| **内存管理** | MMZ 内存区域 | `drivers/char/mmz/mmz.c` |

### 支持的 CPU 架构

| 架构 | 平台目录 | 内核支持 |
|------|----------|----------|
| **ARM** | `arm_virt/` | LiteOS_A, Linux |
| **ARM Cortex-M4** | `arm_mps2_an386/` | LiteOS_M |
| **ARM Cortex-M55** | `arm_mps3_an547/` | LiteOS_M |
| **RISC-V 32位** | `riscv32_virt/` | LiteOS_M |
| **RISC-V 64位** | `riscv64_virt/` | LiteOS_M, Linux |
| **x86_64** | `x86_64_virt/` | Linux |
| **Xtensa (ESP32)** | `esp32/` | LiteOS_M |
| **C-SKY (SmartL)** | `SmartL_E802/` | LiteOS_M |

### 虚拟硬件特性

```
┌─────────────────────────────────────────────────────┐
│              device_qemu 虚拟硬件层                   │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│
│  │   CPU       │  │  Memory     │  │  Timer      ││
│  │  (ARM/RISC-V│  │  (可配置)    │  │  (ARM/ RISC)││
│  │   Xtensa)   │  │             │  │             ││
│  └─────────────┘  └─────────────┘  └─────────────┘│
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│
│  │  VirtIO     │  │    UART     │  │   GPIO      ││
│  │  Devices    │  │  (PL011)    │  │  (模拟)     ││
│  └─────────────┘  └─────────────┘  └─────────────┘│
└─────────────────────────────────────────────────────┘
```

## 运行环境

### 编译环境要求

| 组件 | 要求 | 说明 |
|------|------|------|
| **Python** | 3.7+ | hb 构建工具依赖 |
| **GCC** | 7.0+ | 交叉编译工具链 |
| **GN** | r28+ | 构建系统生成工具 |
| **Ninja** | 1.9+ | 构建执行引擎 |
| **QEMU** | 6.2.0+ | 目标平台模拟器 |

### 依赖仓库

device_qemu 依赖以下 OpenHarmony 子系统：

| 仓库 | 用途 | 依赖方式 |
|------|------|----------|
| `kernel_liteos_a` | LiteOS_A 内核 | 运行时依赖 |
| `kernel_liteos_m` | LiteOS_M 内核 | 运行时依赖 |
| `drivers_hdf_core` | HDF 驱动框架 | 编译时依赖 |
| `build` | 构建子系统 | 编译时依赖 |

### 运行时环境

```
┌─────────────────────────────────────────────────────────┐
│                   运行时环境栈                            │
├─────────────────────────────────────────────────────────┤
│  Host OS: Ubuntu 18.04+ / macOS / Windows (WSL2)       │
├─────────────────────────────────────────────────────────┤
│  QEMU: 6.2.0+ (仿真目标 CPU 和外设)                      │
├─────────────────────────────────────────────────────────┤
│  OpenHarmony: LiteOS_A / LiteOS_M 内核                  │
├─────────────────────────────────────────────────────────┤
│  device_qemu: 虚拟设备驱动 (HDF 驱动)                     │
└─────────────────────────────────────────────────────────┘
```

## 关键概念

### HDF (Hardware Driver Foundation)

HDF 是 OpenHarmony 的硬件驱动统一框架，device_qemu 中的驱动均以 HDF 驱动形式实现。

**证据**: `drivers/virtio/BUILD.gn:29`
```gn
import("//drivers/hdf_core/adapter/khdf/liteos/hdf.gni")

hdf_driver(module_name) {
  sources = [ ... ]
}
```

### VirtIO 标准

VirtIO 是虚拟化 I/O 设备的标准化接口，device_qemu 提供了完整的 VirtIO 设备驱动支持。

**证据**: `drivers/virtio/BUILD.gn:32-42`
```gn
sources = [
  "fakesdio.c",
  "virtblock.c",
  "virtgpu.c",
  "virtinput.c",
  "virtmmio.c",
  "virtnet.c",
]
```

### MMZ (Memory Management Zone)

MMZ 是 OpenHarmony 专有的内存管理机制，用于驱动和硬件之间的内存共享。

**证据**: `drivers/char/mmz/mmz.h`
```c
#define MMZ_MODULE_NAME "mmz"
#define MMZ_IRQ_NAME    "mmz"
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](02_Directory_Structure.md) | 详细目录结构说明 |
| [架构设计](03_Architecture.md) | 驱动架构与数据流 |
| [GN 构建](04_GN_Build.md) | 构建系统详解 |
| [支持的平台](07_Platforms.md) | 各平台详细配置 |
