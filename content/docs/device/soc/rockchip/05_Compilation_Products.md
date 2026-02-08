# 编译产物

## 文档信息

- **目的**: 详细说明 Rockchip OpenHarmony 仓库的编译产物
- **适用范围**: 所有芯片平台
- **关键结论**: 产物包括共享库 (.so)、静态库 (.a)、内核模块和配置文件

## 产物类型概述

### 产物分类

| 类型 | 扩展名 | 说明 |
|------|--------|------|
| 共享库 | .so / .z.so | 运行时动态链接库 |
| 静态库 | .a | 编译时静态链接库 |
| 内核模块 | .ko | Linux 内核模块 |
| 配置文件 | .hcs, .json | HDF/系统配置 |
| 固件 | .bin, .fw | 芯片固件 |

## Display 模块产物

### Display Buffer 产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libdisplay_buffer_vdi_impl.so | shared | /vendor/lib64/ | Display Buffer VDI 实现 |
| libdisplay_buffer_vendor.so | shared | /vendor/lib64/ | Buffer Vendor 库 |
| libhigbm_vendor.a | static | out/ | GBM 封装静态库 |

### Display Composer 产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libdisplay_composer_vdi_impl.so | shared | /vendor/lib64/ | Display Composer VDI 实现 |
| display_composer_vendor.so | shared | /vendor/lib64/ | Composer Vendor 库 |
| display_gfx.so | shared | /vendor/lib64/ | GFX 加速库 |

### 产物依赖关系

```
/vendor/lib64/libdisplay_composer_vdi_impl.so
  ├── /vendor/lib64/display_composer_vendor.so
  │     ├── /vendor/lib64/libdisplay_buffer_vdi_impl.so
  │     ├── /vendor/lib64/librga.so
  │     └── /system/lib64/libdrm.so
  └── /system/lib64/libhilog.so
```

## MPP 模块产物

### MPP 库产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| librockchip_mpp.z.so | shared | /vendor/lib64/ | MPP 媒体处理库 |
| librockchip_mpp.z.so | shared | /vendor/lib/ | MPP 媒体处理库 (32位) |

**来源**: `rk3568/hardware/mpp/lib/` 和 `lib64/`

### MPP HDI 产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libmpp_vdi.so | shared | /vendor/lib64/ | MPP VDI 实现 |
| hdi_mpp_mpi.o | object | out/ | MPP MPI 封装 |

## GPU 模块产物

### Mali GPU 产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| mali-bifrost-g52-g7p0-ohos.so | shared | /vendor/lib64/ | Mali GPU 驱动 |
| libGLES_mali.so | shared | /vendor/lib64/ | OpenGL ES 驱动 |
| libEGL_mali.so | shared | /vendor/lib64/ | EGL 驱动 |

**来源**: `rk3568/hardware/gpu/`

## Codec 模块产物

### Codec HDI 产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libcodec_hdi.so | shared | /vendor/lib64/ | Codec HDI 实现 |
| libjpeg_decoder.so | shared | /vendor/lib64/ | JPEG 硬解码库 |

### OMX IL 产物 (RK3568)

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libOMXPlugin.so | shared | /vendor/lib64/ | OMX 插件 |
| libomx_core.so | shared | /vendor/lib64/ | OMX 核心 |
| libomx_vdec.so | shared | /vendor/lib64/ | 视频解码组件 |
| libomx_venc.so | shared | /vendor/lib64/ | 视频编码组件 |

## WiFi 模块产物

### WiFi 驱动产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| ap6xxx.ko | kernel | /vendor/lib/modules/ | AP6xxx WiFi 驱动 |
| bcmdhd.ko | kernel | /vendor/lib/modules/ | Broadcom WiFi 驱动 |
| fw_bcm4359.bin | firmware | /vendor/firmware/ | WiFi 固件 |

**来源**: `rk3568/hardware/wifi/`, `common/kernel/drivers/net/wireless/`

## ISP 模块产物

### ISP 库产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libisp.so | shared | /vendor/lib64/ | ISP 算法库 |
| libispserver.so | shared | /vendor/lib64/ | ISP 服务库 |

### ISP 配置文件

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| *.iq | config | /vendor/etc/iqfiles/ | 图像质量配置文件 |
| camer3_profiles_*.xml | config | /vendor/etc/ | 相机配置文件 |

**来源**: `rk3568/hardware/isp/etc/`

## RGA 模块产物

### RGA 库产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| librga.so | shared | /vendor/lib64/ | RGA 2D 加速库 |

**来源**: `rk3568/hardware/rga/`

## RK2206 产物 (LiteOS-M)

