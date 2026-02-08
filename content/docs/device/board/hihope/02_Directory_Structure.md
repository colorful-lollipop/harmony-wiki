# 目录结构

## 顶层结构

```
device_board_hihope/
├── neptune100/              # Neptune100 开发板 (LiteOS-M)
├── rk3568/                  # DAYU200 开发板 (RK3568, Linux)
├── dayu210/                 # DAYU210 开发板 (RK3588, Linux)
├── nearlink_dk_3863/       # 星闪开发板 (WS63, LiteOS-M)
├── hcs/                     # HDF 设备配置
├── shields/                 # 板级扩展配置
├── picture/                 # 产品图片资源
├── BUILD.gn                 # 根构建配置
├── Kconfig.liteos_m.boards
├── Kconfig.liteos_m.defconfig.boards
├── Kconfig.liteos_m.shields
├── README_zh.md            # 项目说明
└── wiki/                   # 本文档档
```

---

## Neptune100 目录

> 轻量级 IoT 开发板配置，驱动代码在 `device_soc_winnermicro` 仓库

```
neptune100/
├── BUILD.gn                          # 模块组配置
│   证据: line 17-19 `module_group(module_name) { modules = ["liteos_m"] }`
├── Kconfig.liteos_m.board            # 内核配置菜单
├── Kconfig.liteos_m.defconfig.board  # 默认配置
├── neptune100_defconfig              # 默认配置参数
├── ohos.build                        # OpenHarmony 构建标识
├── liteos_m/
│   ├── BUILD.gn                      # 内核模块
│   │   证据: line 18 `kernel_module(module_name)`
│   └── config.gni                    # 板级配置
│       - board_arch = "ck803"
│       - board_cpu = "ck804ef"
│       - board_toolchain_type = "gcc"
└── README_zh.md                      # 开发板文档
```

---

## RK3568 目录

> 完整多媒体开发板，包含音频/相机/WiFi 驱动

```
rk3568/
├── BUILD.gn                          # 模块组定义
│   证据: line 18-24 `group("rk3568_group") { deps = [...] }`
├── device.gni                        # 设备配置
│   证据: line 14-15 `soc_company = "rockchip"`, `soc_name = "rk3568"`
├── config.gni                        # 板级配置
│   - board_arch = "armv8-a"
│   - board_cpu = "cortex-a55"
│   - board_toolchain_type = "clang"
├── audio_alsa/                       # ALSA 音频适配
│   ├── common.h
│   ├── vendor_capture.c              # 音频采集
│   └── vendor_render.c               # 音频播放
├── audio_drivers/                    # HDF 音频驱动
│   ├── codec/rk809_codec/
│   │   ├── include/
│   │   │   └── rk809_codec_impl.h
│   │   └── src/
│   │       ├── rk809_codec_adapter.c     # HDF 驱动入口
│   │       │   证据: `HdfDriverEntry g_Rk809DriverEntry`
│   │       └── rk809_codec_impl.c
│   ├── dai/
│   │   ├── include/
│   │   │   └── rk3568_dai_ops.h
│   │   └── src/
│   │       ├── rk3568_dai_adapter.c      # HDF 驱动入口
│   │       └── rk3568_dai_ops.c
│   ├── dsp/
│   │   └── rk3568_dsp_adapter.c         # HDF 驱动入口
│   ├── headset_monitor/
│   │   └── analog_headset_core.c         # 耳机监测
│   └── soc/
│       └── rk3568_dma_adapter.c          # DMA 驱动
├── camera/vdi_impl/v4l2/            # V4L2 相机框架
│   ├── demo/
│   │   └── ohos_camera_demo           # 演示程序
│   ├── device_manager/
│   │   ├── include/
│   │   │   ├── imx600.h               # 传感器配置
│   │   │   ├── rkispv5.h              # ISP V5 接口
│   │   │   └── project_hardware.h
│   │   └── src/
│   │       ├── rkispv5.cpp            # ISP V5 实现
│   │       └── camera_device_manager.cpp
│   └── pipeline_core/
│       └── src/node/
│           ├── rk_codec_node.cpp      # 编解码节点
│           ├── rk_face_node.cpp        # 人脸检测
│           └── rk_exif_node.cpp        # EXIF 处理
├── cfg/                              # 系统配置
│   └── init.rk3568.cfg               # 初始化配置
│       证据: `init_board_config.json`
├── distributedhardware/              # 分布式硬件
│   └── distributed_hardware_components_cfg.json
├── kernel/                           # Linux 内核构建
│   └── build_kernel.sh               # 内核构建脚本
├── updater/                          # 升级模块
│   └── updater_files                 # 升级配置
└── wifi/bcmdhd_wifi6/                # WiFi6 驱动
    └── hdfadapt/
        ├── hdf_driver_bdh_register.c # HDF 驱动注册
        ├── hdf_bdh_mac80211.c         # MAC 层
        └── net_bdh_adpater.c          # 网络适配
```

