# device_qemu - OpenHarmony QEMU 设备模拟器

## 快速入门

**device_qemu** 是 OpenHarmony 的官方 QEMU 设备模拟器仓库，提供多种硬件平台的虚拟化支持，使 OpenHarmony 内核能够在无需物理开发板的环境中运行和调试。

### 核心特性

- **多架构支持**: ARM、RISC-V、Xtensa (ESP32)、C-SKY 等
- **VirtIO 虚拟设备**: 块设备、网络、GPU、输入设备等
- **HDF 驱动集成**: 与 OpenHarmony 硬件驱动框架深度集成
- **内核兼容性**: 支持 LiteOS_A 和 LiteOS_M 内核

## 快速开始

### 环境准备

```bash
# 安装 QEMU (以 Ubuntu 为例)
sudo apt install build-essential zlib1g-dev pkg-config libglib2.0-dev \
    binutils-dev libboost-all-dev autoconf libtool libssl-dev \
    libpixman-1-dev virtualenv flex bison

# 下载 QEMU 源码
wget https://download.qemu.org/qemu-6.2.0.tar.xz
tar -xf qemu-6.2.0.tar.xz
cd qemu-6.2.0
./configure --prefix=$PWD/qemu-install
make -j$(nproc)
make install
export PATH=$PWD/qemu-install/bin:$PATH
```

### 构建 OpenHarmony

```bash
# 使用 hb 构建
hb set
hb build -f
```

### 运行示例

```bash
# ARM 虚拟平台 (LiteOS_A)
qemu-system-aarch64 -M virt -cpu cortex-a53 -m 1G \
    -kernel out/arm_virt/OHOS_Image \
    -append "root=devvdb rw console=ttyAMA0" \
    -dtb out/arm_virt/OHOS.dtb \
    -serial stdio

# RISC-V 虚拟平台
qemu-system-riscv32 -M virt -m 256M \
    -kernel out/riscv32_virt/OHOS_Image \
    -append "root=devvdb rw console=ttySIF0" \
    -serial stdio
```

## 架构概览

```
┌─────────────────────────────────────────────────────────┐
│                   OpenHarmony System                     │
├─────────────────────────────────────────────────────────┤
│  User Space                                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │ Applications│ │ Services    │ │ Framework   │       │
│  └─────────────┘ └─────────────┘ └─────────────┘       │
├─────────────────────────────────────────────────────────┤
│  HDF (Hardware Driver Foundation)                         │
│  ┌─────────────────────────────────────────────────────┐│
│  │ device_qemu Drivers                                  ││
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ││
│  │  │ VirtIO  │ │  UART   │ │  CHAR   │ │  MMZ    │ ││
│  │  │ (block) │ │ (serial)│ │ (device)│ │ (memory)│ ││
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ ││
│  └─────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────┤
│  QEMU Hardware Emulation                                  │
│  ┌─────────────────────────────────────────────────────┐│
│  │ CPU │ Memory │ VirtIO Devices │ UART │ Timer │ ... ││
│  └─────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────┤
│  Host OS (Linux/macOS/Windows)                           │
└─────────────────────────────────────────────────────────┘
```

## 文档路线图

| 目标 | 推荐文档 |
|------|----------|
| **了解项目** | [项目概览](01_Project_Overview.md) |
| **理解结构** | [目录结构](02_Directory_Structure.md) |
| **深入架构** | [架构设计](03_Architecture.md) |
| **构建系统** | [GN 构建](04_GN_Build.md) |
| **支持的平台** | [平台列表](07_Platforms.md) |
| **安全考量** | [安全评审](06_Security_Review.md) |
| **遇到问题** | [常见问题](08_Troubleshooting.md) |

## 相关资源

- **官方文档**: [OpenHarmony Docs](https://gitee.com/openharmony/docs)
- **构建指南**: [编译构建子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/编译构建子系统.md)
- **驱动开发**: [HDF 驱动框架](https://gitee.com/openharmony/drivers_hdf_core)
- **内核文档**: [LiteOS_A](https://gitee.com/openharmony/kernel_liteos_a)、[LiteOS_M](https://gitee.com/openharmony/kernel_liteos_m)
