# GN 构建系统

## 文档信息

- **目的**: 详细说明 Rockchip OpenHarmony 仓库的 GN 构建系统配置
- **适用范围**: 所有芯片平台
- **关键结论**: 使用 OpenHarmony 标准 GN 构建系统，各芯片平台独立配置

## 构建系统概述

### GN 简介

GN (Generate Ninja) 是 OpenHarmony 的标准构建系统，用于：
- 定义编译目标 (targets)
- 管理依赖关系 (deps)
- 配置编译选项 (cflags, include_dirs)
- 指定输出产物

### 构建文件类型

| 文件类型 | 说明 | 示例 |
|----------|------|------|
| BUILD.gn | 构建目标定义 | `hardware/BUILD.gn` |
| .gni | 构建配置导入 | `soc.gni`, `board.gni` |
| ohos.gni | OpenHarmony 标准配置 | 系统级 |

## 根构建配置

### 根目录 BUILD.gn

**文件**: `/Volumes/lexar/code/d/work/oh/device/soc/rockchip/BUILD.gn`

```gn
# 仅适用于 LiteOS-M 内核 (RK2206)
if (ohos_kernel_type == "liteos_m") {
  import("//kernel/liteos_m/liteos.gni")
  module_name = get_path_info(rebase_path("."), "name")
  module_group(module_name) {
    modules = []
    if (defined(LOSCFG_SOC_SERIES_RK22XX)) {
      modules += [ "rk2206" ]
    }
  }
}
```

**说明**: 根 BUILD.gn 仅处理 RK2206 (LiteOS-M)，其他平台使用独立的芯片级配置。

## 芯片级构建配置

### RK3568 构建配置

#### soc.gni

**文件**: `rk3568/soc.gni`

```gn
# USB 默认配置路径
usb_default_config_path = "//device/soc/rockchip/common/hal/usb/rk3568/include"

# 显示设备 HAL 路径
display_device_hal = "soc/rockchip/rk3568/hardware"
```

#### hardware/BUILD.gn

**文件**: `rk3568/hardware/BUILD.gn`

```gn
import("//build/ohos.gni")

# 根据内核版本选择 ISP 目录
if (linux_kernel_version == "linux-6.6") {
  ISP_VERSION_DIR = "//device/soc/rockchip/rk3568/hardware/isp-$linux_kernel_version:isp"
} else {
  ISP_VERSION_DIR = "//device/soc/rockchip/rk3568/hardware/isp:isp"
}

# 硬件组目标
group("hardware_group") {
  deps = [
    "//device/soc/rockchip/rk3568/hardware/gpu:mali-bifrost-g52-g7p0-ohos",
    "//device/soc/rockchip/rk3568/hardware/mpp:mpp",
    "//device/soc/rockchip/rk3568/hardware/wifi:ap6xxx",
    ISP_VERSION_DIR,
  ]
}
```

### RK2206 构建配置

#### board.gni

**文件**: `rk2206/board.gni`

```gn
# 路径定义
sdk_path = "//device/soc/rockchip/rk2206/sdk_liteos"
adapter_path = "//device/soc/rockchip/rk2206/adapter"
kernel_path = "//kernel/liteos_m"
hilog_path = "//base/hiviewdfx/hilog_lite"
hal_path = "//device/soc/rockchip/rk2206/hardware"
```

## Display 模块构建配置

### display/BUILD.gn

**文件**: `rk3568/hardware/display/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//drivers/hdf_core/adapter/uhdf2/uhdf.gni")

root_path = "../../../../../.."

#============================
# Display Buffer 模块
#============================

group("display_buffer_model") {
  deps = [
    ":libdisplay_buffer_vdi_impl",
    ":libdisplay_buffer_vendor",
  ]
}

# VDI 实现库
ohos_shared_library("libdisplay_buffer_vdi_impl") {
  sources = [ "src/display_gralloc/display_buffer_vdi_impl.cpp" ]
  
  include_dirs = [
    "./src/display_gralloc",
    "${root_path}/drivers/peripheral/base",
    "${root_path}/drivers/interface/display/buffer",
    # ... 其他 include
  ]
  
  output_name = "libdisplay_buffer_vdi_impl"
  
  cflags = [
    "-DGRALLOC_GBM_SUPPORT",
    "-Wno-macro-redefined",
  ]
  
  deps = [ ":libdisplay_buffer_vendor" ]
  
  external_deps = [
    "c_utils:utils",
    "drivers_interface_display:display_buffer_idl_headers",
    "hdf_core:libhdf_utils",
    "hilog:libhilog",
  ]
  
  install_enable = true
  install_images = [ chipset_base_dir ]
  subsystem_name = "hdf"
  part_name = "rockchip_products"
}

# Buffer Vendor 库
ohos_shared_library("libdisplay_buffer_vendor") {
  sources = [ "src/display_gralloc/display_gralloc_gbm.cpp" ]
  output_name = "libdisplay_buffer_vendor"
  # ... 配置
}

# GBM 静态库
ohos_static_library("libhigbm_vendor") {
  sources = [ "src/display_gralloc/hi_gbm.cpp" ]
  output_name = "libhigbm_vendor"
  # ... 配置
}

#============================
# Display Composer 模块
#============================

group("display_composer_model") {
  deps = [
    ":display_composer_vendor",
    ":display_gfx",
    ":libdisplay_composer_vdi_impl",
  ]
}

# Composer VDI 实现库
ohos_shared_library("libdisplay_composer_vdi_impl") {
  sources = [ "src/display_device/display_composer_vdi_impl.cpp" ]
  output_name = "libdisplay_composer_vdi_impl"
  # ... 配置
}

# Composer Vendor 库
ohos_shared_library("display_composer_vendor") {
  sources = [
    "src/display_device/drm_connector.cpp",
    "src/display_device/drm_crtc.cpp",
    "src/display_device/drm_device.cpp",
    "src/display_device/drm_display.cpp",
    "src/display_device/hdi_session.cpp",
    # ... 其他源文件
  ]
  output_name = "display_composer_vendor"
  # ... 配置
}

# GFX 库
ohos_shared_library("display_gfx") {
  sources = [ "src/display_gfx/display_gfx.c" ]
  output_name = "display_gfx"
  # ... 配置
}
```

