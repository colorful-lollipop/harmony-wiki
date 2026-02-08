# 项目概览

## 项目定位

**device_board_hihope** 是 OpenHarmony HiHope 设备板级配置仓库，托管 HiHope 产品系列的 OpenHarmony 智能硬件配置。

> 证据: `README_zh.md:7` — "该仓托管HiHope产品系列openharmony智能硬件"

### HiHope 平台

润和 HiHope 是全栈解决方案平台，致力于打造以国产芯片为核心的生态系统：
- **产品系列**: 海王星（Neptune）、满天星（HiStar）、大禹（DAYU）
- **覆盖范围**: 芯片 → 终端 → 应用
- **服务对象**: 半导体厂商、模组、板卡、下游客户与场景

---

## 支持的开发板

| 开发板 | 芯片 | 架构 | 内核 | 代码规模 | 特点 |
|--------|------|------|------|----------|------|
| **Neptune100** | 联盛德 W800 | ck803 | liteos_m | 轻量配置 | Wi-Fi/蓝牙双模 IoT |
| **DAYU200 (rk3568)** | 瑞芯微 RK3568 | armv8-a | Linux | ~11,562行 | AI 开发板, 完整多媒体 |
| **DAYU210** | 瑞芯微 RK3588 | armv8-a | Linux | ~4,877行 | 高性能 AI 开发板 |
| **nearlink_dk_3863** | 海思 WS63 | rv32imfc | liteos_m | 轻量配置 | 星闪 (NearLink) 开发板 |

---

## 核心能力

### Neptune100 (IoT 场景)
- **通信**: Wi-Fi (802.11 b/g/n) + 蓝牙 BT/BLE4.2 双模
- **安全**: 硬件加解密 (AES/DES/RSA/SHA/CRC)
- **接口**: GPIO/I2C/UART/SPI/I2S/PWM/ADC

### RK3568 (DAYU200)
- **CPU**: 4×Cortex-A55 @2.0GHz, 22nm 工艺
- **GPU**: 双核心架构
- **NPU**: 高效能神经网络处理器
- **多媒体**: 8K 编解码, 多屏异显 (3屏)
- **接口**: 双千兆以太网, Wi-Fi6, 4K HDMI

### DAYU210 (RK3588)
- **CPU**: 4×Cortex-A76 + 4×Cortex-A55 @2.4GHz
- **GPU**: G610, 支持 OpenGL/Vulkan
- **NPU**: 6.0 TOPs
- **多媒体**: 8K 视频编解码
- **接口**: 2×千兆以太网, MIPI-CSI×4, HDMI/DP/eDP

---

## 运行环境

### 编译环境
```bash
# Ubuntu 18.04 依赖
sudo apt-get install binutils git git-lfs gnupg flex bison \
  gperf build-essential zip curl zlib1g-dev gcc-multilib \
  g++-multilib libc6-dev-i386 lib32ncurses5-dev \
  x11proto-core-dev libx11-dev lib32z1-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip m4 bc \
  gnutls-bin python3.8 python3-pip ruby
```

### 编译命令
```bash
# DAYU210
./build.sh --product-name dayu210

# RK3568 (DAYU200)
hb set -> hihope -> rk3568
hb build -f

# Neptune100 (需安装 csky 工具链)
参考 device_soc_winnermicro 仓库
```

### 产物路径
```
out/rk3588/packages/phone/images/   # DAYU210
out/rk3568/packages/phone/images/  # RK3568
```

---

## 关键概念

### HDF (Hardware Driver Foundation)
OpenHarmony 硬件驱动框架，本仓库所有驱动均基于 HDF：
```c
struct HdfDriverEntry {
    .moduleVersion = 1,
    .moduleName = "XXX",
    .Bind = XxxDriverBind,
    .Init = XxxDriverInit,
    .Release = XxxDriverRelease,
};
HDF_INIT(g_XxxDriverEntry);
```

### V4L2 (Video4Linux2)
相机驱动使用的视频接口框架：
- **设备管理器**: ISP 传感器适配
- **Pipeline Core**: 图像处理节点 (编解码/EXIF/人脸检测)
- **Demo**: 示例程序

### ALSA (Advanced Linux Sound Architecture)
音频驱动标准接口：
- **vendor_capture.c**: 音频采集
- **vendor_render.c**: 音频播放

---

## 相关仓库

| 仓库 | 描述 |
|------|------|
| [vendor/hihope](https://gitee.com/openharmony/vendor_hihope) | 产品配置 |
| [device_soc_rockchip](https://gitee.com/openharmony/device_soc_rockchip) | Rockchip SoC 驱动 |
| [device_soc_winnermicro](https://gitee.com/openharmony/device_soc_winnermicro) | 联盛德 W800 驱动 |
| [device_soc_hisilicon/ws63v100](https://gitee.com/openharmony) | 海思星闪驱动 |

---

## 文档导航

- **目录结构**: [02_Directory_Structure](02_Directory_Structure.md)
- **架构说明**: [03_Architecture](03_Architecture.md)
- **构建配置**: [04_GN_Build](04_GN_Build.md)
- **开发板配置**: [05_Board_Configurations](05_Board_Configurations.md)
