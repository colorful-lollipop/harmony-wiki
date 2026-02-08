# 配置参数速查

## GN 构建参数

### 板级配置 (config.gni)

| 参数 | Neptune100 | RK3568 | DAYU210 | 说明 |
|------|------------|--------|---------|------|
| `board_arch` | ck803 | armv8-a | armv8-a | CPU 架构 |
| `board_cpu` | ck804ef | cortex-a55 | cortex-a55 | CPU 型号 |
| `board_toolchain_type` | gcc | clang | clang | 工具链类型 |
| `board_fpu` | - | neon-fp-armv8 | neon-fp-armv8 | FPU 类型 |

**证据**: `/rk3568/config.gni:15-18`

---

### 设备配置 (device.gni)

| 参数 | RK3568 | DAYU210 | 说明 |
|------|--------|---------|------|
| `soc_company` | rockchip | rockchip | SoC 厂商 |
| `soc_name` | rk3568 | rk3588 | SoC 型号 |
| `is_support_boot_animation` | true | true | 开机动画 |
| `is_support_graphic` | true | true | 图形显示 |
| `is_support_codec` | true | true | 硬件编解码 |
| `is_support_v4l2` | true | true | V4L2 摄像头 |
| `is_support_mpi` | false | - | MPI 支持 |

**证据**: `/rk3568/device.gni:14-45`

---

### Feature Flags

| 宏 | 定义位置 | 默认值 | 用途 |
|-----|----------|--------|------|
| `SUPPORT_V4L2` | device.gni:44 | true | V4L2 摄像头支持 |
| `CAMERA_DEVICE_UTEST` | BUILD.gn | - | 摄像头单元测试 |
| `HITRACE_LOG_ENABLED` | BUILD.gn | - | HiTrace 日志 |
| `CAMERA_BUILT_ON_USB` | BUILD.gn | - | USB 摄像头 |
| `GST_DISABLE_DEPRECATED` | BUILD.gn | - | GStreamer 废弃 API |
| `HAVE_CONFIG_H` | BUILD.gn | - | 配置头文件 |

---

## 内核构建参数

### LiteOS-M (Neptune100)

| 参数 | 值 | 证据 |
|------|-----|------|
| `LOSCFG_PLATFORM` | - | - |
| `LOSCFG_BOARD_NEPTUNE100` | - | neptune100_defconfig |

### Linux (RK3568/DAYU210)

**构建脚本**: `kernel/build_kernel.sh`

| 参数 | 值 |
|------|-----|
| `kernel_version` | 5.10 (DAYU210) |
| `enable_lto_O0` | 条件启用 |
| `enable_ramdisk` | 条件启用 |
| `enable_mesa3d` | 条件启用 |
| `enable_absystem` | 条件启用 |
| `build_variant` | user/eng |

---

## 工具链配置

| 开发板 | 工具链 | 证据 |
|--------|--------|------|
| Neptune100 | csky-elfabiv2-gcc | neptune100/liteos_m/config.gni |
| RK3568 | clang | rk3568/config.gni |
| DAYU210 | clang | dayu210/config.gni |
| NearLink | riscv32-linux-musl-gcc | nearlink_dk_3863/liteos_m/config.gni |

---

## HDF 驱动配置

### hcs 文件

| 文件 | 开发板 | 用途 |
|------|--------|------|
| `neptune100.hcs` | Neptune100 | 设备配置 |
| `init.rk3568.cfg` | RK3568 | 初始化配置 |
| `init.dayu210.cfg` | DAYU210 | 初始化配置 |

---

## 相机 ISP 版本

| ISP 版本 | 开发板 | 传感器 | 证据 |
|---------|--------|--------|------|
| ISP V5 | RK3568 | IMX600 | `/rk3568/camera/vdi_impl/v4l2/device_manager/include/rkispv5.h` |
| ISP V6 | DAYU210 | - | `/dayu210/camera/vdi_impl/v4l2/device_manager/include/rkispv6.h` |

---

## 编译命令速查

| 开发板 | 命令 |
|--------|------|
| RK3568 | `hb set` -> `hihope` -> `rk3568`; `hb build -f` |
| DAYU210 | `./build.sh --product-name dayu210` |
| Neptune100 | 参考 device_soc_winnermicro |

---

## 产物路径

| 开发板 | 路径 |
|--------|------|
| RK3568 | `out/rk3568/packages/phone/images/` |
| DAYU210 | `out/rk3588/packages/phone/images/` |

---

## 相关文档

- [04_GN_Build](04_GN_Build.md) - 构建配置详解
- [05_Board_Configurations](05_Board_Configurations.md) - 开发板配置
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动