## MPP 模块构建配置

### mpp/BUILD.gn

**文件**: `rk3568/hardware/mpp/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")

# 预编译共享库
ohos_prebuilt_shared_library("libmpp") {
  if (target_cpu == "arm") {
    source = "lib/librockchip_mpp.z.so"
  } else {
    source = "lib64/librockchip_mpp.z.so"
  }
  innerapi_tags = [ "passthrough_indirect" ]
  install_images = [ chipset_base_dir ]
  part_name = "rockchip_products"
  install_enable = true
}

group("mpp") {
  deps = [ ":libmpp" ]
}
```

## Target 类型说明

### ohos_shared_library

生成共享库 (.so)：

```gn
ohos_shared_library("target_name") {
  sources = [ "source.cpp" ]
  include_dirs = [ "include" ]
  deps = [ ":dependency" ]
  external_deps = [ "module:lib" ]
  output_name = "liboutput"
  install_enable = true
  install_images = [ chipset_base_dir ]
}
```

### ohos_static_library

生成静态库 (.a)：

```gn
ohos_static_library("target_name") {
  sources = [ "source.cpp" ]
  output_name = "liboutput"
}
```

### ohos_prebuilt_shared_library

使用预编译共享库：

```gn
ohos_prebuilt_shared_library("target_name") {
  source = "path/to/lib.so"
  install_images = [ chipset_base_dir ]
}
```

### group

组合多个 targets：

```gn
group("group_name") {
  deps = [
    ":target1",
    ":target2",
  ]
}
```

## 关键 Targets 列表

### RK3568 主要 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| hardware_group | group | - | 硬件模块组合 |
| mali-bifrost-g52-g7p0-ohos | prebuilt | .so | Mali GPU 驱动 |
| libmpp | prebuilt | .so | MPP 媒体库 |
| ap6xxx | prebuilt | .so | WiFi 驱动 |
| isp | prebuilt | .so | ISP 驱动 |
| libdisplay_buffer_vdi_impl | shared | .so | Display Buffer VDI |
| libdisplay_buffer_vendor | shared | .so | Buffer Vendor |
| libhigbm_vendor | static | .a | GBM 封装 |
| libdisplay_composer_vdi_impl | shared | .so | Display Composer VDI |
| display_composer_vendor | shared | .so | Composer Vendor |
| display_gfx | shared | .so | GFX 加速 |

### RK2206 主要 Targets

| Target | 类型 | 说明 |
|--------|------|------|
| gpio_driver | driver | GPIO HDF 驱动 |
| i2c_driver | driver | I2C HDF 驱动 |
| spi_driver | driver | SPI HDF 驱动 |
| fs_driver | driver | 文件系统 HDF 驱动 |

## 依赖关系

### Display 模块依赖

```
libdisplay_buffer_vdi_impl
  ├── libdisplay_buffer_vendor
  │     └── libhigbm_vendor
  │           └── libdrm
  └── external: drivers_interface_display

libdisplay_composer_vdi_impl
  ├── display_composer_vendor
  │     ├── libdisplay_buffer_vdi_impl
  │     ├── librga
  │     └── libdrm
  └── external: drivers_interface_display
```

### 外部依赖

| 模块 | 外部依赖 | 说明 |
|------|----------|------|
| Display | drivers_interface_display | HDI 接口定义 |
| Display | c_utils | C 工具库 |
| Display | hdf_core | HDF 核心库 |
| Display | hilog | 日志库 |
| Display | libdrm | DRM 库 |

## 构建配置参数

### 编译选项 (cflags)

| 选项 | 说明 |
|------|------|
| -DGRALLOC_GBM_SUPPORT | 启用 GBM 支持 |
| -Wno-macro-redefined | 忽略宏重定义警告 |
| -Wno-error=unused-function | 未使用函数不作为错误 |
| -Wno-error=missing-braces | 缺少括号不作为错误 |

### 安装配置

| 参数 | 说明 |
|------|------|
| install_enable | 是否安装到镜像 |
| install_images | 安装目标镜像 (chipset_base_dir) |
| innerapi_tags | 内部 API 标签 (passthrough) |
| subsystem_name | 子系统名称 (hdf) |
| part_name | 部件名称 (rockchip_products) |

## 构建命令

### 完整构建

```bash
# 进入 OpenHarmony 根目录
cd /path/to/openharmony

# 执行构建
./build.sh --product {product_name} --target-cpu {cpu}

# 示例: 构建 RK3568
./build.sh --product rk3568 --target-cpu arm64
```

### 单独构建模块

```bash
# 构建指定 target
gn gen out --args="target_cpu=\"arm64\""
ninja -C out libdisplay_composer_vdi_impl
```

## 相关链接

- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [编译产物](05_Compilation_Products.md) - 输出文件说明
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档
