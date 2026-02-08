# 项目概览

## 项目定位

`kernel_linux_patches` 是 OpenHarmony 项目的内核补丁仓库，专注于为 OpenHarmony 提供经过适配和加固的 Linux 内核补丁集合。

### 核心定位

- **内核基线**: 基于上游 Linux 内核 LTS（Long Term Support）版本
- **驱动框架**: 集成 OpenHarmony HDF（Hardware Driver Foundation）驱动框架
- **芯片适配**: 提供多芯片平台和多开发板的驱动补丁
- **安全增强**: 包含 CVE 安全补丁和 OpenHarmony 特性补丁

## 技术背景

### 内核版本策略

OpenHarmony 采用多版本内核策略，当前支持以下 LTS 版本：

| 内核版本 | 状态 | 特性 |
|----------|------|------|
| Linux 4.19.y | 稳定 | 成熟稳定，兼容性好 |
| Linux 5.10.y | 稳定 | 增强性能，更多特性 |
| Linux 6.6.y | 预览 | 最新 LTS，支持新硬件 |

### 与上游关系

- **上游基础**: 基于 kernel.org 官方 Linux LTS 版本
- **CVE 补丁**: 持续集成上游安全补丁
- **OpenHarmony 特性**: 在此基础上添加 HDF 等框架

## 核心能力

### 1. 硬件驱动抽象（HDF）

HDF（Hardware Driver Foundation）是 OpenHarmony 的统一驱动框架：

```
驱动架构:
├── HDF Core          # 驱动核心框架
├── Driver Interface  # 标准化驱动接口
└── Platform Adapter  # 平台适配层
```

### 2. 多平台支持

支持的芯片平台覆盖：

| 厂商 | 芯片型号 | 架构 | 状态 |
|------|----------|------|------|
| HiSilicon | Hi3516D V300 | ARM64 | 稳定 |
| Rockchip | RK3568 | ARM64 | 稳定 |
| NXP | i.MX8M Mini | ARM64 | 稳定 |
| Loongson | 3A5000 | LoongArch | 稳定 |
| QEMU | 模拟器 | ARM64/x86_64 | 开发 |

### 3. 补丁管理

- **分类管理**: 按内核版本和芯片平台分类
- **版本隔离**: 不同内核版本独立管理
- **热补丁支持**: 支持增量补丁更新

## 运行环境

### 构建环境要求

| 组件 | 要求 |
|------|------|
| 操作系统 | Ubuntu 20.04+ / 其他 Linux 发行版 |
| 构建工具 | GN, Make, GCC/Clang |
| 内核源码 | 对应版本的 Linux 内核源码 |
| Python | Python 3.8+ |

### 部署环境

- **目标设备**: 基于支持芯片的开发板
- **系统类型**: 小型系统 / 标准系统 / 大型系统

## 关键概念

### 补丁类型

| 类型 | 说明 | 位置 |
|------|------|------|
| common_patch | 通用 HDF 补丁 | `{version}/common_patch/` |
| soc_patch | SOC 特定驱动补丁 | `{version}/{board}_patch/` |
| config_patch | 内核配置补丁 | config 目录 |

### 板卡命名规则

- `hispark_taurus`: Hi3516D V300 开发板
- `rk3568`: Rockchip RK3568 开发板
- `imx8mm`: NXP i.MX8M Mini 开发板
- `qemu-*`: QEMU 模拟器

## 相关文档

- [支持的板卡](./03_Supported_Boards.md)
- [补丁分析](./04_Patches_Analysis.md)
- [构建系统](./05_Build_System.md)
