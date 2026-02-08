# 目录结构

## 文档信息

- **目的**: 详细说明 Rockchip OpenHarmony 仓库的目录结构和模块职责
- **适用范围**: 所有芯片平台 (RK2206/RK3399/RK3566/RK3568/RK3588)
- **关键结论**: 采用分层目录结构，common/ 共享通用代码，各芯片目录包含平台特定实现

## 顶层目录结构

```
device/soc/rockchip/
├── BUILD.gn                    # 根构建配置（仅 LiteOS-M）
├── README.md                   # 项目简介（中文）
├── README_zh.md                # 项目简介（详细中文）
├── README.en.md                # 项目简介（英文）
├── LICENSE                     # 许可证说明
├── LICENSE-Apache              # Apache 2.0 许可证
├── LICENSE-BSD-3-Clause        # BSD-3-Clause 许可证
├── LICENSE-GPL-2.0             # GPL-2.0 许可证
├── LICENSE-MIT                 # MIT 许可证
├── OAT.xml                     # 开源合规检查配置
├── CODEOWNERS                  # 代码审查者配置
├── Kconfig.liteos_m.*          # LiteOS-M 内核配置
├── common/                     # 公共组件（跨平台共享）
├── rk2206/                     # RK2206 MCU 平台
├── rk3399/                     # RK3399 高性能平台
├── rk3566/                     # RK3566 AIoT 平台
├── rk3568/                     # RK3568 主力平台
└── rk3588/                     # RK3588 旗舰平台
```

## common/ 公共组件

**职责**: 跨平台共享的硬件抽象层实现

**文件统计**: ~104 个源文件

### common/hardware/ - 公共硬件抽象

```
common/hardware/
├── display/                    # 显示 HAL 公共实现
│   └── src/
│       ├── display_device/     # 显示设备管理
│       │   ├── hdi_session.cpp # HDI 会话管理
│       │   ├── hdi_session.h
│       │   ├── hdi_display.cpp # 显示设备实现
│       │   ├── hdi_display.h
│       │   ├── hdi_composer.cpp # 合成器实现
│       │   ├── hdi_composer.h
│       │   ├── hdi_layer.cpp   # 图层管理
│       │   ├── hdi_layer.h
│       │   ├── drm_display.cpp # DRM 显示实现
│       │   ├── drm_display.h
│       │   ├── drm_crtc.cpp    # CRTC 管理
│       │   ├── drm_connector.cpp # 连接器管理
│       │   ├── drm_plane.cpp   # Plane 管理
│       │   └── ...
│       └── display_gralloc/    # 图形内存分配
│           ├── display_gralloc_gbm.c
│           ├── hi_gbm.c        # GBM 封装
│           └── hi_gbm.h
├── gpu/                        # GPU 公共头文件
│   └── include/
│       └── gbm.h               # GBM 缓冲区管理
├── mpp/                        # MPP 公共头文件和实现
│   ├── include/                # MPP 头文件
│   │   ├── rk_mpi.h            # MPP 主接口
│   │   ├── mpp_frame.h         # 帧数据定义
│   │   ├── mpp_packet.h        # 数据包定义
│   │   ├── mpp_buffer.h        # 缓冲区管理
│   │   └── ...
│   └── mpp/
│       └── hdi_mpp/            # MPP HDI 实现
│           ├── hdi_mpp_mpi.cpp
│           └── hdi_mpp_mpi.h
└── rga/                        # RGA 公共头文件
    └── include/
        ├── rga.h
        ├── RgaUtils.h
        └── GrallocOps.h
```

### common/kernel/ - 公共内核驱动

```
common/kernel/
└── drivers/
    ├── gpu/                    # GPU 驱动
    │   └── arm/
    │       ├── midgard/        # Mali Midgard (RK3399)
    │       └── bifrost/        # Mali Bifrost (RK3566/RK3568/RK3588)
    └── net/wireless/rockchip_wlan/  # WiFi 驱动
        └── rkwifi/
            └── bcmdhd_wifi6/   # Broadcom WiFi6 驱动
```

### common/sdk_linux/ - Linux SDK 公共代码

```
common/sdk_linux/
├── drivers/gpu/drm/            # DRM 驱动代码
│   ├── drm_auth.c              # DRM 认证
│   ├── drm_ioctl.c             # DRM ioctl
│   └── ...
├── ipc/                        # IPC 工具代码
│   ├── util.c                  # IPC 权限检查
│   ├── msg.c                   # 消息队列
│   ├── shm.c                   # 共享内存
│   └── sem.c                   # 信号量
└── ...
```

## rk2206/ - RK2206 MCU 平台

**定位**: 低功耗 IoT 平台，Cortex-M4F @ 200MHz

**文件统计**: ~60 个源文件

**内核类型**: LiteOS-M