### HDF 驱动产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| libgpio_driver.a | static | out/ | GPIO HDF 驱动 |
| libi2c_driver.a | static | out/ | I2C HDF 驱动 |
| libspi_driver.a | static | out/ | SPI HDF 驱动 |
| libfs_driver.a | static | out/ | 文件系统 HDF 驱动 |

### 固件产物

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| rk2206.elf | elf | out/ | 可执行固件 |
| rk2206.bin | binary | out/ | 二进制固件 |
| rk2206.map | map | out/ | 符号映射文件 |

## 内核模块产物 (RK3588)

### 显示驱动模块

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| rockchipdrm.ko | kernel | /vendor/lib/modules/ | Rockchip DRM 驱动 |
| rockchip_vop2.ko | kernel | /vendor/lib/modules/ | VOP2 显示控制器 |

### MPP 驱动模块

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| mpp_service.ko | kernel | /vendor/lib/modules/ | MPP 服务驱动 |
| rkvdec2.ko | kernel | /vendor/lib/modules/ | 视频解码器驱动 |
| rkvenc.ko | kernel | /vendor/lib/modules/ | 视频编码器驱动 |

### RGA 驱动模块

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| rga3.ko | kernel | /vendor/lib/modules/ | RGA3 2D 加速驱动 |

### ISP 驱动模块

| 产物名称 | 类型 | 路径 | 说明 |
|----------|------|------|------|
| rockchip_isp.ko | kernel | /vendor/lib/modules/ | ISP 驱动 |
| rockchip_cif.ko | kernel | /vendor/lib/modules/ | CIF 接口驱动 |

**来源**: `rk3588/kernel/drivers/`

## 产物安装路径

### 系统分区

| 路径 | 内容 |
|------|------|
| /system/lib64/ | 系统共享库 (libdrm.so, libhilog.so 等) |
| /system/lib/ | 32位系统共享库 |

### Vendor 分区

| 路径 | 内容 |
|------|------|
| /vendor/lib64/ | 厂商共享库 (VDI 实现, GPU 驱动等) |
| /vendor/lib/ | 32位厂商共享库 |
| /vendor/lib/modules/ | 内核模块 (.ko) |
| /vendor/firmware/ | 固件文件 (.bin, .fw) |
| /vendor/etc/ | 配置文件 |

### 运行时加载路径

```
LD_LIBRARY_PATH:
  /vendor/lib64:/system/lib64:/system/lib

Kernel Module Path:
  /vendor/lib/modules/

Firmware Path:
  /vendor/firmware/
```

## 产物加载关系

### Display 子系统加载顺序

```
1. 内核启动
   └── rockchipdrm.ko (DRM 驱动)
       └── rockchip_vop2.ko (VOP2 控制器)

2. 系统服务启动
   └── display_composer_vendor.so
       ├── libdisplay_composer_vdi_impl.so (VDI 实现)
       ├── libdisplay_buffer_vdi_impl.so (Buffer VDI)
       └── librga.so (RGA 加速)

3. 应用调用
   └── libdisplay_composer_proxy_1.2.so (HDI 代理)
       └── libdisplay_composer_vdi_impl.so (VDI 实现)
```

### MPP 子系统加载顺序

```
1. 内核启动
   └── mpp_service.ko
       ├── rkvdec2.ko (解码器)
       └── rkvenc.ko (编码器)

2. 多媒体服务启动
   └── librockchip_mpp.z.so (MPP 库)
       └── hdi_mpp_mpi.cpp (HDI 封装)

3. 应用调用
   └── libcodec_hdi.so (Codec HDI)
       └── librockchip_mpp.z.so
```

## 产物版本信息

### 库版本标识

```bash
# 查看共享库版本
readelf -V /vendor/lib64/librockchip_mpp.z.so

# 查看符号表
readelf -s /vendor/lib64/libdisplay_composer_vdi_impl.so | head -20

# 查看依赖关系
readelf -d /vendor/lib64/display_composer_vendor.so | grep NEEDED
```

### 内核模块版本

```bash
# 查看模块信息
modinfo /vendor/lib/modules/mpp_service.ko

# 查看模块依赖
modprobe --show-depends mpp_service
```

## 产物调试信息

### 符号文件

| 产物 | 符号文件 | 说明 |
|------|----------|------|
| librockchip_mpp.z.so | librockchip_mpp.z.so.sym | MPP 符号文件 |
| display_composer_vendor.so | display_composer_vendor.so.sym | Composer 符号文件 |

### 调试构建

```gn
# 启用调试符号
cflags = [
  "-g",
  "-O0",
  "-DDEBUG",
]

# 生成符号文件
strip --strip-debug -o output.so input.so
```

## 相关链接

- [GN 构建系统](04_GN_Build.md) - 构建配置说明
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档
