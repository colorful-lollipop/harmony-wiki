# 支持的板卡

本文档列出 `kernel_linux_patches` 仓库支持的所有芯片平台和开发板，包括适配状态和关键特性。

## 支持平台总览

| 厂商 | 芯片型号 | 架构 | 内核版本 | 状态 | 用途 |
|------|----------|------|----------|------|------|
| HiSilicon | Hi3516D V300 | ARM64 | 4.19/5.10 | 稳定 | 智慧视觉 |
| Rockchip | RK3568 | ARM64 | 5.10/6.6 | 稳定 | 平板/开发板 |
| NXP | i.MX8M Mini | ARM64 | 5.10 | 稳定 | 工业/消费 |
| Loongson | 3A5000 | LoongArch | 5.10 | 稳定 | 桌面/服务器 |
| 开天 | 3566B | ARM64 | 5.10 | 稳定 | 开发套件 |
| UnionPi | Tiger | ARM64 | 5.10 | 稳定 | 盒子/开发 |
| 扬帆 | - | ARM64 | 5.10 | 稳定 | 开发板 |
| 致远 | - | ARM64 | 5.10 | 稳定 | 开发板 |
| QEMU | ARM 模拟器 | ARM64 | 5.10 | 开发 | 虚拟化测试 |
| QEMU | x86_64 模拟器 | x86_64 | 5.10 | 开发 | 虚拟化测试 |
| HiHope | Phoenix | ARM64 | 5.10 | 稳定 | 开发板 |

## 详细说明

### HiSilicon Hi3516D V300

**位置**: `linux-4.19/hispark_taurus_patch/`, `linux-5.10/hispark_taurus_patch/`

**特性**:
- 专为智慧视觉场景设计
- 支持小型系统和标准系统
- 集成 HDF 驱动框架

**配置**:
```
hispark_taurus_small_defconfig   # 小型系统
hispark_taurus_standard_defconfig # 标准系统
```

### Rockchip RK3568

**位置**: `linux-5.10/rk3568_patch/`, `linux-6.6/rk3568_patch/`

**特性**:
- 四核 ARM Cortex-A55
- 集成 NPU（神经网络处理单元）
- 丰富的接口支持

**配置**:
- 支持小型系统 / 标准系统 / 大型系统
- 集成 GPU 和 VPU 驱动

### NXP i.MX8M Mini

**位置**: `linux-5.10/imx8mm_patch/`

**特性**:
- 四核 ARM Cortex-A53 + Cortex-M4
- 低功耗设计
- 多媒体编解码支持

**补丁组织**:
```
imx8mm_patch/
├── patches/           # 按模块组织的补丁
│   ├── 0001_linux_arch.patch
│   ├── 0002_linux_block.patch
│   └── ...
└── hdf.patch
```

### Loongson 3A5000

**位置**: `linux-5.10/ls3a5000_patch/`

**特性**:
- 基于 LoongArch 架构
- 国产 CPU 代表
- 支持服务器和桌面场景

### QEMU 模拟器

**位置**: `linux-5.10/qemu-arm-linux_patch/`, `qemu-x86_64-linux_patch/`

**用途**:
- 虚拟化开发测试
- 无需真实硬件即可验证
- 快速原型验证

## 适配状态说明

### 状态定义

| 状态 | 含义 |
|------|------|
| 稳定 | 经过充分测试，可用于生产环境 |
| 开发 | 正在开发中，可能存在不稳定因素 |
| 预览 | 新增支持，功能待完善 |

### 各内核版本支持情况

#### Linux 4.19

| 板卡 | 状态 | 备注 |
|------|------|------|
| hispark_taurus | 稳定 | 主要支持 |
| - | - | - |

#### Linux 5.10

| 板卡 | 状态 | 备注 |
|------|------|------|
| hispark_taurus | 稳定 | 主要支持 |
| hispark_phoenix | 稳定 | HiHope 开发板 |
| imx8mm | 稳定 | NXP 平台 |
| khdvk_3566b | 稳定 | 开天 3566B |
| ls3a5000 | 稳定 | 龙芯平台 |
| qemu-arm | 开发 | ARM 模拟器 |
| qemu-x86_64 | 开发 | x86_64 模拟器 |
| rk3568 | 稳定 | Rockchip 平台 |
| unionpi_tiger | 稳定 | UnionPi Tiger |
| yangfan | 稳定 | 扬帆开发板 |
| zhiyuan | 稳定 | 致远开发板 |

#### Linux 6.6

| 板卡 | 状态 | 备注 |
|------|------|------|
| rk3568 | 稳定 | 预览支持 |
| - | - | - |

## 添加新板卡

### 添加步骤

1. **创建补丁目录**: 在对应内核版本下创建 `{board}_patch/`
2. **添加 HDF 补丁**: 创建 `hdf.patch` 文件
3. **添加内核补丁**: 创建 `kernel.patch` 文件
4. **添加配置文件**: 在 `config/{version}/arch/arm/configs/` 添加 defconfig
5. **更新文档**: 在本文档添加新板卡说明

### 目录结构示例

```
linux-{version}/
└── {new_board}_patch/
    ├── kernel.patch      # 内核补丁
    └── hdf.patch         # HDF 补丁
```

## 相关文档

- [补丁分析](./04_Patches_Analysis.md)
- [构建系统](./05_Build_System.md)
- [配置详情](./appendix/Config_Details.md)