```
rk2206/
├── BUILD.gn                    # RK2206 构建配置
├── board.gni                   # 板级配置
├── README_zh.md                # 平台说明
├── adapter/                    # OpenHarmony 适配层
│   └── hals/
│       ├── communication/wifi_lite/  # WiFi 适配
│       ├── iot_hardware/wifiiot_lite/ # IoT 硬件适配
│       ├── update/             # OTA 更新适配
│       └── utils/file/         # 文件系统适配
├── hdf_driver/                 # HDF 驱动实现
│   ├── gpio/gpio_driver.c      # GPIO 驱动
│   ├── i2c/i2c_driver.c        # I2C 驱动
│   ├── spi/spi_driver.c        # SPI 驱动
│   └── fs/fs_driver.c          # 文件系统驱动
├── hdf_config/                 # HDF 配置
│   ├── device_info/device_info.hcs  # 设备信息
│   ├── gpio/gpio_config.hcs    # GPIO 配置
│   ├── i2c/i2c_config.hcs      # I2C 配置
│   └── spi/spi_config.hcs      # SPI 配置
├── hardware/                   # 芯片硬件抽象
│   ├── include/                # 头文件
│   └── lib/                    # 库文件 (CMSIS, BSP)
├── sdk_liteos/                 # LiteOS SDK
│   ├── platform/               # 平台代码
│   │   ├── main/main.c         # 主入口
│   │   ├── network/            # 网络配置
│   │   ├── system/             # 系统代码
│   │   ├── uart/               # UART 驱动
│   │   └── startup/            # 启动代码
│   └── liteos_m/               # LiteOS-M 配置
└── tools/package/              # 打包工具
```

## rk3399/ - RK3399 高性能平台

**定位**: 高端平板/工业平台，双核 A72 + 四核 A53

**文件统计**: ~103 个源文件

**内核类型**: Linux

```
rk3399/
├── soc.gni                     # SoC 构建配置
├── common/hal/                 # HAL 层
│   └── usb/                    # USB HAL
└── hardware/                   # 硬件模块
    ├── display/                # 显示模块
    │   └── src/
    │       ├── display_device/ # 显示设备
    │       └── display_gralloc/ # 图形内存
    ├── mpp/                    # MPP 媒体处理
    └── rga/                    # RGA 2D 加速
```

## rk3566/ - RK3566 AIoT 平台

**定位**: 中端多媒体平台，四核 A55，Mali-G52

**文件统计**: ~52 个源文件

**内核类型**: Linux

```
rk3566/
├── soc.gni                     # SoC 构建配置
├── LICENSE                     # 许可证
├── README.md                   # 平台说明
└── hardware/                   # 硬件模块
    ├── display/                # 显示模块
    │   └── src/
    │       ├── display_device/ # 显示设备 (VDI 实现)
    │       └── display_gralloc/ # 图形内存
    ├── gpu/                    # GPU 模块
    ├── isp/                    # ISP 图像处理
    ├── mpp/                    # MPP 媒体处理
    └── rga/                    # RGA 2D 加速
```

## rk3568/ - RK3568 主力平台

**定位**: 工业/边缘计算主力平台，完整多媒体支持

**文件统计**: ~235 个源文件

**内核类型**: Linux (4.19/5.10/6.6)

```
rk3568/
├── soc.gni                     # SoC 构建配置
├── hardware/                   # 硬件模块
│   ├── BUILD.gn                # 硬件组构建配置
│   ├── codec/                  # 编解码器
│   │   ├── include/            # 头文件
│   │   │   ├── hdi_mpp.h       # MPP HDI 头文件
│   │   │   ├── hdi_mpp_mpi.h   # MPP MPI 接口
│   │   │   └── ...
│   │   ├── jpeg/               # JPEG 解码
│   │   └── src/                # 源码实现
│   ├── display/                # 显示模块
│   │   └── src/
│   │       ├── display_device/ # 显示设备
│   │       │   ├── display_composer_vdi_impl.cpp
│   │       │   ├── display_composer_vdi_impl.h
│   │       │   └── ...
│   │       └── display_gralloc/ # 图形内存
│   │           ├── display_buffer_vdi_impl.cpp
│   │           └── display_buffer_vdi_impl.h
│   ├── gpu/                    # GPU 模块
│   ├── isp/                    # ISP 图像处理
│   │   └── etc/iqfiles/        # 图像质量配置文件
│   ├── mpp/                    # MPP 媒体处理
│   │   └── mpp/
│   │       ├── hdi_mpp/        # HDI 实现
│   │       └── legacy/         # 遗留接口
│   ├── omx_il/                 # OpenMAX IL 实现
│   │   ├── core/               # OMX 核心
│   │   ├── component/          # 编解码组件
│   │   ├── osal/               # OS 抽象层
│   │   └── libOMXPlugin/       # OMX 插件
│   ├── rga/                    # RGA 2D 加速
│   └── wifi/                   # WiFi 驱动配置
│       └── ap6xxx/             # AP6xxx 系列
└── hardware/codec/jpeg/        # JPEG 硬解码
    └── ...
```

