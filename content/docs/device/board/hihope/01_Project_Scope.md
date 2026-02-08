# 项目边界与定位

## 仓库职责

**device_board_hihope** 是 OpenHarmony 设备板级配置的核心仓库，负责定义：
- 开发板硬件抽象配置
- 外设驱动适配
- 构建系统参数
- 系统初始化参数

> 证据: `README_zh.md:7` — "该仓托管HiHope产品系列openharmony智能硬件"

### 职责范围

| 分类 | 包含 | 不包含 |
|------|------|--------|
| **硬件配置** | 板级参数、HDF 驱动、外设配置 | SoC 内部驱动 (在 device_soc_*) |
| **构建配置** | BUILD.gn, config.gni, device.gni | 内核源码 (在 kernel/*) |
| **系统配置** | init.cfg, 分布式硬件 JSON | 系统服务 (在 base/*) |
| **产品配置** | vendor/hihope 产品定义 | 应用层代码 |

---

## 模块职责划分

### Neptune100 (LiteOS-M IoT)

```
职责: Wi-Fi/蓝牙双模 SoC 板级配置
├─ BUILD.gn         # 模块组定义
├─ liteos_m/        # LiteOS-M 内核模块
├─ shields/         # 扩展板配置
└─ neptune100.hcs  # HDF 设备描述
```

**证据**: `/neptune100/BUILD.gn:14-20`

### RK3568 / DAYU200 (Linux 多媒体)

```
职责: 完整多媒体 AI 开发板
├─ audio_drivers/   # HDF 音频驱动 (codec/dai/dsp/dma)
├─ audio_alsa/      # ALSA 标准接口适配
├─ camera/vdi_impl/v4l2/  # V4L2 相机框架
├─ wifi/bcmdhd_wifi6/     # WiFi6 HDF 适配
├─ cfg/             # 系统初始化配置
├─ kernel/          # Linux 构建脚本
└─ updater/         # OTA 升级配置
```

**证据**: `/rk3568/BUILD.gn:18-42`

### DAYU210 (RK3588 高性能)

```
职责: 高性能 AI 开发板
├─ audio_drivers/   # ES8323 编解码器驱动
├─ camera/vdi_impl/v4l2/  # ISP V6 相机框架
├─ cfg/             # 系统初始化配置
├─ kernel/          # Linux 5.10 + 补丁
├─ startup/         # 启动控制
├─ loader/          # U-Boot 引导
└─ updater/         # OTA 升级配置
```

**证据**: `/dayu210/BUILD.gn:18-42`

### nearlink_dk_3863 (星闪)

```
职责: 星闪 (NearLink) 开发板配置
└─ liteos_m/        # LiteOS-M 内核配置
```

**证据**: `/nearlink_dk_3863/liteos_m/config.gni`

---

## 依赖方向

```
                    ┌─────────────────┐
                    │  vendor/hihope  │  ← 产品定义
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌─────────────────┐
│device_board_* │   │device_soc_*  │   │   kernel/*      │
│(本仓库 - 板级) │   │  (SoC 驱动)   │   │   (内核源码)    │
└───────────────┘   └───────────────┘   └─────────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  build/ohos.gni │  ← 构建系统
                    └─────────────────┘
```

---

## 外部依赖

### SoC 厂商仓库

| 仓库 | 依赖内容 |
|------|----------|
| `device/soc/rockchip/rk3568` | RK3568 硬件驱动 (display/codec/rga/mpp) |
| `device/soc/rockchip/rk3588` | RK3588 硬件驱动 |
| `device/soc/winnermicro` | W800 驱动 (Neptune100) |
| `device/soc/hisilicon/ws63v100` | WS63 驱动 (NearLink) |

### OpenHarmony 核心仓库

| 仓库 | 依赖内容 |
|------|----------|
| `build/ohos.gni` | 标准构建系统 |
| `kernel/liteos_m/liteos.gni` | LiteOS-M 构建模板 |
| `drivers/hdf_core/...` | HDF 驱动框架 |

### 产品仓库

| 仓库 | 依赖内容 |
|------|----------|
| `vendor/hihope` | 产品配置 (开机动画, 系统参数) |

---

## 稳定性说明

### 稳定接口

| 接口类型 | 稳定性 | 说明 |
|----------|--------|------|
| HDF 驱动框架 | 稳定 | OpenHarmony 标准驱动模型 |
| GN 构建配置 | 稳定 | 标准构建系统 |
| init.cfg | 稳定 | 系统初始化标准格式 |

### 潜在变更点

| 区域 | 风险 | 说明 |
|------|------|------|
| 内核构建脚本 | 低 | kernel/build_kernel.sh 可能随内核版本更新 |
| 相机 V4L2 | 中 | ISP 版本差异可能导致适配变更 |
| 音频驱动 | 低 | HDF 框架稳定, 内部实现可替换 |

---

## 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Directory_Structure](02_Directory_Structure.md) - 目录结构
- [04_GN_Build](04_GN_Build.md) - 构建配置
- [05_Board_Configurations](05_Board_Configurations.md) - 开发板详情
