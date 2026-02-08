# 项目概览

## 文档信息

- **目的**: 介绍 Rockchip OpenHarmony 仓库的整体定位、核心能力和运行环境
- **适用范围**: 所有芯片平台 (RK2206/RK3399/RK3566/RK3568/RK3588)
- **关键结论**: 本仓库为 SoC 硬件适配层，通过 HDI/VDI 接口对接 OpenHarmony 框架

## 项目定位

### 仓库角色

`device/soc/rockchip` 是 OpenHarmony 系统中 **SoC 硬件适配层** 仓库，承担以下角色：

1. **硬件抽象层 (HAL)** - 将 Rockchip 芯片硬件能力封装为标准化接口
2. **HDI 实现层** - 实现 OpenHarmony HDI (Hardware Driver Interface) 标准接口
3. **内核驱动层** - 提供 Linux 内核驱动和 LiteOS-M HDF 驱动
4. **平台适配层** - 支持多芯片平台的差异化实现

### 系统架构位置

```
┌─────────────────────────────────────────────┐
│         OpenHarmony 应用层                   │
├─────────────────────────────────────────────┤
│         N-API / JS 框架                      │
├─────────────────────────────────────────────┤
│         HDI 标准接口                         │  ← drivers/peripheral
├─────────────────────────────────────────────┤
│         VDI 厂商实现                         │  ← 本仓库 (device/soc/rockchip)
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ Display │ │  MPP    │ │  RGA    │ ...   │
│  └────┬────┘ └────┬────┘ └────┬────┘       │
├───────┼───────────┼───────────┼─────────────┤
│       │           │           │   内核层    │
│  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐       │
│  │  DRM    │ │ MPP驱动 │ │ RGA驱动 │       │
│  └─────────┘ └─────────┘ └─────────┘       │
├─────────────────────────────────────────────┤
│         Rockchip 硬件                        │
└─────────────────────────────────────────────┘
```

### 与上层仓库的关系

| 上层仓库 | 关系 | 说明 |
|----------|------|------|
| `drivers/peripheral` | 接口定义 | 定义 HDI 标准接口 (IDisplayComposerVdi 等) |
| `drivers/hdf_core` | 框架依赖 | RK2206 HDF 驱动依赖 HDF 框架 |
| `foundation/graphic` | 调用方 | 图形子系统调用 Display HDI |
| `foundation/multimedia` | 调用方 | 多媒体子系统调用 Codec HDI |

## 核心能力

### 硬件模块支持

| 模块 | 功能 | 支持平台 |
|------|------|----------|
| **Display** | 显示输出、图层合成、VSync | 全平台 |
| **GPU** | 3D 图形渲染、OpenGL/Vulkan | RK3566/RK3568/RK3588 |
| **ISP** | 图像信号处理、3A 算法 | RK3566/RK3568/RK3588 |
| **MPP** | 视频编解码 (H.264/H.265/VP9) | 全平台 |
| **RGA** | 2D 图形加速、格式转换 | 全平台 |
| **WiFi** | 无线网络 (802.11 a/b/g/n/ac) | RK3568/RK3588 |
| **Codec** | 硬编解码、JPEG 解码 | RK3568/RK3588 |

### 技术特性

1. **多平台支持** - 一套代码支持 5 个芯片平台
2. **HDI 标准化** - 符合 OpenHarmony HDI 接口规范
3. **DRM 显示** - 基于 Linux DRM/KMS 的显示架构
4. **MPP 媒体** - 统一的媒体处理接口
5. **HDF 驱动** - RK2206 支持 HDF 驱动框架

## 运行环境

### 支持的操作系统

| 芯片 | 内核类型 | OS 版本 |
|------|----------|---------|
| RK2206 | LiteOS-M | OpenHarmony 3.0+ |
| RK3399 | Linux 4.19/5.10 | OpenHarmony 3.2+ |
| RK3566 | Linux 4.19/5.10 | OpenHarmony 3.2+ |
| RK3568 | Linux 4.19/5.10/6.6 | OpenHarmony 3.2+ |
| RK3588 | Linux 5.10/6.6 | OpenHarmony 4.0+ |

### 硬件要求