## rk3588/ - RK3588 旗舰平台

**定位**: 旗舰 AI 计算平台，8K 编解码，6TOPS NPU

**文件统计**: ~605 个源文件（含内核驱动）

**内核类型**: Linux (5.10/6.6)

```
rk3588/
├── soc.gni                     # SoC 构建配置
├── include/                    # 平台头文件
├── hardware/                   # 硬件模块
│   ├── codec/                  # 编解码器
│   ├── display/                # 显示模块
│   ├── gpu/                    # GPU 模块
│   ├── isp/                    # ISP 图像处理
│   ├── mpp/                    # MPP 媒体处理
│   ├── rga/                    # RGA 2D 加速
│   └── wifi/                   # WiFi 驱动配置
└── kernel/                     # 内核驱动（466 个文件）
    ├── drivers/
    │   ├── video/rockchip/     # 视频驱动
    │   │   ├── mpp/            # MPP 内核驱动
    │   │   └── rga3/           # RGA3 驱动
    │   ├── gpu/drm/rockchip/   # DRM 显示驱动
    │   ├── media/platform/rockchip/
    │   │   ├── isp/            # ISP 驱动
    │   │   └── cif/            # CIF 接口驱动
    │   ├── net/wireless/       # 无线网卡驱动
    │   ├── pci/controller/dwc/ # PCIe 驱动
    │   └── mmc/host/           # SD/MMC 驱动
    ├── include/                # 内核头文件
    └── arch/                   # 架构相关代码
```

## 模块职责矩阵

| 模块 | common | rk3399 | rk3566 | rk3568 | rk3588 | 说明 |
|------|--------|--------|--------|--------|--------|------|
| Display | ✅ | ✅ | ✅ | ✅ | ✅ | 显示输出 |
| GPU | ✅ | ❌ | ✅ | ✅ | ✅ | 图形渲染 |
| ISP | ❌ | ❌ | ✅ | ✅ | ✅ | 图像处理 |
| MPP | ✅ | ✅ | ✅ | ✅ | ✅ | 视频编解码 |
| RGA | ✅ | ✅ | ✅ | ✅ | ✅ | 2D 加速 |
| WiFi | ✅ | ❌ | ❌ | ✅ | ✅ | 无线网络 |
| Codec | ❌ | ❌ | ❌ | ✅ | ✅ | 硬编解码 |
| OMX IL | ❌ | ❌ | ❌ | ✅ | ❌ | OpenMAX |
| Kernel Drv | ❌ | ❌ | ❌ | ❌ | ✅ | 内核驱动 |

## 文件类型统计

| 平台 | .c | .cpp | .h | 总计 |
|------|-----|------|-----|------|
| common | ~60 | ~30 | ~14 | ~104 |
| rk2206 | ~40 | ~0 | ~20 | ~60 |
| rk3399 | ~70 | ~15 | ~18 | ~103 |
| rk3566 | ~40 | ~5 | ~7 | ~52 |
| rk3568 | ~150 | ~40 | ~45 | ~235 |
| rk3588 | ~350 | ~120 | ~135 | ~605 |

## 关键文件速查

### 显示相关

| 文件 | 路径 | 说明 |
|------|------|------|
| hdi_session.cpp | `common/hardware/display/src/display_device/` | HDI 会话管理 |
| display_composer_vdi_impl.cpp | `rk3568/hardware/display/src/display_device/` | Composer VDI 实现 |
| display_buffer_vdi_impl.cpp | `rk3568/hardware/display/src/display_gralloc/` | Buffer VDI 实现 |
| drm_display.cpp | `common/hardware/display/src/display_device/` | DRM 显示实现 |

### MPP 相关

| 文件 | 路径 | 说明 |
|------|------|------|
| rk_mpi.h | `common/hardware/mpp/include/` | MPP 主接口头文件 |
| hdi_mpp_mpi.cpp | `common/hardware/mpp/mpp/hdi_mpp/` | MPP HDI 实现 |
| hdi_mpp.c | `rk3568/hardware/codec/src/` | 编解码 HDI 实现 |

### 内核驱动相关

| 文件 | 路径 | 说明 |
|------|------|------|
| mpp_service.c | `rk3588/kernel/drivers/video/rockchip/mpp/` | MPP 内核服务 |
| drm_auth.c | `common/sdk_linux/drivers/gpu/drm/` | DRM 认证 |
| gpio_driver.c | `rk2206/hdf_driver/gpio/` | HDF GPIO 驱动 |

## 相关链接

- [项目概览](00_Overview.md) - 项目定位与核心能力
- [架构说明](01_Architecture.md) - 系统架构详解
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档
