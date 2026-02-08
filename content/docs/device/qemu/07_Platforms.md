# 支持的平台

## 平台总览

device_qemu 支持多种 CPU 架构和开发板的 QEMU 虚拟化模拟。

### 平台列表

| 架构 | 平台目录 | 内核支持 | 说明 |
|------|----------|----------|------|
| **ARM** | `arm_virt/` | LiteOS_A, Linux | ARM 虚拟化平台 |
| **ARM Cortex-M4** | `arm_mps2_an386/` | LiteOS_M | MPS2-AN386 模拟 |
| **ARM Cortex-M55** | `arm_mps3_an547/` | LiteOS_M | MPS3-AN547 模拟 |
| **RISC-V 32位** | `riscv32_virt/` | LiteOS_M | RISC-V 32位虚拟平台 |
| **RISC-V 64位** | `riscv64_virt/` | LiteOS_M, Linux | RISC-V 64位虚拟平台 |
| **x86_64** | `x86_64_virt/` | Linux | x86_64 虚拟平台 |
| **Xtensa (ESP32)** | `esp32/` | LiteOS_M | Xtensa ESP32 模拟 |
| **C-SKY (SmartL)** | `SmartL_E802/` | LiteOS_M | C-SKY SmartL_E802 模拟 |

## ARM 虚拟化平台

### arm_virt

**路径**: `arm_virt/`

**配置**: `arm_virt/ohos.build`

**内核支持**: LiteOS_A, Linux

**虚拟硬件**:
- CPU: ARM Cortex-A 系列 (可配置)
- 内存: 可配置 (通常 1GB)
- VirtIO 设备: 块设备、网络、GPU、输入设备
- UART: PL011 串口

**构建配置**:
```json
{
  "parts": {
    "device_arm_virt": {
      "module_list": [
        "//device/qemu/arm_virt:arm_virt"
      ]
    }
  },
  "subsystem": "device_arm_virt"
}
```

### ARM Cortex-M4 (MPS2-AN386)

**路径**: `arm_mps2_an386/`

**配置**: `arm_mps2_an386/ohos.build`

**内核支持**: LiteOS_M

**目标硬件**: ARM MPS2-AN386 开发板 (Cortex-M4)

**特点**:
- 轻量级 MCU 平台
- 适合嵌入式应用开发
- 支持基本外设模拟

### ARM Cortex-M55 (MPS3-AN547)

**路径**: `arm_mps3_an547/`

**配置**: `arm_mps3_an547/ohos.build`

**内核支持**: LiteOS_M

**目标硬件**: ARM MPS3-AN547 开发板 (Cortex-M55)

**特点**:
- 最新 Cortex-M 架构
- 支持 DSP 指令
- 适合 AI/ML 嵌入式应用

## RISC-V 平台

### RISC-V 32位

**路径**: `riscv32_virt/`

**配置**: `riscv32_virt/ohos.build`

**内核支持**: LiteOS_M

**虚拟硬件**:
- CPU: RISC-V 32位 (rv32imac)
- 内存: 可配置 (通常 256MB)
- VirtIO 设备: 块设备、网络
- UART: SiFive UART

**构建配置**:
```json
{
  "parts": {
    "device_riscv32_virt": {
      "module_list": [
        "//device/qemu/riscv32_virt:riscv32_virt"
      ]
    }
  },
  "subsystem": "device_riscv32_virt"
}
```

### RISC-V 64位

**路径**: `riscv64_virt/`

**配置**: `riscv64_virt/ohos.build`

**内核支持**: LiteOS_M, Linux

**虚拟硬件**:
- CPU: RISC-V 64位 (rv64gc)
- 内存: 可配置 (通常 1GB)
- VirtIO 设备: 完整支持
- UART: SiFive UART

## 其他架构

### Xtensa (ESP32)

**路径**: `esp32/`

**配置**: `esp32/ohos.build`

**内核支持**: LiteOS_M

**目标硬件**: Espressif ESP32 芯片

**特点**:
- Xtensa 架构
- WiFi/BT 支持 (模拟)
- 适合 IoT 应用

### C-SKY SmartL_E802

**路径**: `SmartL_E802/`

**配置**: `SmartL_E802/ohos.build`

**内核支持**: LiteOS_M

**目标硬件**: C-SKY SmartL_E802 芯片

**特点**:
- C-SKY 架构
- 嵌入式处理器
- 适合特定嵌入式应用

### x86_64 虚拟平台

**路径**: `x86_64_virt/`

**配置**: `x86_64_virt/ohos.build`

**内核支持**: Linux

**特点**:
- 桌面/服务器级模拟
- 支持完整 VirtIO 设备
- 适合系统级测试

## 平台选择指南

### 按使用场景选择

| 场景 | 推荐平台 | 理由 |
|------|----------|------|
| **应用开发** | `arm_virt` | 完整系统支持 |
| **驱动开发** | `arm_virt` / `riscv64_virt` | 丰富外设 |
| **嵌入式 MCU** | `arm_mps2_an386` | Cortex-M4 轻量级 |
| **AI/ML 嵌入式** | `arm_mps3_an547` | Cortex-M55 DSP |
| **RISC-V 开发** | `riscv32_virt` / `riscv64_virt` | RISC-V 原生支持 |
| **IoT 应用** | `esp32` | WiFi/BT 模拟 |
| **x86 兼容性** | `x86_64_virt` | 桌面级模拟 |

### 资源需求

| 平台 | CPU | 内存 | 存储 |
|------|-----|------|------|
| `arm_virt` | 2+ 核 | 1GB | 2GB |
| `arm_mps2_an386` | 1 核 | 256MB | 512MB |
| `arm_mps3_an547` | 1 核 | 512MB | 1GB |
| `riscv32_virt` | 1 核 | 256MB | 512MB |
| `riscv64_virt` | 2 核 | 1GB | 2GB |
| `x86_64_virt` | 2+ 核 | 2GB | 4GB |

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](01_Project_Overview.md) | 项目定位与核心能力 |
| [架构设计](03_Architecture.md) | 虚拟硬件架构 |
| [GN 构建](04_GN_Build.md) | 平台构建配置 |
| [常见问题](08_Troubleshooting.md) | 平台相关问题 |