- **RK2206**: 256KB RAM, 8MB Flash, Cortex-M4F@200MHz
- **RK3399**: 2GB+ RAM, 8GB+ Storage, 双核 A72 + 四核 A53
- **RK3566**: 1GB+ RAM, 4GB+ Storage, 四核 A55
- **RK3568**: 1GB+ RAM, 4GB+ Storage, 四核 A55
- **RK3588**: 4GB+ RAM, 16GB+ Storage, 四核 A76 + 四核 A55

## 关键概念

### HDI (Hardware Driver Interface)

OpenHarmony 硬件驱动接口标准，定义了硬件能力的抽象接口：

- **IDisplayComposerVdi** - 显示合成器接口
- **IDisplayBufferVdi** - 显示缓冲区接口
- **ICodecComponent** - 编解码器组件接口

### VDI (Vendor Driver Interface)

厂商驱动接口，是 HDI 的具体实现：

```cpp
// VDI 工厂函数
extern "C" IDisplayComposerVdi* CreateComposerVdi();
extern "C" void DestroyComposerVdi(IDisplayComposerVdi* vdi);
extern "C" IDisplayBufferVdi* CreateDisplayBufferVdi();
extern "C" void DestroyDisplayBufferVdi(IDisplayBufferVdi* vdi);
```

### DRM (Direct Rendering Manager)

Linux 内核显示子系统：

- **KMS** - Kernel Mode Setting，显示模式设置
- **GEM** - Graphics Execution Manager，图形内存管理
- **CRTC** - CRT Controller，显示控制器
- **Connector** - 显示连接器 (HDMI/DP/MIPI)

### MPP (Media Process Platform)

Rockchip 媒体处理平台：

- **MPI** - Media Process Interface，媒体处理接口
- **Encoder** - 视频编码器 (H.264/H.265)
- **Decoder** - 视频解码器 (H.264/H.265/VP9/AV1)

## 芯片平台详情

### RK2206

**定位**: 低功耗 IoT MCU 平台

**硬件规格**:
- CPU: Cortex-M4F @ 200MHz
- RAM: 256KB
- Flash: 8MB
- WiFi: 802.11 b/g/n

**软件特性**:
- 操作系统: LiteOS-M
- 驱动框架: HDF (Hardware Driver Foundation)
- 支持驱动: GPIO、I2C、SPI、FS、WiFi

**代码位置**: `rk2206/`

### RK3399

**定位**: 高性能平板/工业平台

**硬件规格**:
- CPU: 双核 A72 @ 1.8GHz + 四核 A53 @ 1.4GHz
- GPU: Mali-T860 MP4
- 内存: 支持 DDR3/DDR4/LPDDR4

**软件特性**:
- 操作系统: Linux
- 显示: DRM/KMS，支持双屏
- 多媒体: MPP 编解码

**代码位置**: `rk3399/`

### RK3566

**定位**: 中端 AIoT 平台

**硬件规格**:
- CPU: 四核 A55 @ 1.8GHz
- GPU: Mali-G52
- NPU: 1TOPS

**软件特性**:
- 操作系统: Linux
- 显示: 完整 HDI 实现
- 多媒体: GPU + ISP + MPP + RGA

**代码位置**: `rk3566/`

### RK3568

**定位**: 工业/边缘计算主力平台

**硬件规格**:
- CPU: 四核 A55 @ 2.0GHz
- GPU: Mali-G52
- NPU: 1TOPS

**软件特性**:
- 操作系统: Linux (4.19/5.10/6.6)
- 显示: 完整 HDI 实现
- 多媒体: GPU + ISP + MPP + RGA + Codec + WiFi
- 编解码: 支持 OMX IL 标准

**代码位置**: `rk3568/`

### RK3588

**定位**: 旗舰 AI 计算平台

**硬件规格**:
- CPU: 四核 A76 @ 2.4GHz + 四核 A55 @ 1.8GHz
- GPU: Mali-G610 MP4
- NPU: 6TOPS

**软件特性**:
- 操作系统: Linux (5.10/6.6)
- 显示: 8K 输出支持
- 多媒体: 8K 编解码，完整内核驱动
- 内核驱动: 466 个驱动文件

**代码位置**: `rk3588/`

## 相关链接

- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [架构说明](01_Architecture.md) - 系统架构详解
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档