---

## DAYU210 目录

> RK3588 高性能 AI 开发板

```
dayu210/
├── BUILD.gn                          # 模块组定义
│   证据: line 18-24 `group("dayu210_group") { deps = [...] }`
├── device.gni                        # 设备配置
│   证据: line 14-15 `soc_company = "rockchip"`, `soc_name = "rk3588"`
├── config.gni                        # 板级配置
│   - board_arch = "armv8-a"
│   - board_cpu = "cortex-a55"
├── audio_drivers/                    # HDF 音频驱动
│   ├── accessory/es8323/
│   │   ├── include/
│   │   │   └── es8323_adapter.h
│   │   └── src/
│   │       └── es8323_adapter.c      # HDF 驱动入口
│   ├── dai/
│   │   └── rk3588_dai_adapter.c
│   └── soc/
│       └── rk3588_dma_adapter.c
├── camera/vdi_impl/v4l2/            # V4L2 相机框架
│   ├── device_manager/
│   │   ├── include/
│   │   │   ├── rkispv6.h             # ISP V6 接口 (区别于 RK3568)
│   │   │   └── project_hardware.h
│   │   └── src/
│   │       └── rkispv6.cpp            # ISP V6 实现
│   └── pipeline_core/
│       └── src/node/
│           ├── rk_codec_node.cpp
│           ├── rk_face_node.cpp
│           └── rk_exif_node.cpp
├── cfg/                              # 系统配置
│   └── init.dayu210.cfg
├── distributedhardware/              # 分布式硬件
├── kernel/                           # Linux 5.10 + 补丁
│   └── kernel_patch/linux-5.10/
│       ├── dayu210_patch/
│       │   ├── kernel.patch          # 内核补丁
│       │   └── hdf.patch             # HDF 补丁
│       └── apply_patch.sh            # 补丁应用脚本
├── loader/                           # 引导加载器
│   └── README_zh.md
├── startup/reboot_loader/            # 启动控制
│   ├── BUILD.gn
│   └── reboot_loader.c               # 重启加载器实现
├── uboot/                            # U-Boot 构建
│   ├── BUILD.gn
│   ├── README_zh.md
│   └── rk3588-uboot溯源说明.md
└── updater/                          # 升级模块
    └── updater_files
```

---

## hcs 目录

> HDF 设备配置描述 (Hardware Configuration Source)

```
hcs/
└── BUILD.gn                          # HDF 驱动模块
    证据: line 17-24 `hdf_driver(module_name) { hcs_sources = [...] }`
    └── neptune100.hcs               # 设备配置源文件
```

---

## shields 目录

> 板级扩展配置

```
shields/
├── BUILD.gn                          # 模块组
└── neptune100/
    ├── BUILD.gn                      # HDF 驱动
    │   证据: line 18-19 `hdf_driver("neptune100")`
    ├── Kconfig.liteos_m.shield       # Shield 配置
    ├── Kconfig.liteos_m.defconfig.shield
    └── neptune100.hcs               # 设备配置
```

---

## 代码规模统计

| 目录 | .c | .cpp | .h | 总行数 | 复杂度 |
|------|-----|------|-----|--------|--------|
| rk3568 | 22 | 7 | 24 | ~11,562 | 高 |
| dayu210 | 10 | 6 | 14 | ~4,877 | 中 |
| shields | 0 | 0 | 0 | - | 低 |
| hcs | 0 | 0 | 0 | - | 低 |
| neptune100 | 0 | 0 | 0 | - | 低 |

---

## 模块职责速查

| 目录 | 主要职责 | 关键文件 |
|------|----------|----------|
| audio_drivers | HDF 音频驱动 | HdfDriverEntry |
| audio_alsa | ALSA 标准接口 | vendor_capture/render.c |
| camera | V4L2 相机框架 | rkispv5/v6.cpp, rk_codec_node.cpp |
| wifi | WiFi6 HDF 适配 | hdf_bdh_mac80211.c |
| kernel | 内核构建脚本 | build_kernel.sh |
| cfg | 系统初始化 | init.*.cfg |
| distributedhardware | 分布式配置 | JSON 配置文件 |
| updater | OTA 升级 | updater_files |
| shields | 板级扩展 | neptune100.hcs |
| hcs | HDF 设备描述 | .hcs 文件 |
